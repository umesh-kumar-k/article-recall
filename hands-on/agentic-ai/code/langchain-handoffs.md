---
aliases:
  - Multi Agent Handoff Workflow
Source 1: https://docs.langchain.com/oss/javascript/langchain/multi-agent/handoffs
---
## Handoffs
* **Key Points:**
  - In the **handoffs** architecture, behavior changes dynamically based on state.
  - The core mechanism: tools update a state variable (e.g., `current_step` or `active_agent`) that persists across turns, and the system reads this variable to adjust behavior—either applying different configuration (system prompt, tools) or routing to a different agent.
  - This pattern supports both handoffs between distinct agents and dynamic configuration changes within a single agent.
  - The term **handoffs** was coined by OpenAI for using tool calls (e.g., `transfer_to_sales_agent`) to transfer control between agents or states.

## Key characteristics
* **Key Points:**
  - State-driven behavior: Behavior changes based on a state variable (e.g., `current_step` or `active_agent`)
  - Tool-based transitions: Tools update the state variable to move between states
  - Direct user interaction: Each state's configuration handles user messages directly
  - Persistent state: State survives across conversation turns

## When to use
* **Key Points:**
  - Use the handoffs pattern when you need to enforce sequential constraints (unlock capabilities only after preconditions are met), the agent needs to converse directly with the user across different states, or you're building multi-stage conversational flows.
  - This pattern is particularly valuable for customer support scenarios where you need to collect information in a specific sequence—for example, collecting a warranty ID before processing a refund.

## Basic implementation
* **Key Points:**
  - The core mechanism is a tool that returns a `Command` to update state, triggering a transition to a new step or agent:
  - Why include a `ToolMessage`? When an LLM calls a tool, it expects a response. The `ToolMessage` with matching `tool_call_id` completes this request-response cycle—without it, the conversation history becomes malformed. This is required whenever your handoff tool updates messages.
* **Technical Entities (Classes/Functions/APIs):** `tool`, `Command`, `ToolMessage`, `ToolRuntime`, `z`, `toolCallId`, `currentStep`
* **Code Snippet:**
```typescript
import { tool, ToolMessage, type ToolRuntime } from "langchain";
import { Command } from "@langchain/langgraph";
import { z } from "zod";

const transferToSpecialist = tool(
  async (_, config: ToolRuntime<typeof StateSchema>) => {
    return new Command({
      update: {
        messages: [
          new ToolMessage({
            content: "Transferred to specialist",
            tool_call_id: config.toolCallId
          })
        ],
        currentStep: "specialist"  // Triggers behavior change
      }
    });
  },
  {
    name: "transfer_to_specialist",
    description: "Transfer to the specialist agent.",
    schema: z.object({})
  }
);
```

## Implementation approaches
* **Key Points:**
  - There are two ways to implement handoffs: **single agent with middleware** (one agent with dynamic configuration) or **multiple agent subgraphs** (distinct agents as graph nodes).

### Single agent with middleware
* **Key Points:**
  - A single agent changes its behavior based on state.
  - Middleware intercepts each model call and dynamically adjusts the system prompt and available tools.
  - Tools update the state variable to trigger transitions:
* **Technical Entities (Classes/Functions/APIs):** `tool`, `ToolMessage`, `ToolRuntime`, `Command`, `StateSchema`, `createAgent`, `createMiddleware`, `wrapModelCall`, `MemorySaver`
* **Code Snippet:**
```typescript
import { tool, ToolMessage, type ToolRuntime } from "langchain";
import { Command } from "@langchain/langgraph";
import { z } from "zod";

const recordWarrantyStatus = tool(
  async ({ status }, config: ToolRuntime<typeof StateSchema>) => {
    return new Command({
      update: {
        messages: [
          new ToolMessage({
            content: `Warranty status recorded: ${status}`,
            tool_call_id: config.toolCallId,
          }),
        ],
        warrantyStatus: status,
        currentStep: "specialist", // Update state to trigger transition
      },
    });
  },
  {
    name: "record_warranty_status",
    description: "Record warranty status and transition to next step.",
    schema: z.object({
      status: z.string(),
    }),
  }
);
```

### Multiple agent subgraphs
* **Key Points:**
  - Multiple distinct agents exist as separate nodes in a graph.
  - Handoff tools navigate between agent nodes using `Command.PARENT` to specify which node to execute next.
  - Subgraph handoffs require careful context engineering. Unlike single-agent middleware (where message history flows naturally), you must explicitly decide what messages pass between agents.
  - Use **single agent with middleware** for most handoffs use cases—it's simpler.
  - Only use **multiple agent subgraphs** when you need bespoke agent implementations (e.g., a node that's itself a complex graph with reflection or retrieval steps).
* **Technical Entities (Classes/Functions/APIs):** `Command.PARENT`, `AIMessage`, `ToolMessage`, `tool`, `ToolRuntime`, `StateGraph`, `START`, `END`, `MessagesValue`, `ConditionalEdgeRouter`
* **Code Snippet:**
```typescript
import {
  tool,
  ToolMessage,
  AIMessage,
  type ToolRuntime,
} from "langchain";
import { Command, StateSchema, MessagesValue } from "@langchain/langgraph";

const CustomState = new StateSchema({
  messages: MessagesValue,
});

const transferToSales = tool(
  async (_, runtime: ToolRuntime<typeof CustomState.State>) => {
    const lastAiMessage = runtime.state.messages
      .reverse()
      .find(AIMessage.isInstance);

    const transferMessage = new ToolMessage({
      content: "Transferred to sales agent",
      tool_call_id: runtime.toolCallId,
    });
    return new Command({
      goto: "sales_agent",
      update: {
        activeAgent: "sales_agent",
        messages: [lastAiMessage, transferMessage].filter(Boolean),
      },
      graph: Command.PARENT,
    });
  },
  {
    name: "transfer_to_sales",
    description: "Transfer to the sales agent.",
    schema: z.object({}),
  }
);
```

#### Context engineering
* **Key Points:**
  - With subgraph handoffs, you control exactly what messages flow between agents. This precision is essential for maintaining valid conversation history and avoiding context bloat that could confuse downstream agents.
  - When handing off between agents, you need to ensure the conversation history remains valid. LLMs expect tool calls to be paired with their responses, so when using `Command.PARENT` to hand off to another agent, you must include both: The `AIMessage` containing the tool call (the message that triggered the handoff), A `ToolMessage` acknowledging the handoff (the artificial response to that tool call)
  - Without this pairing, the receiving agent will see an incomplete conversation and may produce errors or unexpected behavior.
  - Why not pass all subagent messages? While you could include the full subagent conversation in the handoff, this often creates problems. The receiving agent may become confused by irrelevant internal reasoning, and token costs increase unnecessarily.
  - By passing only the handoff pair, you keep the parent graph's context focused on high-level coordination.
  - If the receiving agent needs additional context, consider summarizing the subagent's work in the ToolMessage content instead of passing raw message history.
  - When returning control to the user (ending the agent's turn), ensure the final message is an `AIMessage`. This maintains valid conversation history and signals to the user interface that the agent has finished its work.

## Implementation considerations
* **Key Points:**
  - As you design your multi-agent system, consider:
  - **Context filtering strategy**: Will each agent receive full conversation history, filtered portions, or summaries? Different agents may need different context depending on their role.
  - **Tool semantics**: Clarify whether handoff tools only update routing state or also perform side effects. For example, should `transfer_to_sales()` also create a support ticket, or should that be a separate action?
  - **Token efficiency**: Balance context completeness against token costs. Summarization and selective context passing become more important as conversations grow longer.