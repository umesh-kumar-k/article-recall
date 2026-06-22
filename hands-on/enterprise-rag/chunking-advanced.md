---
aliases:
  - Chunking
Source 1: https://levelup.gitconnected.com/chunking-strategies-in-rag-systems-insights-from-80-genai-interviews-8ceb4a17701a
Source 2: https://weaviate.io/blog/chunking-strategies-for-rag
Source 3: https://unstructured.io/blog/level-up-your-genai-apps-essential-data-preprocessing-for-any-rag-system
Source 4: https://medium.com/@anuragmishra_27746/five-levels-of-chunking-strategies-in-rag-notes-from-gregs-video-7b735895694d
Source 5: https://docs.langchain.com/oss/javascript/integrations/splitters
---
[Code Examples](./code/document-splitters.md) 

# Chunking Strategies in RAG Systems: Insights from 80+ GenAI Interviews

## The Day Everything Changed
* **Key Points:**
  - That moment stuck with me though because he wasn't an outlier. I've sat across from 80-something GenAI engineering candidates now and that blank look? I've seen it way more than I expected. Smart people. Experienced people. People who absolutely should have known this stuff.
  - But nobody actually taught them. There's no "RAG Systems in Production" elective. No "Chunking Strategies guide — 101". You either stumble into the right project at the right company and figure it out, or you're googling at midnight before your next interview hoping a blog post saves you.
  - So I kept notes. Started noticing patterns which questions made people freeze, which ones made something click behind their eyes. Somewhere around candidate #47, I had enough material to actually be useful to someone.

## The Chunking Nightmare (That Shouldn't Be a Nightmare)
* **Key Points:**
  - Here's the truth that nobody tells you: chunking is actually the hardest part of RAG. Not the fancy LLM stuff. Not the vector databases. Chunking.
  - And yet, when I ask candidates about it, I get: "Just fixed-size chunks" (50% of candidates); "Uhhh… semantic?" (30% of candidates, with zero follow-up); "What's a chunk?" (20% of candidates, which is at least honest)

## When Planning Your Chunking Strategy
* **Key Points:**
  - Here's the thing about chunking that took me years to articulate: you need to know your constraints before you pick a strategy.
  - I'd say this to a candidate, and they'd nod. Then I'd ask, "So what's your LLM's context window?" and they'd pause. Because they hadn't thought about it.
  - "Every AI engineer should be able to answer these seven questions": (Fig 1)
  - Let me segregate the above engineering questions into three important constraints such as Model constraints, Business Constraints and Evaluation Readiness. (Fig 2, Fig 3)

## Candidate #34 — "The Thoughtful One"
* **Key Points:**
  - This one walked in and didn't wait for me to finish the question. "Before I pick a chunking strategy, I need to know a few things first." He counted them off: What's the LLM's context window? How does the embedding model actually perform on this content? What does the document structure look like? What accuracy are we targeting? What's the latency budget? Can we even afford the embeddings at scale? "Then I'd pick a strategy."
  - There's no universal answer on chunking. "Chunking isn't a setting you configure, it's a decision you make based on real constraints". And most people never get that far.

## The Chunking Methods Bracket (A Real Tournament)
* **Key Points:**
  - I started tracking which strategies candidates mentioned, and honestly? The distribution was enlightening. (Fig 4)
  - What shocked me most? About 40% of candidates had never heard of "recursive chunking." They either didn't know it existed, or they thought it was something scary. It's not scary. It's literally: "Try to split by paragraphs, then sentences, then words if you have to."
  - I started explaining it like this: "Imagine you're cutting up a pizza. You want to cut between slices. You don't want to cut mid-slice. If you're forced to cut a slice, you cut it between toppings, not through the cheese." Suddenly everyone got it.

## Candidate #67 — "Technically Sounds Right"
* **Key Points:**
  - "For most use cases, 512–1024 tokens is the sweet spot. Here's why: Too small (256 tokens): You lose context. A sentence might be incomplete. Too large (2048 tokens): The chunk becomes a small document. You retrieve noise. Sweet spot (512–1024): Detailed enough to be coherent, small enough to be focused." Then he added: "I'd test on my actual data, though. It depends on the domain."
  - That's the answer. Not because it's technically perfect, but because they understood the principle and knew when to deviate.
  - Here is the point I can share some principles of chunk size by domain.
  - And the thing nobody tells you: overlap is often more important than size.

## Candidate #73 — "The Detail-Oriented One"
* **Key Points:**
  - "512-token chunks, 50-token overlap," they said. "Each adjacent chunk shares about 10% of its content."
  - "Why overlap at all?" "Because if I split at a boundary, I lose the thread of what came before. Overlap keeps the context alive."
  - Keep this optimization loop in the back of your mind. (Fig 7)

## The Pitfalls That Get Everyone
* **Key Points:**
  - After 80 interviews, I've seen the same mistakes over and over. Let me save you from them.

### Pitfall #1: Chunks Too Small
* **Key Points:**
  - The intention is simple — never split mid-thought. Here's what 512 tokens looks like in practice. (Fig 9)
* **Technical Entities (Classes/Functions/APIs):** (Fig 8, Fig 9)

### Pitfall #2: Chunks Too Large
* **Key Points:**
  - The problem: Query: "What's the CEO's name?" Document: Entire Wikipedia article (5000 tokens). Retrieved chunk: Entire 5000 tokens. LLM is drowning in context. 99% of information is irrelevant. LLM attention gets diluted. Hallucination risk: MEDIUM-HIGH.
  - The fix? Start at 512–1024 tokens and adjust. Focus your context always. "The fix is simple I would say like you can start at 512–1024 tokens. Give your LLM room to think, not room to drown." (Fig 10)

### Pitfall #3: No Overlap (The Context Bleeding Crisis)
* **Key Points:**
  - Document: "Authentication is critical. It prevents unauthorized access. Your system must support OAuth. This is the standard. Most applications use it." Chunks (no overlap): Chunk 1: "... OAuth." Chunk 2: "This is the standard..." Problem at boundary: "This is the standard" — standard of WHAT? Without overlap, the second chunk is ambiguous. Result: Lost context.
  - "The fix? A 20–50 token overlap. Let your chunks share a little context so your LLM never has to guess." (Fig 11)

### Pitfall #4: Ignoring Document Structure
* **Key Points:**
  - "Try chunking a PDF the same way you chunk markdown." Uncomfortable laugh. "The fix? Before you write a single line of chunking code, I would say know your format. Your strategy lives or dies there." (Fig 12, Fig 13)

### Pitfall #5: Not Evaluating Retrieval Quality
* **Key Points:**
  - "I deployed my RAG system to production." "Great! What are your metrics?" "… metrics?" This happens way too often. People build RAG systems and have no idea if they actually work.
  - The basics: The fix? Build evaluation into your system from day 1. (Fig 14)

## The Optimization Secrets (That Aren't Secret)
* **Key Points:**
  - By interview #70, I noticed the best candidates were doing three things: Measuring iteratively; Optimizing based on data; Being willing to change strategy.

### Secret #1: Hybrid Retrieval
* **Key Points:**
  - "I'd use vector search for semantic matching, BM25 for keywords, then merge the results." "Because neither is perfect alone. Vector search misses rare terms. BM25 misses semantic synonyms. Together = best of both." Result: +15–20% better retrieval quality. (Fig 15)
* **Technical Entities (Classes/Functions/APIs):** `vector search`, `BM25`

### Secret #2: Re-ranking
* **Key Points:**
  - "I'd retrieve 20 candidates with vector search, then re-rank with a cross-encoder." "Because vector similarity is approximate. A cross-encoder is more accurate. Trade: retrieve more, refine with better ranking." Result: +20-25% accuracy with minimal latency overhead.
* **Technical Entities (Classes/Functions/APIs):** `cross-encoder`, `re-ranking`

### Secret #3: Context Compression
* **Key Points:**
  - "I'd compress retrieved context before sending to the LLM." "Extract only the sentences relevant to the query. Drop the rest." Result: 80% fewer tokens, 100% relevant context. Benefit: 40% faster LLM inference, 40% cheaper.
  - Let me wrap all the above three optimizations "secrets" into one visual diagram. (Fig 16)

## The Questions That Separated Good From Great
* **Key Points:**
  - By interview #80, I had a shortlist. Three questions that more than anything else predicted who would actually succeed at RAG in production.

### The "Constraint" Question
* **Key Points:**
  - "Before you design a chunking strategy, what do you need to know about the system?" Good Answer: "The LLM's context window and the embedding model's limits." Great Answer: "I need to know the LLM's context window, my embedding model's performance, my document structure, my accuracy targets, my latency budget, and whether I can afford semantic chunking."

### The "Evaluation" Question
* **Key Points:**
  - "How would you know if your chunking strategy is working?" Good Answer: "Measure precision and recall." Great Answer: "I'd measure retrieval metrics (precision@K, recall@K, NDCG), generation metrics (faithfulness, hallucination rate), and end-to-end metrics (task success rate). I'd test on my own data before deploying."

### The "Pitfall" Question
* **Key Points:**
  - "What are the most common chunking mistakes you've seen or made?" Good Answer: "Chunks that are too small lose context." Great Answer: "The common mistakes are: chunks too small (incomplete thoughts), chunks too large (noisy context), no overlap (context bleeding), ignoring document structure, and not evaluating quality. I'd address each with specific techniques."

## The Moment Everything Clicked
* **Key Points:**
  - This candidate came in, and when I asked about chunking, they laid out a framework that actually made sense. They said: "I'd start with recursive chunking (512 tokens, 50-token overlap) because it's fast and respects structure. Then I'd measure precision, recall, and faithfulness on my test data. If accuracy is insufficient, I'd try semantic chunking or switch to hybrid retrieval. I'd re-rank if precision drops. I'd compress context if latency becomes an issue. I'd iterate based on metrics."
  - That's when I realized: Most candidates don't fail because they don't know chunking. They fail because they don't understand that chunking is a systems problem, not a theoretical problem. It's not about being "right." It's about being thoughtful, measurable, and iterative.

## The Checklist (Your Interview Survival Guide)
* **Key Points:**
  - If you're interviewing for a GenAI role soon, here's what you need to know:
  - Before You Chunk: Do you know your LLM's context window? (8K, 128K, 200K?); Do you know your embedding model's context limit? (Usually 256–512); Have you looked at your documents? (PDF? Markdown? Database?); What accuracy do you need? (Is 70% acceptable? 90%?); How fast does it need to be? (<2s? <10s? Batch is fine?); Can you afford semantic chunking? (Embedding API calls cost money); Do you have test data with ground truth? (You should)
  - Choosing a Strategy: MVP? Use recursive chunking (fast, balanced); Production? Still recursive (unless you have budget for semantic); High accuracy? Use hybrid recursive-semantic; Complex domain? Use domain-specific + metadata; Very large docs? Use hierarchical chunking
  - Implementing: Chunk size: 512–1024 tokens (adjust per domain); Overlap: 20–50 tokens (at least 10%); Document structure: Respect it (use recursive); Metadata: Preserve it (source, date, author); Deduplication: Remove exact duplicates
  - Optimizing: Measure precision@5, recall@5, NDCG; Measure faithfulness, hallucination rate; Measure task success rate; Use hybrid retrieval (dense + sparse); Add re-ranking if precision matters; Compress context if latency matters; Iterate based on metrics
  - Avoiding Pitfalls: Don't use tiny chunks (<256 tokens); Don't use huge chunks (>2048 tokens); Don't skip overlap (context bleeding is real); Don't ignore document structure (it matters); Don't skip evaluation (you need metrics); Don't assume your first strategy is optimal; Don't forget metadata (it's useful for filtering)