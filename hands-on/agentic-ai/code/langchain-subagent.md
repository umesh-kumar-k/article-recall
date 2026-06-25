---
aliases:
  - Multi Agent Subagent workflow
Source 1: https://docs.langchain.com/oss/javascript/langchain/multi-agent/subagents
---
## Subagents
* **Key Points:**
  - In the **subagents** architecture, a central main agent (often referred to as a **supervisor**) coordinates subagents by calling them as tools.
  - The main agent decides which subagent to invoke, what input to provide, and how to combine results.
  - Subagents are stateless—they don't remember past interactions, with all conversation memory maintained by the main agent.
  - This provides context isolation: each subagent invocation works in a clean context window, preventing context bloat in the main conversation.

## Key characteristics
* **Key Points:**
  - Centralized control: All routing passes through the main agent
  - No direct user interaction: Subagents return results to the main agent, not the user (though you can use interrupts within a subagent to allow user interaction)
  - Subagents via tools: Subagents are invoked via tools
  - Parallel execution: The main agent can invoke multiple subagents in a single turn
  - A supervisor agent (this pattern) is different from a router. The supervisor is a full agent that maintains conversation context and dynamically decides which subagents to call across multiple turns. A router is typically a single classification step that dispatches to agents without maintaining ongoing conversation state.

## When to use
* **Key Points:**
  - Use the subagents pattern when you have multiple distinct domains (e.g., calendar, email, CRM, database), subagents don't need to converse directly with users, or you want centralized workflow control.
  - For simpler cases with just a few tools, use a single agent.
  - Need user interaction within a subagent? While subagents typically return results to the main agent rather than conversing directly with users, you can use interrupts within a subagent to pause execution and gather user input.
* **Technical Entities (Classes/Functions/APIs):** `interrupts`

## Basic implementation
* **Key Points:**
  - The core mechanism wraps a subagent as a tool that the main agent can call:
* **Technical Entities (Classes/Functions/APIs):** `createAgent`, `tool`, `z`, `invoke()`
* **Code Snippet:**
```typescript
import { createAgent, tool } from "langchain";
import { z } from "zod";

// Create a subagent
const subagent = createAgent({ model: "google_genai:gemini-3.5-flash", tools: [...] });

// Wrap it as a tool
const callResearchAgent = tool(
  async ({ query }) => {
    const result = await subagent.invoke({
      messages: [{ role: "user", content: query }]
    });
    return result.messages.at(-1)?.content;
  },
  {
    name: "research",
    description: "Research a topic and return findings",
    schema: z.object({ query: z.string() })
  }
);

// Main agent with subagent as a tool
const mainAgent = createAgent({ model: "google_genai:gemini-3.5-flash", tools: [callResearchAgent] });
```

## Design decisions
* **Key Points:**
  - When implementing the subagents pattern, you'll make several key design choices.
* **Technical Entities (Classes/Functions/APIs):** `Sync vs. async`, `Tool patterns`, `Subagent specs`, `Subagent inputs`, `Subagent outputs`

## Sync vs. async
* **Key Points:**
  - Subagent execution can be **synchronous** (blocking) or **asynchronous** (background).
  - Your choice depends on whether the main agent needs the result to continue.
  - By default, subagent calls are **synchronous**: the main agent waits for each subagent to complete before continuing.
  - Use sync when the main agent's next action depends on the subagent's result.
  - Use **asynchronous execution** when the subagent's work is independent—the main agent doesn't need the result to continue conversing with the user.

### Synchronous (default)
* **Key Points:**
  - By default, subagent calls are **synchronous**: the main agent waits for each subagent to complete before continuing. Use sync when the main agent's next action depends on the subagent's result.

### Asynchronous
* **Key Points:**
  - Use **asynchronous execution** when the subagent's work is independent—the main agent doesn't need the result to continue conversing with the user.
  - **Three-tool pattern:**
  - **Start job**: Kicks off the background task, returns a job ID
  - **Check status**: Returns current state (pending, running, completed, failed)
  - **Get result**: Retrieves the completed result
  - When a job finishes, your application needs to notify the user.

## Tool patterns
* **Key Points:**
  - There are two main ways to expose subagents as tools:
  - Tool per agent: Fine-grained control over each subagent's input/output
  - Single dispatch tool: Many agents, distributed teams, convention over configuration

### Tool per agent
* **Key Points:**
  - The key idea is wrapping subagents as tools that the main agent can call:
  - The main agent invokes the subagent tool when it decides the task matches the subagent's description, receives the result, and continues orchestration.
* **Technical Entities (Classes/Functions/APIs):** `createAgent`, `tool`, `z`
* **Code Snippet:**
```typescript
import { createAgent, tool } from "langchain";
import * as z from "zod";

// Create a sub-agent
const subagent = createAgent({...});

// Wrap it as a tool
const callSubagent = tool(
  async ({ query }) => {
    const result = await subagent.invoke({
      messages: [{ role: "user", content: query }]
    });
    return result.messages.at(-1)?.text;
  },
  {
    name: "subagent_name",
    description: "subagent_description",
    schema: z.object({
      query: z.string().describe("The query to send to subagent")
    })
  }
);

// Main agent with subagent as a tool
const mainAgent = createAgent({ model, tools: [callSubagent] });
```

### Single dispatch tool
* **Key Points:**
  - An alternative approach uses a single parameterized tool to invoke ephemeral sub-agents for independent tasks.
  - Unlike the tool per agent approach where each sub-agent is wrapped as a separate tool, this uses a convention-based approach with a single `task` tool: the task description is passed as a human message to the sub-agent, and the sub-agent's final message is returned as the tool result.
  - Use this approach when you want to distribute agent development across multiple teams, need to isolate complex tasks into separate context windows, need a scalable way to add new agents without modifying the coordinator, or prefer convention over customization.
  - This approach trades flexibility in context engineering for simplicity in agent composition and strong context isolation.
  - **Key characteristics:**
  - Single task tool: One parameterized tool that can invoke any registered sub-agent by name
  - Convention-based invocation: Agent selected by name, task passed as human message, final message returned as tool result
  - Team distribution: Different teams can develop and deploy agents independently
  - Agent discovery: Sub-agents can be discovered via system prompt (listing available agents) or through progressive disclosure (loading agent information on-demand via tools)
  - An interesting aspect of this approach is that sub-agents may have the exact same capabilities as the main agent. In such cases, invoking a sub-agent is **really about context isolation** as the primary reason—allowing complex, multi-step tasks to run in isolated context windows without bloating the main agent's conversation history.
* **Technical Entities (Classes/Functions/APIs):** `tool`, `createAgent`, `SUBAGENTS`
* **Code Snippet:**
```typescript
const task = tool(
  async ({ agentName, description }) => {
    const agent = SUBAGENTS[agentName];
    const result = await agent.invoke({
      messages: [
        { role: "user", content: description }
      ],
    });
    return result.messages.at(-1)?.content;
  },
  {
    name: "task",
    description: `Launch an ephemeral subagent.

Available agents:
- research: Research and fact-finding
- writer: Content creation and editing`,
    schema: z.object({
      agentName: z
        .string()
        .describe("Name of agent to invoke"),
      description: z
        .string()
        .describe("Task description"),
    }),
  }
);
```

## Context engineering
* **Key Points:**
  - Control how context flows between the main agent and its subagents:
  - **Subagent specs**: Ensure subagents are invoked when they should be. Impacts: Main agent routing decisions
  - **Subagent inputs**: Ensure subagents can execute well with optimized context. Impacts: Subagent performance
  - **Subagent outputs**: Ensure the supervisor can act on subagent results. Impacts: Main agent performance

### Subagent specs
* **Key Points:**
  - The **names** and **descriptions** associated with subagents are the primary way the main agent knows which subagents to invoke. These are prompting levers—choose them carefully.
  - **Name**: How the main agent refers to the sub-agent. Keep it clear and action-oriented (e.g., `research_agent`, `code_reviewer`).
  - **Description**: What the main agent knows about the sub-agent's capabilities. Be specific about what tasks it handles and when to use it.
  - For the single dispatch tool design, you must additionally provide the main agent with information about the subagents it can invoke. You can provide this information in different ways based on the number of agents and whether your registry is static or dynamic:
  - **System prompt enumeration**: Small, static agent lists (< 10 agents). Simple, but requires prompt updates when agents change
  - **Enum constraint**: Small, static agent lists (< 10 agents). Type-safe and explicit, but requires code changes when agents change
  - **Tool-based discovery**: Large or dynamic agent registries. Flexible and scalable, but adds complexity
* **Technical Entities (Classes/Functions/APIs):** `task`, `list_agents`, `search_agents`, `Enum`, `AgentName`

### Subagent inputs
* **Key Points:**
  - Customize what context the subagent receives to execute its task.
  - Add input that isn't practical to capture in a static prompt—full message history, prior results, or task metadata—by pulling from the agent's state.
* **Technical Entities (Classes/Functions/APIs):** `AgentState`, `tool`, `Command`, `ToolMessage`
* **Code Snippet:**
```typescript
import { createAgent, tool, AgentState, ToolMessage } from "langchain";
import { Command } from "@langchain/langgraph";
import * as z from "zod";

// Example of passing the full conversation history to the sub agent via the state.
const callSubagent1 = tool(
  async ({query}) => {
    const state = getCurrentTaskInput<AgentState>();
    // Apply any logic needed to transform the messages into a suitable input
    const subAgentInput = someLogic(query, state.messages);
    const result = await subagent1.invoke({
      messages: subAgentInput,
      // You could also pass other state keys here as needed.
      // Make sure to define these in both the main and subagent's
      // state schemas.
      exampleStateKey: state.exampleStateKey
    });
    return result.messages.at(-1)?.content;
  },
  {
    name: "subagent1_name",
    description: "subagent1_description",
  }
);
```

### Subagent outputs
* **Key Points:**
  - Customize what the main agent receives back so it can make good decisions. Two strategies:
  - **Prompt the sub-agent**: Specify exactly what should be returned. A common failure mode is that the sub-agent performs tool calls or reasoning but doesn't include results in its final message—remind it that the supervisor only sees the final output.
  - **Format in code**: Adjust or enrich the response before returning it. For example, pass specific state keys back in addition to the final text using a Command.
* **Technical Entities (Classes/Functions/APIs):** `Command`, `ToolMessage`, `tool_call_id`, `config.toolCall?.id`
* **Code Snippet:**
```typescript
import { tool, ToolMessage } from "langchain";
import { Command } from "@langchain/langgraph";
import * as z from "zod";

const callSubagent1 = tool(
  async ({ query }, config) => {
    const result = await subagent1.invoke({
      messages: [{ role: "user", content: query }]
    });

    // Return a Command to update multiple state keys
    return new Command({
      update: {
        // Pass back additional state from the subagent
        exampleStateKey: result.exampleStateKey,
        messages: [
          new ToolMessage({
            content: result.messages.at(-1)?.text,
            tool_call_id: config.toolCall?.id!
          })
        ]
      }
    });
  },
  {
    name: "subagent1_name",
    description: "subagent1_description",
    schema: z.object({
      query: z.string().describe("The query to send to subagent1")
    })
  }
);
```

## Checkpointing and state inspection
* **Key Points:**
  - By default, subagents use the **inherited checkpointer** mode—each invocation starts with fresh state, supports interrupts, and runs safely in parallel.
  - If you need a subagent to maintain its own persistent conversation history across invocations, compile it with `checkpointer=True` (continuations mode).
  - Because subagents are called inside tool functions, LangGraph cannot statically discover them. This means `get_state` with `subgraphs` will not return subagent state.
  - If you need to read nested graph state (e.g., during an interrupt), invoke the subagent from a node function in a custom graph instead.
* **Technical Entities (Classes/Functions/APIs):** `checkpointer`, `get_state`, `subgraphs`, `interrupt`