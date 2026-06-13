---
aliases:
  - Architectural Decision Framework
Source 1: https://www.dbi-services.com/blog/rag-series-embedding-versioning-with-pgvector-why-event-driven-architecture-is-a-precondition-to-ai-data-workflows/
Source 2: https://redis.io/blog/10-techniques-to-improve-rag-accuracy/
Source 3: https://www.integrate.io/blog/enterprise-data-pipelines/
Source 4: https://www.informatica.com/resources/articles/enterprise-rag-data-ingestion.html
Source 5: https://www.ibm.com/think/topics/data-pipeline
Source 6: https://ragwalla.com/blog/choosing-between-cosine-similarity-dot-product-and-euclidean-distance-for-rag-applications
Source 7: https://www.dbi-services.com/blog/rag-series-agentic-rag/
Source 8: https://www.dbi-services.com/blog/rag-series-adaptive-rag-understanding-confidence-precision-ndcg/
Source 9: https://www.dbi-services.com/blog/rag-series-hybrid-search-with-re-ranking/
Source 10: https://www.dbi-services.com/blog/rag-series-naive-rag/
tags:
  - rag
  - agentic-rag
  - adaptive-rag
  - native-rag
  - rag-accuracy
  - data-pipeline
---
- **Accuracy Requirements**: 60% sufficient for internal tool vs 90 % for customer-facing
- **Latency Tolerance**: Real-time (<2s) interactive (<5s) or batch acceptable
- **Data Scale**: Thousands of docs (simpler stack) vs millions(dedicated vector DB)
- **Update Frequency**: Static datasets vs real-time sync requirements
- **Context Complexity**: Simple lookup vs multi-hop reasoning needs
- **Const Constraints**: Optimize for development velocity vs operational costs
- **Compliance**: Public cloud acceptable vs on-prem mandate
- **Customization**: Generic models sufficient vs domain-specific fine-tuning required
