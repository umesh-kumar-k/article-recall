---
aliases:
  - Langchain Streaming
Source 1: https://docs.langchain.com/oss/javascript/langchain/streaming
---
# Streaming

* **Key Points:**
  - Streaming is crucial for enhancing the responsiveness of applications built on LLMs.
  - By displaying output progressively, even before a complete response is ready, streaming significantly improves user experience (UX), particularly when dealing with the latency of LLMs.
* **Technical Entities (Classes/Functions/APIs):** `LangChain`

## Overview
* **Key Points:**
  - LangChain's streaming system lets you surface live feedback from agent runs to your application.
  - What's possible with LangChain streaming:
    - Stream agent progress—get state updates after each agent step.
    - Stream LLM tokens—stream language model tokens as they're generated.
    - Stream thinking / reasoning tokens—surface model reasoning as it's generated.
    - Stream custom updates—emit user-defined signals (e.g., "Fetched 10/100 records").
    - Stream multiple modes—choose from updates (agent progress), messages (LLM tokens + metadata), or custom (arbitrary user data).

## Supported stream modes
* **Key Points:**
  - Pass one or more of the following stream modes as a list to the stream method:
* **Technical Entities (Classes/Functions/APIs):** `stream method`, `streamMode`, `updates`, `messages`, `custom`
* **Code Snippet:**
```
Mode	Description
updates	Streams state updates after each agent step. If multiple updates are made in the same step (e.g., multiple nodes are run), those updates are streamed separately.
messages	Streams tuples of (token, metadata) from any graph nodes where an LLM is invoked.
custom	Streams custom data from inside your graph nodes using the stream writer.
```

## Agent progress
* **Key Points:**
  - To stream agent progress, use the stream method with streamMode: "updates".
  - This emits an event after every agent step.
  - For example, if you have an agent that calls a tool once, you should see the following updates:
    - LLM node: AIMessage with tool call requests
    - Tool node: ToolMessage with execution result
    - LLM node: Final AI response
  - Pass a thread_id via configurable so the conversation is checkpointed and follow-up turns can resume the same history.
  - thread_id is independent of streamMode; you can also pass context alongside it for per-run data your tools read from runtime.context.
  - Persisting conversation history with thread_id requires the agent to be configured with a checkpointer.
  - On LangSmith deployments a checkpointer is provisioned automatically.
  - Locally, pass one explicitly, for example createAgent({ ..., checkpointer: new MemorySaver() }).
* **Technical Entities (Classes/Functions/APIs):** `stream method`, `streamMode: "updates"`, `thread_id`, `configurable`, `runtime.context`, `checkpointer`, `LangSmith`, `createAgent()`, `MemorySaver`, `streamEvents()`
* **Code Snippet:**
```typescript
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import z from "zod";

const getWeather = tool(
  async ({ city }) => {
    return `The weather in ${city} is always sunny!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

const agent = createAgent({
  model: "ollama:devstral-2",
  tools: [getWeather],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "what is the weather in sf" }] },
  { ...config, version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
      console.log(`Tool result: ${await call.output}`);
    }
  })(),
]);

const finalState = await stream.output;
// Tool call: get_weather({"city":"San Francisco"})
// Tool result: [object ToolMessage]
// According to the data I have, the weather in San Francisco is always sunny! Would you like current conditions or a short forecast for today or the next few days?
```

## LLM tokens
* **Key Points:**
  - To stream tokens as they are produced by the LLM, use streamMode: "messages"
* **Technical Entities (Classes/Functions/APIs):** `streamMode: "messages"`, `createAgent()`, `tool`, `stream method`
* **Code Snippet:**
```typescript
import z from "zod";
import { createAgent, tool } from "langchain";

const getWeather = tool(
    async ({ city }) => {
        return `The weather in ${city} is always sunny!`;
    },
    {
        name: "get_weather",
        description: "Get weather for a given city.",
        schema: z.object({
        city: z.string(),
        }),
    }
);

const agent = createAgent({
    model: "gpt-5.4-mini",
    tools: [getWeather],
});

for await (const [token, metadata] of await agent.stream(
    { messages: [{ role: "user", content: "what is the weather in sf" }] },
    { streamMode: "messages" }
)) {
    console.log(`node: ${metadata.langgraph_node}`);
    console.log(`content: ${JSON.stringify(token.contentBlocks, null, 2)}`);
}
```

## Custom updates
* **Key Points:**
  - To stream updates from tools as they are executed, you can use the writer parameter from the configuration.
  - If you add the writer parameter to your tool, you won't be able to invoke the tool outside of a LangGraph execution context without providing a writer function.
* **Technical Entities (Classes/Functions/APIs):** `writer parameter`, `LangGraphRunnableConfig`, `streamMode: "custom"`, `createAgent()`, `tool`
* **Code Snippet:**
```typescript
import z from "zod";
import { tool, createAgent } from "langchain";
import { LangGraphRunnableConfig } from "@langchain/langgraph";

const getWeather = tool(
    async (input, config: LangGraphRunnableConfig) => {
        // Stream any arbitrary data
        config.writer?.(`Looking up data for city: ${input.city}`);
        // ... fetch city data
        config.writer?.(`Acquired data for city: ${input.city}`);
        return `It's always sunny in ${input.city}!`;
    },
    {
        name: "get_weather",
        description: "Get weather for a given city.",
        schema: z.object({
        city: z.string().describe("The city to get weather for."),
        }),
    }
);

const agent = createAgent({
    model: "gpt-5.4-mini",
    tools: [getWeather],
});

for await (const chunk of await agent.stream(
    { messages: [{ role: "user", content: "what is the weather in sf" }] },
    { streamMode: "custom" }
)) {
    console.log(chunk);
}
```
* **Code Snippet:**
```
Looking up data for city: San Francisco
Acquired data for city: San Francisco
```

## Stream multiple modes
* **Key Points:**
  - You can specify multiple streaming modes by passing streamMode as an array: streamMode: ["updates", "messages", "custom"].
  - The streamed outputs will be tuples of [mode, chunk] where mode is the name of the stream mode and chunk is the data streamed by that mode.
* **Technical Entities (Classes/Functions/APIs):** `streamMode`, `createAgent()`, `tool`, `LangGraphRunnableConfig`, `stream method`
* **Code Snippet:**
```typescript
import z from "zod";
import { tool, createAgent } from "langchain";
import { LangGraphRunnableConfig } from "@langchain/langgraph";

const getWeather = tool(
    async (input, config: LangGraphRunnableConfig) => {
        // Stream any arbitrary data
        config.writer?.(`Looking up data for city: ${input.city}`);
        // ... fetch city data
        config.writer?.(`Acquired data for city: ${input.city}`);
        return `It's always sunny in ${input.city}!`;
    },
    {
        name: "get_weather",
        description: "Get weather for a given city.",
        schema: z.object({
        city: z.string().describe("The city to get weather for."),
        }),
    }
);

const agent = createAgent({
    model: "gpt-5.4-mini",
    tools: [getWeather],
});

for await (const [streamMode, chunk] of await agent.stream(
    { messages: [{ role: "user", content: "what is the weather in sf" }] },
    { streamMode: ["updates", "messages", "custom"] }
)) {
    console.log(`${streamMode}: ${JSON.stringify(chunk, null, 2)}`);
}
```

## Common patterns
* **Key Points:**
  - Below are examples showing common use cases for streaming.

### Streaming thinking / reasoning tokens
* **Key Points:**
  - Some models perform internal reasoning before producing a final answer.
  - You can stream these thinking / reasoning tokens as they're generated by filtering standard content blocks for the type "reasoning".
  - Reasoning output must be enabled on the model.
  - See the reasoning section and your provider's integration page for configuration details.
  - To quickly check a model's reasoning support, see models.dev.
  - To stream thinking tokens from an agent, use streamMode: "messages" and filter for reasoning content blocks.
  - Use a model instance (e.g. ChatAnthropic) with extended thinking enabled when the model supports it.
  - This works the same way regardless of the model provider—LangChain normalizes provider-specific formats (Anthropic thinking blocks, OpenAI reasoning summaries, etc.) into a standard "reasoning" content block type via the content_blocks property.
  - To stream reasoning tokens directly from a chat model (without an agent), see streaming with chat models.
* **Technical Entities (Classes/Functions/APIs):** `streamMode: "messages"`, `reasoning content blocks`, `ChatAnthropic`, `content_blocks property`, `createAgent()`, `tool`, `streamEvents()`
* **Code Snippet:**
```typescript
import z from "zod";
import { createAgent, tool } from "langchain";
import { ChatAnthropic } from "@langchain/anthropic";

const getWeather = tool(
  async ({ city }) => {
    return `It's always sunny in ${city}!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({ city: z.string() }),
  },
);

const agent = createAgent({
  model: new ChatAnthropic({
    model: "claude-sonnet-4-6",
    thinking: { type: "enabled", budget_tokens: 5000 },
  }),
  tools: [getWeather],
});

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "What is the weather in SF?" }] },
  { version: "v3" },
);
for await (const message of stream.messages) {
  for await (const token of message.reasoning) {
    process.stdout.write(`[thinking] ${token}`);
  }
  for await (const token of message.text) {
    process.stdout.write(token);
  }
}
```
* **Code Snippet:**
```
[thinking] The user is asking about the weather in San Francisco. I have a tool
[thinking]  available to get this information. Let me call the get_weather tool
[thinking]  with "San Francisco" as the city parameter.
The weather in San Francisco is: It's always sunny in San Francisco!
```

### Disable streaming
* **Key Points:**
  - In some applications you might need to disable streaming of individual tokens for a given model.
  - This is useful when:
    - Working with multi-agent systems to control which agents stream their output
    - Mixing models that support streaming with those that do not
    - Deploying to LangSmith and wanting to prevent certain model outputs from being streamed to the client
  - Set streaming: false when initializing the model.
  - When deploying to LangSmith, set streaming=False on any models whose output you don't want streamed to the client.
  - This is configured in your graph code before deployment.
  - Not all chat model integrations support the streaming parameter.
  - If your model doesn't support it, use disableStreaming: true instead.
  - This parameter is available on all chat models via the base class.
  - See the LangGraph streaming guide for more details.
* **Technical Entities (Classes/Functions/APIs):** `streaming: false`, `ChatOpenAI`, `LangSmith`, `disableStreaming: true`, `LangGraph streaming guide`
* **Code Snippet:**
```typescript
import { ChatOpenAI } from "@langchain/openai";

const model = new ChatOpenAI({
  model: "gpt-5.5",
  streaming: false,
});
```

## Related
* **Key Points:**
  - Frontend streaming—Build React UIs with useStream for real-time agent interactions
  - Streaming with chat models—Stream tokens directly from a chat model without using an agent or graph
  - Reasoning with chat models—Configure and access reasoning output from chat models
  - Standard content blocks—Understand the normalized content block format used for reasoning, text, and other content types
  - Streaming with human-in-the-loop—Stream agent progress while handling interrupts for human review
  - LangGraph streaming—Advanced streaming options including values, debug modes, and subgraph streaming
* **Technical Entities (Classes/Functions/APIs):** `useStream`, `React`, `LangGraph streaming guide`