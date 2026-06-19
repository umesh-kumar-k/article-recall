---
aliases:
  - Resource Templates
Source 1: https://medium.com/@cstroliadavis/building-mcp-servers-315917582ad1
---
# # Building MCP Servers: Part 2 — Extending Resources with Resource Templates


## What are Resource Templates?
* **Key Points:**
  - Resource templates allow you to define dynamic resources using URI patterns. Unlike static resources that have fixed URIs, templates let you create resources whose URIs and content can be generated based on parameters.
  - Think of them like URL patterns in a web framework where the resource is a bit more dynamic an usually based on some tag or id — they let you match and handle whole families of resources using a single definition.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Why Use Resource Templates?
* **Key Points:**
  - Resource templates are powerful when you need to do things like handling dynamic data, generate content on demand or create things like parameter based resources.
  - Here are some examples:
  - Dynamic Data: "users://{userId}" -> User profiles; "products://{sku}" -> Product information; User: "Can you tell me about user 12345?"; AI Assistant: "Looking up user 12345... They joined in 2023 and have made 50 purchases."
  - Generate Content On-Demand: "reports://{year}/{month}" -> Monthly reports; "analytics://{dateRange}" -> Custom analytics; User: "Show me the report for March 2024"; AI Assistant: "Accessing March 2024 report... Revenue was up 15% compared to February."
  - Parameter-Based Resources: "search://{query}" -> Search results; "filter://{type}/{value}" -> Filtered data; User: "Find all transactions above $1000"; AI Assistant: "Using the filter resource... Found 23 transactions matching your criteria."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Organizing Our Code
* **Key Points:**
  - Let's also improve the code structure we built in the last post by separating some of our concerns. First, let's break our handlers out into a new file (handlers.ts) so we won't have as much clutter.
* **Technical Entities (Classes/Functions/APIs):** `ListResourcesRequestSchema`, `ReadResourceRequestSchema`, `ListResourceTemplatesRequestSchema`, `Server`, `StdioServerTransport`
* **Code Snippet:**
```typescript
// src/handlers.ts
import {
  ListResourcesRequestSchema,
  ReadResourceRequestSchema,
  ListResourceTemplatesRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";
import { type Server } from "@modelcontextprotocol/sdk/server/index.js";

export const setupHandlers = (server: Server): void => {
  // List available resources when clients request them
  server.setRequestHandler(ListResourcesRequestSchema, async () => {
    return {
      resources: [
        {
          uri: "hello://world",
          name: "Hello World Message",
          description: "A simple greeting message",
          mimeType: "text/plain",
        },
      ],
    };
  });
  // Return resource content when clients request it
  server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
    if (request.params.uri === "hello://world") {
      return {
        contents: [
          {
            uri: "hello://world",
            text: "Hello, World! This is my first MCP resource.",
          },
        ],
      };
    }
    throw new Error("Resource not found");
  });
};
```
```typescript
// src/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { setupHandlers } from './handlers.js';

const server = new Server(
  {
    name: "hello-mcp",
    version: "1.0.0",
  },
  {
    capabilities: {
      resources: {},
    },
  }
);

setupHandlers(server);

// Start server using stdio transport
const transport = new StdioServerTransport();
await server.connect(transport);
console.info('{"jsonrpc": "2.0", "method": "log", "params": { "message": "Server running..." }}');
```

## Adding the New Resource
* **Key Points:**
  - It's time to add our new resource template.
  - First, let's add our listing so the AI Assistant knows it's there. In src/handlers.js add the following code after the hello://world resource listing (the one with the first parameter of ListResourcesRequestSchema)
  - Next we can add our content handler. This will not require an additional request handler. We will simply add a new check for a request in this format.
* **Technical Entities (Classes/Functions/APIs):** `ListResourcesRequestSchema`, `ListResourceTemplatesRequestSchema`, `ReadResourceRequestSchema`
* **Code Snippet:**
```typescript
export const setupHandlers = (server: Server): void => {
  // Existing "hello://world" resource listing here ...

  // Resource Templates
  server.setRequestHandler(ListResourceTemplatesRequestSchema, async () => ({
    resourceTemplates: [
      {
        greetings: {
          uriTemplate: 'greetings://{name}',
          name: 'Personal Greeting',
          description: 'A personalized greeting message',
          mimeType: 'text/plain',
        },
      },
    ],
  }));

  // Existing "hello://world" resource content here ... 
};
```
```typescript
// Return resource content when clients request it
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  // ... Existing content handler code

  // Template-based resource code
  const greetingExp = /^greetings:\/\/(.+)$/;
  const greetingMatch = request.params.uri.match(greetingExp);
  if (greetingMatch) {
    const name = decodeURIComponent(greetingMatch[1]);
    return {
        contents: [
        {
            uri: request.params.uri,
            text: `Hello, ${name}! Welcome to MCP.`,
        },
      ],
    };
  }

  // ...
});
```

## Understanding the Code
* **Key Points:**
  - Handler Organization: We've moved handlers to a separate file for better organization; setupHandlers function encapsulates all handler setup; Main file stays clean and focused.
  - Template Definition: The ListResourceTemplateRequestSchema handler exposes available templates; The template name format follows RFC 6570 (a URL that uses {text} to express parameterization); Templates include metadata like name and description.
  - Template Handling: The ReadResourceRequestSchema handler now checks for template matches; We're using regex format to extract the name parameter from URI; We generate dynamic content based on parameters.
* **Technical Entities (Classes/Functions/APIs):** `ListResourceTemplateRequestSchema`, `ReadResourceRequestSchema`, `RFC 6570`
* **Code Snippet:** None

## Testing with the Inspector
* **Key Points:**
  - In our last post, we discussed using the MCP Inspector. Launch the inspector, now: npx tsc; npx @modelcontextprotocol/inspector node build/index.js
  - Test the static resource we created last time to make sure it still works: Click "Resources" tab; Find and click "Hello World Message"; You should see the "Hello, World! This is my first MCP resource." message.
  - Test the template: Click "Resource Templates" tab; Find "Personal Greeting"; Type the name "Alice"
* **Technical Entities (Classes/Functions/APIs):** `MCP Inspector`
* **Code Snippet:**
```bash
npx tsc
npx @modelcontextprotocol/inspector node build/index.js
```
```json
{
  "contents": [
    {
      "uri": "greetings://Alice",
      "text": "Hello, Alice! Welcome to MCP."
    }
  ]
}
```

## Testing with Claude Desktop
* **Key Points:**
  - You should not need to update anything this time in Claude, but you may have to reload (also, make sure you've built the service with npx tsc).
  - As in my last post, the Claude Desktop I have for Mac does not seem to support resources yet, but you may be able to try this in other tools that support MCP, such as Cline, you may have to specifically use a model that is aware of MCP, like Sonnet 3.5 from Anthropic.
  - Try these examples (responses may vary):
  - Static resource: User: "What's in the greeting message?"; Claude: "The greeting message says: 'Hello from MCP! This is your first resource.'"
  - Template resource: User: "Can you get a greeting for Alice?"; Claude: "I'll check the personalized greeting... It says: 'Hello, Alice! Welcome to MCP.'"
  - Listing available resources: User: "What resources and templates are available?"; Claude: "The server provides: 1. A static 'Greeting Message' resource; 2. A 'Personal Greeting' template that can create customized greetings for any name"
* **Technical Entities (Classes/Functions/APIs):** `Claude Desktop`, `Cline`, `Sonnet 3.5`
* **Code Snippet:** None

