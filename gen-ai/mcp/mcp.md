---
aliases:
  - What is Model Context Protocol
Source 1: https://www.databricks.com/blog/what-is-model-context-protocol
---
# What is the Model Context Protocol (MCP)?

## Introduction: Understanding the Model Context Protocol
* **Key Points:**
  - The Model Context Protocol (MCP) is an open standard that enables AI applications to connect seamlessly with external data sources, tools, and systems.
  - Think of the Model Context Protocol as a USB-C port for AI systems—just as a USB-C port standardizes how devices connect to computers, MCP standardizes how AI agents access external resources like databases, APIs, file systems, and knowledge bases.
  - The context protocol addresses a critical challenge in building AI agents: the "N×M integration problem." Without a standardized protocol, each AI application must integrate directly with every external service, creating N×M separate integrations where N represents the number of tools and M represents the number of clients. This approach quickly becomes impossible to scale. The Model Context Protocol MCP solves this by requiring each client and each MCP server to implement the protocol just once, reducing total integrations from N×M to N+M.
  - By enabling AI systems to access real-time data beyond their LLM's training data, MCP helps AI models provide accurate, up-to-date responses rather than relying solely on static training data from their initial learning phase.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## What Is a Model Context Protocol?
* **Key Points:**
  - The Model Context Protocol is an open-source, unified standard for interoperability that enables developers to build context-aware AI applications. MCP complements LLMOps by exposing runtime integration, observability, and governance controls that simplify deployment, monitoring, and lifecycle management of LLM applications.
  - AI applications need access to assets such as local resources, databases, data pipelines (streaming/batch), search engines, calculators, and workflows for prompt conditioning and grounding generation. The context protocol standardizes how applications connect to those assets through a structured way that reduces boilerplate integration code.
  - The scalability issue means that AI models (large language models in particular) typically must rely on pre-existing, static data for training. This can lead to inaccurate or outdated responses because models trained on static datasets need additional updates to incorporate new information. By addressing scalability, MCP enables AI applications to be context aware and provide up-to-date outputs that aren't constrained by the limitations of static training data.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## What Is MCP and Why Is It Used?
* **Key Points:**
  - The Model Context Protocol MCP serves as a standardized way for AI applications to discover and interact with external tools and data sources at runtime. Rather than hardcoding connections to each external service, AI agents using MCP can dynamically discover available tools, understand their capabilities through structured calls, and invoke them with proper tool permissions.
  - MCP is used because it transforms how AI powered tools access information. Traditional AI systems are limited by their training data, which becomes outdated quickly. The context protocol enables developers to build AI agents that can perform tasks using live data from popular enterprise systems, development environments, and other external sources—all through a single, standardized protocol.
  - The open protocol also reduces boilerplate integration code. Instead of writing custom connectors for every new integration, developers implement MCP once on both the client and server sides. This approach is particularly valuable for agentic AI systems that need to autonomously discover and use multiple tools across different contexts.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## What Is MCP vs API?
* **Key Points:**
  - Traditional API Limitations: Traditional APIs expose endpoints with typed parameters that clients must hardcode and update whenever an API changes. Context stitching becomes the client's responsibility because APIs provide minimal semantic guidance about how returned data should be used. An API request typically follows a simple request-response pattern without maintaining state or context between calls.
  - How the Model Context Protocol Differs: MCP defines a different approach from traditional APIs. Rather than hardcoded endpoints, MCP servers expose a machine-readable capability surface discoverable at runtime. AI systems can query available tools, resources, and prompts instead of relying on predefined connections. The Model Context Protocol standardizes resource shapes—documents, database rows, files—reducing serialization complexity so AI models receive relevant context optimized for reasoning.
  - MCP implementations support bidirectional, stateful communication with streaming semantics. This enables MCP servers to push updates and progress notifications directly into an AI agent's context loop, supporting multi-step workflows and partial results that traditional APIs cannot provide natively. This client-server architecture allows for more sophisticated tool usage patterns in agentic systems.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## MCP Versus RAG: Complementary Approaches
* **Key Points:**
  - Retrieval-Augmented Generation (RAG) improves AI accuracy by converting documents into embeddings, storing them in vector databases, and retrieving relevant information during generation. However, RAG typically relies on indexed, static sources from content repositories. The Model Context Protocol provides on-demand access to live APIs, databases, and streams, returning authoritative, up-to-date context when freshness matters.
  - Unlike RAG, which mainly returns read-only context, the context protocol separates resources from tools so AI agents can both fetch data and perform tasks on external systems with controlled schemas. MCP addresses broader integration needs—enabling agentic workflows, multi-turn orchestration, runtime capability discovery, and multi-tenant governance—which RAG does not natively provide.
  - MCP can complement RAG implementations. Organizations can use RAG to index evergreen content for fast retrieval while using the Model Context Protocol for transactional lookups, SQL query execution, and actions requiring the right context from live systems. This hybrid approach provides both speed and accuracy.
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `MCP`
* **Code Snippet:** None

## The Value of Standardization in the MCP Ecosystem
* **Key Points:**
  - As a runtime-discoverable, bidirectional protocol, the Model Context Protocol turns disparate external tools and data into addressable resources and callable actions. A single MCP client can uniformly discover files, database rows, vector snippets, live streams, and API endpoints. Coexisting with indexed RAG caches, MCP offers authoritative, just-in-time lookups and action semantics.
  - The practical result is fewer bespoke connectors, less custom code, faster integrations, and more reliable agentic systems with proper error handling and audit trails. This standardized way of connecting AI assistants to remote resources accelerates development cycles while maintaining enterprise security controls. The MCP ecosystem benefits from this standardization as more MCP server implementations become available for popular enterprise systems.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Core MCP Architecture: Understanding the Client-Server Model
* **Key Points:**
  - The MCP architecture organizes integrations around three key roles—MCP servers, MCP clients, and MCP hosts—connected through persistent communication channels. This client-server architecture enables AI tools to run multi-step, stateful workflows rather than isolated request-response interactions.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

### What MCP Servers Do
* **Key Points:**
  - MCP servers expose data and tools via standardized interfaces and can run in cloud, on-premises, or hybrid environments. Each server publishes a capability surface of named resources, callable tools, prompts, and notification hooks. Resources may include documents, database rows, files, and pipeline outputs.
  - MCP server implementations use JSON-RPC 2.0 methods and notifications, support streaming for long-running operations, and provide machine-readable discovery through the transport layer. This lets MCP hosts and AI models query capabilities at runtime without requiring predefined knowledge of available tools.
  - Popular MCP server implementations connect AI systems to external services like Google Drive, Slack, GitHub, and PostgreSQL databases. These MCP servers handle authentication, data retrieval, and tool execution while presenting a consistent interface through the standardized protocol. Each server in the MCP ecosystem can serve multiple clients simultaneously.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `JSON-RPC 2.0`, `Google Drive`, `Slack`, `GitHub`, `PostgreSQL`
* **Code Snippet:** None

### How MCP Clients Function
* **Key Points:**
  - MCP clients are components within host applications that translate user or model intents into protocol messages. Each client typically maintains a one-to-one connection with an MCP server and manages lifecycle, authentication, and transport details through a structured way.
  - MCP clients serialize requests as structured calls using JSON-RPC, handle asynchronous notifications and partial streams, and present a unified local API to reduce integration complexity. Multiple clients can operate from the same MCP host, each connecting to different MCP servers simultaneously.
  - These clients enable AI agents to interact with external data sources without understanding the implementation details of each external service. The client handles all communication protocols, error handling, and retry logic automatically.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `JSON-RPC`
* **Code Snippet:** None

### The Role of MCP Hosts
* **Key Points:**
  - MCP hosts provide the AI application layer that coordinates MCP clients and server capabilities. Examples include Claude Desktop, Claude Code, AI-powered IDEs, and other platforms where AI agents operate. The MCP host aggregates prompts, conversation state, and client responses to orchestrate multi-tool workflows.
  - The MCP host decides when to call tools, request additional input, or surface notifications. This centralized orchestration enables AI models to work across heterogeneous MCP servers without bespoke per-service code, supporting the MCP ecosystem's goal of universal interoperability in connecting AI assistants to diverse systems.
* **Technical Entities (Classes/Functions/APIs):** `Claude Desktop`, `Claude Code`
* **Code Snippet:** None

### Context Flow and Bidirectional Communication
* **Key Points:**
  - Client-server communication in the Model Context Protocol is bidirectional and message-driven using JSON-RPC 2.0 over the transport layer. MCP clients call methods to fetch resources or invoke tools, while MCP servers return results, stream partial outputs, and send notifications with relevant information.
  - MCP servers can also initiate requests, asking MCP hosts to sample options or elicit user input through function calling mechanisms. This bidirectional capability distinguishes the context protocol from traditional one-way API patterns. MCP's live authoritative lookups complement RAG by supplying just-in-time records with provenance metadata for traceability.
  - Persistent transports preserve message ordering and enable real-time updates, letting AI systems iterate over intermediate outputs and run agentic loops that make autonomous AI agents possible.
* **Technical Entities (Classes/Functions/APIs):** `JSON-RPC 2.0`, `RAG`
* **Code Snippet:** None

## What Are the Requirements for Model Context Protocol?
* **Key Points:**
  - MCP implementations must enforce Transport Layer Security (TLS) for remote transports, strict tool permissions, and scoped credentials to protect against security threats. The protocol requires rate limiting and robust input validation via JSON Schema enforcement on both MCP clients and servers to prevent injection attacks and malformed requests.
  - Audit logging, token rotation, and least-privilege grants are essential requirements for governing long-lived channels. These security measures protect against unauthorized access while preserving the discoverable integration capabilities that the Model Context Protocol enables. Organizations must implement encryption in transit and at rest, masking, and scoped permissions for long-lived MCP channels to ensure data security.
* **Technical Entities (Classes/Functions/APIs):** `TLS`, `JSON Schema`
* **Code Snippet:** None

### Infrastructure and System Requirements
* **Key Points:**
  - Organizations deploying MCP need compute and networking infrastructure that can host large language models, MCP servers, and connected data sources. This includes adequate GPU/CPU capacity, memory, disk I/O, and low-latency network paths between components in the client-server architecture.
  - Cloud platforms should support elastic scaling of model instances and MCP servers. Teams must define autoscaling policies for concurrent streams and long-running operations. The transport layer should support both local STDIO for embedded components and remote streaming channels like HTTP/SSE or WebSocket for distributed deployments.
* **Technical Entities (Classes/Functions/APIs):** `STDIO`, `HTTP/SSE`, `WebSocket`
* **Code Snippet:** None

### Implementation Requirements for MCP Work
* **Key Points:**
  - MCP work requires implementing JSON-RPC 2.0 messaging, discovery endpoints, and resource/tool schemas. MCP servers must publish their capabilities in a machine-readable format through the standard protocol. This enables developers to build discovery-based integrations that support tool discovery without hardcoded connections.
  - Error handling, reconnection strategies, and backpressure management are critical implementation requirements for production reliability. Organizations should implement observability for persistent streams, method latencies, and resource usage using metrics, traces, and logs. Rate limiters, circuit breakers, and quotas protect downstream systems from overload.
* **Technical Entities (Classes/Functions/APIs):** `JSON-RPC 2.0`
* **Code Snippet:** None

## Practical Benefits: Real-Time Data Access and Reduced Hallucination
* **Key Points:**
  - With the Model Context Protocol, AI models retrieve live records, pipeline outputs, API responses, and files on demand rather than relying solely on cached embeddings or static LLM's training data. This grounds responses in current, authoritative data sources and reduces hallucinations where AI systems generate incorrect information.
  - Resources returned at query time include provenance metadata such as source IDs and timestamps, enabling MCP hosts to log origins and make outputs traceable. This transparency is crucial when AI agents perform tasks that require auditability in regulated industries. The context protocol ensures relevant context is always available from authoritative external systems.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

### Support for Agentic AI Workflows
* **Key Points:**
  - Because MCP servers publish resources, tools, and prompts through a standardized way, AI models can discover and call services without hardcoded endpoints. The open standard supports server-initiated elicitation and streaming responses from MCP servers, enabling multi-step reasoning, input clarification, and iteration on partial results.
  - Tools expose JSON Schema-defined inputs/outputs with scoped tool permissions, allowing AI agents to perform controlled actions like creating tickets, executing SQL queries, or running workflows. This autonomous tool discovery, bidirectional interaction, and built-in guardrails provide the foundation for reliable agentic AI across external systems.
  - The Model Context Protocol MCP explicitly enables agentic workflows that rely on dynamic tool discovery and action primitives to perceive, decide, and act across systems. This makes it possible to build AI agents that operate autonomously while maintaining proper governance controls.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `JSON Schema`
* **Code Snippet:** None

### Simplified Development with Standardized Integrations
* **Key Points:**
  - The context protocol enables developers to implement a single server surface that MCP hosts and AI models can reuse with consistent discovery and call semantics. This eliminates separate connectors for common services, reducing the engineering effort required to connect AI assistants to new data sources.
  - Typed resources and JSON Schema reduce custom serialization, validation, and error handling code that would otherwise be necessary. Local STDIO or remote streaming transports let teams choose on-premises, cloud, or hybrid deployments without changing MCP host logic. This flexibility accelerates how teams build AI agents across different development environments.
  - MCP offers a practical way to standardize integrations once rather than building custom adapters for each new integration. This standardized protocol approach benefits the entire MCP ecosystem as more organizations adopt the standard.
* **Technical Entities (Classes/Functions/APIs):** `JSON Schema`, `STDIO`
* **Code Snippet:** None

### Increased Automation Potential for Complex Workflows
* **Key Points:**
  - MCP's persistent, stateful channels enable AI systems to combine lookups, transformations, and side effects across multiple external services in one continuous loop. For long-running operations, MCP servers can stream partial results so AI agents make intermediate decisions, fork workflows, or request human input when needed.
  - Combining indexed retrieval for evergreen content repositories with the Model Context Protocol's on-demand authoritative lookups supports fast, accurate responses. This hybrid approach maintains governance controls while enabling AI powered tools to access both static knowledge bases and dynamic external data sources.
  - The context protocol's support for multi-turn orchestration allows agentic systems to handle complex workflows that require coordination across multiple tools and data sources. This automation potential transforms how organizations deploy AI applications in production environments.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Implementation Best Practices: System Preparation
* **Key Points:**
  - Validate your infrastructure can support LLM hosting, MCP servers, and connected data sources. Ensure adequate GPU/CPU resources, memory allocation, and network bandwidth for the client-server architecture. Choose cloud platforms that support elastic scaling for concurrent users and define autoscaling policies.
  - Standardize secure transports using TLS for all remote connections between MCP clients and servers. Document connection lifecycle management, including reconnection strategies and observable stream health metrics. Implement rate limits, circuit breakers, and quotas to protect downstream external systems from overload.
  - Organizations should standardize streaming channels (HTTP/SSE, WebSocket) plus local STDIO for embedded components. Validate JSON payloads and schema on both server and client to prevent injection attacks and ensure proper error handling throughout the system.
* **Technical Entities (Classes/Functions/APIs):** `TLS`, `HTTP/SSE`, `WebSocket`, `STDIO`
* **Code Snippet:** None

### Leveraging Open-Source Resources in Programming Languages
* **Key Points:**
  - The MCP ecosystem includes community SDKs in multiple programming languages that accelerate client and server development. These SDKs provide established patterns for JSON-RPC messaging, streaming, and schema validation, eliminating the need to reimplement protocol plumbing.
  - Developers can reuse existing MCP server implementations for popular enterprise systems and extend them to domain-specific use cases through the open standard. Building simulators that mimic notifications, long-running streams, and error conditions helps teams test agentic systems before production deployment.
  - Adopt community resources to accelerate MCP work and avoid rebuilding common functionality. These open-source tools enable developers to focus on business logic rather than protocol implementation details.
* **Technical Entities (Classes/Functions/APIs):** `JSON-RPC`
* **Code Snippet:** None

### Integration Strategy for Production Deployment
* **Key Points:**
  - Start with high-impact use cases that demonstrate measurable ROI, such as context-aware AI assistants or automated workflows using AI agents. Limit initial tool scopes and tool permissions, collect telemetry and user feedback, then expand capabilities after stabilizing core functionality.
  - Balance latency and freshness by combining MCP's live lookups with RAG for large static corpora from content repositories. Define SLAs, audit trails, and escalation procedures before broad production rollout. This phased approach reduces risk while building organizational confidence in agentic AI deployments.
  - MCP addresses the need for structured integration plans that scale as more MCP servers and clients are added to the ecosystem. Organizations should document their integration architecture and governance policies early in the deployment process.
* **Technical Entities (Classes/Functions/APIs):** `RAG`
* **Code Snippet:** None

## Common Misconceptions: MCP Is Not Just Another API Framework
* **Key Points:**
  - Reality: The Model Context Protocol standardizes protocol-level integration with persistent context management and dynamic capability discovery. Unlike REST or RPC calls, MCP defines how AI agents discover capabilities, subscribe to streams, and maintain contextual state across interactions through the standard protocol.
  - This standardized protocol means you can build tools once and expose them uniformly to multiple AI agents and model providers through the MCP ecosystem. Rather than treating model context as an ephemeral payload, the context protocol addresses context as a first-class, versioned resource with proper lifecycle management.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

### Tools and Agents Are Different Components
* **Key Points:**
  - Reality: Tools are discrete capabilities exposed through MCP servers—such as database access, file operations, or API integrations. AI agents are decision-making computer programs that discover, orchestrate, and invoke those available tools to perform tasks autonomously.
  - The context protocol enables AI agents to discover tool metadata dynamically, invoke tool interfaces safely with function calling semantics, and integrate outputs into conversations through MCP clients. This separation allows different agentic systems to use the same catalog of tools while tool owners update interfaces independently of agent logic.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

### MCP Manages Comprehensive Data Connectivity
* **Key Points:**
  - Reality: The Model Context Protocol manages comprehensive connectivity to external data sources beyond simple tool usage. It supports streaming notifications, authenticated access to content repositories and vector stores, and consistent semantics for long-running operations and error handling.
  - MCP offers a practical way to unify access to local resources, remote resources, live data queries, and operational actions through a structured way. This unified approach helps governance, observability, and access control scale alongside AI capabilities in enterprise environments. The context protocol handles other tools and external services through a consistent interface.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Future Research Directions and Evolution
* **Key Points:**
  - As the open standard matures, future research directions include enhanced security frameworks for multi-tenant deployments, improved streaming semantics for complex agentic workflows, and standardized patterns for integrating with additional programming languages and development environments.
  - The growing MCP ecosystem continues to expand with new MCP server implementations for emerging external tools and platforms. Community contributions to SDKs, adapters, and reference architectures accelerate adoption while maintaining the protocol's core goal: enabling any AI application to connect with any external service through a standardized way.
  - Organizations exploring the Model Context Protocol should monitor ecosystem developments, contribute to MCP implementations, and participate in working groups shaping how connecting AI assistants to external systems evolves. This collaborative approach ensures MCP implementations remain interoperable as AI systems and external services continue to advance. Future research directions will likely focus on expanding the protocol's capabilities while maintaining its core simplicity.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Conclusion: The Model Context Protocol as Foundation for Modern AI
* **Key Points:**
  - The Model Context Protocol represents a fundamental shift in how AI applications access external data sources and tools. By providing an open protocol for discovery-based integration, the context protocol enables developers to build context-aware AI agents that can perform tasks using live data from popular enterprise systems without extensive boilerplate integration code.
  - The standardized protocol reduces complexity through the client-server architecture, accelerates development cycles, and enables AI systems to move beyond the limitations of static LLM's training data. Through its bidirectional communication between MCP clients and MCP servers, and support for agentic AI workflows, the Model Context Protocol MCP establishes the foundation for more capable, autonomous AI tools across diverse environments.
  - As the MCP ecosystem grows with new MCP server implementations and integrations, organizations can build sophisticated AI agents that discover and orchestrate multiple external services while maintaining proper tool permissions, security controls, and audit trails. This standardized approach to connecting AI systems with external tools and data sources will continue shaping how enterprises deploy production AI applications. The context protocol provides the essential infrastructure that enables developers to build next-generation AI applications with confidence.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None