---
aliases:
  - Langchain Memory Overview
Source 1: https://docs.langchain.com/oss/javascript/concepts/memory
---
# Memory overview

## Short-term memory
* **Key Points:**
  - Short-term memory lets your application remember previous interactions within a single thread or conversation. A thread organizes multiple interactions in a session, similar to the way email groups messages in a single conversation.
  - LangGraph manages short-term memory as part of the agent's state, persisted via thread-scoped checkpoints. This state can normally include the conversation history along with other stateful data, such as uploaded files, retrieved documents, or generated artifacts. By storing these in the graph's state, the bot can access the full context for a given conversation while maintaining separation between different threads.
* **Technical Entities (Classes/Functions/APIs):** `LangGraph`, `thread-scoped checkpoints`

### Manage short-term memory
* **Key Points:**
  - Conversation history is the most common form of short-term memory, and long conversations pose a challenge to today's LLMs. A full history may not fit inside an LLM's context window, resulting in an irrecoverable error. Even if your LLM supports the full context length, most LLMs still perform poorly over long contexts. They get "distracted" by stale or off-topic content, all while suffering from slower response times and higher costs.
  - Chat models accept context using messages, which include developer provided instructions (a system message) and user inputs (human messages). In chat applications, messages alternate between human inputs and model responses, resulting in a list of messages that grows longer over time. Because context windows are limited and token-rich message lists can be costly, many applications can benefit from using techniques to manually remove or forget stale information.
* **Technical Entities (Classes/Functions/APIs):** `Filter`

## Long-term memory
* **Key Points:**
  - Long-term memory in LangGraph allows systems to retain information across different conversations or sessions. Unlike short-term memory, which is thread-scoped, long-term memory is saved within custom "namespaces."
  - Long-term memory is a complex challenge without a one-size-fits-all solution. However, the following questions provide a framework to help you navigate the different techniques:
    - What is the type of memory? Humans use memories to remember facts (semantic memory), experiences (episodic memory), and rules (procedural memory). AI agents can use memory in the same ways. For example, AI agents can use memory to remember specific facts about a user to accomplish a task.
    - When do you want to update memories? Memory can be updated as part of an agent's application logic (e.g., "on the hot path"). In this case, the agent typically decides to remember facts before responding to a user. Alternatively, memory can be updated as a background task (logic that runs in the background / asynchronously and generates memories).
* **Technical Entities (Classes/Functions/APIs):** `LangGraph`, `namespaces`, `Store`, `CoALA paper`

### Memory Types Comparison
* **Key Points:**
  - Memory Type | What is Stored | Human Example | Agent Example
  - Semantic | Facts | Things I learned in school | Facts about a user
  - Episodic | Experiences | Things I did | Past agent actions
  - Procedural | Instructions | Instincts or motor skills | Agent system prompt

### Semantic memory
* **Key Points:**
  - Semantic memory, both in humans and AI agents, involves the retention of specific facts and concepts. In humans, it can include information learned in school and the understanding of concepts and their relationships. For AI agents, semantic memory is often used to personalize applications by remembering facts or concepts from past interactions.
  - Semantic memory is different from "semantic search," which is a technique for finding similar content using "meaning" (usually as embeddings). Semantic memory is a term from psychology, referring to storing facts and knowledge, while semantic search is a method for retrieving information based on meaning rather than exact matches.
* **Technical Entities (Classes/Functions/APIs):** `profile`, `collection`, `Trustcall`, `LangSmith`, `Store`

#### Profile
* **Key Points:**
  - Memories can be a single, continuously updated "profile" of well-scoped and specific information about a user, organization, or other entity (including the agent itself). A profile is generally just a JSON document with various key-value pairs you've selected to represent your domain.
  - When remembering a profile, you will want to make sure that you are updating the profile each time. As a result, you will want to pass in the previous profile and ask the model to generate a new profile (or some JSON patch to apply to the old profile). This can be become error-prone as the profile gets larger, and may benefit from splitting a profile into multiple documents or strict decoding when generating documents to ensure the memory schemas remains valid.

#### Collection
* **Key Points:**
  - Alternatively, memories can be a collection of documents that are continuously updated and extended over time. Each individual memory can be more narrowly scoped and easier to generate, which means that you're less likely to lose information over time. It's easier for an LLM to generate new objects for new information than reconcile new information with an existing profile. As a result, a document collection tends to lead to higher recall downstream.
  - However, this shifts some complexity memory updating. The model must now delete or update existing items in the list, which can be tricky. In addition, some models may default to over-inserting and others may default to over-updating. See the Trustcall package for one way to manage this and consider evaluation (e.g., with a tool like LangSmith) to help you tune the behavior.
  - Working with document collections also shifts complexity to memory search over the list. The Store currently supports both semantic search and filtering by content.
  - Finally, using a collection of memories can make it challenging to provide comprehensive context to the model. While individual memories may follow a specific schema, this structure might not capture the full context or relationships between memories. As a result, when using these memories to generate responses, the model may lack important contextual information that would be more readily available in a unified profile approach.

### Episodic memory
* **Key Points:**
  - Episodic memory, in both humans and AI agents, involves recalling past events or actions. The CoALA paper frames this well: facts can be written to semantic memory, whereas experiences can be written to episodic memory. For AI agents, episodic memory is often used to help an agent remember how to accomplish a task.
  - In practice, episodic memories are often implemented through few-shot example prompting, where agents learn from past sequences to perform tasks correctly. Sometimes it's easier to "show" than "tell" and LLMs learn well from examples. Few-shot learning lets you "program" your LLM by updating the prompt with input-output examples to illustrate the intended behavior. While various best-practices can be used to generate few-shot examples, often the challenge lies in selecting the most relevant examples based on user input.
  - Note that the memory store is just one way to store data as few-shot examples. If you want to have more developer involvement, or tie few-shots more closely to your evaluation harness, you can also use a LangSmith Dataset to store your data and implement your own retrieval logic to select the most relevant examples based on user input.
* **Technical Entities (Classes/Functions/APIs):** `few-shot example prompting`, `LangSmith Dataset`

### Procedural memory
* **Key Points:**
  - Procedural memory, in both humans and AI agents, involves remembering the rules used to perform tasks. In humans, procedural memory is like the internalized knowledge of how to perform tasks, such as riding a bike via basic motor skills and balance. Episodic memory, on the other hand, involves recalling specific experiences, such as the first time you successfully rode a bike without training wheels or a memorable bike ride through a scenic route. For AI agents, procedural memory is a combination of model weights, agent code, and agent's prompt that collectively determine the agent's functionality.
  - In practice, it is fairly uncommon for agents to modify their model weights or rewrite their code. However, it is more common for agents to modify their own prompts.
  - One effective approach to refining an agent's instructions is through "Reflection" or meta-prompting. This involves prompting the agent with its current instructions (e.g., the system prompt) along with recent conversations or explicit user feedback. The agent then refines its own instructions based on this input. This method is particularly useful for tasks where instructions are challenging to specify upfront, as it allows the agent to learn and adapt from its interactions.
* **Technical Entities (Classes/Functions/APIs):** `Reflection`, `meta-prompting`, `LangGraph memory store`
* **Code Snippet:**
```javascript
// Node that *uses* the instructions
const callModel = async (state: State, store: BaseStore) => {
    const namespace = ["agent_instructions"];
    const instructions = await store.get(namespace, "agent_a");
    // Application logic
    const prompt = promptTemplate.format({
        instructions: instructions[0].value.instructions
    });
    // ...
};

// Node that updates instructions
const updateInstructions = async (state: State, store: BaseStore) => {
    const namespace = ["instructions"];
    const currentInstructions = await store.search(namespace);
    // Memory logic
    const prompt = promptTemplate.format({
        instructions: currentInstructions[0].value.instructions,
        conversation: state.messages
    });
    const output = await llm.invoke(prompt);
    const newInstructions = output.new_instructions;
    await store.put(["agent_instructions"], "agent_a", {
        instructions: newInstructions
    });
    // ...
};
```

## Writing memories
* **Key Points:**
  - There are two primary methods for agents to write memories: "in the hot path" and "in the background".

### In the hot path
* **Key Points:**
  - Creating memories during runtime offers both advantages and challenges. On the positive side, this approach allows for real-time updates, making new memories immediately available for use in subsequent interactions. It also enables transparency, as users can be notified when memories are created and stored.
  - However, this method also presents challenges. It may increase complexity if the agent requires a new tool to decide what to commit to memory. In addition, the process of reasoning about what to save to memory can impact agent latency. Finally, the agent must multitask between memory creation and its other responsibilities, potentially affecting the quantity and quality of memories created.
  - As an example, ChatGPT uses a save_memories tool to upsert memories as content strings, deciding whether and how to use this tool with each user message.
* **Technical Entities (Classes/Functions/APIs):** `save_memories tool`, `memory-agent template`

### In the background
* **Key Points:**
  - Creating memories as a separate background task offers several advantages. It eliminates latency in the primary application, separates application logic from memory management, and allows for more focused task completion by the agent. This approach also provides flexibility in timing memory creation to avoid redundant work.
  - However, this method has its own challenges. Determining the frequency of memory writing becomes crucial, as infrequent updates may leave other threads without new context. Deciding when to trigger memory formation is also important. Common strategies include scheduling after a set time period (with rescheduling if new events occur), using a cron schedule, or allowing manual triggers by users or the application logic.
* **Technical Entities (Classes/Functions/APIs):** `memory-service template`

## Memory storage
* **Key Points:**
  - LangGraph stores long-term memories as JSON documents in a store. Each memory is organized under a custom namespace (similar to a folder) and a distinct key (like a file name). Namespaces often include user or org IDs or other labels that makes it easier to organize information. This structure enables hierarchical organization of memories. Cross-namespace searching is then supported through content filters.
* **Technical Entities (Classes/Functions/APIs):** `InMemoryStore`, `put`, `get`, `search`, `BaseStore`, `@langchain/langgraph`
* **Code Snippet:**
```javascript
import { InMemoryStore } from "@langchain/langgraph";

const embed = (texts: string[]): number[][] => {
    // Replace with an actual embedding function or LangChain embeddings object
    return texts.map(() => [1.0, 2.0]);
};

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore({ index: { embed, dims: 2 } });
const userId = "my-user";
const applicationContext = "chitchat";
const namespace = [userId, applicationContext];

await store.put(
    namespace,
    "a-memory",
    {
        rules: [
            "User likes short, direct language",
            "User only speaks English & TypeScript",
        ],
        "my-key": "my-value",
    }
);

// get the "memory" by ID
const item = await store.get(namespace, "a-memory");

// search for "memories" within this namespace, filtering on content equivalence, sorted by vector similarity
const items = await store.search(
    namespace,
    {
        filter: { "my-key": "my-value" },
        query: "language preferences"
    }
);
```