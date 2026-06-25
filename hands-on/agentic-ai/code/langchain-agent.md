---
aliases:
  - Agents
Source 1: https://docs.langchain.com/oss/javascript/langchain/agents
---
## Agents
* **Key Points:**
  - An agent is a model calling tools in a loop until a given task is complete.
  - **Agent = Model + Harness**
  - The job of a harness: get the model the right context at the right time for the given task.
  - A harness is everything around that loop: the model, its prompt, its tools, and any middleware that shapes its behavior.
  - [`create_agent`](https://reference.langchain.com/javascript/langchain/index/createAgent) is a highly configurable harness.
* **Technical Entities (Classes/Functions/APIs):** `create_agent()`, `model`, `tools`
* **Code Snippet:**
```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "openai:gpt-5.4", tools });
```

## Core components
### Model
* **Key Points:**
  - Pass a model identifier string (`"provider:model"`) or an initialized model instance to select the model for your agent.
* **Technical Entities (Classes/Functions/APIs):** `create_agent()`, `model`

### Tools
* **Key Points:**
  - To provide the agent with tools, pass any Python callable, LangChain tool, or tool dict.
* **Technical Entities (Classes/Functions/APIs):** `tool`, `zod`, `create_agent()`
* **Code Snippet:**
```ts
import { tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Results for: ${query}`, {
  name: "search",
  description: "Search for information",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({ model: "openai:gpt-5.4", tools: [search] });
```

### System prompt
* **Key Points:**
  - Shape how the agent approaches tasks.
  - The system prompt parameter accepts a string or `SystemMessage`.
  - For dynamic prompts at runtime, use [middleware](/oss/javascript/langchain/middleware).
* **Technical Entities (Classes/Functions/APIs):** `systemPrompt`, `SystemMessage`
* **Code Snippet:**
```ts
var agent = createAgent({
  model: "openai:gpt-5.4",
  tools,
  systemPrompt: "You are a helpful assistant. Be concise and accurate.",
});
```

### Structured output
* **Key Points:**
  - Return a validated schema from the agent using `response_format=`.
* **Technical Entities (Classes/Functions/APIs):** `responseFormat`, `z.object()`, `invoke()`, `structuredResponse`
* **Code Snippet:**
```ts
const Answer = z.object({ summary: z.string(), confidence: z.number() });

var agent = createAgent({
  model: "openai:gpt-5.4",
  tools,
  responseFormat: Answer,
});
const result = await agent.invoke({
  messages: [{ role: "user", content: "Summarize AI trends" }],
});
result.structuredResponse; // { summary: ..., confidence: ... }
```

## Invocation
* **Key Points:**
  - You can invoke an agent with a message.
  - Behind the scenes that passes an update to the agent's [`State`](/oss/javascript/langgraph/graph-api#state).
  - All agents include a [sequence of messages](/oss/javascript/langgraph/use-graph-api#messagesvalue) in their state; to invoke the agent, pass a new message along with a `thread_id` so the agent can persist and resume conversation history:
  - Persisting conversation history with `thread_id` requires the agent to be configured with a [checkpointer](/oss/javascript/langchain/long-term-memory).
  - When deployed on [LangSmith](/langsmith/deployment), a checkpointer is provisioned automatically.
  - Locally, pass one explicitly, for example `create_agent(..., checkpointer=InMemorySaver())`.
  - If you also need to pass per-run configuration (such as a user ID, API keys, or feature flags) to tools and middleware, pass it as `context` alongside the config.
  - Define the shape of that data with `contextSchema` and access it through `runtime.context`.
  - `thread_id` scopes the *conversation* (message history, checkpoints), while `context` carries *per-run* data your tools and middleware read at invocation time.
* **Technical Entities (Classes/Functions/APIs):** `invoke()`, `State`, `thread_id`, `checkpointer`, `MemorySaver`, `InMemorySaver()`, `contextSchema`, `runtime.context`, `context`
* **Code Snippet:**
```ts
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

let result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  config,
);

// A follow-up turn on the same conversation: reuse the same thread_id to keep history
result = await agent.invoke(
  { messages: [{ role: "user", content: "What about tomorrow?" }] },
  config,
);
```
```ts
import * as z from "zod";
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const contextSchema = z.object({
  user_id: z.string(),
});

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [],
  contextSchema,
  checkpointer: new MemorySaver(),
});

const result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_id: "user-123" },
  },
);
```

## Streaming
* **Key Points:**
  - `invoke` returns the final response at the end of a run.
  - If an agent executes multiple tool calls, users often need progress updates before completion.
  - Use streaming to surface intermediate messages and tool activity as they happen.
* **Technical Entities (Classes/Functions/APIs):** `streamEvents()`, `snapshot`, `messages`, `tool_calls`
* **Code Snippet:**
```ts
const stream = await agent.streamEvents(
  {
    messages: [
      {
        role: "user",
        content: "Search for AI news and summarize the findings",
      },
    ],
  },
  { version: "v3" },
);

for await (const snapshot of stream.values) {
  // Each snapshot contains the full state at that point
  const latestMessage = snapshot.messages.at(-1);
  if (latestMessage?.content) {
    if (latestMessage.type === "human") {
      console.log(`User: ${latestMessage.content}`);
    } else if (latestMessage.type === "ai") {
      console.log(`Agent: ${latestMessage.content}`);
    }
  } else if (latestMessage?.tool_calls?.length) {
    const toolCallNames = latestMessage.tool_calls.map((tc) => tc.name);
    console.log(`Calling tools: ${toolCallNames.join(", ")}`);
  }
}
```

## Configure the harness
* **Key Points:**
  - `create_agent` is highly extensible.
  - Middleware is the primitive for customization: each piece handles one concern, hooks into the agent loop at the right moment, and composes freely with any other.
  - Take exactly what your use case needs and skip the rest.
  - Common patterns are prebuilt as first-class middleware.
  - You can build anything else as [custom middleware](/oss/javascript/langchain/middleware/custom).
  - `create_deep_agent` pre-assembles this stack for long-running coding and research tasks (filesystem, summarization, subagents, and prompt caching included by default).
* **Technical Entities (Classes/Functions/APIs):** `create_agent`, `middleware`, `create_deep_agent`

### Execution environment
* **Key Points:**
  - Agents are especially useful when they can take action rather than just generate text.
  - The execution environment gives the agent a workspace: tools it can call, a filesystem for reading and writing files across turns, and code execution for running scripts or shell commands.
* **Technical Entities (Classes/Functions/APIs):** `createFilesystemMiddleware`, `StateBackend`
* **Code Snippet:**
```ts
import { createAgent } from "langchain";
import { createFilesystemMiddleware, StateBackend } from "deepagents";

var agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [search],
  middleware: [createFilesystemMiddleware({ backend: new StateBackend() })],
});
```

### Context management
* **Key Points:**
  - Every model call has a fixed context window.
  - As an agent runs, that window fills with accumulating history, tool results, and intermediate steps.
  - Summarization compresses history before overflow hits; memory loads persistent instructions at startup so knowledge carries across sessions; skills surface domain knowledge on demand rather than loading everything upfront.
* **Technical Entities (Classes/Functions/APIs):** `createSummarizationMiddleware`, `createSkillsMiddleware`
* **Code Snippet:**
```ts
import { createAgent } from "langchain";
import {
  StateBackend,
  createFilesystemMiddleware,
  createSkillsMiddleware,
  createSummarizationMiddleware,
} from "deepagents";

var backend = new StateBackend();
const model = "anthropic:claude-sonnet-4-6";

var agent = createAgent({
  model,
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    createSummarizationMiddleware({ model, backend }),
    createSkillsMiddleware({ backend, sources: ["./skills/"] }),
  ],
});
```

### Planning and delegation
* **Key Points:**
  - Complex tasks often exceed what one context window can handle.
  - Delegation lets the main agent break work into pieces, hand them to subagents that each run in their own isolated context, and stay focused on coordination rather than execution.
  - Work can run in parallel; the main agent's context stays clean.
* **Technical Entities (Classes/Functions/APIs):** `todoListMiddleware`, `createSubAgentMiddleware`, `StateBackend`, `subagents`
* **Code Snippet:**
```ts
import { createAgent, todoListMiddleware, tool } from "langchain";
import {
  createFilesystemMiddleware,
  createSubAgentMiddleware,
  StateBackend,
} from "deepagents";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var backend = new StateBackend();

var agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    todoListMiddleware(),
    createSubAgentMiddleware({
      defaultModel: "anthropic:claude-sonnet-4-6",
      defaultTools: [],
      subagents: [
        {
          name: "researcher",
          description: "Searches and returns a structured summary.",
          systemPrompt:
            "Use the search tool to research the question and summarize key points.",
          tools: [search],
          model: "anthropic:claude-sonnet-4-6",
          middleware: [],
        },
      ],
    }),
  ],
});
```

### Name your agent
* **Key Points:**
  - Optionally use an identifier for the agent.
  - This is especially useful when embedding the agent as a subgraph in [multi-agent](/oss/javascript/langchain/multi-agent) systems.
* **Technical Entities (Classes/Functions/APIs):** `name`
* **Code Snippet:**
```ts
var agent = createAgent({
  model: "openai:gpt-5.4",
  tools,
  name: "research_assistant",
});
```

### Fault tolerance
* **Key Points:**
  - Agents in production encounter failures that rarely appear in development: rate limits, model timeouts, transient API errors.
  - Fault tolerance middleware handles these at the infrastructure level so your tools and business logic don't need try/catch around every call.
* **Technical Entities (Classes/Functions/APIs):** `modelRetryMiddleware`, `toolRetryMiddleware`
* **Code Snippet:**
```ts
import {
  createAgent,
  modelRetryMiddleware,
  tool,
  toolRetryMiddleware,
} from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [search],
  middleware: [
    modelRetryMiddleware({ maxRetries: 3 }),
    toolRetryMiddleware({ maxRetries: 2 }),
  ],
});
```

### Guardrails
* **Key Points:**
  - Some policies can't live in a prompt—they need to be enforced deterministically regardless of what the model does.
  - Guardrails intercept data as it flows through the agent loop, applying compliance rules or content policies before tool results reach the model's context.
* **Technical Entities (Classes/Functions/APIs):** `piiMiddleware`
* **Code Snippet:**
```ts
import { createAgent, piiMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [search],
  middleware: [piiMiddleware("email")],
});
```

### Steering
* **Key Points:**
  - Full autonomy isn't always appropriate.
  - Steering lets you place humans at specific decision points—before destructive writes, expensive API calls, or anything requiring judgment—without restructuring your agent.
  - The agent pauses and waits; a human approves, edits, or rejects; execution continues.
* **Technical Entities (Classes/Functions/APIs):** `humanInTheLoopMiddleware`
* **Code Snippet:**
```ts
import { createAgent, humanInTheLoopMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [search],
  middleware: [humanInTheLoopMiddleware({ interruptOn: { writeFile: true } })],
});
```

### Middleware resources
* **Key Points:**
  - How the middleware stack works and when hooks fire
  - Full reference with configuration examples
  - Write your own hooks for business logic, PII scrubbing, and more