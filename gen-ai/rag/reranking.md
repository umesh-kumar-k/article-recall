---
aliases:
  - Reranking Models
highlights: |-
  Cohere Rerank: Managed cross-encoder achieving 20 to 40 % accuracy lift;simple integration

  BGE Reranker: Open-source alternative;self-hostable; ~5-10x slower than first-stage retrieval 

  Cross-Encoder (Sentence Transformers): DIY approach; fine-tunable for domain specific ranking
tags:
  - rag
  - re-ranking
Source 1: https://machinelearningmastery.com/top-5-reranking-models-to-improve-rag-results/
Source 2: https://developer.nvidia.com/blog/enhancing-rag-pipelines-with-re-ranking/
---
# Enhancing RAG Pipelines with Re-Ranking


## What is re-ranking?
* **Key Points:**
  - "Re-ranking is a sophisticated technique used to enhance the relevance of search results by using the advanced language understanding capabilities of LLMs."
  - "Initially, a set of candidate documents or passages is retrieved using traditional information retrieval methods like BM25 or vector similarity search. These candidates are then fed into an LLM that analyzes the semantic relevance between the query and each document. The LLM assigns relevance scores, enabling the re-ordering of documents to prioritize the most pertinent ones."
  - "This process significantly improves the quality of search results by going beyond mere keyword matching to understand the context and meaning of the query and documents."
  - "Re-ranking is typically used as a second stage after an initial fast retrieval step, ensuring that only the most relevant documents are presented to the user."
  - "It can also combine results from multiple data sources, as well as integrate in a RAG pipeline to further ensure that context is ideally tuned for the specific query."
* **Technical Entities (Classes/Functions/APIs):** `BM25`, `vector similarity search`, `LLMs`, `NVIDIA NeMo Retriever reranking NIM`, `Mistral-7B`, `LoRA`
* **Code Snippet:** None

---

## Tutorial prerequisites
* **Key Points:**
  - "To make the best use of this tutorial, you need a basic knowledge of LLM inference pipelines along with the following resources: LangChain, NVIDIA AI Foundation Endpoints, Vector store"
* **Technical Entities (Classes/Functions/APIs):** `LangChain`, `NVIDIA AI Foundation Endpoints`, `Vector store`
* **Code Snippet:** None

---

## Set up
* **Key Points:**
  - "To get started, create a free account with the NVIDIA API Catalog and follow these steps: Select any model. Choose Python, Get API Key. Save the generated key as NVIDIA_API_KEY. From there, you should have access to the endpoints."
  - "Now, install LangChain, NVIDIA AI Endpoints, and FAISS: pip install langchain, pip install langchain_nvidia_ai_endpoints, pip install faiss-gpu"
* **Technical Entities (Classes/Functions/APIs):** `LangChain`, `NVIDIA AI Endpoints`, `FAISS`
* **Code Snippet:**
```bash
pip install langchain
pip install langchain_nvidia_ai_endpoints
pip install faiss-gpu
```

---

## Load relevant documents
* **Key Points:**
  - "For this example, load the recent NVIDIA publication on multi-modal LLMs, VILA: On Pre-training for Visual Language Models."
  - "Use this single PDF for all the examples in the post, but the code can be easily extended to load multiple documents."
* **Technical Entities (Classes/Functions/APIs):** `PyPDFLoader` (langchain_community.document_loaders)
* **Code Snippet:**
```python
from langchain_community.document_loaders import PyPDFLoader
 
document = PyPDFLoader("2312.07533v4.pdf").load()
```

---

## Split into chunks
* **Key Points:**
  - "Next, split the documents into separate chunks."
  - "Make sure to pay attention to the chunk_size parameter in TextSplitter. Setting the right chunk size is critical for RAG performance, as much of a RAG pipeline's success is based on the retrieval step finding the right context for generation."
  - "The retrieval step typically examines smaller chunks of the original text rather than all documents."
  - "The entire prompt (retrieved chunks plus the user query) must fit within the LLM's context window."
  - "Don't specify chunk sizes too big and balance them out with the estimated query size. Experiment with different chunk sizes, but typical values should be 100-600 tokens, depending on the LLM."
* **Technical Entities (Classes/Functions/APIs):** `RecursiveCharacterTextSplitter` (langchain_text_splitters)
* **Code Snippet:**
```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(chunk_size=800, chunk_overlap=200)
texts = text_splitter.split_documents(document)
```

---

## Generate embeddings
* **Key Points:**
  - "Next, generate embeddings using NVIDIA AI Foundation endpoints and save the embeddings to an offline vector store in the /embed directory for future re-use."
  - "For this task, use FAISS, a library for efficient similarity search and clustering of dense vectors. It contains algorithms that search in sets of vectors of any size, up to ones that possibly do not fit in RAM."
* **Technical Entities (Classes/Functions/APIs):** `NVIDIAEmbeddings` (langchain_nvidia_ai_endpoints), `FAISS` (langchain_community.vectorstores)
* **Code Snippet:**
```python
from langchain_nvidia_ai_endpoints import NVIDIAEmbeddings
from langchain_community.vectorstores import FAISS
 
embeddings = NVIDIAEmbeddings()
db = FAISS.from_documents(texts, embeddings)
```

---

## Create a basic retriever
* **Key Points:**
  - "Now create a basic retriever based on the document and search for the most relevant chunks for your query."
  - "This code outputs the 45 most relevant chunks to your query based on a simple retrieval algorithm:"
* **Technical Entities (Classes/Functions/APIs):** `retriever`, `invoke()`
* **Code Snippet:**
```python
retriever = db.as_retriever(search_kwargs={"k": 45})
 
query = "Where is the A100 GPU used?"
docs = retriever.invoke(query)
```

---

## Add a re-ranking step
* **Key Points:**
  - "Now add a re-ranking step using the NeMo Retriever reranking NIM."
  - "This is a GPU-accelerated model optimized for providing a probability score that a given passage contains the information to answer a question."
  - "This re-ranks the previously fetched chunks according to which is most relevant using the same query."
  - "You use the NIM as input to the LangChain contextual compression retriever, which improves retrieval by compressing and filtering documents based on the query context before returning them."
  - "The reranking NIM recognizes the most relevant chunk as the paragraph related to training cost at the end of the paper, which specifies the A100 GPU:"
* **Technical Entities (Classes/Functions/APIs):** `NVIDIARerank` (langchain_nvidia_ai_endpoints), `ContextualCompressionRetriever` (langchain.retrievers.contextual_compression), `compress_documents()`
* **Code Snippet:**
```python
from langchain_nvidia_ai_endpoints import NVIDIARerank
from langchain.retrievers.contextual_compression import ContextualCompressionRetriever
 
reranker = NVIDIARerank()
compression_retriever = ContextualCompressionRetriever(
    base_compressor=reranker, base_retriever=retriever
)
 
reranked_chunks = compression_retriever.compress_documents(query)
```

---

## Combining results from multiple data sources
* **Key Points:**
  - "In addition to enhancing accuracy for a single data source, you can use re-ranking to combine multiple data sources in a RAG pipeline."
  - "Consider a pipeline with data from a semantic store, such as the previous example, as well as a BM25 store. Each store is queried independently and returns results that the individual store considers to be highly relevant. Figuring out the overall relevance of the results is where re-ranking comes into play."
  - "The following code example combines the previous semantic search results with BM25 results. The results in combined_docs are ordered by their relevance to the query by the reranking NIM."
* **Technical Entities (Classes/Functions/APIs):** `BM25`, `reranker.top_n`, `compress_documents()`
* **Code Snippet:**
```python
all_docs = docs + bm25_docs
 
reranker.top_n = 5
 
combined_docs = reranker.compress_documents(query=query, documents=all_docs)
```

---

## Connect to a RAG pipeline
* **Key Points:**
  - "In addition to using re-ranking independently, you can add it to a RAG pipeline to further enhance responses by ensuring that they use the most relevant chunks for augmenting the original query."
  - "In this case, connect the compression_retriever object from the previous step to the RAG pipeline."
  - "The RAG pipeline now uses the correct top-ranked chunk and summarizes the main insights: The A100 GPU is used for training the 7B model in the supervised fine-tuning/instruction tuning ablation study."
* **Technical Entities (Classes/Functions/APIs):** `RetrievalQA` (langchain.chains), `ChatNVIDIA` (langchain_nvidia_ai_endpoints), `from_chain_type()`
* **Code Snippet:**
```python
from langchain.chains import RetrievalQA
from langchain_nvidia_ai_endpoints import ChatNVIDIA
 
chain = RetrievalQA.from_chain_type(
    llm=ChatNVIDIA(temperature=0), retriever=compression_retriever
)
result = chain({"query": query})
print(result.get("result"))
```

---

## Conclusion
* **Key Points:**
  - "RAG has emerged as a powerful approach, combining the strengths of LLMs and dense vector representations."
  - "By using dense vector representations, RAG models can scale efficiently, making them well-suited for large-scale enterprise applications, such as multilingual customer service chatbots and code generation agents."
  - "As LLMs continue to evolve, it is clear that RAG will play an increasingly important role in driving innovation and delivering high-quality, intelligent systems that can understand and generate human-like language."
  - "When building your own RAG pipeline, it's important to correctly split the vector store documents into chunks by optimizing the chunk size for your specific content and selecting an LLM with a suitable context length."
  - "In some cases, complex chains of multiple LLMs may be required."
  - "To optimize RAG performance and measure success, use a collection of robust evaluators and metrics."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

# Reranking for Better Answers

## What about Reranking?
* **Key Points:**
  - "Text chunks retrieved solely based on a retrieval metric – that is, raw retrieval– may not be that useful for several different reasons: The retrieved chunks we end up with may vary largely with the selected number of top chunks k. Depending on the number k of top chunks we retrieve, we may get very different results."
  - "We may retrieve chunks that are semantically close to what we are looking for, but still off-topic and, in reality, not appropriate to answer the user's query."
  - "We may get partial matches to specific words included in the user's query, leading to chunks that include those specific words but are in fact irrelevant."
  - "Back to my favorite question from the 'War and Peace' example, if we ask 'Who is Anna Pávlovna?', and use a very small k (like k = 2), the retrieved chunks may not contain enough information to comprehensively answer the question. Conversely, if we allow for a large number of chunks k to be retrieved (say k = 20), we are most probably going to also retrieve some irrelevant text chunks where 'Anna Pávlovna' is just mentioned, but isn't the topic of the chunk."
  - "Thus, the meaning of some of those chunks is going to be unrelated to the user's query and useless for answering it. Therefore, we need a way to distinguish the truly relevant retrieved text chunks out of all the retrieved chunks."
  - "Here, it is worth clarifying that one straightforward solution for this issue would be just retrieving everything and passing everything to the generation step (to the LLM). Unfortunately, this cannot be done for a bunch of reasons, like that the LLMs have certain context windows, or that the LLMs' performance gets worse when overstuffing with information."
  - "So, this is the issue we try to tackle by introducing the reranking step. In essence, reranking means re-evaluating the chunks that are retrieved based on the cosine similarity scores with a more accurate, yet also more expensive and slower method."
  - "There are various methods for doing this, as for instance, cross-encoders, employing an LLM to do the reranking, or using heuristics."
  - "Ultimately, by introducing this extra reranking step, we essentially implement what is called a two-stage retrieval with reranking, which is a standard industry approach. This allows for improving the relevance of the retrieved text chunks and, as a result, the quality of the generated responses."
* **Technical Entities (Classes/Functions/APIs):** `cosine similarity`, `L2 distance`, `dot product`, `cross-encoders`
* **Code Snippet:** None

---

## Reranking with a Cross-Encoder
* **Key Points:**
  - "Cross-encoders are the standard models used for reranking in a RAG framework."
  - "Unlike retriever functions used in the initial retrieval step, which just take into account the similarity scores of different text chunks, cross-encoders are able to perform a more in-depth comparison of each of the retrieved text chunks with the user's query."
  - "More specifically, a cross encoder jointly embeds a document and the user's query and produces a similarity score."
  - "On the flip side, in cosine similarity-based retrieval, the document and the user's query are embedded separately from one another, and then their similarity is calculated."
  - "As a result, some information of the original texts is lost when creating the embeddings separately, and some more information is preserved when the texts are jointly embedded. Consequently, a cross encoder can assess better the relevance between two text chunks (that is, the user's query and a document)."
  - "So why not use a cross-encoder in the first place? The answer is because cross-encoders are very slow."
  - "For instance, a cosine similarity search for about 1,000 passages takes less than a millisecond. On the contrary, using solely a cross-encoder (like ms-marco-MiniLM-L-6-v2) to search the same set of 1,000 passages and match for a single query would be orders-of-magnitude slower!"
  - "This is to be expected if you think about it, since using a cross-encoder means that we have to pair each chunk of the knowledge base with the user's query and embed them on the spot, and for each and every new query. On the contrary, with cosine similarity-based retrieval, we get to create all the embeddings of the knowledge base beforehand, and just once, and then once the user submits a query, we just need to embed the user's query and calculate the pairwise cosine similarities."
  - "For that reason, we adjust our RAG pipeline appropriately and get the best of both worlds; first, we narrow down the candidate relevant chunks with the cosine similarity search, and then, in the second step, we assess the similarity of the retrieved chunks more accurately with a cross-encoder."
* **Technical Entities (Classes/Functions/APIs):** `Cross-encoders`, `ms-marco-MiniLM-L-6-v2`, `cosine similarity`
* **Code Snippet:** None

---

## Back to the 'War and Peace' Example
* **Key Points:**
  - "My code so far looks something like this:"
* **Technical Entities (Classes/Functions/APIs):** `ChatOpenAI` (LangChain), `OpenAIEmbeddings` (LangChain), `FAISS` (LangChain), `RecursiveCharacterTextSplitter` (LangChain), `TextLoader` (LangChain), `Document` (LangChain), `faiss`, `numpy`
* **Code Snippet:**
```python
import os
from langchain.chat_models import ChatOpenAI
from langchain.document_loaders import TextLoader
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.docstore.document import Document

import faiss

api_key = "my_api_key"

# initialize LLM
llm = ChatOpenAI(openai_api_key=api_key, model="gpt-4o-mini", temperature=0.3)

# initialize embeddings model
embeddings = OpenAIEmbeddings(openai_api_key=api_key)

# loading documents to be used for RAG 
text_folder =  "RAG files"  

documents = []
for filename in os.listdir(text_folder):
    if filename.lower().endswith(".txt"):
        file_path = os.path.join(text_folder, filename)
        loader = TextLoader(file_path)
        documents.extend(loader.load())

splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=100)
split_docs = []
for doc in documents:
    chunks = splitter.split_text(doc.page_content)
    for chunk in chunks:
        split_docs.append(Document(page_content=chunk))
        
documents = split_docs

# normalize knowledge base embeddings
import numpy as np
def normalize(vectors):
    vectors = np.array(vectors)
    norms = np.linalg.norm(vectors, axis=1, keepdims=True)
    return vectors / norms

doc_texts = [doc.page_content for doc in documents]
doc_embeddings = embeddings.embed_documents(doc_texts)
doc_embeddings = normalize(doc_embeddings)

# faiss index with inner product
import faiss
dimension = doc_embeddings.shape[1]
index = faiss.IndexFlatIP(dimension)  # inner product index
index.add(doc_embeddings)

# create vector database w FAISS 
vector_store = FAISS(embedding_function=embeddings, index=index, docstore=None, index_to_docstore_id=None)
vector_store.docstore = {i: doc for i, doc in enumerate(documents)}

def main():
    print("Welcome to the RAG Assistant. Type 'exit' to quit.\n")
    
    while True:
        user_input = input("You: ").strip()
        if user_input.lower() == "exit":
            print("Exiting…")
            break

        # embedding + normalize query
        query_embedding = embeddings.embed_query(user_input)
        query_embedding = normalize([query_embedding]) 

        # search FAISS index
        D, I = index.search(query_embedding, k=2)
        
        # get relevant documents
        relevant_docs = [vector_store.docstore[i] for i in I[0]]
        retrieved_context = "\n\n".join([doc.page_content for doc in relevant_docs])
        
        # D contains inner product scores == cosine similarities (since normalized)
        print("\nTop chunks and their cosine similarity scores:\n")
        for rank, (idx, score) in enumerate(zip(I[0], D[0]), start=1):
           print(f"Chunk {rank}:")
           print(f"Cosine similarity: {score:.4f}")
           print(f"Content:\n{vector_store.docstore[idx].page_content}\n{'-'*40}")
               
        # system prompt
        system_prompt = (
            "You are a helpful assistant. "
            "Use ONLY the following knowledge base context to answer the user. "
            "If the answer is not in the context, say you don't know.\n\n"
            f"Context:\n{retrieved_context}"
        )

        # messages for LLM 
        messages = [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_input}
        ]

        # generate response
        response = llm.invoke(messages)
        assistant_message = response.content.strip()
        print(f"\nAssistant: {assistant_message}\n")

if __name__ == "__main__":
    main()
```

---

## Then we can initialise the cross-encoder and define a function for reranking the top k chunks retrieved from the vector search:
* **Key Points:**
  - "import torch from sentence_transformers import CrossEncoder"
  - "initialize cross-encoder model cross_encoder = CrossEncoder('cross-encoder/ms-marco-TinyBERT-L-2', device='cuda' if torch.cuda.is_available() else 'cpu')"
  - "define a function for reranking the top k chunks retrieved from the vector search:"
* **Technical Entities (Classes/Functions/APIs):** `CrossEncoder` (sentence_transformers), `ms-marco-TinyBERT-L-2`, `torch`
* **Code Snippet:**
```python
import torch
from sentence_transformers import CrossEncoder

# initialize cross-encoder model
cross_encoder = CrossEncoder('cross-encoder/ms-marco-TinyBERT-L-2', device='cuda' if torch.cuda.is_available() else 'cpu')

def rerank_with_cross_encoder(query, relevant_docs):
    
    pairs = [(query, doc.page_content) for doc in relevant_docs] # pairs of (query, document) for cross-encoder
    scores = cross_encoder.predict(pairs) # relevance scores from cross-encoder model
    
    ranked_indices = np.argsort(scores)[::-1] # sort documents based on cross-encoder score (the higher, the better)
    ranked_docs = [relevant_docs[i] for i in ranked_indices]
    ranked_scores = [scores[i] for i in ranked_indices]
    
    return ranked_docs, ranked_scores
```

---

## … and also adjust of function as follows:
* **Key Points:**
  - "search FAISS index D, I = index.search(query_embedding, k=6)"
  - "get relevant documents relevant_docs = [vector_store.docstore[i] for i in I[0]]"
  - "rerank with our function reranked_docs, reranked_scores = rerank_with_cross_encoder(user_input, relevant_docs)"
  - "get top reranked chunks retrieved_context = "\n\n".join([doc.page_content for doc in reranked_docs[:2]])"
  - "display top reranked chunks"
* **Technical Entities (Classes/Functions/APIs):** `FAISS`
* **Code Snippet:**
```python
        ...

        # search FAISS index
        D, I = index.search(query_embedding, k=6)
        
        # get relevant documents
        relevant_docs = [vector_store.docstore[i] for i in I[0]]
        
        # rerank with our function
        reranked_docs, reranked_scores = rerank_with_cross_encoder(user_input, relevant_docs)
        
        # get top reranked chunks
        retrieved_context = "\n\n".join([doc.page_content for doc in reranked_docs[:2]])
        
        # D contains inner product scores == cosine similarities (since normalized)
        print("\nTop 6 Retrieved Chunks:\n")
        for rank, (idx, score) in enumerate(zip(I[0], D[0]), start=1):
            print(f"Chunk {rank}:")
            print(f"Similarity: {score:.4f}")
            print(f"Content:\n{vector_store.docstore[idx].page_content}\n{'-'*40}")

        # display top reranked chunks
        print("\nTop 2 Re-ranked Chunks:\n")
        for rank, (doc, score) in enumerate(zip(reranked_docs[:2], reranked_scores[:2]), start=1):
            print(f"Rank {rank}:")
            print(f"Reranker Score: {score:.4f}") 
            print(f"Content:\n{doc.page_content}\n{'-'*40}")
               
        ...
```

---

## On my mind
* **Key Points:**
  - "So, it becomes apparent is an essential step for building a robust RAG pipeline."
  - "Fundamentally, it allows us to bridge the gap between the quick but not so precise vector search, and context-aware answers."
  - "By performing a two-step retrieval, with the vector search being the first step, and the second step being the reranking, we get the best of both worlds: efficiency at scale and higher quality responses."
  - "In practice, this two-stage approach is what makes modern RAG pipelines both practical and powerful."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None