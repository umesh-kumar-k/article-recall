---
aliases:
  - Langgraph Streaming
Source 1: https://docs.langchain.com/oss/javascript/langgraph/streaming
---
# Streaming

## This page covers LangGraph's stream-mode API.

* **Key Points:**
  - It exposes graph execution through stream modes such as updates, values, messages, custom, checkpoints, tasks, and debug.
  - Use it when you need direct access to graph-runtime events or specific stream-mode output.
* **Technical Entities (Classes/Functions/APIs):** `LangGraph`, `stream-mode API`, `updates`, `values`, `messages`, `custom`, `checkpoints`, `tasks`, `debug`

## Get started
### Basic usage
* **Key Points:**
  - LangGraph graphs expose the stream method to yield streamed outputs as iterators.
  - Debug streaming events, inspect token-by-token LLM output, and monitor latency with LangSmith.
  - Follow the tracing quickstart to get set up.
* **Technical Entities (Classes/Functions/APIs):** `stream method`, `LangSmith`
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
  - Pass one or more of the following stream modes as a list to the stream method:
* **Technical Entities (Classes/Functions/APIs):** `stream method`
* **Code Snippet:**
```
Mode	Description
values	Full state after each step.
updates	State updates after each step. Multiple updates in the same step are streamed separately.
messages	2-tuples of (LLM token, metadata) from LLM calls.
custom	Custom data emitted from nodes via the writer config parameter.
tools	Tool-call lifecycle events (on_tool_start, on_tool_event, on_tool_end, on_tool_error).
debug	All available info throughout graph execution.
```

## Graph state
* **Key Points:**
  - Use the stream modes updates and values to stream the state of the graph as it executes.
  - updates streams the updates to the state after each step of the graph.
  - values streams the full value of the state after each step of the graph.
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `StateSchema`, `START`, `END`, `stream method`
* **Code Snippet:**
```typescript
import { StateGraph, StateSchema, START, END } from "@langchain/langgraph";
import { z } from "zod/v4";

const State = new StateSchema({
  topic: z.string(),
  joke: z.string(),
});

const graph = new StateGraph(State)
  .addNode("refineTopic", (state) => {
    return { topic: state.topic + " and cats" };
  })
  .addNode("generateJoke", (state) => {
    return { joke: `This is a joke about ${state.topic}` };
  })
  .addEdge(START, "refineTopic")
  .addEdge("refineTopic", "generateJoke")
  .addEdge("generateJoke", END)
  .compile();
```
```typescript
for await (const chunk of await graph.stream(
  { topic: "ice cream" },
  { streamMode: "updates" }
)) {
  for (const [nodeName, state] of Object.entries(chunk)) {
    console.log(`Node ${nodeName} updated:`, state);
  }
}
```

## LLM tokens
* **Key Points:**
  - Use the messages streaming mode to stream Large Language Model (LLM) outputs token by token from any part of your graph, including nodes, tools, subgraphs, or tasks.
  - The streamed output from messages mode is a tuple [message_chunk, metadata] where:
    - message_chunk: the token or message segment from the LLM.
    - metadata: a dictionary containing details about the graph node and LLM invocation.
  - If your LLM is not available as a LangChain integration, you can stream its outputs using custom mode instead.
* **Technical Entities (Classes/Functions/APIs):** `messages streaming mode`, `ChatOpenAI`, `StateGraph`, `StateSchema`, `GraphNode`, `START`
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

### Filter by LLM invocation
* **Key Points:**
  - You can associate tags with LLM invocations to filter the streamed tokens by LLM invocation.
* **Technical Entities (Classes/Functions/APIs):** `tags`, `ChatOpenAI`
* **Code Snippet:**
```typescript
import { ChatOpenAI } from "@langchain/openai";

// model1 is tagged with "joke"
const model1 = new ChatOpenAI({
  model: "gpt-5.4-mini",
  tags: ['joke']
});
// model2 is tagged with "poem"
const model2 = new ChatOpenAI({
  model: "gpt-5.4-mini",
  tags: ['poem']
});

const graph = // ... define a graph that uses these LLMs

// The streamMode is set to "messages" to stream LLM tokens
// The metadata contains information about the LLM invocation, including the tags
for await (const [msg, metadata] of await graph.stream(
  { topic: "cats" },
  { streamMode: "messages" }
)) {
  // Filter the streamed tokens by the tags field in the metadata to only include
  // the tokens from the LLM invocation with the "joke" tag
  if (metadata.tags?.includes("joke")) {
    console.log(msg.content + "|");
  }
}
```

### Omit messages from the stream
* **Key Points:**
  - Use the nostream tag to exclude LLM output from the stream entirely.
  - Invocations tagged with nostream still run and produce output; their tokens are simply not emitted in messages mode.
  - This is useful when:
    - You need LLM output for internal processing (for example structured output) but do not want to stream it to the client
    - You stream the same content through a different channel (for example custom UI messages) and want to avoid duplicate output in the messages stream
* **Technical Entities (Classes/Functions/APIs):** `nostream tag`, `ChatAnthropic`, `.withConfig()`, `tags`, `streamEvents()`
* **Code Snippet:**
```typescript
import { ChatAnthropic } from "@langchain/anthropic";
import { StateGraph, StateSchema, START } from "@langchain/langgraph";
import * as z from "zod";

const streamModel = new ChatAnthropic({ model: "claude-haiku-4-5-20251001" });
const internalModel = new ChatAnthropic({
  model: "claude-haiku-4-5-20251001",
}).withConfig({
  tags: ["nostream"],
});

const State = new StateSchema({
  topic: z.string(),
  answer: z.string().optional(),
  notes: z.string().optional(),
});

const contentToText = (content: unknown): string => {
  if (typeof content === "string") {
    return content;
  }
  if (Array.isArray(content)) {
    return content
      .map((block) => {
        if (
          typeof block === "object" &&
          block !== null &&
          "text" in block &&
          typeof (block as { text?: unknown }).text === "string"
        ) {
          return (block as { text: string }).text;
        }
        return "";
      })
      .filter(Boolean)
      .join("\n");
  }
  return "";
};

const writeAnswer = async (state: typeof State.State) => {
  const r = await streamModel.invoke([
    { role: "user", content: `Reply briefly about ${state.topic}` },
  ]);
  return { answer: contentToText(r.content) };
};

const internalNotes = async (state: typeof State.State) => {
  // Tokens from this model are omitted from streamMode: "messages" because of nostream
  const r = await internalModel.invoke([
    { role: "user", content: `Private notes on ${state.topic}` },
  ]);
  return { notes: contentToText(r.content) };
};

const graph = new StateGraph(State)
  .addNode("writeAnswer", writeAnswer)
  .addNode("internal_notes", internalNotes)
  .addEdge(START, "writeAnswer")
  .addEdge("writeAnswer", "internal_notes")
  .compile();

const stream = await graph.streamEvents(
  { topic: "AI", answer: "", notes: "" },
  { version: "v3" },
);
```

### Filter by node
* **Key Points:**
  - To stream tokens only from specific nodes, use stream_mode="messages" and filter the outputs by the langgraph_node field in the streamed metadata.
* **Technical Entities (Classes/Functions/APIs):** `stream_mode="messages"`, `langgraph_node field`
* **Code Snippet:**
```typescript
// The "messages" stream mode returns a tuple of [messageChunk, metadata]
// where messageChunk is the token streamed by the LLM and metadata is a dictionary
// with information about the graph node where the LLM was called and other information
for await (const [msg, metadata] of await graph.stream(
  inputs,
  { streamMode: "messages" }
)) {
  // Filter the streamed tokens by the langgraph_node field in the metadata
  // to only include the tokens from the specified node
  if (msg.content && metadata.langgraph_node === "some_node_name") {
    // ...
  }
}
```

## Custom data
* **Key Points:**
  - To send custom user-defined data from inside a LangGraph node or tool, follow these steps:
    - Use the writer parameter from the LangGraphRunnableConfig to emit custom data.
    - Set streamMode: "custom" when calling .stream() to get the custom data in the stream.
  - You can combine multiple modes (e.g., ["updates", "custom"]), but at least one must be "custom".
* **Technical Entities (Classes/Functions/APIs):** `writer parameter`, `LangGraphRunnableConfig`, `streamMode: "custom"`, `StateGraph`, `StateSchema`, `GraphNode`, `START`
* **Code Snippet:**
```typescript
import { StateGraph, StateSchema, GraphNode, START, LangGraphRunnableConfig } from "@langchain/langgraph";
import * as z from "zod";

const State = new StateSchema({
  query: z.string(),
  answer: z.string(),
});

const node: GraphNode<typeof State> = async (state, config) => {
  // Use the writer to emit a custom key-value pair (e.g., progress update)
  config.writer({ custom_key: "Generating custom data inside node" });
  return { answer: "some data" };
};

const graph = new StateGraph(State)
  .addNode("node", node)
  .addEdge(START, "node")
  .compile();

const inputs = { query: "example" };

// Set streamMode: "custom" to receive the custom data in the stream
for await (const chunk of await graph.stream(inputs, { streamMode: "custom" })) {
  console.log(chunk);
}
```

## Tool progress
* **Key Points:**
  - Use the tools stream mode to receive real-time lifecycle events for tool executions.
  - This is useful for showing progress indicators, partial results, and error states in your UI while tools are running.
* **Technical Entities (Classes/Functions/APIs):** `tools stream mode`
* **Code Snippet:**
```
Event	When	Payload
on_tool_start	Tool invocation begins	name, input, toolCallId
on_tool_event	Tool yields intermediate data	name, data, toolCallId
on_tool_end	Tool returns its final result	name, output, toolCallId
on_tool_error	Tool throws an error	name, error, toolCallId
```

### Define tools that stream progress
* **Key Points:**
  - To emit on_tool_event events, define your tool function as an async generator (async function*).
  - Each yield sends intermediate data to the stream, and the return value is used as the tool's final result.
  - Existing tools that return a Promise are fully compatible.
  - They emit on_tool_start and on_tool_end events but no on_tool_event events.
* **Technical Entities (Classes/Functions/APIs):** `async generator`, `async function*`, `tool`, `on_tool_event`, `on_tool_start`, `on_tool_end`
* **Code Snippet:**
```typescript
import { tool } from "@langchain/core/tools";
import { z } from "zod/v4";

const searchFlights = tool(
  async function* (input) {
    const airlines = ["United", "Delta", "American", "JetBlue"];
    const completed: string[] = [];

    for (let i = 0; i < airlines.length; i++) {
      await new Promise((r) => setTimeout(r, 500));
      completed.push(airlines[i]);

      // Each yield emits an on_tool_event to the stream
      yield {
        message: `Searching ${airlines[i]}...`,
        progress: (i + 1) / airlines.length,
        completed,
      };
    }

    // The return value becomes the tool result (ToolMessage.content)
    return JSON.stringify({
      flights: [
        { airline: "United", price: 450, duration: "5h 30m" },
        { airline: "Delta", price: 520, duration: "5h 15m" },
      ],
    });
  },
  {
    name: "search_flights",
    description: "Search for available flights to a destination.",
    schema: z.object({
      destination: z.string(),
      date: z.string(),
    }),
  }
);
```

### Consume tool events server-side
* **Key Points:**
  - Pass streamMode: ["tools"] (or combine with other modes) to graph.stream()
* **Technical Entities (Classes/Functions/APIs):** `streamMode: ["tools"]`, `graph.stream()`, `on_tool_start`, `on_tool_event`, `on_tool_end`, `on_tool_error`
* **Code Snippet:**
```typescript
for await (const [mode, chunk] of await graph.stream(
  { messages: [{ role: "user", content: "Find flights to Tokyo" }] },
  { streamMode: ["updates", "tools"] }
)) {
  if (mode === "tools") {
    switch (chunk.event) {
      case "on_tool_start":
        console.log(`Tool started: ${chunk.name}`, chunk.input);
        break;
      case "on_tool_event":
        console.log(`Tool progress: ${chunk.name}`, chunk.data);
        break;
      case "on_tool_end":
        console.log(`Tool finished: ${chunk.name}`, chunk.output);
        break;
      case "on_tool_error":
        console.error(`Tool failed: ${chunk.name}`, chunk.error);
        break;
    }
  }
}
```

### Use tool progress in React with useStream
* **Key Points:**
  - The useStream hook from @langchain/langgraph-sdk/react exposes a toolProgress array when you include "tools" in your stream modes.
  - Each entry is a ToolProgress object that tracks the current state of a running tool:
* **Technical Entities (Classes/Functions/APIs):** `useStream`, `@langchain/langgraph-sdk/react`, `toolProgress array`, `ToolProgress`
* **Code Snippet:**
```
Field	Description
name	The tool name
state	Current lifecycle state: "starting", "running", "completed", or "error"
toolCallId	The tool call ID from the LLM
input	The tool's input arguments
data	The most recent yielded data from on_tool_event
result	The final result, set on on_tool_end
error	The error, set on on_tool_error
```
```typescript
import { useStream } from "@langchain/langgraph-sdk/react";

function Chat() {
  const stream = useStream({
    assistantId: "my-agent",
    streamMode: ["values", "tools"],
  });

  // Filter for actively running tools
  const activeTools = stream.toolProgress.filter(
    (t) => t.state === "starting" || t.state === "running"
  );

  return (
    <div>
      {stream.messages.map((msg) => (
        <MessageBubble key={msg.id} message={msg} />
      ))}

      {/* Show progress cards for running tools */}
      {activeTools.map((tool) => (
        <ToolProgressCard
          key={tool.toolCallId ?? tool.name}
          name={tool.name}
          state={tool.state}
          data={tool.data}
        />
      ))}
    </div>
  );
}
```

### tools vs custom stream mode
* **Key Points:**
  - Both stream modes can surface tool progress, but they serve different purposes:
    - tools—automatically emits structured lifecycle events (on_tool_start, on_tool_event, on_tool_end, on_tool_error) with no code changes needed in your tools beyond using async function*.
    - The useStream hook provides the reactive toolProgress array out of the box.
    - custom—gives you full control over what data is emitted and when using config.writer().
    - Use this when you need freeform data that doesn't map to the tool lifecycle, or when you want to stream from nodes (not just tools).
* **Technical Entities (Classes/Functions/APIs):** `tools stream mode`, `custom stream mode`, `useStream`, `config.writer()`

## Subgraph outputs
* **Key Points:**
  - To include outputs from subgraphs in the streamed outputs, you can set subgraphs: true in the .stream() method of the parent graph.
  - This will stream outputs from both the parent graph and any subgraphs.
  - The outputs will be streamed as tuples [namespace, data], where namespace is a tuple with the path to the node where a subgraph is invoked, e.g. ["parent_node:<task_id>", "child_node:<task_id>"].
* **Technical Entities (Classes/Functions/APIs):** `subgraphs: true`, `.stream() method`
* **Code Snippet:**
```typescript
for await (const chunk of await graph.stream(
  { foo: "foo" },
  {
    // Set subgraphs: true to stream outputs from subgraphs
    subgraphs: true,
    streamMode: "updates",
  }
)) {
  console.log(chunk);
}
```

## Debug
* **Key Points:**
  - Use the debug streaming mode to stream as much information as possible throughout the execution of the graph.
  - The streamed outputs include the name of the node as well as the full state.
* **Technical Entities (Classes/Functions/APIs):** `debug streaming mode`
* **Code Snippet:**
```typescript
for await (const chunk of await graph.stream(
  { topic: "ice cream" },
  { streamMode: "debug" }
)) {
  console.log(chunk);
}
```

## Multiple modes at once
* **Key Points:**
  - You can pass an array as the streamMode parameter to stream multiple modes at once.
  - The streamed outputs will be tuples of [mode, chunk] where mode is the name of the stream mode and chunk is the data streamed by that mode.
* **Technical Entities (Classes/Functions/APIs):** `streamMode parameter`
* **Code Snippet:**
```typescript
for await (const [mode, chunk] of await graph.stream(inputs, {
  streamMode: ["updates", "custom"],
})) {
  console.log(chunk);
}
```

## Advanced
### Use with any LLM
* **Key Points:**
  - You can use streamMode: "custom" to stream data from any LLM API—even if that API does not implement the LangChain chat model interface.
  - This lets you integrate raw LLM clients or external services that provide their own streaming interfaces, making LangGraph highly flexible for custom setups.
* **Technical Entities (Classes/Functions/APIs):** `streamMode: "custom"`, `StateGraph`, `GraphNode`, `StateSchema`, `config.writer()`
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
  - Set streaming: false when initializing the model.
  - Not all chat model integrations support the streaming parameter.
  - If your model doesn't support it, use disableStreaming: true instead.
  - This parameter is available on all chat models via the base class.
* **Technical Entities (Classes/Functions/APIs):** `streaming: false`, `ChatOpenAI`, `disableStreaming: true`
* **Code Snippet:**
```typescript
import { ChatOpenAI } from "@langchain/openai";

const model = new ChatOpenAI({
  model: "o1-preview",
  // Set streaming: false to disable streaming for the chat model
  streaming: false,
});
```