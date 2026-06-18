---
aliases:
  - Query Patterns
highlights: |-
  Single Turn RAG: Stateless query-response; simplest pattern for factual Q & A

  Conversational RAG: Maintains chat history; query rewriting uses conversation context; session state management required 

  Multi-Query RAG: Generate multiple query variations, retrieve for each, de-duplicate/merge results; improves recall

  Decomposition: Break complex questions into sub-questions, retrieve/answer independently, synthesize final response

  Chain-of-Thought RAG: Interleave retrieval and reasoning steps; retrieval decisions conditioned on intermediate reasoning
tags:
  - rag
  - rag-query
Source 1: https://medium.com/@nikita04/how-retrieval-augmented-generation-rag-and-chain-of-thought-cot-create-89b4c1c97e04
Source 2: https://geekyants.com/blog/teaching-your-rag-system-to-think-a-guide-to-chain-of-thought-retrieval
Source 3: https://redis.io/blog/10-techniques-to-improve-rag-accuracy/
---


# 10 techniques to improve RAG accuracy

## Key takeaways
* **Key Points:**
  - "Retrieval augmented generation (RAG) improves AI accuracy by grounding LLM outputs in verified, domain-specific context rather than relying on static training data alone."
  - "Start simple: baseline a naive RAG pipeline, then measure and iterate with clear metrics."
  - "Strengthen retrieval: use hybrid search to bridge keyword and vector gaps, tune HNSW indices, and optimize chunking."
  - "Specialize models: fine-tune vector embeddings for domain language and LLMs for tone, format, or compliance."
  - "Stabilize answers: apply semantic caching for FAQs and manage long-term memory for multi-turn interactions."
  - "Improve fidelity: use query transforms to clarify vague inputs, add an LLM as judge to evaluate faithfulness, and re-rank noisy results."
* **Technical Entities (Classes/Functions/APIs):** `RAG (Retrieval Augmented Generation)`, `LLMs`, `HNSW`, `RAGAS`
* **Code Snippet:** None

---

## How RAG improves the accuracy of AI responses
* **Key Points:**
  - "You've probably seen this before: your LLM confidently answers a question about your company's refund policy—except the policy changed six months ago, and the model has no idea. LLMs have a hard knowledge cutoff. Their parametric knowledge is frozen the moment training ends, and they don't know what they don't know. Modern LLMs extend beyond this through tool use, web browsing, and retrieval systems, but those are external additions, not inherent model capabilities. Without them, even a capable model will struggle with recent or domain-specific information."
  - "Retrieval augmented generation (RAG) is the most widely adopted approach to closing that gap. It searches a knowledge base for relevant documents before generating a response, grounding outputs in your actual data instead of relying on the model's training alone."
  - "But basic RAG only gets you so far. Poor chunking, weak retrieval, and untuned models all chip away at accuracy in ways that aren't immediately obvious. The fix is systematic optimization: baseline a naive pipeline, define quantitative metrics using frameworks like Retrieval Augmented Generation Assessment (RAGAS), and iterate by fixing one weak link at a time."
  - "This guide covers the 10 RAG accuracy techniques that consistently deliver the largest gains, along with when and why to apply each one."
* **Technical Entities (Classes/Functions/APIs):** `LLMs`, `RAG (Retrieval Augmented Generation)`, `RAGAS (Retrieval Augmented Generation Assessment)`
* **Code Snippet:** None

---

## 1. Hybrid search
* **Key Points:**
  - "The most common RAG accuracy problem is retrieval failure: the right document exists in your corpus, but the system doesn't surface it. Semantic vector search captures conceptual similarity well, but it can miss documents where exact terminology matters, like part numbers, regulatory codes, proper nouns, and domain-specific jargon. Pure keyword matching has the opposite problem, missing conceptually related content that uses different vocabulary than the query."
  - "Hybrid search runs both approaches in parallel and fuses the results. Combining BM25 keyword matching with vector similarity means your system captures both exact-match precision and semantic recall simultaneously. For mixed or structured corpora like legal documents, technical manuals, and medical records, this is often the single biggest accuracy lever available."
  - "Redis Query Engine supports hybrid queries natively through the FT.HYBRID command introduced in Redis 8.4, combining BM25 and vector similarity from a single query interface without external post-processing. Score fusion happens inside the query engine using Reciprocal Rank Fusion (RRF) or linear combination methods, preserving consistent normalization across both retrieval signals. Research on hybrid RAG systems, including Blended RAG and HyPA-RAG, found that combining keyword and vector searches improved retrieval recall by 3 to 3.5 times and raised end-to-end answer accuracy by 11 to 15 percent on complex reasoning tasks. To get started, see the hybrid search tutorial."
* **Technical Entities (Classes/Functions/APIs):** `hybrid search`, `BM25`, `vector similarity`, `Redis Query Engine`, `FT.HYBRID`, `Redis 8.4`, `Reciprocal Rank Fusion (RRF)`, `Blended RAG`, `HyPA-RAG`
* **Code Snippet:** None

---

## 2. Tuning HNSW indices for retrieval precision
* **Key Points:**
  - "Vector search accuracy depends on more than just the embedding model. Hierarchical Navigable Small World (HNSW) indices organize vectors into a multi-layered graph for fast approximate nearest-neighbor search, but the default parameters aren't optimized for every workload."
  - "Three parameters govern the accuracy-performance tradeoff: M (maximum connections per node). Higher M creates denser graphs with more connections, improving recall and retrieval consistency, especially when your corpus contains similar but subtly distinct entries like FAQ variants or near-duplicate documents."
  - "EF_CONSTRUCTION (search depth during index building). Controls how thoroughly the algorithm explores the graph while inserting new vectors. Higher values produce a better-connected graph at the cost of longer index build times."
  - "EF_RUNTIME (search depth during queries). Controls how many candidate neighbors the algorithm examines per query. Higher values improve recall at the cost of slightly higher latency."
  - "Redis' published benchmarks suggest you may have headroom to raise EF_RUNTIME values, and therefore improve recall, without violating latency SLAs, depending on your workload and K. In a billion-scale benchmark, Redis reported 90% precision at 200ms median latency when retrieving the top 100 nearest neighbors under 50 concurrent queries. That performance budget may let you trade a fraction of raw speed for denser graphs that deliver higher retrieval precision in production. More on HNSW algorithms."
* **Technical Entities (Classes/Functions/APIs):** `HNSW (Hierarchical Navigable Small World)`, `M`, `EF_CONSTRUCTION`, `EF_RUNTIME`, `Redis`
* **Code Snippet:** None

---

## 3. Chunking & parsing optimization
* **Key Points:**
  - "How you split documents before indexing them determines what context your LLM actually receives. Fixed-length chunking, splitting at uniform token counts regardless of content structure, is the default for most pipelines, but it has a predictable failure mode: it breaks coherent ideas across chunk boundaries, separating information that should appear together and leaving the model to work with fragments."
  - "Better chunking strategies respect semantic boundaries. Splitting at paragraph breaks, section headers, or points of topical discontinuity (identified through embedding similarity between consecutive sentences) keeps related information together. Each retrieved chunk should represent a complete thought rather than half of one. Moving from fixed-length to adaptive chunking can meaningfully improve both retrieval precision and recall because the model receives more coherent context and makes better use of it."
  - "Redis integrates cleanly with chunking-aware frameworks like LangChain and LlamaIndex, making it straightforward to iterate across chunking strategies without rebuilding your pipeline. See the LangChain RAG notebook for a practical starting point."
* **Technical Entities (Classes/Functions/APIs):** `LangChain`, `LlamaIndex`, `Redis`
* **Code Snippet:** None

---

## 4. Fine-tuning embeddings for domain-specific accuracy
* **Key Points:**
  - "Generic embedding models trained on broad internet-scale text learn general semantic relationships. In specialized domains, those relationships don't always hold. A generic model might score 'cardiac arrhythmia' and 'irregular heartbeat' as moderately similar when a cardiologist would treat them as equivalent, or fail to distinguish between legal concepts that differ only in subtle doctrinal detail."
  - "Fine-tuning vector embeddings on domain-specific labeled pairs, using contrastive learning or MultipleNegativesRankingLoss, teaches the model the semantic relationships that matter in your corpus. Even with modest training data, fine-tuned compact models often outperform much larger generic models on domain-specific retrieval tasks, which also reduces inference overhead."
  - "Redis supports storing and querying multiple embedding models in parallel, so you can run a fine-tuned domain model alongside your existing generic model for A/B testing without rebuilding your index from scratch. More details in our blog on fine-tuning embeddings for RAG."
* **Technical Entities (Classes/Functions/APIs):** `contrastive learning`, `MultipleNegativesRankingLoss`, `Redis`
* **Code Snippet:** None

---

## 5. Fine-tuning the LLM
* **Key Points:**
  - "Retrieval quality determines what information reaches the model, but the model's own weights determine what it does with that information. If your RAG system consistently gets retrieval right but fumbles the response (wrong tone, inconsistent citation format, safety disclaimers that don't match your compliance requirements), the problem is in generation, not retrieval."
  - "LLM fine-tuning addresses this by training the model on examples that demonstrate exactly the behavior you need: how to cite sources, what format responses should follow, how to handle edge cases, and what language to use for your domain. Parameter-efficient techniques like Low-Rank Adaptation (LoRA) make this practical without retraining billion-parameter models from scratch. You adapt the model's behavior without destroying its broader knowledge. This is especially valuable in high-stakes domains like healthcare or finance, where response format and compliance language need to be consistent across every interaction, not just statistically likely."
* **Technical Entities (Classes/Functions/APIs):** `LLM fine-tuning`, `Low-Rank Adaptation (LoRA)`
* **Code Snippet:** None

---

## 6. Semantic caching for consistency & cost
* **Key Points:**
  - "Semantic caching improves RAG accuracy in an often-overlooked way: by making responses consistent. When two users ask semantically identical questions ('How do I reset my password?' and 'I forgot my login credentials'), a system that regenerates responses independently introduces variability. Cached responses are consistent—semantically similar queries that hit the same cache entry get the same answer, which matters when accuracy depends on consistency."
  - "Redis LangCache (preview) implements semantic caching by converting queries to vector embeddings and comparing them against previously cached query embeddings using a similarity threshold. When a new query is semantically close enough to a cached one, the system returns the cached response rather than invoking the LLM again. This is particularly valuable for stable knowledge bases (FAQs, product docs, internal policy queries) where high-confidence answers exist and should be served reliably. In conversational workloads with optimized configurations, Redis LangCache has achieved up to 73% cost reduction, and cached responses return in milliseconds rather than seconds. See the Redis LangCache docs for setup and API details."
* **Technical Entities (Classes/Functions/APIs):** `semantic caching`, `Redis LangCache`
* **Code Snippet:** None

---

## 7. Long-term memory management for multi-turn accuracy
* **Key Points:**
  - "Single-turn RAG is straightforward: retrieve relevant documents, augment the prompt, generate a response. Multi-turn interactions are harder. Without explicit memory management, each query starts from scratch. The model loses context from prior exchanges, can't build on established preferences, and forces users to re-establish context they've already provided."
  - "Long-term memory solves this by storing interaction history as vector embeddings and retrieving semantically relevant past exchanges as context for new queries. Rather than concatenating an ever-growing conversation transcript (which bloats context windows and triggers the 'lost-in-the-middle' accuracy problem), semantic memory retrieval surfaces only the prior exchanges relevant to the current question. This keeps context windows lean and focused, which generally improves model reasoning quality. The langgraph-checkpoint-redis package provides both thread-level session persistence through RedisSaver and cross-thread long-term memory through RedisStore with vector search. More in our blog on building agent memory with Redis and LangGraph."
* **Technical Entities (Classes/Functions/APIs):** `long-term memory`, `langgraph-checkpoint-redis`, `RedisSaver`, `RedisStore`, `LangGraph`
* **Code Snippet:** None

---

## 8. Query transforms to improve retrieval coverage
* **Key Points:**
  - "Retrieval quality depends on how well the query matches the document corpus, and users rarely phrase questions the way technical documents are written. A user asking 'What's broken with my internet?' won't naturally match troubleshooting docs written in terms of 'network connectivity diagnostics' or 'ISP service verification.' The semantic distance between user intent and document language degrades retrieval before generation even starts."
  - "Query transformation techniques close this gap. Hypothetical Document Embeddings (HyDE) generates a synthetic answer to the user's query, then uses that synthetic document as the retrieval query instead of the original question. Because the hypothetical answer is written in the language of answers, not questions, it tends to match actual documents more closely. Multi-query generation creates several reformulations of the original query in parallel, then retrieves using all variants to maximize recall. Both techniques add inference overhead, so they're best applied adaptively: trigger query transformation when initial retrieval returns weak results, skip it when standard retrieval is already surfacing strong candidates."
* **Technical Entities (Classes/Functions/APIs):** `Query transformation`, `Hypothetical Document Embeddings (HyDE)`, `Multi-query generation`
* **Code Snippet:** None

---

## 9. LLM as judge for faithfulness evaluation
* **Key Points:**
  - "Measuring RAG accuracy requires evaluating two things independently: whether retrieval surfaced the right documents, and whether generation stayed faithful to those documents. Human evaluation is the gold standard but doesn't scale. Automated metrics like BLEU or ROUGE measure surface-level similarity, not faithfulness. LLM-as-judge fills the gap by using a capable model to score responses against defined criteria at scale."
  - "A faithfulness judge evaluates whether the generated response is grounded in the retrieved context or invents unsupported details. A relevance judge assesses whether the response actually addresses the user's question. A context precision judge evaluates whether the retrieved documents appear in the right rank order. GPT-4 evaluators match human annotator agreement levels—over 80% on pairwise comparisons—though agreement can drop in specialized domains like medicine or law. Frameworks like RAGAS operationalize this approach and integrate directly with Redis-backed pipelines."
* **Technical Entities (Classes/Functions/APIs):** `LLM-as-judge`, `GPT-4`, `RAGAS`, `BLEU`, `ROUGE`
* **Code Snippet:** None

---

## 10. Re-ranking for precision refinement
* **Key Points:**
  - "Initial vector or hybrid retrieval is optimized for recall: getting potentially relevant documents into the candidate set. But the ranking of those candidates often leaves something to be desired. Documents ranked 8th or 9th in initial retrieval may be more relevant than those ranked 2nd or 3rd, and LLMs tend to pay more attention to context that appears early in their input."
  - "Re-ranking addresses this with a second-pass model that scores query-document pairs jointly rather than independently. Cross-encoder models process query and document tokens together, allowing for richer relevance assessment than the bi-encoder approach used during initial retrieval. The computational cost is higher per document, but because re-ranking operates on a small candidate set (typically top 20 to 100 from initial retrieval) rather than the full corpus, it's practical in production. Mature implementations report 10 to 40 percent precision improvements depending on baseline quality and domain complexity. See our blog on fine-tuning rerankers for better retrieval."
* **Technical Entities (Classes/Functions/APIs):** `Re-ranking`, `Cross-encoder models`, `bi-encoder`
* **Code Snippet:** None

---

## The path to more accurate RAG
* **Key Points:**
  - "These 10 techniques are a proven toolkit for improving RAG accuracy, but the order matters. Start with retrieval fundamentals (hybrid search, chunking, HNSW tuning) before moving to model customization (embedding fine-tuning, LLM fine-tuning). Accuracy measurement through LLM-as-judge should run throughout, not just at the end. Advanced patterns like agentic RAG orchestration, metadata conditioning, and prompt tuning build on top of a solid foundation, not a substitute for one."
  - "Redis provides the unified real-time infrastructure to run this entire stack: vector search and hybrid retrieval through Redis Query Engine, semantic caching through Redis LangCache (preview), agent memory through Redis' native data structures and vector indexing, and integrations with 30+ AI frameworks including LangChain, LangGraph, and LlamaIndex. You don't need a separate vector database, a separate cache, and a separate session store. Redis delivers sub-millisecond access for caching and session operations, and low-latency vector retrieval at scale, simplifying the pipeline and reducing the operational surface area you're debugging when accuracy degrades."
  - "The full set of RAG and GenAI resources in the Redis developer repository covers each technique with working code examples. Try Redis free to see how the platform performs against your workload, or talk to our team about building production-grade RAG pipelines that actually stay accurate at scale."
* **Technical Entities (Classes/Functions/APIs):** `Redis Query Engine`, `Redis LangCache`, `LangChain`, `LangGraph`, `LlamaIndex`
* **Code Snippet:** None

---

## Frequently asked questions about RAG
### What is the recommended order for implementing RAG optimization techniques to get the biggest accuracy improvements first?
* **Key Points:**
  - "Start by establishing baseline metrics using frameworks like RAGAS, then prioritize retrieval quality first: hybrid search, chunking optimization, and HNSW tuning. Even the best LLM cannot compensate for missing or irrelevant source documents. Once retrieval is reliable, move to model specialization like fine-tuning embeddings and adapting the LLM for domain-specific needs."
  - "Run LLM-as-judge evaluation continuously throughout to catch regressions early. Layer on advanced techniques like query transformation, re-ranking, semantic caching, and long-term memory only after foundational elements are stable, as these amplify existing quality rather than compensating for structural weaknesses."
* **Technical Entities (Classes/Functions/APIs):** `RAGAS`, `hybrid search`, `HNSW`, `LLM-as-judge`, `query transformation`, `re-ranking`, `semantic caching`, `long-term memory`
* **Code Snippet:** None

---

### How does hybrid search combining BM25 and vector similarity compare to using either approach alone in terms of retrieval recall and answer accuracy?
* **Key Points:**
  - "BM25 excels at exact terminology and rare tokens but fails on synonyms, while vector search captures semantic meaning but struggles with homonyms and precision-critical terms. Hybrid search fuses both via methods like Reciprocal Rank Fusion. Microsoft's Azure AI Search benchmarks found that hybrid retrieval improved NDCG scores by roughly 10-20% over single-mode approaches across customer and academic datasets, with further gains when combined with semantic reranking. Exact improvements vary by dataset and configuration."
  - "This dual-mode approach is especially valuable in enterprise settings where documents mix structured metadata needing exact matching with unstructured content needing semantic understanding, letting systems handle diverse query types without requiring users to reformulate their questions."
* **Technical Entities (Classes/Functions/APIs):** `BM25`, `vector search`, `hybrid search`, `Reciprocal Rank Fusion`, `Azure AI Search`, `NDCG`
* **Code Snippet:** None

---

### How should I approach tuning HNSW index parameters (M, EF_CONSTRUCTION, EF_RUNTIME) for my RAG pipeline?
* **Key Points:**
  - "The three parameters control different tradeoffs. EF_RUNTIME is the easiest to adjust since it affects query-time recall without requiring reindexing; raise it incrementally until you hit your latency ceiling. M affects graph connectivity and memory consumption; higher values improve recall but increase storage requirements. EF_CONSTRUCTION mainly affects index build quality and time."
  - "The right values depend on your corpus size, query patterns, and latency requirements. Start with your vector database's recommended defaults, then tune incrementally while measuring retrieval recall at your target K values against your actual corpus and query distribution rather than synthetic benchmarks."
* **Technical Entities (Classes/Functions/APIs):** `HNSW`, `M`, `EF_CONSTRUCTION`, `EF_RUNTIME`
* **Code Snippet:** None

---

### How do query transformation techniques like HyDE and multi-query generation work, and when should they be triggered versus skipped to avoid unnecessary inference overhead?
* **Key Points:**
  - "HyDE prompts an LLM to generate a hypothetical answer matching your corpus style, then searches using that synthetic document's embedding instead of the raw query, effectively comparing document-to-document rather than question-to-document. Multi-query generation creates parallel reformulations of the original question and merges retrieval results across all variants to maximize coverage. Both require additional LLM inference calls before retrieval begins."
  - "Use adaptive triggering: skip transformation when initial retrieval returns high-confidence results, and apply it only when results are weak or below a confidence cutoff. Simple factual lookups rarely need transformation, while ambiguous or exploratory queries benefit most. Apply selectively to balance accuracy against latency costs."
* **Technical Entities (Classes/Functions/APIs):** `HyDE (Hypothetical Document Embeddings)`, `Multi-query generation`, `LLM`
* **Code Snippet:** None

---

### How can I measure RAG pipeline accuracy using the LLM-as-judge approach, and what faithfulness score thresholds indicate production readiness?
* **Key Points:**
  - "Construct evaluation prompts that score responses on faithfulness, relevance, and context precision using structured JSON output with numeric scores and reasoning chains. Target faithfulness above 0.85 on a normalized scale for general use, though regulated domains like finance and healthcare may require 0.90+, while customer support may tolerate 0.80 with human escalation."
  - "Validate your judge against human annotations for the first 200–500 responses, use temperature zero for consistency, and monitor score distributions over time, not just averages. Be aware that LLM judges can exhibit systematic biases such as preferring longer responses and showing positional bias, so calibration against human-verified datasets is essential. Sample 5–10 percent of production traffic through the judge pipeline daily to catch silent accuracy degradation from corpus changes and usage drift."
* **Technical Entities (Classes/Functions/APIs):** `LLM-as-judge`, `faithfulness`, `relevance`, `context precision`
* **Code Snippet:** None


---


# Teaching Your RAG System to Think: A Guide to Chain of Thought Retrieval


## The Problem with Vanilla RAG
* **Key Points:**
  - "You have built a RAG system. It works great for simple questions, but then someone asks: How does Anthropic's approach to AI safety differ from OpenAI's? What are the implications for the industry?"
  - "In such a case, your system retrieves a few chunks, generates a response, and...it's shallow. It missed half the question. It didn't connect the dots."
  - "This is the fundamental limitation of single-shot retrieval. Complex questions require reasoning—breaking problems down, retrieving iteratively, and synthesizing across multiple sources. They require your RAG system to think."
  - "Enter Chain of Thought (CoT) for RAG."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `Chain of Thought (CoT)`
* **Code Snippet:** None

---

## What is Chain of Thought Retrieval?
* **Key Points:**
  - "Chain of Thought prompting, introduced by Google researchers in 2022, showed that language models perform dramatically better on complex tasks when they 'show their work'—reasoning step by step rather than jumping to answers."
  - "The insight for RAG systems: don't just retrieve once and generate. Reason about what you need, retrieve it, reason about what's still missing, retrieve again, and synthesize."
  - "Instead of: Query → Retrieve → Generate"
  - "We get: Query → Think → Retrieve → Think → Retrieve → ... → Synthesize"
  - "This simple shift unlocks multi-hop reasoning, self-correction, and dramatically better answers on complex questions."
* **Technical Entities (Classes/Functions/APIs):** `Chain of Thought (CoT)`, `RAG`
* **Code Snippet:** None

---

## 1. Query Decomposition: Plan First, Execute in Parallel
* **Key Points:**
  - "The simplest approach: break the question into sub-questions upfront, retrieve for each (in parallel), then synthesize."
  - "When to use it: Predictable queries where you can anticipate the sub-questions. Great for comparison questions, multi-part requests, and research tasks."
  - "Trade-offs: Fast (parallel retrieval) but inflexible. If your decomposition is wrong, you can't adapt mid-flight."
* **Technical Entities (Classes/Functions/APIs):** `Query Decomposition`, `parallel_retrieve()`, `decomposition_rag()`
* **Code Snippet:**
```python
def decomposition_rag(query: str):
    # Step 1: Break it down
    sub_queries = llm.generate(f"""
        Break this into 2-4 independent sub-questions:
        {query}
    """)
    
    # Step 2: Retrieve in parallel
    results = parallel_retrieve(sub_queries)
    
    # Step 3: Synthesize
    return llm.generate(f"""
        Original question: {query}
        Research: {results}
        Synthesize a complete answer.
    """)
```

---

## 2. ReAct: Reasoning and Acting in a Loop
* **Key Points:**
  - "ReAct (Reasoning + Acting) interleaves thinking with action. The model reasons about what to do, takes an action (like searching), observes the result, and repeats."
  - "The pattern: Thought → Action → Observation → Thought → Action → Observation → ... → Answer"
  - "When to use it: Complex, multi-hop questions where you can't predict what information you'll need. Great when adaptability matters more than speed."
  - "Trade-offs: Highly adaptive and interpretable, but higher latency due to sequential LLM calls. Can also 'over-search' if not carefully constrained."
* **Technical Entities (Classes/Functions/APIs):** `ReAct (Reasoning + Acting)`, `retriever.search()`, `parse_response()`, `react_rag()`
* **Code Snippet:**
```python
def react_rag(query: str, max_steps: int = 5):
    messages = [{"role": "user", "content": f"""
        Answer this question: {query}
        
        Think step by step. Format each step as:
        Thought: <reasoning about what to do>
        Action: search[<query>] OR answer[<final answer>]
    """}]
    
    for _ in range(max_steps):
        response = llm.generate(messages)
        thought, action = parse_response(response)
        
        if action.startswith("answer"):
            return extract_answer(action)
        
        # Execute search and add observation
        results = retriever.search(extract_query(action))
        messages.append({"role": "assistant", "content": response})
        messages.append({"role": "user", "content": f"Observation: {results}"})
    
    return "Could not find sufficient information"
```

---

## 3. Self-Ask: Explicit Intermediate Questions
* **Key Points:**
  - "Similar to 'Reasoning and Acting' (ReAct), but the model explicitly asks and answers intermediate questions. The structure is more rigid but often easier to implement."
  - "The pattern: Question: [complex query], Are follow-up questions needed? Yes., Follow-up: [intermediate question 1], Intermediate answer: [answer after retrieval], Follow-up: [intermediate question 2], Intermediate answer: [answer after retrieval], Final answer: [synthesized response]"
  - "When to use it: Factoid chains where each answer feeds the next question. Particularly good for temporal reasoning and entity resolution."
* **Technical Entities (Classes/Functions/APIs):** `Self-Ask`, `Follow-up`, `Intermediate answer`
* **Code Snippet:** None

---

## 4. Chain-of-Verification (CoVe): Trust but Verify
* **Key Points:**
  - "A different philosophy: generate an answer first, then verify it. This catches hallucinations and improves factual accuracy."
  - "The pattern: Draft Answer → Generate Verification Questions → Retrieve Evidence → Check Claims → Revise"
  - "When to use it: High-stakes applications where accuracy matters more than speed. Legal research, medical information, and financial analysis."
  - "Trade-offs: Highest accuracy but also highest latency. Multiple retrieval and generation rounds."
* **Technical Entities (Classes/Functions/APIs):** `Chain-of-Verification (CoVe)`, `retriever.search()`, `cove_rag()`
* **Code Snippet:**
```python
def cove_rag(query: str):
    # Generate initial answer
    initial_docs = retriever.search(query)
    draft = llm.generate(query, context=initial_docs)
    
    # What should we verify?
    verification_questions = llm.generate(f"""
        Given this answer: {draft}
        What specific facts should be verified?
    """)
    
    # Verify each claim
    evidence = {}
    for question in verification_questions:
        evidence[question] = retriever.search(question)
    
    # Revise if needed
    return llm.generate(f"""
        Original answer: {draft}
        Verification evidence: {evidence}
        Revise any incorrect claims.
    """)
```

---

## 5. FLARE: Retrieve Only When Uncertain
* **Key Points:**
  - "Forward-Looking Active Retrieval (FLARE) is elegant: generate the answer incrementally, but only retrieve when the model's confidence drops."
  - "The insight: Most sentences don't need retrieval. Only fetch when the model is uncertain."
  - "The pattern: generate next sentence, check confidence, retrieve if below threshold, continue"
  - "When to use it: Long-form generation where most content is straightforward but some claims need grounding."
  - "Trade-offs: Efficient (fewer retrievals) but requires confidence estimation, which adds implementation complexity."
* **Technical Entities (Classes/Functions/APIs):** `FLARE (Forward-Looking Active Retrieval)`, `generate_with_confidence()`, `retriever.search()`, `flare_rag()`
* **Code Snippet:**
```python
def flare_rag(query: str):
    answer = ""
    
    while not complete(answer):
        # Generate next sentence with confidence
        next_part, confidence = llm.generate_with_confidence(
            query, partial=answer
        )
        
        if confidence < THRESHOLD:
            # Low confidence → retrieve first
            docs = retriever.search(next_part)
            next_part = llm.generate(query, partial=answer, context=docs)
        
        answer += next_part
    
    return answer
```

---

## 6. Tree of Thoughts: Explore Multiple Paths
* **Key Points:**
  - "For truly ambiguous questions, a single reasoning path may not be enough. Tree of Thoughts explores multiple approaches and selects the best."
  - "The pattern: Generate 3 approaches → Pursue each with retrieval → Evaluate → Select best"
  - "When to use it: Ambiguous or open-ended questions where multiple interpretations are valid."
  - "Trade-offs: Highest quality for complex questions, but expensive (3x+ the compute)."
* **Technical Entities (Classes/Functions/APIs):** `Tree of Thoughts`
* **Code Snippet:** None

---

## 7. Step-Back Prompting: Zoom Out First
* **Key Points:**
  - "Sometimes you need context before specifics. Step-back prompting asks a more general question first."
  - "The pattern: Original Question → Abstract to General Question → Retrieve General Context → Retrieve Specifics → Combine"
  - "When to use it: Conceptual questions that benefit from a broader context. 'Why' questions often work well with this approach"
* **Technical Entities (Classes/Functions/APIs):** `Step-Back Prompting`
* **Code Snippet:** None

---

## Choosing the Right Approach
* **Key Points:**
  - "If your query is... Predictable, parallelizable → Use Query Decomposition"
  - "Complex, multi-hop → Use ReAct"
  - "A chain of dependent facts → Use Self-Ask"
  - "High-stakes, accuracy-critical → Use Chain-of-Verification"
  - "Long-form with occasional facts → Use FLARE"
  - "Ambiguous, multiple valid angles → Use Tree of Thoughts"
  - "Conceptual, needs context → Use Step-Back"
  - "In practice, you'll likely combine approaches. Start simple (decomposition), add ReAct for complex queries, and layer in verification for critical applications."
* **Technical Entities (Classes/Functions/APIs):** `Query Decomposition`, `ReAct`, `Self-Ask`, `Chain-of-Verification`, `FLARE`, `Tree of Thoughts`, `Step-Back`
* **Code Snippet:** None

---

## Implementation Tips
### 1. Set iteration limits.
* **Key Points:**
  - "ReAct and similar patterns can loop forever. Cap at 5-7 iterations."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### 2. Design your action space carefully.
* **Key Points:**
  - "Keep it minimal: search[query] - semantic search, lookup[term] - exact match, answer[response] - terminate"
* **Technical Entities (Classes/Functions/APIs):** `search[query]`, `lookup[term]`, `answer[response]`
* **Code Snippet:** None

---

### 3. Format observations well.
* **Key Points:**
  - "Include source attribution so the model can reason about source quality:"
* **Technical Entities (Classes/Functions/APIs):** `format_results()`
* **Code Snippet:**
```python
def format_results(results):
    return "\n".join([
        f"[Source {i+1} - {r.metadata['source']}]: {r.text[:500]}"
        for i, r in enumerate(results[:3])
    ])
```

---

### 4. Log everything.
* **Key Points:**
  - "The reasoning trace is gold for debugging. Store thoughts, actions, and observations."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### 5. Handle failures gracefully.
* **Key Points:**
  - "When retrieval returns nothing: if not results: observation = 'No results found. Try a different search angle.'"
  - "This guides the model to adapt rather than hallucinate."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## The Architecture
* **Key Points:**
  - "A production CoT-RAG system has distinct layers: Layer: Orchestration Layer — Description: Controls flow, manages state"
  - "Reasoning Layer (LLM) — Thinks, plans, synthesizes"
  - "Action Layer — Search, lookup, calculate"
  - "Retrieval Layer — Vector DB, hybrid search"
  - "Data Layer — Documents, embeddings"
  - "The orchestration layer is key. It parses LLM outputs, routes to actions, manages conversation state, and enforces termination conditions."
* **Technical Entities (Classes/Functions/APIs):** `CoT-RAG`, `Orchestration Layer`, `Reasoning Layer (LLM)`, `Action Layer`, `Retrieval Layer`, `Vector DB`, `hybrid search`, `Data Layer`
* **Code Snippet:** None

---

## Evaluation Matters
* **Key Points:**
  - "How do you know if your CoT-RAG system is working? Track these metrics: Retrieval quality: Are the searches returning relevant documents? How many retrievals to reach a good answer?"
  - "Reasoning quality: Do the thoughts logically connect? Is the model actually using the retrieved information?"
  - "Answer quality: Is the final answer grounded in the observations? Does it address all parts of the question?"
  - "Build evaluation sets with complex, multi-hop questions. Compare single-shot RAG against your CoT approach. The differences will be stark."
* **Technical Entities (Classes/Functions/APIs):** `CoT-RAG`, `RAG`
* **Code Snippet:** None

---

## Conclusion
* **Key Points:**
  - "Standard RAG is powerful but brittle. It assumes one retrieval is enough, that you know what to search for upfront, and that the answer exists in a single chunk."
  - "Chain of Thought retrieval breaks these assumptions. It lets your system reason about what it needs, adapt when initial retrievals fall short, and synthesize across multiple sources."
  - "The techniques range from simple (query decomposition) to sophisticated (tree of thoughts). Start simple, measure what breaks, and add complexity where needed."
  - "The goal isn't to implement every technique. It's to build a system that thinks through problems the way a skilled researcher would: methodically, adaptively, and thoroughly."
  - "Your RAG system shouldn't just retrieve. It should be reasonable."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `Chain of Thought (CoT)`, `query decomposition`, `tree of thoughts`
* **Code Snippet:** None