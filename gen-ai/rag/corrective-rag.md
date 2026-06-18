---
aliases:
  - Corrective RAG
highlights: Self Correcting mechanism that evaluates retrieval quality and triggers web search fallback if internal knowledge insufficient
tags:
  - rag
  - corrective-rag
Source 1: https://www.datacamp.com/tutorial/corrective-rag-crag
---
# Corrective RAG (CRAG) Implementation With LangGraph

## What Is Corrective RAG (CRAG)?
* **Key Points:**
  - "Corrective retrieval-augmented generation (CRAG) is an improved version of RAG that aims to make language models more accurate."
  - "While traditional RAG simply uses retrieved documents to help generate text, CRAG takes it a step further by actively checking and refining these documents to ensure they are relevant and accurate. This helps reduce errors or hallucinations where the model might produce incorrect or misleading information."
  - "The CRAG framework operates through a few key steps, which involve a retrieval evaluator and specific corrective actions."
  - "For any given input query, a standard retriever first pulls a set of documents from a knowledge base. These documents are then reviewed by a retrieval evaluator to determine each document's relevance to the query."
  - "In CRAG, the retrieval evaluator is a fine-tuned T5-large model. The evaluator assigns a confidence score to each document, categorizing them into three levels of confidence: Correct: If at least one document scores above the upper threshold, it is considered correct. The system then applies a knowledge refinement process, using a decompose-then-recompose algorithm to extract the most important and relevant knowledge strips while filtering out any irrelevant or noisy data within the documents. This ensures that only the most accurate and relevant information is retained for the generation process."
  - "Incorrect: If all documents fall below a lower threshold, they are marked as incorrect. In this case, CRAG discards all the retrieved documents and instead performs a web search to gather new, potentially more accurate external knowledge. This step extends the retrieval process beyond static or limited knowledge base by leveraging the vast and dynamic information available on the web, increasing the likelihood of retrieving relevant and accurate data."
  - "Ambiguous: When the retrieved documents contain mixed results, it will be considered ambiguous. In this case, CRAG combines both strategies: it refines information from the initially retrieved documents and incorporates additional knowledge obtained from web searches."
  - "After one of these actions is taken, the refined knowledge is used to generate the final response."
* **Technical Entities (Classes/Functions/APIs):** `CRAG (Corrective Retrieval-Augmented Generation)`, `T5-large`, `retrieval evaluator`, `decompose-then-recompose algorithm`, `knowledge strips`
* **Code Snippet:** None

---

## CRAG vs. traditional RAG
* **Key Points:**
  - "CRAG makes several key improvements over traditional RAG. One of its biggest advantages is its ability to fix errors in the information it retrieves. The retrieval evaluator in CRAG helps spot when information is wrong or irrelevant, so it can be corrected before it affects the final output. This means CRAG provides more accurate and reliable information, cutting down on errors and misinformation."
  - "CRAG also excels in making sure the information is both relevant and accurate. While traditional RAG might only check relevance scores, CRAG goes further by refining the documents to ensure they are not just relevant but also precise. It filters out irrelevant details and focuses on the most important points, so the generated text is based on accurate information."
* **Technical Entities (Classes/Functions/APIs):** `CRAG`, `RAG`
* **Code Snippet:** None

---

## CRAG Implementation Using LangGraph
### Step 1: Setup and installation
* **Key Points:**
  - "First, install the required packages. This step sets up the environment to run the CRAG pipeline."
  - "Next, configure your API keys for Tavily and OpenAI:"
* **Technical Entities (Classes/Functions/APIs):** `langchain_community`, `tiktoken`, `langchain-openai`, `langchainhub`, `chromadb`, `langchain`, `langgraph`, `tavily-python`, `Tavily`, `OpenAI`
* **Code Snippet:**
```bash
pip install langchain_community tiktoken langchain-openai langchainhub chromadb langchain langgraph tavily-python
```
```python
import os
os.environ["TAVILY_API_KEY"] = "YOUR_TAVILY_API_KEY"
os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"
```

---

### Step 2: Set up a proxy knowledge base
* **Key Points:**
  - "To perform RAG, we first need a knowledge base filled with documents. In this step, we'll scrape some sample documents from a Substack newsletter to create a vector store, which acts as our proxy knowledge base. This vector store helps us find relevant documents based on user queries."
  - "We start by loading documents from the provided URLs and splitting them into smaller sections using a text splitter. These sections are then embedded using OpenAIEmbeddings and stored in a vector database (Chroma) for efficient document retrieval."
* **Technical Entities (Classes/Functions/APIs):** `RecursiveCharacterTextSplitter`, `WebBaseLoader`, `Chroma`, `OpenAIEmbeddings`, `retriever`
* **Code Snippet:**
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import WebBaseLoader
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings
urls = [
    "<https://ryanocm.substack.com/p/mystery-gift-box-049-law-1-fill-your>",
    "<https://ryanocm.substack.com/p/105-the-bagel-method-in-relationships>",
    "<https://ryanocm.substack.com/p/098-i-have-read-100-productivity>",
]
docs = [WebBaseLoader(url).load() for url in urls]
docs_list = [item for sublist in docs for item in sublist]
text_splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    chunk_size=250, chunk_overlap=0
)
doc_splits = text_splitter.split_documents(docs_list)
# Add to vectorDB
vectorstore = Chroma.from_documents(
    documents=doc_splits,
    collection_name="rag-chroma",
    embedding=OpenAIEmbeddings(),
)
retriever = vectorstore.as_retriever()
```

---

### Step 3: Set up a RAG chain
* **Key Points:**
  - "In this step, we set up a basic RAG chain that takes a user's question and a set of documents to generate an answer."
  - "The RAG chain uses a predefined prompt and a language model (GPT 4-o mini) to create responses based on the retrieved documents. An output parser then formats the generated text to make it easier to read."
* **Technical Entities (Classes/Functions/APIs):** `hub` (langchain), `StrOutputParser`, `ChatOpenAI`, `rag_prompt`, `rag_llm`, `rag_chain`, `format_docs()`
* **Code Snippet:**
```python
### Generate
from langchain import hub
from langchain_core.output_parsers import StrOutputParser
# Prompt
rag_prompt = hub.pull("rlm/rag-prompt")
# LLM
rag_llm = ChatOpenAI(model_name="gpt-4o-mini", temperature=0)
# Post-processing
def format_docs(docs):
    return "\\n\\n".join(doc.page_content for doc in docs)
# Chain
rag_chain = rag_prompt | rag_llm | StrOutputParser()
print(rag_prompt.messages[0].prompt.template)
```

---

### Step 4: Set up a retrieval evaluator
* **Key Points:**
  - "To improve the accuracy of the generated content, we set up a retrieval evaluator. This tool checks how relevant each retrieved document is to make sure only the most useful information is used."
  - "The retrieval evaluator is configured with a prompt and a language model. It determines whether documents are relevant or not, filtering out any irrelevant content before a response is generated."
* **Technical Entities (Classes/Functions/APIs):** `ChatPromptTemplate`, `BaseModel`, `Field` (pydantic_v1), `ChatOpenAI`, `RetrievalEvaluator`, `retrieval_evaluator_llm`, `structured_llm_evaluator`, `retrieval_grader`
* **Code Snippet:**
```python
### Retrieval Evaluator
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI
# Data model
class RetrievalEvaluator(BaseModel):
    """Classify retrieved documents based on how relevant it is to the user's question."""
    binary_score: str = Field(
        description="Documents are relevant to the question, 'yes' or 'no'"
    )
# LLM with function call
retrieval_evaluator_llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
structured_llm_evaluator = retrieval_evaluator_llm.with_structured_output(RetrievalEvaluator)
# Prompt
system = """You are a document retrieval evaluator that's responsible for checking the relevancy of a retrieved document to the user's question. \\n 
    If the document contains keyword(s) or semantic meaning related to the question, grade it as relevant. \\n
    Output a binary score 'yes' or 'no' to indicate whether the document is relevant to the question."""
retrieval_evaluator_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        ("human", "Retrieved document: \\n\\n {document} \\n\\n User question: {question}"),
    ]
)
retrieval_grader = retrieval_evaluator_prompt | structured_llm_evaluator
```

---

### Step 5: Set up a question rewriter
* **Key Points:**
  - "We'll add a question rewriter to make user queries clearer and more specific, which helps improve the search process."
  - "The rewriter refines the original query to make the search more focused, leading to better and more relevant results."
* **Technical Entities (Classes/Functions/APIs):** `ChatOpenAI`, `ChatPromptTemplate`, `StrOutputParser`, `question_rewriter`, `question_rewriter_llm`
* **Code Snippet:**
```python
### Question Re-writer
# LLM
question_rewriter_llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
# Prompt
system = """You are a question re-writer that converts an input question to a better version that is optimized \\n 
     for web search. Look at the input and try to reason about the underlying semantic intent / meaning."""
re_write_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        (
            "human",
            "Here is the initial question: \\n\\n {question} \\n Formulate an improved question.",
        ),
    ]
)
question_rewriter = re_write_prompt | question_rewriter_llm | StrOutputParser()
```

---

### Step 6: Initialize Tavily web search tool
* **Key Points:**
  - "If the knowledge base doesn't have enough information, CRAG turns to web search to fill in the gaps. This broadens the range of possible information sources. In this step, we use the Tavily API to search the web and find additional documents."
* **Technical Entities (Classes/Functions/APIs):** `TavilySearchResults` (langchain_community.tools.tavily_search), `web_search_tool`
* **Code Snippet:**
```python
### Search
from langchain_community.tools.tavily_search import TavilySearchResults
web_search_tool = TavilySearchResults(k=3)
```

---

### Step 7: Set up LangGraph workflow
* **Key Points:**
  - "To build the CRAG workflow with LangGraph, follow these three main steps: Define the graph state, Define function nodes, Connect all function nodes"
* **Technical Entities (Classes/Functions/APIs):** `TypedDict`, `GraphState`, `Document` (langchain.schema), `StateGraph`, `END`, `START`
* **Code Snippet:**
```python
from typing import List
from typing_extensions import TypedDict
class GraphState(TypedDict):
    """
    Represents the state of our graph.
    Attributes:
        question: question
        generation: LLM generation
        web_search: whether to add search
        documents: list of documents
    """
    question: str
    generation: str
    web_search: str
    documents: List[str]
```

---

### Define function nodes
* **Key Points:**
  - "In the LangGraph workflow, each function node handles a specific task in the CRAG pipeline, such as retrieving documents, generating answers, evaluating relevance, transforming queries, and searching the web."
  - "The retrieve function finds documents from the knowledge base that are relevant to the user's question. It uses a retriever object, which is usually a vector store created from pre-processed documents. This function takes the current state, including the user's question, and uses the retriever to get relevant documents. It then adds these documents to the state."
  - "The generate function creates a response to the user's question using the retrieved documents. It works with the RAG chain, which combines a prompt with a language model. This function takes the retrieved documents and the user's question, processes them through the RAG chain, and then adds the answer to the state."
  - "The evaluate_documents function checks how relevant each retrieved document is to the user's question using the retrieval evaluator. This helps make sure only useful information is used for the final answer. This function rates each document's relevance and filters out those that aren't useful. It also updates the state with a flag web_search to show if a web search is needed when most documents aren't relevant."
  - "The transform_query function improves the user's question to get better search results, especially if the original query doesn't find relevant documents. It uses a question rewriter to make the question clearer and more specific. A better question increases the chances of finding useful documents from both the knowledge base and web searches."
  - "The web_search function looks up additional information online using the refined query. It's used when the knowledge base doesn't have enough information, helping to gather more content. This function uses the Tavily web search tool to find extra documents on the web, which are then added to the existing documents to improve the knowledge base."
  - "The decide_to_generate function decides what to do next: either generate an answer with the current documents or refine the query and search again. It makes this choice based on how relevant the documents are (as assessed earlier)."
* **Technical Entities (Classes/Functions/APIs):** `retrieve()`, `generate()`, `evaluate_documents()`, `transform_query()`, `web_search()`, `decide_to_generate()`, `retriever`, `rag_chain`, `retrieval_grader`, `question_rewriter`, `web_search_tool`
* **Code Snippet:**
```python
from langchain.schema import Document
def retrieve(state):
    """
    Retrieve documents
    Args:
        state (dict): The current graph state
    Returns:
        state (dict): New key added to state, documents, that contains retrieved documents
    """
    print("---RETRIEVE---")
    question = state["question"]
    # Retrieval
    documents = retriever.get_relevant_documents(question)
    return {"documents": documents, "question": question}
```
```python
def generate(state):
    """
    Generate answer
    Args:
        state (dict): The current graph state
    Returns:
        state (dict): New key added to state, generation, that contains LLM generation
    """
    print("---GENERATE---")
    question = state["question"]
    documents = state["documents"]
    # RAG generation
    generation = rag_chain.invoke({"context": documents, "question": question})
    return {"documents": documents, "question": question, "generation": generation}
```
```python
def evaluate_documents(state):
    """
    Determines whether the retrieved documents are relevant to the question.
    Args:
        state (dict): The current graph state
    Returns:
        state (dict): Updates documents key with only filtered relevant documents
    """
    print("---CHECK DOCUMENT RELEVANCE TO QUESTION---")
    question = state["question"]
    documents = state["documents"]
    # Score each doc
    filtered_docs = []
    web_search = "No"
    for d in documents:
        score = retrieval_grader.invoke(
            {"question": question, "document": d.page_content}
        )
        grade = score.binary_score
        if grade == "yes":
            print("---GRADE: DOCUMENT RELEVANT---")
            filtered_docs.append(d)
        else:
            print("---GRADE: DOCUMENT NOT RELEVANT---")
            continue
    if len(filtered_docs) / len(documents) <= 0.7:
        web_search = "Yes"
    return {"documents": filtered_docs, "question": question, "web_search": web_search}
```
```python
def transform_query(state):
    """
    Transform the query to produce a better question.
    Args:
        state (dict): The current graph state
    Returns:
        state (dict): Updates question key with a re-phrased question
    """
    print("---TRANSFORM QUERY---")
    question = state["question"]
    documents = state["documents"]
    # Re-write question
    better_question = question_rewriter.invoke({"question": question})
    return {"documents": documents, "question": better_question}
```
```python
def web_search(state):
    """
    Web search based on the re-phrased question.
    Args:
        state (dict): The current graph state
    Returns:
        state (dict): Updates documents key with appended web results
    """
    print("---WEB SEARCH---")
    question = state["question"]
    documents = state["documents"]
    # Web search
    docs = web_search_tool.invoke({"query": question})
    web_results = "\\n".join([d["content"] for d in docs])
    web_results = Document(page_content=web_results)
    documents.append(web_results)
    return {"documents": documents, "question": question}
```
```python
def decide_to_generate(state):
    """
    Determines whether to generate an answer, or re-generate a question.
    Args:
        state (dict): The current graph state
    Returns:
        str: Binary decision for next node to call
    """
    print("---ASSESS GRADED DOCUMENTS---")
    state["question"]
    web_search = state["web_search"]
    state["documents"]
    if web_search == "Yes":
        # All documents have been filtered check_relevance
        # We will re-generate a new query
        print(
            "---DECISION: ALL DOCUMENTS ARE NOT RELEVANT TO QUESTION, TRANSFORM QUERY---"
        )
        return "transform_query"
    else:
        # We have relevant documents, so generate answer
        print("---DECISION: GENERATE---")
        return "generate"
```

---

### Connect all function nodes
* **Key Points:**
  - "Once all the function nodes have been defined, we can now link all the function nodes together in the LangGraph workflow to build the CRAG pipeline. This means connecting the nodes with edges to manage the flow of information and decisions, making sure the workflow runs correctly based on each step's results."
* **Technical Entities (Classes/Functions/APIs):** `StateGraph`, `add_node()`, `add_edge()`, `add_conditional_edges()`, `compile()`, `app`
* **Code Snippet:**
```python
from langgraph.graph import END, StateGraph, START
workflow = StateGraph(GraphState)
# Define the nodes
workflow.add_node("retrieve", retrieve)  # retrieve
workflow.add_node("grade_documents", evaluate_documents)  # evaluate documents
workflow.add_node("generate", generate)  # generate
workflow.add_node("transform_query", transform_query)  # transform_query
workflow.add_node("web_search_node", web_search)  # web search
# Build graph
workflow.add_edge(START, "retrieve")
workflow.add_edge("retrieve", "grade_documents")
workflow.add_conditional_edges(
    "grade_documents",
    decide_to_generate,
    {
        "transform_query": "transform_query",
        "generate": "generate",
    },
)
workflow.add_edge("transform_query", "web_search_node")
workflow.add_edge("web_search_node", "generate")
workflow.add_edge("generate", END)
# Compile
app = workflow.compile()
```
```python
from IPython.display import Image, display
try:
    display(Image(app.get_graph(xray=True).draw_mermaid_png()))
except Exception:
    # This requires some extra dependencies and is optional
    pass
```

---

### Step 8: Testing the workflow
* **Key Points:**
  - "To test our setup, we run the workflow with sample queries to check how it retrieves information, evaluates document relevance, and generates responses."
  - "The first query checks how well CRAG finds answers within its knowledge base."
  - "And the second query tests CRAG's ability to search the web for additional information when the knowledge base doesn't have the relevant documents."
* **Technical Entities (Classes/Functions/APIs):** `app.stream()`, `pprint`
* **Code Snippet:**
```python
from pprint import pprint
# Run
inputs = {"question": "What's the bagel method?"}
for output in app.stream(inputs):
    for key, value in output.items():
        # Node
        pprint(f"Node '{key}':")
        # Optional: print full state at each node
        pprint(value, indent=2, width=80, depth=None)
    pprint("\\n---\\n")
# Final generation
pprint(value["generation"])
```
```python
from pprint import pprint
# Run
inputs = {"question": "What is prompt engineering?"}
for output in app.stream(inputs):
    for key, value in output.items():
        # Node
        pprint(f"Node '{key}':")
        # Optional: print full state at each node
        pprint(value, indent=2, width=80, depth=None)
    pprint("\\n---\\n")
# Final generation
pprint(value["generation"])
```

---

## The Limitations of CRAG
* **Key Points:**
  - "Even though CRAG improves on traditional RAG, it has some limitations that need attention."
  - "One major issue is its dependence on the quality of the retrieval evaluator. This evaluator is essential for judging whether the retrieved documents are relevant and accurate. However, training and fine-tuning the evaluator can be demanding, requiring a lot of high-quality data and computational power. Keeping the evaluator up-to-date with new types of queries and data sources adds to the complexity and cost."
  - "Another limitation is CRAG's use of web searches to find or replace documents that are incorrect or ambiguous. While this approach can provide more recent and diverse information, it also risks introducing biased or unreliable data. The quality of web content varies widely, and sorting through it to find the most accurate information can be challenging. This means even a well-trained evaluator can't completely prevent the inclusion of poor-quality or biased information."
  - "These challenges highlight the need for ongoing research and development."
* **Technical Entities (Classes/Functions/APIs):** `CRAG`, `RAG`, `retrieval evaluator`
* **Code Snippet:** None

---

## Conclusion
* **Key Points:**
  - "Overall, CRAG improves on traditional RAG systems by adding features that check and refine the information retrieved, making language models more accurate and reliable. This makes CRAG a useful tool for many different applications."
* **Technical Entities (Classes/Functions/APIs):** `CRAG`, `RAG`
* **Code Snippet:** None