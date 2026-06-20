---
aliases:
  - Hybrid Storage Architecture
Source 1: https://hatidata.com/blog/hybrid-sql-vector-search
---
# Hybrid SQL + Vector Search: Why Agents Need Both


## The Limitations of Pure Vector Search
* **Key Points:**
  - Vector databases have become the default tool for AI retrieval. Embed your documents, store them in a vector index, and query by semantic similarity. It works well for straightforward question-answering: "What is our refund policy?" finds the refund policy document regardless of exact wording.
  - But production agents need more than semantic similarity. Consider these real requirements:
    - "Find customer interactions from the last 7 days where sentiment was negative" — This requires a date filter and a metadata filter in addition to semantic matching.
    - "Show me the top 10 highest-revenue accounts that have mentioned integration issues" — This requires joining semantic search results with structured business data and sorting by a numeric column.
    - "Count how many times agents accessed customer data in the EMEA namespace this month" — This is a pure SQL aggregation with no semantic component at all.
    - "Find memories similar to 'pricing concerns' but only for enterprise-tier customers with ARR above 100K" — This combines semantic search with multiple structured filters.
  - Pure vector search handles none of these well. You can add metadata filters to vector queries, but the filter capabilities are limited — no JOINs, no aggregations, no subqueries, no window functions, no CTEs. The moment your retrieval needs move beyond "find similar documents," you are fighting against the limitations of the vector database query language.

## The HatiData Approach: Two Engines, One Interface
* **Key Points:**
  - HatiData runs two search engines in parallel — a columnar query engine for structured queries and a vector index for vector search — unified through a single SQL interface. Every memory stored in HatiData exists in both systems: the structured data (content, metadata, timestamps, namespace, agent ID) lives in the query engine, and the vector embedding lives in the vector index. The two records are linked by a shared memory_id UUID.
  - This dual-storage architecture means that agents can:
    - Use pure SQL when the query is entirely structured
    - Use pure vector search when the query is entirely semantic
    - Combine both in a single query when the requirements are hybrid
  - The key insight is that most real agent queries are hybrid. They need semantic understanding to match by meaning and structured filtering to narrow by time, namespace, metadata, or business logic. HatiData makes hybrid the default, not an afterthought.
* **Technical Entities (Classes/Functions/APIs):** `HatiData`, `memory_id UUID`

## SQL Functions for Semantic Search
* **Key Points:**
  - HatiData extends its SQL dialect with two custom functions that bridge the gap between structured and semantic queries.
  - The semantic_match function returns a boolean — true if the content in the specified column is semantically similar to the query above the given threshold. It works as a WHERE clause filter.
  - Behind the scenes, semantic_match embeds the query string, performs an approximate nearest neighbor lookup in the vector index to get candidate memory IDs, and returns those IDs to the query engine as a filter. The query optimizer integrates this seamlessly with the other WHERE conditions.
  - The semantic_rank function returns a float between 0 and 1 representing the cosine similarity between the column content and the query. It works as a scoring function that can be used in ORDER BY, SELECT, or HAVING clauses.
* **Technical Entities (Classes/Functions/APIs):** `semantic_match()`, `semantic_rank()`
* **Code Snippet:**
```sql
SELECT memory_id, content, created_at, metadata
FROM agent_memories
WHERE namespace = 'customer-support'
  AND semantic_match(content, 'billing dispute or payment issue', 0.75)
  AND created_at > '2026-02-01'
ORDER BY created_at DESC
LIMIT 20;
```

## Hybrid Query Patterns
* **Key Points:**
  - Pattern 1: Semantic Filter with SQL Aggregation - Find the most active topics in customer support for the past month, using semantic matching to categorize conversations.
  - This is impossible with a pure vector database — it combines semantic categorization with SQL aggregation, date filtering, and JSON metadata extraction.
  - Pattern 2: Join Semantic Results with Business Data - Find enterprise customers whose recent interactions suggest churn risk.
  - This query joins the agent's memory store with a structured customers table, filtering by business criteria (enterprise tier, ARR threshold) and semantic criteria (churn signals). A pure vector search could find churn-related content but could not join it with business data or filter by revenue.
  - Pattern 3: Memory Analytics Dashboard - Build an analytics view of agent memory usage across namespaces.
  - This is a pure SQL query that does not use vector search at all — it is analyzing the memory store as structured data. The ability to run analytics queries against the same store that handles semantic search is a key advantage of HatiData's architecture.
* **Technical Entities (Classes/Functions/APIs):** `semantic_match()`, `semantic_rank()`
* **Code Snippet:**
```sql
SELECT
    c.customer_name,
    c.arr,
    c.tier,
    m.content AS recent_interaction,
    semantic_rank(m.content, 'considering alternatives or planning to leave') AS churn_signal
FROM agent_memories m
JOIN customers c ON m.metadata->>'customer_id' = c.id
WHERE m.namespace = 'account-management'
  AND c.tier = 'enterprise'
  AND c.arr > 50000
  AND semantic_rank(m.content, 'considering alternatives or planning to leave') > 0.65
  AND m.created_at > CURRENT_DATE - INTERVAL '30 days'
ORDER BY churn_signal DESC
LIMIT 10;
```

## The Memory-ID Join Pattern
* **Key Points:**
  - The bridge between the query engine and the vector index is the memory_id UUID. Every memory has the same UUID in both systems. When HatiData processes a hybrid query:
    - The semantic components (semantic_match, semantic_rank) are extracted from the SQL
    - The query text is embedded using the embedding service
    - The vector index performs approximate nearest neighbor search and returns matching memory IDs with similarity scores
    - These IDs are passed back to the query engine as a temporary table or IN clause
    - The query engine executes the full query, joining the vector search results with the structured data
  - This pipeline is transparent to the agent — it writes standard SQL and gets back standard results. The two-engine orchestration happens inside HatiData's query pipeline, between the SQL parsing step and the execution step.
  - For agents that only need vector search, the search_memory MCP tool provides a simpler interface. But for agents that need the full power of SQL combined with semantic understanding, the hybrid SQL functions are the way to go.
* **Technical Entities (Classes/Functions/APIs):** `memory_id UUID`, `search_memory MCP tool`

## Graceful Degradation
* **Key Points:**
  - If the vector index is unavailable (network issues, maintenance, etc.), HatiData falls back to SQL-only search. The semantic_match function returns false for all rows (no matches), and semantic_rank returns 0.0. Queries that do not use semantic functions continue to work normally.
  - This graceful degradation ensures that agents are never completely blocked by a vector search outage. They lose semantic capabilities but retain full SQL functionality. The fallback is logged and surfaced in the dashboard, so operators know when hybrid search is running in degraded mode.

## Next Steps
* **Key Points:**
  - Hybrid search is most powerful when combined with HatiData's memory management features — namespace isolation keeps different agents' memories separate, access controls ensure that only authorized agents can search sensitive namespaces, and the embedding pipeline handles vector computation asynchronously. See the LangChain hybrid search cookbook for a complete implementation example.