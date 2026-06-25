---
aliases:
  - MCP Tools
Source 1: https://docs.langchain.com/oss/javascript/deepagents/code/mcp-tools
---
## MCP tools
* **Key Points:**
  - MCP (Model Context Protocol) lets you extend Deep Agents Code with tools from external servers—file systems, APIs, databases, and more—without modifying the agent itself.
  - Deep Agents Code connects to MCP servers at startup, discovers their tools, and makes them available to the agent alongside the built-in tools.
  - Add MCP servers by adding a `.mcp.json` config file to your project for project-level scope, or at user-level to apply to all projects.

## Quickstart
* **Key Points:**
  - This quickstart adds the LangChain documentation MCP server to every Deep Agents Code session on your machine.
* **Technical Entities (Classes/Functions/APIs):** `dcode`, `/mcp`, `.mcp.json`
* **Code Snippet:**
```bash
mkdir -p ~/.deepagents
touch ~/.deepagents/.mcp.json
```
```json
{
    "mcpServers": {
        "docs-langchain": {
            "type": "http",
            "url": "https://docs.langchain.com/mcp"
        }
    }
}
```

## Auto-discovery
* **Key Points:**
  - Deep Agents Code automatically searches for `.mcp.json` files in standard locations. No flags are needed—just place a config file and it gets picked up.
  - Configs are checked in this order (lowest to highest precedence): `~/.deepagents/.mcp.json` (User-level—applies to all projects), `<project>/.deepagents/.mcp.json` (Project-level—`.deepagents` subdirectory), `<project>/.mcp.json` (Project-level—root (Claude Code compatible))
  - The project root is the nearest parent directory containing a `.git` folder, falling back to the current working directory.
  - When multiple config files exist, their `mcpServers` entries are merged. If the same server name appears in more than one file, the higher-precedence config wins.
* **Technical Entities (Classes/Functions/APIs):** `--mcp-config PATH`, `--no-mcp`, `dcode`, `.mcp.json`

### Flags
* **Key Points:**
  - `--mcp-config PATH`: Add an explicit config as the highest-precedence source (merged on top of auto-discovered configs)
  - `--no-mcp`: Disable MCP entirely—no servers are loaded
  - `--mcp-config` and `--no-mcp` are mutually exclusive.

### Claude Code compatibility
* **Key Points:**
  - If you already have a `.mcp.json` at your project root for Claude Code, Deep Agents Code picks it up automatically—no extra setup needed.

## Configuration format
* **Key Points:**
  - Each key under `mcpServers` is a server name. The server's fields determine how Deep Agents Code connects to it.

### stdio servers (default)
* **Key Points:**
  - stdio servers are spawned as child processes. Deep Agents Code communicates with them over stdin/stdout.
* **Technical Entities (Classes/Functions/APIs):** `command`, `args`, `env`
* **Code Snippet:**
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
      "env": {}
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "your-token" }
    }
  }
}
```

### SSE and HTTP servers
* **Key Points:**
  - For remote MCP servers, set `type` to `"sse"` or `"http"` and provide a `url`:
* **Technical Entities (Classes/Functions/APIs):** `type`, `url`, `headers`
* **Code Snippet:**
```json
{
  "mcpServers": {
    "remote-api": {
      "type": "sse",
      "url": "https://api.example.com/mcp",
      "headers": { "Authorization": "Bearer your-token" }
    }
  }
}
```

### Field reference
* **Technical Entities (Classes/Functions/APIs):** `command`, `args`, `env`, `type`, `url`, `headers`, `auth`, `allowedTools`, `disabledTools`

### Header environment variables
* **Key Points:**
  - Header values support `${VAR}` substitution from the parent shell, resolved at server activation rather than at config load. One unset variable only fails the server that needs it; the rest still come up.
* **Code Snippet:**
```json
{
    "mcpServers": {
        "internal-api": {
            "type": "http",
            "url": "https://api.example.com/mcp",
            "headers": { "Authorization": "Bearer ${INTERNAL_API_TOKEN}" }
        }
    }
}
```

## Multiple servers
* **Key Points:**
  - You can configure as many servers as you need. Tools from all servers are merged and available to the agent:
* **Code Snippet:**
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_..." }
    },
    "database": {
      "type": "sse",
      "url": "https://db-mcp.internal:8080/mcp",
      "headers": { "Authorization": "Bearer ..." }
    }
  }
}
```

## Tool filtering
* **Key Points:**
  - Each server may narrow the tools it exposes to the agent with one of two optional fields: `allowedTools`: keep only the listed tools; drop everything else. `disabledTools`: drop the listed tools; keep everything else.
  - Filtering applies to stdio, HTTP, and SSE servers alike.
  - Setting `allowedTools` and `disabledTools` on the same server is rejected at config load.
  - Setting either field to an empty list is rejected at config load (would silently strip every tool, or be a no-op). Omit the field instead.
* **Technical Entities (Classes/Functions/APIs):** `allowedTools`, `disabledTools`, `fnmatch`
* **Code Snippet:**
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
      "allowedTools": ["read_file", "list_directory"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "disabledTools": ["delete_repository", "delete_*_branch"]
    }
  }
}
```

### Match rules
* **Key Points:**
  - Each entry is a literal tool name or an `fnmatch`-style glob (any entry containing `*`, `?`, or `[` is treated as a pattern).
  - Entries are matched against both the bare MCP tool name and the server-prefixed form (`{server}_{tool}`), so either form works:
  - Entries that match no loaded tool are logged as a warning, not an error — the underlying MCP server can evolve its tool list across versions without breaking your config.
* **Technical Entities (Classes/Functions/APIs):** `allowedTools`, `disabledTools`, `fnmatch`

## OAuth login
* **Key Points:**
  - For remote MCP servers that require OAuth (Slack, GitHub, Notion, Linear, and other hosted MCP endpoints), set `"auth": "oauth"` on the server entry and run the login subcommand once. Tokens are persisted to disk and refreshed automatically.
  - `auth: "oauth"` is mutually exclusive with an `Authorization` header on the same entry, and cannot be set on a stdio server.
* **Technical Entities (Classes/Functions/APIs):** `auth`, `oauth`, `dcode mcp login`
* **Code Snippet:**
```json
{
    "mcpServers": {
        "linear": {
            "type": "http",
            "url": "https://mcp.linear.app/mcp",
            "auth": "oauth"
        }
    }
}
```
```bash
dcode mcp login linear
```

### Token storage
* **Key Points:**
  - Tokens are written to: `~/.deepagents/.state/mcp-tokens/<server>-<sha256-16(url)>.json`
  - The `<sha256-16(url)>` segment is the first 16 hex characters of the SHA-256 of the server URL.
  - The directory is locked to mode `0700` and each token file is mode `0600`.
  - Files include the OAuth access token, refresh token, and the dynamically registered client info, all in a schema-versioned payload that's written atomically (write-to-temp + `rename`).
  - Hashing the URL into the filename means the same server name pointing at different URLs (for example, dev vs. prod) gets independent token files and can't trample each other.

### Re-authentication
* **Key Points:**
  - When refresh fails at runtime (the refresh token expired or was revoked), Deep Agents Code marks the server as `unauthenticated` instead of crashing the agent.
  - The welcome banner shows the count of unauthenticated servers, and `/mcp` reports the reason per server.
  - Re-run `dcode mcp login <server>` to refresh credentials — your conversation continues without restarting.

## Server status
* **Key Points:**
  - Each configured server lands in one of three states after startup:
  - `ok`: Connected; tools are loaded and available to the agent
  - `unauthenticated`: OAuth login required or refresh failed — run `dcode mcp login <server>`
  - `error`: Pre-flight, discovery, or transport setup failed; an error message is attached
  - A single failing server no longer aborts startup. The agent runs with whichever servers came up cleanly, and the welcome banner surfaces counts of unauthenticated and errored servers next to the tool count.
  - Open `/mcp` in an interactive session to see per-server status, transport, tool list, and the failure reason for non-`ok` entries.

## Project-level trust
* **Key Points:**
  - Project-level configs can contain stdio servers that execute local commands and remote servers whose `headers` may interpolate `${VAR}` from your environment. To prevent untrusted repositories from running arbitrary code or exfiltrating local secrets on CLI startup, Deep Agents Code enforces a **default-deny** policy for project-level entries.
  - **Interactive mode:** Deep Agents Code prompts for approval before activating project servers, showing each stdio command and remote URL. Approval is persisted using a SHA-256 content fingerprint—if the config changes, you are prompted again.
  - **Non-interactive mode (`-n`):** Project servers are silently skipped unless `--trust-project-mcp` is passed.
  - **Trust covers stdio and remote entries alike** — remote servers can SSRF into localhost or cloud-metadata endpoints during the pre-flight probe and exfiltrate `${VAR}` values via headers, so they're gated the same way as stdio.
  - **User-level configs** (`~/.deepagents/.mcp.json`) are always trusted—the same trust model as `config.toml` and `hooks.json`.
* **Technical Entities (Classes/Functions/APIs):** `--trust-project-mcp`, `-n`, `dcode`, `~/.deepagents/.state/mcp_trust.json`

### Flags
* **Key Points:**
  - `--trust-project-mcp`: Trust all project-level stdio servers without prompting (for CI and automation)
* **Code Snippet:**
```bash
# Skip the approval prompt
dcode --trust-project-mcp

# Non-interactive: explicitly trust project servers
dcode -n "run tests" --trust-project-mcp
```

### Trust store
* **Key Points:**
  - Trust decisions are stored in `~/.deepagents/.state/mcp_trust.json`
  - Each key under `projects` is an absolute project root path. The value is a SHA-256 digest of the concatenated project-level config contents.
  - To revoke trust, delete the entry or modify the project's `.mcp.json` (which invalidates the fingerprint automatically).
  - A trusted stdio MCP server has the same permissions as your user account. Only approve servers from repositories you trust. Review the commands shown in the approval prompt before accepting.

## System prompt awareness
* **Key Points:**
  - Connected MCP servers and their tools are automatically listed in the agent's system prompt, grouped by server name and transport type. This helps the model reason about tool provenance and failure domains without requiring manual context.