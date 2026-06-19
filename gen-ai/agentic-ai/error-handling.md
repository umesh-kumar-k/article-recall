---
aliases:
  - Error Handling & Recovery
Source 1: https://fast.io/resources/ai-agent-error-handling/
Source 2: https://fast.io/resources/ai-agent-retry-patterns/
---

# How to Build Retry Logic for Reliable AI Agents

## Why AI Agents Need Retry Patterns
* **Key Points:**
  - AI agents fail differently than traditional software. LLM API calls fail 1-5% of the time due to rate limits, timeouts, and server errors.
  - Unlike deterministic code, AI agents encounter non-deterministic failures: partial LLM responses, tool timeouts, context window overflow, and model unavailability.
  - A single failed LLM call can cascade through a multi-agent workflow. Without retry logic, one rate limit error stops the entire pipeline. Retry patterns catch these failures, wait, and try again with smarter strategies.
  - Common AI agent failure modes: Rate limit errors (HTTP 429) from LLM providers; Timeout failures from slow tool invocations; Partial responses from context window overflow; Server errors (HTTP 500, 502, 503, 504); Network connection drops; Tool execution failures (file locks, API downtime).
  - The goal is resilience: agents should handle transient failures gracefully without manual intervention.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Core Retry Patterns for AI Agents
* **Key Points:**
  - The basic pattern: try the operation, if it fails, wait a fixed amount, then retry up to N times.
  - When to use: Low-stakes operations, infrequent failures, or when you need predictable timing.
  - Limitations: Fixed delays don't adapt to system load. All agents retry simultaneously, creating retry storms.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Exponential Backoff
* **Key Points:**
  - Retrying with exponential backoff means performing a short sleep when a failure occurs, then retrying. If the request is still unsuccessful, the sleep length increases exponentially, then the process repeats.
  - How it works: Start with a base delay (1 second). On the next retry, wait base_delay × 2. On the third retry, wait base_delay × 4. Continue doubling the wait time up to a maximum delay until the maximum number of retries is reached.
  - Why it works: This gives the external service (like the LLM API) more time to recover if it's experiencing sustained load or issues. According to AWS research on distributed systems, exponential backoff with jitter reduces retry storms by 60-80%.
  - When to use: Rate limits, server-side errors, network failures. This is the default pattern for LLM API retries.
* **Technical Entities (Classes/Functions/APIs):** `AWS`
* **Code Snippet:** None

### Exponential Backoff with Jitter
* **Key Points:**
  - Add a small random amount of time (jitter) to the exponential delay. This prevents a "thundering herd" problem where many clients retry simultaneously after a widespread transient failure, overwhelming the service again.
  - Implementation: wait_time = (base_delay * 2^attempt) + random(0, jitter_max)
  - Example: Instead of waiting exactly 2 seconds, wait 2.3 seconds. The randomness spreads retries over time.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Circuit Breaker
* **Key Points:**
  - Circuit breaker patterns should be considered for agent dependencies. The circuit has three states: Closed (normal operation), Open (failures detected, stop trying), Half-Open (testing if service recovered).
  - How it works: Monitor failure rate for a service (e.g., LLM API); If failures exceed threshold (e.g., 50% in 1 minute), open the circuit; While open, fail fast without attempting the request; After a cooldown period, enter Half-Open and test with one request; If successful, close the circuit. If failed, reopen.
  - When to use: Protecting agents from cascading failures when a dependency is consistently down. Prevents wasting time and credits on requests that will fail.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Fallback Models
* **Key Points:**
  - If the primary LLM fails repeatedly, switch to a backup model.
  - Strategy: Try your primary model first, then fall back to a faster or cheaper alternative if unavailable. Fastio works with Claude, GPT, Gemini, LLaMA, and local models, making multi-LLM fallback straightforward.
  - When to use: High-availability agent systems that can tolerate reduced quality over complete failure.
* **Technical Entities (Classes/Functions/APIs):** `Fastio`, `Claude`, `GPT`, `Gemini`, `LLaMA`
* **Code Snippet:** None

### Human Escalation
* **Key Points:**
  - Some failures can't be resolved automatically. After N retries, escalate to a human.
  - How it works: Agent detects repeated failures, creates a notification or task for a human operator, pauses the workflow until resolved.
  - When to use: Agent job failures where correctness matters more than speed (document processing, invoice generation, contract analysis).
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## How to Implement Exponential Backoff in Python
* **Key Points:**
  - Tenacity is an Apache 2.0 licensed general-purpose retrying library, written in Python, to simplify the task of adding retry behavior to just about anything.
* **Technical Entities (Classes/Functions/APIs):** `Tenacity`
* **Code Snippet:**
```python
from tenacity import retry, wait_exponential, stop_after_attempt

@retry(
    wait=wait_exponential(multiplier=1, min=2, max=60),
    stop=stop_after_attempt(5)
)
def call_llm_api(prompt):
    response = llm_client.chat(prompt)
    return response
```
```python
from tenacity import retry, wait_random_exponential

@retry(
    wait=wait_random_exponential(multiplier=1, max=60),
    stop=stop_after_attempt(5)
)
def call_llm_with_jitter(prompt):
    response = llm_client.chat(prompt)
    return response
```
```python
from tenacity import retry, retry_if_exception_type, stop_after_attempt

@retry(
    retry=retry_if_exception_type((RateLimitError, TimeoutError)),
    stop=stop_after_attempt(5)
)
def safe_llm_call(prompt):
    response = llm_client.chat(prompt)
    return response
```

## How to Choose a Retry Strategy by Failure Type
* **Key Points:**
  - Different failure types need different retry strategies.
  - Rate Limits (HTTP 429): Pattern: Exponential backoff with jitter; Base delay: 1-2 seconds; Max retries: 5-7; Why: Rate limits are temporary. Backoff gives the API time to reset.
  - Server Errors (HTTP 500, 502, 503, 504): Pattern: Exponential backoff; Base delay: 2 seconds; Max retries: 3-5; Why: Server issues may resolve quickly, but don't retry indefinitely.
  - Network Timeouts: Pattern: Simple retry with fixed delay; Delay: 5 seconds; Max retries: 2-3; Why: Network issues are often transient but may indicate a deeper problem.
  - Tool Execution Failures: Pattern: Simple retry with backoff; Delay: Depends on tool (file lock: 1s, API call: 5s); Max retries: 3; Why: Tool failures can be idempotent (safe to retry) or non-idempotent (dangerous to retry).
  - Context Window Overflow: Pattern: Fallback to model with larger context; No retry: Context is deterministic, retrying won't help; Why: Switch to a model with a larger context window, or truncate input.
  - Partial LLM Responses: Pattern: Resume generation with continuation prompt; Max attempts: 2; Why: Partial responses often mean the model hit token limits mid-generation.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Multi-Agent Retry Coordination
* **Key Points:**
  - In multi-agent systems, retry patterns need coordination to prevent cascading failures.
  - Pattern 1: Centralized Retry Queue: Failed tasks go into a shared retry queue; A coordinator agent re-dispatches after delay; Prevents individual agents from clogging the system with retries.
  - Pattern 2: Agent-Level Circuit Breakers: Each agent tracks its own failure rate; If agent A's LLM calls fail 50% of the time, agent A stops making calls; Other agents continue working normally.
  - Pattern 3: Shared State with File Locks: When multiple agents access shared files, use file locks to prevent conflicts; Fastio supports file locks for concurrent access in multi-agent systems; Agents acquire locks, retry if locked, release when done.
  - Pattern 4: Idempotent Operations: Design agent operations to be safely retryable; Use unique task IDs to detect duplicate work; Store completed task IDs to prevent re-execution.
  - Fastio's workspace model supports multi-agent collaboration with granular permissions, file locks, and audit logs to track which agent performed which action.
* **Technical Entities (Classes/Functions/APIs):** `Fastio`
* **Code Snippet:** None

## Storing Agent State for Retries
* **Key Points:**
  - Effective retry patterns need persistent state. If an agent crashes mid-workflow, where does it resume?
  - Checkpointing Strategy: Save workflow state after each successful step; On retry, load the last checkpoint and resume; Avoid re-executing completed work.
  - Where to store state: File-based: Write JSON state files to a workspace (Fastio's Intelligence Mode auto-indexes them for retrieval); Database: SQLite for local agents, PostgreSQL for distributed systems; Object storage: S3 or Fastio workspaces for large state objects.
  - State structure: {"task_id": "generate-report-2026-02-14", "status": "in_progress", "completed_steps": ["fetch_data", "analyze"], "pending_steps": ["generate_pdf", "upload"], "retry_count": 2, "last_error": "Rate limit exceeded", "last_checkpoint": "2026-02-14T10:30:00Z"}
  - Fastio's agent tier includes 50GB free storage with built-in RAG. Agents can query their own state files using natural language: "Show me all tasks that failed with rate limits in the last hour."
* **Technical Entities (Classes/Functions/APIs):** `Fastio`, `RAG`, `SQLite`, `PostgreSQL`, `S3`
* **Code Snippet:** None

## Testing Retry Logic
* **Key Points:**
  - Simulate failures in tests.
* **Technical Entities (Classes/Functions/APIs):** `pytest`
* **Code Snippet:**
```python
import pytest
from unittest.mock import Mock, patch

def test_retry_on_rate_limit():
    mock_api = Mock()
    mock_api.chat.side_effect = [
        RateLimitError("Too many requests"),
        RateLimitError("Too many requests"),
        {"response": "Success"}
    ]

result = call_llm_with_retry(mock_api, "test prompt")
    assert result["response"] == "Success"
    assert mock_api.chat.call_count == 3
```
```python
import time

def test_backoff_timing():
    start = time.time()

with pytest.raises(RateLimitError):
        call_llm_with_retry(always_fails_api, "test")

duration = time.time() - start
    assert duration > 31   # sum of waits (1+2+4+8+16)
    assert duration < 35   # allow some margin
```
```python
def test_circuit_breaker():
    circuit = CircuitBreaker(threshold=3)

for _ in range(3):  # cause 3 failures to open circuit
        with pytest.raises(Exception):
            circuit.call(failing_function)

assert circuit.state == "OPEN"

with pytest.raises(CircuitOpenError):  # verify fast-fail
        circuit.call(failing_function)
```

## Production Monitoring for Retries
* **Key Points:**
  - Track retry metrics to understand agent reliability: Key metrics: Retry rate (retries / total requests); Success after retry rate (successful retries / total retries); Average retry delay (time spent waiting); Failure types (rate limits vs server errors vs timeouts); Circuit breaker state changes.
  - Alerts to set: Retry rate > 10% (something's wrong upstream); Circuit breaker open for > 5 minutes; Max retries exhausted > 5% of the time; Agent stuck in retry loop for > 30 minutes.
  - Fastio provides audit logs for all agent actions, including file access, API calls, and workspace changes. Use webhooks to send retry events to your monitoring system in real time.
* **Technical Entities (Classes/Functions/APIs):** `Fastio`
* **Code Snippet:** None

---


# How to Handle AI Agent Errors: Best Practices for 2025

## Why Do AI Agents Fail Without Proper Error Handling?
* **Key Points:**
  - AI agent error handling is the strategy used to detect, recover from, and learn from failures during autonomous operations.
  - Unlike traditional software, AI agents face non-deterministic failures. A prompt that works once might fail the next time due to model drift, token limits, or hallucinated tool arguments.
  - According to internal benchmarks, production AI agents frequently encounter errors. These aren't just code bugs; they include API timeouts, rate limits, malformed JSON outputs, and context window overflows.
  - Without reliable recovery mechanisms, a single error can derail an entire multi-step workflow.
  - The difference between a prototype and a production-ready agent often comes down to how it handles the unexpected. Agents that anticipate common failure modes and implement structured recovery paths complete tasks at higher rates than those that crash on the first error.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## How to Build Layered Error Handling for AI Agents
* **Key Points:**
  - To build resilient agents, developers must move beyond simple try/catch blocks. The most effective agents employ a layered defense strategy that catches errors at multiple levels before they cascade.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### 1. Exponential Backoff Retries
* **Key Points:**
  - When an API call fails with a rate-limit error, retrying immediately often worsens the problem. Exponential backoff waits increasingly longer intervals between attempts, doubling the delay each time. This gives the downstream service time to recover.
  - Adding jitter (a small random offset) to each interval prevents synchronized retries from multiple agents, which is common in multi-agent deployments.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### 2. The Circuit Breaker Pattern
* **Key Points:**
  - If a specific tool or API fails consistently, a "circuit breaker" temporarily disables it. The agent then switches to a fallback tool or pauses execution, preventing cascading failures that could consume credits or corrupt data.
  - A typical circuit breaker trips after several consecutive failures and periodically re-checks availability before re-enabling the connection.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### 3. Output Validation & Correction
* **Key Points:**
  - LLMs often generate malformed JSON or invalid arguments. Instead of crashing, a well-built agent uses a "validator" step. If validation fails, the error message is fed back to the LLM as a new prompt: "You provided invalid JSON. Please correct it based on this schema."
  - In practice, giving the model one correction attempt resolves most format errors on the first retry.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## How to Manage State and Checkpointing for Recovery
* **Key Points:**
  - Long-running agent workflows are vulnerable to interruption. If an agent crashes partway through a long task, restarting from scratch is costly and inefficient. The solution is to save progress at regular intervals so recovery is fast.
  - State Checkpointing involves saving the agent's memory (context, variable values, completed steps) to persistent storage after every major action. This allows an agent to "wake up" and resume exactly where it left off.
  - A common approach is to serialize state to JSON and write it to a shared file store after each completed step. If the agent restarts, it reads the latest checkpoint and skips already-finished steps.
  - For file-heavy workflows, this means ensuring that file operations are atomic. Fastio's global file system provides immediate consistency, ensuring that if an agent writes a file, it's instantly available for the next step or a backup agent, preventing "file not found" race conditions.
* **Technical Entities (Classes/Functions/APIs):** `Fastio`
* **Code Snippet:** None

## How to Handle File System and Storage Errors
* **Key Points:**
  - File operations are a frequent source of agent failure. Agents may try to read incomplete uploads, overwrite shared files concurrently, or hallucinate non-existent paths. In multi-agent setups, file-related errors account for a large share of total failures, making this category worth special attention.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### File Locking
* **Key Points:**
  - In multi-agent systems, two agents might try to edit the same document simultaneously. Implementing file locks ensures that Agent A must finish writing and release the lock before Agent B can read or edit.
  - Fastio supports this natively through its file locking API, preventing data corruption in collaborative environments. When a lock times out, the system releases it automatically so workflows don't stall indefinitely.
* **Technical Entities (Classes/Functions/APIs):** `Fastio`
* **Code Snippet:** None

### Webhook-Based Recovery
* **Key Points:**
  - Polling for file changes is error-prone and wastes compute cycles. A better pattern uses webhooks. If an upload fails or hangs, a "file upload failed" webhook can trigger a remediation agent to retry the transfer or alert a human, rather than leaving the workflow in limbo.
  - This event-driven approach also lets you track upload completion rates across your agent fleet.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## How to Set Up Observability and Audit Logging
* **Key Points:**
  - When an agent fails, you need to know why immediately. Was it a bad prompt? A network partition? A permission error? Without visibility into what happened, you're left guessing, and fixing one problem often introduces another.
  - Comprehensive Audit Logging is non-negotiable. Every tool call, API response, and file modification should be logged with a timestamp and agent ID. Fastio's audit logs track every file interaction, giving you a forensic trail to debug exactly when and how an agent corrupted a dataset or deleted a critical asset.
  - Beyond logging, set up real-time alerts on error rate thresholds. If an agent's failure rate spikes over a short window, an automated alert can pause the workflow and notify your team before more damage accumulates.
  - Pair this with structured log formats (JSON with consistent field names) so you can query and filter across multiple agents efficiently.
* **Technical Entities (Classes/Functions/APIs):** `Fastio`
* **Code Snippet:** None