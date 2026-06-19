---
aliases:
  - Tools
Source 1: https://www.anthropic.com/engineering/code-execution-with-mcp
Source 2: https://blog.cloudflare.com/code-mode/
---
# Code execution with MCP: Building more efficient agents

## Excessive token consumption from tools makes agents less efficient
* **Key Points:**
  - As MCP usage scales, there are two common patterns that can increase agent cost and latency: Tool definitions overload the context window; Intermediate tool results consume additional tokens.
  - 1. Tool definitions overload the context window: Most MCP clients load all tool definitions upfront directly into context, exposing them to the model using a direct tool-calling syntax.
  - Tool descriptions occupy more context window space, increasing response time and costs. In cases where agents are connected to thousands of tools, they'll need to process hundreds of thousands of tokens before reading a request.
  - 2. Intermediate tool results consume additional tokens: Most MCP clients allow models to directly call MCP tools.
  - Every intermediate result must pass through the model. In this example, the full call transcript flows through twice. For a 2-hour sales meeting, that could mean processing an additional 50,000 tokens. Even larger documents may exceed context window limits, breaking the workflow.
  - With large documents or complex data structures, models may be more likely to make mistakes when copying data between tool calls.
  - The MCP client loads tool definitions into the model's context window and orchestrates a message loop where each tool call and result passes through the model between operations.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `gdrive.getDocument`, `salesforce.updateRecord`
* **Code Snippet:**
```
gdrive.getDocument
     Description: Retrieves a document from Google Drive
     Parameters:
                documentId (required, string): The ID of the document to retrieve
                fields (optional, string): Specific fields to return
     Returns: Document object with title, body content, metadata, permissions, etc.
```
```
salesforce.updateRecord
    Description: Updates a record in Salesforce
    Parameters:
               objectType (required, string): Type of Salesforce object (Lead, Contact,      Account, etc.)
               recordId (required, string): The ID of the record to update
               data (required, object): Fields to update with their new values
     Returns: Updated record object with confirmation
```
```
TOOL CALL: gdrive.getDocument(documentId: "abc123")
        → returns "Discussed Q4 goals...\n[full transcript text]"
           (loaded into model context)

TOOL CALL: salesforce.updateRecord(
			objectType: "SalesMeeting",
			recordId: "00Q5f000001abcXYZ",
  			data: { "Notes": "Discussed Q4 goals...\n[full transcript text written out]" }
		)
		(model needs to write entire transcript into context again)
```

## Code execution with MCP improves context efficiency
* **Key Points:**
  - With code execution environments becoming more common for agents, a solution is to present MCP servers as code APIs rather than direct tool calls. The agent can then write code to interact with MCP servers. This approach addresses both challenges: agents can load only the tools they need and process data in the execution environment before passing results back to the model.
  - There are a number of ways to do this. One approach is to generate a file tree of all available tools from connected MCP servers.
  - Then each tool corresponds to a file.
  - Our Google Drive to Salesforce example above becomes the code.
  - The agent discovers tools by exploring the filesystem: listing the ./servers/ directory to find available servers (like google-drive and salesforce), then reading the specific tool files it needs (like getDocument.ts and updateRecord.ts) to understand each tool's interface. This lets the agent load only the definitions it needs for the current task. This reduces the token usage from 150,000 tokens to 2,000 tokens—a time and cost saving of 98.7%.
  - Cloudflare published similar findings, referring to code execution with MCP as "Code Mode." The core insight is the same: LLMs are adept at writing code and developers should take advantage of this strength to build agents that interact with MCP servers more efficiently.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:**
```
servers
├── google-drive
│   ├── getDocument.ts
│   ├── ... (other tools)
│   └── index.ts
├── salesforce
│   ├── updateRecord.ts
│   ├── ... (other tools)
│   └── index.ts
└── ... (other servers)
```
```typescript
// ./servers/google-drive/getDocument.ts
import { callMCPTool } from "../../../client.js";

interface GetDocumentInput {
  documentId: string;
}

interface GetDocumentResponse {
  content: string;
}

/* Read a document from Google Drive */
export async function getDocument(input: GetDocumentInput): Promise<GetDocumentResponse> {
  return callMCPTool<GetDocumentResponse>('google_drive__get_document', input);
}
```
```typescript
// Read transcript from Google Docs and add to Salesforce prospect
import * as gdrive from './servers/google-drive';
import * as salesforce from './servers/salesforce';

const transcript = (await gdrive.getDocument({ documentId: 'abc123' })).content;
await salesforce.updateRecord({
  objectType: 'SalesMeeting',
  recordId: '00Q5f000001abcXYZ',
  data: { Notes: transcript }
});
```

## Benefits of code execution with MCP
* **Key Points:**
  - Code execution with MCP enables agents to use context more efficiently by loading tools on demand, filtering data before it reaches the model, and executing complex logic in a single step. There are also security and state management benefits to using this approach.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

### Progressive disclosure
* **Key Points:**
  - Models are great at navigating filesystems. Presenting tools as code on a filesystem allows models to read tool definitions on-demand, rather than reading them all up-front.
  - Alternatively, a search_tools tool can be added to the server to find relevant definitions. For example, when working with the hypothetical Salesforce server used above, the agent searches for "salesforce" and loads only those tools that it needs for the current task. Including a detail level parameter in the search_tools tool that allows the agent to select the level of detail required (such as name only, name and description, or the full definition with schemas) also helps the agent conserve context and find tools efficiently.
* **Technical Entities (Classes/Functions/APIs):** `search_tools`
* **Code Snippet:** None

### Context efficient tool results
* **Key Points:**
  - When working with large datasets, agents can filter and transform results in code before returning them.
  - The agent sees five rows instead of 10,000. Similar patterns work for aggregations, joins across multiple data sources, or extracting specific fields—all without bloating the context window.
* **Technical Entities (Classes/Functions/APIs):** `gdrive.getSheet`
* **Code Snippet:**
```javascript
// Without code execution - all rows flow through context
TOOL CALL: gdrive.getSheet(sheetId: 'abc123')
        → returns 10,000 rows in context to filter manually

// With code execution - filter in the execution environment
const allRows = await gdrive.getSheet({ sheetId: 'abc123' });
const pendingOrders = allRows.filter(row => 
  row["Status"] === 'pending'
);
console.log(`Found ${pendingOrders.length} pending orders`);
console.log(pendingOrders.slice(0, 5)); // Only log first 5 for review
```

### More powerful and context-efficient control flow
* **Key Points:**
  - Loops, conditionals, and error handling can be done with familiar code patterns rather than chaining individual tool calls.
  - This approach is more efficient than alternating between MCP tool calls and sleep commands through the agent loop.
  - Additionally, being able to write out a conditional tree that gets executed also saves on "time to first token" latency: rather than having to wait for a model to evaluate an if-statement, the agent can let the code execution environment do this.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:**
```javascript
let found = false;
while (!found) {
  const messages = await slack.getChannelHistory({ channel: 'C123456' });
  found = messages.some(m => m.text.includes('deployment complete'));
  if (!found) await new Promise(r => setTimeout(r, 5000));
}
console.log('Deployment notification received');
```

### Privacy-preserving operations
* **Key Points:**
  - When agents use code execution with MCP, intermediate results stay in the execution environment by default. This way, the agent only sees what you explicitly log or return, meaning data you don't wish to share with the model can flow through your workflow without ever entering the model's context.
  - For even more sensitive workloads, the agent harness can tokenize sensitive data automatically.
  - The MCP client intercepts the data and tokenizes PII before it reaches the model.
  - Then, when the data is shared in another MCP tool call, it is untokenized via a lookup in the MCP client. The real email addresses, phone numbers, and names flow from Google Sheets to Salesforce, but never through the model. This prevents the agent from accidentally logging or processing sensitive data. You can also use this to define deterministic security rules, choosing where data can flow to and from.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:**
```javascript
const sheet = await gdrive.getSheet({ sheetId: 'abc123' });
for (const row of sheet.rows) {
  await salesforce.updateRecord({
    objectType: 'Lead',
    recordId: row.salesforceId,
    data: { 
      Email: row.email,
      Phone: row.phone,
      Name: row.name
    }
  });
}
console.log(`Updated ${sheet.rows.length} leads`);
```
```javascript
// What the agent would see, if it logged the sheet.rows:
[
  { salesforceId: '00Q...', email: '[EMAIL_1]', phone: '[PHONE_1]', name: '[NAME_1]' },
  { salesforceId: '00Q...', email: '[EMAIL_2]', phone: '[PHONE_2]', name: '[NAME_2]' },
  ...
]
```

### State persistence and skills
* **Key Points:**
  - Code execution with filesystem access allows agents to maintain state across operations. Agents can write intermediate results to files, enabling them to resume work and track progress.
  - Agents can also persist their own code as reusable functions. Once an agent develops working code for a task, it can save that implementation for future use.
  - This ties in closely to the concept of Skills, folders of reusable instructions, scripts, and resources for models to improve performance on specialized tasks. Adding a SKILL.md file to these saved functions creates a structured skill that models can reference and use. Over time, this allows your agent to build a toolbox of higher-level capabilities, evolving the scaffolding that it needs to work most effectively.
  - Note that code execution introduces its own complexity. Running agent-generated code requires a secure execution environment with appropriate sandboxing, resource limits, and monitoring. These infrastructure requirements add operational overhead and security considerations that direct tool calls avoid. The benefits of code execution—reduced token costs, lower latency, and improved tool composition—should be weighed against these implementation costs.
* **Technical Entities (Classes/Functions/APIs):** `Skills`, `SKILL.md`
* **Code Snippet:**
```javascript
const leads = await salesforce.query({ 
  query: 'SELECT Id, Email FROM Lead LIMIT 1000' 
});
const csvData = leads.map(l => `${l.Id},${l.Email}`).join('\n');
await fs.writeFile('./workspace/leads.csv', csvData);

// Later execution picks up where it left off
const saved = await fs.readFile('./workspace/leads.csv', 'utf-8');
```
```typescript
// In ./skills/save-sheet-as-csv.ts
import * as gdrive from './servers/google-drive';
export async function saveSheetAsCsv(sheetId: string) {
  const data = await gdrive.getSheet({ sheetId });
  const csv = data.map(row => row.join(',')).join('\n');
  await fs.writeFile(`./workspace/sheet-${sheetId}.csv`, csv);
  return `./workspace/sheet-${sheetId}.csv`;
}

// Later, in any agent execution:
import { saveSheetAsCsv } from './skills/save-sheet-as-csv';
const csvPath = await saveSheetAsCsv('abc123');
```

## Summary
* **Key Points:**
  - MCP provides a foundational protocol for agents to connect to many tools and systems. However, once too many servers are connected, tool definitions and results can consume excessive tokens, reducing agent efficiency.
  - Although many of the problems here feel novel—context management, tool composition, state persistence—they have known solutions from software engineering. Code execution applies these established patterns to agents, letting them use familiar programming constructs to interact with MCP servers more efficiently. If you implement this approach, we encourage you to share your findings with the MCP community.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Acknowledgments
* **Key Points:**
  - This article was written by Adam Jones and Conor Kelly. Thanks to Jeremy Fox, Jerome Swannack, Stuart Ritchie, Molly Vorwerck, Matt Samuels, and Maggie Vo for feedback on drafts of this post.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None


---

# Code Mode: the better way to use MCP

## What's MCP?
* **Key Points:**
  - For those that aren't familiar: Model Context Protocol is a standard protocol for giving AI agents access to external tools, so that they can directly perform work, rather than just chat with you.
  - Seen another way, MCP is a uniform way to: expose an API for doing something, along with documentation needed for an LLM to understand it, with authorization handled out-of-band.
  - MCP has been making waves throughout 2025 as it has suddenly greatly expanded the capabilities of AI agents.
  - The "API" exposed by an MCP server is expressed as a set of "tools". Each tool is essentially a remote procedure call (RPC) function – it is called with some parameters and returns a response. Most modern LLMs have the capability to use "tools" (sometimes called "function calling"), meaning they are trained to output text in a certain format when they want to invoke a tool. The program invoking the LLM sees this format and invokes the tool as specified, then feeds the results back into the LLM as input.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Anatomy of a tool call
* **Key Points:**
  - Under the hood, an LLM generates a stream of "tokens" representing its output. A token might represent a word, a syllable, some sort of punctuation, or some other component of text.
  - A tool call, though, involves a token that does not have any textual equivalent. The LLM is trained (or, more often, fine-tuned) to understand a special token that it can output that means "the following should be interpreted as a tool call," and another special token that means "this is the end of the tool call." Between these two tokens, the LLM will typically write tokens corresponding to some sort of JSON message that describes the call.
  - Upon seeing these special tokens in the output, the LLM's harness will interpret the sequence as a tool call. After seeing the end token, the harness pauses execution of the LLM. It parses the JSON message and returns it as a separate component of the structured API result. The agent calling the LLM API sees the tool call, invokes the relevant MCP server, and then sends the results back to the LLM API.
  - Different LLMs may use different formats for tool calling, but this is the basic idea.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:**
```
I will use the Weather MCP server to find out the weather in Austin, TX.

I will use the Weather MCP server to find out the weather in Austin, TX.

<|tool_call|>
{
  "name": "get_current_weather",
  "arguments": {
    "location": "Austin, TX, USA"
  }
}
<|end_tool_call|>
```
```
<|tool_result|>
{
  "location": "Austin, TX, USA",
  "temperature": 93,
  "unit": "fahrenheit",
  "conditions": "sunny"
}
<|end_tool_result|>
```

## What's wrong with this?
* **Key Points:**
  - The special tokens used in tool calls are things LLMs have never seen in the wild. They must be specially trained to use tools, based on synthetic training data. They aren't always that good at it. If you present an LLM with too many tools, or overly complex tools, it may struggle to choose the right one or to use it correctly. As a result, MCP server designers are encouraged to present greatly simplified APIs as compared to the more traditional API they might expose to developers.
  - Meanwhile, LLMs are getting really good at writing code. In fact, LLMs asked to write code against the full, complex APIs normally exposed to developers don't seem to have too much trouble with it. Why, then, do MCP interfaces have to "dumb it down"? Writing code and calling tools are almost the same thing, but it seems like LLMs can do one much better than the other?
  - The answer is simple: LLMs have seen a lot of code. They have not seen a lot of "tool calls". In fact, the tool calls they have seen are probably limited to a contrived training set constructed by the LLM's own developers, in order to try to train it. Whereas they have seen real-world code from millions of open source projects.
  - Making an LLM perform tasks with tool calling is like putting Shakespeare through a month-long class in Mandarin and then asking him to write a play in it. It's just not going to be his best work.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## But MCP is still useful, because it is uniform
* **Key Points:**
  - MCP is designed for tool-calling, but it doesn't actually have to be used that way.
  - The "tools" that an MCP server exposes are really just an RPC interface with attached documentation. We don't really have to present them as tools. We can take the tools, and turn them into a programming language API instead.
  - But why would we do that, when the programming language APIs already exist independently? Almost every MCP server is just a wrapper around an existing traditional API – why not expose those APIs?
  - Well, it turns out MCP does something else that's really useful: It provides a uniform way to connect to and learn about an API.
  - An AI agent can use an MCP server even if the agent's developers never heard of the particular MCP server, and the MCP server's developers never heard of the particular agent. This has rarely been true of traditional APIs in the past. Usually, the client developer always knows exactly what API they are coding for. As a result, every API is able to do things like basic connectivity, authorization, and documentation a little bit differently.
  - This uniformity is useful even when the AI agent is writing code. We'd like the AI agent to run in a sandbox such that it can only access the tools we give it. MCP makes it possible for the agentic framework to implement this, by handling connectivity and authorization in a standard way, independent of the AI code. We also don't want the AI to have to search the Internet for documentation; MCP provides it directly in the protocol.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## OK, how does it work?
* **Key Points:**
  - We have already extended the Cloudflare Agents SDK to support this new model!
  - You can wrap the tools and prompt with the codemode helper, and use them in your app.
  - With this change, your app will now start generating and running code that itself will make calls to the tools you defined, MCP servers included. We will introduce variants for other libraries in the very near future. Read the docs for more details and examples.
* **Technical Entities (Classes/Functions/APIs):** `Cloudflare Agents SDK`, `codemode`
* **Code Snippet:**
```javascript
const stream = streamText({
  model: openai("gpt-5"),
  system: "You are a helpful assistant",
  messages: [
    { role: "user", content: "Write a function that adds two numbers" }
  ],
  tools: {
    // tool definitions 
  }
})
```
```javascript
import { codemode } from "agents/codemode/ai";

const {system, tools} = codemode({
  system: "You are a helpful assistant",
  tools: {
    // tool definitions 
  },
  // ...config
})

const stream = streamText({
  model: openai("gpt-5"),
  system,
  tools,
  messages: [
    { role: "user", content: "Write a function that adds two numbers" }
  ]
})
```

## Converting MCP to TypeScript
* **Key Points:**
  - When you connect to an MCP server in "code mode", the Agents SDK will fetch the MCP server's schema, and then convert it into a TypeScript API, complete with doc comments based on the schema.
  - For example, connecting to the MCP server at https://gitmcp.io/cloudflare/agents, will generate a TypeScript definition.
  - This TypeScript is then loaded into the agent's context. Currently, the entire API is loaded, but future improvements could allow an agent to search and browse the API more dynamically – much like an agentic coding assistant would.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `Agents SDK`
* **Code Snippet:**
```typescript
interface FetchAgentsDocumentationInput {
  [k: string]: unknown;
}
interface FetchAgentsDocumentationOutput {
  [key: string]: any;
}

interface SearchAgentsDocumentationInput {
  /**
   * The search query to find relevant documentation
   */
  query: string;
}
interface SearchAgentsDocumentationOutput {
  [key: string]: any;
}

interface SearchAgentsCodeInput {
  /**
   * The search query to find relevant code files
   */
  query: string;
  /**
   * Page number to retrieve (starting from 1). Each page contains 30
   * results.
   */
  page?: number;
}
interface SearchAgentsCodeOutput {
  [key: string]: any;
}

interface FetchGenericUrlContentInput {
  /**
   * The URL of the document or page to fetch
   */
  url: string;
}
interface FetchGenericUrlContentOutput {
  [key: string]: any;
}

declare const codemode: {
  /**
   * Fetch entire documentation file from GitHub repository:
   * cloudflare/agents. Useful for general questions. Always call
   * this tool first if asked about cloudflare/agents.
   */
  fetch_agents_documentation: (
    input: FetchAgentsDocumentationInput
  ) => Promise<FetchAgentsDocumentationOutput>;

  /**
   * Semantically search within the fetched documentation from
   * GitHub repository: cloudflare/agents. Useful for specific queries.
   */
  search_agents_documentation: (
    input: SearchAgentsDocumentationInput
  ) => Promise<SearchAgentsDocumentationOutput>;

  /**
   * Search for code within the GitHub repository: "cloudflare/agents"
   * using the GitHub Search API (exact match). Returns matching files
   * for you to query further if relevant.
   */
  search_agents_code: (
    input: SearchAgentsCodeInput
  ) => Promise<SearchAgentsCodeOutput>;

  /**
   * Generic tool to fetch content from any absolute URL, respecting
   * robots.txt rules. Use this to retrieve referenced urls (absolute
   * urls) that were mentioned in previously fetched documentation.
   */
  fetch_generic_url_content: (
    input: FetchGenericUrlContentInput
  ) => Promise<FetchGenericUrlContentOutput>;
};
```

## Running code in a sandbox
* **Key Points:**
  - Instead of being presented with all the tools of all the connected MCP servers, our agent is presented with just one tool, which simply executes some TypeScript code.
  - The code is then executed in a secure sandbox. The sandbox is totally isolated from the Internet. Its only access to the outside world is through the TypeScript APIs representing its connected MCP servers.
  - These APIs are backed by RPC invocation which calls back to the agent loop. There, the Agents SDK dispatches the call to the appropriate MCP server.
  - The sandboxed code returns results to the agent in the obvious way: by invoking console.log(). When the script finishes, all the output logs are passed back to the agent.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `Agents SDK`
* **Code Snippet:** None

## Dynamic Worker loading: no containers here
* **Key Points:**
  - This new approach requires access to a secure sandbox where arbitrary code can run. So where do we find one? Do we have to run containers? Is that expensive?
  - No. There are no containers. We have something much better: isolates.
  - The Cloudflare Workers platform has always been based on V8 isolates, that is, isolated JavaScript runtimes powered by the V8 JavaScript engine.
  - Isolates are far more lightweight than containers. An isolate can start in a handful of milliseconds using only a few megabytes of memory.
  - Isolates are so fast that we can just create a new one for every piece of code the agent runs. There's no need to reuse them. There's no need to prewarm them. Just create it, on demand, run the code, and throw it away. It all happens so fast that the overhead is negligible; it's almost as if you were just eval()ing the code directly. But with security.
* **Technical Entities (Classes/Functions/APIs):** `Cloudflare Workers`, `V8 isolates`, `Worker Loader API`
* **Code Snippet:** None

### The Worker Loader API
* **Key Points:**
  - Until now, though, there was no way for a Worker to directly load an isolate containing arbitrary code. All Worker code instead had to be uploaded via the Cloudflare API, which would then deploy it globally, so that it could run anywhere. That's not what we want for Agents! We want the code to just run right where the agent is.
  - To that end, we've added a new API to the Workers platform: the Worker Loader API. With it, you can load Worker code on-demand.
  - You can start playing with this API right now when running workerd locally with Wrangler (check out the docs), and you can sign up for beta access to use it in production.
* **Technical Entities (Classes/Functions/APIs):** `Worker Loader API`, `workerd`, `Wrangler`
* **Code Snippet:**
```javascript
// Gets the Worker with the given ID, creating it if no such Worker exists yet.
let worker = env.LOADER.get(id, async () => {
  // If the Worker does not already exist, this callback is invoked to fetch
  // its code.

  return {
    compatibilityDate: "2025-06-01",

    // Specify the worker's code (module files).
    mainModule: "foo.js",
    modules: {
      "foo.js":
        "export default {\n" +
        "  fetch(req, env, ctx) { return new Response('Hello'); }\n" +
        "}\n",
    },

    // Specify the dynamic Worker's environment (`env`).
    env: {
      // It can contain basic serializable data types...
      SOME_NUMBER: 123,

      // ... and bindings back to the parent worker's exported RPC
      // interfaces, using the new `ctx.exports` loopback bindings API.
      SOME_RPC_BINDING: ctx.exports.MyBindingImpl({props})
    },

    // Redirect the Worker's `fetch()` and `connect()` to proxy through
    // the parent worker, to monitor or filter all Internet access. You
    // can also block Internet access completely by passing `null`.
    globalOutbound: ctx.exports.OutboundProxy({props}),
  };
});

// Now you can get the Worker's entrypoint and send requests to it.
let defaultEntrypoint = worker.getEntrypoint();
await defaultEntrypoint.fetch("http://example.com");

// You can get non-default entrypoints as well, and specify the
// `ctx.props` value to be delivered to the entrypoint.
let someEntrypoint = worker.getEntrypoint("SomeEntrypointClass", {
  props: {someProp: 123}
});
```

### Workers are better sandboxes
* **Key Points:**
  - The design of Workers makes it unusually good at sandboxing, especially for this use case, for a few reasons:
  - Faster, cheaper, disposable sandboxes: The Workers platform uses isolates instead of containers. Isolates are much lighter-weight and faster to start up. It takes mere milliseconds to start a fresh isolate, and it's so cheap we can just create a new one for every single code snippet the agent generates. There's no need to worry about pooling isolates for reuse, prewarming, etc. We have not yet finalized pricing for the Worker Loader API, but because it is based on isolates, we will be able to offer it at a significantly lower cost than container-based solutions.
  - Isolated by default, but connected with bindings: Workers are just better at handling isolation. In Code Mode, we prohibit the sandboxed worker from talking to the Internet. The global fetch() and connect() functions throw errors. But on most platforms, this would be a problem. On most platforms, the way you get access to private resources is, you start with general network access. Then, using that network access, you send requests to specific services, passing them some sort of API key to authorize private access. But Workers has always had a better answer. In Workers, the "environment" (env object) doesn't just contain strings, it contains live objects, also known as "bindings". These objects can provide direct access to private resources without involving generic network requests. In Code Mode, we give the sandbox access to bindings representing the MCP servers it is connected to. Thus, the agent can specifically access those MCP servers without having network access in general. Limiting access via bindings is much cleaner than doing it via, say, network-level filtering or HTTP proxies. Filtering is hard on both the LLM and the supervisor, because the boundaries are often unclear: the supervisor may have a hard time identifying exactly what traffic is legitimately necessary to talk to an API. Meanwhile, the LLM may have difficulty guessing what kinds of requests will be blocked. With the bindings approach, it's well-defined: the binding provides a JavaScript interface, and that interface is allowed to be used. It's just better this way.
  - No API keys to leak: An additional benefit of bindings is that they hide API keys. The binding itself provides an already-authorized client interface to the MCP server. All calls made on it go to the agent supervisor first, which holds the access tokens and adds them into requests sent on to MCP. This means that the AI cannot possibly write code that leaks any keys, solving a common security problem seen in AI-authored code today.
* **Technical Entities (Classes/Functions/APIs):** `Worker Loader API`, `MCP`
* **Code Snippet:** None