---
aliases:
  - Langchain Essentials
Source 1: https://medium.com/@mehtameet115/langchain-essentials-your-complete-guide-to-building-ai-applications-part-1-935cc0179837
---
# LangChain Essentials: Your Complete Guide to Building AI Applications (Part 1)

## 1. Prompt Templates: Stop Copy-Pasting, Start Templating
* **Key Points:**
  - Remember when you discovered variables in programming? Prompt templates are basically that, but for AI conversations.
  - Instead of writing this every single time: `You are a helpful assistant that translates English to Spanish. Please translate: Hello, how are you?"` You create a template like this.
  - Why this matters for your projects: Consistency: Your AI behaves the same way every time; Flexibility: Change languages, topics, or styles without rewriting everything; Professionalism: Your code looks clean and maintainable.
  - Real world use case: Building a study buddy that explains concepts in different subjects. One template, multiple subjects — just change the {subject} variable.
* **Technical Entities (Classes/Functions/APIs):** `ChatPromptTemplate`, `from_messages`, `invoke`, `langchain_core.prompts`
* **Code Snippet:**
```python
from langchain_core.prompts import ChatPromptTemplate
template = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant that translates English to {language}."),
    ("user", "{text}")
])
# Now you can reuse it
formatted_prompt = template.invoke({
    "language": "Spanish", 
    "text": "Hello, how are you?"
})
```

### 1.1 Role-Based Prompts: Teaching AI to Play Different Characters
* **Key Points:**
  - The coolest part? You can give your AI different personalities.
  - Think Netflix recommendations but for AI behavior — different prompts for different moods and tasks.
* **Technical Entities (Classes/Functions/APIs):** `ChatPromptTemplate`
* **Code Snippet:**
```python
# The tutor prompt
tutor_template = ChatPromptTemplate.from_messages([
    ("system", "You are a patient tutor explaining {subject} to a college student."),
    ("user", "{question}")
])
# The critic prompt  
critic_template = ChatPromptTemplate.from_messages([
    ("system", "You are a constructive critic reviewing {type} work."),
    ("user", "{content}")
])
```

### 1.2 Few-Shot Prompting: Teaching by Example
* **Key Points:**
  - Want your AI to learn from examples? Use few-shot prompting.
* **Technical Entities (Classes/Functions/APIs):** `examples`, `example_template`
* **Code Snippet:**
```python
examples = [
    {"input": "I was charged twice for my subscription this month.", "output": "Billing Issue"},
    {"input": "The app crashes every time I try to log in.", "output": "Technical Problem"},
    {"input": "Can you explain how to upgrade my plan?", "output": "General Inquiry"},
]
example_template = """
Ticket: {input}
Category: {output}
"""
```

## 2. Models: Your AI Brain
* **Key Points:**
  - Models are where the magic happens. They're like having different experts on speed dial — some are great at creative writing, others excel at coding, and some are perfect for quick answers.
  - Pro tip: Start with free models like Gemini or use HuggingFace (we'll cover this next) before investing in paid APIs. Your student budget will thank you later.
  - Think of it like choosing the right tool for the job: Quick tasks: Use faster, smaller models; Complex reasoning: Go for the big guns like GPT-4 or Gemini Pro; Budget projects: HuggingFace has tons of free options.
* **Technical Entities (Classes/Functions/APIs):** `ChatGoogleGenerativeAI`, `langchain_google_genai`
* **Code Snippet:**
```python
from langchain_google_genai import ChatGoogleGenerativeAI
# Your AI brain
llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash-001")
```

## 3. Chains: The Assembly Line of AI
* **Key Points:**
  - Chains are where LangChain gets its name. Instead of doing everything in one giant prompt, you break tasks into smaller, manageable steps.
  - What just happened? Template formats your prompt; LLM processes it; Parser cleans up the output. It's like a factory assembly line, but for AI tasks.
* **Technical Entities (Classes/Functions/APIs):** `StrOutputParser`, `chain`, `invoke`, `langchain_core.output_parsers`
* **Code Snippet:**
```python
from langchain_core.output_parsers import StrOutputParser
# Create the assembly line
chain = template | llm | StrOutputParser()
# Run it
result = chain.invoke({"language": "French", "text": "Good morning!"})
```

### 3.1 Advanced Chaining: Parallel Processing
* **Key Points:**
  - Want to do multiple things at once? Use parallel chains.
  - Perfect for creating study materials — generate notes AND practice questions at the same time.
* **Technical Entities (Classes/Functions/APIs):** `RunnableParallel`, `langchain_core.runnables`
* **Code Snippet:**
```python
from langchain_core.runnables import RunnableParallel
# Create two different chains
notes_chain = notes_prompt | llm | output_parser
quiz_chain = quiz_prompt | llm | output_parser
# Run them simultaneously
parallel_chain = RunnableParallel({
    "notes": notes_chain,
    "quiz": quiz_chain
})
result = parallel_chain.invoke({"topic": "Machine Learning"})
```

### 3.2 Conditional Logic with Branches
* **Key Points:**
  - Sometimes you need your AI to make decisions.
  - Great for building smart systems that respond differently based on user input.
* **Technical Entities (Classes/Functions/APIs):** `RunnableBranch`, `langchain_core.runnables`
* **Code Snippet:**
```python
from langchain_core.runnables import RunnableBranch

def send_follow_up_email(rating: int):
    print(f"Rating is {rating}. Sending a follow-up email.")
    return "Email Sent"

def log_positive_feedback(rating: int):
    print(f"Rating is {rating}. Logging feedback.")
    return "Feedback Logged"

conditional_branch = RunnableBranch(
    (lambda rating: rating < 5, send_follow_up_email),
    log_positive_feedback,  # Default action
)
```

## 4. HuggingFace Integration: Free AI Models for Everyone
* **Key Points:**
  - Here's where things get exciting for broke college students — HuggingFace gives you access to thousands of AI models, many completely free!
  - When to use which: Endpoint: When you want convenience and don't mind internet dependency; Pipeline: When you want full control and have decent hardware (8GB+ RAM recommended).
* **Technical Entities (Classes/Functions/APIs):** `HuggingFaceEndpoint`, `HuggingFacePipeline`, `langchain_huggingface`, `HuggingFaceEmbeddings`, `FAISS`, `langchain_community.vectorstores`
* **Code Snippet:**
```python
# Option 1: HuggingFaceEndpoint (Cloud-based)
from langchain_huggingface import HuggingFaceEndpoint
llm = HuggingFaceEndpoint(
    repo_id="meta-llama/Meta-Llama-3-8B-Instruct",
    task="text-generation",
    max_new_tokens=256,
    temperature=0.1
)

# Option 2: HuggingFacePipeline (Local)
from langchain_huggingface import HuggingFacePipeline
llm = HuggingFacePipeline.from_model_id(
    model_id="microsoft/Phi-3-mini-4k-instruct",
    task="text-generation",
    pipeline_kwargs={
        "max_new_tokens": 256,
        "temperature": 0.1,
    },
)
```

### 4.1 Building a Simple Search System
* **Key Points:**
  - Here's a cool example that creates a mini search engine. This is the foundation of every RAG system you'll build.
* **Technical Entities (Classes/Functions/APIs):** `HuggingFaceEmbeddings`, `FAISS`, `similarity_search`, `langchain_community.vectorstores`
* **Code Snippet:**
```python
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_community.vectorstores import FAISS
# Your knowledge base
documents = [
    "Python is a versatile programming language.",
    "Machine learning uses algorithms to find patterns.",
    "LangChain helps build AI applications.",
]
# Create embeddings and search
embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"
)
vector_store = FAISS.from_documents(documents, embeddings)
results = vector_store.similarity_search("programming", k=1)
```

## 5. Structured Outputs: Getting Clean, Usable Data
* **Key Points:**
  - Ever asked ChatGPT for a list and got a paragraph instead? Structured outputs solve that problem by forcing AI to return data in exactly the format you need.
  - Why this is game-changing: Reliability: You always get data in the same format; Integration: Easy to save to databases or use in other applications; Validation: Automatic error checking.
* **Technical Entities (Classes/Functions/APIs):** `BaseModel`, `Field`, `List`, `with_structured_output`, `pydantic`

### 5.1 Using Pydantic for Structure
* **Key Points:**
  - assume no more filling forms and just saying llm your details and llm then fills form by creating JSON data
* **Technical Entities (Classes/Functions/APIs):** `BaseModel`, `Field`, `with_structured_output`
* **Code Snippet:**
```python
from pydantic import BaseModel, Field
from typing import List

class StudentProfile(BaseModel):
    name: str = Field(description="Student's full name")
    age: int = Field(description="Student's age")
    skills: List[str] = Field(description="List of technical skills")
    is_active: bool = Field(description="Whether student is currently enrolled")

# Make your model return structured data
structured_llm = model.with_structured_output(StudentProfile)
response = structured_llm.invoke(
    "Create a profile for John, 20 years old, knows Python and JavaScript, currently studying"
)
```

### 5.2 JSON Output Parser for Simple Cases
* **Key Points:**
  - For simpler structures, use the JSON parser. Perfect for APIs, data processing, or any time you need clean, structured responses.
* **Technical Entities (Classes/Functions/APIs):** `JsonOutputParser`, `langchain_core.output_parsers`
* **Code Snippet:**
```python
from langchain_core.output_parsers import JsonOutputParser
parser = JsonOutputParser()
prompt = ChatPromptTemplate.from_template("""
Generate a programming joke with rating and sarcasm detection.
Return JSON: {
{"joke": "text",
 "rating": 7.5, 
 "is_sarcastic": false}
}
""")

chain = prompt | model | parser
result = chain.invoke({})
```

## 6. Tools: Giving AI Superpowers
* **Key Points:**
  - Here's where AI goes from "smart chatbot" to "actually useful assistant." Tools let your AI interact with the real world — search the web, run code, access databases, send emails, and more.

## ReAct Agents: AI That Can Think and Act
* **Key Points:**
  - ReAct (Reasoning + Acting) agents are the closest thing to having a human assistant. They can think through problems step by step and use tools when needed.
  - How ReAct Works: Instead of giving one response, ReAct agents follow this pattern: Thought: "I need to find information about X"; Action: Use search tool; Observation: "I found this information"; Thought: "Now I need to calculate Y"; Action: Use calculator tool; Final Answer: Combine everything into a response.
  - What makes this powerful: Multi-step reasoning: Can break down complex problems; Tool selection: Chooses the right tool for each step; Transparency: You can see its thinking process.
* **Technical Entities (Classes/Functions/APIs):** `create_react_agent`, `AgentExecutor`, `hub`, `langchain.agents`, `langchain.smith.langchain.com/hub`
* **Code Snippet:**
```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain import hub
# Get the standard ReAct prompt
prompt = hub.pull("hwchase17/react")
# Create the agent
agent = create_react_agent(
    llm=llm,
    tools=[search_tool, calculator_tool],
    prompt=prompt
)
# Wrap it with executor
agent_executor = AgentExecutor(
    agent=agent,
    tools=[search_tool, calculator_tool],
    verbose=True
)
# Use it
response = agent_executor.invoke({
    "input": "What's the current price of Bitcoin and how much would 0.5 BTC cost?"
})
```

## What's Next? (Part 2 Preview)
* **Key Points:**
  - I have covered the core building blocks, but the real magic happens when you start working with your own data. In Part 2, we'll dive into:
    - Document Loaders: Getting data from PDFs, websites, databases, and more
    - Text Splitters: Breaking down large documents intelligently
    - Embeddings: Converting text into numbers that AI can understand
    - Vector Stores: Storing and searching through your knowledge base
    - Retrievers: Finding the most relevant information for any question
    - Advanced RAG: Building systems that can answer questions about your specific data