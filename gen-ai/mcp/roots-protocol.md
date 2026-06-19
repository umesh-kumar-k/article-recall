---
aliases:
  - Roots Protocol
Source 1: https://www.speakeasy.com/mcp/core-concepts/roots
---
## What are MCP roots?

* **Key Points:**
  - MCP roots are context-defining URIs that establish operational boundaries for MCP servers. They function as "safe zones" or "allowed directories" that an AI agent can access when interacting with your system. When an MCP client provides roots for servers, it's essentially saying, "You're allowed to work within these specific areas."
  - While roots are sometimes described as a security feature, the MCP Specification does not dictate how servers must implement root handling. So when the client tells the server which areas it can work within, the server may ignore this rule and still be within the specification.
  - Any suggestion that roots provide security needs to assume that the server will respect the boundaries set by the client.
* **Technical Entities (Classes/Functions/APIs):** `MCP roots`, `MCP Specification`
* **Code Snippet:** None

## Why are MCP roots useful?
* **Key Points:**
  - MCP roots allow the user (or an LLM) to control where a server should focus or within which boundaries a server should operate.
  - For example, if a user has a stock-trading MCP server, they may want their client to limit the trading server to only certain stocks or exchanges.
  - In this example, the client will prompt the user to select a list of stocks and exchanges, and then notify the server that the list of roots has changed. The server will then request the new roots, and should limit its actions to the selected list of stocks and exchanges.
  - Roots help in several important ways: Security: If a server is known to respect root boundaries, roots act like fences that keep the server from accessing things it shouldn't. Focus: Roots guide servers to look only at relevant information, like giving them a map of where to search. Performance: By limiting where a server can look, the server may run faster and with less searching or filtering logic. Trust: Users feel more comfortable knowing servers can only access specific areas they've approved. This still depends on the server's implementation of roots.
  - MCP roots can point to different kinds of places: Folders on your computer: file:///home/user/projects/myapp; Websites and APIs: https://api.example.com/v1; Database connections: db://mycompany/customers; Special locations: note://meeting-notes or config://app-settings
* **Technical Entities (Classes/Functions/APIs):** `MCP roots`
* **Code Snippet:** None

## How MCP roots work
* **Key Points:**
  - Here's how MCP roots could work in practice: Client to the server: "Hello, I can use roots!" - The client tells the server it supports roots when they first connect. Server to the client: "What roots do you have?" - The server asks the client for available roots using roots/list. Client to the server: "Stay within these boundaries." - The client returns a list of roots. User to the client: "I want to change the boundaries." - The user updates the boundaries the client and server should focus on. Client to the server: "Roots have changed." - If needed, the client can tell the server when roots are added or removed using notifications/roots/list_changed. Server to the client: "What roots do you have?" - The server requests a list of roots using roots/list. Client to the server: "Stay within these boundaries." - The client responds with a list of roots.
  - It bears repeating that while the client owns and defines the list of allowed roots, the server should stay within those boundaries but is free to implement root handling how it wants. The MCP Specification does not enforce that servers respect those boundaries. It only standardizes how roots are communicated.
* **Technical Entities (Classes/Functions/APIs):** `roots/list`, `notifications/roots/list_changed`, `MCP Specification`
* **Code Snippet:**
```json
{
  "roots": [
    {
      "uri": "file:///home/user/projects/myapp",
      "name": "My Application Code"
    },
    {
      "uri": "https://api.example.com/v1",
      "name": "Example API Endpoint"
    }
  ]
}
```

## Implementing MCP roots in Python
* **Key Points:**
  - Let's look at examples of how to implement MCP roots using FastMCP.
* **Technical Entities (Classes/Functions/APIs):** `FastMCP`, `Client`, `FastMCPTransport`
* **Code Snippet:**
```python
# server-example.py

from fastmcp import FastMCP
import asyncio
from mcp.server.stdio import stdio_server
from mcp.server import InitializationOptions, NotificationOptions
 
# Create an MCP server
app = FastMCP("Roots Example Server")
 
# Define roots using the standard resource pattern
@app.resource("roots://list")
def list_roots():
    return {
        "roots": [
            {
                "uri": "file:///home/projects/roots-example/frontend",
                "name": "Frontend Repository"
            },
            {
                "uri": "https://api.openf1.org/",
                "name": "F1 API Endpoint"
            }
        ]
    }
 
# Define a simple tool that processes a file
@app.tool()
def process_file(filename: str) -> str:
    return f"Processing file: {filename}"
 
# Server entrypoint
async def main():
    async with stdio_server() as (read_stream, write_stream):
        await mcp.run(
            read_stream,
            write_stream,
            InitializationOptions(
                server_name="roots-example-server",
                server_version="0.1.0",
                capabilities=mcp.get_capabilities(
                    notification_options=NotificationOptions(),
                    experimental_capabilities={},
                ),
            ),
        )
 
if __name__ == "__main__":
    asyncio.run(main())
```
```python
# client-example.py

from fastmcp import Client
from fastmcp.client.transports import FastMCPTransport
import asyncio
 
async def client_example():
    # Connect to a running MCP server and specify the roots we want to use
    async with Client(transport_url="stdio:", roots=["roots://list"]) as client:
        # Call a tool on the server
        result = await client.call_tool("process_file", {"filename": "data.csv"})
        print(result)
 
if __name__ == "__main__":
    asyncio.run(client_example())
```

## Implementation patterns
* **Key Points:**
  - Let's look at some examples of how we can use MCP roots within a server.
* **Technical Entities (Classes/Functions/APIs):** `MCP roots`
* **Code Snippet:** None

### File system access
* **Key Points:**
  - When granting access to file system directories, always verify that accessed paths are within approved roots.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```python
def read_file(file_path):
    # Convert to absolute path
    abs_path = os.path.abspath(file_path)
 
    # Check if path is within any approved root
    for root in app.roots:
        if root.uri.startswith("file://"):
            root_path = root.uri.replace("file://", "")
            if abs_path.startswith(root_path):
                # Safe to read
                with open(abs_path, 'r') as f:
                    return f.read()
 
    # Not within any approved root
    raise SecurityError(f"Access denied: {file_path} is outside approved roots")
```

### API endpoint access
* **Key Points:**
  - Similarly, when making API calls, make sure they respect root boundaries.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```python
async def make_api_call(endpoint, method="GET", data=None):
    # Check if endpoint is within an approved root
    for root in app.roots:
        if root.uri.startswith("https://") and endpoint.startswith(root.uri):
            # Safe to proceed
            async with httpx.AsyncClient() as client:
                response = await client.request(method, endpoint, json=data)
                return response.json()
 
    # Not within any approved root
    raise SecurityError(f"Access denied: {endpoint} is outside approved API roots")
```

### Tool integration
* **Key Points:**
  - MCP roots also work with MCP tools. If a tool is granted access to a root, it can only operate on files within that root.
* **Technical Entities (Classes/Functions/APIs):** `MCP tools`
* **Code Snippet:**
```python
@app.tool("create_file")
async def create_file(path: str, content: str):
    # Convert to absolute path
    abs_path = os.path.abspath(path)
 
    # Check against allowed file system roots
    for root in app.current_roots:
        if root.uri.startswith("file://"):
            root_path = root.uri.replace("file://", "")
            if abs_path.startswith(root_path):
                # Write the file
                with open(abs_path, 'w') as f:
                    f.write(content)
                return {"success": True, "path": abs_path}
 
    return {
        "success": False,
        "error": "Permission denied: path is outside allowed roots"
    }
```

### Resource integration
* **Key Points:**
  - Resources can also be filtered based on allowed roots.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```python
@app.on_resources_list()
async def list_resources():
    all_resources = await get_all_available_resources()
 
    # Filter to only include resources within allowed roots
    allowed_resources = []
    for resource in all_resources:
        for root in app.current_roots:
            if resource.uri.startswith(root.uri):
                allowed_resources.append(resource)
                break
 
    return allowed_resources
```

## Roots vs tools
* **Key Points:**
  - MCP roots are distinct from MCP tools, which are callable server-side actions. While MCP roots define where operations can occur, MCP tools define what actions can be performed.
* **Technical Entities (Classes/Functions/APIs):** `MCP roots`, `MCP tools`
* **Code Snippet:** None
