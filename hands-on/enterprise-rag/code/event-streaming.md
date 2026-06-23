---
aliases:
  - Langchain Event Streaming
Source 1: https://docs.langchain.com/oss/javascript/langchain/event-streaming
---
# Event streaming

* **Key Points:**
  - LangChain agents are built on LangGraph, so they support the same streaming stack with agent-focused projections for messages, tool calls, state, and custom updates.
  - For most application and frontend use cases, use Event Streaming through stream_events(..., version="v3").
  - Event Streaming returns a run object with typed projections, so each projection can be consumed independently instead of parsing stream-mode tuples.
* **Technical Entities (Classes/Functions/APIs):** `LangChain`, `LangGraph`, `stream_events()`, `version="v3"`, `createAgent()`, `tool`, `streamEvents()`
* **Code Snippet:**
```typescript
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  async ({ city }) => `It's always sunny in ${city}!`,
  {
    name: "get_weather",
    description: "Get weather for a city.",
    schema: z.object({ city: z.string() }),
  }
);

const agent = createAgent({
  model: "gpt-5-nano",
  tools: [getWeather],
});

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "What is the weather in SF?" }] },
  { version: "v3" }
);

for await (const message of stream.messages) {
  for await (const delta of message.text) {
    process.stdout.write(delta);
  }
}

const finalState = await stream.output;
```

## What you can stream
* **Key Points:**
  - stream.messages yields message streams.
  - Each message stream exposes .text, .reasoning, .toolCalls, .output, and .usage.
  - Async projections can be iterated for live deltas or awaited for final values.
* **Technical Entities (Classes/Functions/APIs):** `stream.messages`, `message.text`, `message.reasoning`, `message.toolCalls`, `message.output`, `message.usage`, `stream.values`, `stream.output`, `stream.subgraphs`, `stream.extensions`, `stream.toolCalls`

## Agent messages
* **Key Points:**
  - Use stream.messages when you want model output from each LLM call.
  - message.output gives you the finalized AI message, including provider-specific content blocks.
  - In TypeScript, use message.usage when you only need token counts or other usage metadata; in Python, read usage from message.output.usage_metadata.
* **Technical Entities (Classes/Functions/APIs):** `stream.messages`, `message.output`, `message.usage`, `message.output.usage_metadata`
* **Code Snippet:**
```typescript
const stream = await agent.streamEvents(input, { version: "v3" });

for await (const message of stream.messages) {
  process.stdout.write(`[${message.node}] `);
  for await (const delta of message.text) {
    process.stdout.write(delta);
  }

  const fullMessage = await message.output;
  console.log(fullMessage.content);

  const usage = await message.usage;
  if (usage) {
    console.log(usage);
  }
}
```

## Reasoning content
* **Key Points:**
  - Reasoning content uses the same shape as text content, but it is available only when the selected model emits reasoning blocks.
  - See the reasoning guide and your provider's integration page for model configuration details.
* **Technical Entities (Classes/Functions/APIs):** `message.reasoning`
* **Code Snippet:**
```typescript
const stream = await agent.streamEvents(input, { version: "v3" });

for await (const message of stream.messages) {
  for await (const delta of message.reasoning) {
    process.stdout.write(`[thinking] ${delta}`);
  }

  for await (const delta of message.text) {
    process.stdout.write(delta);
  }
}
```

## Tool calls
* **Key Points:**
  - There are two useful tool-call projections:
    - message.tool_calls streams tool-call argument chunks while the model is producing the tool call.
    - stream.tool_calls streams the lifecycle of tool execution after the tool call starts.
* **Technical Entities (Classes/Functions/APIs):** `message.toolCalls`, `stream.toolCalls`
* **Code Snippet:**
```typescript
const stream = await agent.streamEvents(input, { version: "v3" });

await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const chunk of message.toolCalls) {
        console.log("tool call chunk", chunk);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(call.name, call.input);
      console.log(await call.output, await call.error);
    }
  })(),
]);
```

## Streaming sub-agents
* **Key Points:**
  - When a create_agent call invokes another named create_agent (via a wrapping tool, typically), the inner agent's events flow at a nested namespace.
  - The name= you pass to create_agent identifies that inner agent in the stream, so you can filter and label per agent.
  - Named sub-agents surface as handles on stream.subgraphs, alongside any plain subgraphs.
  - Each handle exposes the inner agent's .messages, .values, .toolCalls, and .output; filter on subagent.name (the name= you passed) to act on a specific agent.
  - Plain StateGraph subgraphs invoked from a tool also surface on stream.subgraphs — set name= on .compile(name=...) to get a label in subagent.graph_name.
  - Named sub-agents share the stream.subgraphs projection with plain subgraphs; the filter you write into your loop is what separates them.
* **Technical Entities (Classes/Functions/APIs):** `create_agent`, `name=`, `stream.subgraphs`, `subagent.name`, `subagent.messages`, `subagent.values`, `subagent.toolCalls`, `subagent.output`, `subagent.graph_name`, `StateGraph`, `.compile(name=...)`
* **Code Snippet:**
```typescript
import { createAgent, tool } from "langchain";
import { z } from "zod";

const getWeather = tool(
  async ({ city }) => `It's always sunny in ${city}!`,
  { name: "get_weather", schema: z.object({ city: z.string() }) }
);

const weatherAgent = createAgent({
  model: "openai:gpt-5.5",
  tools: [getWeather],
  name: "weather_agent",
});

const callWeather = tool(
  async ({ query }) => {
    const result = await weatherAgent.invoke({
      messages: [{ role: "user", content: query }],
    });
    return result.messages.at(-1)?.text ?? "";
  },
  { name: "call_weather", schema: z.object({ query: z.string() }) }
);

const supervisor = createAgent({
  model: "openai:gpt-5.5",
  tools: [callWeather],
  name: "supervisor",
});

const stream = await supervisor.streamEvents(
  { messages: [{ role: "user", content: "What's the weather in Boston?" }] },
  { version: "v3" }
);

for await (const subagent of stream.subgraphs) {
  if (subagent.name !== "weather_agent") continue;
  process.stdout.write(`${subagent.name}: `);
  for await (const message of subagent.messages) {
    for await (const token of message.text) {
      process.stdout.write(token);
    }
  }
  process.stdout.write("\n");
}
```

## State and final output
* **Key Points:**
  - Use stream.values for state snapshots and stream.output for the final agent state.
* **Technical Entities (Classes/Functions/APIs):** `stream.values`, `stream.output`
* **Code Snippet:**
```typescript
const stream = await agent.streamEvents(input, { version: "v3" });

for await (const snapshot of stream.values) {
  console.log(snapshot);
}

const finalState = await stream.output;
```

## Multiple projections
* **Key Points:**
  - Use concurrent consumers when you want multiple projections in JavaScript.
  - To access channels that aren't exposed as typed projections, or to inspect the full event envelope, iterate raw protocol events.
* **Technical Entities (Classes/Functions/APIs):** `stream.messages`, `stream.toolCalls`, `stream`
* **Code Snippet:**
```typescript
const stream = await agent.streamEvents(input, { version: "v3" });

await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      console.log(await message.text);
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(call.name, call.input);
    }
  })(),
]);
```
```typescript
for await (const event of stream) {
  console.log(event.method, event.params.namespace, event.params.data);
}
```

## Custom updates
* **Key Points:**
  - Use custom stream transformers when your application needs a projection that is not built in, such as retrieval progress, artifacts, or domain-specific events.
* **Technical Entities (Classes/Functions/APIs):** `stream.extensions`, `transformers`, `toolActivityTransformer`
* **Code Snippet:**
```typescript
const stream = await agent.streamEvents(input, {
  version: "v3",
  transformers: [toolActivityTransformer],
});

for await (const activity of stream.extensions.toolActivity) {
  console.log(activity);
}
```

## Register transformers on middleware
* **Key Points:**
  - Middleware-registered transformers require langchain@1.4.3 or later.
  - Middleware can declare stream transformer factories alongside its hooks and tools.
  - The factory shape differs between languages.
  - Pass streamTransformers to createMiddleware as a tuple of factories.
  - Each factory has the shape () => StreamTransformer<any> (zero arguments) and is invoked once per scope.
  - Returning a fresh transformer per call keeps each subgraph isolated.
  - At compile time, createAgent merges middleware-registered factories with anything passed to its own streamTransformers option.
  - The final order on the compiled graph is:
    - The built-in ToolCallTransformer.
    - Middleware-registered factories, in middleware order.
    - Caller-supplied streamTransformers from createAgent.
  - This keeps the built-in tool-call projection in front of consumer transformers and gives caller-supplied entries the final word.
  - See Build your own projection for the transformer contract.
* **Technical Entities (Classes/Functions/APIs):** `langchain@1.4.3`, `createMiddleware()`, `streamTransformers`, `StreamTransformer`, `createAgent`, `streamTransformers option`, `ToolCallTransformer`, `Build your own projection`
* **Code Snippet:**
```typescript
import { createAgent, createMiddleware } from "langchain";

const toolActivityMiddleware = createMiddleware({
  name: "ToolActivityMiddleware",
  streamTransformers: [toolActivityTransformer],
});

const agent = createAgent({
  model: "gpt-5-nano",
  tools: [getWeather],
  middleware: [toolActivityMiddleware],
});
```

## Related
* **Key Points:**
  - Streaming covers low-level Pregel stream modes.
  - Build your own projection covers writing application-specific projections.
  - Frontend streaming patterns shows UI use cases built on streamed state.
* **Technical Entities (Classes/Functions/APIs):** `Pregel stream modes`, `Build your own projection`, `Frontend streaming patterns`