---
aliases:
  - Execution Runtime
highlights: Sandboxed environment(containers, VMs , serverless) for running agent code & tool invocations with resource limits
Sour: https://blaxel.ai/blog/ai-agent-runtime-environment
Source 2: https://www.langchain.com/blog/runtime-behind-production-deep-agents
---
# What is an AI agent runtime environment?

* **Key Points:**
  - The AI agent runtime gap shows up the same way every time. An agent works in a demo. It parses documents, generates code, and calls APIs. Then it hits production and breaks. Not because the model failed, but because the execution layer underneath wasn't built for stateful, tool-heavy, long-running workloads.
  - This pattern is widespread. Over 80% of AI projects fail, twice the rate of traditional IT projects. RAND identifies infrastructure investment as a key lever for improving success rates. Gartner projects that over 40% of agentic AI projects will be canceled by end of 2027. The cause isn't model quality. It's the "real cost and complexity" of deploying agents in production: reliability, governance, cost control, and operational infrastructure.
  - Making an LLM API call takes a few lines of code. Running a tool-heavy agent that persists state, coordinates tool calls, and recovers from mid-task failures is a different discipline. So is satisfying compliance requirements for autonomous actions.
  - That discipline has a name: the AI agent runtime environment. It's the execution infrastructure that manages AI agents in production. The runtime handles state, tools, memory, security, and lifecycle between model calls.
  - This article covers what an AI agent runtime does, how it differs from frameworks and model serving, what components matter for production, and how to evaluate one for enterprise-style requirements.
* **Technical Entities (Classes/Functions/APIs):** `LLM API`
* **Code Snippet:** None.

---

## What is an AI agent runtime environment?

* **Key Points:**
  - An AI agent runtime environment is the execution infrastructure that hosts AI agents in production as a stateful control loop. The runtime repeatedly invokes a stateless reasoning core (the LLM) and interprets its outputs. It executes tool calls, persists memory across sessions, and enforces security boundaries.
  - The result: stateless model inference becomes stateful, autonomous action. Palantir outlines security and governance dimensions for agentic runtimes, and IBM offers its own framing of runtime security.
  - Traditional application runtimes handle short-lived, stateless requests. A web server receives a request, processes it, and returns a response. Each request is independent. Agent workloads break this model. Sessions run for minutes or hours.
  - Most platforms that claim to host agents are actually hosting applications. They deploy an agent as a long-running process — always-on, stateless by default, scaled like a web service. Context, state, and session management get bolted on by the developer. The result is functional but brittle: a hand-rolled state layer on top of infrastructure that was never designed for it.
  - The timeout problem illustrates why this matters. Vercel caps execution at a few minutes. Lambda hard-limits at fifteen. These limits exist because the underlying model is synchronous HTTP — you maintain a connection, and the infrastructure must hold that connection open for the duration. Raising the timeout is a bandage. Perpetual execution environments break the constraint entirely. The agent runs asynchronously inside a sandbox that remains independently of any client connection. Send an input, disconnect, come back hours later, reconnect, and pick up where the agent left off. The sandbox doesn't need your connection to keep working.
  - A purpose-built agent runtime inverts the application hosting model. The atomic unit isn't a deployment — it's a session. Each user interaction gets its own isolated compute, its own state, and its own lifecycle.
  - Different agents define sessions differently: a coding agent's session is a repository plus a conversation, a document processing agent's session is a pipeline plus approval state. The runtime provides I/O primitives — storage, networking, lifecycle management — that make session management easy without forcing a single model.
  - Think of it as the I/O layer of an operating system rebuilt for agentic workloads: it manages resources and enforces boundaries, enforces permissions, and handles scheduling, but the agent decides what runs inside. Dozens of tool calls execute, context persists across turns, and the agent decides its own next step. The core unit of compute is no longer a single invocation. It's a full session.
  - The runtime retrieves context from memory stores and calls the model. The model returns a plan with tool calls. Those tools get invoked: querying a database, calling an API, or executing code. Results flow back, state updates, and the model receives everything for the next iteration. This loop repeats until the task completes. The runtime manages everything between model calls, turning raw reasoning into completed work.
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `Vercel`, `Lambda`, `API`
* **Code Snippet:** None.

---

## Agent runtime vs. agent framework vs. model serving

* **Key Points:**
  - These three layers occupy different positions in the stack. They have different owners, concerns, and failure modes. Conflating them leads to decisions that work in development but collapse in production.
  - How an agent runtime differs from an agent framework: An agent framework is a developer-facing library for building agents. Frameworks like LangGraph, CrewAI, and the Vercel AI SDK define planning logic, tool bindings, multi-agent coordination, and workflow state machines. Engineers use them to express agent behavior in code. The agent runtime is the production execution environment where those agents run. Session isolation, state persistence across failures, scaling, authentication, and observability all live here. These capabilities work independently of which framework built the agent. A clean architectural principle applies: infrastructure problems belong to cloud primitives, while reasoning problems belong to agent frameworks. The framework defines what the agent does. The runtime determines whether it runs safely under production load.
  - How an agent runtime differs from model serving: Model serving focuses on hosting foundation models as callable API endpoints. Primary concerns are GPU utilization, request routing, inference latency, and throughput. Each request is stateless. The model serving layer has no concept of agent sessions, workflow continuity, or inter-step state. The agent runtime orchestrates everything around those model calls. Sessions last minutes or hours. Tool invocations route to external systems. State persists so a crashed agent can resume without losing all progress. Authorization policies govern actions through fine-grained, context-aware checks. The model provides intelligence. The runtime provides operational control.
* **Technical Entities (Classes/Functions/APIs):** `LangGraph`, `CrewAI`, `Vercel AI SDK`, `API`, `Vertex AI`, `Amazon Bedrock FMs`, `Azure OpenAI Service`, `Google ADK`, `OpenAI Agents SDK`, `Amazon Bedrock AgentCore Runtime`, `Vertex AI Agent Engine`
* **Code Snippet:** None.

---

## Core components of an AI agent runtime

* **Key Points:**
  - Each component addresses a specific runtime responsibility. Some agents need all of them. Others need a subset. The architecture decisions depend on what the agent does. Executing untrusted code requires a different infrastructure than routing support tickets. These are the building blocks platform teams evaluate when choosing or building a runtime layer.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Orchestration and workflow engine

* **Key Points:**
  - The orchestration engine coordinates the agent loop: input arrives, context is retrieved, the model gets called, tools are invoked, state updates, and a continue-or-stop decision is made. For a single-turn agent, this is straightforward. Multi-step tasks that branch or fan out across sub-tasks add complexity. Human approval at decision points adds another dimension.
  - Long-running workflows create specific challenges. An agent processing a code review might clone a repository, run tests across multiple configurations, analyze results, and generate a summary. Each step depends on the previous one.
  - The orchestration engine tracks sequence, handles parallelism, and enforces policy decisions like when to escalate for human approval. For workloads that split large jobs into asynchronous background tasks, batch execution infrastructure becomes part of the runtime picture.
  - Frameworks define the workflow logic. The runtime executes it reliably. A framework specifies "run these three tools in parallel, then merge results." The orchestration engine provisions the parallel execution, handles timeouts, and ensures the merge happens correctly under partial failure.
  - Multi-agent handoffs introduce a concurrency problem that most runtime discussions ignore entirely. When Agent A finishes a sub-task and needs to transfer control to Agent B (along with the session state) the transfer must be atomic. A release-then-reacquire pattern (unlock the session, then have the next agent lock it) creates a race condition window. At agentic loop speeds, that window is wide enough for a third agent or a duplicate request to slip in and corrupt state. Production runtimes need locking primitives purpose-built for agent handoffs: borrow (temporary transfer), handover (permanent transfer), both executing atomically without releasing the lock in between. This is not an edge case. Any multi-agent system where agents delegate sub-tasks to other agents will hit this within the first week of production traffic.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## State and memory management

* **Key Points:**
  - Agents need memory across two horizons. Short-term working memory holds the current task context: what the agent has done, intermediate results, and the current plan. Long-term memory stores information across sessions: user preferences, organizational knowledge, and semantic embeddings in vector databases.
  - The harder design problem isn't what to store, it's making state transparent to the agent developer. In most current architectures, the developer has to wire up persistence manually: choose a storage backend, configure mounts, handle serialization, and write recovery logic. Every framework-runtime integration becomes a custom state management project.
  - A well-designed agent runtime makes state a platform primitive, not a developer responsibility. Each session should have a private workspace that is auto-mounted at startup. The agent writes files, SQLite databases, intermediate results, whatever it needs. That workspace persists transparently across sandbox replacements and standby cycles. When the session resumes, the filesystem is exactly where the agent left it.
  - Shared context works differently. A knowledge base, a RAG corpus, or a shared codebase should be mountable into any session as a read-only (or read-write) layer, separate from the session's private state. This separation matters: session state belongs to one agent interaction, shared context belongs to the organization. Conflating the two leads to permission errors, stale data, and state corruption when multiple agents write to the same paths without coordination.
  - Blue-green deployments depend on this separation. When a new version of agent code is deployed, active sessions should be able to finish their current turn on the old version, then seamlessly pick up the next turn on the new version — because the state lives on a persistent layer that survives sandbox replacement. Sessions can run for months with continuous code updates, and the handoff happens at the turn boundary, not the session boundary. Without state that outlives compute, every code deploy forces a session restart.
* **Technical Entities (Classes/Functions/APIs):** `vector databases`, `SQLite`, `RAG`
* **Code Snippet:** None.

---

## Tool and API integration layer

* **Key Points:**
  - The integration layer routes tool calls to APIs, databases, code execution environments, and enterprise systems. It normalizes inputs and outputs across tool interfaces. Timeouts and rate limits are enforced. Retries handle transient failures, and error responses flow back to the agent.
  - Model Context Protocol (MCP) is changing how this works. Before MCP, each model-tool combination required a custom integration. MCP collapses this to a modular pattern where agents discover tools dynamically at runtime rather than through hardcoded integrations. Enterprise software vendors are increasingly launching their own MCP servers, reflecting this shift toward standardized connectivity.
  - Perpetual sandbox platforms like Blaxel provide MCP Servers Hosting with pre-built integrations for dynamic agent-to-tool connectivity. Agents connect through the standard MCP client protocol using streamable HTTP transport. Teams avoid building custom integration code for each external system. New tool capabilities become available to agents automatically as MCP servers update, without redeploying agent code.
* **Technical Entities (Classes/Functions/APIs):** `APIs`, `Model Context Protocol (MCP)`, `MCP servers`, `Blaxel`, `HTTP`
* **Code Snippet:** None.

---

## Execution sandboxes and compute isolation

* **Key Points:**
  - Agents that execute code or run high-risk tools need isolated environments. Failures and exploits must stay contained. The compute layer provisions these environments and manages resource quotas for CPU, memory, and storage. Lifecycle management covers starting, pausing, resuming, and terminating agent sessions safely.
  - The isolation model matters. Containers share the host kernel. A kernel vulnerability inside one container can affect every workload on the same host. MicroVMs run a separate kernel per workload, providing hardware-enforced boundaries.
  - NIST SP 800-233 describes this isolation hierarchy explicitly: containers provide some degree of isolation, microVMs a stronger degree, and full VMs the strongest. For agents running LLM-generated code that hasn't been human-reviewed, it's the difference between a contained failure and a lateral breach.
  - Perpetual sandbox platforms like Blaxel use microVMs inspired by the technology behind AWS Lambda rather than containers. Sandboxes resume from standby in under 25ms with the exact previous state restored.
  - They return to standby after 15 seconds of network inactivity, with zero compute charges while idle. For guaranteed long-term data persistence beyond the standby state, volumes are required. This combination gives agents that execute untrusted code a stronger isolation boundary and a lifecycle designed for production use.
* **Technical Entities (Classes/Functions/APIs):** `MicroVMs`, `AWS Lambda`, `Blaxel`, `NIST SP 800-233`
* **Code Snippet:** None.

---

## Observability, audit, and governance

* **Key Points:**
  - The governance layer captures logs, traces, metrics, and audit trails for every decision and tool call. Observability dashboards consume these signals for debugging. Compliance systems use the same data for audit.
  - Human-in-the-loop controls sit here too. Approval workflows pause an agent before high-risk actions. Guardrails constrain agent behavior within defined boundaries. Escalation paths activate when the agent reaches a decision it shouldn't make autonomously.
  - For engineering leaders, this layer answers a critical compliance question: "Can we prove what the agent did and why?" Gartner predicts AI regulatory violations will result in a 30% increase in legal disputes for technology companies by 2028.
  - Without immutable audit trails, organizations can't demonstrate that agent actions were authorized, bounded, and compliant. That exposure becomes a procurement blocker for organizations with strict compliance requirements.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Enterprise requirements for evaluating an agent runtime

* **Key Points:**
  - Choosing an agent runtime is an infrastructure decision with multi-year implications. This applies especially to teams deploying tool-heavy, long-running, or code-executing agents. The categories below map to common evaluation criteria.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Scalability and multi-agent coordination

* **Key Points:**
  - Agent workloads are bursty. A coding agent platform might handle moderate sessions during off-hours and spike several times over during peak development time. The runtime needs to scale horizontally across those peaks without pre-provisioned capacity sitting idle.
  - Multi-agent coordination multiplies the challenge. Agents delegate sub-tasks to other agents, and governance must sit outside both build and orchestration environments. Ask: does the platform support durable state across long-running workflows? Can it trace decisions through all participating agents in a delegation chain? For fan-out workloads, ask whether the runtime supports batch execution built for scheduled and parallel processing.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Security, isolation, and data protection

* **Key Points:**
  - Each agent should operate as an independent security principal with scoped, short-lived credentials separate from human accounts. Restrict each agent to the minimum permissions it needs. A document processing agent gets read-only access to the document store and no network access beyond its required APIs.
  - An agent that queries a database shouldn't have write access. This scoping prevents a compromised agent from becoming a pivot point for lateral movement.
  - Secure sandboxing for code execution requires workload isolation where a vulnerability in one sandbox can't reach the host or neighbors. General Data Protection Regulation (GDPR) Article 22 requires specific safeguards for automated decision-making, including the right to obtain human intervention. Ask: does the platform treat agents as independent security principals with their own identities and access controls?
* **Technical Entities (Classes/Functions/APIs):** `APIs`, `GDPR`
* **Code Snippet:** None.

---

## Reliability and failure recovery

* **Key Points:**
  - A mid-session failure without checkpointing means restarting from scratch. Consider a document processing agent late in a multi-step workflow. Data has been extracted from multiple PDFs. Fields are validated against business rules. External APIs are called.
  - Then the compute node fails. Without checkpointing, everything restarts. Documents get re-processed, and APIs re-called, potentially triggering duplicate transactions. With checkpointing, the agent resumes from where it left off with full context intact.
  - Evaluate: Does the platform support mid-execution checkpointing? Are there circuit breakers that isolate failures in one tool call from cascading? Research from Stanford's Human-Centered AI Institute shows that while top AI systems outperform humans at short time horizons, human performance surpasses AI at longer ones. Reliable session management is a precondition for agents tackling multi-hour tasks.
* **Technical Entities (Classes/Functions/APIs):** `APIs`
* **Code Snippet:** None.

---

## Compliance and auditability

* **Key Points:**
  - SOC 2 Type II's processing integrity criterion requires that system processing is complete, valid, accurate, timely, and authorized. The Health Insurance Portability and Accountability Act (HIPAA) requires technical safeguards protecting electronic protected health information (ePHI).
  - When ePHI flows through agent execution environments, the same access controls, audit controls, and transmission security requirements apply. Ask: does the vendor hold a current SOC 2 Type II report with the runtime in scope? Does it sign a Business Associate Agreement (BAA)?
* **Technical Entities (Classes/Functions/APIs):** `SOC 2 Type II`, `HIPAA`, `ePHI`, `Business Associate Agreement (BAA)`
* **Code Snippet:** None.

---

## Observability and cost control

* **Key Points:**
  - Token consumption can spiral without controls. An agent stuck in a reasoning loop burns tokens with every iteration. Repeated iterations make a single session unexpectedly expensive. Budget guardrails per agent, session, or tenant prevent a single misbehaving agent from consuming the monthly budget. A model gateway can centralize those controls while standardizing telemetry across providers.
  - Look for OpenTelemetry compatibility so agent workloads integrate into your existing monitoring stack rather than requiring a parallel one. In stacks that expose a unified model access layer, that same gateway can help with LLM routing, token visibility, and centralized policy enforcement.
* **Technical Entities (Classes/Functions/APIs):** `OpenTelemetry`, `LLM`
* **Code Snippet:** None.

---

## Agent runtime patterns in production

* **Key Points:**
  - Two patterns dominate early production deployments. They stress-test different runtime capabilities: sandboxed code execution in the first case, durable multi-step workflows in the second.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Code generation and developer tooling

* **Key Points:**
  - Coding agents generate, test, and apply code changes across full repositories. This is one of the most validated production patterns for agent runtimes.
  - The agent doesn't generate code and immediately open a PR. Instead, code deploys to an isolated sandbox where integration tests run. Load is generated and logs are analyzed before the results surface to a developer.
  - The runtime manages sandbox lifecycle: spinning up isolated environments, executing generated code, running tests, and returning results. State persistence keeps the repo cloned and ready between sessions. Security isolation ensures untrusted generated code runs in a boundary that can't reach production systems.
  - Agentic developer workflows can also use isolated sandboxes for agents and MCP servers. This prevents a compromised component from impacting the broader system. Agents are firewalled and access only resources explicitly specified by developers.
  - Blaxel's perpetual sandboxes keep environments ready in standby indefinitely. The sandbox resumes in under 25ms with its previous state restored rather than forcing a cold start when the user returns. Co-located Agents Hosting reduces latency between agent logic and the sandbox, keeping the loop responsive for interactive workloads – while a future Blaxel product, Agent Runtime (expected Q2 2026), a purpose-built agent runtime that crosses the requirements of this article, will manage agentic sessions.
* **Technical Entities (Classes/Functions/APIs):** `MCP servers`, `Blaxel`, `Agent Runtime`
* **Code Snippet:** None.

---

## Document processing and workflow automation

* **Key Points:**
  - Document processing agents extract data, validate against business rules, call external APIs, and route for human approval. The runtime manages multi-step workflows where each step depends on the previous one.
  - State persistence is the critical runtime capability here. A failure late in the workflow needs to resume from the last checkpoint, not restart from the beginning. Approval gates require human review before the agent proceeds. The runtime pauses the agent, persists its full context, and resumes when approval arrives. That resumption might happen hours later.
  - Organizations are moving from agents hardcoded into applications toward decoupled agent platforms. BCG's enterprise AI research points to broader changes in how enterprises orchestrate AI-driven processes.
  - The runtime layer makes that decoupling possible. It provides a durable execution substrate that lets agents operate across systems without being tightly coupled to any single backend. Where workflows fan out across many asynchronous tasks, batch jobs become a natural extension of the runtime architecture.
* **Technical Entities (Classes/Functions/APIs):** `APIs`
* **Code Snippet:** None.

---

## Build your AI agent runtime on infrastructure designed for these workloads

* **Key Points:**
  - The runtime layer determines whether production agents are reliable, safe, and auditable. Teams that treat the execution layer as an afterthought often rebuild it under pressure when failures expose the gaps. The complexity of building this from scratch pushes many teams toward specialized platforms.
  - Perpetual sandbox platforms like Blaxel provide building blocks that map to the components covered here. Sandboxes use microVMs inspired by the technology behind AWS Lambda and resume from standby in under 25ms with the exact previous state restored.
  - For guaranteed long-term persistence, volumes are required. MCP Servers Hosting covers standardized tool connectivity. Co-located Agents Hosting reduces latency between agent logic and execution environments. Batch Jobs support parallel and long-running background work. Model Gateway centralizes model access, telemetry, and token cost controls. Finally, the upcoming product Blaxel Agent Runtime, expected Q2 2026, will be a purpose-built agent runtime that handles agentic sessions, as described in this article.
* **Technical Entities (Classes/Functions/APIs):** `Blaxel`, `microVMs`, `AWS Lambda`, `MCP Servers Hosting`, `Co-located Agents Hosting`, `Batch Jobs`, `Model Gateway`, `Blaxel Agent Runtime`
* **Code Snippet:** None.


## The runtime behind production deep agents

* **Key Points:**
  - Deploying long horizon agents in production requires purpose-built infrastructure. This guide covers durable execution, memory, HITL, observability, and how deepagents deploy ships it all to production.
  - To build a good agent, you need a good harness. To deploy that agent, you need a good runtime.
  - The harness is the system you build around the model to help your agent be successful in its domain. That includes prompts, tools, skills, and anything else supporting the model and tool calling loop that defines an agent. The runtime is everything underneath: durable execution, memory, multi-tenancy, observability, the machinery that keeps an agent running in production without your team reinventing it.
  - This guide walks through the production requirements that surface once you deploy agents, the runtime capabilities that meet them, and how deepagents deploy packages those capabilities into something you can ship.
* **Technical Entities (Classes/Functions/APIs):** `deepagents deploy`, `harness`, `runtime`
* **Code Snippet:** None.

---

## Runtime capabilities for production agents

* **Key Points:**
  - Throughout this section, "the runtime" refers to LangSmith Deployment (LSD) and its Agent Server: LSD runs agents in production, and Agent Server is the interface for assistants, threads, runs, memory, and scheduled jobs. The table below maps each production requirement to the runtime primitive that meets it.
* **Technical Entities (Classes/Functions/APIs):** `LangSmith Deployment (LSD)`, `Agent Server`, `assistants`, `threads`, `runs`, `memory`, `scheduled jobs`
* **Code Snippet:** None.

---

## Durable execution

* **Key Points:**
  - Agents work by running a loop: Given a prompt, the model reasons, calls tools, observes the results, and repeats until it decides the task is complete.
  - Unlike a typical web request that returns in milliseconds, this loop can span minutes or hours. A single run might make dozens of model calls, spawn subagents, or wait indefinitely for a human to approve a draft. A crash, deploy, or transient failure anywhere in that loop shouldn't erase the work leading up to it.
  - In practice, you feel it in two places: Long runs need to survive infrastructure failures. A research agent spending twenty minutes gathering sources and synthesizing findings can't afford to restart from scratch if the worker process dies: the agent already paid for the tokens and executed the tool calls. What you want is resumption from the last completed step, with all prior state intact. Agents need to be able to stop and wait. An agent that pauses for a human to approve a transaction doesn't know if the human will respond in thirty seconds or three days. Tying up a worker process or a client connection for that entire window isn't viable. The agent needs to truly stop: free resources, release workers, then pick up later exactly where it left off.
  - Both requirements are solved by the same thing: durable execution.
  - Agents run on a managed task queue with automatic checkpointing, so any run can be retried, replayed, or resumed from the exact point of interruption.
  - Each super-step of graph execution writes a checkpoint to the persistence layer (PostgreSQL by default), keyed by a thread_id that acts as a persistent cursor into the run.
  - When a worker crashes, the run's lease is released and another worker picks it up from the latest checkpoint.
  - When an agent waits for human input, the process hands off its slot and the run sleeps indefinitely until resumed.
  - Configurable retry policies control backoff, max attempts, and which exceptions trigger retries on a per-node basis.
  - Durability is the foundation the rest of this list depends on. Because execution can pause and resume across process boundaries, agents can wait indefinitely for human input, run in the background, survive deploys mid-run, and handle concurrent inputs without corrupting state.
* **Technical Entities (Classes/Functions/APIs):** `PostgreSQL`, `thread_id`, `checkpoint`, `persistence layer`
* **Code Snippet:** None.

---

## Memory

* **Key Points:**
  - Agents need two different kinds of memory, and the distinction matters.
  - Short-term memory is what the agent accumulates within a single conversation. The messages exchanged, the tool calls made, the intermediate state built up across a run. This lives in the checkpoint for the thread, scoped to a thread_id, and disappears (conceptually) when the conversation ends. A follow-up message on the same thread sees everything that came before on that thread.
  - Long-term memory is what the agent carries across conversations. This can include user preferences learned across conversations, project conventions and best practices, or a knowledge base enhanced with each new query. None of this belongs to any single thread. It's user-level or organization-level context that should persist across every conversation the agent has. Checkpoints alone can't do this, because checkpoint state is scoped to a single thread.
  - Long-term memory is what the Agent Server's built-in store is for. It's a key-value interface where memories are organized by namespace tuples (for example, (user_id, "memories")) and persisted across threads. Your agent writes to the store in one conversation and reads from it in the next. Backed by PostgreSQL by default, it supports semantic search via embedding configuration so agents can retrieve memories by meaning rather than exact match, and you can swap in a custom backend if you need different storage characteristics. The namespace structure is flexible: scope by user, assistant, organization, or any combination that fits your data model.
  - Because memory that accumulates over months is some of the most valuable data the system produces, it matters where it lives. The store is queryable directly via API, and if you self-host, it lives in your own PostgreSQL instance. Keeping this data in a standard format you control is what lets you migrate between models, analyze it, or build on top of it outside the agent itself.
* **Technical Entities (Classes/Functions/APIs):** `Agent Server`, `store`, `PostgreSQL`, `API`, `thread_id`
* **Code Snippet:** None.

---

## Multi-tenancy

* **Key Points:**
  - The moment your agent serves more than one user, a set of problems appears that didn't exist in single-player mode. These break down into three distinct concerns, and the Agent Server handles each with its own primitive.
  - Isolating one user's data from another. User A's run should only touch User A's threads, and only read User A's memories. Custom authentication runs as middleware on every request: your @auth.authenticate handler validates the incoming credential and returns the user's identity and permissions, which get attached to the run context. Authorization handlers registered with @auth.on.threads, @auth.on.assistants.create, and so on then enforce who can see or modify what by tagging resources with ownership metadata on creation and returning filter dictionaries on reads. Handlers are matched from most specific to least, so you can start with a single global handler and add resource-specific ones as your model grows.
  - Letting the agent act on behalf of a user. Agents often need to call third-party services using the user's credentials—reading their calendar, posting to their Slack, opening a PR in their repo. Agent Auth handles the OAuth dance and token storage for this pattern, so the agent gets user-scoped credentials at runtime without you managing the refresh flow yourself. The user authenticates once; the agent can act on their behalf across subsequent runs.
  - Controlling who can operate the system itself. Separate from end-user access, there's the question of which members of your team can deploy agents, configure them, view traces, or change auth policies. RBAC handles this operator-level access control.
  - The three layers compose: end users authenticate via your auth handler, the agent calls third-party services via Agent Auth, and your team operates the deployment under RBAC policies.
* **Technical Entities (Classes/Functions/APIs):** `Agent Server`, `@auth.authenticate`, `@auth.on.threads`, `@auth.on.assistants.create`, `Agent Auth`, `OAuth`, `RBAC`
* **Code Snippet:** None.

---

## Human-in-the-loop (HITL)

* **Key Points:**
  - Agents work by running a loop: given a prompt, a model reasons and decides to call tools, observes the results, and repeats until it decides it has completed the task at hand. Most of the time you want that loop to run uninterrupted. That's where the value comes from. But sometimes you need a human in the middle of the loop at key decision points.
  - There are two common situations where this comes up: Reviewing a proposed tool call. Before the agent executes a consequential action (sending an email, executing a financial transaction, deleting files), you want a human to see exactly what it's about to do and decide how to respond. Take the email case: the agent drafts a message and pauses before sending. You can approve it as-is, edit the subject or body before it goes out, or reject it with a reason and specific edit requests so the agent can revise and try again. An agent asking a clarifying question. Sometimes an agent reaches a decision point it can't resolve on its own, not because it lacks a tool but because the right answer depends on human judgment or preference. Rather than guessing, the agent can surface the question directly: "I found three config files matching that pattern. Which one should I modify?" or "Should this deploy to staging or production?" Your answer becomes the return value of the interrupt, and the agent continues from exactly where it stopped.
  - The Agent Server handles this with two primitives: interrupt() pauses execution and surfaces a payload to the caller; Command(resume=...) continues it with the human's response. Together they let you build approval gates, draft review loops, input validation, and any workflow where a human needs to weigh in mid-execution.
  - Under the hood, interrupt() triggers the runtime's checkpointer to write the full graph state to durable storage, keyed by a thread_id that acts as a persistent cursor. The process then frees resources and waits indefinitely. Unlike static breakpoints that pause before or after specific nodes, interrupt() is dynamic: place it anywhere in your code, wrap it in conditionals, or embed it inside a tool function so approval logic travels with the tool. When Command(resume=...) arrives—minutes, hours, or days later—the resume value becomes the return value of the interrupt() call, and execution picks up exactly where it stopped. Because resume accepts any JSON-serializable value, the response isn't limited to approve/reject: a reviewer can return an edited draft, a human can supply missing context, a downstream system can inject computed results. When parallel branches each call interrupt(), all pending interrupts are surfaced together and can be resumed in a single invocation, or one at a time as responses come back.
* **Technical Entities (Classes/Functions/APIs):** `Agent Server`, `interrupt()`, `Command(resume=...)`, `thread_id`
* **Code Snippet:** None.

---

## Real-time interaction

* **Key Points:**
  - Human-in-the-loop is an interaction mode where execution can pause for a person to review or provide input—sometimes immediately, sometimes much later. Separately, there are "live session" problems that show up when the agent is actively working while the user is present: making progress visible (streaming) and coordinating concurrent messages (double-texting).
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Streaming

* **Key Points:**
  - An agent that takes thirty seconds to produce a response leaves the user staring at a spinner with no signal about whether it's making progress, stuck, or about to fail. They also can't start reading the answer until the whole thing is done. Streaming solves both: partial output flows to the client as the agent produces it, so the user sees the response materialize in real time.
  - The Streaming API supports several modes depending on what granularity you want: full state snapshots after each graph step, state updates only, token-by-token LLM output, or custom application events. You can also combine them. Run streaming (client.runs.stream()) is scoped to a single run; thread streaming (client.threads.joinStream()) opens a long-lived connection that delivers events from every run on a thread, useful when follow-up messages, background runs, or HITL resumptions all trigger activity on the same thread.
  - Thread streaming supports resumption via the Last-Event-ID header: the client reconnects with the ID of the last event it received, and the server replays from there with no gaps. Without this, every dropped connection means the client either misses output or has to start over.
* **Technical Entities (Classes/Functions/APIs):** `Streaming API`, `client.runs.stream()`, `client.threads.joinStream()`, `Last-Event-ID`, `LLM`
* **Code Snippet:** None.

---

## Double-texting

* **Key Points:**
  - The second real-time problem: a user sends a new message while the agent is still working on the previous one. This happens constantly in chat UIs. Someone types a question, realizes they meant something slightly different, and fires off a correction before the first run finishes. We call this double-texting, and the runtime has to take a position on how to handle it.
  - There are four strategies, and the right one depends on your application:
  - enqueue (the default): The new input waits for the current run to finish, then processes sequentially.
  - reject: Refuse any new input until the current run finishes.
  - interrupt: Halt the current run, preserve progress, and process the new input from that state. Useful when the second message builds on the first.
  - rollback: Halt the current run, revert all progress including the original input, and process the new message as a fresh run. Useful when the second message replaces the first.
  - interrupt gives the snappiest chat feel but requires your graph to handle partial tool calls cleanly (a tool call initiated but not completed when the interrupt hits may need cleanup on resume). enqueue is the safest default—no state corruption, at the cost of making the user wait.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Guardrails

* **Key Points:**
  - Not every production concern can be expressed as "run the loop durably." Some have to shape the loop itself: intercepting model inputs, filtering tool outputs, enforcing limits on expensive operations. These policies belong in code, not in a prompt. They need to run every time, not whenever the model happens to remember them.
  - Two cases make this concrete: Redacting sensitive data before the model sees it. A customer support agent processes user messages containing PII (names, emails, account numbers). You don't want the model to see them, you don't want them in traces, and compliance likely requires redaction before logging. This has to happen before every model call, deterministically. Capping expensive operations. An agent that can call a paid external API needs a hard ceiling on how many calls it makes per run, because a confused model will otherwise happily call it fifty times and burn through your budget before lunch.
  - Both are handled by middleware, which wraps the agent loop at defined hooks—before_model, wrap_model_call, wrap_tool_call, after_model—so policies execute deterministically around every relevant step.
  - LangChain ships built-in middleware covering the common cases: PIIRedactionMiddleware, ModelRetryMiddleware, ModelFallbackMiddleware, ToolCallLimitMiddleware, SummarizationMiddleware, HumanInTheLoopMiddleware, OpenAIModerationMiddleware, and you can write custom middleware for application-specific policies.
  - Middleware is open source, but it only really pays off when it runs inside the agent runtime. When it does, those same policies become part of every interaction mode the runtime supports—streaming, human-in-the-loop pauses/resumes, retries, background runs, and long-lived threads. In practice, that means your guardrails and instrumentation aren't "best effort": they consistently wrap every model call and every tool call, at the exact points you expect, no matter what the agent is doing.
* **Technical Entities (Classes/Functions/APIs):** `middleware`, `before_model`, `wrap_model_call`, `wrap_tool_call`, `after_model`, `LangChain`, `PIIRedactionMiddleware`, `ModelRetryMiddleware`, `ModelFallbackMiddleware`, `ToolCallLimitMiddleware`, `SummarizationMiddleware`, `HumanInTheLoopMiddleware`, `OpenAIModerationMiddleware`
* **Code Snippet:** None.

---

## Observability

* **Key Points:**
  - You don't know what an agent will do in production until you run it. Unlike a traditional application where you can reason about behavior from the code, an agent's execution path depends on the model's choices at runtime: which tools to call, what to pass them, how to interpret the results, and when to give up and try something else. When something goes wrong, you can't just re-read the function. You need to see what actually happened.
  - A support ticket says "the agent kept asking the same question over and over." Without traces, you're guessing from the user's description. With traces, you see the full execution tree: the user's message, the model's planned response, the tool it called, the result it got back, the next message it generated, the loop it fell into. You can filter by cost to find runs that burned through tokens, by error to find runs that failed, by user to see what a specific customer experienced. You can spot patterns across thousands of runs that no individual trace would reveal.
  - Every LangSmith Deployment is automatically wired to a tracing project. You get the full execution tree out of the box—model calls, tool calls, subagent runs, middleware hooks—with structured metadata you can query by user, time window, cost, latency, error state, feedback, or custom tags.
  - Traces are the foundation of the improvement loop:
  - Polly, the LangSmith AI assistant, analyzes traces and surfaces insights—common failure modes, slow tool calls, repeated patterns—so you're not reading thousands by hand. Online Evals run LLM-as-judge or custom scorers against production traces automatically, so regressions get caught as they happen. We used this loop to improve Deep Agents by 13.7 points on Terminal Bench 2.0 by only changing the harness—the whole argument for why the agent improvement loop starts with a trace is worth reading in full.
* **Technical Entities (Classes/Functions/APIs):** `LangSmith Deployment`, `Polly`, `LangSmith AI assistant`, `Online Evals`, `Deep Agents`, `Terminal Bench 2.0`
* **Code Snippet:** None.

---

## Time travel

* **Key Points:**
  - Observability tells you what happened. Time travel lets you ask what would have happened if something had gone differently.
  - The motivating case is debugging a run that went off the rails. Your agent made a bad decision at step 5 of a 20-step run: it called the wrong tool, misread a tool result, or asked a clarifying question when it should have kept going. You want to understand why, and you want to try alternatives without re-running the whole thing from scratch. More generally, any time an agent's path depends on state at a particular checkpoint, you want the ability to rewind to that checkpoint, change the state, and let the rest of the run unfold differently.
  - Because every super-step writes a checkpoint, every point in a run's history is already a snapshot you can return to. Time travel makes this explicit: pick a checkpoint from a thread's history, optionally modify its state, and resume from there. The modified checkpoint forks the thread's history. The original stays intact, and the new path runs forward as its own branch. LLM calls, tool calls, and interrupts all re-trigger on replay, so forks exercise the real agent loop rather than a stub of it.
  - This unlocks patterns that are hard to build otherwise: debugging why the agent chose tool A when it should have chosen tool B, comparing two prompts against the same upstream context, recovering from a run that went sideways by rewinding to the last good state, or exploring counterfactuals across many forks to understand model behavior. The LangSmith Studio UI gives you a visual interface for all of this; the API is what most production debugging workflows end up using.
* **Technical Entities (Classes/Functions/APIs):** `LangSmith Studio UI`, `API`, `LLM`
* **Code Snippet:** None.

---

## Code execution

* **Key Points:**
  - An agent that can only call the tools you pre-wired is limited to what you anticipated. An agent that can run arbitrary code is general-purpose: it can install dependencies, clone repos, execute tests, run data analysis, generate documents, and render plots. This is the gap between "chatbot with function calling" and "agent that can actually do things."
  - Arbitrary code execution requires isolation. If the agent runs rm -rf / on your host, you have a bad day. If it reads your environment variables, it exfiltrates your API keys. You need a boundary between the agent's execution environment and everything you care about, and you need it before the agent writes its first command.
  - In Deep Agents, isolation happens through sandbox backends. When you configure a backend that implements SandboxBackendProtocol, the agent automatically gets an execute tool for running shell commands in the sandbox alongside the standard filesystem tools. Without a sandbox backend, the execute tool isn't even visible to the agent. Supported providers include Daytona, Modal, Runloop, and LangSmith Sandboxes, and you can swap between them with a single configuration change.
  - LangSmith Sandboxes (currently in private preview) are worth a specific callout because they're built to integrate with the rest of the runtime. Templates define container images, resource limits, and volumes declaratively. Warm pools pre-provision sandboxes with automatic replenishment, eliminating cold start latency for interactive agents. And the auth proxy solves a problem every team hits eventually: the agent needs to call authenticated APIs, but putting credentials inside the sandbox is a security risk. The proxy runs as a sidecar, intercepts outbound requests, and injects credentials from workspace secrets automatically—the sandbox code calls api.openai.com with no headers, and the proxy adds the right Authorization header on the way out. Secrets never enter the sandbox, and the agent can't exfiltrate what it can't see.
  - One piece of security guidance worth repeating: sandboxes protect your host, not the sandbox itself. An attacker who controls the agent's input (via prompt injection in a scraped webpage, a malicious email, a poisoned tool result) can instruct the agent to run commands inside the sandbox. The sandbox keeps the attacker off your machine, but anything inside the sandbox—including credentials placed there directly—is compromised. The auth proxy pattern exists for exactly this reason.
* **Technical Entities (Classes/Functions/APIs):** `Deep Agents`, `SandboxBackendProtocol`, `execute`, `Daytona`, `Modal`, `Runloop`, `LangSmith Sandboxes`, `auth proxy`, `api.openai.com`, `Authorization`
* **Code Snippet:** None.

---

## Integrations

* **Key Points:**
  - Agents are most useful when they plug into the systems people and organizations already use. A coding agent becomes more powerful when it can reach into GitHub, Linear, and your CI system. A research agent becomes more useful when its output feeds into your publishing pipeline. An internal agent becomes a platform when other agents can call it as a building block. If every one of those integrations is a hand-rolled adapter, your agents stay isolated. The boundary between "agent" and "everything else" becomes a wall.
  - Open protocols solve this by letting agents and external systems discover and talk to each other without either side knowing the other's implementation. The Agent Server provisions three integration surfaces automatically.
  - MCP: MCP (Model Context Protocol) is the open standard for connecting agents to tools and data sources. Every LangSmith Deployment automatically exposes an MCP endpoint, making your agent discoverable by any MCP-compliant client—Claude Desktop, IDEs, other agents, custom applications—without you writing adapter code. In the other direction, your agent can call out to any MCP server (Linear, GitHub, Notion, and hundreds of others) to reach tools and data your users already have.
  - A2A: A2A (Agent-to-Agent) is the analogous standard for agent-to-agent communication, and every deployment exposes an A2A endpoint automatically as well. This is what makes multi-agent architectures across deployments tractable: an orchestrator agent in one deployment can discover and call worker agents in another using a protocol both sides understand, with no hand-rolled HTTP contracts.
  - Webhooks: Webhooks handle the outbound case: your agent finishes a run, and you want to kick off something downstream without polling. Pass a webhook URL when creating a run, and the server POSTs the run payload to that URL on completion. This is how you chain agent runs into existing workflows—a research run completes and triggers a publishing pipeline, a daily summary finishes and notifies Slack, a compliance check completes and writes to your audit log. Headers, domain allowlists, and HTTPS enforcement are all configurable for production environments.
* **Technical Entities (Classes/Functions/APIs):** `Agent Server`, `MCP (Model Context Protocol)`, `LangSmith Deployment`, `MCP endpoint`, `Claude Desktop`, `MCP server`, `Linear`, `GitHub`, `Notion`, `A2A (Agent-to-Agent)`, `Webhooks`, `POST`, `HTTPS`
* **Code Snippet:** None.

---

## Cron

* **Key Points:**
  - The agents we've been talking about so far are reactive: a user sends a message, the agent responds. But a lot of valuable agent work is proactive—it happens on a schedule, with no human triggering it.
  - Two patterns in particular: Sleep-time compute. Agents that do useful work during idle periods, so users benefit from accumulated thinking rather than on-demand latency. A research agent that runs nightly to catch up on new papers in your field. A prep agent that reviews tomorrow's calendar and drafts briefing notes before you start your day. A triage agent that classifies overnight support tickets so your team walks into a prioritized queue. The work happens while nobody's waiting, and the output is ready when the user shows up. Health and monitoring loops. Agents that periodically check on something and act (or escalate) if they find an issue. An on-call agent that reviews alerts every fifteen minutes, an agent that monitors your staging environment for regressions, a compliance agent that sweeps for policy violations on a cadence. These need the same durability, tracing, and auth as user-facing runs, but no user is waiting on them.
  - The Agent Server has cron jobs built in, so scheduled runs get the same durability, tracing, and auth guarantees as any other run—no separate scheduler to maintain, no second observability story to wire up. You pass a standard cron expression (UTC) and an input, and the server triggers runs on schedule.
  - Two flavors fit different patterns: Stateful cron (client.crons.create_for_thread) ties the schedule to a specific thread_id, so every triggered run appends to the same conversation. This fits agents that should see their own history—a daily research agent that builds on yesterday's findings, or a monitoring agent that remembers what it already flagged. Stateless cron (client.crons.create) spins up a fresh thread for each execution, which fits batch-style work that doesn't need continuity between runs. Control thread cleanup via on_run_completed: "delete" (the default) removes the thread when the run finishes, "keep" preserves it for later retrieval via client.runs.search(metadata={"cron_id": cron_id}).
  - Every cron run shows up in tracing, respects auth handlers and middleware, and supports resumption on failure—a cron that hits a transient model outage at 3am doesn't silently fail, it gets retried like any other run. One operational note: delete crons when you're done with them. They keep running (and billing) until you do.
  - We see enterprise teams with varying deployment requirements, so the runtime supports cloud, hybrid, and self-hosted deployments. The capabilities are the same regardless of where you run it.
* **Technical Entities (Classes/Functions/APIs):** `Agent Server`, `cron`, `client.crons.create_for_thread`, `thread_id`, `client.crons.create`, `on_run_completed`, `client.runs.search`
* **Code Snippet:** None.

---

## deepagents deploy

* **Key Points:**
  - deepagents deploy is the packaging step that deploys your agent on the runtime described above. You define your agent in deepagents.toml, and the CLI bundles your configuration and deploys it as a LangSmith Deployment with all of the aforementioned features.
  - Memory uses a virtual filesystem with pluggable backends that gives agents both ephemeral scratch space and persistent cross-conversation storage. Deep Agents support memory scoped to users or assistants (or both)!
  - Sandbox providers (LangSmith Sandboxes, Daytona, Modal, Runloop, or custom) are a single config value. When a sandbox is present, the harness automatically adds an execute tool. Sandbox lifecycle (thread-scoped vs assistant-scoped) is handled through graph factories. Credentials inside sandboxes are managed through the sandbox auth proxy so API keys never appear in sandbox code or logs.
  - Skills and instructions are auto-detected from your skills/ directory and AGENTS.md. MCP servers are picked up from mcp.json. The name field is the only required config value; everything else has sensible defaults.
  - The result is a deployment that can evolve over time, with new skills, tools, and memory policies, without a full rewrite. For the complete set of production considerations (credential management, async patterns, frontend integration, and more), see the going-to-production guide.
* **Technical Entities (Classes/Functions/APIs):** `deepagents deploy`, `deepagents.toml`, `LangSmith Deployment`, `LangSmith Sandboxes`, `Daytona`, `Modal`, `Runloop`, `execute`, `auth proxy`, `AGENTS.md`, `mcp.json`, `MCP servers`
* **Code Snippet:** None.

---

## Open Harness

* **Key Points:**
  - There's a growing trend in agent infrastructure where moving to a managed solution comes with reduced builder choice—lock-in to a single model provider, a closed harness, or harness functionality hidden behind APIs (like server-side compaction that generates encrypted summaries you can't use outside one ecosystem). The practical consequence is that teams lose visibility into how their agent actually works, and lose the ability to change it when it doesn't.
  - One note on vendor lock-in: deepagents deploy is built to avoid it. The harness is MIT licensed and fully open source, agent instructions use AGENTS.md (an open standard), and agents are exposed via open protocols—MCP, A2A, Agent Protocol. There's no model or sandbox lock-in, and nothing about the harness is a black box. The default harness offers the following capabilities:
  - Additionally, Deep Agents allows you to inspect, customize, and extend every layer of agent behavior, including rate limits, retry logic, model fallback, PII detection, and file permissions via LangChain's middleware.
* **Technical Entities (Classes/Functions/APIs):** `deepagents deploy`, `MIT licensed`, `AGENTS.md`, `MCP`, `A2A`, `Agent Protocol`, `LangChain`, `middleware`
* **Code Snippet:** None.

---

## Take your agents to production

* **Key Points:**
  - The capabilities this guide outlines—durable execution, memory, multi-tenancy, guardrails, human-in-the-loop, observability, sandboxed code execution, scheduled runs, and more—are the infrastructure requirements production agents can't function without. deepagents deploy packages all of it so teams don't have to assemble it from scratch, and keeps the stack open, configurable, and yours throughout.
  - Building agents is a deeply iterative cycle: traces surface what's actually happening in production, online evals catch regressions before they compound, and memory means the agent gets more useful over time. The infrastructure isn't just supporting the live agent, it's the foundation for making it better.
  - If you want to try it out, the quickstart will get you from deepagents.toml to a running deployment in minutes. For the full production playbook including memory scoping, sandbox lifecycle, credential management, guardrails, and frontend integration, see the going-to-production guide. For a deeper look at the runtime itself, see the LangSmith Deployment and Agent Server docs.
* **Technical Entities (Classes/Functions/APIs):** `deepagents deploy`, `deepagents.toml`, `LangSmith Deployment`, `Agent Server`
* **Code Snippet:** None.
