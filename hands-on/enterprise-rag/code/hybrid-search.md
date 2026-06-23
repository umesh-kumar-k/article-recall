---
aliases:
  - Hybrid Search
---
# Perform Hybrid Search with MongoDB and LangChain

* **Key Points:**
  - You can integrate MongoDB with LangChain to perform hybrid search.
  - In this tutorial, you complete the following steps: Set up the environment. Use MongoDB as a vector store. Create a MongoDB Vector Search and MongoDB Search index on your data. Run hybrid search queries. Pass the query results into your RAG pipeline.
* **Technical Entities (Classes/Functions/APIs):** `MongoDB`, `LangChain`

## Prerequisites
* **Key Points:**
  - To complete this tutorial, you must have the following: One of the following MongoDB cluster types: An Atlas cluster running MongoDB version 6.0.11, 7.0.2, or later. Ensure that your IP address is included in your Atlas project's access list. A local Atlas deployment created using the Atlas CLI. A MongoDB Community or Enterprise cluster with Search and Vector Search installed. A Voyage AI API key. An OpenAI API Key. You must have an OpenAI account with credits available for API requests. An environment to run interactive Python notebooks such as Colab.
* **Technical Entities (Classes/Functions/APIs):** `MongoDB Atlas`, `Voyage AI`, `OpenAI API`, `Colab`

## Set Up the Environment
* **Key Points:**
  - Set up the environment for this tutorial.
  - Create an interactive Python notebook by saving a file with the .ipynb extension.
  - This notebook allows you to run Python code snippets individually, and you'll use it to run the code in this tutorial.
* **Technical Entities (Classes/Functions/APIs):** `Python notebook`, `.ipynb`
* **Code Snippet:**
```python
pip install --quiet --upgrade langchain langchain-community langchain-core langchain-mongodb langchain-voyageai langchain-openai pymongo pypdf
```
```python
import os
os.environ["VOYAGE_API_KEY"] = "<voyage-api-key>"
os.environ["OPENAI_API_KEY"] = "<openai-api-key>"
MONGODB_URI = "<connection-string>"
```

## Use MongoDB as a Vector Store
* **Key Points:**
  - You must use MongoDB as a vector store for your data.
  - You can instantiate a vector store by using an existing collection in MongoDB.
* **Technical Entities (Classes/Functions/APIs):** `MongoDBAtlasVectorSearch`, `VoyageAIEmbeddings`, `from_connection_string()`, `sample_mflix.embedded_movies`, `voyage-3-large`, `dotProduct`
* **Code Snippet:**
```python
from langchain_mongodb import MongoDBAtlasVectorSearch
from langchain_voyageai import VoyageAIEmbeddings
# Create the vector store
vector_store = MongoDBAtlasVectorSearch.from_connection_string(
   connection_string = MONGODB_URI,
   embedding = VoyageAIEmbeddings(model = "voyage-3-large", output_dimension = 2048),
   namespace = "sample_mflix.embedded_movies",
   text_key = "plot",
   embedding_key = "plot_embedding_voyage_3_large",
   relevance_score_fn = "dotProduct"
)
```

## Create the Indexes
* **Key Points:**
  - To enable hybrid search queries on your vector store, create a MongoDB Vector Search and MongoDB Search index on the collection.
  - You can create the indexes by using either the LangChain helper methods or the PyMongo Driver method:
  - The indexes should take about one minute to build.
  - While they build, the indexes are in an initial sync state.
  - When they finish building, you can start querying the data in your collection.
* **Technical Entities (Classes/Functions/APIs):** `MongoDB Vector Search index`, `MongoDB Search index`, `create_vector_search_index()`, `create_fulltext_search_index()`, `PyMongo`, `MongoClient`
* **Code Snippet:**
```python
# Use helper method to create the vector search index
vector_store.create_vector_search_index(
   dimensions = 2048 # The dimensions of the vector embeddings to be indexed
)
```
```python
from langchain_mongodb.index import create_fulltext_search_index
from pymongo import MongoClient
# Connect to your cluster
client = MongoClient(MONGODB_URI)
# Use helper method to create the search index
create_fulltext_search_index(
   collection = client["sample_mflix"]["embedded_movies"],
   field = "plot",
   index_name = "search_index"
)
```

## Run a Hybrid Search Query
* **Key Points:**
  - Once MongoDB builds your indexes, you can run hybrid search queries on your data.
  - The following code uses the MongoDBAtlasHybridSearchRetriever retriever to perform a hybrid search for the string "time travel".
  - The retriever returns a list of documents sorted by the sum of the full-text search score and the vector search score.
  - The final output of the code example includes the title, plot, and the different scores for each document.
* **Technical Entities (Classes/Functions/APIs):** `MongoDBAtlasHybridSearchRetriever`, `vectorstore`, `search_index_name`, `top_k`, `fulltext_penalty`, `vector_penalty`, `post_filter`, `invoke()`
* **Code Snippet:**
```python
from langchain_mongodb.retrievers.hybrid_search import MongoDBAtlasHybridSearchRetriever
# Initialize the retriever
retriever = MongoDBAtlasHybridSearchRetriever(
    vectorstore = vector_store,
    search_index_name = "search_index",
    top_k = 5,
    fulltext_penalty = 50,
    vector_penalty = 50,
    post_filter=[
        {
            "$project": {
                "plot_embedding": 0,
                "plot_embedding_voyage_3_large": 0
            }
        }
    ])
# Define your query
query = "time travel"
# Print results
documents = retriever.invoke(query)
for doc in documents:
   print("Title: " + doc.metadata["title"])
   print("Plot: " + doc.page_content)
   print("Search score: {}".format(doc.metadata["fulltext_score"]))
   print("Vector Search score: {}".format(doc.metadata["vector_score"]))
   print("Total score: {}\n".format(doc.metadata["fulltext_score"] + doc.metadata["vector_score"]))
```

## Pass Results to a RAG Pipeline
* **Key Points:**
  - You can pass your hybrid search results into your RAG pipeline to generate responses on the retrieved documents.
  - The sample code does the following: Defines a LangChain prompt template to instruct the LLM to use the retrieved documents as context for your query. LangChain passes these documents to the {context} input variable and your query to the {query} variable. Constructs a chain that specifies the following: The hybrid search retriever you defined to retrieve relevant documents. The prompt template that you defined. An LLM from OpenAI to generate a context-aware response. By default, this is the gpt-3.5-turbo model. Prompts the chain with a sample query and returns the response.
* **Technical Entities (Classes/Functions/APIs):** `StrOutputParser`, `PromptTemplate`, `RunnablePassthrough`, `ChatOpenAI`, `gpt-3.5-turbo`, `chain`, `invoke()`
* **Code Snippet:**
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import  RunnablePassthrough
from langchain_openai import ChatOpenAI
# Define a prompt template
template = """
   Use the following pieces of context to answer the question at the end.
   {context}
   Question: Can you recommend some movies about {query}?
"""
prompt = PromptTemplate.from_template(template)
model = ChatOpenAI()
# Construct a chain to answer questions on your data
chain = (
   {"context": retriever, "query": RunnablePassthrough()}
   | prompt
   | model
   | StrOutputParser()
)
# Prompt the chain
query = "time travel"
answer = chain.invoke(query)
print(answer)
```