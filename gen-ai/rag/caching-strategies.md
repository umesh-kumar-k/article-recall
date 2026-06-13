---
aliases:
  - Caching Strategies
highlights: |-
  Embedding Cache: Store computed embeddings to avoid preprocessing unchanged documents

  Retrieval Cache: Cache query results for frequent/similar queries; semantic similarity for cache hits 

  LLM Response Cache: Exact or fuzzy matching on (query, context) pairs, GPTCache, Redis-based solutions

  Semantic Cache: Vector similarity on query embeddings to return cached responses for semantically similar questions
tags:
  - rag
  - cache
Source 1: https://redis.io/blog/using-redis-for-real-time-rag-goes-beyond-a-vector-database/
Source 2: https://towardsdatascience.com/beyond-prompt-caching-5-more-things-you-should-cache-in-rag-pipelines/
Source 3: https://towardsdatascience.com/zero-waste-agentic-rag-designing-caching-architectures-to-minimize-latency-and-llm-costs-at-scale/
---
