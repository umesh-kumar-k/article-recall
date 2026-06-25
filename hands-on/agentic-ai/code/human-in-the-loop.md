---
aliases:
  - Langchain HITL
---
[Langgraph Interrupts](interrupts.md) 

## Human-in-the-loop
* **Key Points:**
  - The Human-in-the-Loop (HITL) middleware lets you add human oversight to agent tool calls.
  - When a model proposes an action that might require review—for example, writing to a file or executing SQL—the middleware can pause execution and wait for a decision.
  - It does this by checking each tool call against a configurable policy. If intervention is needed, the middleware issues an interrupt that halts execution.
  - The graph state is saved using LangGraph's persistence layer, so execution can pause safely and resume later.
  - A human decision then determines what happens next: the action can be approved as-is (`approve`), modified before running (`edit`), rejected with feedback (`reject`), or responded to directly (`respond`) for "ask user" style tools.

## Interrupt decision types
* **Key Points:**
  - The middleware defines four built-in ways a human can respond to an interrupt:
  - `approve`: The action is approved as-is and executed without changes.
  - `edit`: The tool call is executed with modifications.
  - `reject`: The tool call is rejected, with an explanation added to the conversation.
  - `respond`: Tool execution is skipped; the human's message becomes the tool result.
  - The available decision types for each tool depend on the policy you configure in `interrupt_on`.
  - When multiple tool calls are paused at the same time, each action requires a separate decision.
  - Decisions must be provided in the same order as the actions appear in the interrupt request.
  - Use `reject` when the human is denying the requested action. Use `respond` only when the human is acting as the tool, such as answering an `ask_user` prompt. Do not use `respond` to deny side-effecting tools, because its message is treated as a successful tool result.
  - When editing tool arguments, make changes conservatively. Significant modifications to the original arguments may cause the model to re-evaluate its approach and potentially execute the tool multiple times or take unexpected actions.

## Configuring interrupts
* **Key Points:**
  - To use HITL, add the middleware to the agent's `middleware` list when creating the agent.
  - You configure it with a mapping of tool actions to the decision types that are allowed for each action. The middleware will interrupt execution when a tool call matches an action in the mapping.
  - You must configure a checkpointer to persist the graph state across interrupts. In production, use a persistent checkpointer like `AsyncPostgresSaver`. For testing or prototyping, use `InMemorySaver`.
  - When invoking the agent, pass a `config` that includes the **thread ID** to associate execution with a conversation thread.
* **Technical Entities (Classes/Functions/APIs):** `createAgent`, `humanInTheLoopMiddleware`, `MemorySaver`, `interruptOn`, `write_file`, `execute_sql`, `read_data`, `allowedDecisions`, `descriptionPrefix`, `checkpointer`, `AsyncPostgresSaver`, `InMemorySaver`, `thread_id`
* **Code Snippet:**
```ts
import { createAgent, humanInTheLoopMiddleware } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
    model: "gpt-5.5",
    tools: [writeFileTool, executeSQLTool, readDataTool],
    middleware: [
        humanInTheLoopMiddleware({
            interruptOn: {
                write_file: true, // All decisions (approve, edit, reject, respond) allowed
                execute_sql: {
                    allowedDecisions: ["approve", "reject"],
                    // No editing allowed
                    description: "🚨 SQL execution requires DBA approval",
                },
                // Safe operation, no approval needed
                read_data: false,
            },
            // Prefix for interrupt messages - combined with tool name and args to form the full message
            // e.g., "Tool execution pending approval: execute_sql with query='DELETE FROM...'"
            // Individual tools can override this by specifying a "description" in their interrupt config
            descriptionPrefix: "Tool execution pending approval",
        }),
    ],
    // Human-in-the-loop requires checkpointing to handle interrupts.
    // In production, use a persistent checkpointer like AsyncPostgresSaver.
    checkpointer: new MemorySaver(),
});
```

## Conditional interrupts
* **Key Points:**
  - By default, every tool call listed in `interrupt_on` pauses for review. To pause only some calls, add a `when` predicate to a tool's `InterruptOnConfig`. The predicate receives a `ToolCallRequest` and returns `True` to interrupt or `False` to auto-approve, so you can gate on the tool's arguments.
  - Conditional interrupts are currently available in Python only.
* **Technical Entities (Classes/Functions/APIs):** `when`, `InterruptOnConfig`, `ToolCallRequest`

## Responding to interrupts
* **Key Points:**
  - When you invoke the agent, it runs until it either completes or an interrupt is raised. An interrupt is triggered when a tool call matches the policy you configured in `interrupt_on`.
  - With `version="v2"`, the result is a `GraphOutput` with an `interrupts` attribute containing the actions that require review. You can then present those actions to a reviewer and resume execution once decisions are provided.
  - You must provide a thread ID to associate the execution with a conversation thread, so the conversation can be paused and resumed (as is needed for human review).
* **Technical Entities (Classes/Functions/APIs):** `Command`, `resume`, `decisions`, `type`, `approve`, `reject`, `edit`, `respond`, `editedAction`, `message`, `GraphOutput`, `interrupts`, `__interrupt__`, `actionRequests`, `reviewConfigs`
* **Code Snippet:**
```typescript
import { HumanMessage } from "@langchain/core/messages";
import { Command } from "@langchain/langgraph";

// You must provide a thread ID to associate the execution with a conversation thread,
// so the conversation can be paused and resumed (as is needed for human review).
const config = { configurable: { thread_id: "some_id" } };

// Run the graph until the interrupt is hit.
const result = await agent.invoke(
    {
        messages: [new HumanMessage("Delete old records from the database")],
    },
    config
);


// The interrupt contains the full HITL request with action_requests and review_configs
console.log(result.__interrupt__);
// > [
// >    Interrupt(
// >       value: {
// >          actionRequests: [
// >             {
// >                name: 'execute_sql',
// >                arguments: { query: 'DELETE FROM records WHERE created_at < NOW() - INTERVAL \'30 days\';' },
// >                description: 'Tool execution pending approval\n\nTool: execute_sql\nArgs: {...}'
// >             }
// >          ],
// >          reviewConfigs: [
// >             {
// >                actionName: 'execute_sql',
// >                allowedDecisions: ['approve', 'reject']
// >             }
// >          ]
// >       }
// >    )
// > ]

// Resume with approval decision
await agent.invoke(
    new Command({
        resume: { decisions: [{ type: "approve" }] }, // or "reject"
    }),
    config // Same thread ID to resume the paused conversation
);
```

### Decision types
* **Technical Entities (Classes/Functions/APIs):** `approve`, `edit`, `reject`, `respond`, `editedAction`, `message`, `ToolMessage`

## Streaming with human-in-the-loop
* **Key Points:**
  - You can stream real-time updates while the agent runs and handles interrupts using `stream_events()`.
  - Use `stream.messages` to stream LLM tokens and `stream.values` to check agent state snapshots for interrupts.
* **Technical Entities (Classes/Functions/APIs):** `stream_events()`, `stream.messages`, `stream.values`, `stream.interrupted`, `stream.interrupts`, `Command`, `resume`
* **Code Snippet:**
```typescript
import { Command } from "@langchain/langgraph";

const config = { configurable: { thread_id: "some_id" } };

// Stream agent progress and LLM tokens until interrupt
const stream = await agent.streamEvents(
    { messages: [{ role: "user", content: "Delete old records from the database" }] },
    { ...config, version: "v3" }
);
for await (const message of stream.messages) {
    for await (const token of message.text) {
        process.stdout.write(token);
    }
}

// Check whether the run paused for human input
if (stream.interrupted) {
    console.log(`\n\nInterrupt: ${JSON.stringify(stream.interrupts)}`);
}

// Resume with streaming after human decision
const resumeStream = await agent.streamEvents(
    new Command({ resume: { decisions: [{ type: "approve" }] } }),
    { ...config, version: "v3" }
);
for await (const message of resumeStream.messages) {
    for await (const token of message.text) {
        process.stdout.write(token);
    }
}
```

## Execution lifecycle
* **Key Points:**
  - The middleware defines an `after_model` hook that runs after the model generates a response but before any tool calls are executed:
  - The agent invokes the model to generate a response.
  - The middleware inspects the response for tool calls.
  - If any calls require human input, the middleware builds a `HITLRequest` with `action_requests` and `review_configs` and calls interrupt.
  - The agent waits for human decisions.
  - Based on the `HITLResponse` decisions, the middleware executes approved or edited calls, synthesizes ToolMessage's for rejected calls, returns human replies directly as ToolMessage's for `respond` decisions, and resumes execution.
* **Technical Entities (Classes/Functions/APIs):** `after_model`, `HITLRequest`, `action_requests`, `review_configs`, `interrupt`, `HITLResponse`, `ToolMessage`

## Custom HITL logic
* **Key Points:**
  - For more specialized workflows, you can build custom HITL logic directly using the interrupt primitive and middleware abstraction.
* **Technical Entities (Classes/Functions/APIs):** `interrupt`, `middleware`