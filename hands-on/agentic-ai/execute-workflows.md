---
aliases:
  - Execute Workflows
Source 1: https://docs.langchain.com/oss/python/langchain/multi-agent
Source 2:
---
[SubAgent Workflow Code Example](./code/langchain-subagent.md) 

[Handoff Workflow Code Example](./code/langchain-handoffs.md) 

[Skills Workflow Code Example](./code/langchain-skills.md) 

[Router Workflow Code Example](./code/langchain-router.md) 

[Custom Workflow Code Example](./code/langchain-custom.md) 

[Langgraph Workflows Code Example](./code/langgraph-workflow.md) 

## Multi-agent
* **Key Points:**
  - Multi-agent systems coordinate specialized components to tackle complex workflows.
  - However, not every complex task requires this approach—a single agent with the right (sometimes dynamic) tools and prompt can often achieve similar results.
  - When developers say they need "multi-agent," they're usually looking for one or more of these capabilities:
  - **Context management**: Provide specialized knowledge without overwhelming the model's context window.
  - **Distributed development**: Allow different teams to develop and maintain capabilities independently, composing them into a larger system with clear boundaries.
  - **Parallelization**: Spawn specialized workers for subtasks and execute them concurrently for faster results.
  - Multi-agent patterns are particularly valuable when a single agent has too many tools and makes poor decisions about which to use, when tasks require specialized knowledge with extensive context (long prompts and domain-specific tools), or when you need to enforce sequential constraints that unlock capabilities only after certain conditions are met.
  - At the center of multi-agent design is context engineering—deciding what information each agent sees.
* **Technical Entities (Classes/Functions/APIs):** `Deep Agents`, `subagents`, `skills`

## Patterns
* **Key Points:**
  - Here are the main patterns for building multi-agent systems, each suited to different use cases:
  - **Subagents**: A main agent coordinates subagents as tools. All routing passes through the main agent, which decides when and how to invoke each subagent.
  - **Handoffs**: Behavior changes dynamically based on state. Tool calls update a state variable that triggers routing or configuration changes, switching agents or adjusting the current agent's tools and prompt.
  - **Skills**: Specialized prompts and knowledge loaded on-demand. A single agent stays in control while loading context from skills as needed.
  - **Router**: A routing step classifies input and directs it to one or more specialized agents. Results are synthesized into a combined response.
  - **Custom workflow**: Build bespoke execution flows with LangGraph, mixing deterministic logic and agentic behavior.

### Choosing a pattern
* **Key Points:**
  - **Distributed development**: Can different teams maintain components independently?
  - **Parallelization**: Can multiple agents execute concurrently?
  - **Multi-hop**: Does the pattern support calling multiple subagents in series?
  - **Direct user interaction**: Can subagents converse directly with the user?
  - You can mix patterns!
* **Technical Entities (Classes/Functions/APIs):** `Subagents`, `Handoffs`, `Skills`, `Router`, `Custom workflow`, `LangGraph`

## Performance comparison
* **Key Points:**
  - Different patterns have different performance characteristics.
  - Understanding these tradeoffs helps you choose the right pattern for your latency and cost requirements.
  - **Model calls**: Number of LLM invocations. More calls = higher latency (especially if sequential) and higher per-request API costs.
  - **Tokens processed**: Total context window usage across all calls. More tokens = higher processing costs and potential context limits.

### One-shot request
* **Key Points:**
  - Subagents are **stateless by design**—each invocation follows the same flow
  - The main agent maintains conversation context, but subagents start fresh each time
  - This provides strong context isolation but repeats the full flow
  - The coffee agent is **still active** from turn 1 (state persists)
  - No handoff needed—agent directly calls `buy_coffee` tool (call 1)
  - Agent responds to user (call 2)
  - **Saves 1 call by skipping the handoff**
  - The skill context is **already loaded** in conversation history
  - No need to reload—agent directly calls `buy_coffee` tool (call 1)
  - Agent responds to user (call 2)
  - **Saves 1 call by reusing loaded skill**
  - Routers are **stateless**—each request requires an LLM routing call
  - Can be optimized by wrapping as a tool in a stateful agent

### Multi-domain
* **Key Points:**
  - Each subagent works in **isolation** with only its relevant context. Total: **9K tokens**.
  - Handoffs executes **sequentially**—can't research all three languages in parallel. Growing conversation history adds overhead. Total: **\~14K+ tokens**.
  - After loading, **every subsequent call processes all 6K tokens of skill documentation**. Subagents processes 67% fewer tokens overall due to context isolation. Total: **15K tokens**.
  - Router uses an **LLM for routing**, then invokes agents in parallel. Similar to Subagents but with explicit routing step. Total: **9K tokens**.

### Summary
* **Key Points:**
  - Choosing a pattern:
  - Single requests: Handoffs, Skills, Router
  - Repeat requests: Handoffs, Skills
  - Parallel execution: Subagents, Router
  - Large-context domains: Subagents, Router
  - Simple, focused tasks: Skills