---
aliases:
  - Hybrid Search
Source 1: https://www.infoq.com/articles/vector-search-hybrid-retrieval-rag/
---
[Code Examples](./code/hybrid-search.md) 

# Why Vector Search Alone Isn't Enough: Hybrid Retrieval for RAG


* **Key Points:**
  - In this post, we demonstrate a solution to improve the quality of answers in such use cases over traditional RAG systems by introducing an interactive clarification component using LangChain.
  - The key idea is to enable the RAG system to engage in a conversational dialogue with the user when the initial question is unclear.

## Where Vector-Only RAG Pipelines Break
* **Key Points:**
  - To understand why this situation happens, it helps to zoom out and look at the full pipeline.
  - A RAG pipeline has three stages, as shown in Figure 1.
  - The elements in Figure 1 can be defined as follows: Chunking breaks the source corpus into indexable units. Retrieval takes the user's query, searches over those chunks and returns the top-K most relevant ones. Generation hands those chunks to an LLM as context and asks it to produce an answer.
  - Assume the first and third stages work correctly.
  - The retrieval stage is where the failure from the introduction occurs.
  - The retriever embeds the query, compares it to the indexed document vectors, and returns whichever documents sit closest in embedding space.
  - Closeness in embedding space means semantic similarity, not identity.
  - The retriever cannot reliably distinguish them.
  - That is the breaking point, where the same vector space and the same scoring mechanism retrieves the wrong document at the top.

## The Problem Is That Embeddings Are Approximation Engines
* **Key Points:**
  - Embedding models like BERT convert text into fixed-dimensional numerical vectors that capture the semantic meaning of text.
  - Text with similar meanings produces similar vectors.
  - This clustering is extremely useful when a user searches for an idea.
  - It becomes a precision problem when a user needs an exact entity, a specific feature flag name, a specific error code, or a specific deployment version.
  - The embedding model behaves exactly as designed.
  - It is built to find similar, not identical things.
  - When the distinguishing element is small relative to the surrounding text, embeddings collapse the distinction.
  - Search queries fall into three broad categories based on which retrieval method handles them best:
    - Semantic Queries: A user asking "What's our protocol when a region goes offline?" is asking about a concept.
    - Exact-Match Queries: These queries are also called lexical queries in the IR literature.
    - Hybrid Queries: A user searching "rollback runbook for v3.2 deployment" needs a semantic understanding (i.e., a runbook for the deployment-rollback operation) and exact matches on the distinguishing tokens.
  - In my experience with production RAG systems, they are the majority.

## BM25 Provides Precision Where Embeddings Approximate
* **Key Points:**
  - Vector search needs a partner and the partner is BM25, the probabilistic ranking function at the heart of classical information retrieval.
  - It is the default scorer in Elasticsearch, OpenSearch, and most lexical search engines, as well as the dominant keyword-search algorithm for the better part of three decades.
  - It succeeds precisely where vector search fails.
  - It relies on probabilistic information retrieval theory with three built-in mechanisms that directly address the exact-match problem:
    - Inverse document frequency (IDF) measures how rare a term is across the corpus. Common words like "service" or "deployment" receive low weight, while rare distinguishing tokens like "v3.2", "ERR_PAYMENT_GATEWAY_TIMEOUT" or "payment_v2_enforce" receive high weight.
    - Term frequency (TF) saturation controls the impact of repeated terms. The first mention of a term significantly impacts the score, but subsequent mentions yield diminishing returns.
    - Length normalization addresses another bias in text retrieval. Longer documents tend to score higher simply because they contain more words, giving them more opportunities to match query terms. Length normalization corrects for this scenario by factoring in document length when computing relevance scores, factoring in not just how many times a term appears, but how often relative to the document's length.
* **Technical Entities (Classes/Functions/APIs):** `BM25`, `Elasticsearch`, `OpenSearch`, `Inverse document frequency (IDF)`, `Term frequency (TF) saturation`, `Length normalization`

## Hybrid Search with Reciprocal Rank Fusion
* **Key Points:**
  - Looking at Figure 3, hybrid retrieval runs BM25 and vector search in parallel, fuses their ranked lists with RRF and optionally reranks with a cross-encoder before passing the top-K chunks to the LLM.
  - At this point, we have two retrievers with complementary strengths, vector search and BM25.
  - Vector search captures semantic meaning, while BM25 matches exact tokens.
  - Each produces its own ranked list.
  - To handle hybrid queries, those two lists need to be combined into one.
  - The combination itself is the hard part.
  - Vector cosine similarity is bounded between -1 and 1.
  - BM25 scores are unbounded.
  - Normalizing them onto a common scale is tricky.
  - The correct weights are query-dependent: for one query, the right weight on BM25 might be 0.7, for another 0.3.
  - Calibrating these weights per-query at production scale is impractical.
  - This situation is where Reciprocal Rank Fusion (RRF) helps.
* **Technical Entities (Classes/Functions/APIs):** `RRF`, `Reciprocal Rank Fusion`, `cross-encoder`

### Deep Dive into How RRF Helps Combine Scores
* **Key Points:**
  - RRF sidesteps the normalization problem entirely by ignoring raw scores from either retriever.
  - It operates on rank position alone: RRF_Score(d) = Σ 1 / (k + rank_r(d))
  - The constant k, which is typically 60 (Cormack, Clarke, and Buettcher 2009), smooths the contribution of each rank position.
  - A document at rank 1 contributes 1/61 ≈ 0.0164. A document at rank 10 contributes 1/70 ≈ 0.0143.
  - A document missing from a retriever's top-K contributes 0 from that retriever.
  - The mechanism is straightforward.
  - Documents that both retrievers rank in their top results get the highest fused scores, because they receive non-zero contributions from each.
  - Documents that only one retriever finds get demoted, even if that retriever ranks them at the top.
  - RRF rewards consensus.
  - In my experience, production query distributions are dominated by the third case.
  - Most real-world queries combine semantic intent with specific identifiers, version numbers, error codes, or other tokens that demand exact matching.
  - Hybrid retrieval is the engineering response to that distribution.

## Hybrid Retrieval in Production
* **Key Points:**
  - Production RAG systems have converged on hybrid retrieval.
  - Perplexity fuses lexical and embedding-based scorers across hundreds of billions of URLs on Vespa, with a multi-stage ranking that ends in cross-encoder reranking.
  - Glean layers, lexical retrieval and dense embeddings over a proprietary knowledge graph for enterprise search.
  - Two different domains, the same architectural pattern.
* **Technical Entities (Classes/Functions/APIs):** `Vespa`, `Glean`

### Production Implementation of Elasticsearch
* **Key Points:**
  - Elasticsearch and OpenSearch both support hybrid retrieval natively through the retriever API (Elasticsearch 8.13+ with OpenSearch following).
  - Native support means the fusion happens inside the search engine in a single query, with no application-level merging logic.
  - The examples below use Elasticsearch syntax; OpenSearch syntax is nearly identical.
* **Technical Entities (Classes/Functions/APIs):** `Elasticsearch`, `OpenSearch`, `retriever API`

### Index Mapping
* **Key Points:**
  - Your index requires a standard text field for BM25 and a dense vector field for embeddings
* **Technical Entities (Classes/Functions/APIs):** `dense_vector`, `text`
* **Code Snippet:**
```json
PUT /rag_knowledge_base
{
  "mappings": {
    "properties": {
      "title":   { "type": "text" },
      "content": { "type": "text", "analyzer": "standard" },
      "content_vector": {
        "type": "dense_vector",
        "dims": 768,
        "index": true,
        "similarity": "cosine"
      }
    }
  }
}
```

### Hybrid Query with RRF
* **Key Points:**
  - The query structure runs both retrievers and fuses them in a single request
* **Technical Entities (Classes/Functions/APIs):** `rrf`, `retrievers`, `standard`, `knn`
* **Code Snippet:**
```json
POST /rag_knowledge_base/_search
{
  "retriever": {
    "rrf": {
      "retrievers": [
        {
          "standard": {
            "query": { "match": { "content": "rollback runbook for v3.2 deployment" } }
          }
        },
        {
          "knn": {
            "field": "content_vector",
            "query_vector": [0.12, -0.45, ...],
            "k": 50,
            "num_candidates": 100
          }
        }
      ],
      "rank_constant": 60
    }
  }
}
```

### Tuning for Production
* **Key Points:**
  - The default configuration above is a reasonable starting point, but production systems almost always need tuning.
  - Three parameters in particular drive most of the relevance and latency tradeoffs you will encounter:
    - Rank Constant (k): The rank constant is the smoothing parameter in the RRF formula that controls how steeply rank contributions decay; a document at rank r contributes 1/(k + r) to its fused score.
    - kNN Candidates: The num_candidates parameter sets how many vectors the HNSW graph traversal explores before returning the top-K, controlling the recall-latency tradeoff in approximate nearest neighbor search.
    - Reranking with Cross-Encoders: Hybrid retrieval with RRF gets you a strong candidate set, but a cross-encoder re-ranking stage can meaningfully improve final relevance.
* **Technical Entities (Classes/Functions/APIs):** `num_candidates`, `HNSW`, `cross-encoder`, `ms-marco-MiniLM-L-6-v2`, `BEIR`
* **Key Points:**
  - In practice, the pattern is to have RRF retrieve twenty to fifty candidates, then pass them through a cross-encoder like ms-marco-MiniLM-L-6-v2 for final ordering.
  - Cross-encoders are too slow for first-stage retrieval (they require a forward pass per query-document pair), but for re-ranking a small candidate set, the latency is usually acceptable, typically under one hundred milliseconds for fifty candidates on a GPU.
  - Cross-encoders consistently outperform bi-encoders on standard retrieval benchmarks like BEIR, with larger models showing the largest gains on out-of-domain queries and even lightweight models providing meaningful gains in-domain.
  - For production systems where every percentage point of relevance matters, this final stage is worth the investment.

## Conclusion
* **Key Points:**
  - Dense embeddings solve the generalization problem in retrieval since they find conceptually relevant documents even when query terms do not match document terms.
  - BM25 solves the precision problem where it finds exact matches based on rare, distinguishing tokens.
  - But neither alone is sufficient for the production RAG.
  - Embeddings are approximation engines, which is their strength and their limitation.
  - Hybrid search with RRF is not a workaround for a temporary gap in model quality; it is the architecturally correct approach for systems that must handle both conceptual and exact-match queries.
  - If you are running a RAG pipeline on embeddings alone, you are leaving retrieval quality on the table.
  - Add BM25, fuse with RRF, and consider a cross-encoder re-ranking stage.