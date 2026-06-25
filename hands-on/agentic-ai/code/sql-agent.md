---
aliases:
  - SQL Agent
Source 1: https://docs.langchain.com/oss/javascript/langchain/sql-agent
---
# Build a SQL agent


## Overview
* **Key Points:**
  - In this tutorial, you will learn how to build an agent that can answer questions about a SQL database using LangChain agents.
  - At a high level, the agent will:
    - Fetch the available tables and schemas from the database
    - Decide which tables are relevant to the question
    - Fetch the schemas for the relevant tables
    - Generate a query based on the question and information from the schemas
    - Double-check the query for common mistakes using an LLM
    - Execute the query and return the results
    - Correct mistakes surfaced by the database engine until the query is successful
    - Formulate a response based on the results
  - Building Q&A systems of SQL databases requires executing model-generated SQL queries. There are inherent risks in doing this. Make sure that your database connection permissions are always scoped as narrowly as possible for your agent's needs. This will mitigate, though not eliminate, the risks of building a model-driven system.

### Concepts
* **Key Points:**
  - The following tutorial covers the following concepts:
  - Tools for reading from SQL databases
  - LangChain agents
  - Human-in-the-loop processes

## Setup
### Install dependencies
* **Code Snippet:**
```bash
npm i langchain @langchain/core sqlite3 zod
```

### Set up LangSmith
* **Key Points:**
  - Set up LangSmith to inspect what is happening inside your chain or agent. Then set the following environment variables:
* **Technical Entities (Classes/Functions/APIs):** `LANGSMITH_TRACING`, `LANGSMITH_API_KEY`
* **Code Snippet:**
```shell
export LANGSMITH_TRACING="true"
export LANGSMITH_API_KEY="..."
```

## Build your SQL agent

### Select an LLM
* **Key Points:**
  - Select a model that supports tool-calling:
* **Technical Entities (Classes/Functions/APIs):** `initChatModel`, `ChatOpenAI`, `ChatAnthropic`, `AzureChatOpenAI`, `ChatGoogleGenerativeAI`, `ChatBedrockConverse`

### Configure the database
* **Key Points:**
  - You will be creating a SQLite database for this tutorial. SQLite is a lightweight database that is easy to set up and use. We will be loading the `chinook` database, which is a sample database that represents a digital media store.
  - For convenience, we have hosted the database (`Chinook.db`) on a public GCS bucket.
* **Technical Entities (Classes/Functions/APIs):** `fs`, `path`, `fetch`
* **Code Snippet:**
```ts
import fs from "node:fs/promises";
import path from "node:path";

const url =
  "https://storage.googleapis.com/benchmarks-artifacts/chinook/Chinook.db";
const localPath = path.resolve("Chinook.db");

async function resolveDbPath() {
  try {
    await fs.access(localPath);
    return localPath;
  } catch {
    // Chinook.db not present locally; download it.
  }
  const resp = await fetch(url);
  if (!resp.ok)
    throw new Error(`Failed to download DB. Status code: ${resp.status}`);
  const buf = Buffer.from(await resp.arrayBuffer());
  await fs.writeFile(localPath, buf);
  return localPath;
}
```

### Add tools for database interactions
* **Key Points:**
  - The following database tools are minimal wrappers for demonstration purposes only. They are not intended to be secure or used in production. Use narrowly scoped database permissions and add application-specific validation before executing model-generated SQL.
  - We will use the `sqlite3` library to query the database and fetch schemas:
* **Technical Entities (Classes/Functions/APIs):** `sqlite3`, `runQuery()`, `getSchema()`
* **Code Snippet:**
```ts
import sqlite3 from "sqlite3";

// Below are minimal tools for demonstration purposes.
async function runQuery(query: string): Promise<any[]> {
  const dbPath = await resolveDbPath();
  const db = new sqlite3.Database(dbPath);
  return new Promise((resolve, reject) => {
    db.all(query, [], (err, rows) => {
      db.close();
      if (err) reject(err);
      else resolve(rows);
    });
  });
}

async function getSchema() {
  const tables = await runQuery(
    "SELECT sql FROM sqlite_master WHERE type='table' AND name NOT LIKE 'sqlite_%';",
  );
  return tables.map((row) => row.sql).join("\n\n");
}
```

### Create the agent
* **Key Points:**
  - Before running the command, do a check to check the LLM generated command in `_safe_sql`:
  - Then, execute commands with an `execute_sql` tool:
  - Use `createAgent` to build a ReAct agent with minimal code. The agent will interpret the request and generate a SQL command. The tools will check the command for safety and then try to execute the command. If the command has an error, the error message is returned to the model. The model can then examine the original request and the new error message and generate a new command. This can continue until the LLM generates the command successfully or reaches an end count. This pattern of providing a model with feedback - error messages in this case - is very powerful.
  - Initialize the agent with a descriptive system prompt to customize its behavior:
  - Now, create an agent with the model, tools, and prompt:
* **Technical Entities (Classes/Functions/APIs):** `DENY_RE`, `HAS_LIMIT_TAIL_RE`, `sanitizeSqlQuery()`, `tool`, `z`, `executeSql`, `SystemMessage`, `createAgent`, `getSystemPrompt()`
* **Code Snippet:**
```ts
const DENY_RE =
  /\b(INSERT|UPDATE|DELETE|ALTER|DROP|CREATE|REPLACE|TRUNCATE)\b/i;
const HAS_LIMIT_TAIL_RE = /\blimit\b\s+\d+(\s*,\s*\d+)?\s*;?\s*$/i;

function sanitizeSqlQuery(q) {
  let query = String(q ?? "").trim();

  // block multiple statements (allow one optional trailing ;)
  const semis = [...query].filter((c) => c === ";").length;
  if (semis > 1 || (query.endsWith(";") && query.slice(0, -1).includes(";"))) {
    throw new Error("multiple statements are not allowed.");
  }
  query = query.replace(/;+\s*$/g, "").trim();

  // read-only gate
  if (!query.toLowerCase().startsWith("select")) {
    throw new Error("Only SELECT statements are allowed");
  }
  if (DENY_RE.test(query)) {
    throw new Error("DML/DDL detected. Only read-only queries are permitted.");
  }

  // append LIMIT only if not already present
  if (!HAS_LIMIT_TAIL_RE.test(query)) {
    query += " LIMIT 5";
  }
  return query;
}
```
```ts
import { tool } from "langchain";
import * as z from "zod";

const executeSql = tool(
  async ({ query }) => {
    const q = sanitizeSqlQuery(query);
    try {
      const result = await runQuery(q);
      return JSON.stringify(result, null, 2);
    } catch (e) {
      const message = e instanceof Error ? e.message : String(e);
      throw new Error(message);
    }
  },
  {
    name: "execute_sql",
    description: "Execute a READ-ONLY SQLite SELECT query and return results.",
    schema: z.object({
      query: z.string().describe("SQLite SELECT query to execute (read-only)."),
    }),
  },
);
```
```ts
import { SystemMessage } from "langchain";

const getSystemPrompt = async () =>
  new SystemMessage(`You are a careful SQLite analyst.

Authoritative schema (do not invent columns/tables):
${await getSchema()}

Rules:
- Think step-by-step.
- When you need data, call the tool \`execute_sql\` with ONE SELECT query.
- Read-only only; no INSERT/UPDATE/DELETE/ALTER/DROP/CREATE/REPLACE/TRUNCATE.
- Limit to 5 rows unless user explicitly asks otherwise.
- If the tool returns 'Error:', revise the SQL and try again.
- Limit the number of attempts to 5.
- If you are not successful after 5 attempts, return a note to the user.
- Prefer explicit column lists; avoid SELECT *.
`);
```
```ts
import { createAgent } from "langchain";

let agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [executeSql],
  systemPrompt: await getSystemPrompt(),
});
```

### Run the agent
* **Key Points:**
  - Run the agent on a sample query and observe its behavior:
  - The agent correctly wrote a query, checked the query, and ran it to inform its final response.
  - You can inspect all aspects of the above run, including steps taken, tools invoked, what prompts were seen by the LLM, and more in the LangSmith trace.
* **Technical Entities (Classes/Functions/APIs):** `agent.streamEvents()`, `stream.messages`, `stream.toolCalls`, `stream.output`
* **Code Snippet:**
```ts
let question = "Which genre, on average, has the longest tracks?";

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: question }] },
  { version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
      console.log(`Tool result: ${await call.output}`);
    }
  })(),
]);

const finalState = await stream.output;
```

### (Optional) Use Studio
* **Key Points:**
  - Studio provides a "client side" loop as well as memory so you can run this as a chat interface and query the database. You can ask questions like "Tell me the scheme of the database" or "Show me the invoices for the 5 top customers". You will see the SQL command that is generated and the resulting output.
* **Technical Entities (Classes/Functions/APIs):** `langgraph-cli`, `langgraph.json`, `agent`, `graph`
* **Code Snippet:**
```json
{
  "dependencies": ["."],
  "graphs": {
      "agent": "./sqlAgent.ts:agent",
      "graph": "./sqlAgentLanggraph.ts:graph"
  },
  "env": ".env"
}
```

### Implement human-in-the-loop review
* **Key Points:**
  - It can be prudent to check the agent's SQL queries before they are executed for any unintended actions or inefficiencies.
  - LangChain agents feature support for built-in human-in-the-loop middleware to add oversight to agent tool calls. Let's configure the agent to pause for human review on calling the `execute_sql` tool:
  - We've added a checkpointer to our agent to allow execution to be paused and resumed.
  - On running the agent, it will now pause for review before executing the `execute_sql` tool:
  - We can resume execution, in this case accepting the query, using Command:
* **Technical Entities (Classes/Functions/APIs):** `humanInTheLoopMiddleware`, `MemorySaver`, `checkpointer`, `Command`, `agent.streamEvents()`, `stream.messages`, `stream.toolCalls`, `stream.interrupted`, `stream.interrupts`
* **Code Snippet:**
```ts
import { humanInTheLoopMiddleware } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [executeSql],
  systemPrompt: await getSystemPrompt(),
  middleware: [
    humanInTheLoopMiddleware({
      interruptOn: {
        execute_sql: true,
      },
      descriptionPrefix: "Tool execution pending approval",
    }),
  ],
  checkpointer: new MemorySaver(),
});
```
```ts
question = "Which genre, on average, has the longest tracks?";
const config = { configurable: { thread_id: "1" } };

const hitlStream = await agent.streamEvents(
  { messages: [{ role: "user", content: question }] },
  { ...config, version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of hitlStream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of hitlStream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
    }
  })(),
]);
if (hitlStream.interrupted) {
  console.log("INTERRUPTED:");
  for (const interrupt of hitlStream.interrupts) {
    for (const request of interrupt.payload.actionRequests) {
      console.log(request.description);
    }
  }
}
```
```ts
import { Command } from "@langchain/langgraph";

const resumeStream = await agent.streamEvents(
  new Command({ resume: { decisions: [{ type: "approve" }] } }),
  { ...config, version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of resumeStream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of resumeStream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
    }
  })(),
]);
if (resumeStream.interrupted) {
  console.log("INTERRUPTED:");
  for (const interrupt of resumeStream.interrupts) {
    for (const request of interrupt.payload.actionRequests) {
      console.log(request.description);
    }
  }
}
```

## Next steps
* **Key Points:**
  - For deeper customization, check out this tutorial for implementing a SQL agent directly using LangGraph primitives.