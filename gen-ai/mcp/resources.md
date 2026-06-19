---
aliases:
  - Resources
Source 1: https://medium.com/@laurentkubaski/mcp-resources-explained-and-how-they-differ-from-mcp-tools-096f9d15f767
Source 2:
---
# MCP Resources explained (and how they differ from MCP Tools)

## 1. MCP Resources vs MCP Tools
* **Key Points:**
  - "MCP Resources allow MCP servers to share data that provides context to language models, such as files, database schemas, or application-specific information" Source: MCP Specification
  - "MCP Tools enable models to interact with external systems, such as querying databases, calling APIs, or performing computations" Source: MCP Specification
  - Yes you're right: there is not much difference between sharing the content of a file (first definition) and querying a database (second definition), which is why the #1 question people ask when they discover MCP Resources is "what the hell is the difference between a MCP Resource and a MCP Tool?".
  - The real difference between the two is their user interaction model: Tools are designed to be model-controlled, meaning that the language model can discover and invoke tools automatically based on its contextual understanding and the user's prompts (source: MCP Specification); Resources are designed to be application-driven, with host applications determining how to incorporate context based on their needs (source: MCP Specification)
  - Put differently: the model is never going to automatically retrieve a Resource, it's up to the client application to decide when to retrieve a Resource and when to send it to the model.
* **Technical Entities (Classes/Functions/APIs):** `MCP Resources`, `MCP Tools`, `FastMCP`
* **Code Snippet:**
```python
from fastmcp import FastMCP

server = FastMCP("Greetings")

@server.resource("resource://greeting")
def get_greeting_resource() -> str:
    return "Hello from FastMCP Resource"

@server.tool
def get_greeting_tool() -> str:
    return "Hello from FastMCP Tool"

if __name__ == "__main__":
    server.run(transport="stdio")
```

## 2. Application-Driven?
* **Key Points:**
  - But according to me, stating that "Resources are designed to be application-driven" is confusing. This implies that a client application can decide on its own which Resources to use and when… but how?
  - And that's where the MCP specification gets confusing with the examples it provides: For example, applications could: Expose Resources through UI elements for explicit selection, in a tree or list view; Allow the user to search through and filter available Resources; Implement automatic context inclusion, based on heuristics or the AI model's selection
  - In the first two examples, The MCP Resource is selected by the user and in the last example, it is selected by the application with the help from the model. So said different, even though in the end it's indeed the application that decides what to do with the Resource, most of the time it's really the user who decides which Resource to use.
  - So with that said, my definition of an MCP Resource becomes the following: "MCP Resources represent data or files that an MCP client can read. They are typically selected by the user, and applications then determine how to incorporate them in the context used by the model" Source: Laurent Kubaski
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:**
```python
@server.resource("resource://greeting")
def get_greeting_resource() -> str:
    return "Hello from FastMCP Resource"
```

## 3. When to use a Tool and when to use a Resource?
* **Key Points:**
  - Now you know: If the model needs to discover and call it, it's a Tool. If the user and/or application decides when it's relevant to call it so that it can be used as context to the model, it's a Resource.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## 4. How MCP Resources are used by various MCP Clients
* **Key Points:**
  - In this chapter I'll use this simple MCP Server created using FastMCP 2.0 that defines a single Resource.
* **Technical Entities (Classes/Functions/APIs):** `FastMCP 2.0`
* **Code Snippet:**
```python
from fastmcp import FastMCP

server = FastMCP("Greetings")

@server.resource("resource://greeting")
def get_greeting() -> str:
    return "Hello from FastMCP Resources"

if __name__ == "__main__":
    server.run(transport="stdio")
```

### 4.1. Using MCP Resources in Claude Desktop
* **Key Points:**
  - I'll assume that Claude Desktop is already connected to the MCP Server above (see Connecting your Python MCP Server to Claude Desktop)
  - In Claude Desktop, Click the "+" button just below the place where you type your prompts.
  - Select "Add from [name of your MCP server]": Select your Resource
  - And voilà, the Resource is now added as an attachment and you can ask Claude Desktop to interact with it. For example you can type "summarize this" and hit enter.
  - So if we come back to my earlier definition of an MCP Resource where I wrote "They are typically selected by the user, and applications then determine how to incorporate them in the context used by the model", you can see that in this case, the Claude Desktop application adds the resource to the context as an attachment.
* **Technical Entities (Classes/Functions/APIs):** `Claude Desktop`
* **Code Snippet:** None

### 4.2. Using MCP Resources in VS Code GitHub Copilot
* **Key Points:**
  - I'll assume that Visual Studio Code is already connected to the MCP Server above (see Connecting your Python MCP Server to Copilot Chat in Visual Studio Code)
  - In the Copilot Chat field, click the "Add Context" button: Click "MCP Resources": Choose the MCP Resource
  - That's it, the MCP Resource is now ready to be used.
* **Technical Entities (Classes/Functions/APIs):** `VS Code GitHub Copilot`
* **Code Snippet:** None

### 4.3. Using MCP Resources in Claude Code
* **Key Points:**
  - First, the public documentation
  - OK so to use our resource, type "Display this in uppercase: @" and then in the dropdown, select the resource.
  - Then hit "enter" and there you go.
* **Technical Entities (Classes/Functions/APIs):** `Claude Code`
* **Code Snippet:** None



---
# What Are MCP Resources? Model Context Protocol Explained

## What Are MCP Resources? Model Context Protocol Explained

## Resources vs Tools: Understanding the Difference
* **Key Points:**
  - The key distinction is about control: Resources are application-controlled: The client application decides when and how resources are used. For example, in an application like Claude Desktop, users explicitly select which resources to include before starting a conversation. Tools are model-controlled: The AI model itself determines when to invoke tools based on the conversation context and available capabilities.
  - Here is a quick side-by-side breakdown: MCP Resources: Control: Application-controlled — the host app or user decides when to load them; Access: Read-only access to data and content; Addressing: Identified by URIs (e.g., file:///path, config://app/settings); Best for: Documentation, configuration, schemas, and reference materials. MCP Tools: Control: Model-controlled — the AI model decides when to invoke them; Access: Can perform actions, writes, and side effects; Addressing: Called by function name with structured parameters; Best for: API calls, data mutations, calculations, and automated workflows.
  - This difference matters in practice. If you want data to be automatically available to the model, use a Tool. If you want the application or user to control what context gets loaded, use a Resource.
* **Technical Entities (Classes/Functions/APIs):** `MCP Resources`, `MCP Tools`
* **Code Snippet:** None

## What Makes an MCP Resource
* **Key Points:**
  - Resources are identified by URIs following the format protocol://host/path, such as: file:///home/user/documents/report.pdf; postgres://database/customers/schema; config://app/settings
  - Resources support both text content (UTF-8 encoded) and binary content (base64 encoded), making them work for everything from source code and logs to images and PDFs.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## How Resources are Discovered
* **Key Points:**
  - Clients discover resources through two main methods: direct resource lists via the resources/list endpoint, and URI templates for dynamic resources. Resource templates allow servers to define patterns like file:///{path} that clients can use to construct valid URIs on demand.
* **Technical Entities (Classes/Functions/APIs):** `resources/list`
* **Code Snippet:** None

## Implement MCP Resources in an MCP Server
* **Key Points:**
  - To illustrate how that works in code, we'll use a basic example of returning some HTML content as an MCP Resource.
  - It all begins in your OpenAPI document, where you can define the Resource route you want to expose. Resources must use the GET method, and you add MCP metadata via the mcp property inside x-zuplo-route.
  - The mcp property within x-zuplo-route tells the MCP Server Handler to expose this route as a resource rather than a tool. Set type to "resource" (the default is "tool"), and provide a descriptive name, description, uri, and mimeType so clients understand how to use it.
  - Next, wire the resource into the MCP Server Handler by referencing the operation in the operations array.
* **Technical Entities (Classes/Functions/APIs):** `OpenAPI`, `MCP Server Handler`, `x-zuplo-route`
* **Code Snippet:**
```json
{
  "/html": {
    "get": {
      "operationId": "html",
      "description": "Returns the AI applet's HTML",
      "x-zuplo-route": {
        "corsPolicy": "none",
        "handler": {
          "export": "default",
          "module": "$import(./modules/html)"
        },
        "mcp": {
          "type": "resource",
          "name": "html_doc",
          "description": "The HTML document for the AI applet",
          "uri": "ui://html",
          "mimeType": "text/html"
        }
      }
    }
  }
}
```
```json
{
  "paths": {
    "/mcp": {
      "post": {
        "x-zuplo-route": {
          "handler": {
            "export": "mcpServerHandler",
            "module": "$import(@zuplo/runtime)",
            "options": {
              "name": "example-mcp-server",
              "version": "1.0.0",
              "operations": [
                {
                  "file": "./config/routes.oas.json",
                  "id": "html"
                }
              ]
            }
          }
        }
      }
    }
  }
}
```

## MCP Resources in the Wild
* **Key Points:**
  - While Resources are often discussed in abstract terms like "reading files" or "exposing documentation," one of the most recent implementations can be found in OpenAI's Apps SDK, which uses embedded resources to power custom UI components in ChatGPT.
  - In the Apps SDK, Resources serve static UI code that agents can retrieve and render, for example: ui://widget/kanban-board.html; ui://widget/styles.css; ui://widget/app.js
  - When a tool is invoked that requires this UI the agent fetches the HTML, CSS, and JavaScript from the resource, hydrates it with structured data from the tool response, and renders it in an iframe in ChatGPT.
  - This embedded resource pattern demonstrates a practical, production-ready use of MCP Resources that goes beyond basic file reading. It shows how Resources can enable rich, interactive experiences while maintaining the application-controlled nature that defines them.
* **Technical Entities (Classes/Functions/APIs):** `OpenAI's Apps SDK`
* **Code Snippet:** None

## Best Practices for Resources
* **Key Points:**
  - Naming and Documentation: Use clear, descriptive resource names and URIs that indicate what they do; Include helpful descriptions that guide LLM understanding of when to use the resource; Ensure you set the appropriate MIME types to help clients process content correctly.
  - URI Design: Implement custom URI schemes that reflect the domain of the content you are working with (e.g., docs://, config://, db://); Consider using the default mcp://resources/{name} format for simple resources; Keep URIs consistent and predictable across related resources.
  - Performance and Scale: Cache resource contents when appropriate to reduce latency; Consider pagination for large resource lists; Be mindful of resource size, extremely large resources may impact performance.
  - Security: Expose read-only content that provides useful context to AI systems; Validate all resource URIs to prevent directory traversal or injection attacks; Implement appropriate access controls for sensitive data; Sanitize file paths and user inputs.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Common Use Cases for MCP Resources
* **Key Points:**
  - Documentation Resources: Expose API documentation, guides, or reference materials that AI systems can use to answer questions about your platform. When users ask "How do I authenticate with the API?" or "What API endpoints are available?", the AI can read your actual documentation and provide accurate answers.
  - Configuration Resources: Provide access to current configuration, feature flags, or schema information that helps AI understand your system state. This is valuable for questions like "What features are enabled?" or "What's the current API version?"
  - UI Component Resources: Share reusable UI components, templates, or design system elements that AI can reference when helping users build interfaces. AI assistants can read your CSS, HTML templates, or component libraries to provide consistent guidance aligned with your design system.
  - These patterns let AI agents or local LLM based applications provide accurate, contextually relevant help by accessing your system's documentation, configuration, and resources on demand.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```json
"mcp": {
  "type": "resource",
  "name": "api_guide",
  "description": "API usage guide and best practices",
  "uri": "docs://api-guide",
  "mimeType": "text/markdown"
}
```
```json
"mcp": {
  "type": "resource",
  "name": "api_config",
  "description": "Current API configuration and settings",
  "uri": "config://api",
  "mimeType": "application/json"
}
```
```json
"mcp": {
  "type": "resource",
  "name": "component_library",
  "description": "Available UI components and their usage",
  "uri": "ui://components",
  "mimeType": "text/html"
}
```

## Choosing Between MCP Resources and Tools
* **Key Points:**
  - Choose Resources rather than Tools when you want the user or application to control what data the AI can access, not the AI model itself. This is important for: User-driven context: When users should explicitly choose what information to share (like selecting specific documentation or logs to include in a conversation); Sensitive data: When you need users to opt-in before exposing certain information to the AI; Large reference materials: When you have extensive documentation or data that shouldn't be automatically loaded but should be available on demand; Compliance requirements: When regulations or policies require explicit user consent before data access.
  - Resources work well for: Documentation and guides that users might reference during conversations; Configuration and state that changes infrequently but provides important context; Reference materials like schemas, templates, or style guides; Historical data like logs or records that users explicitly want to analyze.
  - Choose Tools instead when you want the AI model to automatically decide when to access data or perform actions based on the conversation. You can also pair Resources with MCP prompt templates to give AI agents both the context and the instructions they need.
  - For example, if you want the AI to automatically fetch current weather data when a user asks about the weather, that would be a Tool use scenario. If you want the user to explicitly load weather data before the conversation starts, use a Resource.
* **Technical Entities (Classes/Functions/APIs):** `MCP prompt templates`
* **Code Snippet:** None

## Why MCP Resources Matter
* **Key Points:**
  - The application-controlled nature makes Resources ideal when you want explicit control over what context gets loaded, rather than having the model automatically access data. This gives users and applications the power to choose what information the AI can see, making interactions more predictable and controllable.
  - If you're building AI-powered products that consume multiple APIs and MCP servers, an MCP gateway can help you centralize authentication, access control, and observability across all your MCP connections.
* **Technical Entities (Classes/Functions/APIs):** `MCP gateway`
* **Code Snippet:** None