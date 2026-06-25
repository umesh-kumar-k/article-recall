---
aliases:
  - Memory
Source 1: https://docs.langchain.com/oss/javascript/langgraph/add-memory
---
## Memory
* **Key Points:**
  - AI applications need memory to share context across multiple interactions. In LangGraph, you can add two types of memory:
  - Add short-term memory as a part of your agent's state to enable multi-turn conversations.
  - Add long-term memory to store user-specific or application-level data across sessions.

## Add short-term memory
* **Key Points:**
  - Short-term memory (thread-level persistence) enables agents to track multi-turn conversations.
* **Technical Entities (Classes/Functions/APIs):** `MemorySaver`, `StateGraph`, `compile()`, `invoke()`, `thread_id`
* **Code Snippet:**
```typescript
import { MemorySaver, StateGraph } from "@langchain/langgraph";

const checkpointer = new MemorySaver();

const builder = new StateGraph(...);
const graph = builder.compile({ checkpointer });

await graph.invoke(
  { messages: [{ role: "user", content: "hi! i am Bob" }] },
  { configurable: { thread_id: "1" } }
);
```

### Use in production
* **Key Points:**
  - In production, use a checkpointer backed by a database:
* **Technical Entities (Classes/Functions/APIs):** `PostgresSaver`, `MongoDBSaver`, `MongoClient`

### Use in subgraphs
* **Key Points:**
  - If your graph contains subgraphs, you only need to provide the checkpointer when compiling the parent graph. LangGraph will automatically propagate the checkpointer to the child subgraphs.
  - You can configure subgraph-specific checkpointing behavior.
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `StateSchema`, `START`, `MemorySaver`, `z`, `compile()`

## Add long-term memory
* **Key Points:**
  - Use long-term memory to store user-specific or application-specific data across conversations.
* **Technical Entities (Classes/Functions/APIs):** `InMemoryStore`, `StateGraph`, `compile()`
* **Code Snippet:**
```typescript
import { InMemoryStore, StateGraph } from "@langchain/langgraph";

const store = new InMemoryStore();

const builder = new StateGraph(...);
const graph = builder.compile({ store });
```

### Access the store inside nodes
* **Key Points:**
  - Once you compile a graph with a store, LangGraph automatically injects the store into your node functions.
  - The recommended way to access the store is through the `Runtime` object.
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `StateSchema`, `MessagesValue`, `GraphNode`, `START`, `runtime.context`, `runtime.store`, `store.search()`, `store.put()`
* **Code Snippet:**
```typescript
import { StateGraph, StateSchema, MessagesValue, GraphNode, START } from "@langchain/langgraph";

const State = new StateSchema({
  messages: MessagesValue,
});

const callModel: GraphNode<typeof State> = async (state, runtime) => {
  const userId = runtime.context?.userId;
  const namespace = [userId, "memories"];

  // Search for relevant memories
  const memories = await runtime.store?.search(namespace, {
    query: state.messages.at(-1)?.content,
    limit: 3,
  });
  const info = memories?.map((d) => d.value.data).join("\n") || "";

  // ... Use memories in model call

  // Store a new memory
  await runtime.store?.put(namespace, crypto.randomUUID(), { data: "User prefers dark mode" });
};

const builder = new StateGraph(State)
  .addNode("call_model", callModel)
  .addEdge(START, "call_model");
const graph = builder.compile({ store });

// Pass context at invocation time
await graph.invoke(
  { messages: [{ role: "user", content: "hi" }] },
  { configurable: { thread_id: "1" }, context: { userId: "1" } }
);
```

### Use in production
* **Key Points:**
  - In production, use a store backed by a database:
* **Technical Entities (Classes/Functions/APIs):** `PostgresStore`, `MongoDBStore`

### Use semantic search
* **Key Points:**
  - Enable semantic search in your graph's memory store to let graph agents search for items in the store by semantic similarity.
* **Technical Entities (Classes/Functions/APIs):** `OpenAIEmbeddings`, `InMemoryStore`, `store.put()`, `store.search()`
* **Code Snippet:**
```typescript
import { OpenAIEmbeddings } from "@langchain/openai";
import { InMemoryStore } from "@langchain/langgraph";

// Create store with semantic search enabled
const embeddings = new OpenAIEmbeddings({ model: "text-embedding-3-small" });
const store = new InMemoryStore({
  index: {
    embeddings,
    dims: 1536,
  },
});

await store.put(["user_123", "memories"], "1", { text: "I love pizza" });
await store.put(["user_123", "memories"], "2", { text: "I am a plumber" });

const items = await store.search(["user_123", "memories"], {
  query: "I'm hungry",
  limit: 1,
});
```

## Manage short-term memory
* **Key Points:**
  - With short-term memory enabled, long conversations can exceed the LLM's context window. Common solutions are:
  - Trim messages: Remove first or last N messages (before calling LLM)
  - Delete messages from LangGraph state permanently
  - Summarize messages: Summarize earlier messages in the history and replace them with a summary
  - Manage checkpoints to store and retrieve message history
  - Custom strategies (e.g., message filtering, etc.)
  - This allows the agent to keep track of the conversation without exceeding the LLM's context window.

### Trim messages
* **Key Points:**
  - Most LLMs have a maximum supported context window (denominated in tokens). One way to decide when to truncate messages is to count the tokens in the message history and truncate whenever it approaches that limit.
  - If you're using LangChain, you can use the trim messages utility and specify the number of tokens to keep from the list, as well as the `strategy` (e.g., keep the last `maxTokens`) to use for handling the boundary.
* **Technical Entities (Classes/Functions/APIs):** `trimMessages`, `StateSchema`, `MessagesValue`, `GraphNode`
* **Code Snippet:**
```typescript
import { trimMessages } from "@langchain/core/messages";
import { StateSchema, MessagesValue, GraphNode } from "@langchain/langgraph";

const State = new StateSchema({
  messages: MessagesValue,
});

const callModel: GraphNode<typeof State> = async (state) => {
  const messages = trimMessages(state.messages, {
    strategy: "last",
    maxTokens: 128,
    startOn: "human",
    endOn: ["human", "tool"],
  });
  const response = await model.invoke(messages);
  return { messages: [response] };
};

const builder = new StateGraph(State)
  .addNode("call_model", callModel);
  // ...
```

### Delete messages
* **Key Points:**
  - You can delete messages from the graph state to manage the message history. This is useful when you want to remove specific messages or clear the entire message history.
  - To delete messages from the graph state, you can use the `RemoveMessage`. For `RemoveMessage` to work, you need to use a state key with `messagesStateReducer` reducer, like `MessagesValue`.
  - When deleting messages, make sure that the resulting message history is valid. Check the limitations of the LLM provider you're using.
* **Technical Entities (Classes/Functions/APIs):** `RemoveMessage`, `MessagesValue`
* **Code Snippet:**
```typescript
import { RemoveMessage } from "@langchain/core/messages";

const deleteMessages = (state) => {
  const messages = state.messages;
  if (messages.length > 2) {
    // remove the earliest two messages
    return {
      messages: messages
        .slice(0, 2)
        .map((m) => new RemoveMessage({ id: m.id })),
    };
  }
};
```

### Summarize messages
* **Key Points:**
  - The problem with trimming or removing messages, as shown above, is that you may lose information from culling of the message queue. Because of this, some applications benefit from a more sophisticated approach of summarizing the message history using a chat model.
  - Prompting and orchestration logic can be used to summarize the message history.
* **Technical Entities (Classes/Functions/APIs):** `StateSchema`, `MessagesValue`, `GraphNode`, `z`, `RemoveMessage`, `HumanMessage`
* **Code Snippet:**
```typescript
import { StateSchema, MessagesValue, GraphNode } from "@langchain/langgraph";
import { z } from "zod/v4";

const State = new StateSchema({
  messages: MessagesValue,
  summary: z.string().optional(),
});
```
```typescript
import { RemoveMessage, HumanMessage } from "@langchain/core/messages";

const summarizeConversation: GraphNode<typeof State> = async (state) => {
  // First, we get any existing summary
  const summary = state.summary || "";

  // Create our summarization prompt
  let summaryMessage: string;
  if (summary) {
    // A summary already exists
    summaryMessage =
      `This is a summary of the conversation to date: ${summary}\n\n` +
      "Extend the summary by taking into account the new messages above:";
  } else {
    summaryMessage = "Create a summary of the conversation above:";
  }

  // Add prompt to our history
  const messages = [
    ...state.messages,
    new HumanMessage({ content: summaryMessage })
  ];
  const response = await model.invoke(messages);

  // Delete all but the 2 most recent messages
  const deleteMessages = state.messages
    .slice(0, -2)
    .map(m => new RemoveMessage({ id: m.id }));

  return {
    summary: response.content,
    messages: deleteMessages
  };
};
```

### Manage checkpoints
* **Key Points:**
  - You can view and delete the information stored by the checkpointer.
* **Technical Entities (Classes/Functions/APIs):** `graph.getState()`, `graph.getStateHistory()`, `checkpointer.deleteThread()`, `thread_id`, `checkpoint_id`

#### View thread state
* **Code Snippet:**
```typescript
const config = {
  configurable: {
    thread_id: "1",
    // optionally provide an ID for a specific checkpoint,
    // otherwise the latest checkpoint is shown
    // checkpoint_id: "1f029ca3-1f5b-6704-8004-820c16b69a5a"
  },
};
await graph.getState(config);
```

#### View the history of the thread
* **Code Snippet:**
```typescript
const config = {
  configurable: {
    thread_id: "1",
  },
};

const history = [];
for await (const state of graph.getStateHistory(config)) {
  history.push(state);
}
```

#### Delete all checkpoints for a thread
* **Code Snippet:**
```typescript
const threadId = "1";
await checkpointer.deleteThread(threadId);
```

## Database management
* **Key Points:**
  - If you are using any database-backed persistence implementation (such as Postgres, Redis, or Oracle) to store short and/or long-term memory, you will need to run migrations to set up the required schema before you can use it with your database.
  - By convention, most database-specific libraries define a `setup()` method on the checkpointer or store instance that runs the required migrations.
  - We recommend running migrations as a dedicated deployment step, or you can ensure they're run as part of server startup.
* **Technical Entities (Classes/Functions/APIs):** `BaseCheckpointSaver`, `BaseStore`, `setup()`