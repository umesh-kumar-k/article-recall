---
aliases:
  - Memory System
Source 1: https://www.comet.com/site/blog/memory-in-langchain-a-deep-dive-into-persistent-context/
---
# Memory in LangChain: A Deep Dive into Persistent Context

## Memory in LangChain
* **Key Points:**
  - Remember Dory from 'Finding Nemo' and her notorious short-term memory loss? LLMs, especially chat-based ones, are like that. You need a way to ensure your system doesn't 'pull a Dory' building LLM applications. Luckily, LangChain has a memory module…

### What is it?
* **Key Points:**
  - In LangChain, the Memory module is responsible for persisting the state between calls of a chain or agent, which helps the language model remember previous interactions and use that information to make better decisions.
  - It provides a standard interface for persisting state between calls of a chain or agent, enabling the language model to have memory and context.
  - Memory is critical for personal assistants, autonomous agents, and agent simulations, where the language model needs to remember previous interactions to make informed decisions.

### What does it do?
* **Key Points:**
  - The Memory module enables the language model to have memory and context, allowing the LLM to make informed decisions.
  - It allows the model to remember user inputs, system responses, and any other relevant information. The stored information can be accessed and utilized during subsequent interactions.

### Why do I need it?
* **Key Points:**
  - The Memory module helps you build more interactive and personalized applications.
  - It gives the language model a sense of continuity and memory of past interactions. With memory, the model can provide more contextually relevant responses and make informed decisions based on previous inputs.

### When do I use it?
* **Key Points:**
  - You should use the Memory module whenever you want to create applications that require context and persistence between interactions.
  - It is handy for tasks like personal assistants, where the model needs to remember user preferences, previous queries, and other relevant information.

### Incorporating Memory
* **Key Points:**
  - Every memory system performs two main tasks: reading and writing. Every chain has core logic that requires specific inputs. Some inputs originate from the user, while others derive from memory. During a run, a chain accesses its memory system twice:
    - It reads from memory to supplement user inputs before executing core logic.
    - After processing but before responding, it writes the current run's data to memory for future reference.
  - Two fundamental decisions shape any memory system:
    - The method of storing state.
    - The approach to querying that state.
  - Storing: At the heart of memory lies a record of all chat interactions. LangChain's memory module offers various ways to store these chats, ranging from temporary in-memory lists to enduring databases.
  - Querying: While storing chat logs is straightforward, designing algorithms and structures to interpret them isn't. A basic memory system might display recent messages. A more advanced one could summarize the last 'K' messages. The most refined systems might identify entities from stored chats and present details only about those entities in the current session.
  - Different applications demand unique memory querying methods. LangChain's memory module simplifies the initiation with basic systems and supports creating tailored systems when necessary.

### Implementing Memory
* **Key Points:**
  - Setup prompt and memory
  - Initialize LLMChain
  - Call LLMChain
* **Technical Entities (Classes/Functions/APIs):** `langchain`, `openai`, `tiktoken`, `os`, `getpass`

## ConversationBufferMemory
* **Key Points:**
  - In this section, you will explore the Memory functionality in LangChain. Specifically, you will learn how to interact with an arbitrary memory class and use ConversationBufferMemory in chains. ConversationBufferMemory is a simple memory type that stores chat messages in a buffer and passes them to the prompt template.
* **Technical Entities (Classes/Functions/APIs):** `ConversationBufferMemory`, `add_user_message`, `add_ai_message`, `load_memory_variables`, `memory_key`, `return_messages`
* **Code Snippet:**
```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory()
memory.chat_memory.add_user_message("wagwan, bruv?")
memory.chat_memory.add_ai_message("Alright, guv'nor? Just been 'round the old manor, innit?")

memory.load_memory_variables({})
# {'history': "Human: wagwan, bruv?\nAI: Alright, guv'nor? Just been 'round the old manor, innit?"}
```

### Reading variables from memory
* **Key Points:**
  - Before going into the chain, variables are read from memory. Variable names need to align with what the chain expects. You can inspect variables by calling memory.load_memory_variables.
  - Notice that load_memory_variables returns a single key, history. This means that your chain (and likely your prompt) expects an input named history. You control this variable through parameters on the memory class. For example, if you want the memory variables to be returned in the key chat_history you can do the following.
* **Technical Entities (Classes/Functions/APIs):** `load_memory_variables`, `memory_key`

### Memory as strings, or list of strings
* **Key Points:**
  - When it comes to memory, one of the most common types is the storage and retrieval of chat messages. There are two ways to retrieve these messages:
    - A single string that concatenates all the messages together, which is useful when the messages will be passed in Language Models
    - A list of ChatMessages, which is useful when the messages are passed into ChatModels.
  - By default, chat messages are returned as a single string. However, if you want to return them as a list of messages, you can set the parameter return_messages to True.
* **Technical Entities (Classes/Functions/APIs):** `return_messages`, `HumanMessage`, `AIMessage`

### Keys in Memory
* **Key Points:**
  - In some cases, chains receive or provide multiple input/output keys. In this scenario, it'll be challenging to determine which keys to store in the chat message history. But you can manage this by using the input_key and output_key parameters in memory types. By default, these parameters are set to None, which means that if there is only one input/output key, it will be used. However, if there are multiple input/output keys, you must specify the name of the one to be used.

### Putting it all together
* **Key Points:**
  - Let's look at using this in an LLMChain and show working with both an LLM and a ChatModel.

#### Using an LLM
* **Technical Entities (Classes/Functions/APIs):** `OpenAI`, `PromptTemplate`, `LLMChain`, `ConversationBufferMemory`
* **Code Snippet:**
```python
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory

#instantiate the language model
llm = OpenAI(temperature=0.1)

# Look how "chat_history" is an input variable to the prompt template
template = """
You are Spider-Punk, Hobart Brown from Earth-138.
Your manner of speaking is rebellious and infused with punk rock lingo,
often quippy and defiant against authority.
Speak with confidence, wit, and a touch of brashness, always ready
to challenge the status quo with passion.
Your personality swings between that classic cockney sensibility
and immeasurable Black-British street swagger

Previous conversation:
{chat_history}

New human question: {question}
Response:
"""

prompt = PromptTemplate.from_template(template)

# Notice that we need to align the `memory_key`
memory = ConversationBufferMemory(memory_key="chat_history")

conversation = LLMChain(
    llm=llm,
    prompt=prompt,
    verbose=True,
    memory=memory
)

conversation({"question":"wagwan, bruv?"})
```

#### Using a ChatModel and exploring different memory types
* **Key Points:**
  - We'll use the ChatOpenAI model with the same prompt for all the following examples.
* **Technical Entities (Classes/Functions/APIs):** `ChatOpenAI`, `ChatPromptTemplate`, `MessagesPlaceholder`, `SystemMessagePromptTemplate`, `HumanMessagePromptTemplate`, `LLMChain`, `ConversationBufferMemory`
* **Code Snippet:**
```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import (
    ChatPromptTemplate,
    MessagesPlaceholder,
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate,
)
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory

llm = ChatOpenAI()

prompt = ChatPromptTemplate(
    messages=[
        SystemMessagePromptTemplate.from_template(
            """
            You are Spider-Punk, Hobart Brown from Earth-138.
            Your manner of speaking is rebellious and infused with punk rock lingo,
            often quippy and defiant against authority.
            Speak with confidence, wit, and a touch of brashness, always ready
            to challenge the status quo with passion.
            Your personality swings between that classic cockney sensibility
            and immeasurable Black-British street swagger
            """
        ),
        # The `variable_name` here is what must align with memory
        MessagesPlaceholder(variable_name="chat_history"),
        HumanMessagePromptTemplate.from_template("{question}")
    ]
)

# Notice that we `return_messages=True` to fit into the MessagesPlaceholder
# Notice that `"chat_history"` aligns with the MessagesPlaceholder name.
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)

conversation = LLMChain(
    llm=llm,
    prompt=prompt,
    verbose=True,
    memory=memory
)

conversation.predict(question="wagwan, bruv?")
```

## ConversationBufferWindowMemory
* **Key Points:**
  - The ConversationBufferWindowMemory is a tool that keeps track of past interactions in a conversation. It does this by maintaining a list of the most recent interactions, and only using the last K interactions. This helps to ensure that the buffer doesn't become too large and allows for a sliding window of the most recent interactions to be kept. This type of memory is beneficial for keeping the history of past interactions small and manageable. By only capturing the most recent interactions, it helps to prevent the buffer from becoming too large, which can be overwhelming and challenging to manage.
* **Technical Entities (Classes/Functions/APIs):** `ConversationBufferWindowMemory`, `ConversationChain`
* **Code Snippet:**
```python
from langchain.llms import OpenAI
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferWindowMemory

memory = ConversationBufferWindowMemory(k=2)

conversation_with_summary = ConversationChain(
    llm=OpenAI(temperature=0),
    memory=ConversationBufferWindowMemory(k=3),
    verbose=True
)

conversation_with_summary.predict(input="Wagwan, Bruv?")
```

## Wrapping Up Memory in LangChain
* **Key Points:**
  - As we've traversed the intricate pathways of LangChain's Memory module, it's evident how pivotal memory and context are in making interactions with language models feel genuine and continuous. No longer are we stuck in the cyclical loop of starting every conversation from scratch. With the power of LangChain's memory capabilities, every interaction can be a continuation of the last, allowing for richer and more personalized engagements.
  - Whether building a personal assistant, an autonomous agent, or running agent simulations, integrating memory is no longer a luxury — it's a necessity. Through the tools and strategies discussed, LangChain offers both simplicity for beginners and depth for experts.
  - But this is just the tip of the iceberg. In our upcoming piece, we will delve into more advanced memory types, showcasing how LangChain continuously pushes boundaries to offer even more nuanced and sophisticated memory solutions for varied applications.
  - In the vast landscape of language models, LangChain stands out, ensuring that every 'chat' feels like a chat and not a repeated introduction. So, the next time you engage with a chat model powered by LangChain, remember the unsung hero working behind the scenes: the Memory module, ensuring continuity and relevance in every conversation.