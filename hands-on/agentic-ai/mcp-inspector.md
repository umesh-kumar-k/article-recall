---
aliases:
  - MCP Inspector
Source 1: https://modelcontextprotocol.io/docs/tools/inspector
---
# MCP Inspector


* **Key Points:**
  - The MCP Inspector is an interactive developer tool for testing and debugging MCP servers.

## Getting started
### Installation and basic usage
* **Key Points:**
  - The Inspector runs directly through npx without requiring installation:
* **Technical Entities (Classes/Functions/APIs):** `npx`, `@modelcontextprotocol/inspector`
* **Code Snippet:**
```bash
npx @modelcontextprotocol/inspector <command>
npx @modelcontextprotocol/inspector <command> <arg1> <arg2>
```

### Inspecting servers from npm or PyPI
* **Key Points:**
  - A common way to start server packages from npm or PyPI.
* **Code Snippet:**
```bash
npx -y @modelcontextprotocol/inspector npx <package-name> <args>
# For example
npx -y @modelcontextprotocol/inspector npx @modelcontextprotocol/server-filesystem /Users/username/Desktop
```

### Inspecting locally developed servers
* **Key Points:**
  - To inspect servers locally developed or downloaded as a repository, the most common way is:
  - Please carefully read any attached README for the most accurate instructions.
* **Technical Entities (Classes/Functions/APIs):** `node`
* **Code Snippet:**
```bash
npx @modelcontextprotocol/inspector node path/to/server/index.js args...
```

## Feature overview
* **Key Points:**
  - The Inspector provides several features for interacting with your MCP server:

### Server connection pane
* **Key Points:**
  - Allows selecting the transport for connecting to the server
  - For local servers, supports customizing the command-line arguments and environment

### Resources tab
* **Key Points:**
  - Lists all available resources
  - Shows resource metadata (MIME types, descriptions)
  - Allows resource content inspection
  - Supports subscription testing

### Prompts tab
* **Key Points:**
  - Displays available prompt templates
  - Shows prompt arguments and descriptions
  - Enables prompt testing with custom arguments
  - Previews generated messages

### Tools tab
* **Key Points:**
  - Lists available tools
  - Shows tool schemas and descriptions
  - Enables tool testing with custom inputs
  - Displays tool execution results

### Notifications pane
* **Key Points:**
  - Presents all logs recorded from the server
  - Shows notifications received from the server

## Best practices
### Development workflow
* **Key Points:**
  - Start Development: Launch Inspector with your server, Verify basic connectivity, Check capability negotiation
  - Iterative testing: Make server changes, Rebuild the server, Reconnect the Inspector, Test affected features, Monitor messages
  - Test edge cases: Invalid inputs, Missing prompt arguments, Concurrent operations, Verify error handling and error responses