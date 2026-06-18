# Core & Advanced Concepts

### Fundamentals

-  [Embedding Models:](embedding-models.md) Transform text/data into dense vector representations (768 to 4096 dimensions) enabling semantic similarity search; choice impacts retrieval quality significantly 
- [Vector Databases:](vector-databases.md) Purpose-built datastores(Pinecone,Weaviate,Qdrant, pgvector) optimized for similarity search using ANN algorithms(HNSW,IVF)
- [Chunking Strategy:](chunking-strategy.md) Document segmentation approach(fixed-size,semantic,recursive) that balances context completeness vs retrieval precision; typically 256 to 1024 tokens
- [Retrieval Metrics:](retrieval-metrics.md) Evaluate using Precision@K, Recall@K, MRR,NDCG; distinguish between retrieval quality & end to end answer quality

## Advanced Techniques

- [Hybrid Search:](hybrid-search.md) Combines dense vector search with sparse retrieval(BM25, keyword) using reciprocal rank fusion for better recall across diverse query types
- [Reranking:](reranking.md) Two stage retrieval where initial candidates are rescored using cross-encoders or LLM based rankers to improve precision
- [Query Transformation:](query-transformation.md) Techniques like HyDE(hpothetical document embeddings), query expansion, step-back prompting to bridge query-document vocabulary gaps.
- [Self-RAG:](self-rag.md) System generates, retrieves & critiques its own output using reflection tokens to improve answer reliably.
- [Contextual Compression:](contextual-compression.md) Post retrieval filtering/summarization to remove irrelevant chunks before passing to LLM, optimizing context window usage
- [Metadata Filtering:](metadata-filtering.md) Pre Filtering by structured attributes(date,source,department) before vector search to enforce access control & relevance
- [Agentic RAG:](agentic-rag.md) Multi Step reasoning where LLM decides when/what to retrieve, potentially making multiple retrieval calls with adaptive queries
- [Raptor:](raptor.md) Recursive abstractive processing creates hierarchical summaries enabling retrieval across different abstraction levels 
- [Graph RAG:](graph-rag.md) Combines knowledge graphs with vector search to capture entity relatioships and multi hop reasoning paths
- [Corrective RAG:](corrective-rag.md) Self Correcting mechanism that evaluates retrieval quality & triggers web search fallback if internal knowledge insufficient



# System & Solution Architecture

## Reference Architecture

- [Ingestion Pipeline:](ingestion-pipeline.md) Document loaders -> preprocessing/cleaning -> chunking -> embedding -> vector DB indexing (batch or streaming)
- Query Pipeline: User query -> optional transformation -> embedding -> retrieval -> reranking -> prompt construction -> LLM generation -> response
- Orchestration Layer: Framework (Langchain, LlamaIndex, Haystack) managing pipeline execution, prompt templates & component integration
- Observability Stack: Tracing(Langsmith,Phoneix), logging retrieval decision, monitoring latency/costs, evaluation datasets.
## Architectural Variants

- Naive RAG: Simple retrieve then generate; good starting point but limited accuracy(40 to 60%) typical
- [Advanced RAG:](advanced-rag-algorithms.md) Adds query transformation, hybrid search, reranking, production baseline(60 to 80% accuracy)
- Modular RAG: Pluggable components(routing, retrieval,generation) allowing specialized handling per query type
- [Multi-Index RAG:](multi-index.md) Separate indices for different data types(docs,code,tables,images) with query routing logic


## Enterprise Considerations

- [Multi-Tenancy:](multi-tenancy.md) Logical data isolation via metadata filtering or physical isolation via tenant-specific indices; impacts cost/complexity tradeoff.
- [Access Control:](access-control.md) Integration with IAM(OAuth, RBAC) enforced at retrieval time; metadata tags mapped to user permissions.
- [Data Freshness:](data-freshness.md) Delta updates vs full reindexing strategy: CDC pipelines for near real time sync from source systems
- [Federated Search:](federated-search.md) RAG across multiple siloed data sources(SharePoint, Confluence,databases) with unified ranking
- Audit & Compliance: Citation tracking, retrieval provenance logging, PII detection/redaction in chunks



# Key Tools , Platforms & Frameworks

## Orchestration Frameworks

- LangChain: Most mature ecosystem with extensive integrations; verbose API; strong community; production-ready.
- LlamaIndex: Data-centric focus with excellent ingestion utilities;query engines with built-in optimization; cleaner abstractions than LangChain
- Haystack: Deepset's framework emphasizing production readiness, pipelines-as-YAML, strong evaluation tools
- Semantic Kernel(Microsoft): Enterprise focused with .NET/C# first class support; tight Azure integration
- Amazon Bedrock KB: Managed RAG services; handles ingestion, chunking, embeddings; lock-in traderoff for operational simplicity


## Vector Databases

[reference](vector-database.md) 

- Pinecone: Fully managed, auto-scaling, excelled DX; higher costl no self-hosted option
- Weaviate: Hybrid search native, GraphQL, API,modular architecture; self-hosted or managed 
- Qdrant: Rust-based, high performance, payload filtering, growing ecosystem, self-hosted or managed 
- Milvis/Ziliz: Open-source, highly scalable (billion+ vectors) , CNCF project; complex operations
- pgvector: PostgreSQL extension; leverage existing Postgres infra; simpler for moderate scale (<10M vectors) 
- Azure AI Search: Managed hybrid search with integrated chunking. OCR,built-in security, Azure lock-in 
- Chroma: Embedded/lightweight for development; not production-scale


## Embedding Models

[reference](embedding-models.md) 

- OpenAI text-embedding-3: 1536 or 3073 dimensions; high quality; API dependency and cost 
- Cohere Embed v3: Multilingual,optimized for RAG, compression-aware; competetive quality 
- Azure OpenAI Embeddings: OpenAI models with enterprise SLA and data residency guarantees 
- BGE(BAAI): Open-source SOTA models (bge-large-en-v1.5); self-hostable on GPUs 
- E5(Microsoft): Instruction-turned embeddings; string zero shot cross domain performance 
- Voyage AI: Specialized domain embeddings(code,finance, legal);higher accuracy than general-purpose 
- Sentence Transformers: Ecosystem for fine-tuning custom domain embeddings; requires ML expertise


## Reranking Models

[reference](reranking.md) 

- Cohere Rerank: Managed cross-encoder achieving 20 to 40 % accuracy lift;simple integration 
- BGE Reranker: Open-source alternative;self-hostable; ~5-10x slower than first-stage retrieval 
- Cross-Encoder (Sentence Transformers): DIY approach; fine-tunable for domain specific ranking


# Design Pattern & Architecture Style

## Query Patterns

[reference](query-patterns.md) 

- Single Turn RAG: Stateless query-response; simplest pattern for factual Q & A 
- Conversational RAG: Maintains chat history; query rewriting uses conversation context; session state management required 
- Multi-Query RAG: Generate multiple query variations, retrieve for each, de-duplicate/merge results; improves recall 
- Decomposition: Break complex questions into sub-questions, retrieve/answer independently, synthesize final response 
- Chain-of-Thought RAG: Interleave retrieval and reasoning steps; retrieval decisions conditioned on intermediate reasoning


## Ingestion Patterns

[reference](ingestion-patterns.md) 

- Batch Ingestion: Scheduled full/incremental loads; suitable for stable datasets; simpler orchestration (Airflow, Prefect) 
- Streaming Ingestion: Real-time indexing via CDC(Debezium, Kafka); required for time-sensitive use cases; operational complexity 
- Lazy Indexing: Index on-demand when documents accessed; reduces upfront costs; adds latency to first query 
- Hierarchical Chunking: Parent-child relationships (summarizes -> sections -> paragraphs); retrieve at appropriate granularity


## Prompt Patterns

[reference](prompt-patterns.md) 

- Stuff: Insert all retrieved chunks into single prompt; simple but limited by context window 
- Map-Reduce: Summarize each chunk independently , then combine summaries; handles large contexts but loses cross-chunk reasoning 
- Refine: Iteratively update answer as new chunks processed; better systhesis but multiple LLM calls 
- Agentic: LLM decides retrieval strategy and when sufficient information obtained; most capable but least predictable


# Caching Strategies

[reference](caching-strategies.md) 

- Embedding Cache: Store computed embeddings to avoid preprocessing unchanged documents 
- Retrieval Cache: Cache query results for frequent/similar queries; semantic similarity for cache hits 
- LLM Response Cache: Exact or fuzzy matching on (query, context) pairs, GPTCache, Redis-based solutions 
- Semantic Cache: Vector similarity on query embeddings to return cached responses for semantically similar questions