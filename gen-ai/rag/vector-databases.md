---
aliases:
  - Vector Databases
tags:
  - similarity-search
  - vector-db
  - rag
highlights: |-
  Purpose built datastores (Pinecone,Weaviate,Qdrant,pgvector) optimized for similarity search using ANN algorithms (HNSW,IVF)


  Pinecone: Fully managed, auto-scaling, excellent DX; higher cost; no self-hosted option

  Weaviate: Hybrid search native, GraphQL, API,modular architecture; self-hosted or managed

  Qdrant: Rust-based, high performance, payload filtering, growing ecosystem, self-hosted or managed

  Milvis/Ziliz: Open-source, highly scalable (billion+ vectors) , CNCF projectl complex operations

  pgvector: PostgreSQL extension; leverage existing Postgres infra; simpler for moderate scale (<10M vectors)

  Azure AI Search: Managed hybrid search with integrated chunking. OCR,built-in security, Azure lock-in

  Chroma: Embedded/lightweight for development; not production-scale
Source 1: https://labelbox.com/blog/how-vector-similarity-search-works/
Source 2: https://medium.com/@sachinsoni600517/introduction-to-rag-retrieval-augmented-generation-and-vector-database-b593e8eb6a94
---
# How vector similarity search works

## How vector similarity search works
* **Key Points:**
  - "A vector search database, also known as a vector similarity search engine or vector database, is a type of database that is designed to store, retrieve, and search for vectors based on their similarity given a query."
  - "Vector search databases are frequently used in applications such as image retrieval, natural language processing, recommendation systems, and more."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Vector search differs from the previous generation of search engines because these databases represented data primarily using the text that they contain.
* **Key Points:**
  - "They performed various processing steps on that text to improve the performance of search against that text, which included steps like preprocessing, query expansion, synonym lists, etc."
  - "These approaches aimed to address the challenges of words that have similar meanings and create an accurate representation of the intended meaning in a given text."
  - "The issue at hand is how to identify and compare two texts that express the same idea, but do not match exactly."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Since deep learning has become popular, we've started to represent things using embeddings (which are vector representations of high-dimensional data), rather than using explicit text features.
* **Key Points:**
  - "The key idea behind vector search databases is to represent data items (e.g., images, documents, user profiles) as vectors in a high-dimensional space."
  - "Similarity between vectors is then measured using a distance metric, such as cosine similarity or Euclidean distance."
  - "The goal of a vector search database is to quickly find the most similar vectors to a given query vector."
* **Technical Entities (Classes/Functions/APIs):** `cosine similarity`, `Euclidean distance`
* **Code Snippet:** None

---

## Let's take a look at how vector search databases typically work:
### 1.Vector embeddings generation:
* **Key Points:**
  - "Data items are first converted into vectors using a feature extraction or embedding technique."
  - "For example, images can be represented as vectors using convolutional neural networks (CNNs), and text documents can be represented as vectors using word embeddings or sentence embeddings."
* **Technical Entities (Classes/Functions/APIs):** `CNNs (convolutional neural networks)`
* **Code Snippet:** None

---

### 2. Indexing & querying:
* **Key Points:**
  - "The vectors are then indexed in the vector search database."
  - "Indexing is the process of organizing the vectors in a way that allows for efficient similarity search."
  - "Various indexing techniques and data structures, such as k-d trees, ball trees, and approximate nearest neighbor (ANN) algorithms, can be used to speed up the search process."
  - "Given a query vector, the vector search database retrieves the most similar vectors from the indexed dataset."
  - "The query vector is typically generated using the same feature extraction or embedding technique used to create the indexed vectors."
  - "The similarity between the query vector and the indexed vectors is measured using a distance metric, and the most similar vectors are returned as the search results."
  - "The retrieved vectors are ranked based on their similarity scores, and the top-k most similar vectors are returned to the user."
* **Technical Entities (Classes/Functions/APIs):** `k-d trees`, `ball trees`, `approximate nearest neighbor (ANN)`
* **Code Snippet:** None

---

## Vector embeddings generation
* **Key Points:**
  - "The vectors used in vector search can represent various types of data, such as text, images, audio, or other data types."
  - "The process of creating embeddings for vector search depends on the type of data being represented and the specific use case."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### 1. Text data:
* **Key Points:**
  - "For text data, embeddings can be created using methods such as Word2Vec, GloVe or BERT."
  - "These methods create vector representations of words, phrases, or sentences based on the semantic and syntactic relationships between them."
  - "The embeddings are typically generated by training neural network models on large collections of text."
  - "Some popular methods for creating text embeddings include: Bag-of-words (BoW) model, Word embeddings (Word2Vec, GloVe), Pre-trained language models (BERT, GPT)"
* **Technical Entities (Classes/Functions/APIs):** `Word2Vec`, `GloVe`, `BERT`, `Bag-of-words (BoW)`, `GPT`
* **Code Snippet:** None

---

### 2. Image data:
* **Key Points:**
  - "For image data, embeddings can be created using convolutional neural networks (CNNs)."
  - "CNNs are trained on large datasets of labeled images to perform tasks such as image classification or object detection."
  - "The intermediate layers of the CNN can be used to extract feature vectors (embeddings) that represent the content of the images."
  - "These embeddings can then be used for vector search to find similar images."
  - "Pre-trained language models: Such as VGG, ResNet, Inception, and MobileNet can be used as feature extractors to create image embeddings."
  - "Autoencoders: These unsupervised models learn to compress and reconstruct images. The compressed representation (latent space) serves as the embedding."
* **Technical Entities (Classes/Functions/APIs):** `CNNs`, `VGG`, `ResNet`, `Inception`, `MobileNet`, `Autoencoders`
* **Code Snippet:** None

---

### 3. Audio data:
* **Key Points:**
  - "For audio data, embeddings can be created using methods such as spectrogram analysis or deep learning models like recurrent neural networks (RNNs) or CNNs."
  - "These models can be trained on audio data to extract meaningful features and create embeddings that capture the characteristics of the audio signals."
  - "Mel-Frequency Cepstral Coefficients (MFCCs): A widely used feature extraction technique for audio signals, which captures the spectral shape of the signal."
  - "Spectrogram-based embeddings: Convert audio signals into spectrograms and use them as input to CNNs or other models to learn embeddings."
  - "Recurrent Neural Networks (RNNs): Audio signals can be represented as sequences and processed by RNNs (e.g., LSTMs or GRUs) to learn embeddings."
* **Technical Entities (Classes/Functions/APIs):** `RNNs (recurrent neural networks)`, `CNNs`, `Mel-Frequency Cepstral Coefficients (MFCCs)`, `LSTMs`, `GRUs`
* **Code Snippet:** None

---

### 4. Multimodal embeddings
* **Key Points:**
  - "Multi-modal embeddings refer to the process of creating vector representations (embeddings) for data that comes from multiple modalities, such as text, images, audio, and video."
  - "The goal of multi-modal embeddings is to create a shared embedding space where similar items from different modalities are close to each other, regardless of the modality they originate from."
  - "This shared space enables tasks such as cross-modal retrieval, multi-modal classification, and multi-modal generation."
  - "Examples include: OpenAI CLIP and BLIP."
* **Technical Entities (Classes/Functions/APIs):** `CLIP` (OpenAI), `BLIP`
* **Code Snippet:** None

---

### 5. Other data types:
* **Key Points:**
  - "For other types of data, such as tabular data or time-series data, embeddings can be created using various machine learning techniques, including autoencoders, clustering algorithms, or other neural network architectures."
  - "The goal is to create embeddings that capture the relevant features and patterns in your data."
* **Technical Entities (Classes/Functions/APIs):** `autoencoders`
* **Code Snippet:** None

---

## Once embeddings are created for different modalities, they can be indexed using approximate nearest neighbor (ANN) algorithms like Annoy, HNSW, or Faiss, allowing efficient similarity search in high-dimensional vector spaces.
* **Key Points:**
  - "Let's take a look at how indexing and querying works by utilizing these types of algorithms."
* **Technical Entities (Classes/Functions/APIs):** `Annoy`, `HNSW`, `Faiss`
* **Code Snippet:** None

---

## How indexing and querying works
* **Key Points:**
  - "Vector search finds similar data using approximate nearing neighbor (ANN) algorithms."
  - "Compared to traditional keyword search, vector search yields more relevant results and executes faster."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Approximate Nearest Neighbors (ANN) is a class of algorithms used to find the nearest neighbors of a query point in a high-dimensional dataset.
* **Key Points:**
  - "These algorithms are called 'approximate' because they trade off a small amount of accuracy for a significant speedup compared to exact nearest neighbor search algorithms."
  - "ANN algorithms are commonly used in applications such as recommendation systems, image retrieval, natural language processing, and more."
  - "The general idea behind ANN algorithms is to preprocess the dataset to create an index or data structure that allows for efficient querying."
  - "When a query point is provided, the algorithm uses the index to quickly identify a set of candidate points that are likely to be close to the query point."
  - "This way, when querying the vector database to find the nearest neighbors of a query point, instead of computing distances between the query point and all vectors in the database, we only compute distances between the query point and the small number of candidate points around it."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## There are several popular ANN algorithms, each with its own approach to building the index and searching for nearest neighbors.
### 1. Brute force:
* **Key Points:**
  - "Whilst not technically an ANN algorithm it provides the most intuitive solution and a baseline to evaluate all other models."
  - "Also known as the exact nearest neighbor search, this is a straightforward method for finding the nearest neighbors of a query vector in a dataset."
  - "Unlike approximate methods, the brute force algorithm guarantees finding the exact nearest neighbors by exhaustively computing the distances between the query vector and every vector in the dataset."
  - "Due to its simplicity, the brute force algorithm is often used as a baseline for evaluating the performance of more sophisticated ANN algorithms."
* **Technical Entities (Classes/Functions/APIs):** `Brute force`
* **Code Snippet:** None

---

### 2. Locality-Sensitive Hashing (LSH):
* **Key Points:**
  - "LSH is based on the idea of hashing similar points to the same hash bucket."
  - "The dataset is hashed multiple times using different hash functions, each of which is designed to ensure that similar points are likely to collide."
  - "It differs from conventional hashing techniques in that hash collisions are maximized, not minimized."
  - "During the query phase, the query point is hashed using the same hash functions, and the algorithm retrieves the points in the corresponding hash buckets as candidates."
* **Technical Entities (Classes/Functions/APIs):** `Locality-Sensitive Hashing (LSH)`
* **Code Snippet:** None

---

### 3. k-d Trees:
* **Key Points:**
  - "k-d trees are binary search trees that partition the data along different dimensions at each level of the tree."
  - "During the construction of the k-d tree, the algorithm selects a dimension and a splitting value to partition the data into two subsets."
  - "The process is recursively applied to each subset until the tree is fully constructed."
  - "During the query phase, the algorithm traverses the tree to find the nearest neighbors."
  - "k-d trees work well for low-dimensional data but become less efficient as the dimensionality increases due to the curse of dimensionality."
* **Technical Entities (Classes/Functions/APIs):** `k-d Trees`
* **Code Snippet:** None

---

### 4. Annoy (Approximate Nearest Neighbors Oh Yeah):
* **Key Points:**
  - "Annoy is an open-source library by Spotify that builds a forest of binary search trees for approximate nearest neighbor search."
  - "Each tree is constructed by recursively splitting the data using random hyperplanes."
  - "During the query phase, the algorithm traverses multiple trees to find the nearest neighbors."
  - "It has the ability to use static files as indexes."
  - "In particular, this means one can share indexes across processes."
  - "Annoy also decouples creating indexes from loading them, so one can pass around indexes as files and map them into memory quickly."
  - "Annoy is designed to work efficiently with high-dimensional data and is currently used by Spotify for their music recommendation engine."
* **Technical Entities (Classes/Functions/APIs):** `Annoy (Approximate Nearest Neighbors Oh Yeah)` (Spotify)
* **Code Snippet:** None

---

### 5. Hierarchical Navigable Small World (HNSW) Graphs:
* **Key Points:**
  - "The Hierarchical Navigable Small World (HNSW) algorithm is an approximate nearest neighbor search method used for vector search in high-dimensional spaces."
  - "It constructs a hierarchical graph where each node represents a data point, and edges connect nearby points."
  - "The graph has multiple layers, with each layer representing a different level of granularity."
  - "The algorithm allows for efficient nearest neighbor searches by navigating the graph's layers."
  - "HNSW is known for its high search speed and accuracy."
* **Technical Entities (Classes/Functions/APIs):** `Hierarchical Navigable Small World (HNSW)`
* **Code Snippet:** None

---

### 6. ScaNN (Scalable Nearest Neighbors)
* **Key Points:**
  - "The ScaNN (Scalable Nearest Neighbors) algorithm is an approximate nearest neighbor search method developed by Google Research."
  - "It is designed to efficiently search for nearest neighbors in large-scale, high-dimensional datasets."
  - "ScaNN achieves high search accuracy and speed by combining several techniques, including quantization, vector decomposition, and graph-based search."
* **Technical Entities (Classes/Functions/APIs):** `ScaNN (Scalable Nearest Neighbors)` (Google Research)
* **Code Snippet:** None

---

### 7. Hybrid, as the name suggests, is some form of a combination of the above implementations.
* **Key Points:**
  - "There's many more algorithms currently being researched and developed."
  - "This field is progressing rapidly given the rise of foundation models and importance of vector search in this context."
* **Technical Entities (Classes/Functions/APIs):** `Hybrid`
* **Code Snippet:** None

---

## Using vector databases
* **Key Points:**
  - "There are a number of options when it comes to vector databases."
  - "Each of them has its unique advantages."
  - "Depending on the nature of your application, and whether you're trying to build the infrastructure from scratch, one of these may be a right option for you."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Vector database options include:
* **Key Points:**
  - "Vertex matching engine by Google"
  - "Pinecone, a fully managed vector database"
  - "Weaviate, an open-source vector search engine"
  - "Redis as a vector database"
  - "Qdrant, a vector search engine"
  - "Milvus, a vector database built for scalable similarity search"
  - "Chroma, an open-source embeddings store"
  - "Elastic, an open source and hosted database"
* **Technical Entities (Classes/Functions/APIs):** `Vertex matching engine` (Google), `Pinecone`, `Weaviate`, `Redis`, `Qdrant`, `Milvus`, `Chroma`, `Elastic`
* **Code Snippet:** None

---

## Utilizing Labelbox Catalog for vector search applications
* **Key Points:**
  - "Labelbox Catalog is designed with vector search capabilities to help you better organize, enrich and make useful applications with unstructured data."
  - "With Catalog, teams can easily upload text snippets, conversations or PDF documents within the UI or via the Python SDK."
  - "Labelbox will automatically generate embeddings with the uploaded data and allow you to start capitalizing on the power of vector search."
* **Technical Entities (Classes/Functions/APIs):** `Labelbox Catalog`, `Python SDK`
* **Code Snippet:** None

---

## Natural language search
* **Key Points:**
  - "Natural language search is powered by vector embeddings."
  - "A vector embedding is a numerical representation of a piece of data (e.g., an image, text, document, or video) that translates the raw data into a lower-dimensional space."
  - "Recent advances in the machine learning field enable some neural networks (e.g., CLIP by OpenAI) to recognize a wide variety of visual concepts in images and associate them with keywords."
  - "You can now surface images in Catalog by describing them in natural language."
* **Technical Entities (Classes/Functions/APIs):** `CLIP` (OpenAI), `Catalog`
* **Code Snippet:** None

---

## Similarity search
* **Key Points:**
  - "Labelbox's similarity search tool is designed to help you programmatically identify all of your data that is similar or dissimilar."
  - "You can utilize this tool to mine data and look for examples of rare assets or edge cases that will dramatically improve your model's performance."
  - "This similarity search engine also gives your team an advantage by helping you surface high-impact data rows in an ocean of data."
* **Technical Entities (Classes/Functions/APIs):** `Labelbox`
* **Code Snippet:** None

---

## Similarity search on text data using embeddings:
* **Key Points:**
  - (No text provided; image reference present)
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Similarity search on image data using embeddings:
* **Key Points:**
  - (No text provided; image reference present)
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None



--- 
## Limitations of Large Language Models Before RAG :
* **Key Points:**
  - "Outdated Knowledge: LLMs can't access new information after they're trained. They rely only on the data they were trained with, so they can't provide real-time or up-to-date information."
  - "Factual Mistakes: LLMs can generate fluent text but sometimes give incorrect or misleading answers, especially on less common or specialized topics."
  - "Hallucination Problem: LLMs sometimes 'hallucinate,' meaning they confidently generate information that sounds reasonable but is completely false. This happens because they rely only on patterns from their training data rather than real-time information."
  - "No Access to External Information: LLMs can't look up answers from external sources, like the internet or a database, which limits their ability to provide specific or accurate details on certain topics."
* **Technical Entities (Classes/Functions/APIs):** `LLMs (Large Language Models)`, `GPT-3`
* **Code Snippet:** None

---

## How RAG Solves These Issues
* **Key Points:**
  - "Retrieval-Augmented Generation (RAG) helps solve many of these problems by allowing LLMs to fetch relevant information from external sources. Instead of relying solely on their internal knowledge, RAG models can search databases or documents for real-time, up-to-date information, which helps reduce hallucinations and improves the accuracy of generated content. By integrating retrieval, RAG ensures that the model generates more reliable, fact-based answers, even for specialized or complex queries."
* **Technical Entities (Classes/Functions/APIs):** `RAG (Retrieval-Augmented Generation)`, `LLMs`
* **Code Snippet:** None

---

## Understanding Information Retrieval Before RAG
* **Key Points:**
  - "Before diving into Retrieval-Augmented Generation (RAG), it's crucial to understand Information Retrieval (IR), which plays a foundational role. As the name suggests, information retrieval is about finding and extracting relevant data from large datasets. Think of it like this: when we were kids, we used to answer questions based on a given paragraph. We didn't use the entire paragraph; instead, we picked the exact information needed to answer the question. This is the essence of IR — retrieving only the relevant information from massive collections, which could be text, images, audio, or even video."
  - "The process of IR generally consists of a few key steps, the first of which is indexing. Indexing involves converting external data sources into numerical representations, making it easier to search through large datasets. For example, if you're looking for information on the 'ICC Cricket World Cup 2023,' IR systems will scan through the database and rank all related documents, prioritizing the ones that contain the most relevant information."
* **Technical Entities (Classes/Functions/APIs):** `IR (Information Retrieval)`, `indexing`
* **Code Snippet:** None

---

## Workflow of RAG :
* **Key Points:**
  - "In Retrieval-Augmented Generation (RAG), the workflow revolves around three main components: Retrieve, Augment, and Generate."
* **Technical Entities (Classes/Functions/APIs):** `RAG`
* **Code Snippet:** None

---

## 1. Retrieve
* **Key Points:**
  - "This phase is responsible for fetching relevant information from an external knowledge base, database, or document repository. The process begins with a query, usually derived from the user's input or a given prompt."
  - "Embedding Model: The input query is first converted into vector embeddings using an embedding model. This model maps the input into a numerical form that can be used for similarity searches."
  - "Vector Database: Once the query is embedded, it is sent to a Vector DB, which contains embeddings of documents, text data, or any relevant external information. This database is indexed based on vector similarity (cosine similarity is often used)."
  - "Retriever & Ranker: A retriever component then selects the top N documents or relevant data points based on similarity. These documents are ranked in order of relevance, typically using semantic search or other retrieval algorithms like sparse or dense retrieval methods."
* **Technical Entities (Classes/Functions/APIs):** `Embedding Model`, `Vector Database`, `cosine similarity`, `Retriever`, `Ranker`, `semantic search`, `sparse retrieval`, `dense retrieval`
* **Code Snippet:** None

---

## 2. Augment
* **Key Points:**
  - "In this phase, the retrieved information is used to provide additional context to the query or prompt, enhancing the model's understanding of the task."
  - "Retrieved Context: The top N documents fetched in the retrieval stage are passed back to the model as retrieved context. This information is appended or 'augmented' with the original user query to provide additional details and improve the relevance and accuracy of the response."
  - "The goal here is to leverage both the external knowledge base and the model's trained knowledge to handle specific or unseen questions better."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## 3. Generate
* **Key Points:**
  - "The final stage is responsible for generating the actual output, which combines the original prompt/query with the augmented data from the retrieval phase."
  - "LLMs (Large Language Models): The augmented prompt, along with the retrieved context, is passed to the LLMs (e.g., GPT, BERT, or any transformer-based model). The LLMs processes the input and generates a response that is more context-aware and accurate, thanks to the extra information it received from the retrieval phase."
  - "Formatted Response: The output is returned as a formatted response, usually displayed in the user interface. The user query is enriched with additional, often domain-specific or up-to-date, information, addressing some of the inherent limitations of standard language models."
* **Technical Entities (Classes/Functions/APIs):** `LLMs (Large Language Models)`, `GPT`, `BERT`, `transformer-based model`
* **Code Snippet:** None

---

## Visualizing the RAG Workflow:
### Explanation of Above diagram workflow :
#### A. Left Side (Retrieval Methods)
* **Key Points:**
  - "Private or Custom Data: This represents a large corpus of documents or information that may not be part of the trained model (such as private databases, custom datasets, or any source of external knowledge)."
  - "2. Embedding Model: An embedding model (like BERT, or Sentence Transformers) is used to convert the documents and the user's query into dense vector representations (embeddings). These vectors represent the semantic meaning of the documents and queries in high-dimensional space."
  - "3. Vector DB: The embeddings are stored in a Vector Database, which allows for efficient similarity search. When a query is issued, the database retrieves the documents based on their vector similarity to the query."
  - "4. Retriever and Ranker: Retriever: When a query is provided, the retriever pulls the top N most relevant documents (based on vector similarity). Ranker: The ranker ranks these documents based on relevance (using similarity measures like cosine similarity)."
  - ". Retrieved Context: The top N most relevant documents or pieces of text are selected as the retrieved context to be used by the generative model in the next step."
* **Technical Entities (Classes/Functions/APIs):** `BERT`, `Sentence Transformers`, `Vector Database`, `Retriever`, `Ranker`, `cosine similarity`
* **Code Snippet:** None

---

#### B. Right Side (Generative Approach)
* **Key Points:**
  - "External Data Source: This highlights the limitation of purely generative models (such as LLMs) that do not have access to new or external data beyond their training corpus. They may: Not be up-to-date."
  - "Suffer from hallucinations."
  - "Lack specific domain knowledge."
  - "2. Embedding Model: This step represents how the query and retrieved documents are encoded into embeddings to be processed by the generative model (e.g., LLMs)."
  - "3. LLMs (Large Language Model): The LLMs (like GPT, BART, T5, etc.) takes the retrieved context (from the left side of the diagram) along with the original query and generates a response. This helps to improve the factual accuracy of the output by integrating external, relevant documents."
  - "4. Formatted Response (User Interface): The user receives the final output, which is more informed and factually correct, as it integrates both generative capabilities and retrieved information."
  - "So far, we have covered the basics of RAG. Now, let's delve into the concept of Vector Databases."
* **Technical Entities (Classes/Functions/APIs):** `LLMs`, `GPT`, `BART`, `T5`, `Vector Databases`
* **Code Snippet:** None

---

## Understanding Vector Databases :
* **Key Points:**
  - "In today's digital landscape, when you perform a search on Google, such as 'calories in apple' versus 'employees in Apple,' the search engine cleverly distinguishes between the fruit and the company. But have you ever wondered how Google achieves this? The answer lies in a technique known as semantic search."
  - "Semantic search moves beyond simple keyword matching to understand the intent behind a user's query and leverage context for more accurate results. At its core, semantic search relies on the concept of embeddings — numerical representations of text."
* **Technical Entities (Classes/Functions/APIs):** `semantic search`, `embeddings`, `Vector Databases`
* **Code Snippet:** None

---

## What Are Embeddings ?
* **Key Points:**
  - "Embeddings transform words or sentences into numeric vectors. For instance, consider the word 'Apple.' In one context, it could refer to the fruit, while in another, it might denote the tech company. To represent this, we create a vector that encodes features related to each context. For 'Apple' the fruit, the vector might reflect attributes like 'fruit,' 'sweet,' and 'edible.' For 'Apple' the company, the vector would focus on 'technology,' 'company,' and 'innovation.'"
  - "These vectors allow us to capture the semantic similarity between words. For example, the vectors for 'Apple' and 'orange' might show similarities in their fruit-related attributes, while vectors for 'Apple' and 'Samsung' would highlight their similarities in the tech context."
* **Technical Entities (Classes/Functions/APIs):** `embeddings`
* **Code Snippet:** None

---

## The Role of Vector Databases :
* **Key Points:**
  - "With thousands or even millions of embeddings to manage, storing and searching these vectors efficiently becomes crucial."
  - "Traditional relational databases might initially seem like a viable option. You'd generate embeddings, store them in a SQL database, and then compare new query embeddings to retrieve relevant results. However, this approach struggles with scalability and efficiency when dealing with vast amounts of data."
  - "To address these challenges, vector databases come into play. They are optimized for storing and querying large-scale vector embeddings. Instead of linear search, which is computationally expensive and slow for large datasets, vector databases use techniques like indexing and locality-sensitive hashing (LSH) to speed up searches."
* **Technical Entities (Classes/Functions/APIs):** `Vector databases`, `SQL database`, `indexing`, `locality-sensitive hashing (LSH)`
* **Code Snippet:** None

---

## Locality Sensitive Hashing (LSH) :
* **Key Points:**
  - "LSH is a technique that partitions vectors into 'buckets' based on their similarity. When a search query is performed, it is hashed into one of these buckets, significantly reducing the number of comparisons needed. Instead of comparing the query vector with every stored vector, you only need to compare it with those in the same bucket. This method accelerates search times and enhances efficiency."
* **Technical Entities (Classes/Functions/APIs):** `Locality Sensitive Hashing (LSH)`
* **Code Snippet:** None

---

## Why Vector Databases?
* **Key Points:**
  - "Vector databases offer two primary benefits: Faster Searches: By employing techniques like LSH, they can rapidly locate relevant vectors."
  - "Optimal Storage: They are designed to handle the unique requirements of vector data, ensuring efficient storage and retrieval."
  - "As vector databases continue to evolve, they are becoming increasingly essential for applications involving semantic search, recommendation systems, and more."
  - "I hope this overview helps you understand the fundamentals of vector databases and their significance in modern data retrieval and search technologies."
* **Technical Entities (Classes/Functions/APIs):** `Vector databases`, `LSH`, `semantic search`, `recommendation systems`
* **Code Snippet:** None