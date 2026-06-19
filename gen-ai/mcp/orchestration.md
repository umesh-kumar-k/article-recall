---
aliases:
  - Multiple MCP server Orchestration
Source 1: https://portkey.ai/blog/orchestrating-multiple-mcp-servers-in-a-single-ai-workflow/
Source 2: https://aws.amazon.com/blogs/devops/flexibility-to-framework-building-mcp-servers-with-controlled-tool-orchestration/
---
# Orchestrating multiple MCP servers in a single AI workflow

## The multi-server orchestration challenge
* **Key Points:**
  - MCP servers are the connective tissue between AI agents and the tools they rely on. They allow LLMs and agents to read from databases, trigger workflows, post updates, and interact with countless external systems, all through a consistent, standardized interface.
  - But as teams start to run workflows that span multiple MCP servers, each hosting different sets of tools, a new layer of complexity emerges. Authenticating to each server separately, managing scattered permissions, and keeping policies consistent across environments quickly turns into operational overhead.
  - In a simple setup, an agent might connect to a single MCP server and run a handful of tools it exposes. But in real-world deployments, the toolset an agent needs is rarely confined to one server.
  - Consider an incident-response workflow: MCP Server A hosts a knowledge base search tool. MCP Server B connects to the team's issue tracker. MCP Server C integrates with a messaging platform.
  - The agent needs to query recent incidents, create a ticket for the new issue, and post an update, all in one flow. Without orchestration, this means: Authenticating to each MCP server independently. Applying and updating access rules in multiple places. Keeping policies in sync as tools or agents change. Managing separate logs and audits for each server.
  - As the number of servers and tools grows, so does the likelihood of policy drift, where different servers enforce slightly different rules. The result is fragmented control, inconsistent permissions, and a higher risk of unauthorized or unintended actions.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Governance and authentication is the missing layer
* **Key Points:**
  - In the context of MCP servers, governance means more than just "who can log in." It's about defining and consistently enforcing the rules for which agents can access which tools, under what conditions, and in which environments.
  - It includes setting limits, ensuring compliance, and maintaining full visibility over every interaction.
  - Authentication is the front door: verifying that the calling agent is who it claims to be before it can invoke any tool.
  - Without a unifying layer, governance and authentication become distributed problems. Policies may differ from server to server. Credentials might be managed inconsistently. And when something needs to change like revoking a tool permission or adjusting a rate limit, it has to be done in multiple places, increasing both operational cost and the chance of mistakes.
  - A centralized governance and authentication layer changes the equation. It gives you one place to define access policies, validate requests, and manage credentials, no matter how many MCP servers are in play.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Centralized governance for MCP servers
* **Key Points:**
  - A centralized governance layer acts as the control plane for all MCP server activity. Instead of each server handling authentication and authorization in isolation, every agent request flows through this central point before it reaches any tool.
  - In this model: Authenticate once – An agent proves its identity to the control plane, which then issues short-lived, scoped credentials for any MCP server it's authorized to use. Govern once – Access policies are defined in a single location and applied uniformly, regardless of where the tools are hosted. Audit once – All tool calls and policy decisions are logged in one system, creating a unified record for compliance, debugging, and analytics.
  - The benefits compound quickly: Consistency – No more policy drift between servers. Speed – Onboarding a new tool or agent means updating one policy store, not five different servers. Security – Short-lived, server-specific credentials reduce the blast radius if a token is compromised. Visibility – A complete view of which agents used which tools, when, and with what parameters.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Governing MCP server tools
* **Key Points:**
  - When orchestration spans multiple servers, tools become the unit of control. Governance works best when each tool is treated as a first-class resource with clear metadata, risk posture, and policy hooks.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

### Make tools addressable and auditable
* **Key Points:**
  - Stable identity: each tool has a canonical name, server identifier, and version.
  - Capability tags: search, write, admin, destructive. Useful for broad policy rules.
  - Provenance: who published the server, when it was last updated, and any attestations.
  - Ownership: a team or owner accountable for lifecycle and incident response.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Classify risk and data domains
* **Key Points:**
  - Risk tiers: read, mutate, destructive. Tie tiers to stricter approval and monitoring.
  - Data domains: customer data, code, finance, HR. Limit cross-domain flows by default.
  - Environment scope: dev, stage, prod. Most tools should not span all three.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Define contracts and guardrails
* **Key Points:**
  - Input and output schemas: structured contracts allow validation and redaction.
  - Allowed arguments: enforce ranges, patterns, and allowlists for high-risk fields.
  - Side-effect flags: mark tools that create, update, or delete to trigger extra checks.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Apply quotas and rate limits
* **Key Points:**
  - Per agent, per tool, per environment controls prevent runaway usage.
  - Burst vs sustained limits keep critical systems responsive during incidents.
  - Approval gates for sensitive actions, with a human in the loop when needed.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Versioning and rollout policy
* **Key Points:**
  - Pin versions for production workflows to ensure repeatability.
  - Staged rollout: dev → stage → prod with automatic rollback on error budgets.
  - Sunset rules: deprecate old tools and enforce migration deadlines.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Observability and audit
* **Key Points:**
  - Uniform telemetry: success, failure, latency, input size, and data domain tags.
  - Traceability: every call includes who, what tool, which server, and why it was allowed.
  - Outcome review: periodic audits on high-risk tools to validate policy efficacy.
  - The result is a shared language for control: tools are discoverable, classifiable, and governable across all servers, without rewriting server-side logic or duplicating policies.
  - Governance also needs to define which agents can use which tools, in which environments, and under what conditions. Without this, even the most carefully classified tools can be misused.
  - By centralizing governance, administrators don't have to define permissions separately for each MCP server. Instead: One policy store defines which agents can use which tools. Scoped credentials are issued dynamically, allowing only the approved calls.
  - The promise of MCP is that a single agent can access a wide variety of tools. In practice, the most valuable workflows span multiple MCP servers.
  - With centralized governance, this becomes safe and seamless: agents authenticate once, policies are applied consistently, and every tool call, no matter which server it lives on, is governed, authorized, and logged through a single control plane.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Benefits of this model
* **Key Points:**
  - Consistency: each tool call, whether on Server A, B, or C, goes through the same policy checks.
  - Safety: no tool is ever invoked without an explicit policy decision, preventing accidental overreach.
  - Efficiency: agents can run multi-server workflows without redundant re-authentication.
  - Traceability: one audit log covers the entire flow, making it clear which tools were invoked, why, and under what authority.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Centralized governance as the next step
* **Key Points:**
  - As MCP adoption grows, the ability to orchestrate across multiple servers will become the norm. Centralized governance provides the foundation for that future, a single layer to handle authentication, policy, and audit across every tool.
  - The next step is standardizing these governance patterns so multi-server workflows are not just possible, but a built-in part of how MCP is used at scale.
  - At Portkey, we're building an MCP Hub — a centralized governance module for MCP that makes this vision real. If you'd like a sneak peek, you can book a demo and see how centralized governance works in practice.
* **Technical Entities (Classes/Functions/APIs):** `Portkey`, `MCP Hub`
* **Code Snippet:** None


---

# Orchestrating multiple MCP servers in a single AI workflowFlexibility to Framework: Building MCP Servers with Controlled Tool Orchestration

## The Challenge – Enforcing Tool Ordering in MCP
* **Key Points:**
  - When you think of MCP, you likely think of choice. Arguably one of the main reasons you may want to use an MCP server, is to allow a Large Language Model (LLM) (through agents) to access a set of tools such as reading from a database, sending an email, or in something along those lines. The MCP framework doesn't provide a native mechanism to enforce the sequence in which tools must be called.
  - Let's take as an example two tools – fetch_weather_data() and send_email(). For the LLM using your MCP server, it is reasonable to think that you may want to enforce that an email that is sent has the current weather included. Or for another example, tools getOrderId() and getOrderDetail(), where the OrderId would be required to subsequently fetch the OrderDetail. Since MCP currently lacks tool ordering preferences, these types of sequential dependencies can be challenging to enforce.
  - MCP tools are designed to be independent functions that an LLM can invoke as needed. There's no built-in concept of "workflow" or "sequence" in the MCP framework itself. Each tool call is treated as a separate operation, with no inherent knowledge of what came before or what should come after. This means that by default, an LLM can technically call your tools in any order it chooses, regardless of the logical workflow you intend.
  - While LLMs excel at flexible decision-making, some scenarios like infrastructure management require strict operational ordering. This presents a unique challenge when building MCP servers: how do you maintain the LLM's natural flexibility while enforcing critical sequential dependencies?
  - When you think of Infrastructure as Code (IaC), you think of repeatability, consistency, versioning, and continuous integration/continuous deployment (CI/CD). Within CI/CD you have a set flow: Pull request is generated; CI/CD pipeline is triggered; Series of steps runs to run linting, security tests, unit tests, end-to-end tests, etc.; A failure in any stage should stop the entire pipeline run
  - This posed a challenge with IaC and LLMs. Generative AI is non-deterministic, meaning the same prompt may not always generate the same exact response. If the result deviates significantly from what it should be, it is considered a hallucination. So, what can be done to guide the LLM on what you want it to do? Let's talk about how this was addressed in the CCAPI MCP server.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `CCAPI MCP server`, `fetch_weather_data()`, `send_email()`, `getOrderId()`, `getOrderDetail()`
* **Code Snippet:** None

## Understanding MCP Tool Discovery and Initialization
* **Key Points:**
  - Before diving into the solution, it's important to understand how MCP servers communicate with AI Agents. During initialization, the MCP protocol follows specific lifecycle phases where capabilities and tools are discovered.
  - The Model Context Protocol defines a structured lifecycle for client-server connections that ensures proper capability negotiation and state management.
  - These phases include: Initialization: Capability negotiation and protocol version agreement; Operation: Normal protocol communication; Shutdown: Graceful termination of the connection
  - The initialization phase establishes protocol compatibility and shares implementation details. This is when an AI Agent learns about available tools through schema definitions and receives instructions for tool usage. This initialization process is crucial to the solution, as it's where AI Agents first discover what tools are available and how they should be used. During this phase, the client sends information about its protocol version, capabilities, and implementation details. This is how tools like Amazon Q CLI receive information about an MCP server's version, available tools, and usage instructions.
  - Note: For more information on the MCP lifecycle, see these docs.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `Amazon Q CLI`
* **Code Snippet:** None

## Solution – Token-Based Tool Orchestration: A New Pattern for AI Agents in MCP
* **Key Points:**
  - MCP presents a specific challenge: tools cannot directly communicate with each other to enforce execution order. The CCAPI MCP server addresses this through a token messenger pattern shown above, where the server generates and controls validation tokens, and the AI Agent (as the MCP client) passes these tokens between tool calls.
  - Core Implementation:
  - Function Enhancement – The mcp.tool() decorator transforms each function into a more capable entity. It wraps the function with a schema that defines required inputs and their validation rules, while preserving detailed documentation through docstrings. Each enhanced function clearly communicates its requirements and provides explicit error messages when dependencies aren't met.
  - Dependency Discovery – During the initialize phase in the MCP lifecycle, the AI Agent (as the MCP client) receives a complete map of all defined tools and their schemas from the MCP server. The LLM, which is part of the AI Agent, uses these schemas to understand dependencies through both parameter descriptions and required input arguments. For instance, when a tool requires a parameter described as "Result from get_aws_session_info()" and defines security_scan_token as a required input argument, the LLM understands it needs both valid tokens before proceeding. This combination of descriptive text and explicit input requirements enables the AI Agent to execute sequences like get_aws_session_info() → generate_infrastructure_code() → run_checkov() → create_resource().
  - Token Validation Control –The server generates and controls all workflow tokens through a unified server-side storage system (_workflow_store). Each tool in the workflow generates cryptographically secure tokens, and these tokens are stored server-side with their associated data.
  - The AI Agent maintains these tokens in its conversation context throughout the workflow, passing them between tool calls. For security, each token used by the AI Agent must be validated against the server's token storage. Since these tokens are short-lived, they are stored in memory (RAM) and are actively managed by the MCP server, which deletes tokens after use to maintain freshness. Any remaining tokens are automatically cleared when the server process ends or restarts. If a token doesn't exist in the server's storage (either because it's invalid or already consumed), the operation fails immediately with an error. This validation is uniform across all token types, ensuring the AI Agent cannot create or modify tokens.
  - As the workflow progresses, tools consume existing tokens and generate new ones. For example, when explain() receives a properties_token, it first validates it exists and matches what is in _workflow_store, then consumes it and generates a new explained_properties_token. This creates a cryptographically secure chain of operations that enforces the workflow sequence (generate → scan → create), with server-side validation at every step.
  - The result is a predictable workflow system with strong security controls – tokens must be generated by the server and validated against server-side storage at each step, helping ensure the integrity of the infrastructure management process. This approach provides robust workflow enforcement within the confines of the current functionality of the FastMCP framework. While explicit schema-defined dependencies like @mcp.tool(depends_on=["run_checkov"]) as mentioned in this GitHub Issue would be ideal and could hopefully be added in future FastMCP versions, the current token-based approach with descriptive parameter names and clear validation provides reliable tool ordering that LLMs consistently follow without confusion.
* **Technical Entities (Classes/Functions/APIs):** `CCAPI MCP server`, `mcp.tool()`, `FastMCP`, `_workflow_store`, `get_aws_session_info()`, `generate_infrastructure_code()`, `run_checkov()`, `create_resource()`
* **Code Snippet:** None

## Potential Limitations and Solutions
* **Key Points:**
  - Session Management – When an AI Agent's session ends or refreshes, any in-progress workflows must be restarted. This is by design – tokens are meant to be short-lived and tied to specific workflow sequences. AWS credentials naturally expire within hours as part of standard security practices, providing a natural boundary for workflow sessions.
  - Concurrent Workflows – Each AI Agent interaction operates independently, which is appropriate for maintaining security boundaries between different workflow instances. While this means each session starts fresh, it ensures clean separation between different infrastructure operations.
  - Implementation Options – For organizations requiring workflow persistence, traditional database storage could maintain session state between restarts. However, since tokens are designed to be short-lived security controls, most implementations can rely on the default in-memory storage with natural session boundaries.
  - The token messenger pattern provides a solid foundation for secure workflow orchestration, with its intentionally ephemeral tokens ensuring proper tool sequencing and data integrity during infrastructure operations.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## The Future of MCP
* **Key Points:**
  - While the above solution works, this process made me think about the future of MCP and how it can and should continue to grow. There are many updates to the framework I've seen recently, and it's great to see activity. For Agentic AI in general, there are strong signs that the future of agentic platforms may be more deterministic in nature, as highlighted by Claude Code's new support for lifecycle hooks. Per their docs, "Hooks provide deterministic control over Claude Code's behavior, ensuring certain actions always happen rather than relying on the LLM to choose to run them." For IaC and other deterministic technologies that it is desired to integrate AI with, this is essential for wide-scale adoption.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `Claude Code`
* **Code Snippet:** None

## Conclusion
* **Key Points:**
  - The journey of Model Control Protocol (MCP) and this new frontier of leveraging AI for managing cloud infrastructure continues to evolve, presenting both opportunities and challenges in the world of cloud computing and artificial intelligence. Current approaches using prompt loading and parameter dependencies have helped address initial challenges around tool ordering and security protocols, demonstrating how MCP can be effectively used in enterprise applications.
  - While the current implementation using workflow tokens and validation checks provides a functional solution, we continue to explore ways to enhance the protocol's capabilities. For those interested in contributing to MCP's evolution, you can find our proposals for protocol improvements, including enhanced dependency management, in the modelcontextprotocol GitHub org as well as in the FastMCP GitHub repository.
  - If you'd like to learn more about the AWS Cloud Control API MCP server mentioned in this blog, check out the documentation and GitHub repo. If you'd like to get hands on with it and other AWS MCP servers, check out this AWS workshop. Happy vibe coding my friends.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `FastMCP`, `AWS Cloud Control API MCP server`
* **Code Snippet:** None