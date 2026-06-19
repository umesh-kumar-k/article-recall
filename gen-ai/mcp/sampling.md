---
aliases:
  - Sampling
Source 1: https://www.oreilly.com/radar/mcp-sampling-when-your-tools-need-to-think/
---
## MCP Sampling: When Your Tools Need to Think

## How Sampling Works
* **Key Points:**
  - When an MCP client like goose connects to an MCP server, it establishes a two-way channel. The server can expose tools for the AI to call, but it can also request that the AI generate text on its behalf.
  - The ctx.sample() call sends a prompt back to the connected AI and waits for a response. From the user's perspective, they just called a "summarize" tool. But under the hood, that tool delegated the hard part to the AI itself.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `goose`, `FastMCP`, `ctx.sample()`
* **Code Snippet:** None

## A Real Example: Council of Mine
* **Key Points:**
  - Council of Mine is an MCP server that takes sampling to an extreme. It simulates a council of nine AI personas who debate topics and vote on each other's opinions.
  - But there's no LLM running inside the server. Every opinion, every vote, every bit of reasoning comes from sampling requests back to the user's connected LLM.
  - The council has nine members, each with a distinct personality: 🔧 The Pragmatist – "Will this actually work?"; 🌟 The Visionary – "What could this become?"; 🔗 The Systems Thinker – "How does this affect the broader system?"; 😊 The Optimist – "What's the upside?"; 😈 The Devil's Advocate – "What if we're completely wrong?"; 🤝 The Mediator – "How can we integrate these perspectives?"; 👥 The User Advocate – "How will real people interact with this?"; 📜 The Traditionalist – "What has worked historically?"; 📊 The Analyst – "What does the data show?"
  - Each personality is defined as a system prompt that gets prepended to sampling requests.
  - When you start a debate, the server makes nine sampling calls, one for each council member.
  - That temperature=0.8 setting encourages diverse, creative responses. Each council member "thinks" independently because each is a separate LLM call with a different personality prompt.
  - After opinions are collected, the server runs another round of sampling. Each member reviews everyone else's opinions and votes for the one that resonates most with their values.
  - The server parses the structured response to extract votes and reasoning.
  - One more sampling call generates a balanced summary that incorporates all perspectives and acknowledges the winning viewpoint.
  - Total LLM calls per debate: 19: 9 for opinions; 9 for voting; 1 for synthesis
  - All of those calls go through the user's existing LLM connection. The MCP server itself has zero LLM dependencies.
* **Technical Entities (Classes/Functions/APIs):** `Council of Mine`
* **Code Snippet:**
```
Council of members 1
```
```
The council has voted
```

## Benefits of Sampling
* **Key Points:**
  - Sampling enables a new category of MCP servers that orchestrate intelligent behavior without managing their own LLM infrastructure.
  - No API key management: The MCP server doesn't need its own credentials. Users bring their own AI, and sampling uses whatever they've already configured.
  - Model flexibility: If a user switches from GPT to Claude to a local Llama model, the server automatically uses the new model.
  - Simpler architecture: MCP server developers can focus on building a tool, not an AI application. They can let the AI be the AI, while the server focuses on orchestration, data access, and domain logic.
* **Technical Entities (Classes/Functions/APIs):** `MCP`
* **Code Snippet:** None

## When to Use Sampling
* **Key Points:**
  - Sampling makes sense when a tool needs to: Generate creative content (summaries, translations, rewrites); Make judgment calls (sentiment analysis, categorization); Process unstructured data (extract info from messy text)
  - It's less useful for: Deterministic operations (math, data transformation, API calls); Latency-critical paths (each sample adds round-trip time); High-volume processing (costs add up quickly)
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## The Mechanics
* **Key Points:**
  - If you're implementing sampling, here are the key parameters: Sampling parameters
  - The response object contains the generated text, which you'll need to parse. Council of Mine includes robust extraction logic because different LLM providers return slightly different response formats.
* **Technical Entities (Classes/Functions/APIs):** `Council of Mine`
* **Code Snippet:**
```
Sampling parameters
```
```
Council of Mine robust extraction logic
```

## Security Considerations
* **Key Points:**
  - When you're passing user input into sampling prompts, you're creating a potential prompt injection vector. Council of Mine handles this with clear delimiters and explicit instructions.
  - This isn't bulletproof, but it raises the bar significantly.
* **Technical Entities (Classes/Functions/APIs):** `Council of Mine`
* **Code Snippet:**
```
Council of Mine delimiters and instructions
```