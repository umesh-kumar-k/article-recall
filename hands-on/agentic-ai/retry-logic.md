---
aliases:
  - Retry Logic
Source 1: https://www.langchain.com/blog/fault-tolerance-in-langgraph
Source 2:
Source 3:
---
[Code Examples](./code/retry.md) 

# Fault Tolerance in LangGraph: Retries, Timeouts, and Error Handlers

* **Key Points:**
  - In the real world, agents hit errors that prototypes never see: network failures, tool call errors, LLM rate limits.
  - Writing the happy path is usually the easy part. The error handling boilerplate that makes it survive in production (retries, timeouts, fallbacks) is often longer than the business logic itself.
  - LangGraph models your agent as a set of discrete steps (nodes), organized as a graph.
  - Because LangGraph controls execution, it's also where you handle what happens when any of those steps fail.
  - The three primitives LangGraph gives you for fault tolerance are: RetryPolicy: automatic retries with backoff/jitter for transient errors. TimeoutPolicy: a wall-clock or progress-based cap on a node attempt. error_handler: a node that runs after retries are exhausted, with the failure context attached.
  - In LangGraph, you define your agent by adding nodes and edges to a StateGraph. All three primitives attach directly to a node via add_node, so your fault tolerance config lives right next to the logic it protects.

## Starting from retries
* **Key Points:**
  - Transient failures are the most common kind of failure in any non-trivial graph: an LLM provider returns a 5xx, a vector store hits a connection reset, a downstream HTTP service is briefly unavailable. Every one of these is fundamentally a "try again in a moment and it'll probably work" kind of error.
  - Without first-class support you end up writing the same wrapper inside every node.
  - LangGraph's RetryPolicy removes that boilerplate. It applies per node attempt, with exponential backoff, optional jitter, and a configurable predicate for which exceptions count as retryable.
  - The default retry_on is intentionally conservative: it retries ConnectionError, 5xx responses from httpx/requests, and a few generic transient categories.
  - By default it does not retry ValueError, TypeError, RuntimeError, etc., which are almost always programming bugs.
  - The retry_on spec can be a collection of error types or a callable that checks an error at runtime to see if it matches retry criteria.
* **Technical Entities (Classes/Functions/APIs):** `RetryPolicy`, `initial_interval`, `backoff_factor`, `max_interval`, `max_attempts`, `jitter`, `retry_on`
* **Code Snippet:**
```python
from langgraph.types import RetryPolicy

policy = RetryPolicy(
    initial_interval=0.5,
    backoff_factor=2.0,
    max_interval=128.0,
    max_attempts=3,
    jitter=True,
    retry_on=(ConnectionError, TimeoutError),  # or a callable
)
```

## Timeout: a special case of "transient failure"
* **Key Points:**
  - A timeout is really just "the attempt is treated as a transient failure because it's been hanging too long." Without an explicit timeout, a stuck HTTP call or a frozen subprocess can hang a graph run indefinitely.
  - LangGraph's TimeoutPolicy supports two types of timeouts: run_timeout is a hard wall-clock cap on a single attempt. Useful when you simply do not care to ever wait more than N seconds for a node. idle_timeout resets on every "progress" signal: channel writes, streamed chunks (automatically emitted from LangChain LLM models), child task events, LangChain callback events. Long-running but actively-streaming work doesn't trip it, but a truly hung call does.
  - Internally, it relies on "heartbeat" for every signal. If you control the work and emit your own progress beats, you can switch to refresh_on="heartbeat" and explicitly call runtime.heartbeat() from inside the node.
  - When a timeout fires, the node attempt is cancelled and a NodeTimeoutError is raised.
* **Technical Entities (Classes/Functions/APIs):** `TimeoutPolicy`, `run_timeout`, `idle_timeout`, `refresh_on`, `runtime.heartbeat()`, `NodeTimeoutError`
* **Code Snippet:**
```python
from langgraph.types import TimeoutPolicy

TimeoutPolicy(
    run_timeout=30.0,    # hard wall-clock cap on a single attempt
    idle_timeout=5.0,    # max time without observable progress
    refresh_on="auto",   # or "heartbeat"
)
```

## Error handlers: when retries aren't enough
* **Key Points:**
  - Retries handle "this will probably work in 5 seconds." However, they don't handle the cases where retry exhaustion and you need to run some logic.
  - In LangGraph, this is now supported naturally: It only fires after retries are exhausted. This is the property that makes the feature actually useful. If you want to run on every exception, you'd just need to write a try/except inside the node.
  - The failure context is injected. The handler can use parameter as NodeError to get the failing node's name plus the exception (error.node, error.error).
  - The transition is atomic. When the original node fails, its ERROR write is committed to the checkpoint, and the handler task is scheduled as a new task in the same step.
  - If the host process crashes mid-handler, next time it will resume the run re-schedules the handler, not the original failing node.
  - The error handler runs in the same execution cycle. When a node fails, the error handler is scheduled immediately alongside any other nodes that were already running in that step. It doesn't wait for them to finish, and they don't wait for it.
  - You can set a default handler for every node. set_node_defaults applies to every regular node that doesn't specify its own, but a per-node error_handler= always wins.
  - You can't set another error handler for an error handler. So you don't get infinite-recursion behavior.
* **Technical Entities (Classes/Functions/APIs):** `NodeError`, `error_handler`, `error.node`, `error.error`, `set_node_defaults`
* **Code Snippet:**
```python
from langgraph.errors import NodeError

def on_call_llm_failed(state: State, error: NodeError) -> State:
    log.error("call_llm failed after retries:%s", error.error)
    return {"status": "llm_unavailable"}

  StateGraph(State)
  .add_node(
      "call_llm",
      call_llm,
      retry_policy=RetryPolicy(max_attempts=4),
      error_handler=on_call_llm_failed,
  )
```

## Putting it together: fault tolerant flight booking
* **Key Points:**
  - The three primitives above compose naturally, but their real power shows up in workflows that involve side effects: operations that change real-world state.
  - Consider a flight booking: it's not one action, it's a sequence. Reserve a seat, process payment, issue a ticket. Each step talks to an external system. Any of them can fail.
  - The naive approach (just retry the whole thing) breaks down fast. If the reserving seat went through but the payment or issuing ticket fails, the reservation is stuck in a bad state.
  - What you actually need is to retry each step individually, and if a step exhausts its retries, undo only the steps that already ran (including failed one because it's unknown).
  - This is called the SAGA pattern, and it's a standard way to handle failures in distributed systems where you can't wrap everything in a single database transaction.
  - What this gives you: Per-step backoff retries with the configured policy, An atomic transition into compensate once any step's retries are exhausted, Persistent state tracking which steps actually completed, so compensate only undoes what needs to revert
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `START`, `END`, `Command`, `RetryPolicy`, `NodeError`, `set_node_defaults`, `add_node`, `add_edge`, `compile`, `checkpointer`
* **Code Snippet:**
```python
from typing import TypedDict, Annotated, Literal
import operator
from langgraph.graph import StateGraph, START, END
from langgraph.types import Command, RetryPolicy
from langgraph.errors import NodeError

class BookingState(TypedDict, total=False):
    booking_id: str
    passenger: str
    flight: str
    seat: str  # assigned once the seat is reserved
    amount: int  # fare to charge, in minor units
    payment_ref: str  # set once payment is captured
    ticket_no: str  # set once the ticket is issued
    completed: Annotated[list[str], operator.add]  # accumulates per-step

def to_compensate(state: BookingState, error: NodeError) -> Command:
    """Route any retry-exhausted step to the compensation node."""
    return Command(
        """include the failed node"""
        update={"completed": [f"FAILED:{error.node}"]},
        goto="compensate",
    )

def reserve_seat(state) -> BookingState:
    # Call the seat-inventory service to hold a seat for this itinerary.
    ...
    return {"seat": "12A", "completed": ["reserve_seat"]}

def process_payment(state) -> BookingState:
    # Charge the fare via the payment processor while the seat is held.
    ...
    return {"payment_ref": "pay_abc123", "completed": ["process_payment"]}

def issue_ticket(state) -> BookingState:
    # Confirm the seat and issue the ticket once payment is captured.
    ...
    return {"ticket_no": "TKT-7788", "completed": ["issue_ticket"]}

def compensate(state) -> Command[Literal["__end__"]]:
    # Inspect `state["completed"]` and undo only the steps that actually ran,
    # in reverse order, to keep the booking all-or-nothing.
    if "issue_ticket" in state["completed"]:
        void_ticket(state)
    if "process_payment" in state["completed"]:
        refund_payment(state)
    if "reserve_seat" in state["completed"]:
        release_seat(state)
    return Command(goto=END)

graph = (
    StateGraph(BookingState)
    # All steps share the same retry policy and the same fallback target;
    # per-step overrides are still possible.
    .set_node_defaults(retry_policy=RETRYABLE, error_handler=to_compensate)
    .add_node("reserve_seat", reserve_seat)
    .add_node("process_payment", process_payment)
    .add_node("issue_ticket", issue_ticket)
    .add_node("compensate", compensate)
    .add_edge(START, "reserve_seat")
    .add_edge("reserve_seat", "process_payment")
    .add_edge("process_payment", "issue_ticket")
    .add_edge("issue_ticket", END)
    .compile(checkpointer=checkpointer)
)
```

## Final words
* **Key Points:**
  - Agents are taking on more autonomy, and with that comes more power to act. They're booking flights, filing tickets, executing payments, calling internal services. The actions they take are increasingly high-consequence and difficult to reverse.
  - That raises the bar for reliability. A 1% transient failure rate is a minor inconvenience in a demo. In a production agent with dozens of steps and real-world consequences, it compounds quickly.
  - RetryPolicy, TimeoutPolicy, and error_handler are built into LangGraph so that it's easy to build an agent that's resilient to all sorts of errors. All you have to do is define policies that make sense for your use case, and the LangGraph agent runtime handles the rest.