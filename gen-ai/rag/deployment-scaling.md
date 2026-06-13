---
aliases:
  - Deployment and Scaling
tags:
  - rag
  - rag-scaling
Source 1: https://apxml.com/courses/optimizing-rag-for-production/chapter-1-production-rag-foundations/rag-performance-bottlenecks
Source 2: https://medium.com/@bijit211987/designing-high-performing-rag-systems-464260b76815
Source 3: https://medium.com/@chinmayd49/rag-production-optimizations-and-trade-offs-a623e5834e65
Source 4: https://openlayer.com/blog/post/what-are-embedding-models-complete-guide
Source 5: https://gse.kz/en/blog/storage-for-rag-storage-tiers-datasets-logs
Source 6: https://www.datacamp.com/tutorial/prompt-compression
Source 7: https://apxml.com/courses/large-scale-distributed-rag/chapter-3-optimizing-llms-distributed-rag/multi-llm-rag-architectures-routing
---
- Startup/Product Perspective
	- **Managed Services First**: Pinecone + OpenAI embeddings + GPT-4 minimizes operational overhead; optimize costs later
	- **Rapid Iteration**: Start with native RAG, measure baseline, incrementally add complexity (reranking, hybrid search) based on eval metrics
	- **Serverless-First**: Lambda/Cloud Functions for orchestration; event-driven ingestion; scale-to-zero cost benefits
	- **Vendor Lock-in acceptable**: Speed-to-market prioritized; architecture allows component swapping once PMF achieved
	- **Observability via Saas**: Langsmith, traceloop for debuggin; deferred investment in custom observability

- Enterprise Perspective 
	- **Hybrid/Self-Hosted**: On-prem vector DB(Qdrant, Milvus) for data residency; Azure OpenAI for compliance (HIPAA,FedRAMP)
	- **Multi-Region Deployment**: Vector DB replication across regions; embedding model locality to reduce latency; CDN for static assets
	- **Auto-Scaling Config**: Kubernetes HPA for orchestration services; vector DB read replicas for query load; separate read/write paths
	- **Data Governance**: Metadata taxonomy design; automated PII scanning in ingestion; audit logs for retrieval per-user
	- **Disaster Recovery**: Vector index backups; point-in-time recovery; blue-green deployments for index updates
	- **Cost Attribution**: Chargeback models per business unit; usage metering for embeddings/LLM callls; reserved capacity planning

- Infrastructure Patterns 
	- **Compute**: GPU nodes (A10,T4) for self hosted embeddings/reranking; CPI for orchestration/APIs
	- **Queue-Based Ingestion**: SQS/RabbitMQ buffering document processing; autoscaling workers;DLQ for failures
	- **API Gateway**: Rate limiting,auth,request routing to RAG backend; WAF for injection protection
	- **CDN/Edge**: Cache embedding results , static propmt templates; edge functions for low-latency simple queries
	- **Monitoring**: Promethues/Grafana for infrasturcture; custom metrics for RAG-specific KPIs (retrieval hit rate, answer accuracy)

- Scaling Bottlenecks
	- Vector Search Latency: Sharding Indices, HNSW paramter tuning (ef_construction,M), GPU acceleration (cuVS).
	- Embedding Throughput: Batch embedding calls, async processing , model quantization(ONNX,TensorRT)
	- LLM API Limits: Rate limiting, request batching, fallback to multiple providers(GPT-4 -> Claude -> Llama)
	- Ingestion Througput: Parallel document processing, chunking pipeline optimization, incremental updates over full reindex

- Cost Optimization
	- Embedding Model Tradeoff: Smaller models (384d) reduce storage 50% with 5 to 10% accuracy loss; validate per use case
	- Tiered Storage: Hot(frequently accessed) in-memory /SSD, warm (archived) on cheaper storage with rehydration latency
	- Prompt Compression: LLMLingua-style compression reduces context tokens 50 to 70%; risks information loss
	- Cascade Models: Route simple queries to cheaper models (GPT-4o-mini), complex to expensive (GPT-4); 30 to 50% cost reduction
	- Reserved Capacity: Commit to baseline usage for 3o to 50% discounts(Azure, AWS) combine with spot/pre-emptible for spikes