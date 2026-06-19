---
aliases:
  - Resource List
Source 1: https://dev.to/alexmercedcoder/a-journey-from-ai-to-llms-and-mcp-8-resources-in-mcp-serving-relevant-data-securely-to-llms-2ei
---
# A Journey from AI to LLMs and MCP - 8 - Resources in MCP — Serving Relevant Data Securely to LLMs

## What Are Resources in MCP?
* **Key Points:**
  - Resources represent data that a model or client can read.
  - This might include: Local files (e.g. file:///logs/server.log); Database records (e.g. postgres://db/customers); Web content (e.g. https://api.example.com/data); Images or screenshots (e.g. screen://localhost/monitor1); Structured system data (e.g. logs, metrics, config files)
  - Each resource is identified by a URI, and can be read, discovered, and optionally subscribed to for updates.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Resource Discovery
* **Key Points:**
  - Clients can ask a server to list available resources using: resources/list
  - The server responds with an array of structured metadata.
  - Clients (or users) can browse these like a menu, selecting what context to send to the model.
* **Technical Entities (Classes/Functions/APIs):** `resources/list`
* **Code Snippet:**
```json
{
  "resources": [
    {
      "uri": "file:///logs/app.log",
      "name": "Application Logs",
      "description": "Recent server logs",
      "mimeType": "text/plain"
    }
  ]
}
```

## Resource Templates
* **Key Points:**
  - In addition to static lists, servers can expose URI templates using RFC 6570 syntax.
  - This allows dynamic access to parameterized content—great for APIs, time-based logs, or file hierarchies.
* **Technical Entities (Classes/Functions/APIs):** `RFC 6570`
* **Code Snippet:**
```json
{
  "uriTemplate": "file:///logs/{date}.log",
  "name": "Log by Date",
  "description": "Access logs by date (e.g., 2024-04-01)",
  "mimeType": "text/plain"
}
```

## Reading a Resource
* **Key Points:**
  - To retrieve the content of a resource, clients use: resources/read With a payload like: {"uri": "file:///logs/app.log"}
  - The server responds with the content in one of two formats:
  - Text Resource
  - Binary Resource (e.g. image, PDF)
  - Clients can choose how and when to inject these into the model's prompt, depending on MIME type and length.
* **Technical Entities (Classes/Functions/APIs):** `resources/read`
* **Code Snippet:**
```json
{
  "contents": [
    {
      "uri": "file:///logs/app.log",
      "mimeType": "text/plain",
      "text": "Error: Timeout on request...\n"
    }
  ]
}
```
```json
{
  "contents": [
    {
      "uri": "screen://localhost/display1",
      "mimeType": "image/png",
      "blob": "iVBORw0KGgoAAAANSUhEUgAAA..."
    }
  ]
}
```

## Real-Time Updates
* **Key Points:**
  - Resources aren't static—they can change. MCP supports subscriptions to keep context fresh.
  - List Updates: If the list of resources changes, the server can notify the client with: notifications/resources/list_changed. Useful when new logs, files, or endpoints become available.
  - Content Updates: Clients can subscribe to specific resource URIs: resources/subscribe. When the resource changes, the server sends: notifications/resources/updated. This is ideal for live logs, dashboards, or real-time documents.
* **Technical Entities (Classes/Functions/APIs):** `notifications/resources/list_changed`, `resources/subscribe`, `notifications/resources/updated`
* **Code Snippet:** None

## Security Best Practices
* **Key Points:**
  - Exposing resources to models requires careful control. MCP includes flexible patterns for securing access.
  - Best Practices for Server Developers: Validate all URIs: No open file reads!; Whitelist paths or endpoints for file access; Use descriptive names and MIME types to help clients filter content; Provide helpful descriptions for the LLM and user; Support URI templates for scalable access; Audit access and subscriptions; Avoid leaking secrets in content or metadata.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

### Example: Safe Log Server
* **Key Points:**
  - None specified
* **Technical Entities (Classes/Functions/APIs):** `ListResourcesRequestSchema`, `ReadResourceRequestSchema`
* **Code Snippet:**
```javascript
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: "file:///logs/app.log",
        name: "App Logs",
        mimeType: "text/plain"
      }
    ]
  };
});

server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const uri = request.params.uri;

  if (!uri.startsWith("file:///logs/")) {
    throw new Error("Access denied");
  }

  const content = await readFile(uri); // Add sanitization here
  return {
    contents: [{
      uri,
      mimeType: "text/plain",
      text: content
    }]
  };
});
```

## Why Resources Matter for AI Agents
* **Key Points:**
  - LLMs are context-hungry. They reason better when they have: Real-time logs; Source code; System metrics; API responses
  - By serving these as resources, MCP gives agents the data they need—on demand, with full user control, and without bloating prompt templates.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## Recap: Resources at a Glance
* **Key Points:**
  - Feature Description: URI-based identifiers Unique path to each piece of content; Text & binary support Suitable for logs, images, PDFs, etc.; Dynamic templates Construct URIs on the fly; Real-time updates Subscriptions for changing content; Secure access patterns URI validation, MIME filtering, whitelisting
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Coming Up Next: Tools in MCP — Giving LLMs the Power to Act
* **Key Points:**
  - So far, we've shown how MCP feeds models with data. But what if we want the model to take action?
  - In the next post, we'll explore tools in MCP: How LLMs call functions safely; Tool schemas and invocation patterns; Real-world examples: shell commands, API calls, and more
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None