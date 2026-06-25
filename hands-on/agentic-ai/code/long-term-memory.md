---
aliases:
  - Long Term Memory
Source 1: https://docs.langchain.com/oss/javascript/langchain/long-term-memory
---
# Long-term memory

* **Key Points:**
  - Long-term memory lets your agent store and recall information across different conversations and sessions.
  - Unlike short-term memory, which is scoped to a single thread, long-term memory persists across threads and can be recalled at any time.
  - Long-term memory is built on LangGraph stores, which save data as JSON documents organized by namespace and key.

## Usage
* **Key Points:**
  - To add long-term memory to an agent, create a store and pass it to `create_agent`:
* **Technical Entities (Classes/Functions/APIs):** `createAgent`, `InMemoryStore`, `PostgresStore`, `store.setup()`
* **Code Snippet:**
```ts
import { createAgent } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore();

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [],
  store,
});
```
```ts
import { createAgent } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [],
  store,
});
```

## Memory storage
* **Key Points:**
  - LangGraph stores long-term memories as JSON documents in a store.
  - Each memory is organized under a custom `namespace` (similar to a folder) and a distinct `key` (like a file name). Namespaces often include user or org IDs or other labels that makes it easier to organize information.
  - This structure enables hierarchical organization of memories. Cross-namespace searching is then supported through content filters.
* **Technical Entities (Classes/Functions/APIs):** `InMemoryStore`, `PostgresStore`, `store.put()`, `store.get()`, `store.search()`, `filter`, `query`
* **Code Snippet:**
```ts
import { InMemoryStore } from "@langchain/langgraph";

const embed = (texts: string[]): number[][] => {
  // Replace with an actual embedding function or LangChain embeddings object
  return texts.map(() => [1.0, 2.0]);
};

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore({ index: { embed, dims: 2 } });
const userId = "my-user";
const applicationContext = "chitchat";
const namespace = [userId, applicationContext];

await store.put(namespace, "a-memory", {
  rules: [
    "User likes short, direct language",
    "User only speaks English & TypeScript",
  ],
  "my-key": "my-value",
});

// get the "memory" by ID
const item = await store.get(namespace, "a-memory");

// search for "memories" within this namespace, filtering on content equivalence, sorted by vector similarity
const items = await store.search(namespace, {
  filter: { "my-key": "my-value" },
  query: "language preferences",
});
```
```ts
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const embed = (texts: string[]): number[][] => {
  return texts.map(() => [1.0, 2.0]);
};

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI, {
  index: { embed, dims: 2 },
});
await store.setup();

const userId = "my-user";
const applicationContext = "chitchat";
const namespace = [userId, applicationContext];

await store.put(namespace, "a-memory", {
  rules: [
    "User likes short, direct language",
    "User only speaks English & TypeScript",
  ],
  "my-key": "my-value",
});

const item = await store.get(namespace, "a-memory");
const items = await store.search(namespace, {
  filter: { "my-key": "my-value" },
  query: "language preferences",
});
```

## Read long-term memory in tools
* **Key Points:**
  - Tools can then read from and write to the store using the `runtime.store` parameter.
* **Technical Entities (Classes/Functions/APIs):** `z`, `createAgent`, `tool`, `ToolRuntime`, `InMemoryStore`, `PostgresStore`, `contextSchema`, `store.put()`, `store.get()`, `runtime.store`, `runtime.context`
* **Code Snippet:**
```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();
const contextSchema = z.object({
  userId: z.string(),
});

// Write sample data to the store using the put method
await store.put(
  ["users"], // Namespace to group related data together (users namespace for user data)
  "user_123", // Key within the namespace (user ID as key)
  {
    name: "John Smith",
    language: "English",
  }, // Data to store for the given user
);

const getUserInfo = tool(
  // Look up user info.
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    // Access the store - same as that provided to `createAgent`
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Retrieve data from store - returns StoreValue object with value and metadata
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [getUserInfo],
  contextSchema,
  // Pass store to agent - enables agent to access store when running tools
  store,
});

// Run the agent
const result = await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);

console.log(result.messages.at(-1)?.content);

/**
 * Outputs:
 * User Information:
 * - **Name:** John Smith
 * - **Language:** English
 */
```
```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

await store.put(["users"], "user_123", {
  name: "John Smith",
  language: "English",
});

const getUserInfo = tool(
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [getUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);
```

## Write long-term memory from tools
* **Key Points:**
  - Tools can then read from and write to the store using the `runtime.store` parameter.
* **Technical Entities (Classes/Functions/APIs):** `z`, `tool`, `createAgent`, `ToolRuntime`, `InMemoryStore`, `PostgresStore`, `contextSchema`, `store.put()`, `runtime.store`, `store.get()`
* **Code Snippet:**
```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();

const contextSchema = z.object({
  userId: z.string(),
});

// Schema defines the structure of user information for the LLM
const UserInfo = z.object({
  name: z.string(),
});

// Tool that allows agent to update user information (useful for chat applications)
const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Store data in the store (namespace, key, data)
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  {
    name: "save_user_info",
    description: "Save user info",
    schema: UserInfo,
  },
);

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

// Run the agent
await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  // userId passed in context to identify whose information is being updated
  { context: { userId: "user_123" } },
);

// You can access the store directly to get the value
const result = await store.get(["users"], "user_123");
console.log(result?.value); // Output: { name: "John Smith" }
```
```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

const UserInfo = z.object({ name: z.string() });

const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  { name: "save_user_info", description: "Save user info", schema: UserInfo },
);

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  { context: { userId: "user_123" } },
);

const result = await store.get(["users"], "user_123");
console.log(result?.value);
```


