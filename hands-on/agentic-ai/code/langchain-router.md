---
aliases:
  - Multi Agent Router Workflow
Source 1: https://docs.langchain.com/oss/javascript/langchain/multi-agent/router
---
## Router
* **Key Points:**
  - In the **router** architecture, a routing step classifies input and directs it to specialized agents.
  - This is useful when you have distinct **verticals** (separate knowledge domains that each require their own agent).

## Key characteristics
* **Key Points:**
  - Router decomposes the query
  - Zero or more specialized agents are invoked in parallel
  - Results are synthesized into a coherent response

## When to use
* **Key Points:**
  - Use the router pattern when you have distinct verticals (separate knowledge domains that each require their own agent), need to query multiple sources in parallel, and want to synthesize results into a combined response.

## Basic implementation
* **Key Points:**
  - The router classifies the query and directs it to the appropriate agent(s).
  - Use `Command` for single-agent routing or `Send` for parallel fan-out to multiple agents.
* **Technical Entities (Classes/Functions/APIs):** `Command`, `Send`, `z`
* **Code Snippet:**
```typescript
import { z } from "zod";
import { Command } from "@langchain/langgraph";

const ClassificationResult = z.object({
  query: z.string(),
  agent: z.string(),
});

function classifyQuery(query: string): z.infer<typeof ClassificationResult> {
  // Use LLM to classify query and determine the appropriate agent
  // Classification logic here
  ...
}

function routeQuery(state: z.infer<typeof ClassificationResult>) {
  const classification = classifyQuery(state.query);

  // Route to the selected agent
  return new Command({ goto: classification.agent });
}
```

## Stateless vs. stateful
* **Key Points:**
  - Two approaches: **Stateless routers** address each request independently, **Stateful routers** maintain conversation history across requests.

## Stateless
* **Key Points:**
  - Each request is routed independently—no memory between calls.
  - Router vs. Subagents: Both patterns can dispatch work to multiple agents, but they differ in how routing decisions are made:
  - **Router**: A dedicated routing step (often a single LLM call or rule-based logic) that classifies the input and dispatches to agents. The router itself typically doesn't maintain conversation history or perform multi-turn orchestration—it's a preprocessing step.
  - **Subagents**: An main supervisor agent dynamically decides which subagents to call as part of an ongoing conversation. The main agent maintains context, can call multiple subagents across turns, and orchestrates complex multi-step workflows.
  - Use a **router** when you have clear input categories and want deterministic or lightweight classification. Use a **supervisor** when you need flexible, conversation-aware orchestration where the LLM decides what to do next based on evolving context.

## Stateful
* **Key Points:**
  - For multi-turn conversations, you need to maintain context across invocations.
  - The simplest approach: wrap the stateless router as a tool that a conversational agent can call. The conversational agent handles memory and context; the router stays stateless.
  - If you need the router itself to maintain state, use persistence to store message history.
  - When routing to an agent, fetch previous messages from state and selectively include them in the agent's context—this is a lever for context engineering.
  - Stateful routers require custom history management.
  - If the router switches between agents across turns, conversations may not feel fluid to end users when agents have different tones or prompts.
  - With parallel invocation, you'll need to maintain history at the router level (inputs and synthesized outputs) and leverage this history in routing logic.
  - Consider the handoffs pattern or subagents pattern instead—both provide clearer semantics for multi-turn conversations.
* **Technical Entities (Classes/Functions/APIs):** `tool`, `createAgent`, `z`, `workflow.invoke()`
* **Code Snippet:**
```typescript
const searchDocs = tool(
  async ({ query }) => {
    const result = await workflow.invoke({ query });
    return result.finalAnswer;
  },
  {
    name: "search_docs",
    description: "Search across multiple documentation sources",
    schema: z.object({
      query: z.string().describe("The search query"),
    }),
  }
);

// Conversational agent uses the router as a tool
const conversationalAgent = createAgent({
  model,
  tools: [searchDocs],
  systemPrompt: "You are a helpful assistant. Use search_docs to answer questions.",
});
```