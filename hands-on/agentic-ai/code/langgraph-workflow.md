---
aliases:
  - Langgraph Workflows
Source 1: https://docs.langchain.com/oss/javascript/langgraph/workflows-agents
---
# Workflows and agents


* **Key Points:**
  - Workflows have predetermined code paths and are designed to operate in a certain order.
  - Agents are dynamic and define their own processes and tool usage.
  - LangGraph offers several benefits when building agents and workflows, including persistence, streaming, and support for debugging as well as deployment.

## Setup
* **Key Points:**
  - To build a workflow or agent, you can use any chat model that supports structured outputs and tool calling.
* **Technical Entities (Classes/Functions/APIs):** `ChatAnthropic`

## LLMs and augmentations
* **Key Points:**
  - Workflows and agentic systems are based on LLMs and the various augmentations you add to them.
  - Tool calling, structured outputs, and short term memory are a few options for tailoring LLMs to your needs.
* **Technical Entities (Classes/Functions/APIs):** `z`, `tool`, `withStructuredOutput`, `bindTools`, `invoke()`, `tool_calls`
* **Code Snippet:**
```typescript
import * as z from "zod";
import { tool } from "langchain";

// Schema for structured output
const SearchQuery = z.object({
  search_query: z.string().describe("Query that is optimized web search."),
  justification: z
    .string()
    .describe("Why this query is relevant to the user's request."),
});

// Augment the LLM with schema for structured output
const structuredLlm = llm.withStructuredOutput(SearchQuery);

// Invoke the augmented LLM
const output = await structuredLlm.invoke(
  "How does Calcium CT score relate to high cholesterol?"
);

// Define a tool
const multiply = tool(
  ({ a, b }) => {
    return a * b;
  },
  {
    name: "multiply",
    description: "Multiply two numbers",
    schema: z.object({
      a: z.number(),
      b: z.number(),
    }),
  }
);

// Augment the LLM with tools
const llmWithTools = llm.bindTools([multiply]);

// Invoke the LLM with input that triggers the tool call
const msg = await llmWithTools.invoke("What is 2 times 3?");

// Get the tool call
console.log(msg.tool_calls);
```

## Prompt chaining
* **Key Points:**
  - Prompt chaining is when each LLM call processes the output of the previous call.
  - It's often used for performing well-defined tasks that can be broken down into smaller, verifiable steps.
  - Some examples include: Translating documents into different languages, Verifying generated content for consistency
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `StateSchema`, `GraphNode`, `ConditionalEdgeRouter`, `z`, `addNode`, `addEdge`, `addConditionalEdges`, `compile()`, `invoke()`
* **Code Snippet:**
```typescript
import { StateGraph, StateSchema, GraphNode, ConditionalEdgeRouter } from "@langchain/langgraph";
import { z } from "zod/v4";

// Graph state
const State = new StateSchema({
  topic: z.string(),
  joke: z.string(),
  improvedJoke: z.string(),
  finalJoke: z.string(),
});

// Define node functions

// First LLM call to generate initial joke
const generateJoke: GraphNode<typeof State> = async (state) => {
  const msg = await llm.invoke(`Write a short joke about ${state.topic}`);
  return { joke: msg.content };
};

// Gate function to check if the joke has a punchline
const checkPunchline: ConditionalEdgeRouter<typeof State, "improveJoke"> = (state) => {
  // Simple check - does the joke contain "?" or "!"
  if (state.joke?.includes("?") || state.joke?.includes("!")) {
    return "Pass";
  }
  return "Fail";
};

// Second LLM call to improve the joke
const improveJoke: GraphNode<typeof State> = async (state) => {
  const msg = await llm.invoke(
    `Make this joke funnier by adding wordplay: ${state.joke}`
  );
  return { improvedJoke: msg.content };
};

// Third LLM call for final polish
const polishJoke: GraphNode<typeof State> = async (state) => {
  const msg = await llm.invoke(
    `Add a surprising twist to this joke: ${state.improvedJoke}`
  );
  return { finalJoke: msg.content };
};

// Build workflow
const chain = new StateGraph(State)
  .addNode("generateJoke", generateJoke)
  .addNode("improveJoke", improveJoke)
  .addNode("polishJoke", polishJoke)
  .addEdge("__start__", "generateJoke")
  .addConditionalEdges("generateJoke", checkPunchline, {
    Pass: "improveJoke",
    Fail: "__end__"
  })
  .addEdge("improveJoke", "polishJoke")
  .addEdge("polishJoke", "__end__")
  .compile();
```

## Parallelization
* **Key Points:**
  - With parallelization, LLMs work simultaneously on a task.
  - This is either done by running multiple independent subtasks at the same time, or running the same task multiple times to check for different outputs.
  - Parallelization is commonly used to: Split up subtasks and run them in parallel, which increases speed, Run tasks multiple times to check for different outputs, which increases confidence
  - Some examples include: Running one subtask that processes a document for keywords, and a second subtask to check for formatting errors, Running a task multiple times that scores a document for accuracy based on different criteria, like the number of citations, the number of sources used, and the quality of the sources
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `StateSchema`, `GraphNode`, `z`, `addNode`, `addEdge`, `compile()`, `invoke()`, `Promise.all`

## Routing
* **Key Points:**
  - Routing workflows process inputs and then directs them to context-specific tasks.
  - This allows you to define specialized flows for complex tasks.
  - For example, a workflow built to answer product related questions might process the type of question first, and then route the request to specific processes for pricing, refunds, returns, etc.
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `StateSchema`, `GraphNode`, `ConditionalEdgeRouter`, `z`, `withStructuredOutput`, `addNode`, `addEdge`, `addConditionalEdges`, `compile()`, `invoke()`

## Orchestrator-worker
* **Key Points:**
  - In an orchestrator-worker configuration, the orchestrator: Breaks down tasks into subtasks, Delegates subtasks to workers, Synthesizes worker outputs into a final result
  - Orchestrator-worker workflows provide more flexibility and are often used when subtasks cannot be predefined the way they can with parallelization.
  - This is common with workflows that write code or need to update content across multiple files.
  - For example, a workflow that needs to update installation instructions for multiple Python libraries across an unknown number of documents might use this pattern.
  - Orchestrator-worker workflows are common and LangGraph has built-in support for them.
  - The `Send` API lets you dynamically create worker nodes and send them specific inputs.
  - Each worker has its own state, and all worker outputs are written to a shared state key that is accessible to the orchestrator graph.
  - This gives the orchestrator access to all worker output and allows it to synthesize them into a final output.
* **Technical Entities (Classes/Functions/APIs):** `Send`, `StateGraph`, `StateSchema`, `ReducedValue`, `GraphNode`, `ConditionalEdgeRouter`, `z`, `addNode`, `addEdge`, `addConditionalEdges`, `compile()`, `invoke()`
* **Code Snippet:**
```typescript
import { StateGraph, StateSchema, ReducedValue, GraphNode, Send } from "@langchain/langgraph";
import * as z from "zod";

// Graph state
const State = new StateSchema({
  topic: z.string(),
  sections: z.array(z.custom<SectionsSchema>()),
  completedSections: new ReducedValue(
    z.array(z.string()).default(() => []),
    { reducer: (a, b) => a.concat(b) }
  ),
  finalReport: z.string(),
});

// Worker state
const WorkerState = new StateSchema({
  section: z.custom<SectionsSchema>(),
  completedSections: new ReducedValue(
    z.array(z.string()).default(() => []),
    { reducer: (a, b) => a.concat(b) }
  ),
});

// Nodes
const orchestrator: GraphNode<typeof State> = async (state) => {
  // Generate queries
  const reportSections = await planner.invoke([
    { role: "system", content: "Generate a plan for the report." },
    { role: "user", content: `Here is the report topic: ${state.topic}` },
  ]);

  return { sections: reportSections.sections };
};

const llmCall: GraphNode<typeof WorkerState> = async (state) => {
  // Generate section
  const section = await llm.invoke([
    {
      role: "system",
      content: "Write a report section following the provided name and description. Include no preamble for each section. Use markdown formatting.",
    },
    {
      role: "user",
      content: `Here is the section name: ${state.section.name} and description: ${state.section.description}`,
    },
  ]);

  // Write the updated section to completed sections
  return { completedSections: [section.content] };
};

const synthesizer: GraphNode<typeof State> = async (state) => {
  // List of completed sections
  const completedSections = state.completedSections;

  // Format completed section to str to use as context for final sections
  const completedReportSections = completedSections.join("\n\n---\n\n");

  return { finalReport: completedReportSections };
};

// Conditional edge function to create llm_call workers that each write a section of the report
const assignWorkers: ConditionalEdgeRouter<typeof State, "llmCall"> = (state) => {
  // Kick off section writing in parallel via Send() API
  return state.sections.map((section) =>
    new Send("llmCall", { section })
  );
};

// Build workflow
const orchestratorWorker = new StateGraph(State)
  .addNode("orchestrator", orchestrator)
  .addNode("llmCall", llmCall)
  .addNode("synthesizer", synthesizer)
  .addEdge("__start__", "orchestrator")
  .addConditionalEdges(
    "orchestrator",
    assignWorkers,
    ["llmCall"]
  )
  .addEdge("llmCall", "synthesizer")
  .addEdge("synthesizer", "__end__")
  .compile();
```

## Evaluator-optimizer
* **Key Points:**
  - In evaluator-optimizer workflows, one LLM call creates a response and the other evaluates that response.
  - If the evaluator or a human-in-the-loop determines the response needs refinement, feedback is provided and the response is recreated.
  - This loop continues until an acceptable response is generated.
  - Evaluator-optimizer workflows are commonly used when there's particular success criteria for a task, but iteration is required to meet that criteria.
  - For example, there's not always a perfect match when translating text between two languages. It might take a few iterations to generate a translation with the same meaning across the two languages.
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `StateSchema`, `GraphNode`, `ConditionalEdgeRouter`, `z`, `withStructuredOutput`, `addNode`, `addEdge`, `addConditionalEdges`, `compile()`, `invoke()`

## Agents
* **Key Points:**
  - Agents are typically implemented as an LLM performing actions using tools.
  - They operate in continuous feedback loops, and are used in situations where problems and solutions are unpredictable.
  - Agents have more autonomy than workflows, and can make decisions about the tools they use and how to solve problems.
  - You can still define the available toolset and guidelines for how agents behave.
* **Technical Entities (Classes/Functions/APIs):** `tool`, `z`, `ToolNode`, `StateGraph`, `StateSchema`, `MessagesValue`, `GraphNode`, `ConditionalEdgeRouter`, `SystemMessage`, `ToolMessage`, `addNode`, `addEdge`, `addConditionalEdges`, `compile()`, `invoke()`
* **Code Snippet:**
```typescript
import { tool } from "@langchain/core/tools";
import * as z from "zod";

// Define tools
const multiply = tool(
  ({ a, b }) => {
    return a * b;
  },
  {
    name: "multiply",
    description: "Multiply two numbers together",
    schema: z.object({
      a: z.number().describe("first number"),
      b: z.number().describe("second number"),
    }),
  }
);

const add = tool(
  ({ a, b }) => {
    return a + b;
  },
  {
    name: "add",
    description: "Add two numbers together",
    schema: z.object({
      a: z.number().describe("first number"),
      b: z.number().describe("second number"),
    }),
  }
);

const divide = tool(
  ({ a, b }) => {
    return a / b;
  },
  {
    name: "divide",
    description: "Divide two numbers",
    schema: z.object({
      a: z.number().describe("first number"),
      b: z.number().describe("second number"),
    }),
  }
);

// Augment the LLM with tools
const tools = [add, multiply, divide];
const toolsByName = Object.fromEntries(tools.map((tool) => [tool.name, tool]));
const llmWithTools = llm.bindTools(tools);
```

### ToolNode
* **Key Points:**
  - `ToolNode` is a prebuilt node that executes tools in LangGraph workflows.
  - It handles parallel tool execution, error handling, and state injection automatically.
  - Use `ToolNode` when you need fine-grained control over how your graph executes tools.
  - This is the building block that powers tool execution in many LangGraph agent patterns.
  - Tools executed by `ToolNode` receive the arguments generated by the model as their first argument.
  - Tools can only access the state values passed to the `ToolNode`. When `ToolNode` is added directly as a `StateGraph` node, that input is the current graph state. If you invoke a `ToolNode` manually from another node, pass the full state when tools need custom state fields. For example, `tool_node.invoke(state)` or `toolNode.invoke(state, config)` exposes the full state, while passing only `{"messages": state["messages"]}` or `{ messages: state.messages }` only exposes `messages`.
* **Technical Entities (Classes/Functions/APIs):** `ToolNode`, `tool`, `z`, `ToolRuntime`, `MessagesValue`, `StateGraph`, `StateSchema`, `START`, `AIMessage`, `invoke()`
* **Code Snippet:**
```typescript
import { ToolNode } from "@langchain/langgraph/prebuilt";
import { tool } from "@langchain/core/tools";
import * as z from "zod";

const search = tool(
  ({ query }) => `Results for: ${query}`,
  {
    name: "search",
    description: "Search for information.",
    schema: z.object({ query: z.string() }),
  }
);

const calculator = tool(
  ({ expression }) => String(eval(expression)),
  {
    name: "calculator",
    description: "Evaluate a math expression.",
    schema: z.object({ expression: z.string() }),
  }
);

const toolNode = new ToolNode([search, calculator]);
```