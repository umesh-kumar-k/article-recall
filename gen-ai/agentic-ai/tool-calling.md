---
highlights: "Tool/Function Calling: Structured mechanism for LLMs to invoke external APIs , databases or services via JSON schemas; enables agents to interact with real-world systems"
tags:
  - tool-calling
Source 1: https://composio.dev/content/ai-agent-tool-calling-guide
Source 2: https://www.scalekit.com/blog/tool-calling-authentication-ai-agents
Source 3: https://www.scalekit.com/blog/oauth-tokens-m2m-authentication
Source 4: https://auth0.com/blog/genai-tool-calling-intro/
Source 5: https://www.anthropic.com/engineering/writing-tools-for-agents
---
# Writing effective tools for agents — with agents

## Writing effective tools for agents — with agents

* **Key Points:**
  - Agents are only as effective as the tools we give them.
  - We share how to write high-quality tools and evaluations, and how you can boost performance by using Claude to optimize its tools for itself.
  - The Model Context Protocol (MCP) can empower LLM agents with potentially hundreds of tools to solve real-world tasks.
* **Technical Entities (Classes/Functions/APIs):** `Model Context Protocol (MCP)`, `Claude`, `Claude Code`, `LLM agents`
* **Code Snippet:** None.

---

## What is a tool?

* **Key Points:**
  - In computing, deterministic systems produce the same output every time given identical inputs, while non-deterministic systems—like agents—can generate varied responses even with the same starting conditions.
  - When we traditionally write software, we're establishing a contract between deterministic systems. For instance, a function call like getWeather("NYC") will always fetch the weather in New York City in the exact same manner every time it is called.
  - Tools are a new kind of software which reflects a contract between deterministic systems and non-deterministic agents.
  - When a user asks "Should I bring an umbrella today?," an agent might call the weather tool, answer from general knowledge, or even ask a clarifying question about location first. Occasionally, an agent might hallucinate or even fail to grasp how to use a tool.
  - This means fundamentally rethinking our approach when writing software for agents: instead of writing tools and MCP servers the way we'd write functions and APIs for other developers or systems, we need to design them for agents.
  - Our goal is to increase the surface area over which agents can be effective in solving a wide range of tasks by using tools to pursue a variety of successful strategies.
  - Fortunately, in our experience, the tools that are most "ergonomic" for agents also end up being surprisingly intuitive to grasp as humans.
* **Technical Entities (Classes/Functions/APIs):** `getWeather()`, `MCP servers`, `APIs`, `LLM agents`
* **Code Snippet:** None.

---

## How to write tools

* **Key Points:**
  - In this section, we describe how you can collaborate with agents both to write and to improve the tools you give them.
  - Start by standing up a quick prototype of your tools and testing them locally. Next, run a comprehensive evaluation to measure subsequent changes. Working alongside agents, you can repeat the process of evaluating and improving your tools until your agents achieve strong performance on real-world tasks.
* **Technical Entities (Classes/Functions/APIs):** `agents`
* **Code Snippet:** None.

---

## Building a prototype

* **Key Points:**
  - It can be difficult to anticipate which tools agents will find ergonomic and which tools they won't without getting hands-on yourself. Start by standing up a quick prototype of your tools.
  - If you're using Claude Code to write your tools (potentially in one-shot), it helps to give Claude documentation for any software libraries, APIs, or SDKs (including potentially the MCP SDK) your tools will rely on.
  - LLM-friendly documentation can commonly be found in flat llms.txt files on official documentation sites (here's our API's).
  - Wrapping your tools in a local MCP server or Desktop extension (DXT) will allow you to connect and test your tools in Claude Code or the Claude Desktop app.
  - To connect your local MCP server to Claude Code, run claude mcp add <name> <command> [args...].
  - To connect your local MCP server or DXT to the Claude Desktop app, navigate to Settings > Developer or Settings > Extensions, respectively.
  - Tools can also be passed directly into Anthropic API calls for programmatic testing.
  - Test the tools yourself to identify any rough edges. Collect feedback from your users to build an intuition around the use-cases and prompts you expect your tools to enable.
* **Technical Entities (Classes/Functions/APIs):** `Claude Code`, `Claude`, `MCP SDK`, `llms.txt`, `MCP server`, `Desktop extension (DXT)`, `Claude Desktop app`, `Anthropic API`
* **Code Snippet:**
```bash
claude mcp add <name> <command> [args...]
```

---

## Running an evaluation

* **Key Points:**
  - Next, you need to measure how well Claude uses your tools by running an evaluation.
  - Start by generating lots of evaluation tasks, grounded in real world uses.
  - We recommend collaborating with an agent to help analyze your results and determine how to improve your tools.
  - See this process end-to-end in our tool evaluation cookbook.
* **Technical Entities (Classes/Functions/APIs):** `Claude`, `tool evaluation cookbook`
* **Code Snippet:** None.

---

## Generating evaluation tasks

* **Key Points:**
  - With your early prototype, Claude Code can quickly explore your tools and create dozens of prompt and response pairs.
  - Prompts should be inspired by real-world uses and be based on realistic data sources and services (for example, internal knowledge bases and microservices).
  - We recommend you avoid overly simplistic or superficial "sandbox" environments that don't stress-test your tools with sufficient complexity.
  - Strong evaluation tasks might require multiple tool calls—potentially dozens.
  - Here are some examples of strong tasks:
  - Schedule a meeting with Jane next week to discuss our latest Acme Corp project. Attach the notes from our last project planning meeting and reserve a conference room.
  - Customer ID 9182 reported that they were charged three times for a single purchase attempt. Find all relevant log entries and determine if any other customers were affected by the same issue.
  - Customer Sarah Chen just submitted a cancellation request. Prepare a retention offer. Determine: (1) why they're leaving, (2) what retention offer would be most compelling, and (3) any risk factors we should be aware of before making an offer.
  - And here are some weaker tasks:
  - Schedule a meeting with jane@acme.corp next week.
  - Search the payment logs for purchase_complete and customer_id=9182.
  - Find the cancellation request by Customer ID 45892.
  - Each evaluation prompt should be paired with a verifiable response or outcome.
  - Your verifier can be as simple as an exact string comparison between ground truth and sampled responses, or as advanced as enlisting Claude to judge the response.
  - Avoid overly strict verifiers that reject correct responses due to spurious differences like formatting, punctuation, or valid alternative phrasings.
  - For each prompt-response pair, you can optionally also specify the tools you expect an agent to call in solving the task, to measure whether or not agents are successful in grasping each tool's purpose during evaluation. However, because there might be multiple valid paths to solving tasks correctly, try to avoid overspecifying or overfitting to strategies.
* **Technical Entities (Classes/Functions/APIs):** `Claude Code`, `Claude`
* **Code Snippet:** None.

---

## Running the evaluation

* **Key Points:**
  - We recommend running your evaluation programmatically with direct LLM API calls.
  - Use simple agentic loops (while-loops wrapping alternating LLM API and tool calls): one loop for each evaluation task. Each evaluation agent should be given a single task prompt and your tools.
  - In your evaluation agents' system prompts, we recommend instructing agents to output not just structured response blocks (for verification), but also reasoning and feedback blocks.
  - Instructing agents to output these before tool call and response blocks may increase LLMs' effective intelligence by triggering chain-of-thought (CoT) behaviors.
  - If you're running your evaluation with Claude, you can turn on interleaved thinking for similar functionality "off-the-shelf". This will help you probe why agents do or don't call certain tools and highlight specific areas of improvement in tool descriptions and specs.
  - As well as top-level accuracy, we recommend collecting other metrics like the total runtime of individual tool calls and tasks, the total number of tool calls, the total token consumption, and tool errors.
  - Tracking tool calls can help reveal common workflows that agents pursue and offer some opportunities for tools to consolidate.
* **Technical Entities (Classes/Functions/APIs):** `LLM API`, `agentic loops`, `chain-of-thought (CoT)`, `Claude`, `interleaved thinking`
* **Code Snippet:** None.

---

## Analyzing results

* **Key Points:**
  - Agents are your helpful partners in spotting issues and providing feedback on everything from contradictory tool descriptions to inefficient tool implementations and confusing tool schemas.
  - However, keep in mind that what agents omit in their feedback and responses can often be more important than what they include. LLMs don't always say what they mean.
  - Observe where your agents get stumped or confused. Read through your evaluation agents' reasoning and feedback (or CoT) to identify rough edges. Review the raw transcripts (including tool calls and tool responses) to catch any behavior not explicitly described in the agent's CoT. Read between the lines; remember that your evaluation agents don't necessarily know the correct answers and strategies.
  - Analyze your tool calling metrics. Lots of redundant tool calls might suggest some rightsizing of pagination or token limit parameters is warranted; lots of tool errors for invalid parameters might suggest tools could use clearer descriptions or better examples.
  - When we launched Claude's web search tool, we identified that Claude was needlessly appending 2025 to the tool's query parameter, biasing search results and degrading performance (we steered Claude in the right direction by improving the tool description).
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `Claude`, `Claude's web search tool`
* **Code Snippet:** None.

---

## Collaborating with agents

* **Key Points:**
  - You can even let agents analyze your results and improve your tools for you.
  - Simply concatenate the transcripts from your evaluation agents and paste them into Claude Code.
  - Claude is an expert at analyzing transcripts and refactoring lots of tools all at once—for example, to ensure tool implementations and descriptions remain self-consistent when new changes are made.
  - In fact, most of the advice in this post came from repeatedly optimizing our internal tool implementations with Claude Code.
  - Our evaluations were created on top of our internal workspace, mirroring the complexity of our internal workflows, including real projects, documents, and messages.
  - We relied on held-out test sets to ensure we did not overfit to our "training" evaluations. These test sets revealed that we could extract additional performance improvements even beyond what we achieved with "expert" tool implementations—whether those tools were manually written by our researchers or generated by Claude itself.
* **Technical Entities (Classes/Functions/APIs):** `Claude Code`, `Claude`
* **Code Snippet:** None.

---

## Principles for writing effective tools

* **Key Points:**
  - In this section, we distill our learnings into a few guiding principles for writing effective tools.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Choosing the right tools for agents

* **Key Points:**
  - More tools don't always lead to better outcomes. A common error we've observed is tools that merely wrap existing software functionality or API endpoints—whether or not the tools are appropriate for agents. This is because agents have distinct "affordances" to traditional software—that is, they have different ways of perceiving the potential actions they can take with those tools
  - LLM agents have limited "context" (that is, there are limits to how much information they can process at once), whereas computer memory is cheap and abundant.
  - Consider the task of searching for a contact in an address book. Traditional software programs can efficiently store and process a list of contacts one at a time, checking each one before moving on. However, if an LLM agent uses a tool that returns ALL contacts and then has to read through each one token-by-token, it's wasting its limited context space on irrelevant information (imagine searching for a contact in your address book by reading each page from top-to-bottom—that is, via brute-force search). The better and more natural approach (for agents and humans alike) is to skip to the relevant page first (perhaps finding it alphabetically).
  - We recommend building a few thoughtful tools targeting specific high-impact workflows, which match your evaluation tasks and scaling up from there.
  - In the address book case, you might choose to implement a search_contacts or message_contact tool instead of a list_contacts tool.
  - Tools can consolidate functionality, handling potentially multiple discrete operations (or API calls) under the hood.
  - For example, tools can enrich tool responses with related metadata or handle frequently chained, multi-step tasks in a single tool call.
  - Here are some examples:
  - Instead of implementing a list_users, list_events, and create_event tools, consider implementing a schedule_event tool which finds availability and schedules an event.
  - Instead of implementing a read_logs tool, consider implementing a search_logs tool which only returns relevant log lines and some surrounding context.
  - Instead of implementing get_customer_by_id, list_transactions, and list_notes tools, implement a get_customer_context tool which compiles all of a customer's recent & relevant information all at once.
  - Make sure each tool you build has a clear, distinct purpose. Tools should enable agents to subdivide and solve tasks in much the same way that a human would, given access to the same underlying resources, and simultaneously reduce the context that would have otherwise been consumed by intermediate outputs.
  - Too many tools or overlapping tools can also distract agents from pursuing efficient strategies. Careful, selective planning of the tools you build (or don't build) can really pay off.
* **Technical Entities (Classes/Functions/APIs):** `LLM agents`, `search_contacts`, `message_contact`, `list_contacts`, `schedule_event`, `search_logs`, `get_customer_context`
* **Code Snippet:** None.

---

## Namespacing your tools

* **Key Points:**
  - Your AI agents will potentially gain access to dozens of MCP servers and hundreds of different tools–including those by other developers. When tools overlap in function or have a vague purpose, agents can get confused about which ones to use.
  - Namespacing (grouping related tools under common prefixes) can help delineate boundaries between lots of tools; MCP clients sometimes do this by default.
  - For example, namespacing tools by service (e.g., asana_search, jira_search) and by resource (e.g., asana_projects_search, asana_users_search), can help agents select the right tools at the right time.
  - We have found selecting between prefix- and suffix-based namespacing to have non-trivial effects on our tool-use evaluations. Effects vary by LLM and we encourage you to choose a naming scheme according to your own evaluations.
  - Agents might call the wrong tools, call the right tools with the wrong parameters, call too few tools, or process tool responses incorrectly. By selectively implementing tools whose names reflect natural subdivisions of tasks, you simultaneously reduce the number of tools and tool descriptions loaded into the agent's context and offload agentic computation from the agent's context back into the tool calls themselves. This reduces an agent's overall risk of making mistakes.
* **Technical Entities (Classes/Functions/APIs):** `MCP servers`, `MCP clients`, `LLM`, `asana_search`, `jira_search`, `asana_projects_search`, `asana_users_search`
* **Code Snippet:** None.

---

## Returning meaningful context from your tools

* **Key Points:**
  - In the same vein, tool implementations should take care to return only high signal information back to agents. They should prioritize contextual relevance over flexibility, and eschew low-level technical identifiers (for example: uuid, 256px_image_url, mime_type). Fields like name, image_url, and file_type are much more likely to directly inform agents' downstream actions and responses.
  - Agents also tend to grapple with natural language names, terms, or identifiers significantly more successfully than they do with cryptic identifiers. We've found that merely resolving arbitrary alphanumeric UUIDs to more semantically meaningful and interpretable language (or even a 0-indexed ID scheme) significantly improves Claude's precision in retrieval tasks by reducing hallucinations.
  - In some instances, agents may require the flexibility to interact with both natural language and technical identifiers outputs, if only to trigger downstream tool calls (for example, search_user(name='jane') → send_message(id=12345)). You can enable both by exposing a simple response_format enum parameter in your tool, allowing your agent to control whether tools return "concise" or "detailed" responses (images below).
  - You can add more formats for even greater flexibility, similar to GraphQL where you can choose exactly which pieces of information you want to receive.
  - Even your tool response structure—for example XML, JSON, or Markdown—can have an impact on evaluation performance: there is no one-size-fits-all solution. This is because LLMs are trained on next-token prediction and tend to perform better with formats that match their training data. The optimal response structure will vary widely by task and agent. We encourage you to select the best response structure based on your own evaluation.
* **Technical Entities (Classes/Functions/APIs):** `Claude`, `search_user()`, `send_message()`, `response_format`, `GraphQL`, `XML`, `JSON`, `Markdown`, `LLM`
* **Code Snippet:**
```enum
enum ResponseFormat {
   DETAILED = "detailed",
   CONCISE = "concise"
}
```

---

## Optimizing tool responses for token efficiency

* **Key Points:**
  - Optimizing the quality of context is important. But so is optimizing the quantity of context returned back to agents in tool responses.
  - We suggest implementing some combination of pagination, range selection, filtering, and/or truncation with sensible default parameter values for any tool responses that could use up lots of context.
  - For Claude Code, we restrict tool responses to 25,000 tokens by default.
  - We expect the effective context length of agents to grow over time, but the need for context-efficient tools to remain.
  - If you choose to truncate responses, be sure to steer agents with helpful instructions. You can directly encourage agents to pursue more token-efficient strategies, like making many small and targeted searches instead of a single, broad search for a knowledge retrieval task. Similarly, if a tool call raises an error (for example, during input validation), you can prompt-engineer your error responses to clearly communicate specific and actionable improvements, rather than opaque error codes or tracebacks.
* **Technical Entities (Classes/Functions/APIs):** `Claude Code`
* **Code Snippet:** None.

---

## Prompt-engineering your tool descriptions

* **Key Points:**
  - We now come to one of the most effective methods for improving tools: prompt-engineering your tool descriptions and specs. Because these are loaded into your agents' context, they can collectively steer agents toward effective tool-calling behaviors.
  - When writing tool descriptions and specs, think of how you would describe your tool to a new hire on your team. Consider the context that you might implicitly bring—specialized query formats, definitions of niche terminology, relationships between underlying resources—and make it explicit. Avoid ambiguity by clearly describing (and enforcing with strict data models) expected inputs and outputs. In particular, input parameters should be unambiguously named: instead of a parameter named user, try a parameter named user_id.
  - With your evaluation you can measure the impact of your prompt engineering with greater confidence. Even small refinements to tool descriptions can yield dramatic improvements. Claude Sonnet 3.5 achieved state-of-the-art performance on the SWE-bench Verified evaluation after we made precise refinements to tool descriptions, dramatically reducing error rates and improving task completion.
  - You can find other best practices for tool definitions in our Developer Guide.
  - If you're building tools for Claude, we also recommend reading about how tools are dynamically loaded into Claude's system prompt.
  - Lastly, if you're writing tools for an MCP server, tool annotations help disclose which tools require open-world access or make destructive changes.
* **Technical Entities (Classes/Functions/APIs):** `Claude Sonnet 3.5`, `SWE-bench Verified`, `Claude`, `MCP server`, `tool annotations`
* **Code Snippet:** None.

---

## Looking ahead

* **Key Points:**
  - To build effective tools for agents, we need to re-orient our software development practices from predictable, deterministic patterns to non-deterministic ones.
  - Through the iterative, evaluation-driven process we've described in this post, we've identified consistent patterns in what makes tools successful: Effective tools are intentionally and clearly defined, use agent context judiciously, can be combined together in diverse workflows, and enable agents to intuitively solve real-world tasks.
  - In the future, we expect the specific mechanisms through which agents interact with the world to evolve—from updates to the MCP protocol to upgrades to the underlying LLMs themselves. With a systematic, evaluation-driven approach to improving tools for agents, we can ensure that as agents become more capable, the tools they use will evolve alongside them.
* **Technical Entities (Classes/Functions/APIs):** `MCP protocol`, `LLM`
* **Code Snippet:** None.


# Tool calling authentication for AI agents

## Tool calling authentication for AI agents

* **Key Points:**
  - Tool calling: AI agents securely call external tools using scoped tokens and delegated authentication and authorization, bypassing login redirects and user sessions. These tool calls interacts with the tool functions and receive tool call results in the form of structured function responses.
  - Agent-specific tokens: Each agent receives a unique, scoped token via one-time user consent, ensuring granular permission control and clear audit trails for agent actions. These tokens are mapped to tool schemas to ensure secure interaction and are tied to agent identity.
  - Token rotation: Tokens are dynamically issued with minimal TTL and auto-rotation, ensuring agents don't hold long-lived credentials vulnerable to compromise. Tool call arguments are securely handled in this process for secure AI agents operating at scale.
  - Credential management: Agents fetch credentials on demand for each tool, avoiding hardcoded secrets and ensuring security across workflows so your secure AI agents remain stateless and auditable. Each tool call ID is logged to track the tool result for future reference.
  - Audit logging: Structured logs with correlation IDs, immutable audit trails, and real-time monitoring ensure compliance and facilitate forensic analysis of agent actions. These logs capture tool call IDs for tracking tool schemas and tool call results, and can be surfaced through an MCP server when tools are exposed via the MCP protocol.
* **Technical Entities (Classes/Functions/APIs):** `MCP server`, `MCP protocol`, `tool schemas`, `tool call ID`
* **Code Snippet:** None.

---

## Why tool calling breaks authentication

* **Key Points:**
  - AI agents can't use standard login flows when calling external APIs
  - Modern AI agents don't just generate text; they take actions. Tool calling is a method by which agents invoke real-world functionality, such as calling APIs, querying databases, posting updates to SaaS applications, and chaining those steps together into workflows.
  - Unlike traditional user-triggered flows, tool-calling agents operate independently, retrieving customer data, enriching profiles, sending alerts, or updating dashboards, all without requiring human input or UI clicks.
  - These actions can be automated with prebuilt tools and model providers that integrate through tool call arguments and structured authentication flows.
  - Autonomous AI agents are moving beyond chat interfaces. They now perform actions across APIs, query databases, and interact with SaaS tools, all without requiring user intervention.
  - To ground the concepts in this guide, let's walk through a hypothetical agent called SupportBot. It's designed for customer operations and automates tasks like pulling CRM data from Salesforce, running embeddings on historical tickets in Pinecone, posting follow-ups in Slack, and updating dashboards in Metabase, all without human involvement at each step.
  - It does this with no human present to initiate access, which means the authentication flows cannot depend on browser redirects.
  - That's a sharp break from traditional app flows. When a user interacts with a ChatGPT plugin, the plugin makes a one-time call to your API, such as fetching a document or triggering a webhook, using the user's session and permissions. But when an agent runs in the background, it doesn't just make one call. It orchestrates a chain: querying a vector database like Weaviate, pushing updates into tools like Notion or Slack, fetching customer profiles from Segment, and logging results into a business intelligence tool like Metabase. And it does all of this without any human present to approve or initiate access.
  - These interactions are facilitated through tool schemas and structured response formats, ensuring consistency.
  - OAuth was designed around human-in-the-loop consent. The user logs in. The browser redirects. The user approves scopes. This works for web apps. For example, when you connect your Google account to a fitness app, a redirect flow grants access to your calendar. But agents don't have browsers. They don't wait for buttons to be clicked. A data enrichment agent won't stop and ask a human before retrieving contact info from Clearbit or updating CRM entries; it needs credentials ahead of time.
  - They need delegated, secure, and autonomous ways to authenticate across tools.
  - This guide breaks down the architectural and security challenges in tool-calling authentication. This guide breaks down how tool-calling agents authenticate safely using delegated credentials, scoped tokens, and secure vaults across APIs, services, and databases, even when no user is present.
* **Technical Entities (Classes/Functions/APIs):** `SupportBot`, `Salesforce`, `Pinecone`, `Slack`, `Metabase`, `ChatGPT plugin`, `Weaviate`, `Notion`, `Segment`, `OAuth`, `tool schemas`
* **Code Snippet:** None.

---

## Understanding tool-calling flows

* **Key Points:**
  - Tool-calling agents interact with multiple APIs, often without user presence. In a tool-calling workflow, an AI agent doesn't just make one API call; it runs multi-step processes across fundamental tools.
  - For example: It might query a vector database like Pinecone or Weaviate to retrieve semantically similar documents. Then fetch customer records from a CRM like Salesforce or HubSpot. Post updates into Slack or Notion to alert internal teams. And log summaries into dashboards using Metabase or Grafana.
  - Each of these tools has its own authentication model, and most weren't built with autonomous agents in mind. These tools respond with tool call results formatted with tool schemas, which can also be exposed via an MCP server for consistent discovery and invocation.
  - That's where tool calling diverges from traditional API usage. In a typical web app, a user clicks a button, and the app makes a single API call using a session token tied to that login. Tool-calling agents don't have that context. They must authenticate and act on their own, across multiple services without user sessions or UIs. Each tool call ID tracks these interactions, helping organise and manage tool call arguments.
  - Each API in that chain may require a different type of authentication: OAuth tokens, API keys, or signed credentials. The agent identity must be known to the credential system so that it can issue the right scoped token at the right time.
* **Technical Entities (Classes/Functions/APIs):** `Pinecone`, `Weaviate`, `Salesforce`, `HubSpot`, `Slack`, `Notion`, `Metabase`, `Grafana`, `MCP server`, `tool schemas`, `tool call ID`, `OAuth`
* **Code Snippet:**
```javascript
async function postSlackUpdate({ agentId, message, vault }) {
  const token = await vault.getCredential(agentId, 'slack-bot-token');

  const res = await fetch('https://slack.com/api/chat.postMessage', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      channel: '#support-updates',
      text: message,
    }),
  });

  const result = await res.json();
  if (!result.ok) throw new Error(`Slack API error: ${result.error}`);

  return result;
}
```

---

## Authentication patterns for tool calling

* **Key Points:**
  - Tool-calling agents can't rely on browser-based login flows. Instead, they use machine-to-machine authentication methods that support autonomous access to external APIs. Depending on the service and the agent's responsibilities, three common patterns emerge.
  - Service-to-service authentication: Some APIs allow agents to authenticate directly using static credentials. This includes API keys, client credentials (in OAuth 2.0), or signed JWTs. These flows don't require user presence and are ideal when the agent is acting on its own behalf, not on behalf of a specific user.
  - Example: An analytics agent pulling usage metrics from an internal metrics API using a pre-issued service token.
  - Delegated authentication: When an agent needs to act on behalf of a user, it requires delegated access. This often uses OAuth access tokens that were initially authorized by a user and later used by the agent. The challenge is issuing these tokens without requiring repeated interaction, while enforcing authentication and authorization boundaries per task.
  - Example: A user grants access to their CRM account once, and the agent stores a refreshable token to pull customer updates or deal statuses later.
  - API key management: Even when APIs use static keys, agents still need a secure way to store and retrieve them. Hardcoding secrets or injecting them via environment variables creates security risks and audit gaps. A secure key vault or token service should handle storage, access control, rotation, and log how authentication and authorization decisions were applied per request.
* **Technical Entities (Classes/Functions/APIs):** `OAuth 2.0`, `JWT`, `API keys`, `key vault`, `token service`
* **Code Snippet:**
```javascript
async function getKey(vaultUrl, agentId, vaultToken) {
  const res = await fetch(`${vaultUrl}/v1/agents/${agentId}/credentials/slack-api-key`, {
    headers: {
      'Authorization': `Bearer ${vaultToken}`,
    },
  });

  if (!res.ok) throw new Error('Failed to retrieve key');
  const { apiKey } = await res.json();
  return apiKey;
}
```

---

## Token-based tool authentication

* **Key Points:**
  - Many third-party APIs like Google, Salesforce, and HubSpot require OAuth tokens. In a typical app, users complete the OAuth flow interactively via a browser. But tool-calling agents can't do that. They operate in the background. No UI. No redirect. No human-in-the-loop. Instead, agents rely on delegated access: a token granted once (via user consent), then reused securely behind the scenes. This delegated model preserves the separation of duties between business logic and credentials for secure AI agents.
  - Instead, agents rely on delegated access: a token is granted once (via user consent), then reused securely behind the scenes. For this to work safely, agents must enforce scoping, handle token lifecycles, and design for failure recovery.
  - Token scoping: Tokens must be scoped narrowly. For example, if an agent only needs to read CRM contacts, the token should not include write permissions, file access, or email scopes. To do this: When initiating the OAuth grant (e.g., via your IdP), define the exact scopes needed. On the server, map each agent task to a minimum required permission set. During delegation, retrieve only tokens pre-scoped for that task.
  - Token lifecycle: Tokens expire. Agents can't be allowed to fail just because a user isn't online. So the refresh logic needs to be completely autonomous. Best practice: store refresh tokens in a secure vault or broker. When the agent needs access: It asks the vault for a valid access token. If expires, the vault silently refreshes it. The agent never sees the refresh token or manages the expiry logic.
* **Technical Entities (Classes/Functions/APIs):** `Google`, `Salesforce`, `HubSpot`, `OAuth`, `IdP`, `vault`, `refresh tokens`
* **Code Snippet:**
```javascript
// Step 1: Define the required OAuth scopes for the agent
const requiredScopes = 'contacts.read crm.deals.read';

// Step 2: Store the refreshable token received from OAuth response in the vault
await vault.storeToken(agentId, 'hubspot', refreshToken, requiredScopes);

// Step 3: Fetch the scoped token from the vault for the agent's task
const scopedToken = await vault.getScopedToken(agentId, 'hubspot', requiredScopes);

// Make an API call with the scoped token
const res = await fetch('https://api.hubapi.com/crm/v3/objects/contacts', {
  headers: { 'Authorization': `Bearer ${scopedToken}` },
});
const data = await res.json();
```

---

## End-to-end implementation example

* **Key Points:**
  - This is a full flow where the agent fetches a valid token for HubSpot, makes a request, and handles expiry.
* **Technical Entities (Classes/Functions/APIs):** `vault`, `HubSpot`
* **Code Snippet:**
```javascript
async function callWithToken(vault, agentId, service, apiUrl) {
  const token = await vault.getScopedToken(agentId, service);

  let res = await fetch(apiUrl, {
    headers: {
      'Authorization': `Bearer ${token}`,
    },
  });

  if (res.status === 401) {
    const newToken = await vault.refreshToken(agentId, service);
    res = await fetch(apiUrl, {
      headers: {
        'Authorization': `Bearer ${newToken}`,
      },
    });
  }

  if (!res.ok) {
    const errText = await res.text();
    throw new Error(`API call failed: ${res.status} ${errText}`);
  }

  return await res.json();
}
```

---

## Managing credentials across multiple tools

* **Key Points:**
  - Agents often need to juggle multiple auth methods across services.
  - Real-world agent workflows rarely interact with just one API. A single task might involve calling Google Calendar, Salesforce, and Notion, each requiring different authentication methods, token formats, or credential types. The agent must coordinate these services securely in real-time. When interacting with multiple services, agents fetch credentials on demand, ensuring that only the correct tool schemas are used for each tool call result.
  - In this example, SupportBot retrieved tool call arguments from a centralized vault, ensuring each tool call ID is tied to the appropriate credential type and scope, maintaining security and proper credential rotation.
  - This complexity compounds fast. Some services expect OAuth tokens scoped per user. Others require static API keys. A few rely on service accounts, signed requests, or mTLS. The agent must coordinate them all securely and in real time.
  - Credential management: Agent workflows often span multiple tools, each with its own authentication method. One agent might query Pinecone with an API key, pull customer data from HubSpot using OAuth, and log results to Metabase using a service token. These methods are not interchangeable, and agents must be able to handle them all. To do this securely, agents need a way to store and retrieve credentials at runtime, without hardcoding secrets or maintaining persistent sessions.
  - How agents manage credentials across services: Credentials are stored in a centralized vault. Each record links an agent to the tool, the credential type, and its scope. This allows the agent to request only what it needs, when it needs it.
* **Technical Entities (Classes/Functions/APIs):** `Google Calendar`, `Salesforce`, `Notion`, `OAuth`, `mTLS`, `Pinecone`, `HubSpot`, `Metabase`, `vault`, `tool schemas`, `tool call ID`
* **Code Snippet:**
```json
{
  "agentId": "support-agent-1",
  "credentials": [
    {
      "service": "pinecone",
      "type": "api_key",
      "source": "env:PINECONE_API_KEY",
      "scope": "read-only"
    },
    {
      "service": "hubspot",
      "type": "oauth_token",
      "source": "vault:hubspot/support-agent-1",
      "scope": "contacts.read"
    },
    {
      "service": "metabase",
      "type": "service_token",
      "source": "vault:metabase/svc-token",
      "scope": "dashboard.write"
    }
  ]
}
```

---

## Agent workflow calling multiple services

* **Key Points:**
  - Below is a sample implementation where an agent retrieves and uses credentials for three different tools. The agent does not store any secrets locally.
* **Technical Entities (Classes/Functions/APIs):** `vault`
* **Code Snippet:**
```javascript
async function runAgentWorkflow(vault, agentId) {
  const pineconeKey = await vault.getCredential(agentId, 'pinecone');
  const hubspotToken = await vault.getCredential(agentId, 'hubspot');
  const metabaseToken = await vault.getCredential(agentId, 'metabase');

  const pineconeResult = await callPinecone(pineconeKey);
  const hubspotResult = await callHubspot(hubspotToken);
  const dashboardUpdate = await callMetabase(metabaseToken);

  return { pineconeResult, hubspotResult, dashboardUpdate };
}
```

---

## Handling authentication failures in multi-tool workflows

* **Key Points:**
  - If a token expires or a key is revoked, the agent should catch that failure without stopping the entire workflow.
* **Technical Entities (Classes/Functions/APIs):** `HubSpot`
* **Code Snippet:**
```javascript
async function callHubspot(token) {
  const res = await fetch('https://api.hubapi.com/crm/v3/objects/contacts', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });

  if (res.status === 401) {
    // Token expired or invalid
    throw new Error('HubSpot token invalid or expired');
  }

  if (!res.ok) {
    throw new Error(`HubSpot API error: ${res.statusText}`);
  }

  return await res.json();
}
```

---

## Security considerations

* **Key Points:**
  - Secure authentication isn't just about getting tokens; it's about controlling and observing their use: When agents handle credentials to access external tools, the risks go beyond token theft. Improper scoping, leaked logs, or invisible usage patterns can lead to silent failures or serious breaches. To prevent this, tool call results are logged with correlation IDs for each tracking, ensuring proper security practices are followed throughout the tool calling process. Designing observability and audit from day one is essential for secure AI agents.
  - By establishing immutable audit trails and using structured logs, agents can safely manage credentials across multiple tools while maintaining real-time monitoring for anomalies.
  - Principle of least privilege: Each credential an agent uses should be scoped to only what's needed for that task. A token that can write to a user's entire workspace should never be used to simply read analytics dashboards. Overbroad permissions widen the blast radius if the agent is compromised.
  - Preventing credential leakage: Secrets must never be logged, echoed, or embedded in error messages. In tool-calling agents, this becomes especially important since failures often involve tokens or keys passed through headers or request bodies. Agents should sanitize logs by stripping out sensitive information before logging. This prevents accidental exposure in shared environments or observability tools.
* **Technical Entities (Classes/Functions/APIs):** `correlation IDs`, `immutable audit trails`
* **Code Snippet:**
```javascript
const apiUrl = 'https://api.example.com/data';
const token = 'Bearer sk_live_1234567890abcdef';

try {
  const res = await fetch(apiUrl, {
    headers: { 'Authorization': token },
  });

  if (!res.ok) throw new Error(`Request failed: ${res.status}`);
  const data = await res.json();
  return data;
} catch (err) {
  const redacted = token.replace(/.(?=.{4})/g, '*'); // e.g., ************cdef
  console.error(`Tool call failed. Auth token used: ${redacted}`);
  throw err;
}
```

---

## Monitoring agent behavior

* **Key Points:**
  - Unusual tool-calling patterns should raise alerts. Spikes in API calls, repeated auth failures, or access outside expected hours may indicate a compromised agent or a misconfigured credential.
  - Security monitoring should log: Which agent accessed what service; When and how often it was called; Whether the call was successful; What credential was used (by ID, not raw value).
  - This supports incident response and post-incident review across your authentication and authorization controls. With scoped access, redacted logs, and real-time monitoring, agents can authenticate safely, even across dozens of tools.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Conclusion: Building authentication for agent-based systems

* **Key Points:**
  - Tool-calling agents like SupportBot represent a shift in how applications interact with external systems. They operate independently, span multiple services, and require secure, delegated access to APIs without user involvement. Traditional authentication flows, especially browser-based OAuth, weren't designed for this model. The result is a growing gap between what agents need to do and what existing auth flows allow.
  - In this guide, we walked through the architecture of agent-initiated tool calls, the differences from user-driven API access, and the key patterns that support safe authentication: service-to-service auth, delegated token handling, secure key management, and scoped token workflows. We also covered how to coordinate authentication across APIs, databases, and SaaS tools, handle errors gracefully, and apply least-privilege principles with proper audit and monitoring.
  - If you're building AI agents that call APIs, this isn't an edge case; it's the foundation. Secure, scalable agent authentication is quickly becoming critical infrastructure. Rather than relying on one-off workarounds, the next wave of agent platforms will need modular, standards-aligned, and auditable auth layers that can grow with the complexity of agent workflows.
  - Start today by abstracting credential handling using Scalekit, eliminating static secrets, and following emerging standards. The sooner you decouple agents from user-centric login models, the faster you can build secure systems that scale.
* **Technical Entities (Classes/Functions/APIs):** `SupportBot`, `OAuth`, `Scalekit`
* **Code Snippet:** None.

---

## FAQ

* **Key Points:**
  - How can agents authenticate with external APIs without exposing secrets or relying on user interaction? Agents should never store long-lived tokens or secrets locally. Instead, use a credential vault that supports runtime token retrieval across tools like databases, CRM platforms, and internal APIs. For user-delegated access, the vault securely stores and refreshes the token. The agent fetches scoped, short-lived credentials on demand, making calls stateless and secure even when users aren't present.
  - What's the right way to handle multiple auth types (OAuth, API keys, service tokens) in one agent workflow? Introduce an abstraction layer between the agent and the credentials. This could be a centralized vault or credential broker that exposes a consistent interface (e.g., getCredential(agentId, serviceName)) while internally resolving the right auth method. This lets secure AI agents remain agnostic to whether a tool requires an OAuth token, an API key, etc. It also preserves clear authentication and authorization boundaries.
  - How does Scalekit Agent Connect handle OAuth token refresh? Scalekit automatically refreshes OAuth tokens in the background using stored refresh tokens. Once a user completes the initial grant, Agent Connect stores the access and refresh tokens securely. When an agent requests access, Scalekit ensures the returned token is fresh; agents never see refresh tokens or need to manage expiry windows themselves. These policies can be combined with tool exposure through an MCP server for uniform discovery and policy enforcement.
  - Can Scalekit help prevent token leakage or over-permissioning? Yes. Scalekit enforces fine-grained access policies per agent and per credential. Agents only receive scoped, short-lived tokens based on the task context. All credentials are redacted in logs, never returned in plaintext outside of request scope, and every request is logged with audit metadata (agent ID, credential type, scope, timestamp).
  - How do I handle token expiration or failure mid-agent workflow? Design agents to treat authentication errors, like a 401 Unauthorized, as recoverable signals. Use a retry logic that can fetch a fresh token from your vault. In multi-step workflows, ensure each API call requests a new token when needed, rather than assuming one token can span the full chain. Scalekit simplifies this by guaranteeing that every request to Agent Connect yields a fresh, scoped credential valid for immediate use.
* **Technical Entities (Classes/Functions/APIs):** `Scalekit`, `Scalekit Agent Connect`, `OAuth`, `MCP server`, `getCredential()`
* **Code Snippet:** None.


# OAuth Tokens Explained: for M2M Authentication

## OAuth Tokens Explained: for M2M Authentication

* **Key Points:**
  - Think payment gateways syncing data with accounting platforms, CI/CD pipelines triggering deployments to cloud environments, or microservices fetching configurations from central stores. All these are examples of machine-to-machine (M2M) interactions, often involving multiple services or two backend services communicating securely within a larger system.
  - Securing these service-to-service communications is critical, especially to ensure secure authentication and protect sensitive data during M2M exchanges, and that's where OAuth 2.0 steps in.
  - Not in its traditional, user-centric form, but in a streamlined, service-first variant: the Client Credentials Flow.
  - With this method, only the authorization tokens are exchanged between backend services over an authorized communication channel, ensuring secure and verified access.
  - This writeup is a deep dive into how OAuth tokens power secure, scoped, and efficient authentication methods for M2M scenarios.
  - We'll walk through the technical foundation of OAuth in the context of backend-to-backend systems, decode how the Client Credentials flow works, break down the anatomy of access tokens (especially JWTs), and explore real-world integration examples using tools like Postman, Node.js, and cloud-native gateways.
* **Technical Entities (Classes/Functions/APIs):** `OAuth 2.0`, `Client Credentials Flow`, `JWTs`, `Postman`, `Node.js`, `cloud-native gateways`
* **Code Snippet:** None.

---

## What is OAuth and why do tokens matter in M2M scenarios?

* **Key Points:**
  - OAuth is best described as a delegated authorization protocol. It enables one system to access another system's protected resources, without needing to share credentials, by issuing short-lived access tokens.
  - These tokens define what a client can access, for how long, and under what conditions.
  - In M2M scenarios, the authentication process is streamlined to allow secure, automated connections between services, enhancing operational efficiency and simplifying integration steps.
  - While OAuth is commonly associated with user authorization (like allowing a third-party app to read your Google Calendar), its principles, including considerations like OAuth token lifetime, extend to non-interactive systems too.
  - This is where M2M, machine-to-machine, authentication comes in. In M2M contexts, two backend systems interact with each other directly.
  - Each service uses its own client credentials, such as a Client ID and Secret, for authentication. There's no user to log in, approve, or provide consent.
  - It's a completely automated exchange, like a backend service pushing data to another API, or a cloud automation tool provisioning resources.
  - OAuth tokens, in this context, act as the security wrapper that governs what services can do, helps enforce zero-trust boundaries, and ensures that backend systems don't overstep their roles. They're lightweight, time-bound, and traceable, making them ideal for securing autonomous system interactions in distributed architectures.
* **Technical Entities (Classes/Functions/APIs):** `OAuth`, `Client ID`, `Secret`, `access tokens`
* **Code Snippet:** None.

---

## Authorization server: The gatekeeper in OAuth

* **Key Points:**
  - In the OAuth 2.0 ecosystem, the authorization server is the central authority that controls access to protected resources.
  - For machine-to-machine (M2M) authentication, it acts as the gatekeeper, ensuring that only trusted machines—with valid client credentials—can obtain an access token.
  - When a client (such as a backend service) wants to access a resource server, it presents its client ID and client secret to the authorization server.
  - The authorization server verifies these credentials, and if everything checks out, issues an OAuth access token.
  - This access token is then used by the client to access protected resources on the resource server, enabling secure machine-to-machine communication.
  - By strictly validating client credentials and issuing time-bound access tokens, the authorization server enforces access control and helps prevent unauthorized access in M2M authentication scenarios.
* **Technical Entities (Classes/Functions/APIs):** `OAuth 2.0`, `authorization server`, `client ID`, `client secret`, `access token`, `resource server`
* **Code Snippet:** None.

---

## Client Credentials Flow: A quick primer

* **Key Points:**
  - At a high level, the flow is straightforward, but what makes it powerful is how it encapsulates trust, scopes, and lifecycle control into a single access token.
  - Unlike using an API key for authentication (Which acts as a static unique identifier and can pose security risks if leaked or left with a long lifespan), the client credentials flow provides more robust lifecycle management and secure authentication.
  - This makes it ideal for automating secure service-to-service communication in cloud-native environments, microservice architectures, or third-party integrations in B2B apps.
  - At a high level, the flow is straightforward, but what makes it powerful is how it encapsulates trust, scopes, and lifecycle control into a single authorization token.
  - Let's walk through how this flow works, step by step:
  - The client authenticates itself to the authorization server. The client (e.g., a backend service or daemon) sends a request to the authorization server's /token endpoint. It includes its client_id and client_secret, usually in the HTTP basic auth header or request body, to prove its identity.
  - The authorization server validates the credentials and returns an access token. If the client's credentials are correct and the request is authorized, the server responds with an access token (often a JWT). This token includes claims like the scope, issuer, expiration time, and other metadata that govern how the token can be used.
  - The client uses the token to access a protected resource. The client now includes the token in the Authorization: Bearer <token> header when calling the target service's API. The target service verifies the token before granting access to the requested resources.
  - Each token is issued for a specific scope, limiting what the client is allowed to do, which enhances security and ensures services only perform actions they're explicitly allowed to.
* **Technical Entities (Classes/Functions/APIs):** `API key`, `client credentials flow`, `authorization server`, `/token endpoint`, `client_id`, `client_secret`, `JWT`, `Authorization: Bearer <token>`
* **Code Snippet:** None.

---

## Token anatomy: What's inside an OAuth access token

* **Key Points:**
  - Once a token is issued in the client credentials flow, it becomes the only credential your client needs to call a protected API.
  - But what's actually inside this token? And how do you interpret or trust it?
  - First, let's distinguish between the common token types:
  - Access token: This is what your service sends in API calls. It authorizes access to a protected resource and is the main focus in M2M use cases.
  - Refresh token: Mostly used in user-based flows to get new access tokens. Refresh tokens play a key role in maintaining secure and efficient access control by allowing short-lived access tokens and enabling token renewal without requiring user re-authentication. Rarely used in M2M, services usually just re-authenticate, but in some M2M scenarios, implementing a token refresh mechanism is important to automate token renewal and ensure seamless access without manual intervention.
  - ID token: Part of OpenID Connect, used to authenticate users. It has no place in the client credentials flow since there is no user involved.
  - Token formats: JWT vs. Opaque
  - JWT (JSON Web Token): Self-contained token with all claims inside. You can decode it and verify the signature locally.
  - Opaque token: A random string that means nothing unless validated against the authorization server.
  - M2M flows often use JWTs because they support stateless validation and performance at scale.
  - JWT Structure: A JWT has three parts, separated by dots: xxxxx.yyyyy.zzzzz
  - Header: Specifies the algorithm and token type.
  - Payload: Contains the claims.
  - Signature: Ensures the token wasn't tampered with.
* **Technical Entities (Classes/Functions/APIs):** `Access token`, `Refresh token`, `ID token`, `OpenID Connect`, `JWT (JSON Web Token)`, `Opaque token`, `authorization server`
* **Code Snippet:**
```javascript
{
	"iss": "https://auth.example.com/",
	"sub": "client-123",
	"aud": "https://api.example.com/",
	"exp": 1713290000,
	"iat": 1713286400,
	"scope": "read:invoices write:invoices",
	"client_id": "client-123"
}
```

---

## How OAuth tokens are issued in Client Credentials Flow

* **Key Points:**
  - Before your service can get a token, you need to set up an identity provider (IdP) to trust it. This setup varies by provider, but the core steps are generally the same.
  - Step 1: Register a Client: In your IdP (Auth0, Okta, Keycloak, etc.), create a new application or client: Assign it a unique client_id and client_secret; Mark it as a machine-to-machine application; Define scopes and assign API permissions (e.g. read:users, write:data)
  - Step 2: Request the Token: Use curl, Postman, or code to request a token
  - Sample Response: access_token: The token your service will use in API calls; token_type: Typically "Bearer", meaning it's passed in the authorization header; expires_in: Token lifespan in seconds (here, 1 hour)
  - Optional security configs: Token lifetime: Most IdPs let you configure how long tokens last (expires_in). Secret rotation: You can periodically rotate client_secret to improve security. Scope enforcement: Configure APIs to validate incoming tokens against required scopes. This restricts what a token can do
* **Technical Entities (Classes/Functions/APIs):** `identity provider (IdP)`, `Auth0`, `Okta`, `Keycloak`, `client_id`, `client_secret`, `curl`, `Postman`, `expires_in`, `Bearer`
* **Code Snippet:**
```shell
curl --request POST \
--url https://auth.example.com/oauth/token \
--header 'Content-Type: application/x-www-form-urlencoded' \  
--data 'grant_type=client_credentials&client_id=abc123&client_secret=xyz789&scope=read:users write:users'
```

---

## Validating access tokens in your API

* **Key Points:**
  - After a token is issued and received by a client, the next critical step is validation, verifying that the token is legitimate, unaltered, and within its allowed scope.
  - Where and how you do this can vary depending on your architecture and the type of token used.
  - Where should you validate? You typically have two options: At the API gateway: Centralized token validation, often used in service meshes or when using tools like Kong, Istio, or AWS API Gateway; Inside each microservice: Each service is responsible for validating incoming tokens, ensuring full control and auditability
  - Token introspection (for opaque tokens): If your tokens are opaque (unreadable to the services), you'll need to validate them using the IdP's introspection endpoint.
  - Validating JWTs: JWTs can be validated locally without making a network call on every request, once the required public key is fetched and cached.
  - To validate a JWT: Verify the signature using the appropriate method: RS256: Use the public key provided by your IdP (often available via a JWKS endpoint). HS256: Use the shared secret. Fetch and rotate public keys: If using RS256, fetch the signing keys from the IdP's JWKS URI and cache them. Periodically refresh the keys to handle key rotation securely. Check standard claims: exp: Has the token expired? iss: Was it issued by your expected issuer? aud: Is it intended for your API?
* **Technical Entities (Classes/Functions/APIs):** `API gateway`, `Kong`, `Istio`, `AWS API Gateway`, `introspection endpoint`, `JWT`, `RS256`, `HS256`, `JWKS endpoint`, `exp`, `iss`, `aud`
* **Code Snippet:**
```javascript
const jwt = require('jsonwebtoken');
const fs = require('fs');
const token = req.headers.authorization?.split(" ")[1];
const publicKey = fs.readFileSync('./public.pem');
try {
	const decoded = jwt.verify(token, publicKey, {
		algorithms: ['RS256'],
		audience: 'https://api.myapp.com',
		issuer: 'https://auth.myidp.com'  
		});  
	console.log(decoded); // token is valid
} catch (err) {  
		console.error("Invalid token", err.message);
	}
```

---

## Practical example: Securing a service-to-service API with OAuth

* **Key Points:**
  - Let's bring it all together with a concrete scenario.
  - Microservice A (Payments) needs to securely call Microservice B (Billing). Both are registered clients in your identity provider. You want to ensure only authorized services can talk to each other using scoped tokens.
  - Step-by-step: Step 1: Register Clients in the Identity Provider: Register Payments and Billing services in your IdP. For the Payments client, allow client_credentials flow. Assign the scope billing.write to the Payments client, allowing it to write data to Billing.
  - Step 2: Get Access Token (from Payments Service)
  - Step 3: Use Token to Call Billing API
  - Optional: Verify Token in Billing Service: In Billing's backend, validate the token as shown earlier using jsonwebtoken or Spring Security, depending on your stack.
* **Technical Entities (Classes/Functions/APIs):** `identity provider (IdP)`, `client_credentials`, `billing.write`, `jsonwebtoken`, `Spring Security`
* **Code Snippet:**
```javascript
//token.js
const axios = require('axios');
require('dotenv').config();
async function getAccessToken() {  
	const res = await axios.post('https://auth.example.com/oauth/token', null, {
		headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
		auth: {
			username: process.env.CLIENT_ID,
			password: process.env.CLIENT_SECRET,
			},
		params: {
			grant_type: 'client_credentials',
			scope: 'billing.write',    
		}  
	});  
	return res.data.access_token;
}
```
```javascript
// callBilling.js
const axios = require('axios');
const { getAccessToken } = require('./token');
async function callBillingAPI() {  
	const token = await getAccessToken();  
	const res = await axios.post('https://billing.internal/api/invoices', {    
	amount: 1000,    
	currency: 'USD',  
	}, {
				headers: {      
					Authorization: `Bearer ${token}`    
				}  
			});  
	console.log('Response from Billing API:', res.data);
}

callBillingAPI();
```
```java
spring:
	security:
		oauth2:
			resourceserver:
				jwt:
					issuer-uri: https://auth.example.com/
```
```java
<dependency>  
<groupId>org.springframework.boot</groupId> 
<artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

---

## Benefits of OAuth 2.0 for M2M authentication

* **Key Points:**
  - OAuth 2.0 brings a host of advantages to machine-to-machine (M2M) authentication, making it the go-to standard for secure backend integrations:
  - Secure access: By leveraging client credentials and access tokens, OAuth 2.0 ensures that only authorized machines can access protected resources, minimizing the risk of unauthorized entry.
  - Standardization: As a widely adopted protocol, OAuth 2.0 makes it easier to integrate with a variety of external services, APIs, and platforms, streamlining interoperability.
  - Flexibility: With support for multiple grant types—including the client credentials grant flow—OAuth 2.0 adapts to different M2M authentication needs, from simple service-to-service calls to complex automation.
  - Scalability: Designed to handle high volumes of authentication requests, OAuth 2.0 is well-suited for large-scale deployments where many machines need to access protected resources simultaneously.
  - These benefits make OAuth 2.0 a robust foundation for granting access and maintaining secure communication between backend services.
* **Technical Entities (Classes/Functions/APIs):** `OAuth 2.0`, `client credentials`, `access tokens`, `client credentials grant flow`
* **Code Snippet:** None.

---

## Best practices for M2M authentication

* **Key Points:**
  - To maximize the security and reliability of your machine-to-machine (M2M) authentication setup, consider these best practices:
  - Use secure communication protocols: Always use TLS/SSL to encrypt data exchanged between clients and the authorization server, ensuring secure communication and protecting sensitive credentials.
  - Rotate client secrets regularly: Periodically update client secrets to reduce the risk of credential compromise and maintain strong access control.
  - Issue short-lived access tokens: Configure your authorization server to generate short-lived OAuth access tokens, limiting the window of opportunity for misuse if a token is leaked.
  - Validate tokens on every request: Ensure that your backend services validate OAuth access tokens on each API call, checking for token integrity, expiration, and proper scope.
  - Monitor and log authentication events: Keep detailed logs of authentication attempts and token usage to quickly detect anomalies and respond to potential security incidents.
  - By following these practices, you can strengthen your M2M authentication and maintain secure, reliable access to protected resources.
* **Technical Entities (Classes/Functions/APIs):** `TLS/SSL`, `authorization server`, `OAuth access tokens`
* **Code Snippet:** None.

---

## Troubleshooting M2M authentication

* **Key Points:**
  - Even with a solid OAuth 2.0 setup, machine-to-machine (M2M) authentication can encounter issues. Here are some common problems and how to address them:
  - Invalid client credentials: Double-check that the client ID and client secret are correct, up-to-date, and properly configured in both your client and authorization server.
  - Token expiration: Monitor the lifetime of your access tokens and ensure your client has a mechanism to request new tokens before the current ones expire, preventing authentication failures.
  - Authorization server errors: Review the authorization server's logs for error messages or exceptions that could indicate misconfiguration or operational issues.
  - Network connectivity issues: Verify that your client can reliably reach the authorization server, and that there are no firewall or DNS issues blocking communication.
  - Invalid token issuer: Make sure your OAuth access token is issued by a trusted authorization server. If you encounter an invalid token error, check the token's issuer claim and validate it against your expected authorization server.
  - By systematically addressing these areas, you can quickly resolve most M2M authentication issues and maintain secure, uninterrupted access between your backend services.
* **Technical Entities (Classes/Functions/APIs):** `OAuth 2.0`, `client ID`, `client secret`, `authorization server`, `access tokens`, `issuer claim`
* **Code Snippet:** None.

---

## Conclusion

* **Key Points:**
  - Machine-to-machine (M2M) authentication, often relying on machine authentication tokens, has become a foundational requirement in modern backend ecosystems, whether it's microservices talking internally, third-party service integrations, or automation pipelines invoking APIs.
  - The OAuth 2.0 client credentials flow was built precisely for these scenarios: secure, no user involvement, and designed for service identity.
  - Now that you've got a solid grasp on OAuth tokens in M2M scenarios, it's time to put it into action. Start by implementing the client credentials flow in one of your internal services—preferably where two backend systems already exchange data without user involvement.
  - Need inspiration? Secure your CI/CD pipeline APIs or inter-microservice calls.
* **Technical Entities (Classes/Functions/APIs):** `OAuth 2.0 client credentials flow`, `CI/CD pipeline APIs`, `inter-microservice calls`
* **Code Snippet:** None.

---

## FAQs

* **Key Points:**
  - 1. How do OAuth tokens work? OAuth tokens are proof of authorization. When a client is authenticated using the correct credentials (in M2M, this is typically client_id + client_secret), the Identity Provider issues an access token. This token is then included in API requests to prove the client has permission to access a specific resource or perform a specific action.
  - 2. How to generate an OAuth token? To generate a token using the client credentials flow: Register your client in the Identity Provider. Make a POST request to the token endpoint with: grant_type=client_credentials; Your client_id and client_secret; The desired scope(s). You'll receive a JSON response with an access token.
  - 3. How do APIs pass in auth tokens? The access token is passed in the HTTP Authorization header of each request: Authorization: Bearer eyJhbGciOi... APIs should extract the token from this header and validate it before allowing access to protected endpoints.
  - 4. What is token exchange in OAuth? Token exchange is an advanced OAuth feature (RFC 8693) that allows a client to trade one token for another, usually with different scopes, audiences, or identity contexts. It's useful in complex, chained service calls where a system needs to act on behalf of a different identity or role downstream. For example: Service A receives a token, exchanges it for a limited-scope token, and passes that to Service B for a more restricted operation.
* **Technical Entities (Classes/Functions/APIs):** `client_id`, `client_secret`, `Identity Provider`, `access token`, `grant_type=client_credentials`, `Authorization: Bearer`, `token exchange`, `RFC 8693`
* **Code Snippet:**
```shell
curl -X POST https://auth.example.com/oauth/token \
-u client_id:client_secret \
-d "grant_type=client_credentials&scope=read:invoices"
```

