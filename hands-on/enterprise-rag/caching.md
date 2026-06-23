---
aliases:
  - Caching
Source 1: https://machinelearningmastery.com/the-complete-guide-to-inference-caching-in-llms/
Source 2: https://aws.amazon.com/blogs/database/optimize-llm-response-costs-and-latency-with-effective-caching/
Source 3: https://ngrok.com/blog/prompt-caching
---
## The Complete Guide to Inference Caching in LLMs
* **Key Points:**
  - In this article, you will learn how inference caching works in large language models and how to use it to reduce cost and latency in production systems.
* **Technical Entities (Classes/Functions/APIs):** `LLMs`

## Introduction
* **Key Points:**
  - Calling a large language model API at scale is expensive and slow.
  - A significant share of that cost comes from repeated computation: the same system prompt processed from scratch on every request, and the same common queries answered as if the model has never seen them before.
  - Inference caching addresses this by storing the results of expensive LLM computations and reusing them when an equivalent request arrives.
  - Depending on which caching layer you apply, you can skip redundant attention computation mid-request, avoid reprocessing shared prompt prefixes across requests, or serve common queries from a lookup without invoking the model at all.
  - In production systems, this can significantly reduce token spend with almost no change to application logic.
* **Technical Entities (Classes/Functions/APIs):** `inference caching`

## What Is Inference Caching?
* **Key Points:**
  - Inference caching is the practice of storing the results of that computation — at various levels of granularity — and reusing them when a similar or identical request arrives.
  - There are three distinct types to understand, each operating at a different layer of the stack:
    - KV caching: Caches the internal attention states — key-value pairs — computed during a single inference request, so the model does not recompute them at every decode step. This happens automatically inside the model and is always on.
    - Prefix caching: Extends KV caching across multiple requests. When different requests share the same leading tokens, such as a system prompt, a reference document, or few-shot examples, the KV states for that shared prefix are stored and reused across all of them. You may also see this called prompt caching or context caching.
    - Semantic caching: A higher-level, application-side cache that stores complete LLM input/output pairs and retrieves them based on semantic similarity. Unlike prefix caching, which operates on attention states mid-computation, semantic caching short-circuits the model call entirely when a sufficiently similar query has been seen before.
  - These are not interchangeable alternatives. They are complementary layers.
  - KV caching is always running.
  - Prefix caching is the highest-leverage optimization you can add to most production applications.
  - Semantic caching is a further enhancement when query volume and similarity are high enough to justify it.
* **Technical Entities (Classes/Functions/APIs):** `KV caching`, `Prefix caching`, `prompt caching`, `context caching`, `Semantic caching`

## Understanding How KV Caching Works
* **Key Points:**
  - KV caching is the foundation that everything else builds on.
  - Modern LLMs use the transformer architecture with self-attention.
  - For every token in the input, the model computes three vectors: Q (Query) — What is this token looking for? K (Key) — What does this token offer to other tokens? V (Value) — What information does this token carry?
  - Attention scores are computed by comparing each token's query against the keys of all previous tokens, then using those scores to weight the values.
  - LLMs generate output autoregressively — one token at a time.
  - Without caching, generating token N would require recomputing K and V for all N-1 previous tokens from scratch.
  - During a forward pass, once the model computes the K and V vectors for a token, those values are saved in GPU memory.
  - For each subsequent decode step, the model looks up the stored K and V pairs for the existing tokens rather than recomputing them.
  - Only the newly generated token requires fresh computation.
  - It is automatic and universal; every LLM inference framework enables it by default.
  - You do not need to configure it.
* **Technical Entities (Classes/Functions/APIs):** `transformer architecture`, `self-attention`, `Q (Query)`, `K (Key)`, `V (Value)`, `GPU memory`

## Using Prefix Caching to Reuse KV States Across Requests
* **Key Points:**
  - Prefix caching — also called prompt caching or context caching depending on the provider — takes the KV caching concept one step further.
  - Instead of caching attention states only within a single request, it caches them across multiple requests — specifically for any shared prefix those requests have in common.
  - Prefix caching only works when the cached portion of the prompt is byte-for-byte identical.
  - A single character difference — a trailing space, a changed punctuation mark, or a reformatted date — invalidates the cache and forces a full recomputation.
  - Place static content first and dynamic content last.
  - System instructions, reference documents, and few-shot examples should lead every prompt.
  - Per-request variables — the user's message, a session ID, or the current date — should appear at the end.
  - Similarly, avoid non-deterministic serialization.
  - Several major API providers expose prefix caching as a first-class feature.
  - Anthropic calls it prompt caching. You opt in by adding a cache_control parameter to the content blocks you want cached.
  - OpenAI applies prefix caching automatically for prompts longer than 1024 tokens.
  - Google Gemini calls it context caching and charges for stored cache separately from inference.
  - Open-source frameworks like vLLM and SGLang support automatic prefix caching for self-hosted models, managed transparently by the inference engine without any changes to your application code.
* **Technical Entities (Classes/Functions/APIs):** `Prefix caching`, `prompt caching`, `context caching`, `cache_control`, `Anthropic`, `OpenAI`, `Google Gemini`, `vLLM`, `SGLang`

## Understanding How Semantic Caching Works
* **Key Points:**
  - Semantic caching operates at a different layer: it stores complete LLM input/output pairs and retrieves them based on meaning, not exact token matches.
  - Prefix caching makes processing a long shared system prompt cheaper on every request.
  - Semantic caching skips the model call entirely when a semantically equivalent query has already been answered, regardless of whether the exact wording matches.
  - Here is how semantic caching works in practice: A new query arrives. Compute its embedding vector. Search a vector store for cached entries whose query embeddings exceed a cosine similarity threshold. If a match is found, return the cached response directly without calling the model. If no match is found, call the LLM, store the query embedding and response in the cache, and return the result.
  - In production, you can use vector databases such as Pinecone, Weaviate, or pgvector, and apply an appropriate TTL so stale cached responses do not persist indefinitely.
  - Semantic caching adds an embedding step and a vector search to every request.
  - That overhead only pays off when your application has sufficient query volume and repeated questions such that the cache hit rate justifies the added latency and infrastructure.
  - It works best for FAQ-style applications, customer support bots, and systems where users ask the same questions in slightly different ways at high volume.
* **Technical Entities (Classes/Functions/APIs):** `Semantic caching`, `embedding vector`, `cosine similarity`, `Pinecone`, `Weaviate`, `pgvector`, `TTL`

## Choosing The Right Caching Strategy
* **Key Points:**
  - These three types operate at different layers and solve different problems.
  - The most effective production systems layer these strategies.
  - KV caching is always running underneath.
  - Add prefix caching for your system prompt — this is the highest-leverage change for most applications.
  - Layer semantic caching on top if your query patterns and volume justify the additional infrastructure.

## Conclusion
* **Key Points:**
  - Inference caching is not a single technique.
  - It is a set of complementary tools that operate at different layers of the stack:
    - KV caching runs automatically inside the model on every request, eliminating redundant attention recomputation during the decode stage.
    - Prefix caching, also called prompt caching or context caching, extends KV caching across requests so a shared system prompt or document is processed once, regardless of how many users access it.
    - Semantic caching sits at the application layer and short-circuits the model call entirely for semantically equivalent queries.
  - For most production applications, the first and highest-leverage step is enabling prefix caching for your system prompt.
  - From there, add semantic caching if your application has the query volume and user patterns to make it worthwhile.
  - As a concluding note, inference caching stands out as a practical way to improve large language model performance while reducing costs and latency.
  - Across the different caching techniques discussed, the common theme is avoiding redundant computation by storing and retrieving prior results where possible.
  - When applied thoughtfully — with attention to cache design, invalidation, and relevance — these techniques can significantly enhance system efficiency without compromising output quality.