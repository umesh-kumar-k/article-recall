---
aliases:
  - Conversation Memory
Source 1: https://supermemory.ai/blog/how-to-add-conversational-memory-to-llms-using-langchain/
Source 2: https://zenvanriel.com/ai-engineer-blog/conversational-rag-systems/
Source 3: https://mischavandenburg.com/zet/conversation-memory-rag-production-one-gpu/
Source 4: https://medium.com/@thakermadhav/part-2-build-a-conversational-rag-with-langchain-and-mistral-7b-6a4ebe497185
---
# Code Reference

- [Memory Overview](./code/memory-overview.md) 
- [Basic Memory](./code/basic-memory.md) 
- [Short Term Memory](./code/short-term-memory.md) 
- [Long Term Memory](./code/long-term-memory.md) 
- [LangGraph Memory](./code/langgraph-memory.md) 
# How To Add Conversational Memory To LLMs Using LangChain

## Primer: What is Conversational Memory?

* **Key Points:**
  - Conversational memory allows chatbots to remember the context of their conversation with the user and tailor their answers to new prompts accordingly for a more intelligent experience.
  - It's basically meant to model how you'd talk to another person. Imagine trying to hold an hour-long conversation with someone who keeps forgetting everything you've said. Not fun, right?
  - At an architectural level, memory in LLMs works like this:
    - Input and Output History: Every user message and model response is captured.
    - Memory Storage: That conversation history is either: stored in the prompt window (ephemeral), summarized and compressed (short-term memory), or persisted in a separate, retrievable vector DB or external memory (long-term memory).
    - Retrieval Layer: At inference time, relevant pieces of past conversation are pulled in via raw replay, windowing, summarization, or vector similarity.
    - Augmentation: That context is appended to the current prompt before sending to the LLM.
* **Technical Entities (Classes/Functions/APIs):** `LangGraph`, `MemorySaver`, `StateGraph`, `MessagesState`

## LangGraph Memory Architecture: Short-Term vs Long-Term
* **Key Points:**
  - LangGraph introduces a new way of handling memory that's far more powerful than the old ConversationBufferMemory-style classes you might be used to. It supports both short-term and long-term memory through state management and memory stores.
  - LangGraph has a built-in persistence layer that allows it to remember what's going on. It has a few key concepts like state, threads, and checkpoints.
  - Think of building a chatbot with LangGraph as writing in a notebook. The state refers to the notebook page where you're recording the actual conversation: what's been said, what tools were used, what decisions were made. Every time something happens in your app, this state, or notebook page, gets updated with what happened.
  - Now, imagine that every user, or session, gets their own notebook. That's what a thread is: an isolated session that stores several different states and has a unique ID to identify it.
  - If a user returns tomorrow, you just give LangGraph the same thread ID, and it picks up right where you left off, flipping back to the last used page.
  - Now, let's say you want to bookmark key moments in your notebook so you can return to them. That's what checkpoints are: a snapshot of the state at a specific time. LangGraph automatically creates a checkpoint every time you invoke a graph with a thread ID.
* **Technical Entities (Classes/Functions/APIs):** `LangGraph`, `MemorySaver`, `StateGraph`, `MessagesState`

## How LangChain Implements Short-Term Memory
* **Key Points:**
  - You basically store the conversation history in the state, usually as a list of messages. Each time a user sends a message or the LLM generates a response, it is appended to this list, which is the short-term memory of the LLM. You can also store documents, uploaded files, and other metadata in the graph's state so that the LLM has access to the full context.
  - There are two main ways to maintain this list of messages:
    - Message Buffering: Keeps the last k messages in memory
    - Summarization: Replace older history with a summary

## Adding Conversational Memory To Our LLM

### Basic Setup
* **Key Points:**
  - First, we'll start by storing all the messages in our therapy chatbot's context, and then we'll move on to advanced techniques like message trimming and summarizing.
* **Technical Entities (Classes/Functions/APIs):** `langchain_openai`, `ChatOpenAI`, `SystemMessage`, `HumanMessage`, `MemorySaver`, `StateGraph`, `START`, `MessagesState`
* **Code Snippet:**
```python
from langchain_openai import ChatOpenAI
from langchain.schema import SystemMessage, HumanMessage
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, MessagesState
import os
```

### Basic Memory
* **Key Points:**
  - Let's start building the chatbot.
  - You'll start by setting your OpenAI API key as an environment variable.
  - Retrieve the environment variable.
  - Initialize your model.
  - The next step while building an LLM with LangGraph is to initialize the graph that represents the workflow that gets carried out.
  - Since the chatbot stores the chat history, this code defines MessagesState as the schema, which means the state contains a list of messages.
  - Now, if you think about it, graphs contain nodes that carry out the actual logic. Our main logic is going to be the chatbot that acts like a therapist, asks questions, generates responses, and stores them in its memory.
  - The node is passed the state, which is the chat history. Prompts to LLMs contain a system instruction (developer instructions on how the LLM must behave) and a human message.
  - The code instructs the LLM to act as a therapy assistant, and our prompt contains both the system message and the complete list of messages since we're passing the entire context to it. This prompt is sent to OpenAI, which returns a response.
  - The MemorySaver() stores the state in memory, and the builder.compile compiles and executes the graph with the memory checkpointer.
  - If you go back to our breakdown of LangChain's architecture, you'll remember that LangChain used a unique thread identifier to set a unique ID for each conversation and store its memory separately.
  - Finally, let's allow the user to enter messages and update our app's state accordingly with the following code.
  - This code takes the user input, wraps it in the HumanMessage class, and appends the list of messages stored. After that, the graph is invoked with chat_app.invoke. The result is then printed.
  - This approach is pretty simple and gives us a workable chatbot, but it has some severe drawbacks. LangGraph replays all the messages stored in memory for every new message, and the full system prompt + message history is sent to the LLM as input.
  - LLMs have a finite token limit. Eventually, the prompt becomes too large to fit into the model's context window, which leads to a shit ton of errors.
  - On top of that, more tokens = more cost = higher OpenAI bills. Our therapy chatbot would have long conversations, so that would become super expensive, super fast. Lastly, token-heavy prompts would also degrade performance.
* **Technical Entities (Classes/Functions/APIs):** `ChatOpenAI`, `StateGraph`, `MessagesState`, `MemorySaver`, `thread_id`
* **Code Snippet:**
```python
model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

builder = StateGraph(state_schema=MessagesState)

def chat_node(state: MessagesState):
    system_message = SystemMessage(content="You're a kind therapy assistant.")
    history = state["messages"]
    prompt = [system_message] + history
    response = model.invoke(prompt)
    return {"messages": response}

builder.add_node("chat", chat_node)
builder.add_edge(START, "chat")

memory = MemorySaver()
chat_app = builder.compile(checkpointer=memory)

thread_id = "1"

while True:
    user_input = input("You: ")
    state_update = {"messages": [HumanMessage(content=user_input)]}
    result = chat_app.invoke(
        state_update,
        {"configurable": {"thread_id": thread_id}}
    )
    print(result)
    ai_msg = result["messages"][-1]
    print("Bot:", ai_msg.content)
```

### Memory With Message Trimming
* **Key Points:**
  - Message trimming only stores a certain k number of messages in the memory, which prevents memory overload, stays within token limits, and focuses only on recent context.
  - You'll notice most of this is the same as the previous file. Except, now we're importing the trim_messages function.
  - This trimmer counts every message as 1 token, and only keeps the last 10 tokens (5 pairs of human + AI conversation).
  - First, the messages are trimmed by invoking the trimmer on the current state, and then those trimmed messages are passed to the prompt. Thus, the entire state is not passed, saving tokens.
  - As you can see, when I ask the what's my name question, the previous 5 memories don't contain it, thus the chatbot forgets. It's an improvement over the basic model, but it still has drawbacks:
    - Trimming may lead to loss of crucial context, which is not a part of the last 10 tokens.
    - Trimming isn't intelligent, so there's an arbitrary information cutoff.
    - Even with trimming, token creep might happen if the input/output is extremely long.
* **Technical Entities (Classes/Functions/APIs):** `trim_messages`, `from langchain_core.messages`
* **Code Snippet:**
```python
from langchain_core.messages import trim_messages

trimmer = trim_messages(strategy="last", max_tokens=10, token_counter=len)

def chat_node(state: MessagesState):
    trimmed_messages = trimmer.invoke(state["messages"])
    system_message = SystemMessage(content="You're a kind therapy assistant.")
    prompt = [system_message] + trimmed_messages
    response = model.invoke(prompt)
    return {"messages": response}
```

### Memory With Summarization
* **Key Points:**
  - Summarization allows the LLM to summarize all conversations before the current one, thus reducing token usage while also ensuring minimal context loss.
  - This has some more significant changes. Firstly, you're importing the RemoveMessages function, which wasn't there previously.
  - In terms of logic, there are changes in the chat_node function. The summarization logic only kicks in when the total conversation has gone over 8 messages.
  - If that happens, first the history is retrieved, except the latest HumanMessage, which is why we've used the [:-1] operator. Then, the model is invoked with a prompt to summarize the content, and the history variable is passed along with the instructions.
  - After that, all the summarized messages are removed from the state to save on tokens. The latest human message is wrapped again inside the HumanMessage() class, and then the model is invoked again with the summary and the human message, but this time with the therapy chatbot system message.
  - Finally, the model response is returned with the new information. If the history is less than 8 messages, then normal conversation chaining is used. This makes the chatbot efficient for shorter chats as well.
  - As you'll see, everything else from before was summarized, while the two latest messages are stored in memory. This is the best approach we've explored until now, but as always, there's always room for improvement. You see:
    - Summaries are only as good as the prompt + model, so the quality's a bit unreliable.
    - Hard deletes can be risky. If the summarization is off, you've lost the original data.
* **Technical Entities (Classes/Functions/APIs):** `RemoveMessage`, `from langchain_core.messages`
* **Code Snippet:**
```python
from langchain_core.messages import RemoveMessage

def chat_node(state: MessagesState):
    system_message = SystemMessage(content="You're a kind therapy assistant.")
    history = state["messages"][:-1]
    if len(history) >= 8:
        last_human_message = state["messages"][-1]
        summary_prompt = (
            "Distill the above chat messages into a single summary message. "
            "Include as many specific details as you can."
        )
        summary_message = model.invoke(history + [HumanMessage(content=summary_prompt)])
        delete_messages = [RemoveMessage(id=m.id) for m in state["messages"]]
        human_message = HumanMessage(content=last_human_message.content)
        response = model.invoke([system_message, summary_message, human_message])
        message_updates = [summary_message, human_message, response] + delete_messages
    else:
        message_updates = model.invoke([system_message] + state["messages"])
    return {"messages": message_updates}
```

## Evaluating both the techniques
* **Key Points:**
  - For evaluation of the trimming and summarizing chatbots, we created 6 generic questions/responses we'd pass into the chatbots to see how they'd perform. We're also measuring them across 6 quantitative metrics:
    - Fact Retention: Mentions or uses previous facts correctly (name, job, etc.). Scoring (0–1). 1 = correct use; 0 = wrong/missing.
    - Entity Count: Number of correctly named entities reused. 0 to worse chatbot; 1 to the better one. If tied, 1 to both. Proxy for detail richness.
    - Latency / Turn Depth: How far back memory is used (e.g., Turn 1 fact used in Turn 6). 0-1. Higher is better.
    - Token Usage: Total tokens used in the conversation. 0-1. Counted via OpenAI or LangSmith tools.
    - Response Length: Average word/token count per AI reply. 0-1. Spot unnecessarily long answers.
    - Repetition Penalty: Counts of repeated generic phrases. -1 or 0. Penalize fluff like "I understand how you feel."
  - Our 6 prompts are as follows:
    - "Hi, I'm John. I've been feeling overwhelmed lately."
    - "It started after I got promoted to product manager.."
    - "I usually go for a run in the morning, but I've stopped doing that recently."
    - "My therapist told me to start journaling, but I haven't."
    - "Also, I had a fight with my brother Jack yesterday."
    - "Can you help me make a plan to feel more in control again?"

### Trimming (set at 8 tokens)
* **Key Points:**
  - Tokens used: 1.618K completion tokens

### Summarizing (set at 8 messages)
* **Key Points:**
  - Tokens used: 1,940 tokens

### Evaluation Results
* **Key Points:**
  - Metric | Description | Trimming | Summarization | Notes
  - Fact Retention | Correct recall/use of facts from earlier turns | 1 | 1 | Both bots correctly referenced all six facts (John, promotion, running, journaling, Jack, and feeling overwhelmed).
  - Entity Count | Number of correctly named entities reused | 1 | 1 | Both mentioned John, Jack, running, journaling, etc. Summarization didn't sacrifice detail.
  - Latency / Turn Depth | How far back a memory is reused (e.g., Turn 1 info in Turn 6) | 0 | 1 | Summarization reused context from all 5 previous turns in the final response, trimming lost older context earlier.
  - Token Usage | Total tokens used | 1 | 0 | Trimming uses fewer tokens due to shorter prompts and hard message cuts; summarization adds summarizer calls and more verbose responses.
  - Avg. Response Length | Average length of bot replies | 1 | 0 | Trimming's responses are shorter, while summarization is more verbose; it can be less desirable for token-sensitive applications.
  - Repetition Penalty | Penalty for repeated generic phrases (lower = better) | -1 | 0 | Trimming led to more generic fallback lines ("That's understandable."); summarization preserved specificity despite verbosity.
  - Final scores are 3-3 for both models, tied. While both trimming and summarization scored equally in our evaluation, the tie doesn't imply that the two strategies are interchangeable.
  - Trimming excels in token economy and snappy responses. It's simple, lightweight, and great for short or transactional conversations. But once the dialogue gets deeper or longer, it starts forgetting context, leading to generic or disconnected replies, which earns it a repetition penalty.
  - Summarization, on the other hand, maintains coherent long-form memory. It preserves emotional and factual continuity over time, allowing for more natural conversations. But this comes at a cost: higher token usage and longer responses, which might not be ideal for production environments with strict token budgets.

## Supermemory
* **Key Points:**
  - With Supermemory, you can add memory with just one line of code. That's it. Supermemory automatically ingests and manages the context, using a combination of Graphs and Vector store.
* **Technical Entities (Classes/Functions/APIs):** `OpenAI`, `baseUrl: "https://api.supermemory.ai/v3/https://api.openai.com/v1/", headers: {"x-supermemory-user-id": "dhravya"}`
* **Code Snippet:**
```javascript
import OpenAI from "openai"

const client = new OpenAI({
    baseUrl: "https://api.supermemory.ai/v3/https://api.openai.com/v1/"
}, {
    headers: {
        "x-supermemory-user-id": "dhravya"
    }
})
```

## Conclusion
* **Key Points:**
  - This article covered everything from how conversational memory works to implementing it in LangChain, using both trimming and summarizing, and then evaluating them for their particular use cases.
  - However, it starts to fall apart as complexity grows. If you're building apps and AI agents that need to remember across sessions, adapt to users, or scale without manual patchwork, Supermemory's Memory API might be a better solution for you.


---


# Build a Conversational RAG with Mistral-7B and LangChain

## Before diving into the advanced aspects
* **Key Points:**
  - Before diving into the advanced aspects of building Retrieval-Augmented Generation (RAG) applications with LangChain, it is crucial to first explore the foundational groundwork laid out in Part 1 of this series. If you haven't read it yet, I strongly recommend starting there.
  - Part 1 provides an essential overview of the limitations in existing Large Language Models (LLMs) and how RAGs can effectively address these challenges. It also guides you through the process of constructing a basic RAG application, which is a vital step in grasping the core concepts of RAG technology.
  - While Part 1 was instrumental in building your initial understanding, it's important to recognize that it was just a preliminary step. Real-world applications of RAG technology demand a more nuanced approach. In this article, we will explore the complexities that come with refining a RAG application to provide a smooth, conversational user experience.

## Conversational RAG Architecture
* **Key Points:**
  - Before we go further, it's important to note that there are abstracted classes available which can simplify a lot of this work for us. Here are a couple examples: Retrieval QA; RAGs with Agents.
  - However, having a solid understanding of what's happening 'under the hood' is crucial which is why this tutorial will leverage low level LangChain components. Working at this level will let you understand why you may or may not be getting the result you are expecting and ultimately have more control over your RAG application.
  - In the highlighted section, we pass in the query as is but really we need to pass a transformed version that can appropriately query our vector database.
  - Let's review the key updates: We now save the conversation history to memory and leverage it to generate a standalone question; We add a second LLM which will be responsible for generating a standalone question that can appropriately query the vector database.
* **Technical Entities (Classes/Functions/APIs):** `Retrieval QA`, `RAGs with Agents`, `LangChain`

## How do we build it?
* **Key Points:**
  - Similar to my original article, I'm going to use an article from fantasypros.com and ask the LLM questions that it would only be able to answer if it has access to that data.

### Create both LLM pipelines
* **Key Points:**
  - We will load the model the exact same way as we did in the original article. We're using the following model/machine to build this: Model: mistralai/Mistral-7B-Instruct-v0.2; Machine: 1 Nvidia L4 GPU
  - The standalone_query_generation_pipeline uses a temperature of 0.0 instead of 0.2 for our response_generation_pipeline. I do this because I want to make sure there is as little chance for hallucinations when generating the standalone query since that impacts the application's ability to retrieve relevant context.
  - For those unfamiliar with temperature, the excerpt highlights why 0.0 makes sense for our standalone_query_generation_pipeline. More information on temperature and other LLM parameters can be found here. Temperature is a close second to prompt engineering when it comes to controlling the output of the Generate model. It determines how creative the model should be. A Temperature of 0 makes the model deterministic. It limits the model to use the word with the highest probability. You can run it over and over and get the same output. As you increase the Temperature, the limit softens, allowing it to use words with lower and lower probabilities.
  - Now, in this example, I've just used the same Mistral model with different temperate settings but you really could/should use different LLMs altogether for both. For example, you could leverage (or build) a fine-tuned model that is optimized for the standalone query generate task.
* **Technical Entities (Classes/Functions/APIs):** `mistralai/Mistral-7B-Instruct-v0.2`, `HuggingFacePipeline`, `pipeline`, `temperature`
* **Code Snippet:**
```python
from transformers import pipeline
from langchain.llms import HuggingFacePipeline

standalone_query_generation_pipeline = pipeline(
    model=mistral_model,
    tokenizer=tokenizer,
    task="text-generation",
    temperature=0.0,
    repetition_penalty=1.1,
    return_full_text=True,
    max_new_tokens=1000,
)
standalone_query_generation_llm = HuggingFacePipeline(pipeline=standalone_query_generation_pipeline)

response_generation_pipeline = pipeline(
    model=mistral_model,
    tokenizer=tokenizer,
    task="text-generation",
    temperature=0.2,
    repetition_penalty=1.1,
    return_full_text=True,
    max_new_tokens=1000,
)
response_generation_llm = HuggingFacePipeline(pipeline=response_generation_pipeline)
```

### Standalone Questions Generation Chain
* **Key Points:**
  - I decided to use a few shot prompt engineering approach to help guide the LLM.
  - 7B models are performant but they're not perfect so providing a handful of examples in the prompt is a good idea.
  - The most important step here is being able to store the conversation history. Fortunately, LangChain makes it easy for us to do that.
* **Technical Entities (Classes/Functions/APIs):** `PromptTemplate`, `ChatPromptTemplate`, `ConversationBufferMemory`, `RunnablePassthrough`, `RunnableLambda`, `itemgetter`, `get_buffer_string`
* **Code Snippet:**
```python
from langchain.prompts.prompt import PromptTemplate
from langchain_core.prompts.chat import ChatPromptTemplate
_template = """
[INST] 
Given the following conversation and a follow up question, 
rephrase the follow up question to be a standalone question, in its original language, 
that can be used to query a FAISS index. This query will be used to retrieve documents with additional context.

Let me share a couple examples.

If you do not see any chat history, you MUST return the "Follow Up Input" as is:
...
Chat History:
Follow Up Input: How is Lawrence doing?
Standalone Question:
How is Lawrence doing?

If this is the second question onwards, you should properly rephrase the question like this:
...
Chat History:
Human: How is Lawrence doing?
AI: 
Lawrence is injured and out for the season.
Follow Up Input: What was his injury?
Standalone Question:
What was Lawrence's injury?

Now, with those examples, here is the actual chat history and input question.
Chat History:
{chat_history}
Follow Up Input: {question}
Standalone question:
[your response here]
[/INST] 
"""

STANDALONE_QUESTION_PROMPT = PromptTemplate.from_template(_template)
```
```python
from langchain.schema import format_document
from langchain_core.messages import AIMessage, HumanMessage, get_buffer_string
from langchain_core.runnables import RunnableParallel
from langchain_core.runnables import RunnableLambda, RunnablePassthrough

# Instantiate ConversationBufferMemory
memory = ConversationBufferMemory(
    return_messages=True, output_key="answer", input_key="question"
)

# First, load the memory to access chat history
loaded_memory = RunnablePassthrough.assign(
    chat_history=RunnableLambda(memory.load_memory_variables) | itemgetter("history"),
)

# Define the standalone_question step to process the question and chat history
standalone_question = {
    "standalone_question": {
        "question": lambda x: x["question"],
        "chat_history": lambda x: get_buffer_string(x["chat_history"]),
    }
    | STANDALONE_QUESTION_PROMPT,
}

# Finally, output the result of the CONDENSE_QUESTION_PROMPT
output_prompt = {
    "standalone_question_prompt_result": itemgetter("standalone_question"),
}

# Combine the steps into a final chain
standalone_query_generation_prompt = loaded_memory | standalone_question | output_prompt
```

### Complete Chain
* **Key Points:**
  - Fortunately, the hard part is done! We now need to take the output of the Standalone Query Generation chain and query our vector database to retrieve relevant context.
  - Now, this isn't necessary but it is a best practice. This allows us to: Include additional clean up of the input documents; Process and summarize retrieved documents. This can come in handy to avoid overly verbose prompt strings.
  - Now we have a string version of the retrieved documents and standalone question ready for the response generation LLM to provide a final response to the user.
  - To polish this up a bit more, I also include the standalone question and context into the output dictionary. This is useful information to have in a RAG application.
* **Technical Entities (Classes/Functions/APIs):** `ChatPromptTemplate`, `format_document`, `retriever`
* **Code Snippet:**
```python
template = """
[INST] 
Answer the question based only on the following context:
{context}

Question: {standalone_question}
[/INST] 
"""

RESPONSE_PROMPT = ChatPromptTemplate.from_template(template)

DEFAULT_DOCUMENT_PROMPT = PromptTemplate.from_template(template="{page_content}")

def _combine_documents(
    docs, document_prompt=DEFAULT_DOCUMENT_PROMPT, document_separator="\n\n"
):
    doc_strings = [format_document(doc, document_prompt) for doc in docs]
    return document_separator.join(doc_strings)

# First we add a step to load memory
# This adds a "memory" key to the input object
loaded_memory = RunnablePassthrough.assign(
    chat_history=RunnableLambda(memory.load_memory_variables) | itemgetter("history"),
)

# Now we calculate the standalone question
standalone_question = {
    "standalone_question": {
        "question": lambda x: x["question"],
        "chat_history": lambda x: get_buffer_string(x["chat_history"]),
    }
    | CONDENSE_QUESTION_PROMPT
    | standalone_query_generation_llm,
}

# Now we retrieve the documents
retrieved_documents = {
    "docs": itemgetter("standalone_question") | retriever,
    "standalone_question": lambda x: x["standalone_question"],
}

# Now we construct the inputs for the final prompt
final_inputs = {
    "context": lambda x: _combine_documents(x["docs"]),
    "standalone_question": itemgetter("standalone_question"),
}

# And finally, we do the part that returns the answers
answer = {
    "answer": final_inputs | ANSWER_PROMPT | response_generation_llm,
    "standalone_question": itemgetter("standalone_question"),
    "context": final_inputs["context"]
}

# And now we put it all together!
final_chain = loaded_memory | standalone_question | retrieved_documents | answer
```

### Examples
* **Key Points:**
  - We see that "Who are good alternatives to Mahomes right now?" was translated to "Who are some good alternatives to Mahomes as quarterback right now?" which is what we want! Now that question was used to retrieve relevant documents from our vector database.
  - Again, "How many PPG are both averaging?" was translated to "What is the average points per game (PPG) for both Baker Mayfield and Joe Flacco?". This standalone question LLM was able to understand who I meant when I said "both". And with that, we have a RAG that you can naturally converse with!

## Recap
* **Key Points:**
  - To recap, we further explored the practicalities of creating a conversational RAG. We learned the importance of maintaining conversation history and transforming input questions into standalone queries that effectively retrieve relevant information from vector databases. The use of two distinct LLMs, one for generating standalone queries and the other for generating responses, demonstrated a significant improvement in contextual understanding and relevance of the responses.
  - The step-by-step guide to building a conversational RAG highlighted the power and flexibility of LangChain in managing conversation flows and memory, as well as the effectiveness of Mistral in generating precise queries and responses. This approach ensures that each query is contextually relevant to the ongoing conversation, leading to more accurate and useful answers.