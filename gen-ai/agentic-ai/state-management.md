---
aliases:
  - State Management
highlights: Persistence layer(Redis, Dynamo DB, Postgres) storing conversation history, agent memory , and task progress
Source 1: https://www.tigerdata.com/learn/building-ai-agents-with-persistent-memory-a-unified-database-approach
Source 2: https://redis.io/blog/ai-agent-memory-stateful-systems/
Source 3: https://redis.io/blog/build-smarter-ai-agents-manage-short-term-and-long-term-memory-with-redis/
---
# Building AI Agents with Persistent Memory: A Unified Database Approach

* **Key Points:**
  - The underlying language models in AI agents are stateless at the API level. They respond to prompts and immediately forget everything. This is a major limitation if you're building autonomous production systems that need to learn, remember, and evolve.
  - When it comes to implementing agent memory, the fragmented approach of using separate databases for vectors, relational data, and time-series creates operational complexity, consistency problems, and infrastructure costs that scale faster than your data.
  - This guide teaches you how to consolidate AI agent persistent memory into a single PostgreSQL database using Tiger Data, combining hypertables for time-series conversation history, pgvectorscale for semantic search, and standard PostgreSQL for structured state—eliminating the multi-database complexity that plagues most agent architectures.
* **Technical Entities (Classes/Functions/APIs):** `PostgreSQL`, `Tiger Data`, `hypertables`, `pgvectorscale`, `API`
* **Code Snippet:** None.

---

## What is AI Agent Persistent Memory Architecture?

* **Key Points:**
  - AI agent persistent memory architecture is a unified database system that stores episodic events, semantic knowledge, and procedural state in PostgreSQL, enabling agents to maintain context across conversations while scaling to production workloads. Instead of managing separate vector databases, time-series stores, and relational systems, production-grade agents now consolidate all three memory types using PostgreSQL extensions: hypertables partition conversation history by time, pgvector indexes enable semantic search over embedded knowledge, and standard tables store user preferences with ACID guarantees.
* **Technical Entities (Classes/Functions/APIs):** `PostgreSQL`, `hypertables`, `pgvector`, `ACID`
* **Code Snippet:** None.

---

## The Three Types of Agent Memory

* **Key Points:**
  - Production AI agents require three distinct cognitive memory systems working together.
  - Episodic memory captures timestamped events—every conversation turn, tool invocation, and API result becomes a queryable timeline. When users ask "What did we discuss yesterday?", agents query episodic memory. This demands time-series optimization because range queries like "last 50 messages" are constant, not exceptional.
  - Semantic memory represents embedded knowledge stored as vector embeddings. When users upload PDFs or agents scrape documentation, content gets chunked, embedded, and indexed for similarity search. This powers retrieval-augmented generation (RAG), letting agents fetch relevant context before generating responses. Production systems require vector similarity search with sub-50ms latency even at millions of embeddings.
  - Procedural memory stores structured preferences and learned behaviors in relational tables. When agents remember "this user prefers Python" or "always format code as TypeScript", that's procedural memory requiring transactional updates with foreign key constraints.
* **Technical Entities (Classes/Functions/APIs):** `API`, `RAG`, `vector embeddings`
* **Code Snippet:** None.

---

## Why Use a Unified Database for AI Agent Memory?

* **Key Points:**
  - Unified PostgreSQL architecture reduces infrastructure costs by as much as 66% while enabling faster context retrieval through single-query joins across episodic, semantic, and procedural memory—impossible when data spans separate vector databases, time-series stores, and relational systems. A single database connection constructs complete context windows in one transaction with consistent point-in-time snapshots. One backup strategy protects all memory. One query language joins temporal, vector, and relational data without network round trips between systems.
* **Technical Entities (Classes/Functions/APIs):** `PostgreSQL`
* **Code Snippet:** None.

---

## Schema Design: Conversation Store (Episodic Memory)

* **Key Points:**
  - The create_hypertable call partitions the table into 7-day chunks. When querying recent messages, PostgreSQL scans only relevant chunks. As data ages, Tiger Data compresses old chunks at 10:1 ratios, keeping storage manageable without sacrificing query performance.
* **Technical Entities (Classes/Functions/APIs):** `create_hypertable`, `Tiger Data`
* **Code Snippet:**
```sql
-- Episodic memory: timestamped conversation history
CREATE TABLE messages (
  id BIGSERIAL,
  conversation_id UUID NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('user', 'assistant', 'system', 'tool')),
  content TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  
  -- Performance and cost tracking
  model_name TEXT,
  tokens_used INTEGER,
  latency_ms INTEGER,
  metadata JSONB,
  
  PRIMARY KEY (conversation_id, created_at, id)
);

-- Convert to hypertable for time-series optimization
SELECT create_hypertable(
  'messages',
  by_range('created_at', INTERVAL '7 days')
);

-- Index for fast conversation retrieval
CREATE INDEX idx_messages_conversation_time 
  ON messages (conversation_id, created_at DESC);
```

---

## Schema Design: Knowledge Base (Semantic Memory)

* **Key Points:**
  - The diskann index uses pgvectorscale's implementation of Microsoft's DiskANN algorithm, optimized for SSD storage with lower RAM requirements than HNSW. pgvectorscale achieves 471 QPS at 99% recall on 50M vectors—competitive with specialized vector databases while staying in PostgreSQL.
  - Temporal validity columns (valid_from, valid_until) enable time-aware RAG. Pure vector similarity returns semantically similar but potentially outdated information. Temporal windows mark old knowledge as invalid without deletion, supporting queries like "What was the architecture recommendation in Q3 2024?"
* **Technical Entities (Classes/Functions/APIs):** `pgvectorscale`, `DiskANN`, `HNSW`, `RAG`
* **Code Snippet:**
```sql
-- Semantic memory: embedded knowledge with temporal validity
CREATE TABLE knowledge_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  content TEXT NOT NULL,
  source_url TEXT,
  embedding vector(1536),  -- OpenAI ada-002 dimension
  
  -- Temporal validity tracking for time-aware RAG
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  valid_from TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  valid_until TIMESTAMPTZ,
  
  -- Categorization for filtered search
  tags TEXT[],
  category TEXT,
  metadata JSONB
);

-- StreamingDiskANN index for production vector search
CREATE INDEX idx_knowledge_embedding ON knowledge_items 
  USING diskann (embedding);

-- GIN index for hybrid search (vector + keyword)
CREATE INDEX idx_knowledge_content_fts ON knowledge_items 
  USING GIN (to_tsvector('english', content));

-- Temporal filtering for time-aware RAG
CREATE INDEX idx_knowledge_temporal 
  ON knowledge_items (valid_from, valid_until, category);
```

---

## Schema design: Agent State (Procedural Memory)

* **Key Points:**
  - Relational structure supports transactional updates like "update user preferences and default agent atomically". JSONB columns provide schema flexibility while foreign keys ensure data integrity.
* **Technical Entities (Classes/Functions/APIs):** `JSONB`
* **Code Snippet:**
```sql
-- Procedural memory: user preferences and agent configuration
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  preferences JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE agents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  system_prompt TEXT NOT NULL,
  learned_strategies JSONB DEFAULT '{}',
  avg_response_time_ms INTEGER,
  total_conversations INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE user_agents (
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
  is_default BOOLEAN DEFAULT FALSE,
  PRIMARY KEY (user_id, agent_id)
);
```

---

## How to Implement Hybrid RAG with Time Filtering?

* **Key Points:**
  - Hybrid RAG combines vector similarity search with temporal validity filtering and full-text keyword matching, returning only knowledge that is both semantically relevant and currently valid—preventing agents from retrieving outdated information despite high embedding similarity. This pattern queries episodic, semantic, and procedural memory in one database round trip.
  - This query uses the DiskANN index for vector search (<-> operator), GIN index for full-text (@@ operator), and B-tree indexes for temporal filtering—all in one PostgreSQL query. The valid_until column prevents outdated knowledge retrieval: facts remain in the database but queries exclude them after invalidation.
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `PostgreSQL`, `DiskANN`, `GIN`
* **Code Snippet:**
```typescript
import { Pool } from 'pg';

const pool = new Pool({ connectionString: process.env.DATABASE_URL });

interface SearchOptions {
  query: string;
  embedding: number[];
  limit?: number;
  maxAgeDays?: number;
  category?: string;
  semanticWeight?: number; // Weight for vector similarity (0-1)
  textWeight?: number;     // Weight for text search (0-1)
}

async function hybridSearchWithTemporalValidity(options: SearchOptions) {
  const { 
    query, 
    embedding, 
    limit = 10, 
    maxAgeDays = 30, 
    category,
    semanticWeight = 0.6,
    textWeight = 0.4
  } = options;

  // Build dynamic query parts
  const params: any[] = [
    `[${embedding.join(',')}]`, // $1
    query,                       // $2
    maxAgeDays,                  // $3
    limit                        // $4
  ];
  
  let categoryFilter = '';
  if (category) {
    params.push(category);      // $5
    categoryFilter = `AND category = $${params.length}`;
  }

  const result = await pool.query(`
    SELECT 
      id,
      content,
      source_url,
      created_at,
      embedding <-> $1::vector AS distance,
      ts_rank(to_tsvector('english', content), plainto_tsquery('english', $2)) AS text_rank,
      -- Hybrid score: combine normalized scores
      (
        (1 - (embedding <-> $1::vector)) * ${semanticWeight} +
        ts_rank(to_tsvector('english', content), plainto_tsquery('english', $2)) * ${textWeight}
      ) AS hybrid_score
    FROM knowledge_items
    WHERE 
      -- Temporal validity: only currently valid knowledge
      valid_from <= NOW()
      AND (valid_until IS NULL OR valid_until > NOW())
      -- Recency filter (using parameterized query)
      AND created_at > NOW() - ($3 || ' days')::INTERVAL
      -- Full-text relevance filter
      AND to_tsvector('english', content) @@ plainto_tsquery('english', $2)
      ${categoryFilter}
    ORDER BY hybrid_score DESC
    LIMIT $4
  `, params);

  return result.rows;
}
```

---

## Context window construction: Single-query pattern

* **Key Points:**
  - One database query constructs the complete context window. CTEs execute in parallel (PostgreSQL's query planner handles this), and the final SELECT combines everything into JSON. One network round trip, one transaction, consistent point-in-time snapshot across all memory types.
* **Technical Entities (Classes/Functions/APIs):** `PostgreSQL`, `CTEs`, `JSON`
* **Code Snippet:**
```typescript
interface ContextWindow {
  recentMessages: Array<{ role: string; content: string; created_at: Date }>;
  relevantFacts: Array<{ content: string; source: string; distance: number }>;
  userPreferences: Record<string, any>;
  tokenCount?: number; // Track total tokens used
}

async function buildContextWindow(
  userId: string,
  conversationId: string,
  queryEmbedding: number[],
  queryText: string, // For full-text search
  maxTokens: number = 100000 // Token budget
): Promise<ContextWindow> {
  const result = await pool.query(`
    WITH recent_messages AS (
      SELECT role, content, created_at,
             -- Estimate token count (rough: ~4 chars = 1 token)
             LENGTH(content) / 4 AS estimated_tokens
      FROM messages
      WHERE conversation_id = $1
        AND created_at > NOW() - INTERVAL '24 hours'
      ORDER BY created_at ASC  -- Chronological order
      LIMIT 50
    ),
    relevant_knowledge AS (
      SELECT 
        content, 
        source_url, 
        embedding <-> $2::vector AS distance
      FROM knowledge_items
      WHERE valid_from <= NOW()
        AND (valid_until IS NULL OR valid_until > NOW())
        AND created_at > NOW() - INTERVAL '30 days'
        -- CRITICAL FIX: Add full-text search pre-filter
        AND to_tsvector('english', content) @@ plainto_tsquery('english', $4)
      ORDER BY distance ASC
      LIMIT 5
    ),
    user_prefs AS (
      SELECT preferences
      FROM users
      WHERE id = $3
    )
    SELECT 
      jsonb_build_object(
        'messages', (
          SELECT jsonb_agg(
            jsonb_build_object(
              'role', role,
              'content', content,
              'created_at', created_at,
              'estimated_tokens', estimated_tokens
            ) ORDER BY created_at ASC  -- Maintain order
          ) 
          FROM recent_messages
        ),
        'facts', (
          SELECT jsonb_agg(
            jsonb_build_object(
              'content', content,
              'source', source_url,
              'distance', distance
            ) ORDER BY distance ASC
          )
          FROM relevant_knowledge
        ),
        'preferences', (SELECT preferences FROM user_prefs)
      ) AS context
  `, [
    conversationId, 
    `[${queryEmbedding.join(',')}]`, 
    userId,
    queryText  // Full-text search parameter
  ]);
  
  const context = result.rows[0]?.context || {};
  
  // Token budget enforcement
  let totalTokens = 0;
  const messages = (context.messages || []).filter((msg: any) => {
    totalTokens += msg.estimated_tokens || 0;
    return totalTokens <= maxTokens * 0.8; // Reserve 20% for facts/system prompt
  });
  
  return {
    recentMessages: messages,
    relevantFacts: context.facts || [],
    userPreferences: context.preferences || {},
    tokenCount: totalTokens
  };
}
```

---

## Memory Consolidation for Cost Optimization

* **Key Points:**
  - This function converts episodic memory (raw messages) into semantic memory (summaries). Old conversations are selected, aggregated, embedded (via external logic), and indexed in the knowledge_items table. After consolidation, historical message data chunks are compressed using columnar compression (e.g., via TimescaleDB/Tiger Data), achieving high data reduction while the messages remain queryable for compliance or debugging.
* **Technical Entities (Classes/Functions/APIs):** `TimescaleDB`, `Tiger Data`
* **Code Snippet:**
```sql
-- Background consolidation: convert old episodic memory to semantic summaries
CREATE OR REPLACE FUNCTION consolidate_old_conversations()
RETURNS void AS $$
DECLARE
  conversation RECORD;
  summary TEXT;
  summary_embedding vector(1536);
  -- Variable to hold the chunk name during the compression phase
  chunk_to_compress NAME;
BEGIN
  FOR conversation IN
    SELECT DISTINCT conversation_id
    FROM messages
    WHERE created_at < NOW() - INTERVAL '30 days'
      AND NOT EXISTS (
        SELECT 1 FROM knowledge_items
        WHERE metadata->>'conversation_id' = conversation_id::TEXT
          AND category = 'conversation_summary'
      )
  LOOP
    SELECT string_agg(content, E'\n' ORDER BY created_at)
    INTO summary
    FROM messages
    WHERE conversation_id = conversation.conversation_id;
    
    -- Generate embedding via application logic (placeholder)
    summary_embedding := '[0.01, 0.02, ...]'::vector;
    
    INSERT INTO knowledge_items (
      content,
      embedding,
      category,
      metadata
    ) VALUES (
      summary,
      summary_embedding,
      'conversation_summary',
      jsonb_build_object(
        'conversation_id', conversation.conversation_id,
        'consolidated_at', NOW()
      )
    );

  END LOOP;

  -- Compress the historical chunk(s) ONCE after consolidation
  -- This iterates over ALL relevant chunks in the 30-40 day window and compresses them.
  FOR chunk_to_compress IN
    	SELECT show_chunks('messages',
        newer_than => NOW() - INTERVAL '40 days',
        older_than => NOW() - INTERVAL '30 days')
  LOOP
    	PERFORM compress_chunk(chunk_to_compress);
  END LOOP;

END;
$$ LANGUAGE plpgsql;
```

---

## Framework Integrations

* **Key Points:**
  - Critical update: LangChain v0.3+ deprecated individual memory classes in favor of LangGraph persistence via checkpointers. The PGVector class is deprecated—migrate to PGVectorStore (as used above) with a connection string. The PostgresSaver from langgraph-checkpoint-postgres is the correct, current class for agent state check-pointing (for persistence and recovery), which is separate from the raw message logging handled by PostgresChatMessageHistory.
  - Postgres, via its extension pgvector, can efficiently serve as a unified Document Store (raw text/metadata) and Vector Store (embeddings) within LlamaIndex. This simplifies the RAG infrastructure, allowing LlamaIndex to store all components and perform Fast Approximate Nearest Neighbor (ANN) searches directly within the highly reliable and ACID-compliant relational database.
* **Technical Entities (Classes/Functions/APIs):** `LangChain`, `LangGraph`, `PGVectorStore`, `PostgresSaver`, `langgraph-checkpoint-postgres`, `PostgresChatMessageHistory`, `pgvector`, `LlamaIndex`, `RAG`, `ANN`, `ACID`
* **Code Snippet:**
```python
from langchain_postgres import PGVectorStore, PostgresChatMessageHistory
from langgraph.checkpoint.postgres import PostgresSaver
from psycopg import Connection

# Updated vector store pattern (PGVector class deprecated)
conn_string = "postgresql://user:pass@host:5432/db"

# Vector storage for semantic memory
vector_store = PGVectorStore(
    embeddings=embeddings,
    connection=conn_string,
    collection_name="knowledge_items",
    use_jsonb=True
)

# Chat history for episodic memory
chat_history = PostgresChatMessageHistory(
    connection_string=conn_string,
    session_id="conversation_123"
)

# LangGraph checkpointer for agent state persistence
with Connection.connect(conn_string) as conn:
    checkpointer = PostgresSaver(conn)
    checkpointer.setup()  # Create necessary tables
```

---

## Production & Performance

* **Key Points:**
  - DiskANN achieves 28x lower p95 latency than Pinecone's s1 index at 99% recall, but choose based on your scale: HNSW for <100M vectors fitting in RAM, DiskANN for billion-scale or cost-sensitive deployments. Compression policies run automatically in the background, reducing storage by 10:1 without blocking writes.
* **Technical Entities (Classes/Functions/APIs):** `DiskANN`, `HNSW`, `Pinecone`
* **Code Snippet:**
```sql
-- Choose vector index based on workload characteristics
-- DiskANN: Best for billion-scale datasets, lower RAM requirements
CREATE INDEX idx_knowledge_diskann ON knowledge_items 
  USING diskann (embedding);

-- HNSW: Best for datasets fitting in RAM, fastest queries
CREATE INDEX idx_knowledge_hnsw ON knowledge_items 
  USING hnsw (embedding vector_cosine_ops);

-- Enable automatic compression for cost reduction
ALTER TABLE messages SET (
  timescaledb.compress,
  timescaledb.compress_segmentby = 'conversation_id',
  timescaledb.compress_orderby = 'created_at DESC'
);

SELECT add_compression_policy('messages', INTERVAL '7 days');
```

---

## Monitoring Agent Token Usage

* **Key Points:**
  - Use SQL query similar to the one shown below to monitor agent token usage.
* **Technical Entities (Classes/Functions/APIs):** `SQL`
* **Code Snippet:**
```sql
-- Monitor daily token usage per model
SELECT
    time_bucket('1 day', created_at) AS day,
    model_name,
    SUM(tokens_used) AS total_tokens
FROM messages
WHERE tokens_used IS NOT NULL
GROUP BY 1, 2
ORDER BY 1 DESC, 2;
```

---

## Conclusion

* **Key Points:**
  - AI agent persistent memory consolidation into PostgreSQL eliminates the operational complexity of managing separate vector databases, time-series stores, and relational systems. Tiger Data delivers production-grade performance through hypertables (automatic time-partitioning), pgvectorscale (471 QPS at 99% recall on 50M vectors), and native compression (90%+ storage reduction). One database connection constructs complete context windows spanning episodic, semantic, and procedural memory in a single query with ACID guarantees.
  - Implement temporal validity modeling through valid_from and valid_until columns to prevent outdated knowledge retrieval. Use hybrid search combining vector similarity, full-text matching, and temporal filtering. For LangChain integration, migrate to PGVectorStore and PostgresSaver patterns following v0.3+ deprecations. Reserve specialized vector databases only for billion-scale deployments—unified PostgreSQL handles sub-100M vector workloads with lower complexity and cost.
* **Technical Entities (Classes/Functions/APIs):** `PostgreSQL`, `Tiger Data`, `hypertables`, `pgvectorscale`, `ACID`, `LangChain`, `PGVectorStore`, `PostgresSaver`
* **Code Snippet:** None.