---
aliases:
  - Agent Execution Sandbox
Source 1: https://www.langchain.com/blog/give-your-ai-agent-its-own-computer
Source 2:
Source 3:
Source 4:
---
## Give your agent its own computer
* **Key Points:**
  - LLMs can reason. But reasoning alone doesn't get much done.
  - Running code execution in an AI agent is harder than it looks. Your agent needs a real computer (filesystem, shell, package manager, persistent state) but handing it access to your infrastructure is dangerous.
  - Think about it this way: you use one laptop. You are n of one. But agents are going to run millions of tasks, and each one needs its own computer to work from.
  - Satya Nadella put it plainly: "Every agent needs a computer."

## What becomes possible when an agent has a computer
* **Key Points:**
  - Think about what Cursor, Claude Code, or ChatGPT's code interpreter can do that a plain chat interface can't. They don't just answer questions: they run the code, see the error, fix it, run it again, and hand you something that works. That feedback loop is what makes them useful.
  - That same loop is what separates a demo agent from a production agent. Once your agent can execute, a whole category of work opens up:
  - A coding assistant that doesn't just suggest a fix: it applies the fix, runs your tests, and confirms nothing broke
  - A data analyst that pulls a CSV, runs Python against it, and hands you a formatted report
  - A CI agent that clones your repo, installs dependencies, runs the full test suite, and opens a PR (like OpenSWE)
  - A research agent that browses, scrapes, synthesizes, and writes — not just searches
  - A content pipeline that generates, renders, and exports finished artifacts
  - An RL or eval harness that needs to spin up environments in parallel, run episodes at burst scale, and tear them down immediately — zero to thousands of sandboxes, then back to zero
  - The common thread: these agents need more than a token stream. They need a place to work.

## Why you can't just hand your agent your laptop
* **Key Points:**
  - The obvious next question is: why not just let the agent run code locally? Or in a Docker container? Teams do this in early prototypes. It stops working in production for two reasons.
  - First: agents run untrusted code by definition.
  - The code your agent executes might come from a model, a user prompt, a cloned repo, or an installed package. You didn't write it. You can't fully vet it.
  - In September 2025, a self-replicating npm worm called Shai-Hulud backdoored 500+ packages — code that executed in preinstall before any validation could run. A second wave in November hit 796 more packages and 25,000+ GitHub repos in hours. An agent that installs npm packages as part of its workflow is exposed to exactly this.
  - Second: containers aren't enough.
  - The common instinct is "just run it in Docker." Containers are great for isolating known, vetted application code (i.e. a web server, a background job). They're not designed for an agent that's installing arbitrary dependencies, running model-generated scripts, and persisting state across a long-running session.
  - And critically: containers share a kernel with the host. A kernel exploit reaches through them. Copy Fail (CVE-2026-31431) is a 732-byte Python script that roots every major Linux distribution back to 2017 via the kernel crypto API. AI tooling found it in about an hour.
  - A container boundary is not an isolation boundary. For untrusted, model-generated code, you need hardware-level separation.

## LangSmith Sandboxes: a computer for every agent
* **Key Points:**
  - The mental model that helps here: a sandbox needs to be two things at once. It needs the instant startup of a serverless function because you can't make an agent wait two minutes for a VM to boot. And it needs the statefulness of a full machine because agents aren't stateless request-handlers; they're mid-session workers that install dependencies, edit files, and pick up where they left off.
  - LangSmith Sandboxes are built for that model. Each one is a hardware-virtualized microVM. Not a container, a full machine with its own kernel.
  - It can install packages, run scripts, edit files, spin up a local server, and keep working across a long session — all without touching your production infrastructure or any other agent's sandbox. When the work is done, the sandbox disappears.
  - You access it through the same LangSmith SDK and API key you already use:
  - It just takes one call, and your agent has a computer.
  - There's also a less obvious benefit for teams running GPU workloads: when your sandbox spins up instantly, your GPU doesn't idle waiting for CPU compute to provision. Fast sandboxes are a GPU efficiency multiplier — a detail that compounds quickly at scale.
* **Technical Entities (Classes/Functions/APIs):** `Client`, `client.create_sandbox()`, `sandbox.run()`
* **Code Snippet:**
```python
from langsmith import Client

client = Client()
sandbox = client.create_sandbox()

# Give the agent a shell
result = sandbox.run("pip install pandas && python analysis.py")
print(result.stdout)
```

## What you get beyond basic execution
* **Key Points:**
  - A sandbox is more than a place to run code. The GA release ships a set of primitives that make agent workflows production-ready:
  - Snapshots and forks: Capture a sandbox mid-session and boot new ones from it. Forks use copy-on-write, so spinning up ten parallel branches costs roughly the same as one. When your agent goes down the wrong path, restore and try again, without rebuilding from scratch.
  - Blueprints for pre-warmed environments: Define a base image (your repo cloned, your deps installed, your config in place) and boot sandboxes from it in seconds instead of minutes.
  - Service URLs: If the agent starts a local web server — say, to preview a generated report — you get an authenticated URL you can open in a browser or share with a teammate. No port forwarding.
  - Auth proxy: Outbound requests from the sandbox flow through a proxy that injects credentials at the network layer. Secrets never touch the agent runtime.
  - Creator-private by default: Only the user who launched a sandbox (and workspace admins) can access it. Share when you're ready.

## When to reach for Sandboxes
* **Key Points:**
  - Sandboxes are the right layer when your agent needs to do something, not just say something. Concretely:
  - Your agent generates code and you want it to verify that code runs before responding
  - You're building a coding assistant, CI agent, or data pipeline that operates on real files
  - You're running multi-step workflows where state needs to persist across tool calls
  - You need burst capacity (i.e. thousands of parallel environments for RL training or evals) that has to scale from zero in seconds
  - You're accepting any user-supplied input that could end up being executed
  - Sandboxes are overkill if your agent only calls APIs with fixed schemas and never executes dynamic code. A retrieval agent that searches docs and returns citations doesn't need one. An agent that writes and runs a Python script does.

## How teams are using this today
* **Key Points:**
  - At monday.com, Sandboxes power their Sidekick AI assistant, giving it a secure environment to write and run code for advanced user workflows, including data analysis and multimedia generation.
  - "LangSmith Sandboxes are helping us make our Sidekick much more capable for monday.com users. With secure environments, Sidekick can write and run code, and use the results to create richer workflows, like running data analysis and generating multimedia." — Omri Bruchim, AI Platform Group Manager, monday.com

## The shift worth paying attention to
* **Key Points:**
  - For the last few years, making an agent more capable meant giving it better tools: a search API, a calculator, a database connector. That's still true. But the ceiling on what predefined tools can do is low.
  - The agents that will actually replace workflows (not just assist with them!) are the ones that can pick up whatever tool they need, run it, see what happened, and adapt. That's what having a computer makes possible. It's not an infrastructure detail. It's the difference between an agent that can think and an agent that can act.
  - You use one laptop. Your agents will each need their own. LangSmith Sandboxes are how you give them one.

---

# The two patterns by which agents connect sandboxes

* **Key Points:**
  - An increasing number of agents need a workspace - a computer where they can run code, install packages, and access files. That workspace needs to be isolated so the agent can't access your credentials, files, or network.
  - Sandboxes provide this isolation by creating a boundary between the agent's environment and your host system.
  - The question teams building these agents face isn't whether to use sandboxes - it's how to integrate them with their agent architecture.
  - There are two common patterns based on where the agent runs: inside the sandbox or outside of it. Each pattern has different benefits and trade-offs.
  - Note: this post focuses on sandboxes that give agents a full 'computer' - complete execution environments like Docker containers or VMs. We won't cover process-level sandboxes (like bubblewrap) or language-level sandboxes (like Pyodide).

## Pattern 1: Agent Runs IN Sandbox
* **Key Points:**
  - In this pattern, the agent runs inside the sandbox. You communicate with it over the network.
  - What this looks like in practice: You build a Docker or VM image with your agent framework pre-installed, run it inside the sandbox, and connect from outside to send messages. The agent exposes an API endpoint (typically HTTP or WebSocket), and your application communicates with it across the sandbox boundary.
  - Benefits: This pattern mirrors local development closely—if you run deepagents in your terminal locally, you run the same command in the sandbox. The agent has direct filesystem access and can modify its environment. This is useful when the agent and execution environment are tightly coupled, such as when the agent needs to interact with specific libraries or maintain complex environment state.
  - Trade-offs: Communication across the sandbox boundary requires infrastructure. Some providers handle this in their SDK—for example, agents like OpenCode run a server inside the sandbox, and providers like E2B can expose this through a clean API. If your provider doesn't offer this, you'll need to build the WebSocket or HTTP layer yourself, including session management and error handling.
  - API keys must live inside the sandbox to allow the agent to make inference calls. This creates a potential security risk if the sandbox is compromised, whether through a vulnerability in the isolation technology or through prompt injection attacks that exfiltrate credentials. Note: we see providers like E2B and Runloop working on secret vault capabilities, which addresses this.
  - Updates require rebuilding the container image and redeploying, which can slow iteration cycles during development.
  - Another downside is that the sandbox must be resumed before the agent becomes active, which often requires extra logic.
  - For those worried about protecting the IP of their agents, if your agent is running in the sandbox it becomes much easier to exfiltrate the entire code and prompts of the agent.
  - Nuno Campos from Witan Labs also points out another security risk: "I'd say another downside of agent in sandbox is that effectively no part of your agent can have more privileges than the bash tool does. E.g. imagine you want an agent that has a bash tool and a tool that can do web search or web fetch, then all the LLM generated code can do unlimited web fetches (which is a big security risk). If it's sandbox as tool then you can have tools with more permissions than you give to llm generated code (which sounds very useful for many agents) trivially, as the security boundary is around the bash tool, not the whole agent."

## Pattern 2: Sandbox as Tool
* **Key Points:**
  - In this pattern, the agent runs on your machine or server. When it needs to execute code, it calls a remote sandbox via API.
  - What this looks like in practice: Your agent runs locally (or on your server), and when it generates code that needs to execute, it calls out to a sandbox provider's API (like E2B, Modal, Daytona, or Runloop). The provider's SDK handles all the communication details. From your agent's perspective, the sandbox is just another tool.
  - Benefits: You can update agent code instantly without rebuilding container images, which speeds up iteration during development. API keys stay outside the sandbox—only execution happens in isolation. This provides cleaner separation of concerns: agent state (conversation history, reasoning chains, memory) lives where your agent runs, separate from the sandbox. This means sandbox failures don't lose your agent's state, and you can switch sandbox backends without affecting your agent's core logic.
  - Two other benefits of this option, as pointed out by Tomas Beran of E2B: Having the option to run tasks in multiple remote sandboxes in parallel, Paying for sandboxes only when executing code, rather than for the whole process runtime.
  - Ben Guo adds a final point about the benefits of separating agent runtime from sandbox runtime: "We chose Pattern 2 for the reasons you mention, but also in preparation for a future where it makes sense to run the agent harness in a GPU machine – generally feels like the environment requirements will diverge between the persistent sandbox and the inference harness"
  - Trade-offs: Network latency is the main downside. Each execution call crosses the network boundary. For workloads with many small executions, this can add up.
  - Many sandbox providers offer stateful sessions where variables, files, and installed packages persist across invocations within the same session. This can mitigate some of the latency concerns by reducing the number of round trips needed.

## Choosing Between Patterns
* **Key Points:**
  - Choose Pattern 1 when: The agent and execution environment are tightly coupled (for example, the agent needs persistent access to specific libraries or complex environment state), You want production to mirror local development closely, Your provider's SDK handles the communication layer for you
  - Choose Pattern 2 when: You need to iterate quickly on agent logic during development, You want to keep API keys outside the sandbox, You prefer cleaner separation between agent state and execution environment

## Implementation Example
* **Key Points:**
  - To make these patterns concrete, we'll show examples using deepagents, an open-source agent framework with built-in sandbox support. Similar patterns apply to other agent frameworks.
* **Technical Entities (Classes/Functions/APIs):** `Daytona`, `ChatAnthropic`, `create_deep_agent`, `DaytonaSandbox`

### Pattern 1: Agent IN Sandbox
* **Key Points:**
  - For Pattern 1, first you build an image with your agent pre-installed: Then run it inside the sandbox. A complete implementation requires additional infrastructure to handle communication between your application and the agent inside the sandbox (WebSocket or HTTP server, session management, error handling). This is beyond the scope of this post, but we will have some follow up posts diving into this in more detail.
* **Code Snippet:**
```dockerfile
FROM python:3.11
RUN pip install deepagents-cli
```

### Pattern 2: Sandbox as Tool
* **Technical Entities (Classes/Functions/APIs):** `Daytona`, `ChatAnthropic`, `create_deep_agent`, `DaytonaSandbox`, `agent.invoke()`, `sandbox.stop()`
* **Code Snippet:**
```python
from daytona import Daytona
from langchain_anthropic import ChatAnthropic

from deepagents import create_deep_agent
from langchain_daytona import DaytonaSandbox

# Can also do this with E2B, Runloop, Modal
sandbox = Daytona().create()
backend = DaytonaSandbox(sandbox=sandbox)

agent = create_deep_agent(
    model=ChatAnthropic(model="claude-sonnet-4-20250514"),
    system_prompt="You are a Python coding assistant with sandbox access.",
    backend=backend,
)

result = agent.invoke(
    {
        "messages": [
            {
                "role": "user",
                "content": "Run a small python script",
            }
        ]
    }
)

sandbox.stop()
```
* **Key Points:**
  - Here's what happens when this code runs: The agent plans locally on your machine, It generates Python code to solve the problem, It calls the Runloop API, which executes the code in a remote sandbox, The sandbox returns the result, The agent sees the output and continues reasoning locally

## Conclusion
* **Key Points:**
  - Agents need to execute code in isolated environments for security. There are two architecture patterns: running the agent inside the sandbox (mirrors local development, tight coupling) or running it outside with the sandbox as a tool (easy updates, API keys stay secure). Each has different benefits and trade-offs depending on your needs.
  - deepagents supports both patterns with simple configuration. Try it out to see which pattern works best for your use case.

---

## How to Build a Custom Agent Harness
* **Key Points:**
  - Building useful agents is largely about customization: connecting your agent to the right context, data, and environment(s) for the task at hand.
  - At its core, an agent is a model calling tools in a loop until it completes a task and returns a result.
  - You can also define an agent as: agent = model + harness
  - The harness is the scaffolding around the model that connects it to the real world.
  - The remainder of this post assumes the following: An agent is only as good as the context provided to the model, The job of a harness is to provide context to the model at every step
  - So, to build a useful agent, you need a harness that's great at delivering the right context for the given task to the model.

## The base harness
* **Key Points:**
  - create_agent is LangChain's primitive for building a harness. Pass in a model, tools, and a system prompt, and you have a working agent.
  - Harnesses like Deep Agents and the Claude Agent SDK come pre-assembled with an opinionated middleware (explained below) stack: memory, context management, sandboxing, and more. They're designed to get you to a production-ready agent fast, and they work well for most cases. But many agents need finer grained customization than these harnesses support: custom prompting, business logic, guardrails, etc.
  - create_agent takes a different approach: it's purposefully minimalistic. Our philosophy is similar to that of Pi, a highly configurable coding agent harness. create_agent just implements the core agent loop, and it exposes middleware as a primitive for customization.
* **Technical Entities (Classes/Functions/APIs):** `create_agent`, `model`, `tools`, `system_prompt`

## Middleware: how you customize the harness

* **Key Points:**
  - Middleware hooks into the agent loop at each step: before and after model calls, before and after tool calls, at agent startup and teardown. Each piece handles one concern and composes freely with any other.
  - Middleware allows you to add capabilities to your agent via a few levers that often work together:
  - Deterministic Logic. Business logic, policy enforcement, dynamic agent control — anything that needs to fire at a specific point in the loop. This includes runtime control over the agent itself: swapping the model based on task complexity, adjusting the prompt, and updating the agent's message history (during compaction, for example). The right place for anything that can't (or shouldn't) live in a prompt.
  - Tools. Rather than registering tools directly on the agent, middleware can handle the full lifecycle — setup, teardown, registration — and hand the agent a clean set of tools to work with. This matters when tools have dependencies, require initialization, or need to be torn down cleanly at the end of a run. It also keeps tool configuration close to the logic that governs it, rather than scattered across the agent definition.
  - Custom state. If your middleware needs to track state across hooks, middleware can extend the agent's state with custom properties. This enables middleware to track state throughout execution (maintain counters, flags, or other values that persist throughout agent runs) and share data between hooks.
  - Stream handlers. Middleware can intercept and transform the agent's output stream — filtering events, injecting metadata, routing different event types to different consumers. Useful when different parts of your stack need to react to different things the agent does: a UI consuming token deltas, an audit log capturing tool calls, a monitoring system tracking latency.
  - The beauty of middleware is that it: Enables customization at any point in the agent loop, Bundles related logic in composable, sharable units of code
  - LangChain ships prebuilt middleware for the most common patterns. Anything bespoke to your use case is one custom middleware away. Because each piece is isolated, the same middleware can be reused across every agent in an organization so that new agents inherit battle-tested behavior without rebuilding it.

## Harness capabilities
* **Key Points:**
  - The job of a harness is to get the model the right context at the right time for the given task.
  - The table below maps common capabilities to middleware that support them. Most production agents end up using several together, depending on the agent's needs (is it long running? how complex are the tasks? how sensitive are the agent's actions?, etc):
  - Prevent context overflow: Long-running sessions accumulate message history fast. Without intervention, it overflows the context window. Middleware: SummarizationMiddleware, ContextEditingMiddleware
  - Access and update memory: Load relevant knowledge at startup, write it back at the end of a run. Lets the agent improve over time from real usage. Middleware: FilesystemMiddleware, MemoryMiddleware, SkillsMiddleware
  - Take actions in an environment: A fixed toolset limits what an agent can do. Access to a filesystem and execution environment unlocks more creative solutions, often with greater token efficiency. Middleware: ShellToolMiddleware, FilesystemMiddleware, CodeInterpreterMiddleware
  - Delegate tasks: Subagents handle complex sub-tasks with clean context windows. A todo list tracks progress across a long run. Middleware: SubAgentMiddleware, AsyncSubAgentMiddleware, TodoListMiddleware
  - Handle transient failures: Models and tools fail unpredictably. Production agents need retry logic with backoff and fallbacks when a model is unavailable. Middleware: ToolRetryMiddleware, ModelRetryMiddleware, ModelFallbackMiddleware
  - Enforce policies: PII handling, compliance checks, approval gates — these need to fire on every call regardless of what the model does. They don't belong in a prompt. Middleware: PIIMiddleware, HumanInTheLoopMiddleware
  - Steer the agent: Full autonomy isn't always appropriate. Pause before consequential actions and wait for a human to approve, reject, or redirect. Middleware: HumanInTheLoopMiddleware
  - Control costs: Prompt caching reduces token spend on long-running tasks. Call limits prevent costs from accumulating unchecked. Middleware: ModelCallLimitMiddleware, ToolCallLimitMiddleware, PromptCachingMiddleware
* **Technical Entities (Classes/Functions/APIs):** `SummarizationMiddleware`, `ContextEditingMiddleware`, `FilesystemMiddleware`, `MemoryMiddleware`, `SkillsMiddleware`, `ShellToolMiddleware`, `CodeInterpreterMiddleware`, `SubAgentMiddleware`, `AsyncSubAgentMiddleware`, `TodoListMiddleware`, `ToolRetryMiddleware`, `ModelRetryMiddleware`, `ModelFallbackMiddleware`, `PIIMiddleware`, `HumanInTheLoopMiddleware`, `ModelCallLimitMiddleware`, `ToolCallLimitMiddleware`, `PromptCachingMiddleware`

## Task-harness fit
* **Key Points:**
  - Task-harness fit is how well your harness matches the actual demands of the task: the context it needs, the failures it'll encounter, the policies it must enforce, the environment it operates in. A harness for a customer service agent looks very different from one built for a long-running coding agent.
  - Every agent we build at LangChain, including our GTM agent, asynchronous coding agent, and our no-code agent builder, is built on create_agent with a middleware stack tailored to that agent's mission.
  - The best agents aren't just built with capable models, they're built with harnesses that tightly fit the task. The easiest way to build a custom harness is with create_agent.