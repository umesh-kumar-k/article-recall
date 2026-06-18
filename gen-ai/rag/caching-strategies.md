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
# Using Redis for real-time RAG goes beyond a Vector Database

## Why does RAG need real-time data?
* **Key Points:**
  - "We're seeing Retrieval Augmented Generation (RAG) become the de facto standard architecture for GenAI applications that require access to private data. Nevertheless, some may wonder why it's important to have real-time access to this data. The answer is quite simple: you don't want your application to stop running fast when you add AI to your stack."
  - "So, what is a fast application? Paul Buchheit (the creator of Gmail) coined The 100ms Rule. It says every interaction should be faster than 100ms. Why? 100ms is the threshold 'where interactions feel instantaneous.'"
  - "Let's examine what a typical RAG-based architecture looks like and what latency boundaries each component currently has as well as the expected end-to-end latency."
  - "Network round trip — assuming the end users of your app and data centers are in the US, the round trip is expected to be in the range of 20-50ms."
  - "LLM — from ChatGPT: 'As of my last update, the LLM processing time for generating a response typically ranges from tens to hundreds of milliseconds, depending on the specifics mentioned above. This processing time can be affected by the model's architecture, the length and complexity of the input text, and any additional tasks performed alongside generating the response (such as context analysis or formatting).'"
  - "GenAI app — we can expect tens of milliseconds for local operations and hundreds of milliseconds when calling third-party services."
  - "Vector database — stores the dataset or corpus you want to use to add context to the LLMs to generate accurate and relevant responses. The dataset captures semantic information about the documents (i.e. vectors) and enables efficient similarity-based retrieval during the retrieval phase. A vector search query is usually a high computation complexity operation query. In a comprehensive benchmark we conducted and will soon publish, a vector search query plus 10 documents results in a median response time of 569 milliseconds across multiple datasets and loads"
  - "Agent based architecture — may drive multiple execution cycles of components 2-4"
  - "Based on this analysis, a GenAI application built using the above architecture should expect an average of 1,513ms (or 1.5 seconds) end-to-end response time. This means you'll probably lose your end users' interest after a few interactions."
  - "To build a real-time GenAI application that allows closer to the 100ms Rule experience, you need to rethink your data architecture."
* **Technical Entities (Classes/Functions/APIs):** `RAG (Retrieval Augmented Generation)`, `GenAI`, `LLM`, `vector database`
* **Code Snippet:** None

---

## How does Redis make RAG real-time?
* **Key Points:**
  - "To deal with the above challenges, Redis offers three main datastore capabilities for AI that will enable real-time RAG: Real-time vector database"
  - "Redis supports vector data type and vector search capabilities even before the term GenAI was coined. The Redis vector search algorithm uses highly efficient in-memory data structures and a dedicated search engine, resulting in up to 50 times faster search (we will shortly release our comprehensive benchmark results) and two orders of magnitude faster retrieval of documents. It will be shown later in this blog how real-time vector search can significantly improve user experience end-to-end."
  - "Semantic cache: Traditional caching techniques in Redis (and generally) use keyword matching, which struggles to capture the semantic similarity between similar queries to LLM-based services, resulting in very low hits. Using existing caches, we don't detect the semantic similarity between 'give me suggestions for a comedy movie' and 'recommend a funny movie', leading to a cache miss. Semantic caching goes beyond exact matches: It uses clever algorithms to understand the meaning of a query. Even if the wording differs, the cache can recognize if it's contextually similar to a previous query and return the corresponding response (if it has it). According to a recent study, 31% of queries to LLM can be cached (or, in other words, 31% of the queries are contextually repeatable), which can significantly improve response time in GenAI apps running in RAG-based architectures while dramatically reducing LLM costs."
  - "You can think of semantic cache as the new caching for LLM. Utilizing vector search, semantic cache can have significant performance and deployment cost benefits, as we'll explain in the following sections."
  - "LLM Memory (or extended Conversation History): LLM Memory is the record of all previous interactions between the LLM and a specific user; think of it as the session store for LLM, except it can also record information across different user sessions. Implemented using existing Redis data structures and vector search, LLM Memory can be incredibly valuable for several reasons: Improved personalization: Understanding user preferences: by analyzing past conversations, the LLM can identify a user's preferred topics, communication style, and terminology. This allows the LLM to tailor its responses to better suit the user's needs and interests."
  - "Building rapport: referencing past discussions and acknowledging the user's history helps create a sense of continuity and rapport, which fosters a more natural and engaging user experience."
  - "Enhanced Context Awareness: Disambiguating queries: LLM Memory allows the LLM to understand the context of a current query. It can connect the user's current question with previous discussions, leading to more accurate and relevant responses. Semantic Caching can leverage LLM Memory to answer generic questions like 'What's next?'"
  - "Building on previous knowledge: the LLM can leverage past conversations to build upon existing knowledge about the user's interests or goals. This allows it to provide more comprehensive and informative responses that go beyond basic information retrieval."
  - "Example: Planning a Trip: Without LLM Memory: User: 'I'm planning a trip to Italy. What are some interesting places to visit?' LLM: 'Italy has many beautiful cities! Here are some popular tourist destinations: Rome, Florence, Venice…'"
  - "With LLM Memory: User: 'I'm planning a trip to Italy. I'm interested in art and history, not so much crowded places.' (Let's assume this is the first turn of the conversation) LLM: 'Since you're interested in art and history, how about visiting Florence? It's known for its Renaissance art and architecture.' (LLM uses conversation history to identify user preference and suggests a relevant location) User: 'That sounds great! Are there any museums I shouldn't miss?' LLM (referencing conversation history): 'The Uffizi Gallery and the Accademia Gallery are must-sees for art lovers in Florence.' (LLM leverages conversation history to understand the user's specific interests within the context of the trip)"
  - "In this example, LLM memory (or conversation history) allows the LLM to personalize its response based on the user's initial statement. It avoids generic recommendations and tailors its suggestions to the user's expressed interests, leading to a more helpful and engaging user experience."
* **Technical Entities (Classes/Functions/APIs):** `Redis`, `vector database`, `semantic cache`, `LLM Memory`, `conversation history`
* **Code Snippet:** None

---

## How does real-time RAG with Redis work?
* **Key Points:**
  - "To explain real-time RAG with Redis' capabilities for AI, nothing beats a diagram and a short explanation: Upon receiving the user's prompt, the GenAI App calls an embedding service (e.g., OpenAI Ada2) to vectorize it."
  - "GenA AppI initiates a semantic caching operation, looking for similar responses (in this case, >= 97% similarity, but other parameters can also be used). If the app hits the cache (over 30% of the time), it just has to send the cache response back to the user."
  - "Upon cache miss, the GenAI App will retrieve the historical context from Redis' LLM Memory based on the vectorized prompt."
  - "To get the top K documents that match the vectorized prompt, a similarity search is running in Redis (in this case, Redis is used as a real-time vector database)."
  - "The GenAI App generates a grounded prompt based on conversation history and documents from the Vector Database and sends it to LLM."
  - "The response is processed and then sent to the user while the Semantic Cache is updated."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `Redis`, `GenAI App`, `OpenAI Ada2`, `semantic caching`, `LLM Memory`, `Vector Database`
* **Code Snippet:** None

---

## How fast is real-time RAG with Redis?
* **Key Points:**
  - "There are two scenarios that we should look at: Semantic caching hit — in which all LLM calls are saved, as the relevant response lies in the cache (as mentioned earlier, this makes up approximately 30% of queries to GenAI)."
  - "Semantic caching misses (70% of the cases) — GenAI will trigger a similar process to the non-real-time RAG architecture, using Redis' LLM Memory, real-time vector search, and real-time document retrieval."
  - "In order to understand how real-time RAG applications perform end-to-end, let's analyze each option."
* **Technical Entities (Classes/Functions/APIs):** `semantic cache`, `Redis`, `LLM Memory`, `vector search`
* **Code Snippet:** None

---

## Semantic cache hit analysis
* **Key Points:**
  - "On receiving the vectorized prompt, GenAI App executes a vector search and a retrieval call to retrieve the cached response and send it back to the user. We assume this process is 33% shorter than the cache miss process, 20-60ms."
  - "Redis – only semantic caching is performed, corresponding to a single vector search plus a single cached response retrieval. The median latency of this operation is 40ms, and a detailed calculation can be found here."
  - "The network access remains unchanged and accounts for a latency of 20-50ms"
  - "Average end-to-end latency in case of cache hit – 100ms"
* **Technical Entities (Classes/Functions/APIs):** `GenAI App`, `Redis`, `semantic caching`
* **Code Snippet:** None

---

## Semantic cache miss analysis
* **Key Points:**
  - "Network – remains unchanged and accounts for a latency of 20-50ms"
  - "GenAI App – executes all steps detailed here in case of cache miss, still within 20-100ms"
  - "Redis – executes all of the available AI functionality of Redis: (1) Semantic Caching, (2) LLM Memory, and (3) Vector Database. We benchmarked a variety of datasets and loads scenarios and found that the median time for all combined operations (including roundtrip) is 79ms."
  - "LLM – based on historical context, we expect that LLM processing will improve by up to 25% due to a shorter prompt (fewer tokens) that is more relevant and accurate, i.e. 40-400ms"
  - "Agent-based architecture — remains unchanged and may drive multiple execution cycles of components 2-4"
  - "Average end-to-end latency – 513ms"
* **Technical Entities (Classes/Functions/APIs):** `Redis`, `Semantic Caching`, `LLM Memory`, `Vector Database`
* **Code Snippet:** None

---

## Total real-time RAG response time analysis
* **Key Points:**
  - "RAG architectures based on Redis have an average end-to-end response time of 389ms, which is around x3.2 faster than non-real-time RAG architectures and much closer to Paul Buchheit's 100ms Rule. This allows existing and new applications to run LLM components in their stack with minimal performance impact, if any."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `Redis`
* **Code Snippet:** None

---

## Additional benefits
* **Key Points:**
  - "Apart from making sure your fast applications stay fast, Redis-based real-time RAG architecture offers these other benefits: Costs — with semantic caching, you can cut up to 30% of your API calls to LLM. This can be a huge savings!"
  - "More accurate responses — LLM Memory provides context about historical conversations and helps LLM understand user preferences, build rapport, and improve responses based on previous knowledge."
* **Technical Entities (Classes/Functions/APIs):** `Redis`, `semantic caching`, `LLM Memory`
* **Code Snippet:** None

---

## Summary and next steps
* **Key Points:**
  - "The blog analyzes the response time of RAG-based architectures and explains how Redis can provide real-time end-user experiences in complex, fast-changing LLM environments. If you want to try everything discussed here, we recommend Redis Vector Library (RedisVL), a Python-based client for AI applications that uses Redis capabilities for real-time RAG (Semantic Caching, LLM Memory, and Vector Database). RedisVL works with your Redis Cloud instance or your self-deployed Redis Stack."
* **Technical Entities (Classes/Functions/APIs):** `Redis Vector Library (RedisVL)`, `Redis Cloud`, `Redis Stack`
* **Code Snippet:** None

---

## Appendix: Detailed E2E latency analysis
### The non-real-time RAG
* **Key Points:**
  - "For the non-real-time RAG, we averaged the results across all disk-based databases (special purpose and general purpose). We took the normalized median value across four different sets of tests and all the vendors under test because of the big data skew."
  - "Network round-trip: (20+50)/2 = 35ms"
  - "LLM: (50+500)/2 = 275ms"
  - "GenAI App *assuming 20ms when no other services are called and 100ms otherwise: (20+100)/2=60ms"
  - "Vector database *assuming one vector/hybrid search + 10 docs: One vector search query (we took the median value across low and high loads) – 63ms, 10x document retrieval – 10x50ms = 500ms, Total – 563ms"
  - "The agent-based architecture assumes 1/3 of the calls to LLM trigger agent processing, which triggers an application call from LLM and another iteration of data retrieval and LLM call: 33% x (LLM + App + VectorDB)"
  - "Total: 35 + {275+60+563}⅔ + {275+60+563}2*⅓ = 1232"
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `LLM`, `GenAI App`, `Vector database`
* **Code Snippet:** None

---

### The real-time RAG
* **Key Points:**
  - "For the real-time RAG, we looked at two scenarios: cache hit (using semantic caching) and cache miss. Based on this research, we calculated a weighted average assuming that 30% of queries would hit the cache (and 70% would miss it). We took the Redis median latency value across all the datasets under test in the benchmark."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `semantic caching`, `Redis`
* **Code Snippet:** None

---

### Semantic caching hit
* **Key Points:**
  - "Network round-trip: (20+50)/2 = 35ms"
  - "GenAI App *Assuming a cache hit would result in a 33% reduction in app processing time: 40ms"
  - "Redis Semantic Caching: One vector search query (we took the median value across low and high loads) – 24.6ms, 1x document retrieval – 1×0.5ms, Total – 25ms"
  - "Total: 35+40+25 = 100ms"
* **Technical Entities (Classes/Functions/APIs):** `GenAI App`, `Redis Semantic Caching`
* **Code Snippet:** None

---

### Semantic caching miss
* **Key Points:**
  - "Network round-trip: 35ms"
  - "LLM*Based on historical context, we assume that LLM processing will improve by 25% due to a shorter prompt (much fewer tokens) that is more accurate and relevant: (40+400)/2 =220ms"
  - "GenAI App *assuming 20ms when no other services are called and 100ms otherwise: (20+100)/2=60ms"
  - "Redis: Semantic cache miss – 24.6ms, LLM Memory search (24.6ms) + 5 context retrievals (5x 0.5ms ) = 27.1ms, Vector search (24.6ms) + 5 context retrievals (5x 0.5ms ) = 27.1ms, Total – 24.6+27.1+27.1 = 79ms"
  - "The agent-based architecture assumes 1/3 of the calls to LLM trigger agent processing, which triggers an application call from LLM and another iteration of data retrieval and LLM call: 33% x (LLM + App + Redis)"
  - "Total: 35 + {220+60+79}⅔ + {220+60+79}2*⅓ = 513ms"
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `GenAI App`, `Redis`, `LLM Memory`, `Vector search`
* **Code Snippet:** None

---

### End-to-end application latency
* **Key Points:**
  - "The weighted average of cache hits and misses is calculated as follows: 30% * 100ms + 70% * 513ms = 389ms"
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None


---

# Beyond Prompt Caching: 5 More Things You Should Cache in RAG Pipelines

## Why does it make sense to cache other things?
* **Key Points:**
  - "So, Prompt Caching makes sense because we expect system prompts and instructions to be passed as input to the LLM, in exactly the same format every time. But beyond this, we can also expect user queries to be repeated or look alike to some extent. Especially when talking about deploying RAG or other AI apps within an organization, we expect a large portion of the queries to be semantically similar, or even identical. Naturally, groups of users within an organization are going to be interested in similar things most of the time, like 'how many days of annual leave is an employee entitled to according to the HR policy', or 'what is the process for submitting travel expenses'. Nevertheless, statistically, it is highly unlikely that multiple users will ask the exact same query (the exact same words allowing for an exact match), unless we provide them with proposed, standardized queries within the UI of the app. Nonetheless, there is a very high chance that users ask queries with different words that are semantically very similar. Thus, it makes sense to also think of a semantic cache apart from the conventional cache."
  - "In this way, we can further distinguish between the two types of cache: Exact-Match Caching, that is, when we cache the original text or some normalized version of it. Then we hit cache only with exact, word-for-word matches of the text. Exact-match caching can be implemented using a KV cache like Redis."
  - "Semantic Caching, that is, creating an embedding of the text. Then we hit cache with any text that is semantically similar to it and exceeds a predefined similarity score threshold (like cosine similarity above ~0.95). Since we are interested in the semantics of the texts and we perform a similarity search, a vector database, such as ChromaDB, would need to be used as a cache store."
  - "Unlike Prompt Caching, where we get to use a cache integrated into the API service of the LLM, to implement caching in other stages of a RAG pipeline, we have to use an external cache store, like Redis or ChromaDB mentioned above. While this is a bit of a hassle, as we need to set up those cache stores ourselves, it also provides us with more control over the parametrization of the cache. For instance, we get to decide about our Cache Expiration policies, meaning how long a cached item remains valid and can be reused. This parameter of the cache memory is defined as Time-To-Live (TTL)."
  - "As illustrated in my previous posts, a very simple RAG pipeline looks something like this: Even in the simplest form of a RAG pipeline, we already use a caching-like mechanism without even realizing it. That is, storing the embeddings in a vector database and retrieving them from there, instead of making requests to an embedding model every time and recalculating the embeddings. This is very straightforward and essentially a non-negotiable part (it would be silly of us to not do it) even of a very simple RAG pipeline, because the embeddings of the documents generally remain the same (we need to recalculate an embedding only when a document of the knowledge base is altered), so it makes sense to calculate once and store it somewhere."
  - "But apart from storing the knowledge base embeddings in a vector database, other parts of the RAG pipeline can also be reused, and we can benefit from applying caching to them. Let's see what those are in more detail!"
* **Technical Entities (Classes/Functions/APIs):** `Prompt Caching`, `LLM`, `RAG`, `exact-match caching`, `semantic caching`, `Redis`, `ChromaDB`, `TTL (Time-To-Live)`
* **Code Snippet:** None

---

## 1. Query Embedding Cache
* **Key Points:**
  - "The first thing that is done in a RAG system when a query is submitted is that the query is transformed into an embedding vector, so that we can perform semantic search and retrieval against the knowledge base. Apparently, this step is very lightweight in comparison to calculating the embeddings of the entire knowledge base. Nonetheless, in high-traffic applications, it can still add unnecessary latency and cost, and in any case, recalculating the same embeddings for the same queries over and over again is wasteful."
  - "So, instead of computing the query embedding every time from scratch, we can first check if we have already computed the embedding for the same query before. If yes, we simply reuse the cached vector. If not, we generate the embedding once, store it in the cache, and make it available for future reuse."
  - "The most straightforward way to implement query embedding caching is by looking for the exact-match of the raw user query. For example: What area codes correspond to Athens, Greece?"
  - "Nevertheless, we can also use a normalized version of the raw user query by performing some simple operations, like making it lowercase or stripping punctuation. In this way, the following queries… What area codes correspond to athens greece? What area codes correspond to Athens, Greece what area codes correspond to Athens // Greece? … would all map to … what area codes correspond to athens greece?"
  - "We then search for this normalized query in the KV store, and if we get a cache hit, we can then directly use the embedding that is stored in the cache, with no need to make a request to the embedding model again. That is going to be an embedding looking something like this, for example: [0.12, -0.33, 0.88, ...]"
  - "In general, for the query embedding cache, the key-values have the following format: query → embedding"
  - "As you may already imagine, the hit for this can significantly improve if we propose the users with standardized queries within the app's UI, beyond letting them type their own queries in free text."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `query embedding cache`, `KV store`, `embedding model`, `standardized queries`
* **Code Snippet:** None

---

## 2. Retrieval Cache
* **Key Points:**
  - "Caching can also be utilized at the retrieval step of an RAG pipeline. This means that we can cache the retrieved results for a specific query and minimize the need to perform a full retrieval for similar queries. In this case, the key of the cache may be the raw or normalized user query, or the query embedding. The value we get back from the cache is the retrieved document chunks."
  - "So for our normalized query… what area codes correspond to athens greece? or from the query embedding… [0.12, -0.33, 0.88, ...] we would directly get back from the cache the retrieved chunks. [ chunk_12, chunk_98, chunk_42 ]"
  - "In this way, when an identical or even somewhat similar query is submitted, we already have the relevant chunks and documents in the cache — there is no need to perform the retrieval step. In other words, even for queries that are only moderately similar (for example, cosine similarity above ~0.85), the exact response may not exist in the cache, but the relevant chunks and documents needed to answer the query often do."
  - "In general, for the retrieval cache, the key-values have the following format: query → retrieved_chunks"
  - "One may wonder how this is different from the query embedding cache. After all, if the query is the same, why not directly hit the cache in the retrieval cache and also include a query embedding cache? The answer is that in practice, the query embedding cache and the retrieval cache may have different TTL policies. That is because the documents in the knowledge base may change, and even if we have the same query or the same query embedding, the corresponding chunks may be different. This explains the usefulness of the query embedding cache existing individually."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `retrieval cache`, `TTL`
* **Code Snippet:** None

---

## 3. Reranking Cache
* **Key Points:**
  - "Another way to utilize caching in the context of RAG is by caching the results of the reranker model (if we use one). More specifically, this means that instead of passing the retrieved ranked results to a reranker model and getting back the reranked results, we directly get the reranked order from the cache, for a specific query and retrieved chunks."
  - "In our Athens area codes example, for our normalized query: what area codes correspond to athens greece? and hypothetical retrieved and ranked chunks [ chunk_12, chunk_98, chunk_42 ] we could directly get the reranked chunks as output of the cache: [chunk_98, chunk_12, chunk_42]"
  - "In general, for the reranking cache, the keys and values have the following format: (query + retrieved_chunks) → reranked_chunks"
  - "Again, one may wonder: if we hit the reranking cache, shouldn't we also always hit the retrieval cache? At first glance, this might seem true, but in practice, it is not necessarily the case. One reason is that, as explained already, different caches may have different TTL policies. Even if the reranking result is still cached, the retrieval cache may have already expired and require performing the retrieval step from scratch."
  - "But beyond this, in a complex RAG system, we most probably are going to use more than one retrieval mechanism (e.g., semantic search, BM25, etc.). As a result, we may hit the retrieval cache for one of the retrieval mechanisms, but not for all, and thus not hit the cache for reranking. Vice versa, we may hit the cache for reranking, but miss on the individual caches of the various retrieval mechanisms — we may end up with the same set of documents, but by retrieving different documents from each individual retrieval mechanism. For these reasons, the retrieval and reranking caches are conceptually and practically different."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `reranking cache`, `reranker model`, `TTL`, `semantic search`, `BM25`
* **Code Snippet:** None

---

## 4. Prompt Assembly Cache
* **Key Points:**
  - "Another useful place to apply caching in a RAG pipeline is during the prompt assembly stage. That is, once retrieval and reranking are completed, the relevant chunks are combined with the system prompt and the user query to form the final prompt that is sent as input to the LLM. So, if the query, system prompt, and reranked chunks all match, then we hit cache. This means that we don't need to reconstruct the final prompt again, but we can get parts of it (the context) or even the entire final prompt directly from cache."
  - "Continuing with our Athens example, suppose the user submits the query… what area codes correspond to athens greece? and after retrieval and reranking, we get the following chunks (either from the reranker or the reranking cache): [chunk_98, chunk_12, chunk_42]"
  - "During the prompt assembly step, these chunks are combined with the system prompt and the user query to construct the final prompt that will be sent to the LLM. For example, the assembled prompt may look something like: System: You are a helpful assistant that answers questions using the provided context. Context: [chunk_98] [chunk_12] [chunk_42] User: what area codes correspond to athens greece?"
  - "In general, for the prompt assembly cache, the key values have the following format: (query + system_prompt + retrieved_chunks) → assembled_prompt"
  - "Apparently, the computational savings here are smaller compared to the other caching layers mentioned above. Nonetheless, context caching can still reduce latency and simplify prompt construction in high-traffic systems. In particular, prompt assembly caching makes sense to implement in systems where prompt assembly is complex and includes more operations than a simple concatenation, like inserting guardrails."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `prompt assembly cache`, `system prompt`, `guardrails`
* **Code Snippet:** None

---

## 5. Query – Response Caching
* **Key Points:**
  - "Last but not least, we can cache pairs of entire queries and responses. Intuitively, when we talk about caching, the first thing that comes to mind is caching query and response pairs. And this would be the ultimate jackpot for our RAG pipeline, as in this case, we don't need to run any of it, and we can just provide a response to the user's query solely by using the cache."
  - "More specifically, in this case, we store entire query — final response pairs in the cache, and completely avoid any retrieval (in case of RAG) and re-generation of a response. In this way, instead of retrieving relevant chunks and generating a response from scratch, we directly get a precomputed response, which was generated at some earlier time for the same or a similar query."
  - "To safely implement query-response caching, we either have to use exact matches in the form of a key-value cache or use semantic caching with a very strict threshold (like 0.99 cosine similarity between user query and cached query)."
  - "Continuing with our Athens example, suppose a user asks the query: what area codes correspond to athens greece? Assume that earlier, the system already processed this query through the full RAG pipeline, retrieving relevant chunks, reranking them, assembling the prompt, and generating the final answer with the LLM. The generated response might look something like: The main telephone area code for Athens, Greece is 21. Numbers in the Athens metropolitan area typically start with the prefix 210, followed by the local subscriber number."
  - "The next time an identical or extremely similar query appears, the system does not need to run the retrieval, reranking, or generation steps again. Instead, it can immediately return the cached response."
  - "In general, for the query-response cache, the key values have the following format: query → final_response"
* **Technical Entities (Classes/Functions/APIs):** `query-response caching`, `RAG`, `semantic caching`, `cosine similarity`
* **Code Snippet:** None

---

## On my mind
* **Key Points:**
  - "Apart from Prompt Caching directly provided in the API services of the various LLMs, several other caching mechanisms can be utilized in an RAG application to achieve cost and latency savings. More specifically, we can utilize caching mechanisms in the form of query embeddings cache, retrieval cache, reranking cache, prompt assembly cache, and query response cache. In practice, in a real-world RAG, many or all of these cache stores can be used in combination to provide improved performance in terms of cost and time as the users of the app scale."
* **Technical Entities (Classes/Functions/APIs):** `Prompt Caching`, `RAG`, `query embeddings cache`, `retrieval cache`, `reranking cache`, `prompt assembly cache`, `query response cache`
* **Code Snippet:** None


