---
aliases:
  - Hybrid Search & Reranking
Source 1: https://medium.com/@nadikapoudel16/advanced-rag-implementation-using-hybrid-search-reranking-with-zephyr-alpha-llm-4340b55fef22
---
# Advanced RAG Implementation using Hybrid Search and Reranking

* **Key Points:**
  - Retrieval-Augmented Generation (RAG) presents a solution to some of the limitations faced by LLMs.
  - Retrieval Augmented Generation refers to the terminology where we use external data as the context specific data and feed it to the LLM.
  - This data can be domain specific data and fetch it for LLM at inference, this reduce the likelihood of hallucinations.
  - It can also pull data from real-time overcoming the outdated information issue inherent in LLMs and reducing the need of continuos LLM re-training and saving the computational resources.
  - In this article, we will be implementing Advanced RAG system using following concepts: Hybrid Search; Reranking
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `LLMs`

## Hybrid Search
* **Key Points:**
  - Hybrid Search is basically a combination of keyword style search and a vector style search.
  - It has the advantage of doing keyword search as well as the advantage of doing a semantic lookup that we get from embeddings and a vector search.
* **Technical Entities (Classes/Functions/APIs):** `Hybrid Search`

### Keyword Search
* **Key Points:**
  - Here we use BM25 Algorithm.
  - It generates a sparse vector.
  - BM25 (Best Match 25) is an information retrieval algorithm used to rank and score the relevance of documents to a particular search query.
  - It's an extension of the TF-IDF (Term Frequency-Inverse Document Frequency) approach.
* **Technical Entities (Classes/Functions/APIs):** `BM25`, `TF-IDF`

### Semantic Search
* **Key Points:**
  - Semantic search is a search method that aims to improve the accuracy and relevance of search results by understanding the context and meaning behind a search query.
  - Unlike traditional keyword-based search, which primarily relies on matching keywords, semantic search tries to comprehend the intent and context of the user's query and the content of the documents being searched.
  - In our implementation we have used FAISS for semantic search and BM25 for keyword search to implement Hybrid Search using langchainEnsembleRetriever.
* **Technical Entities (Classes/Functions/APIs):** `FAISS`, `BM25`, `langchainEnsembleRetriever`

## Re-ranking
* **Key Points:**
  - Reranking is a technique that can be used to improve the performance of Retrieval Augmented Generation (RAG) pipelines.
  - In RAG, we perform a semantic search across many text documents, and then use a language model to generate a response.
  - To ensure fast search times at scale, we typically use vector search.
  - However, there is some information loss because we're compressing this information into a single vector.
  - Because of this information loss, we often see that the top few vector search documents will miss relevant information.
  - Reranking is one of the simplest methods for dramatically improving recall performance in RAG or any other retrieval-based pipeline.
  - It involves reordering the most relevant items at the top, even though using a better retrieval model is also an option.
  - Cohere AI Rerankers is a tool that can be used to implement reranking in a RAG pipeline.
  - It considers relevant context further down the list, enhancing vector search results.
  - Cohere breaks documents into 512 token chunks.
  - Contextual Compression is another technique that can be used to improve document retrieval.
  - Instead of immediately returning retrieved documents as-is, we can compress them using the context of the given query so that only the relevant information is returned.
  - Compressing here refers to both compressing the contents of an individual document and filtering out documents wholesale.
  - The idea is simple: instead of immediately returning retrieved documents as-is, we can compress them using the context of the given query so that only the relevant information is returned.
* **Technical Entities (Classes/Functions/APIs):** `Reranking`, `Cohere AI Rerankers`, `Contextual Compression`

## Implementation Stack
* **Key Points:**
  - Dataset: Tech-News dataset from BBC. Here,I have used 110 text documents here but can be increased further.
  - LLM Framework: Langchain
  - RAG Techniques used: Hybrid Search and Re-ranking to retrieve document faster provided with the given context.
  - Vector Database: FAISS
  - Vector databases are particularly optimized for similarity search.
  - This means they are capable of quickly finding vectors that are closest (or most similar) to a given query vector.
  - FAISS (Facebook AI Similarity Search), developed by Facebook's AI team is specifically engineered for the efficient similarity search and clustering of dense vectors.
  - FAISS excels in handling large datasets, offering both speed and accuracy in retrieving similar items.
  - This made it an ideal choice for this task that involves searching through large volumes of high-dimensional data i.e. identifying documents relevant to a particular query.
  - LLM — Zephyr-7b-alpha
* **Technical Entities (Classes/Functions/APIs):** `Langchain`, `FAISS`, `Zephyr-7b-alpha`, `HuggingFaceH4/zephyr-7b-alpha`

## Implementation Details
* **Technical Entities (Classes/Functions/APIs):** `TextLoader`, `RecursiveCharacterTextSplitter`, `HuggingFaceInferenceAPIEmbeddings`, `FAISS`, `ContextualCompressionRetriever`, `CohereRerank`, `ChatPromptTemplate`, `StrOutputParser`, `RunnablePassthrough`, `BM25Retriever`, `EnsembleRetriever`
* **Code Snippet:**
```python
!pip install -q langchain sentence-transformers cohere
!pip install faiss-cpu
!pip install rank_bm25
```
```python
from langchain.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import HuggingFaceInferenceAPIEmbeddings
from langchain.vectorstores import FAISS
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import CohereRerank
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain.retrievers import BM25Retriever, EnsembleRetriever
```
```python
import os
from getpass import getpass

HF_token = getpass()
os.environ['HUGGINGFACEHUB_API_TOKEN'] = HF_token
```
```python
dataset_folder_path='/content/drive/MyDrive/Tech_news_dataset/dataset_news/'
documents=[]
for file in os.listdir(dataset_folder_path):
  loader=TextLoader(dataset_folder_path+file)
  documents.extend(loader.load())
documents[:3]
```
```python
text_splitter=RecursiveCharacterTextSplitter(chunk_size=512,chunk_overlap=50)
text_splits=text_splitter.split_documents(documents)
print(len(text_splits))
```
```python
embeddings=HuggingFaceInferenceAPIEmbeddings(
    api_key=HF_token,
    model_name='BAAI/bge-base-en-v1.5'
)
```
```python
vectorstore = FAISS.from_documents(text_splits, embeddings)
```
```python
retriever_vectordb = vectorstore.as_retriever(search_kwargs={"k": 5})
keyword_retriever = BM25Retriever.from_documents(text_splits)
keyword_retriever.k =  5
ensemble_retriever = EnsembleRetriever(retrievers=[retriever_vectordb,keyword_retriever],
                                       weights=[0.5, 0.5])

query="How many cafes were closed in 2004?"
docs_rel=ensemble_retriever.get_relevant_documents(query)
docs_rel
```
```python
from langchain.llms import HuggingFaceHub
model=HuggingFaceHub(repo_id='HuggingFaceH4/zephyr-7b-alpha',
                     model_kwargs={"temperature":0.5,"max_new_tokens":512,"max_length":64}
)
```
```python
Cohere_API_token = getpass()
os.environ["COHERE_API_KEY"] =Cohere_API_token

compressor = CohereRerank()
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor, base_retriever=ensemble_retriever
)
compressed_docs = compression_retriever.get_relevant_documents(query)
compressed_docs
```
```python
template = """
<|system|>>
You are an AI Assistant that follows instructions extremely well.
Please be truthful and give direct answers. Please tell 'I don't know' if user query is not in CONTEXT

CONTEXT: {context}
</s>
<|user|>
{query}
</s>
<|assistant|>
"""

prompt = ChatPromptTemplate.from_template(template)
output_parser = StrOutputParser()

chain = (
    {"context": compression_retriever, "query": RunnablePassthrough()}
    | prompt
    | model
    | output_parser
)
query="How many cafes were closed in 2004 in China?"
response = chain.invoke(query)
response
```

## Additional notes
* **Key Points:**
  - Making sure the model does not hallucinate
  - It was done with prompt engineering, giving randomness to the model with temperature and with reranking.