---
aliases:
  - Embedding Versioning
Source 1: https://tianpan.co/blog/2026-04-19-embedding-refresh-vector-store-database-engineer
---
# The Embedding Refresh Problem: Running a Vector Store Like a Database Engineer

## Why Embeddings Go Stale (and Why It's Hard to Notice)

* **Key Points:**
  - The root cause is simple: source data changes, but vector indexes don't update automatically.
  - A document gets revised and the embedding in the index still reflects the old version. A product is discontinued but the ghost vector keeps surfacing in search results. A policy changes and queries about that policy start retrieving context that's legally incorrect. None of this produces an error. Retrieval still "succeeds" — it just retrieves the wrong thing.
  - Several forces accelerate the problem:
    - Temporal drift is the most common. Financial reports, compliance rules, product specifications, and API documentation all have shelf lives. Embeddings generated from these documents capture a snapshot that becomes less accurate over time. Research on temporal embeddings shows retrieval accuracy declining 15–20% on time-sensitive queries as indexes age without refresh.
    - Deletion lag is subtler. When you delete a document from your source system, the corresponding vector usually stays in the index. HNSW and other graph-based ANN structures don't support clean deletes — the common workaround is soft deletion with a filter flag, but that approach bloats the index over time and creates compliance risk if you need to guarantee that deleted data is truly gone.
    - Model version drift is the most disruptive. When your embedding model provider releases a new model version and you want to upgrade, all existing vectors become incompatible. They were computed in a different geometric space. Querying a v3 embedding against a v1-embedded index doesn't throw an error — it just returns nonsense.
    - In-process architecture failures are a category of their own. Developers running Chroma in library mode (in-process) discover that each worker process loads its own in-memory copy of the index. When one worker ingests new documents, the others remain completely unaware. The fix is to run vector stores in server mode, but teams often only discover this need after a production staleness incident.
* **Technical Entities (Classes/Functions/APIs):** `HNSW`, `Chroma`

## The Write-Once Antipattern
* **Key Points:**
  - Most teams embed their corpus once, at ingestion time, and treat the index as static infrastructure. This works fine during the prototype phase when the corpus is small and changing slowly. It falls apart in production.
  - The write-once antipattern has three symptoms:
    - Infrequent batch refreshes — teams run a full reindex nightly or weekly, accepting that every answer served during the window is some number of hours stale
    - No freshness monitoring — there's no alerting when the gap between source-of-truth and vector index exceeds acceptable bounds
    - Architectural rigidity — because incremental update was never designed in, the only path to refreshing the index is a full rebuild

## Full Reindex vs. Incremental Update
* **Key Points:**
  - The right strategy depends on how your corpus changes and how much staleness you can tolerate.
  - Full reindexing gives you consistency and a clean index structure. It's operationally simple — you rebuild the index on a schedule and swap it in. The downsides are cost (proportional to corpus size), latency (the index rebuild takes time, during which you're serving from the old version), and the fundamental problem that your freshness ceiling is bounded by how often you can afford to rebuild.
  - Incremental updates target only changed documents. The challenge is that HNSW and similar graph-based indexes are optimized for querying, not frequent mutation. Dynamic insertions require adding nodes and updating neighbor connections, which degrades index quality over time as the graph structure becomes unbalanced. At high insert volume, query performance starts to degrade in ways that aren't obvious without careful monitoring.
  - Hybrid approaches — incremental updates between periodic full reindexes — are the production pattern most mature teams converge on. Write changes incrementally as they happen. Schedule a full rebuild during low-traffic periods (typically weekly or monthly) to restore index health. This balances freshness with structural integrity.
  - The selection criteria roughly maps to:
    - Corpus rarely changes, small size: full reindex on a schedule works fine
    - Corpus changes frequently, can tolerate eventual consistency: incremental update with periodic full rebuild
    - Corpus changes in real time, freshness SLA measured in minutes: change-data-capture pipeline feeding continuous incremental updates
* **Technical Entities (Classes/Functions/APIs):** `HNSW`

## Change-Data-Capture for Vector Indexes
* **Key Points:**
  - The data engineering pattern that solves the real-time freshness problem is change-data-capture (CDC) — the same pattern that database engineers use to replicate changes between systems.
  - In the vector index context, CDC works like this: every insert, update, or delete in the source system emits a change event. A pipeline consumes those events, re-embeds affected documents, and writes the new vectors to the index. The gap between source change and index update shrinks from hours to seconds.
  - This sounds straightforward but has operational nuances. The synchronization pipeline needs to handle:
    - Ordering guarantees — a delete followed immediately by a re-insert of an updated document must be processed in order, or you'll end up with a stale vector for the re-inserted document
    - Idempotency — if the pipeline retries a failed event, re-embedding the same document shouldn't produce duplicate vectors
    - Back-pressure — if the source system produces bursts of changes (a bulk import, a schema migration), the embedding pipeline needs to absorb load without falling over
    - Transactional consistency — if the source database update and the vector index update need to be atomic (they often do), you need an outbox pattern or dual-write with compensation logic
  - For teams running relational databases as their source of truth, pgvector is worth considering: by storing embeddings in the same Postgres instance as source data, you can wrap the source update and embedding update in a single transaction, eliminating the consistency gap entirely.
* **Technical Entities (Classes/Functions/APIs):** `CDC`, `pgvector`, `Postgres`

## Detecting Drift Before Users Do
* **Key Points:**
  - The problem with embedding staleness is that it degrades silently. You need instrumentation that detects when the index has diverged from source truth before users notice.
  - Freshness tracking is the simplest signal: for every document in the source system, track when it was last embedded and compare that timestamp to when it was last updated. Percentage of documents outside a freshness SLA is a number you should be watching on a dashboard.
  - Embedding drift detection measures whether the statistical distribution of vectors in the index has shifted relative to a baseline. The most production-reliable approach is Euclidean distance between the centroid of a current vector sample and a historical baseline. Cosine distance also works but produces less stable alert thresholds. Drift scores above 0.05–0.10 in cosine space typically indicate meaningful distribution shift worth investigating.
  - Retrieval quality probes are golden-path queries where you know the correct answer. Running these on a schedule and monitoring whether retrieval rank of the known-good document is degrading gives you a direct measure of staleness impact, rather than a proxy metric.
  - Index health metrics specific to the ANN structure round out the picture: ratio of soft-deleted vectors (should stay below 10–15%), query latency at P95 (degrades as index becomes fragmented), and recall at k (approximated by comparing results to a flat exhaustive search on a sample).

## Surviving Model Version Migrations
* **Key Points:**
  - Upgrading embedding models is where the write-once antipattern becomes most dangerous. Old vectors and new queries live in incompatible geometric spaces. Searching a v1 index with a v3 query doesn't fail — it just returns garbage results with no error signal.
  - The naive approach is to rebuild the entire index with the new model before switching. At scale this is prohibitively slow and expensive, and it requires a hard cutover that inevitably creates a service disruption window.
  - The side-by-side column pattern avoids the cutover problem:
    - Keep the old embedding column and add a new column for the new model's vectors
    - Run a background job that re-embeds documents and populates the new column, with rate limiting to avoid exhausting API quotas
    - Route a shadow traffic percentage through the new column to validate quality before fully switching
    - Use a feature flag to switch the live query path to the new column once validation passes
    - Clean up the old column after a stability window
  - This approach extends a migration that would otherwise require a maintenance window into a zero-downtime 48-hour process.
  - Drift-Adapter is a research approach for cases where re-embedding the full corpus is genuinely infeasible. It trains a lightweight transformation layer that maps new-model query embeddings into the old model's vector space, allowing the unchanged index to serve queries from the new model. The quality recovery is substantial (95–99% recall recovery with a low-rank affine transformation) and the compute cost is roughly 100× less than full re-indexing. It's not a permanent solution, but it buys time.
* **Technical Entities (Classes/Functions/APIs):** `Drift-Adapter`

## The Data Engineering Mindset
* **Key Points:**
  - The operational discipline for vector stores maps almost directly onto what data engineers already know about relational databases:
    - Treat the vector index as a derived data store, not the source of truth. Source of truth lives in your operational database. The index is a materialized view that must be kept in sync.
    - Design for writes, not just reads. If your indexing pipeline can't handle continuous updates, you've built infrastructure that can only be batch-refreshed, and your freshness ceiling is determined by your compute budget rather than your actual freshness requirements.
    - Version your schemas. When you upgrade embedding models, treat it like a database schema migration — dual-write during the transition, validate the new schema before switching reads, maintain rollback capability.
    - Monitor the gap between source and index, not just query latency. A vector store that returns results fast but serves stale content is not meeting its reliability requirements.
    - Design deletions explicitly. If you need to guarantee that deleted documents don't appear in search results (GDPR erasure, product discontinuation, document retraction), you need a deletion propagation pipeline, not just soft deletes that you filter at query time.
  - RAG systems don't fail because the retrieval algorithm is wrong. They fail because the data the algorithm is searching over is wrong. The AI engineering problem of building a good RAG system is, at its core, a data engineering problem of maintaining a synchronized derived data store at scale. Teams that figure this out early ship more reliable systems. Teams that treat the vector index as a write-once artifact discover it the hard way, in production, when the evidence has already gone stale.