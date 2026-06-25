---
aliases:
  - Stores
Source 1: https://docs.langchain.com/oss/javascript/langgraph/stores
---
## Stores
* **Key Points:**
  - LangGraph stores provide cross-thread long-term memory, complementing per-thread checkpointer persistence.
  - Stores let agents persist information across threads, including user preferences, accumulated knowledge, and facts that should survive beyond a single conversation.
  - Unlike checkpointers, which save the full graph state scoped to one thread, stores hold arbitrary key-value data accessible from any thread.
  - When using the Agent Server, you do not need to implement or configure stores manually. The API handles all storage infrastructure for you behind the scenes.
* **Technical Entities (Classes/Functions/APIs):** `InMemoryStore`, `MemoryStore`, `BaseStore`, `PostgresStore`, `MongoDBStore`, `RedisStore`

## Basic usage
* **Key Points:**
  - The following code snippet shows the InMemoryStore in isolation without using LangGraph:
  - Memories are namespaced by a `tuple`, which is `(<user_id>, "memories")` in the following example. The namespace can be any length and represent anything, does not have to be user specific.
  - Use the `store.put` method to save memories to the namespace in the store. Specify the namespace, as defined above, and a key-value pair for the memory: the key is simply a unique identifier for the memory (`memory_id`) and the value (a dictionary) is the memory itself.
  - Read out memories from your namespace using the `store.search` method, which returns memories for a given user as a list, up to the `limit` argument (default `10`).
  - With `InMemoryStore`, items are returned in insertion order, so the most recent memory is last in the list; other backends may order memories differently.
* **Technical Entities (Classes/Functions/APIs):** `MemoryStore`, `store.put()`, `store.search()`, `crypto.randomUUID()`
* **Code Snippet:**
```typescript
import { MemoryStore } from "@langchain/langgraph";

const memoryStore = new MemoryStore();
```
```typescript
const userId = "1";
const namespaceForMemory = [userId, "memories"];
```
```typescript
const memoryId = crypto.randomUUID();
const memory = { food_preference: "I like pizza" };
await memoryStore.put(namespaceForMemory, memoryId, memory);
```
```typescript
const memories = await memoryStore.search(namespaceForMemory);
memories[memories.length - 1];

// {
//   value: { food_preference: 'I like pizza' },
//   key: '07e0caf4-1631-47b7-b15f-65515d4c1843',
//   namespace: ['1', 'memories'],
//   createdAt: '2024-10-02T17:22:31.590602+00:00',
//   updatedAt: '2024-10-02T17:22:31.590605+00:00'
// }
```

## Listing items in a namespace
* **Key Points:**
  - Calling `store.search` with no `query` and no `filter` returns the items stored under the namespace prefix, up to `limit`. Use this to enumerate everything in a namespace when you don't need semantic ranking.
  - `namespace_prefix` matches by prefix, not exactly. `("alice",)` also returns items under `("alice", "memories")`, `("alice", "preferences")`, and so on. To restrict to a single level, pass the full namespace or filter the returned items client-side on `item.namespace`.
  - Results past `limit` are silently truncated. There is no overflow signal—set `limit` above your expected maximum, or paginate with `offset`.
  - Default ordering depends on the store backend. `PostgresStore` and `AsyncPostgresStore` return results ordered by `updated_at` descending (most recently updated first). `InMemoryStore` returns results in insertion order (most recently inserted last). Do not rely on a specific order across implementations; sort client-side on `item.updated_at` if order matters.
* **Technical Entities (Classes/Functions/APIs):** `store.search()`, `store.listNamespaces()`
* **Code Snippet:**
```ts
// Return up to 100 items stored under ["alice", "memories"].
const items = await store.search(["alice", "memories"], { limit: 100 });
```
```ts
const pageSize = 50;
let offset = 0;
while (true) {
  const page = await store.search(["alice", "memories"], { limit: pageSize, offset });
  if (page.length === 0) break;
  for (const item of page) {
    // ...
  }
  offset += pageSize;
}
```
```ts
// All namespaces that start with ["alice"], truncated to two levels deep.
const namespaces = await store.listNamespaces({ prefix: ["alice"], maxDepth: 2 });
```

## Semantic search
* **Key Points:**
  - Beyond simple retrieval, the store also supports semantic search, allowing you to find memories based on meaning rather than exact matches.
  - To enable this, configure the store with an embedding model:
  - Now when searching, you can use natural language queries to find relevant memories:
  - You can control which parts of your memories get embedded by configuring the `fields` parameter or by specifying the `index` parameter when storing memories:
* **Technical Entities (Classes/Functions/APIs):** `OpenAIEmbeddings`, `InMemoryStore`, `store.put()`, `store.search()`
* **Code Snippet:**
```typescript
import { OpenAIEmbeddings } from "@langchain/openai";

const store = new InMemoryStore({
  index: {
    embeddings: new OpenAIEmbeddings({ model: "text-embedding-3-small" }),
    dims: 1536,
    fields: ["food_preference", "$"], // Fields to embed
  },
});
```
```typescript
// Find memories about food preferences
// (This can be done after putting memories into the store)
const memories = await store.search(namespaceForMemory, {
  query: "What does the user like to eat?",
  limit: 3, // Return top 3 matches
});
```
```typescript
// Store with specific fields to embed
await store.put(
  namespaceForMemory,
  crypto.randomUUID(),
  {
    food_preference: "I love Italian cuisine",
    context: "Discussing dinner plans",
  },
  { index: ["food_preference"] } // Only embed "food_preferences" field
);

// Store without embedding (still retrievable, but not searchable)
await store.put(
  namespaceForMemory,
  crypto.randomUUID(),
  { system_info: "Last updated: 2024-01-01" },
  { index: false }
);
```

## Using in LangGraph
* **Key Points:**
  - The `memoryStore` works hand-in-hand with the checkpointer: the checkpointer saves state to threads, as discussed above, and the `memoryStore` allows you to store arbitrary information for access *across* threads.
  - Compile the graph with both the checkpointer and the `memoryStore` as follows.
  - Then invoke the graph with a `thread_id`, as before, and also with a `user_id`, which serves as the namespace for memories for this particular user as before.
  - You can access the store and the `userId` from *any node* with the `runtime` argument.
  - You can use it to save memories:
  - You can also access the store from any node and use the `store.search` method to get memories. Memories are returned as a list of objects that can be converted to a dictionary.
  - You access the memories and use them in model calls.
  - If you create a new thread, you can still access the same memories so long as the `user_id` is the same.
  - When you use LangSmith locally (e.g., in Studio) or hosted, the base store is available to use by default and you do not need to specify it during graph compilation.
  - To enable semantic search, however, you **do** need to configure the indexing settings in your `langgraph.json` file.
* **Technical Entities (Classes/Functions/APIs):** `MemorySaver`, `workflow.compile()`, `graph.stream()`, `runtime.context`, `runtime.store`, `store.put()`, `store.search()`, `GraphNode`, `StateSchema`, `MessagesValue`
* **Code Snippet:**
```typescript
import { MemorySaver } from "@langchain/langgraph";

// We need this because we want to enable threads (conversations)
const checkpointer = new MemorySaver();

// ... Define the graph ...

// Compile the graph with the checkpointer and store
const graph = workflow.compile({ checkpointer, store: memoryStore });
```
```typescript
// Invoke the graph
const userId = "1";
const config = { configurable: { thread_id: "1" }, context: { userId } };

// First let's just say hi to the AI
for await (const update of await graph.stream(
  { messages: [{ role: "user", content: "hi" }] },
  { ...config, streamMode: "updates" }
)) {
  console.log(update);
}
```
```typescript
import { StateSchema, MessagesValue, Runtime } from "@langchain/langgraph";

const MessagesState = new StateSchema({
  messages: MessagesValue,
});

const updateMemory: GraphNode<typeof MessagesState> = async (state, runtime) => {
  // Get the user id from the config
  const userId = runtime.context?.user_id;
  if (!userId) throw new Error("User ID is required");

  // Namespace the memory
  const namespace = [userId, "memories"];

  // ... Analyze conversation and create a new memory
  const memory = "Some memory content";

  // Create a new memory ID
  const memoryId = crypto.randomUUID();

  // We create a new memory
  await runtime.store?.put(namespace, memoryId, { memory });
};
```
```json
{
    ...
    "store": {
        "index": {
            "embed": "openai:text-embeddings-3-small",
            "dims": 1536,
            "fields": ["$"]
        }
    }
}
```