---
aliases:
  - Best Practices & Trade Offs
---
- Chunking Trade-offs
	- **Small Chunks** (128-256 tokens): Higher retrieval precision, risk missing context, more chunks to rerank, better for specific facts
	- **Large Chunks** (512-1024 tokens): More context preserved, lower precision, fewer LLM calls, better for conceptual questions
	- **Sliding Window**: Overlapping chunks capture cross-boundary context; increase index size 2-3x but improves recall
	- **Semantic Chunking**: Boundary detection using embedding similarity; higher quality but computationally expensive

- Retrieval Configuration
	- **Top-K Selection (5-20)**: More chunks improve recall  but add noise and cost; diminishing returns beyond 10-15 for most tasks 
	- **Similarity Threshold**: Filter low-relevance results; prevents irrelevant context poisoning but risks no results for edge queries
	- **MMR (Maximal Marginal Relevance)**: Diversify retrieved chunks to reduce redundancy; useful when documents repeat

- LLM Selection
	- **GPT-4/Claude-3.5**: Best reasoning over complex contenxts; higher cost(10 to 30 $ 1M tokens);acceptable latency for interactive use
	- **GPT-4-o-mini/Claude-3.5-Haiku**: 10x cheaper, 2-3x faster; sufficient for straightforward RAG; good cost-performance balance
	- **Llama-3.1/Mistral**: Self-hostable alternatives; 70B+ models approach GPT-3.5 quality; GPU infrastructure and expertise required
	- **Long-Context Models(100k-200k tokens)**: Reduce retrieval necessity but don't eliminate need ; "lost in the middle" degradation still present

- Production Considerations
	- 