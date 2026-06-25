---
aliases:
  - Streaming
Source 1: https://docs.langchain.com/oss/javascript/langgraph/streaming
Source 2: https://docs.langchain.com/oss/javascript/langgraph/event-streaming
---
## Streaming
* **Key Points:**
  - For new applications, we recommend event streaming—the typed-projection API introduced in LangGraph v1.2. Event streaming gives you separate iterators per projection (messages, values, subgraphs, output) so you can consume them independently instead of branching on `stream_mode` chunks.
  - This page covers LangGraph's stream-mode API. It exposes graph execution through stream modes such as `updates`, `values`, `messages`, `custom`, `checkpoints`, `tasks`, and `debug`. Use it when you need direct access to graph-runtime events or specific stream-mode output.

## Get started
### Basic usage
* **Key Points:**
  - LangGraph graphs expose the `stream` method to yield streamed outputs as iterators.
* **Technical Entities (Classes/Functions/APIs):** `stream`, `streamMode`
* **Code Snippet:**
```typescript
for await (const chunk of await graph.stream(inputs, {
  streamMode: "updates",
})) {
  console.log(chunk);
}
```

## Stream modes
* **Key Points:**
  - Pass one or more of the following stream modes as a list to the `stream` method:
  - `values`: Full state after each step.
  - `updates`: State updates after each step. Multiple updates in the same step are streamed separately.
  - `messages`: 2-tuples of (LLM token, metadata) from LLM calls.
  - `custom`: Custom data emitted from nodes via the `writer` config parameter.
  - `tools`: Tool-call lifecycle events (`on_tool_start`, `on_tool_event`, `on_tool_end`, `on_tool_error`).
  - `debug`: All available info throughout graph execution.
* **Technical Entities (Classes/Functions/APIs):** `stream`, `streamMode`

### Graph state
* **Key Points:**
  - Use the stream modes `updates` and `values` to stream the state of the graph as it executes.
  - `updates` streams the **updates** to the state after each step of the graph.
  - `values` streams the **full value** of the state after each step of the graph.
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `StateSchema`, `START`, `END`, `z`, `stream`, `streamMode`

### LLM tokens
* **Key Points:**
  - Use the `messages` streaming mode to stream Large Language Model (LLM) outputs **token by token** from any part of your graph, including nodes, tools, subgraphs, or tasks.
  - The streamed output from `messages` mode is a tuple `[message_chunk, metadata]` where: `message_chunk`: the token or message segment from the LLM. `metadata`: a dictionary containing details about the graph node and LLM invocation.
  - If your LLM is not available as a LangChain integration, you can stream its outputs using `custom` mode instead.
* **Technical Entities (Classes/Functions/APIs):** `ChatOpenAI`, `StateGraph`, `StateSchema`, `GraphNode`, `START`, `z`, `stream`, `streamMode`, `messages`
* **Code Snippet:**
```typescript
import { ChatOpenAI } from "@langchain/openai";
import { StateGraph, StateSchema, GraphNode, START } from "@langchain/langgraph";
import * as z from "zod";

const MyState = new StateSchema({
  topic: z.string(),
  joke: z.string().default(""),
});

const model = new ChatOpenAI({ model: "gpt-5.4-mini" });

const callModel: GraphNode<typeof MyState> = async (state) => {
  // Call the LLM to generate a joke about a topic
  // Note that message events are emitted even when the LLM is run using .invoke rather than .stream
  const modelResponse = await model.invoke([
    { role: "user", content: `Generate a joke about ${state.topic}` },
  ]);
  return { joke: modelResponse.content };
};

const graph = new StateGraph(MyState)
  .addNode("callModel", callModel)
  .addEdge(START, "callModel")
  .compile();

// The "messages" stream mode returns an iterator of tuples [messageChunk, metadata]
// where messageChunk is the token streamed by the LLM and metadata is a dictionary
// with information about the graph node where the LLM was called and other information
for await (const [messageChunk, metadata] of await graph.stream(
  { topic: "ice cream" },
  { streamMode: "messages" }
)) {
  if (messageChunk.content) {
    console.log(messageChunk.content + "|");
  }
}
```

#### Filter by LLM invocation
* **Key Points:**
  - You can associate `tags` with LLM invocations to filter the streamed tokens by LLM invocation.
* **Technical Entities (Classes/Functions/APIs):** `tags`, `stream`, `streamMode`

#### Omit messages from the stream
* **Key Points:**
  - Use the `nostream` tag to exclude LLM output from the stream entirely. Invocations tagged with `nostream` still run and produce output; their tokens are simply not emitted in `messages` mode.
  - This is useful when: You need LLM output for internal processing (for example structured output) but do not want to stream it to the client, You stream the same content through a different channel (for example custom UI messages) and want to avoid duplicate output in the `messages` stream
* **Technical Entities (Classes/Functions/APIs):** `nostream`, `withConfig`, `ChatAnthropic`

#### Filter by node
* **Key Points:**
  - To stream tokens only from specific nodes, use `stream_mode="messages"` and filter the outputs by the `langgraph_node` field in the streamed metadata:
* **Technical Entities (Classes/Functions/APIs):** `langgraph_node`

### Custom data
* **Key Points:**
  - To send **custom user-defined data** from inside a LangGraph node or tool, follow these steps: Use the `writer` parameter from the `LangGraphRunnableConfig` to emit custom data. Set `streamMode: "custom"` when calling `.stream()` to get the custom data in the stream. You can combine multiple modes (e.g., `["updates", "custom"]`), but at least one must be `"custom"`.
* **Technical Entities (Classes/Functions/APIs):** `writer`, `LangGraphRunnableConfig`, `stream`, `streamMode`, `custom`

### Tool progress
* **Key Points:**
  - Use the `tools` stream mode to receive real-time lifecycle events for tool executions. This is useful for showing progress indicators, partial results, and error states in your UI while tools are running.
  - The `tools` stream mode emits four event types: `on_tool_start` (Tool invocation begins), `on_tool_event` (Tool yields intermediate data), `on_tool_end` (Tool returns its final result), `on_tool_error` (Tool throws an error)
* **Technical Entities (Classes/Functions/APIs):** `tools`, `on_tool_start`, `on_tool_event`, `on_tool_end`, `on_tool_error`, `tool`, `async function*`, `useStream`, `ToolProgress`

#### Define tools that stream progress
* **Key Points:**
  - To emit `on_tool_event` events, define your tool function as an **async generator** (`async function*`). Each `yield` sends intermediate data to the stream, and the `return` value is used as the tool's final result.
  - Existing tools that return a `Promise` are fully compatible. They emit `on_tool_start` and `on_tool_end` events but no `on_tool_event` events.
* **Technical Entities (Classes/Functions/APIs):** `tool`, `async function*`, `yield`, `return`

#### Consume tool events server-side
* **Key Points:**
  - Pass `streamMode: ["tools"]` (or combine with other modes) to `graph.stream()`:
* **Technical Entities (Classes/Functions/APIs):** `stream`, `streamMode`, `tools`

#### Use tool progress in React with `useStream`
* **Key Points:**
  - The `useStream` hook from `@langchain/langgraph-sdk/react` exposes a `toolProgress` array when you include `"tools"` in your stream modes. Each entry is a `ToolProgress` object that tracks the current state of a running tool.
* **Technical Entities (Classes/Functions/APIs):** `useStream`, `toolProgress`, `ToolProgress`

#### `tools` vs `custom` stream mode
* **Key Points:**
  - Both stream modes can surface tool progress, but they serve different purposes:
  - `tools`—automatically emits structured lifecycle events (`on_tool_start`, `on_tool_event`, `on_tool_end`, `on_tool_error`) with no code changes needed in your tools beyond using `async function*`. The `useStream` hook provides the reactive `toolProgress` array out of the box.
  - `custom`—gives you full control over what data is emitted and when using `config.writer()`. Use this when you need freeform data that doesn't map to the tool lifecycle, or when you want to stream from nodes (not just tools).

### Subgraph outputs
* **Key Points:**
  - To include outputs from subgraphs in the streamed outputs, you can set `subgraphs: true` in the `.stream()` method of the parent graph. This will stream outputs from both the parent graph and any subgraphs.
  - The outputs will be streamed as tuples `[namespace, data]`, where `namespace` is a tuple with the path to the node where a subgraph is invoked, e.g. `["parent_node:<task_id>", "child_node:<task_id>"]`.
* **Technical Entities (Classes/Functions/APIs):** `subgraphs: true`, `stream`, `namespace`

### Debug
* **Key Points:**
  - Use the `debug` streaming mode to stream as much information as possible throughout the execution of the graph. The streamed outputs include the name of the node as well as the full state.
* **Technical Entities (Classes/Functions/APIs):** `debug`, `stream`, `streamMode`

### Multiple modes at once
* **Key Points:**
  - You can pass an array as the `streamMode` parameter to stream multiple modes at once.
  - The streamed outputs will be tuples of `[mode, chunk]` where `mode` is the name of the stream mode and `chunk` is the data streamed by that mode.
* **Technical Entities (Classes/Functions/APIs):** `stream`, `streamMode`

## Advanced
### Use with any LLM
* **Key Points:**
  - You can use `streamMode: "custom"` to stream data from **any LLM API**—even if that API does **not** implement the LangChain chat model interface.
  - This lets you integrate raw LLM clients or external services that provide their own streaming interfaces, making LangGraph highly flexible for custom setups.
* **Technical Entities (Classes/Functions/APIs):** `streamMode`, `custom`, `config.writer`
* **Code Snippet:**
```typescript
import { StateGraph, GraphNode, StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const State = new StateSchema({ result: z.string() });

const callArbitraryModel: GraphNode<typeof State> = async (state, config) => {
  // Example node that calls an arbitrary model and streams the output
  // Assume you have a streaming client that yields chunks
  // Generate LLM tokens using your custom streaming client
  for await (const chunk of yourCustomStreamingClient(state.topic)) {
    // Use the writer to send custom data to the stream
    config.writer({ custom_llm_chunk: chunk });
  }
  return { result: "completed" };
};

const graph = new StateGraph(State)
  .addNode("callArbitraryModel", callArbitraryModel)
  // Add other nodes and edges as needed
  .compile();

// Set streamMode: "custom" to receive the custom data in the stream
for await (const chunk of await graph.stream(
  { topic: "cats" },
  { streamMode: "custom" }
)) {
  // The chunk will contain the custom data streamed from the llm
  console.log(chunk);
}
```

### Disable streaming for specific chat models
* **Key Points:**
  - If your application mixes models that support streaming with those that do not, you may need to explicitly disable streaming for models that do not support it.
  - Set `streaming: false` when initializing the model.
  - Not all chat model integrations support the `streaming` parameter. If your model doesn't support it, use `disableStreaming: true` instead. This parameter is available on all chat models via the base class.
* **Technical Entities (Classes/Functions/APIs):** `streaming: false`, `disableStreaming: true`, `ChatOpenAI`


---

## Event streaming
* **Key Points:**
  - Event streaming is the recommended in-process streaming model for most LangGraph application code.
  - It returns a run stream object that can be consumed in multiple ways at the same time.

## Quickstart
* **Technical Entities (Classes/Functions/APIs):** `graph.streamEvents()`, `stream.messages`, `stream.output`
* **Code Snippet:**
```ts
const stream = await graph.streamEvents(
  { messages: [{ role: "user", content: "What is 42 * 17?" }] },
  { version: "v3" }
);

for await (const message of stream.messages) {
  for await (const token of message.text) {
    process.stdout.write(token);
  }
}

const finalState = await stream.output;
```

## How the pieces fit together
* **Key Points:**
  - The streaming stack has two main layers: Streaming emits raw graph execution events from the Pregel engine. Event streaming normalizes those events, runs them through stream transformers, and exposes typed projections.
  - The event router is the bridge between the two layers. It receives normalized Pregel events and passes each event through the registered stream transformers.
  - Built-in transformers create standard projections such as `stream.messages`, `stream.values`, `stream.subgraphs`, and `stream.output`. Custom transformers can add application-specific projections under `stream.extensions`.

## What event streaming provides
* **Key Points:**
  - The run stream exposes typed projections over one underlying event flow:
  - `stream`: Iterate every protocol event.
  - `stream.messages`: Stream chat model messages and token deltas.
  - `stream.values`: Iterate state snapshots and await the final value.
  - `stream.output`: Await the final output.
  - `stream.subgraphs`: Discover and observe nested graph executions.
  - `stream.interrupts`: Inspect human-in-the-loop interrupt payloads.
  - `stream.interrupted`: Check whether the run paused for human input.
  - `stream.extensions`: Consume custom stream transformer projections.
  - Multiple consumers can read these projections concurrently. Reading `stream.messages` does not consume events needed by `stream.values`, `stream.subgraphs`, or `stream.output`.
  - Event streaming sits one level above streaming, which exposes raw graph execution events through `stream_mode` modes such as `updates`, `values`, `messages`, `custom`, `checkpoints`, `tasks`, and `debug`. Use streaming when you need low-level access to those modes; use event streaming when application code benefits from typed projections.
* **Technical Entities (Classes/Functions/APIs):** `stream`, `stream.messages`, `stream.values`, `stream.output`, `stream.subgraphs`, `stream.interrupts`, `stream.interrupted`, `stream.extensions`

## Stream messages
* **Key Points:**
  - Use `stream.messages` for chat model output:
  - `message.text` is both an async iterable and a promise-like value. Iterate it for token-by-token output, or await it for the complete text.
* **Technical Entities (Classes/Functions/APIs):** `stream.messages`, `message.text`, `message.usage`
* **Code Snippet:**
```ts
const stream = await graph.streamEvents(input, { version: "v3" });

for await (const message of stream.messages) {
  const text = await message.text;
  const usage = await message.usage;

  console.log(text);
  console.log(usage);
}
```

## Stream subgraphs
* **Key Points:**
  - Use `stream.subgraphs` to observe nested graph work without parsing namespace strings:
  - `subgraph.graph_name` is the `name` of the compiled graph or agent.
  - A named agent dispatched from a tool (for example, a `create_agent(name=...)` invoked through the Deep Agents `task` tool) surfaces here under that name, and the `lifecycle` event that opens the scope carries a `cause` linking back to the dispatching tool call.
* **Technical Entities (Classes/Functions/APIs):** `stream.subgraphs`, `subgraph.name`, `subgraph.path`, `subgraph.messages`
* **Code Snippet:**
```ts
const stream = await graph.streamEvents(input, { version: "v3" });

for await (const subgraph of stream.subgraphs) {
  console.log(subgraph.name, subgraph.path);

  for await (const message of subgraph.messages) {
    console.log(await message.text);
  }
}
```

## Stream state
* **Key Points:**
  - Use `stream.values` to stream full state snapshots after each step:
* **Technical Entities (Classes/Functions/APIs):** `stream.values`, `stream.output`
* **Code Snippet:**
```ts
const stream = await graph.streamEvents(input, { version: "v3" });

for await (const snapshot of stream.values) {
  console.log(snapshot);
}

const finalState = await stream.output;
```

## Stream multiple projections
* **Key Points:**
  - Use concurrent consumers when you need multiple projections in JavaScript:
* **Technical Entities (Classes/Functions/APIs):** `Promise.all`
* **Code Snippet:**
```ts
await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      console.log(await message.text);
    }
  })(),
  (async () => {
    for await (const subgraph of stream.subgraphs) {
      console.log(subgraph.path);
    }
  })(),
]);
```

## Resume after an interrupt
* **Key Points:**
  - When a graph pauses for human input, inspect `stream.interrupted` and `stream.interrupts`, then resume by calling `stream_events(..., version="v3")` again with `Command`.
  - Resume requires a graph compiled with a checkpointer and a config carrying a thread ID — see persistence.
* **Technical Entities (Classes/Functions/APIs):** `stream.interrupted`, `stream.interrupts`, `Command`, `stream.output`
* **Code Snippet:**
```ts
import { Command } from "@langchain/langgraph";

let stream = await graph.streamEvents(input, { version: "v3" });

for await (const message of stream.messages) {
  console.log(await message.text);
}

if (stream.interrupted) {
  console.log(stream.interrupts);
}

stream = await graph.streamEvents(
  new Command({ resume: { decisions: [{ type: "approve" }] } }),
  { version: "v3" }
);
const finalState = await stream.output;
```

## Stream all protocol events
* **Key Points:**
  - Use the run object itself when you want the raw protocol event stream:
  - Each event is a `ProtocolEvent` envelope wrapping a channel-specific payload. The same shape is what a transformer's `process(event)` receives.
  - The `namespace` is a path from the root graph to the scope that emitted the event. The root is the empty array `[]`. Each child execution adds one `"name:runtime_id"` segment, so a nested tool call inside a subgraph looks like `["researcher:6f4d", "tools:91ac"]`. The name before `:` is the stable graph or node name; the suffix is a per-invocation runtime ID.
  - Filter raw events by namespace yourself when you only care about a specific subtree — `stream.subgraphs` already does this for nested graph executions.
* **Technical Entities (Classes/Functions/APIs):** `ProtocolEvent`, `namespace`, `method`, `params`
* **Code Snippet:**
```ts
const stream = await graph.streamEvents(
  { messages: [{ role: "user", content: "What is 42 * 17?" }] },
  { version: "v3" }
);

for await (const event of stream) {
  const namespace = event.params.namespace;
  console.log(namespace, event.method, event.params.data);
}
```

## Channels and event lifecycle
* **Key Points:**
  - Raw events flow on channels. The channel name appears as the event's `method`; each channel emits a specific event shape.
  - The typed projections (`stream.messages`, `stream.values`, etc.) are built from these channels. The channel name appears as the `method` field on raw events when you iterate the run object directly.
* **Technical Entities (Classes/Functions/APIs):** `values`, `updates`, `messages`, `tools`, `lifecycle`, `checkpoints`, `input`, `tasks`, `custom`, `custom:<name>`

### Messages
* **Key Points:**
  - The `messages` channel models output as content blocks. The data's `event` field is one of: `message-start`, `content-block-start`, `content-block-delta`, `content-block-finish`, `message-finish`
  - Content blocks have explicit boundaries: a block starts, emits zero or more deltas, and finishes before the next block in the same message starts. This makes token streaming, reasoning blocks, tool-call blocks, and multimodal content explicit without requiring provider-specific formats.
  - `message-finish` may include token usage; unrecoverable model-call failures arrive as message error events.
* **Technical Entities (Classes/Functions/APIs):** `messages`, `content-block-delta`, `text-delta`, `reasoning-delta`

### Tools
* **Key Points:**
  - The `tools` channel exposes tool execution. The data's `event` field is one of: `tool-started`, `tool-output-delta`, `tool-finished`, `tool-error`
  - Tool events are correlated by tool call ID, so a tool execution can be joined back to its originating tool-call content block on the `messages` channel.
* **Technical Entities (Classes/Functions/APIs):** `tools`, `tool-started`, `tool-output-delta`, `tool-finished`, `tool-error`

### Lifecycle
* **Key Points:**
  - The `lifecycle` channel tracks root run, subgraph, and subagent status. The data's `event` field is one of: `started`, `running`, `completed`, `failed`, `interrupted`
  - Beyond `event`, lifecycle data may include an optional `graph_name`, `error`, and `cause` describing why a child scope started (parent tool call, fan-out send, edge transition).
* **Technical Entities (Classes/Functions/APIs):** `lifecycle`, `started`, `running`, `completed`, `failed`, `interrupted`, `graph_name`, `cause`

## Build your own projection
* **Key Points:**
  - Stream transformers are the projection layer in event streaming. They observe protocol events, keep their own state, and expose derived views of a run — things like tool activity, token totals, progress events, artifacts, or messages for another protocol.
  - `StreamChannel` is the projection primitive transformers use to publish those views.
  - Built-in projections (`stream.messages`, `stream.values`, `stream.subgraphs`, `stream.output`) and product-specific projections (LangChain's `stream.tool_calls`, Deep Agents' `stream.subagents`) are themselves transformers using this same contract.
  - User transformers stack on top via compile-time or call-time registration, and their projections appear under `stream.extensions`.
* **Technical Entities (Classes/Functions/APIs):** `StreamChannel`, `stream.extensions`, `process(event)`, `finalize()`, `fail()`

### How transformers work
* **Key Points:**
  - Event streaming starts with streaming output from the LangGraph Pregel engine. The runtime normalizes those chunks into protocol events, then a stream handler routes each event through a stack of stream transformers.
  - The stream handler is the central dispatcher for one stream. For every protocol event, it: Calls each registered transformer's `process(event)` hook in order, Wires named `StreamChannel` pushes back onto the protocol event stream, Stores the event in the run stream unless a transformer suppresses it, Calls `finalize()` or `fail()` on every transformer when the run ends.
  - Transformers are observational. They do not call back into the graph runtime. Instead, they consume events and push derived values into `StreamChannel`, promises, or other projection objects.

### Transformer shape
* **Key Points:**
  - A transformer implements the `StreamTransformer` interface:
  - `init()` creates the projection object. User transformer projections appear under `stream.extensions`.
  - `process()` observes each protocol event. Return `false` only when you intentionally want to suppress the original event.
  - `finalize()` closes or resolves non-channel projections after a successful stream.
  - `fail()` propagates errors to non-channel projections.
* **Technical Entities (Classes/Functions/APIs):** `StreamTransformer`, `init()`, `process()`, `finalize()`, `fail()`

### Declaring required stream modes
* **Key Points:**
  - `required_stream_modes` controls which Pregel stream modes the underlying graph emits during the stream. The runtime takes the union of every registered transformer's `required_stream_modes` and passes that union as the `stream_mode` argument to the graph's `.stream()` call.
  - Modes that no transformer requests are never emitted — declaring `("custom",)` is what causes `custom` events to flow through the run at all.
  - `process()` receives every event the graph emits and is responsible for filtering by `event["method"]`. The declaration turns on upstream emission; it does not narrow what `process()` sees.
  - Valid values are the Pregel stream modes: `"messages"`, `"tools"`, `"custom"`, `"values"`, `"updates"`, `"checkpoints"`, `"tasks"`, `"debug"`. Each transformer must declare every mode it acts on — an omitted mode is not emitted by the graph and never reaches `process()`.
* **Technical Entities (Classes/Functions/APIs):** `required_stream_modes`, `stream_mode`

### StreamChannel
* **Key Points:**
  - `StreamChannel` is the projection primitive a transformer uses for streaming values. It always exposes an iterable stream on `stream.extensions.<name>`. The constructor argument decides whether each `push()` also flows into the run's main event stream as a `custom:<name>` event—that is, whether the projection's values show up when iterating raw protocol events.
  - Named channel payloads must be serializable, because each pushed value also becomes a `custom:<name>` protocol event in the main stream.
  - Keep promises, async iterables, class instances, and other in-process handles in unnamed channels.
  - The stream handler owns channel lifecycle. Once `init()` returns a channel, the handler closes or fails it for you when the run ends. Transformers only push values.
* **Technical Entities (Classes/Functions/APIs):** `StreamChannel`, `push()`, `stream.extensions`

### Example: named channel
* **Key Points:**
  - Pass a string name to `StreamChannel` to expose a streaming projection through `stream.extensions` *and* forward each pushed value into the run's main event stream as a `custom:<name>` protocol event:
* **Technical Entities (Classes/Functions/APIs):** `StreamChannel`, `push()`, `stream.extensions`, `custom:<name>`
* **Code Snippet:**
```ts
import { StreamChannel } from "@langchain/langgraph";

const toolActivityTransformer = () => {
  const activity = new StreamChannel<{
    name: string;
    status: "started" | "finished" | "error";
  }>("toolActivity");

  return {
    init: () => ({ toolActivity: activity }),
    process(event) {
      if (event.method === "tools") {
        const data = event.params.data as { tool_name?: string; event?: string };
        if (data.tool_name && data.event) {
          activity.push({
            name: data.tool_name,
            status: data.event === "tool-error" ? "error" : "started",
          });
        }
      }
      return true;
    },
  };
};
```

### Example: unnamed channel
* **Key Points:**
  - Without a name, the channel is a side-channel projection only — accessible on `stream.extensions` but not visible to consumers iterating raw events. This is the right choice for projections that hold in-process handles (promises, async iterables, class instances) that can't be serialized onto the main event stream.
* **Technical Entities (Classes/Functions/APIs):** `StreamChannel`, `stream.extensions`, `get_stream_writer`
* **Code Snippet:**
```ts
import { StreamChannel } from "@langchain/langgraph";

const customTransformer = () => {
  const custom = new StreamChannel<unknown>();

  return {
    init: () => ({ custom }),
    process(event) {
      if (event.method === "custom") {
        custom.push(event.params.data);
      }
      return true;
    },
  };
};
```

### Example: final-value projection
* **Key Points:**
  - Use unnamed streams, promises, or other in-process objects when the projection should not flow into the main event stream:
* **Technical Entities (Classes/Functions/APIs):** `Promise`, `resolve`, `finalize()`
* **Code Snippet:**
```ts
const statsTransformer = () => {
  let totalTokens = 0;
  let resolveTotal!: (value: number) => void;
  const totalTokensPromise = new Promise<number>((resolve) => {
    resolveTotal = resolve;
  });

  return {
    init: () => ({ totalTokens: totalTokensPromise }),
    process(event) {
      if (event.method === "messages") {
        const data = event.params.data as { usage?: { output_tokens?: number } };
        totalTokens += data.usage?.output_tokens ?? 0;
      }
      return true;
    },
    finalize: () => resolveTotal(totalTokens),
  };
};
```

### Register at call time or compile time
* **Key Points:**
  - Pass transformers at call time for local experimentation:
  - Compile transformers into the graph when every run of that graph should produce the projection:
* **Technical Entities (Classes/Functions/APIs):** `streamEvents`, `transformers`, `compile`
* **Code Snippet:**
```ts
const stream = await graph.streamEvents(input, {
  version: "v3",
  transformers: [statsTransformer, toolActivityTransformer],
});
```
```ts
const graph = builder.compile({
  transformers: [statsTransformer, toolActivityTransformer],
});
```

### Built-in: `ToolCallTransformer`
* **Technical Entities (Classes/Functions/APIs):** `ToolCallTransformer`

## Related
* **Key Points:**
  - LangGraph defines the streaming primitives. For using streaming with LangChain or Deep Agents, review the relevant product docs:
  - LangChain agent streaming covers ReAct-style agent messages, tool calls, and middleware updates.
  - Deep Agents streaming covers subagents, nested messages, and subagent tool calls.
  - LangChain frontend patterns and LangGraph frontend patterns show UI use cases built on top of streamed state.
  - LangSmith Streaming API covers streaming against a graph deployed behind an Agent Server.
  - The wire-level event and command formats are defined in the Agent Protocol repository and consumable as `langchain-protocol` on PyPI and `@langchain/protocol` on npm.
* **Technical Entities (Classes/Functions/APIs):** `langchain-protocol`, `@langchain/protocol`