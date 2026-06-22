---
aliases:
  - Vector Search
Source 1: https://bigdataboutique.com/blog/opensearch-and-elasticsearch-vector-search-an-introduction-6af584
Source 2: https://bigdataboutique.com/blog/sparse-vs-dense-vectors-how-lexical-and-semantic-search-actually-work
---

# Vector Search in Elasticsearch & OpenSearch: How It Works + When to Use It

## What Is Vector Search?
* **Key Points:**
  - Vector search is a method used in information retrieval and machine learning where documents or data points are represented as vectors in a high-dimensional space. Each vector dimension corresponds to a distinct data characteristic or attribute.
  - The more characteristics shared between vectors, the closer they will be located in the vector space. This is why vectors representing text documents will be closer when the text is about a similar theme.
  - In order to be able to perform vector search, raw data such as text, photos, audio, or video need to be embedded using an embedding function to create the vectors.
  - Then, by evaluating the distance between different vectors, a vector-search engine can find similar results based on their proximity in this space. Allowing for quick and scalable retrieval of similar records.

## Benefits and Challenges of Vector Search
* **Key Points:**
  - Vector search brings search to an entirely new level because it captures semantics much better than other search methods.
  - Common use cases for vector search include:
    - Semantic search – Vector search lets you find what users mean and goes beyond exact keyword match, opening the door to finding synonyms, analogs, and taxonomies.
    - Recommendation engines – Similar documents and their vectors in the embedding space get identified by the model that creates the embedding.
    - Image and audio search - This refers to embedding meaning in vectors for aspects that aren't part of the text for similarity search or finding similar items based on various definitions of similarity. For example, you can search for images that are similar in size, color, or content or song in a genre that you prefer.

## Keyword search under the hood
* **Key Points:**
  - At their core, both Elasticsearch and OpenSearch use BM25 to rank documents for keyword search. BM25 is a ranking function that builds on the Bag of Words (BoW) representation by introducing term weighting and document length normalization.
  - Most values would be 0, hence this type of vector is also known as a sparse vector.
  - However, as we mentioned earlier, this approach has major limitations:
    - Word order is ignored - "cat chases dog" vs. "dog chases cat" are treated the same.
    - Synonyms and related concepts are not captured - "smartphone" and "mobile phone" would be created as different terms.
    - Sparse and high-dimensional - since each word gets its own axis in the vector space, the resulting vectors are very large.
* **Technical Entities (Classes/Functions/APIs):** `BM25`, `Bag of Words (BoW)`, `Elasticsearch`, `OpenSearch`

## Embeddings and Dense Vectors
* **Key Points:**
  - To overcome these limitations, embedding models are AI models that are trained to create dense vector representations that capture semantic meaning rather than just word occurrence.
  - This vector has a much lower dimensionality, depending on the embedding model, but it's highly expressive (hence the term dense vector), and while it might be unclear to the naked eye how each value relates to the original sentence, it will capture the meaning of the original sentence, rather than individual words.
  - We can store the vector alongside the original text, to use it later, for search. Then if we want to find documents to answer the question.
  - We would convert the question into a vector using the same embedding algorithm, then we would search for the documents closest to it (In terms of cosine similarity or Euclidean distance), and retrieve the relevant results, even if none of the exact keywords are present.
* **Technical Entities (Classes/Functions/APIs):** `OpenAI embeddings`, `Cohere embedding`, `cosine similarity`, `Euclidean distance`

## Vector Search: A Dive Into kNN vs ANN
* **Key Points:**
  - The final piece of the puzzle is understanding how Elasticsearch and OpenSearch retrieve the closest vectors for a given query.
  - To do this, they rely on two main techniques: k-Nearest Neighbors (k-NN) and Approximate Nearest Neighbors (ANN), both of which scan through documents with embeddings to identify the most relevant matches.
  - k-NN (k-Nearest Neighbors) performs an exhaustive brute-force search, calculating the distance between the query vector and every vector in the index. This method guarantees 100% accuracy but is computationally expensive, making it impractical for large-scale datasets. It is best suited for smaller indices where precision is prioritized over speed.
  - ANN (Approximate Nearest Neighbors), introduced in Elasticsearch 8.0, improves search speed by using an optimized indexing algorithm, such as HNSW (Hierarchical Navigable Small World). This approach trades off some accuracy for significantly faster retrieval, making it ideal for large-scale semantic search applications.
  - In short, k-NN ensures perfect accuracy at the cost of performance, while ANN prioritizes speed, making it the preferred choice for production use cases.
* **Technical Entities (Classes/Functions/APIs):** `k-Nearest Neighbors (k-NN)`, `Approximate Nearest Neighbors (ANN)`, `HNSW (Hierarchical Navigable Small World)`, `Elasticsearch 8.0`

## Summary
* **Key Points:**
  - We explored the evolution of search in Elasticsearch and OpenSearch, from traditional keyword-based retrieval using BM25 to semantic search powered by dense vector embeddings.
  - In other articles, we dive deeper into embedding models, show you how to generate embeddings for text (for instance using Elasticsearch ELSER), and walk through setting up hybrid search that combines keyword and vector search in both Elasticsearch and OpenSearch. Also worth noting is our Elasticsearch vs OpenSearch comparison specifically in the context of vector search.
  - Bringing vector search into production on OpenSearch? Our OpenSearch consulting services cover k-NN engine selection, HNSW tuning, embedding pipelines via ML Commons, and end-to-end hybrid retrieval architectures.
* **Technical Entities (Classes/Functions/APIs):** `Elasticsearch ELSER`, `ML Commons`

---

# Sparse vs Dense Vectors: How Lexical and Semantic Search Actually Work

## Sparse vs Dense Vectors: How Lexical and Semantic Search Actually Work
* **Key Points:**
  - Every keyword search you've ever run is a sparse vector operation. Every time BM25 scores a document, it's comparing sparse representations - arrays of term weights where most values are zero. This has worked well for decades. But the way people search is changing fast.
  - LLMs now generate search queries on behalf of users. ChatGPT alone has 800 million weekly active users, and 69% of Google searches end without a click as AI-generated answers take over. When ChatGPT does web search, when Claude Code looks up documentation, when Perplexity retrieves sources - they don't type keywords the way humans do. They paraphrase, they expand, they use conceptual language.
  - If your search system only does keyword matching, it will return results about "auto index rollover" or "index lifecycle management" instead of the actual "Create Index API" page. Your precision drops. Your recall drops. Your users get bad answers.
  - This is why understanding both sparse and dense vectors - what they actually do, where each one breaks - matters more now than ever.

## Sparse Vectors: From BM25 to Learned Representations
* **Key Points:**
  - A sparse vector is a high-dimensional representation where most values are zero. Each non-zero dimension corresponds to a specific feature - typically a word, token, or n-gram. A document about "Elasticsearch cluster optimization" might produce a vector with 30,000 dimensions, but only a few dozen carry non-zero weights.

### Classic Sparse: TF-IDF and BM25
* **Key Points:**
  - TF-IDF (Term Frequency-Inverse Document Frequency) weights terms by how often they appear in a document relative to the corpus. Common words get low weight. Rare, domain-specific terms get high weight.
  - BM25 improves on TF-IDF with saturation functions and document length normalization. It's the backbone of lexical search in Elasticsearch, OpenSearch, and every Lucene-based system. BM25 scores around 0.429 nDCG@10 on the BEIR benchmark averaged across 18 datasets - a solid baseline that many dense models struggled to beat until recently.
  - The strengths are real: sub-millisecond latency through inverted indexes, full explainability (you can see that "Elasticsearch" matched with weight 4.2 and "cluster" with weight 3.8), no GPU required, and battle-tested reliability.
  - The weakness is just as real: a query for "automobile" won't match documents about "cars." The word "learn" won't match "teach." BM25 has zero semantic understanding.
* **Technical Entities (Classes/Functions/APIs):** `TF-IDF`, `BM25`, `Elasticsearch`, `OpenSearch`, `Lucene`, `BEIR`

### Learned Sparse: SPLADE and Beyond
* **Key Points:**
  - SPLADE (Sparse Lexical and Expansion) bridges this gap. It uses BERT's masked language modeling head to expand terms contextually while maintaining sparsity. Feed it "affordable electric cars" and it generates a sparse vector that includes weights for "cheap," "vehicle," "price," and "Tesla" - terms not in the original text but semantically related. The key insight: SPLADE doesn't just learn synonyms. It learns which expansions matter for retrieval.
  - The latest generation of learned sparse models pushes further. LACONIC, built on Llama-3, achieves 60.2 nDCG@10 on the MTEB Retrieval benchmark - the only sparse model in the top 15, a position dominated by dense retrievers. It also uses 71% less index memory than an equivalent dense model.
  - Learned sparse models still run on inverted indexes at query time. The GPU cost is only at indexing. That's a meaningful operational advantage over dense retrieval.
* **Technical Entities (Classes/Functions/APIs):** `SPLADE`, `BERT`, `LACONIC`, `Llama-3`, `MTEB Retrieval benchmark`

### Sparse Models Comparison Table
* **Key Points:**
  - Model | Type | nDCG@10 (BEIR avg) | GPU Required | Explainable
  - BM25 | Classic sparse | 42.9 | No | Yes
  - SPLADE v3 | Learned sparse | 51.3 | For encoding | Partially
  - CSPLADE | Learned sparse | 54.6 | For encoding | Partially
  - LACONIC-8B | Learned sparse | 60.2 | For encoding | Partially

## Dense Vectors: Semantic Power at a Cost
* **Key Points:**
  - A dense vector is fully populated - every dimension carries a value, learned through neural networks. Where sparse vectors say "this document contains the word cluster with weight 3.8," dense vectors encode something more abstract: the meaning of the text as a point in a high-dimensional space. Two documents that discuss the same concept end up near each other, even if they share no words.

### The Embedding Model Landscape
* **Key Points:**
  - The field moves fast. As of early 2026, the MTEB leaderboard looks very different from a year ago.
  - Open-source models now match or exceed closed-source alternatives. Qwen3-Embedding and NV-Embed lead the MTEB leaderboard while being free to self-host. For teams that can run inference infrastructure, this eliminates per-token API costs entirely.
  - Matryoshka Representation Learning makes high-dimensional models more practical. Models trained with MRL let you truncate embeddings - say, from 3072 to 768 dimensions - while retaining 90-95% of retrieval accuracy. OpenAI's text-embedding-3-large at 256 dimensions still outperforms their older ada-002 at 1536 dimensions. This means you can trade a small accuracy hit for 4-12x less memory and proportionally faster search.
* **Technical Entities (Classes/Functions/APIs):** `Google Gemini Embedding`, `NV-Embed (NVIDIA)`, `Qwen3-Embedding-8B`, `Cohere Embed v4`, `OpenAI text-embedding-3-large`, `BGE-M3`, `Matryoshka Representation Learning (MRL)`, `MTEB`

### Embedding Models Comparison Table
* **Key Points:**
  - Model | Dimensions | MTEB Score | Cost per 1M tokens | Open Source
  - Google Gemini Embedding | 3072 | 68.3 | ~$0.004/1K tokens | No
  - NV-Embed (NVIDIA) | 4096 | 69.3 | Self-host | Yes
  - Qwen3-Embedding-8B | 1024 | 70.6 (multilingual) | Self-host | Yes (Apache 2.0)
  - Cohere Embed v4 | 1536 | 65.2 | $0.12 | No
  - OpenAI text-embedding-3-large | 3072 | 64.6 | $0.13 | No
  - BGE-M3 | 1024 | 63.0 | Self-host | Yes (MIT)

### Token Limits and Chunking
* **Key Points:**
  - Every embedding model has an input limit. BERT-based models typically cap at 512 tokens. Newer models like BGE-M3 and OpenAI's embeddings accept up to 8,192 tokens. But even with a high token limit, embedding a 20,000-word document as a single vector is a bad idea. Different sections discuss different topics, and a single vector can't faithfully represent all of them.
  - This is where chunking comes in. You split documents into coherent segments - each one gets its own vector representing its specific meaning. Research on optimal chunk sizes shows recursive 512-token splitting with 10-20% overlap as a pragmatic starting point, though domain-specific content may benefit from topic-boundary chunking that aligns splits with logical section breaks.
  - The chunking strategy you pick matters as much as your embedding model choice. Get it wrong and you'll embed fragments too small to carry meaning or chunks too large to be specific.

### HyDE: Flipping the Retrieval Problem
* **Key Points:**
  - HyDE (Hypothetical Document Embeddings) takes a different approach to the query-document mismatch problem. Instead of embedding the user's query and hoping it lands near the right documents, you ask an LLM to generate a hypothetical answer - even if it's wrong - and embed that instead. The hypothesis shares vocabulary and structure with real documents, so its embedding lands closer to relevant results.
  - On TREC DL-20, HyDE achieves nDCG@10 of 61.3 versus 44.5 for the unsupervised Contriever baseline - a 38% improvement with zero training data. The trade-off is latency: you're adding an LLM generation step before retrieval, which adds 25-60% overhead depending on model size.
  - HyDE works best for short, ambiguous queries where the direct embedding would be too vague. For keyword-heavy or exact-match queries, it adds cost without benefit.
* **Technical Entities (Classes/Functions/APIs):** `HyDE (Hypothetical Document Embeddings)`, `TREC DL-20`, `Contriever`

### The Cost of Dense Search
* **Key Points:**
  - Dense vectors aren't cheap at scale. A collection of 10 million documents with 1536-dimensional float32 embeddings requires roughly 60GB just for the vectors - all of which needs to be in memory or on fast storage for ANN search.
  - The scoring is opaque: you get a similarity score of 0.87, but there's no breakdown of why two items matched. In regulated industries - healthcare, finance, legal - this lack of explainability can be a dealbreaker.
  - Elasticsearch offers its own sparse model, ELSER, as an alternative to external dense embeddings - worth evaluating if you're already in the Elastic ecosystem.
  - There's also a subtler failure: semantic drift. A query about "Python 3.11 performance improvements" might return results about "Python optimization techniques" in general. The embeddings are close in vector space, but the retrieved documents don't actually answer the question. You know that results matched but not why, which makes debugging these failures harder than debugging a BM25 miss.
* **Technical Entities (Classes/Functions/APIs):** `ANN search`, `ELSER`, `Elasticsearch`

## Where Each Approach Fails
* **Key Points:**
  - Neither sparse nor dense retrieval is universally better. Each has failure modes that the other handles well.

### Dense Vector Failures
* **Key Points:**
  - Semantic drift. Dense models retrieve conceptually related but factually wrong results. Searching for a specific error code like FORBIDDEN/12/index read-only might return generic articles about Elasticsearch permissions instead of the exact troubleshooting page.
  - Over-generalization. Model numbers, API endpoint names, error messages, product SKUs - anything that demands exact lexical matching is a weak spot. Dense embeddings can miss the exact document while returning ten "close enough" alternatives.
  - Hallucinated similarity. Two documents using similar language about completely different topics can score high. A document about "scaling a Redis cluster" and one about "scaling an Elasticsearch cluster" might get near-identical embeddings, even when only one is relevant.

### Sparse Vector Failures
* **Key Points:**
  - Synonym gaps. Without expansion (like SPLADE), "automobile" doesn't match "car." "Laptop" doesn't match "notebook computer." This is the classic vocabulary mismatch problem.
  - No semantic understanding. "The product is not bad" and "The product is bad" differ by one word, but their sparse representations are nearly identical. Sparse vectors can't handle negation, sarcasm, or implied meaning.
  - LLM query mismatch. As search queries increasingly come from LLMs rather than humans, the gap between how queries are phrased and how documents are written keeps growing. An LLM might search for "containerized deployment orchestration platform comparison" when the relevant document title is "Kubernetes vs Docker Swarm."

## The Takeaway
* **Key Points:**
  - Research confirms what production systems have shown: to reach a recall@1000 of 0.98, you need both sparse and dense retrieval. Neither alone gets there.
  - The production standard is combining both. Hybrid search - running sparse and dense retrieval in parallel and fusing results - improves recall 15-30% over either method alone.

## Key Takeaways
* **Key Points:**
  - Keyword search alone won't survive the AI era. LLMs generate queries differently from humans - more conversational, more paraphrased, less keyword-dependent. Systems that rely solely on BM25 will see increasing recall degradation.
  - Sparse vectors aren't obsolete. BM25 still dominates for exact-match scenarios, and learned sparse models like LACONIC now compete with dense retrievers on benchmark accuracy while using 71% less index memory.
  - Dense vectors aren't magic. They excel at semantic matching but fail on exact precision, explainability, and cost at scale. A 10M document collection needs ~60GB just for vector storage at 1536 dimensions.
  - Chunking strategy matters as much as model choice. Dense vectors require splitting documents into coherent segments. Bad chunking - too small, too large, or splitting mid-thought - can undermine even the best embedding model.
  - HyDE is worth considering for ambiguous queries. Generating a hypothetical answer before embedding can boost retrieval by 38%, but at the cost of added latency from an LLM call.
  - The production standard is combining both. Hybrid search - running sparse and dense retrieval in parallel and fusing results - improves recall 15-30% over either method alone.