---
aliases:
  - Hybrid Search
highlights: Combines dense vector search with sparse retrieval (BM25, keyword) using reciprocal rank fusion for better recall across diverse query types
tags:
  - rag
  - hybrid-search
  - sparse
  - sparse-vector
  - dense-vector
  - bm25
  - reciprocal-rank-fusion
Source 1: https://www.meilisearch.com/blog/hybrid-search
Source 2: https://medium.com/@devalshah1619/mathematical-intuition-behind-reciprocal-rank-fusion-rrf-explained-in-2-mins-002df0cc5e2a
Source 3: https://www.mongodb.com/resources/basics/reciprocal-rank-fusion
---

# Hybrid Search 101: how it works and why It's important

## What is hybrid search?
* **Key Points:**
  - "Hybrid search systems combine keyword-based retrieval (sparse vector methodology) with semantic search systems (dense vector embeddings) to optimize precision and contextual relevance."
  - "Semantic search relies on dense vectors, requiring both the search query and target data to be embedded using Machine Learning (ML) models."
  - "Some methods, like neural search, leverage Deep Neural Networks (DNNs) to generate rich contextual insights for embedding, retrieval, and ranking."
  - "Vector search is another type of semantic search that uses embedding models to create dense vectors, ML algorithms such as Approximate Nearest Neighbors (ANN) for information retrieval, and cosine similarity search for ranking."
  - "On the other hand, keyword search relies on sparse vectors generated through algorithms."
  - "For instance, our full-text search relies on a vector of documents associated with each word in the dataset."
  - "When a search query is entered, the system preprocesses the input by extracting individual words and matching them against the sparse vector values of the documents. These are then ranked based on keyword relevance using recursive bucket scoring."
* **Technical Entities (Classes/Functions/APIs):** `ML models`, `Deep Neural Networks (DNNs)`, `Approximate Nearest Neighbors (ANN)`, `cosine similarity`
* **Code Snippet:** None

---

## Dense vectors
* **Key Points:**
  - "The dense vectors are extensive and can have hundreds or thousands of floats (numerical representations) to identify a single document."
  - "They represent the similarity of objects in a vector space and can have the following shape: json dense_vector = [0.8, 0.4, 0.2, 0.7, 0.9, 0.1, … ]"
  - "Dense vectors are usually multidimensional and don't have zero values because they are created continuously to capture the complete information of the document or query."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```json
dense_vector = [0.8, 0.4, 0.2, 0.7, 0.9, 0.1, … ]
```

---

## Sparse vectors
* **Key Points:**
  - "In full-text search, sparse vectors come in the form of documents associated with each word in a dataset."
  - "This vector representation forms the foundation for ranking and retrieving relevant and accurate results."
  - "Hybrid search combines the strengths of dense and sparse vector retrieval to enhance search scores."
  - "It first gathers matches from both methods and then refines the final output by globally ordering the results based on relevance."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## How does hybrid search work?
* **Key Points:**
  - "Hybrid search works by leveraging the semantic capabilities of dense vectors and the exact matching and accuracy of sparse vectors."
  - "The outputs retrieved from both methods are then blended to deliver more relevant search outcomes."
  - "By looking at the schematic above, we can structure the hybrid search workflow into distinct steps, allowing both semantic and keyword searches to operate in parallel: Data cleansing & preprocessing"
  - "Keyword Search: Requires robust data cleaning (e.g., Meilisearch includes native support for stop words, though manual configuration is required for full-text seatch) to ensure accurate term matching."
  - "Semantic Search: Benefits from noise reduction and strategic text segmentation (chunking) to improve the quality of document embeddings."
  - "Embedding: dense and sparse representations: Semantic: Models like Bidirectional Encoder Representations from Transformers (BERT) and Global Vectors (GloVe) transform documents into dense vectors, capturing nuanced contextual meanings."
  - "Keywords: Some algorithms use frequency-based scoring, while others use neural networks to generate sparse embeddings."
  - "Retrieval mechanisms of dense and sparse vectors: Semantic Retrieval: Utilizes algorithms like Approximate Nearest Neighbor (ANN) and K-Nearest Neighbor (KNN) to search within the dense vector space efficiently."
  - "Keyword Retrieval: Directly matches query terms with document vectors."
  - "Ensemble retrieval: Hybrid search: The final step involves combining results from both retrieval methods to determine the most relevant ones."
  - "The hybrid search process can be tweaked to assign importance to one type of result over another."
  - "If contextual meaning outweighs lexical matching, the system prioritizes outputs from the semantic search. Otherwise, it prioritizes keyword matching. This feature is available and easily controlled in Meilisearch's hybrid search setup."
* **Technical Entities (Classes/Functions/APIs):** `BERT (Bidirectional Encoder Representations from Transformers)`, `GloVe (Global Vectors)`, `Approximate Nearest Neighbor (ANN)`, `K-Nearest Neighbor (KNN)`, `Meilisearch`
* **Code Snippet:** None

---

## What is a hybrid search engine example?
* **Key Points:**
  - "Companies have adopted hybrid search engines to improve the accuracy and relevance of search results."
  - "One of the most advanced hybrid search systems is Google Search, which combines multiple search techniques and algorithms to deliver precise and contextually relevant results."
  - "Google integrates both keyword-based search and machine learning models to interpret user queries, rank web pages, and present the most relevant information."
  - "Currently, they utilize Vertex AI Embedding models to generate dense vectors that capture semantic meaning while simultaneously creating sparse vectors for keyword-based retrieval."
  - "To configure search results, Google merges output from semantic and keyword-based searches using Reciprocal Rank Fusion (RRF), as detailed in their official notebook."
  - "As of January 2025, Google Search maintains an 89.79% market share, continuing to dominate the search landscape."
* **Technical Entities (Classes/Functions/APIs):** `Google Search`, `Vertex AI Embedding models`, `Reciprocal Rank Fusion (RRF)`
* **Code Snippet:** None

---

## What are the benefits of hybrid search?
* **Key Points:**
  - "Hybrid search provides several advantages over standalone keyword-based or semantic search methods."
  - "Enhanced search accuracy and relevance: Hybrid search provides high-quality results by combining exact matches with semantic understanding. This level of accuracy ultimately retains users and reduces the bounce rate."
  - "Improved user experience: The system can deliver meaningful content even if users enter inaccurate terms or vague keywords. This ease of retrieving information allows designers to create engaging search elements."
  - "Just ask CarbonGraph: 'We migrated to Meilisearch from Pinecone to consolidate our search service [...] The setup of the OpenAI embedder was very straightforward, and we love that embeddings are created automatically using the contents of a search document.'"
  - "Cost-effective implementation: Lexical matching in hybrid search reduces memory usage compared to pure semantic search engines. This is crucial for lowering cloud costs related to storage and computational demand, especially since keyword search algorithms do not depend on GPUs."
  - "Increased search speed: Opinly, a company that allows you to monitor your competitors' websites, adopted hybrid search technologies to enhance the quality and contextual accuracy of its search results."
  - "Personalization and adaptability: Hybrid search systems can be configured to dynamically adjust the weight of keywords and semantic relevance or provide the user with control over it."
  - "The NFSA collection has this option available in their search engine, very similar to Meilisearch's hybrid search."
  - "Hybrid search provides significant advantages across various business domains, offering speed, robustness, and efficiency."
* **Technical Entities (Classes/Functions/APIs):** `Meilisearch`, `Pinecone`, `OpenAI embedder`
* **Code Snippet:** None

---

## What are the drawbacks of hybrid search?
* **Key Points:**
  - "While hybrid search offers the best of both keyword-based and semantic search, it also comes with challenges that can impact implementation, performance, and user experience."
  - "Increased complexity in implementation: Hybrid search requires integrating multiple search algorithms (e.g., semantic search using dense embeddings). This integration can be technically complex and requires deep technical understanding."
  - "Difficulty in balancing keyword precision and context: Over-relying on one method may diminish the benefits of the other (ex., more semantic power than keyword precision). If a good balance is not achieved, this could lead to a bad user experience and increased bounce rates."
  - "Bad user experience: If users can adjust semantic weight, the interface should be intuitive or designed for an audience familiar with the terminology. Otherwise, it may lead to confusion and increase the risk of user drop-off. According to this Toptotal report, 88% of users are less likely to return after a bad user experience."
  - "Despite these challenges, hybrid search remains a powerful tool when applied correctly."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## When should you use hybrid search?
* **Key Points:**
  - "Hybrid search isn't the optimal solution in all scenarios. In cases where data is highly structured – such as product inventories or specific academic research – precision is key, and similar-sounding terms with distinct meanings must be strictly differentiated."
  - "Hybrid search truly excels in the following examples: E-commerce platforms: Online retailers like Amazon implement hybrid search to enhance product discovery. When customers input vague queries, the system leverages keyword matching and semantic analysis to present relevant products."
  - "Enterprise knowledge bases: Organizations often maintain extensive repositories of documents, manuals, and communications. Hybrid search enables employees to retrieve pertinent information efficiently and increase productivity."
  - "Streaming services: Platforms such as Netflix utilize hybrid search to help users find content, whether they search by specific titles or describe themes."
  - "Marketplaces: Hybrid search in e-commerce can improve search accuracy, handle complex queries, and increase product discovery, leading to increased sales."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## How can you implement hybrid search?
* **Key Points:**
  - "Implementing a hybrid search requires a vector store solution. Several languages and AI frameworks can be used for implementation, but Python with Langchain is often a good stack to start building efficiently."
  - "The AI-enhanced hybrid search from Meilisearch allows third-party embedding models and control over the semantic weight of the outputs, allowing a deeper semantic understanding of the user inputs."
  - "To start using Meilisearch's hybrid search features, you must create an account and gain access to the API keys and cloud platform. You can register for free and enjoy a 14-day trial."
  - "After registering, you can create a new project and use the vector store to add and index documents, run queries, monitor analytics, and more."
  - "In the settings tab, you'll find an option called Embedders, where you can enhance your hybrid search capabilities by integrating any embedding model of your choice."
  - "After adding a model, you can jump to the search preview tab and control the semantic weight directly there – you're using hybrid search!"
  - "To integrate the search engine into your workflow, use Meilisearch's API, available on the main page of the cloud dashboard."
* **Technical Entities (Classes/Functions/APIs):** `Langchain`, `Meilisearch`, `OpenAI embedding model`, `Meilisearch API`
* **Code Snippet:**
```python
import meilisearch


client = meilisearch.Client(
    '<meilisearch_server_url>',
    '<master_token>')
query = "Give me a book about a post-apocalyptic world"
results = client.index('books').search(query, opt_params={
  'hybrid': {
    'semanticRatio': 0.7,
    'embedder': 'openai'
  },
  'limit':4
})


for result in results['hits']:
    print(result['metadata']['text'])
```

---

## How does hybrid search compare to other search types?
* **Key Points:**
  - "Hybrid search combines two key approaches: semantic search and keyword search."
  - "However, semantic search is a broader term for methods that retrieve contextual or semantic meaning, including vector search and neural search."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## What is the difference between hybrid search and vector search?
* **Key Points:**
  - "Hybrid search enhances vector search by incorporating keyword matching for improved accuracy."
  - "As a form of semantic search, vector search relies on an embedder to generate dense vectors and retrieval algorithms like ANN and KNN to identify relevant results."
  - "By merging these with sparse vector outputs, hybrid search optimizes retrieval using techniques such as RFF."
* **Technical Entities (Classes/Functions/APIs):** `ANN`, `KNN`, `RFF`
* **Code Snippet:** None

---

## What is the difference between semantic search and hybrid search?
* **Key Points:**
  - "Hybrid search is a combination of semantic and keyword searches."
  - "The quality of the hybrid search system response highly depends on the embedder used for the semantic search."
  - "The better the embedder and the retrieval algorithm applied on the dense vectors, the better the semantical or contextual response of the hybrid search."
  - "In addition, the hybrid search uses keyword matching for lexical accuracy."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## What is the difference between keyword search and hybrid search?
* **Key Points:**
  - "Hybrid search utilizes keyword search to deliver precise lexical results."
  - "Keyword search relies on algorithms to generate sparse vectors from queries and documents, enabling fast and accurate retrieval."
  - "However, it lacks semantic understanding, so hybrid search integrates semantic search technologies to enhance search relevance and context."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## What is the difference between hybrid search and neural search?
* **Key Points:**
  - "Neural search, a type of semantic search, can be integrated into a hybrid search system."
  - "It leverages deep neural networks (DNNs) to deliver highly contextual results and supports various data input types."
  - "In a hybrid setup, neural search enhances typo tolerance while maintaining accuracy through lexical matching when users enter precise terms."
* **Technical Entities (Classes/Functions/APIs):** `DNNs (deep neural networks)`
* **Code Snippet:** None

---

## Hybrid search gives you the best of both worlds
* **Key Points:**
  - "Hybrid search delivers exceptional accuracy and contextual relevance, but implementing it can be complex."
  - "It involves selecting the right vector database, choosing optimal embedding models, and fine-tuning outputs from both dense and sparse vectors."
  - "Meilisearch simplifies this process by providing an intuitive platform backed by insightful tutorials."
  - "On this platform, you can effortlessly upload datasets, experiment with embeddings, fine-tune semantic relevance, and access advanced metrics like analytics and monetization."
* **Technical Entities (Classes/Functions/APIs):** `Meilisearch`
* **Code Snippet:** None



# Better RAG Results With Reciprocal Rank Fusion (RRF) and Hybrid Search


## Key takeaways
* **Key Points:**
  - "Retrieval‑augmented generation (RAG) is a technique to fetch relevant data and combine it with a language model to give accurate, context‑aware answers."
  - "A keyword search produces search results based on the given input keywords, whereas vector search uses semantic search to produce context aware results. A hybrid search combines both."
  - "Reranking reorders results from multiple retrieval methods, ensuring the most relevant documents appear at the top."
  - "RRF is an efficient reranking algorithm that combines rankings from multiple search methods into a single ranking, improving accuracy and relevance."
  - "Some important use cases of RRF are healthcare support and research, product search, financial forecasting, and market analysis."
* **Technical Entities (Classes/Functions/APIs):** `RAG (Retrieval-Augmented Generation)`, `RRF (Reciprocal Rank Fusion)`
* **Code Snippet:** None

---

## Search results using keyword search
* **Key Points:**
  - "Keyword search solely relies on the keywords given in the user query."
  - "The search results are based only on the keyword matches and disregard any contextual relevance, particularly in the case of ambiguous queries."
  - "For example, if you search for reciprocal rank fusion using keyword search, it will return only those results that contain these exact keywords, but you may not get related yet contextually appropriate results, like reranking, retrieval-augmented generation, and hybrid search."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Search results using vector search
* **Key Points:**
  - "Vector search looks for semantically appropriate results, based on the KNN algorithm."
  - "There are two types of searches: Sparse vector search: This method extends the keyword search by using weighted, high-dimensional sparse vectors that look for frequencies (less/more/absence/presence) of search terms. However, it does not fully capture the context or meaning of a search query."
  - "Dense vector search: Dense vectors represent data (for example, text) as numerical vectors in multiple dimensions, where each dimension captures some detail of the data. These dense vectors are created using a deep learning model. When a query is fired, the similarity between the vectors is measured to identify which vector representations are closest to each other. The closer the vectors are, the more related they are contextually. This way, dense vector search gives semantically appropriate search results."
* **Technical Entities (Classes/Functions/APIs):** `KNN algorithm`
* **Code Snippet:** None

---

## Hybrid search
* **Key Points:**
  - "Hybrid search combines various types of searches, like keyword and vector search, to produce multiple lists of relevant documents."
  - "These combined results can then be passed to a reranking module, which optimizes the final ranking order based on relevance signals or learned scoring."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Different types of retrieval methods
### BM25
* **Key Points:**
  - "BM25 is the traditional keyword search algorithm, where the results are based on the exact input words given by the user."
  - "If you want an exact match result, like in a document search, BM25 is an excellent choice."
* **Technical Entities (Classes/Functions/APIs):** `BM25`
* **Code Snippet:** None

---

### Vector search
* **Key Points:**
  - "Vector search retrieval captures semantic similarity between the input terms and tries to provide contextually relevant results."
  - "It uses dense vector embeddings to find close proximity words and improve the relevance of responses."
  - "It is quite useful for LLM-based question-answer systems and chatbots."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Multi-query retrieval
* **Key Points:**
  - "The multi-query retriever uses a large language model (LLM) to produce multiple versions or interpretations of the same user query."
  - "The search outcomes of each query are then aggregated to get a more comprehensive final query response."
  - "This can be useful for product recommendation systems, document retrievals, and academic research."
* **Technical Entities (Classes/Functions/APIs):** `multi-query retriever`
* **Code Snippet:** None

---

### Ensemble
* **Key Points:**
  - "Ensemble retriever combines keyword search (sparse retriever like BM25) and dense retriever (vector search) to produce a list of relevant documents."
  - "It uses methods like reciprocal rank fusion to combine scores from multiple retrieval methods and provide the final ranking and unified result."
  - "Combining semantic matches with keyword matches produces more effective and accurate results."
  - "Ensemble retriever is a good choice for search engines, recommendation systems, and many more use cases."
* **Technical Entities (Classes/Functions/APIs):** `Ensemble retriever`, `BM25`, `reciprocal rank fusion`
* **Code Snippet:** None

---

## Retrieval-augmented generation (RAG) using hybrid search
* **Key Points:**
  - "Using hybrid search greatly enhances RAG, as the results are more accurate compared to just keyword or semantic search."
  - "Reranking of search results using reciprocal rank fusion further ensures high quality results."
  - "Once the LLM receives a user prompt, it generates a query and sends both the query and the prompt to the data store to retrieve relevant documents."
  - "Since hybrid search is used, two separate ranked lists with scores are generated—one from keyword search and another from vector search. These scores are then combined, typically using a fusion technique such as reciprocal rank fusion, followed by reranking."
  - "The final set of results is returned to the LLM, which uses the information to generate a response for the end user."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `reciprocal rank fusion`
* **Code Snippet:** None

---

## Why RRF?
* **Key Points:**
  - "Implementing reciprocal rank fusion has several advantages."
  - "It combines the strength of various methods, including sparse and dense vector results. This improves the relevance of the results."
  - "An RRF algorithm assigns a reciprocal rank score based on the document ranks from multiple retrieval methods."
  - "This reduces hallucination and mitigates errors that might occur due to the use of individual methods, thus improving performance and reliability of search results."
  - "The accuracy and reliability is crucial in RAG applications like content summarization, information retrieval, and question answer systems."
* **Technical Entities (Classes/Functions/APIs):** `RRF (Reciprocal Rank Fusion)`
* **Code Snippet:** None

---

## How RRF works
* **Key Points:**
  - "As a first step, whenever a user fires a query, multiple searches are kicked off. It can be a keyword search, semantic search, or both. Each of these methods generates a ranking (of results)."
  - "The next step is to calculate the reciprocal rank score of each of the generated results. The score is calculated as: Score = 1/(rank+k)"
  - "k is a constant that helps in balancing the influence of individual rankings. The value of k decides the sensitivity to rank positions."
  - "In the above formula, rank is the position of the document in the list."
  - "In some implementations, different search strategies can be assigned different weights before combining their scores. This allows the system to favor one retrieval method—such as semantic search—over another, depending on the use case or domain requirements."
  - "The next step is to combine the scores obtained from each strategy and sum them to obtain a single score. Then, rank the documents again (rerank), based on the combined score. Documents having higher scores will be placed on the top in the final ranking."
  - "The final fused ranking is the list obtained from the above step, which is a blended result rather than the normalized result, a more accurate way to rank the results."
* **Technical Entities (Classes/Functions/APIs):** `RRF`
* **Code Snippet:**
```
Score = 1/(rank+k)
```

---

## Example query translation
* **Key Points:**
  - "Let's consider our previous example of Best places to visit in Paris, to illustrate how reciprocal rank fusion is applied to the results of multiple algorithms to produce the most relevant results."
  - "First, we will calculate the reciprocal ranking scores of the results obtained through keyword search, keeping the value of k as 60: Eiffel Tower = 1/(1+60) = 0.0164, Louvre Museum = 1/(2+60) = 0.0161, Notre-Dame Cathedral = 1/(3+60) = 0.0159"
  - "Next, we calculate the reciprocal ranking scores of the results obtained through semantic search: Montmartre = 1/(1+60) = 0.0164, Eiffel Tower = 1/(2+60) = 0.0161, Le Marais = 1/(3+60) = 0.0159, Seine River Cruise = 1/(4+60) = 0.0156"
  - "Now, let's add the scores and combine the results, with the highest score on top: Eiffel Tower = 0.0164 + 0.0161 = 0.0325, Montmartre = 0.0164, Louvre Museum = 0.0161, Le Marais = 0.0158, Notre-Dame Cathedral = 0.0158"
  - "While keyword search focuses on exact keywords, like sight-seeing locations, semantic search also considers individuals' personal experiences, gathered from blogs or reviews. Combining both can give the best of both worlds to a user."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Use cases of RAG with RRF
### E-commerce and retail:
* **Key Points:**
  - "Customers don't need to search for the exact product names, and can simply type what they want."
  - "Reranking helps produce results that are most useful to the user based on his search terms."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Healthcare and medical research:
* **Key Points:**
  - "Getting search data based on keyword (exact search words) and semantic search (similar studies) gives the most relevant evidence for diagnosis or treatment."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Market analysis:
* **Key Points:**
  - "Using data from multiple sources—like news reports, transcripts, blogs, and filings—and merging it with domain-specific data searches, produces a much more accurate and unified view of the financial data for analysis."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Recruitment process:
* **Key Points:**
  - "RAG can speed up the recruitment process by combining the resume details, along with the interview transcripts of a person."
  - "RRF can then produce accurate ranking of multiple candidates based on the practical interview experience as well as the data from the resume."
* **Technical Entities (Classes/Functions/APIs):** `RRF`
* **Code Snippet:** None