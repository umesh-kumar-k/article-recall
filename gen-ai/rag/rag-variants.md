---
aliases:
  - RAG architectural variants
tags:
  - rag
  - architecture
  - native-rag
  - advanced-rag
  - modular-rag
  - multi-index-rag
highlights: |-
  Native RAG: Simple retrieve then generate ; good starting point but limited accuracy (40 to 60 %) 

  Advanced RAG: Adds query transformation , hybrid search , reranking; production baseline (60 to 80 % accuracy)

  Modular RAG: Pluggable components (routeing, retrieval, generation) allowing specialized handling per query type

  Multi-index RAG: Separate indices for different data types (docs, code, tables, images) with query routing logic

  Temporal RAG: Time aware retrieval prioritizing recency or specific time ranges; critical for enterprise with version-controlled content
Source 1: https://www.decodingai.com/p/the-4-advanced-rag-algorithms-you
Source 2: https://medium.com/kx-systems/guide-to-multi-index-retrieval-for-optimized-rag-0acd2283e649
---
