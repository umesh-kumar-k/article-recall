---
aliases:
  - Multi Agent Custom Workflow
Source 1: https://docs.langchain.com/oss/javascript/langchain/multi-agent/custom-workflow
---
## Custom workflow
* **Key Points:**
  - In the **custom workflow** architecture, you define your own bespoke execution flow using LangGraph.
  - You have complete control over the graph structure—including sequential steps, conditional branches, loops, and parallel execution.

## Key characteristics
* **Key Points:**
  - Complete control over graph structure
  - Mix deterministic logic with agentic behavior
  - Support for sequential steps, conditional branches, loops, and parallel execution
  - Embed other patterns as nodes in your workflow

## When to use
* **Key Points:**
  - Use custom workflows when standard patterns (subagents, skills, etc.) don't fit your requirements, you need to mix deterministic logic with agentic behavior, or your use case requires complex routing or multi-stage processing.
  - Each node in your workflow can be a simple function, an LLM call, or an entire agent with tools.
  - You can also compose other architectures within a custom workflow—for example, embedding a multi-agent system as a single node.

## Basic implementation
* **Key Points:**
  - The core insight is that you can call a LangChain agent directly inside any LangGraph node, combining the flexibility of custom workflows with the convenience of prebuilt agents:
* **Technical Entities (Classes/Functions/APIs):** `createAgent`, `StateGraph`, `START`, `END`, `StateSchema`, `MessagesValue`, `GraphNode`
* **Code Snippet:**
```typescript
import { z } from "zod";
import { createAgent } from "langchain";
import { StateGraph, START, END, StateSchema, MessagesValue } from "@langchain/langgraph";

const agent = createAgent({ model: "openai:gpt-4o", tools: [...] });

const AgentState = new StateSchema({
  messages: MessagesValue,
  query: z.string(),
});

const agentNode: GraphNode<typeof AgentState> = (state) => {
  // A LangGraph node that invokes a LangChain agent
  const result = await agent.invoke({
    messages: [{ role: "user", content: state.query }]
  });
  return { answer: result.messages.at(-1)?.content };
}

// Build a simple workflow
const workflow = new StateGraph(State)
  .addNode("agent", agentNode)
  .addEdge(START, "agent")
  .addEdge("agent", END)
  .compile();
```

## Example: RAG pipeline
* **Key Points:**
  - A common use case is combining retrieval with an agent.
  - The workflow demonstrates three types of nodes:
  - **Model node** (Rewrite): Rewrites the user query for better retrieval using structured output.
  - **Deterministic node** (Retrieve): Performs vector similarity search — no LLM involved.
  - **Agent node** (Agent): Reasons over retrieved context and can fetch additional information via tools.
  - You can use LangGraph state to pass information between workflow steps. This allows each part of your workflow to read and update structured fields, making it easy to share data and context across nodes.
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `Annotation`, `START`, `END`, `createAgent`, `tool`, `ChatOpenAI`, `OpenAIEmbeddings`, `MemoryVectorStore`, `z`, `withStructuredOutput`, `asRetriever`, `invoke()`, `addNode`, `addEdge`, `compile()`