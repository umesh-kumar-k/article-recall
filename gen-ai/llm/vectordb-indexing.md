---
aliases:
  - Vector DB Indexing Algorithm
---
# HNSW vs IVF-Flat: Choosing the Right Vector Index for Similarity Search

## The Core Problem
* **Key Points:**
  - When you store millions of embeddings, the goal is to find the nearest neighbours of a query vector quickly. A brute-force approach, which computes distances between the query and every stored vector, is accurate but too slow for large-scale applications.
  - Approximate methods like HNSW and IVF-Flat reduce the search space drastically while maintaining high recall. Both are designed to deliver good trade-offs between speed, accuracy, and memory usage.

## What is IVF-Flat?
* **Key Points:**
  - IVF-Flat stands for Inverted File with Flat quantisation. It organises vectors into clusters using k-means. Each cluster represents a "list" of vectors that are spatially close.
  - During search:
    - The query vector is compared with all cluster centroids.
    - The closest few clusters (defined by nprobe) are selected.
    - The search is then restricted to vectors within those clusters.
  - This approach avoids searching the entire dataset, leading to much faster queries.
  - Key advantages:
    - Fast query performance when the number of clusters and probes are tuned correctly.
    - Moderate memory usage.
    - Well suited for very large datasets.
  - Limitations:
    - Requires an initial training phase for clustering.
    - Insertions or deletions after training are less efficient.
    - Accuracy depends on the number of clusters probed.
  - Ideal use cases: IVF-Flat is a strong choice for large, mostly static datasets where high throughput and moderate recall are acceptable. Examples include product recommendation catalogs, image retrieval systems, or offline document search.
* **Technical Entities (Classes/Functions/APIs):** `IVF-Flat`, `k-means`, `nprobe`

## What is HNSW?
* **Key Points:**
  - HNSW stands for Hierarchical Navigable Small World graph. It builds a multi-layer graph where each node represents a vector and is connected to a set of nearby nodes. Higher layers form a coarse representation of the space, while lower layers capture fine-grained relationships.
  - When searching:
    - The algorithm starts at the top layer and navigates toward the closest node.
    - It moves down through the layers, refining the search until it reaches the bottom layer.
  - This hierarchical structure allows HNSW to find approximate nearest neighbours with very high recall and low latency.
  - Key advantages:
    - Very high recall, often close to brute-force accuracy.
    - Supports real-time insertions and deletions.
    - Excellent for dynamic or continuously updated datasets.
  - Limitations:
    - Higher memory usage due to graph connectivity.
    - Slower index construction compared to IVF-Flat.
    - Requires tuning parameters like M, efConstruction, and efSearch.
  - Ideal use cases: HNSW is well suited for systems that require high accuracy and frequent updates, such as conversational AI retrieval, personalisation engines, or any RAG (Retrieval-Augmented Generation) pipeline that evolves over time.
* **Technical Entities (Classes/Functions/APIs):** `HNSW`, `M`, `efConstruction`, `efSearch`

## Practical Example
* **Key Points:**
  - Suppose you are building a document retrieval system for a customer support chatbot.
  - If your content library changes often, with new FAQs and articles added regularly, HNSW would be a better choice. It offers near-perfect recall and supports incremental updates without retraining the entire index.
  - On the other hand, if you manage a product database with millions of vector representations that change infrequently, IVF-Flat would be more efficient. Once the clusters are trained, searches are extremely fast, and memory usage remains manageable.

## Hybrid Approaches
* **Key Points:**
  - In some modern systems, both methods are combined. IVF-HNSW is a common hybrid approach where IVF partitions the dataset into clusters and HNSW performs fine-grained search within each cluster. This setup delivers scalability with strong recall performance and is supported in FAISS and Milvus.
* **Technical Entities (Classes/Functions/APIs):** `IVF-HNSW`, `FAISS`, `Milvus`

## Conclusion
* **Key Points:**
  - Both HNSW and IVF-Flat are powerful indexing methods, but the choice depends on your application's characteristics.
  - Use HNSW when you need high recall, low latency, and frequent updates.
  - Use IVF-Flat when your dataset is large, stable, and query throughput is a higher priority than update frequency.
  - Ultimately, no single method is universally best. It's worth benchmarking both on your data and measuring recall, latency, and memory consumption. The right choice depends on how your system needs to balance precision, speed, and scale.