---
aliases:
  - MCP
Source 1: https://docs.langchain.com/oss/javascript/langchain/mcp
---
## Model Context Protocol (MCP)
* **Key Points:**
  - Model Context Protocol (MCP) is an open protocol that standardizes how applications provide tools and context to LLMs.
  - LangChain agents can use tools defined on MCP servers using the `@langchain/mcp-adapters` library.
* **Technical Entities (Classes/Functions/APIs):** `@langchain/mcp-adapters`, `MultiServerMCPClient`, `createAgent`

## Quickstart
* **Key Points:**
  - `MultiServerMCPClient` is **stateless by default**. Each tool invocation creates a fresh MCP `ClientSession`, executes the tool, and then cleans up.
* **Technical Entities (Classes/Functions/APIs):** `MultiServerMCPClient`, `ChatAnthropic`, `createAgent`, `client.getTools()`, `stdio`, `http`
* **Code Snippet:**
```ts
import { MultiServerMCPClient } from "@langchain/mcp-adapters";
import { ChatAnthropic } from "@langchain/anthropic";
import { createAgent } from "langchain";

const client = new MultiServerMCPClient({
    math: {
        transport: "stdio",  // Local subprocess communication
        command: "node",
        // Replace with absolute path to your math_server.js file
        args: ["/path/to/math_server.js"],
    },
    weather: {
        transport: "http",  // HTTP-based remote server
        // Ensure you start your weather server on port 8000
        url: "http://localhost:8000/mcp",
    },
});

const tools = await client.getTools();
const agent = createAgent({
    model: "claude-sonnet-4-6",
    tools,
});

const mathResponse = await agent.invoke({
    messages: [{ role: "user", content: "what's (3 + 5) x 12?" }],
});

const weatherResponse = await agent.invoke({
    messages: [{ role: "user", content: "what is the weather in nyc?" }],
});
```

## Custom servers
* **Key Points:**
  - To create your own MCP servers, you can use the `@modelcontextprotocol/sdk` library. This library provides a simple way to define tools and run them as servers.
* **Technical Entities (Classes/Functions/APIs):** `@modelcontextprotocol/sdk`, `Server`, `StdioServerTransport`, `SSEServerTransport`, `CallToolRequestSchema`, `ListToolsRequestSchema`
* **Code Snippet:**
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
    CallToolRequestSchema,
    ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
    {
        name: "math-server",
        version: "0.1.0",
    },
    {
        capabilities: {
            tools: {},
        },
    }
);

server.setRequestHandler(ListToolsRequestSchema, async () => {
    return {
        tools: [
        {
            name: "add",
            description: "Add two numbers",
            inputSchema: {
                type: "object",
                properties: {
                    a: {
                        type: "number",
                        description: "First number",
                    },
                    b: {
                        type: "number",
                        description: "Second number",
                    },
                },
                required: ["a", "b"],
            },
        },
        {
            name: "multiply",
            description: "Multiply two numbers",
            inputSchema: {
                type: "object",
                properties: {
                    a: {
                        type: "number",
                        description: "First number",
                    },
                    b: {
                        type: "number",
                        description: "Second number",
                    },
                },
                required: ["a", "b"],
            },
        },
        ],
    };
});

server.setRequestHandler(CallToolRequestSchema, async (request) => {
    switch (request.params.name) {
        case "add": {
            const { a, b } = request.params.arguments as { a: number; b: number };
            return {
                content: [
                {
                    type: "text",
                    text: String(a + b),
                },
                ],
            };
        }
        case "multiply": {
            const { a, b } = request.params.arguments as { a: number; b: number };
            return {
                content: [
                {
                    type: "text",
                    text: String(a * b),
                },
                ],
            };
        }
        default:
            throw new Error(`Unknown tool: ${request.params.name}`);
    }
});

async function main() {
    const transport = new StdioServerTransport();
    await server.connect(transport);
    console.error("Math MCP server running on stdio");
}

main();
```

## Transports
* **Key Points:**
  - MCP supports different transport mechanisms for client-server communication.
  - The `http` transport (also referred to as `streamable-http`) uses HTTP requests for client-server communication.
  - Client launches server as a subprocess and communicates via standard input/output. Best for local tools and simple setups.

### HTTP
* **Technical Entities (Classes/Functions/APIs):** `transport: "sse"`, `url`
* **Code Snippet:**
```typescript
const client = new MultiServerMCPClient({
    weather: {
        transport: "sse",
        url: "http://localhost:8000/mcp",
    },
});
```

### stdio
* **Technical Entities (Classes/Functions/APIs):** `transport: "stdio"`, `command`, `args`
* **Code Snippet:**
```typescript
const client = new MultiServerMCPClient({
    math: {
        transport: "stdio",
        command: "node",
        args: ["/path/to/math_server.js"],
    },
});
```

## Core features
### Tools
* **Key Points:**
  - Tools allow MCP servers to expose executable functions that LLMs can invoke to perform actions—such as querying databases, calling APIs, or interacting with external systems.
  - LangChain converts MCP tools into LangChain tools, making them directly usable in any LangChain agent or workflow.
  - When an MCP tool execution fails (`CallToolResult` with `isError: true`), `@langchain/mcp-adapters` raises a `ToolException`. Wrap tool calls in a try/catch to handle these errors. Unlike the Python adapter, the TypeScript adapter does not return the error to the model as a failed tool message.
* **Technical Entities (Classes/Functions/APIs):** `MultiServerMCPClient`, `client.getTools()`, `createAgent`, `ToolException`

#### Loading tools
* **Key Points:**
  - Use `client.getTools()` to retrieve tools from MCP servers and pass them to your agent:
* **Technical Entities (Classes/Functions/APIs):** `MultiServerMCPClient`, `client.getTools()`, `createAgent`
* **Code Snippet:**
```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";
import { createAgent } from "langchain";

const client = new MultiServerMCPClient({...});
const tools = await client.getTools();
const agent = createAgent({ model: "claude-sonnet-4-6", tools });
```