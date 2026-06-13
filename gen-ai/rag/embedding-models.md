---
aliases:
  - Embedding Models
Source 1: https://medium.com/bright-ai/choosing-the-right-embedding-for-rag-in-generative-ai-applications-8cf5b36472e1
Source 2: https://www.mongodb.com/company/blog/technical/how-choose-best-embedding-model-for-your-llm-application
Source 3: https://milvus.io/ai-quick-reference/what-are-dense-and-sparse-embeddings
Source 4: https://mlokhandwalas.medium.com/dense-and-sparse-embeddings-a-comprehensive-overview-c5f6473ee9d0
tags:
  - embedding
  - embedding-models
  - rag
highlights: |-
  Transform text/data into dense vector representations 768/4096 dimensions , enabling semantic similarity search; choice impacts retrieval quality significantly


  OpenAI text-embedding-3: 1536 or 3073 dimensions; high quality; API dependency and cost

  Cohere Embed v3: Multilingual,optimized for RAG, compression-aware; competetive quality

  Azure OpenAI Embeddings: OpenAI models with enterprise SLA and data residency guarantees

  BGE(BAAI): Open-source SOTA models (bge-large-en-v1.5); self-hostable on GPUs

  E5(Microsoft): Instruction-turned embeddings; string zero shot cross domain performance

  Voyage AI: Specialized domain embeddings(code,finance, legal);higher accuracy than general-purpose

  Sentence Transformers: Ecosystem for fine-tuning custom domain embeddings; requires ML expertise
---
