---
aliases:
  - Raptor
highlights: Recursive abstractive processing creates hierarchical summaries enabling retrieval across different abstraction levels
tags:
  - rag
  - raptor
Source 1: https://nextbrain.ai/blog/from-rag-to-raptor-improving-ai-retrieval-with-open-source-models/
---
# From RAG to RAPTOR: Improving AI Retrieval with Open-Source Models


## Understanding RAG (Retrieval Augmented Generation)
* **Key Points:**
  - "RAG improves AI responses by supplementing pre-trained models with user-specific data from external sources like JSON, PDFs, or GitHub repositories. This data is transformed into chunks stored in a vector store, allowing AI to retrieve relevant information, reducing hallucinations and incorrect answers."
* **Technical Entities (Classes/Functions/APIs):** `RAG (Retrieval Augmented Generation)`, `vector store`
* **Code Snippet:** None

---

## Advancing to RAG Fusion
* **Key Points:**
  - "RAG Fusion builds on RAG by incorporating multi-query generation and reciprocal ranking. Multi-query generation involves AI creating multiple questions from the user's original query, enhancing the retrieval process. Reciprocal ranking then ranks these answers, ensuring the most accurate and relevant response is provided."
* **Technical Entities (Classes/Functions/APIs):** `RAG Fusion`, `multi-query generation`, `reciprocal ranking`
* **Code Snippet:** None

---

## Introducing RAPTOR (Recursive Abstractive Processing for Tree-Organized Retrieval)
* **Key Points:**
  - "RAPTOR takes AI retrieval a step further. Unlike traditional RAG, RAPTOR organizes data in a tree structure, summarizing at each layer from the bottom up. This method captures broader context and enhances the representation of large-scale discourse, overcoming limitations of retrieving only short text chunks."
* **Technical Entities (Classes/Functions/APIs):** `RAPTOR (Recursive Abstractive Processing for Tree-Organized Retrieval)`
* **Code Snippet:** None

---

## Practical Implementation with Ollama and Open-Source Tools
* **Key Points:**
  - "Using tools available on GitHub, RAPTOR can be implemented with ease. The process involves converting data into JSON, creating embeddings, and organizing them into a tree structure stored in a vector store. Ollama provides the infrastructure to run these processes locally, ensuring privacy and security."
* **Technical Entities (Classes/Functions/APIs):** `Ollama`, `GitHub`, `JSON`, `vector store`
* **Code Snippet:** None

---

## Step-by-Step: From RAG to RAPTOR
* **Key Points:**
  - "Data Preparation: Convert data sources into JSON and load into LangChain."
  - "Text Splitting: Use recursive or semantic chunking to divide text into manageable pieces."
  - "Embeddings Creation: Generate embeddings using local models to ensure privacy."
  - "Tree Organization: Build a tree structure by summarizing data at each layer."
  - "Storage and Retrieval: Store the tree in a vector store for efficient retrieval."
* **Technical Entities (Classes/Functions/APIs):** `LangChain`
* **Code Snippet:** None

---

## Benefits of RAPTOR
* **Key Points:**
  - "RAPTOR's tree-organized approach provides several benefits: Enhanced Context: Captures larger discourse structure, improving response accuracy."
  - "Efficient Retrieval: Tree structure enables faster and more relevant information retrieval."
  - "Scalability: Suitable for handling large datasets with complex structures."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Conclusion
* **Key Points:**
  - "The evolution from RAG to RAPTOR marks a significant leap in AI retrieval capabilities. This progression addresses key challenges in AI responses, making it a valuable tool for developers and researchers."
  - "So staying informed and engaged is crucial as we navigate this exciting era. With our platform,NextBrain AI, you can harness the full power of language models to effortlessly analyze your data and gain strategic insights.Schedule your demo todayto see what AI can reveal from your data."
* **Technical Entities (Classes/Functions/APIs):** `NextBrain AI`
* **Code Snippet:** None