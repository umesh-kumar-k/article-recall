# System Design Interview Blueprint: High-Scale Ticket Booking & Reservation System (Frontend Architect Focus)

---

## 1. The Frontend Architecture Lens

- **Micro-Frontend Architecture & Independent Deployability**
  - Seat selection widget, payment form, and booking confirmation as independently deployable micro-frontends composed via Module Federation (Webpack Module Federation, Single SPA, Nx Monorepo).
  - Separate teams own and release the payment UI independently of the seat map.
  - Trade-offs: Independent deployment vs. increased operational complexity; requires shared design system and versioning strategy.

- **Real-Time UI Updates for Seat Maps**
  - **SSE (Server-Sent Events)** preferred for seat map availability updates: unidirectional, HTTP-native, auto-reconnect, works through proxies/CDNs without special config.
  - **WebSockets** used for interactive features: live bidding on dynamic-priced tickets, multi-user collaborative seat selection.
  - **Long Polling** only as a fallback for environments blocking WebSocket upgrades.
  - **Stateful Server Connections** challenge: WebSockets require sticky sessions or Redis Pub/Sub fan-out layer; SSE is stateless-friendly.

- **Optimistic UI & Rollback Strategy**
  - On seat selection, UI immediately marks seat as selected and disables it for others without waiting for hold API response.
  - On hold failure (seat taken), UI rolls back with a clear error message.
  - Tools: TanStack Query `onMutate`/`onError` rollback, Immer for state snapshots.

- **Strict State Synchronization**
  - **Server state** (availability, pricing) managed by React Query/SWR with polling or SSE invalidation—avoids duplicating server state in Redux.
  - **Client-only UI state** (selected seats, step progress) in Zustand or Context API.
  - Trade-off: React Query simplifies cache invalidation but requires careful `staleTime` tuning to avoid showing stale seat availability.

- **Multi-Layered Caching Strategy**
  - **Static assets** (JS, CSS, images): immutable cache with content hash filenames (`Cache-Control: max-age=31536000, immutable`).
  - **Event listing pages**: CDN cached with stale-while-revalidate; purged on inventory update events.
  - **Seat availability**: no CDN cache; served directly from origin with short-lived Redis cache.
  - Tools: CloudFront, Fastly, Cloudflare Pages, Next.js ISR.

- **SSR/SSG/ISR Strategy for Booking Flows**
  - **SSR** for event detail pages (SEO & first-load performance); rendered per request with current pricing.
  - **ISR** for venue listing pages regenerated every 60 seconds; balances freshness with edge caching.
  - **SSG** for static marketing/FAQ pages; fully CDN-served.
  - Tools: Next.js (all three modes), Vercel, AWS Amplify.

- **Banking-Grade Frontend Security**
  - **Content Security Policy (CSP)** headers prevent XSS; `Referrer-Policy: strict-origin` on all pages.
  - **Payment form** isolated in a sandboxed iframe (Stripe Elements/Braintree Drop-in)—card data never touches the application server.
  - **Idempotency-Key header** (UUID per booking attempt) stored in Redis with 24-hour TTL; retried requests return cached response without re-executing payments or holds.
  - **Optimistic Locking (Version Column)** combined with JWT `iat` claim validation to reject stale tokens.

---

## 2. The Backend & Integration Boundaries (Frontend Architect Must-Know)

- **API Design & Gateway Patterns**
  - **REST API**: Resources modeled as nouns (`POST /bookings`, `GET /events/{id}/seats`, `PATCH /bookings/{id}/confirm`). HTTP status codes used semantically: `202 Accepted` for async booking initiation, `409 Conflict` for double booking, `412 Precondition Failed` for optimistic lock failure.
  - **Backend for Frontend (BFF)**: Separate BFF instances per client type (web BFF, mobile BFF) aggregate calls to Inventory, Pricing, and Seat Map services into a single response, reducing chatty round-trips. Web BFF can perform SSR with Next.js; mobile BFF returns a leaner payload without seat map SVG data.
  - **API Gateway**: Handles cross-cutting concerns: JWT validation, rate limiting, IP allow/block lists, request logging, SSL termination, canary routing. Routes `/api/search/*` to Search Service and `/api/bookings/*` to Booking BFF, decoupling client from internal topology.
  - **Idempotency at the Gateway**: API Gateway stamps a `Request-ID` on every inbound request; downstream services propagate it for correlation & idempotent replay.

- **Distributed Locking for Seat Selection**
  - **Redis `SET key value EX 600 NX`** automatically acquires a lock on a seat/ticket with a 10-minute TTL, preventing concurrent holds without a database round-trip.
  - **Redlock algorithm** for multi-seat bookings using a single composite key held atomically via a Lua script.
  - Trade-off: Redis is single source of truth for holds; if Redis fails before DB is updated, inventory can appear phantom held. Mitigate with persistence (AOF) and Redis Sentinel/Cluster.

- **Transactional Consistency Models**
  - **Two-Phase Reservation**:
    - **Hold Phase**: Decrement `available_count` in DB & write a `PENDING` reservation row; simultaneously set a Redis TTL key.
    - **Confirm Phase**: On payment success, update reservation status to `CONFIRMED` & publish a `booking.confirmed` event to Kafka.
    - **Release Phase**: Kafka consumer or scheduled job listens for TTL expiry & sets status to `EXPIRED`, incrementing `available_count`.
  - **Optimistic Locking (Version Column)**: Each inventory row carries a `version` integer. Update queries include `WHERE id = ? AND version = ?`; zero rows updated means a conflict occurred. Preferred over timestamp-based optimistic locking because distributed clocks drift; version numbers are monotonically reliable. On conflict, client retries with exponential back-off (up to 3 attempts) before returning a "seat no longer available" error.
  - **Pessimistic Locking**: `SELECT ... FOR UPDATE SKIP LOCKED` isolates rows without blocking the entire table. `SKIP LOCKED` allows parallel workers to process other rows without waiting—useful for queue-style seat claim workers. Trade-off: Higher throughput cost under low contention vs. optimistic; suitable only at the final confirmation step, not during browsing.

- **Rate Limiting & Throttling**
  - Per-user & per-IP rate limits enforced at the API Gateway; booking endpoints limited to 5 req/min per user to prevent seat hold abuse.
  - Sliding window algorithm preferred over fixed window to smooth burst traffic.
  - Token bucket for burst tolerance to avoid frustrating legitimate users during high-demand drops.

- **CDN & Edge Strategies**
  - Static assets served via CDN with WebP/AVIF format negotiation.
  - Event listing pages: CDN cached with stale-while-revalidate; purged on inventory update events.
  - Seat availability: no CDN cache; served directly from origin with short-lived Redis cache.

- **Communication Protocols (Browser to Backend)**
  - **HTTP/2** baseline for browser to gateway; multiplexing eliminates head-of-line blocking for concurrent API calls.
  - **HTTP/3 (QUIC)** improves mobile experience on lossy networks (important for mobile users) by removing TCP re-transmission stalls.
  - **HTTP/1.1** fallback for legacy load balancers.
  - **gRPC** used for inter-service communication (Inventory-Booking, Booking-Payment): binary Protobuf encoding is ~5-7x smaller than JSON; supports bidirectional streaming for real-time inventory feeds.
  - **REST** used for public-facing browser-accessible APIs and third-party integrations (webhooks, partner APIs).
  - **GraphQL** used at the BFF layer for flexible data aggregation; not exposed directly to public.

- **Saga/Orchestrator Pattern**
  - The booking flow (`Hold → Pay → Confirm → Notify`) is a distributed transaction spanning multiple services; a Saga orchestrator manages compensating transactions on failure.
  - **Orchestration Saga**: Temporal.io workflow explicitly drives each step & compensation (release hold on payment failure).
  - Trade-off vs. Choreography: Orchestration is easier to reason about and debug; choreography is more loosely coupled but harder to trace.

- **Event-Driven Architecture (Domain Events)**
  - Services communicate via domain events rather than direct calls; Booking Service publishes events consumed by Notification, Analytics, and Loyalty services—fully decoupled.
  - **Outbox pattern** (write event to DB outbox table in same transaction, then relay to Kafka) ensures no lost events between DB commit & broker publish.
  - Tools: Debezium (CDC for outbox), Kafka Connect.

- **CQRS (Command Query Responsibility Segregation)**
  - Command side: Booking/Inventory services handle writes with strong consistency.
  - Query side: Elasticsearch or a separate read DB serves search & seat map queries, updated asynchronously via Kafka consumers.
  - Trade-off: Increases architectural complexity; justified by the read-heavy nature (100:1 read-to-write ratio typical).

---

## 3. Critical Edge Cases & Failure Modes

- **Traffic Spikes (High-Demand Ticket Drops)**
  - **Rate limiting** at API Gateway (5 req/min per user) with sliding window to prevent seat hold abuse.
  - **Token bucket** for burst tolerance to avoid frustrating legitimate users.
  - **CDN caching** for event listings with stale-while-revalidate; seat availability no CDN cache.
  - **Database scaling**: Write path single primary PostgreSQL with connection pooling via PgBouncer; read replicas for search queries; sharding by `event_id` or `venue_id` for extreme scale.
  - **Load balancing**: L6 load balancing at API Gateway with least connections algorithm; health checks auto-remove failed pods. Sticky sessions only for WebSocket connections (hash by user ID); all other services stateless.

- **Race Conditions & Double Booking**
  - **Database-level**: `UNIQUE` constraint on `(seat_id, event_id, status = 'CONFIRMED')` as the final safety net.
  - **Application-level**: Redis distributed lock (`SETNX`) per seat acquired before any inventory write.
  - Trade-off: DB constraint alone causes hard failures under race conditions; Redis lock provides a softer, earlier rejection with a better UX error message.
  - **Optimistic Locking** retry with exponential back-off (up to 3 attempts) before returning "seat no longer available" error.

- **Network Dropouts Mid-Transaction**
  - **Idempotency tokens**: Clients generate a UUID (`Idempotency-Key` header) per booking attempt; server stores `(idempotency_key → response)` in Redis with 24-hour TTL. Retried requests return the cached response without re-executing payments or holds.
  - **Circuit Breaker** pattern: Booking Service wraps calls to Payment Service; on 5 consecutive failures, it opens and returns a graceful "payment unavailable" error. Prevents cascading failure during payment provider outages.
  - **Backpressure/Bulkhead**: Isolates thread pools per downstream dependency (Payment, Inventory) so a slow payment provider doesn't exhaust threads for inventory calls. Kafka consumer groups apply natural backpressure; consumers lag rather than overwhelming the DB.
  - **Optimistic UI rollback** on hold failure: TanStack Query `onError` rollback with Immer for state snapshots.

- **Data Sync Failures (Eventual Consistency Issues)**
  - **Keyspace Notifications** (`notify-keyspace-events KEx`) trigger a Lambda/consumer to reconcile the DB on Redis TTL expiry.
  - **Outbox pattern** ensures no lost events between DB commit & Kafka publish.
  - **Circuit Breaker** prevents cascading failures; graceful degradation with "payment unavailable" error.
  - **Reconciliation jobs**: Background worker (Celery, BullMQ) scans expired holds & restores inventory counts via compensating transactions.
  - **Monitoring**: Kafka consumer group lag monitoring (Burrow); Redis Sentinel/Cluster for high availability.

- **Redis Failure & Phantom Holds**
  - Redis is single source of truth for holds; if Redis fails before DB is updated, inventory can appear phantom held.
  - Mitigation: Persistence (AOF) and Redis Sentinel/Cluster for high availability.
  - Fallback: DB as source of truth; Redis for fast checks; Keyspace Notifications for reconciliation.

- **Stale Cache & Pricing Inconsistencies**
  - Pricing Service evaluates demand signals (current hold count, time-to-event, historical fill rate) & returns a price per request.
  - Prices cached in Redis with short TTL (30–60s) to avoid per-request DB hits while staying near real-time.
  - Trade-off: Aggressive caching can show stale prices; use short TTL + cache invalidation on demand threshold events.
  - Client-side: React Query `staleTime` tuning to avoid showing stale seat availability.

- **Security Threats (Compromised/Replayed Requests)**
  - **Optimistic Locking (Version Column)** prevents a compromised or replayed request from overwriting a newer booking state (e.g., replaying a `PENDING` state over a `CONFIRMED` booking).
  - Combined with JWT `iat` (issued-at) claim validation to reject stale tokens.
  - **Zero Trust Architecture**: Every internal service call is authenticated & authorized; no implicit trust based on network location. mTLS enforced between all microservices via a service mesh; certificates rotated automatically.
  - **Secrets Management**: No secrets in source code or environment variables directly; all credentials fetched from a secret manager at runtime. Dynamic DB credentials rotated automatically; short-lived.

---

## 4. End-to-End Booking Flow (Frontend Architect's Mental Model)

1. **Search** – User queries available events/rooms; Search Service queries Elasticsearch; CDN-cached results returned via BFF with current pricing from Pricing Service.
2. **Availability** – Seat map loaded; Inventory Service returns real-time availability (Redis-backed); SSE channel opened for live seat status updates.
3. **Seat Selection** – User selects seat; optimistic UI marks it as selected immediately.
4. **Hold** – `POST /bookings` with `Idempotency-Key`; Booking Service acquires Redis distributed lock on seat, writes `PENDING` reservation, decrements `available_count`; TTL set to 10 min.
5. **Payment Authorization** – BFF renders Stripe Elements iframe; user submits card details directly to Stripe; Stripe returns a `PaymentIntent` client secret; client confirms payment.
6. **Payment Capture** – Stripe webhook (`payment_intent.succeeded`) received by Payment Service; event published to Kafka `payment.authorized` topic.
7. **Confirm** – Booking Service consumes event; updates reservation to `CONFIRMED`; DB version incremented; Redis hold key deleted.
8. **Notification** – Notification Service consumes `booking.confirmed`; sends email via SendGrid and push via Firebase; PDF ticket generated asynchronously.
9. **Analytics** – Analytics consumer updates real-time dashboards; Loyalty Service awards points; Search index refreshed via Debezium CDC.

---

## 5. Key Trade-Offs Summary Table

| **Decision**          | **Option A**               | **Option B**                      | **Recommendation for Frontend Architect**                                                                 |
| --------------------- | -------------------------- | --------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Consistency**       | Strong (synchronous DB)    | Eventual (async events)           | Strong for holds/payments; eventual for notifications/analytics. Frontend must handle stale states gracefully. |
| **Locking**           | Optimistic (version col)   | Pessimistic (`SELECT FOR UPDATE`) | Optimistic for general inventory; pessimistic for last-seat scenarios. Frontend retries on `412` errors.    |
| **API Style**         | REST                       | GraphQL                           | REST for public APIs (cacheable, simpler); GraphQL at BFF for complex aggregation (reduces round-trips).    |
| **Real-Time**         | WebSockets                 | SSE                               | SSE for seat availability (unidirectional, stateless-friendly); WebSockets for interactive features (bidding). |
| **Broker**            | Kafka                      | RabbitMQ                          | Kafka for event streaming/audit (replay, ordering); RabbitMQ for task queues (simpler ops).                |
| **Cache Risk**        | High TTL (performance)     | Low TTL (freshness)               | Short TTL (30s) for inventory; long TTL for static content. Frontend uses `stale-while-revalidate`.        |
| **Architecture**      | Monolith                   | Microservices                     | Microservices with domain boundaries; Frontend communicates via BFF & API Gateway.                         |
| **Saga Style**        | Orchestration (Temporal)   | Choreography (events)             | Orchestration – easier to trace and compensate in financial flows. Temporal for critical paths.            |
| **Hold Expiry**       | Redis TTL only             | DB + Redis                        | DB as source of truth; Redis for fast checks; Keyspace Notifications for reconciliation.                   |
| **Frontend Rendering** | CSR (SPA)                  | SSR/ISR (Next.js)                 | SSR for event pages (SEO); CSR for seat selection (interactivity). ISR for listing pages.                  |

---

## 6. GenAI Integration Patterns (Bonus for Target Role)

While the core system is transactional, a **Senior Frontend Architect + GenAI Integration** role requires familiarity with:

- **RAG (Retrieval-Augmented Generation) for Customer Support**
  - Vector embeddings of event FAQs, seating policies, and cancellation rules stored in Pinecone/Weaviate.
  - Customer query → embedding → vector search → context → LLM response (Gemini/GPT-4) served via a **Chat API**.
  - Frontend integration: Chat widget streaming responses via SSE (Server-Sent Events) for real-time conversational UX.

- **Agentic AI Frameworks for Personalization**
  - AI agents (LangChain, CrewAI) that:
    - Suggest seat upgrades based on user's purchase history and loyalty tier.
    - Offer dynamic pricing recommendations based on demand forecasts.
    - Automate post-booking upsell (parking, dining, merchandise) via personalized recommendations.
  - Frontend surfaces these suggestions via **recommendation carousels** and **personalized landing pages** (ISR with user-specific data).

- **Model Context Protocol (MCP) for Context-Aware Booking**
  - MCP servers providing context (user's loyalty status, past cancellations, event preferences) to LLMs.
  - LLM agents use MCP to:
    - Understand user intent ("I want front-row seats for the Sunday show").
    - Provide natural language booking assistance (e.g., voice/chat commands).
    - Detect fraud or unusual booking patterns.
  - Frontend: Conversational UI with real-time validation via WebSocket/SSE.

- **Caching & Performance for AI Features**
  - AI-generated responses cached at the CDN edge with TTL (e.g., recommendations cached for 10 minutes).
  - Embeddings pre-computed for static content (event descriptions, FAQs) to avoid latency.

- **Security & Compliance in GenAI**
  - **PII redaction**: Customer queries redacted before sending to external LLMs.
  - **Audit logging**: Every AI interaction logged for compliance (PCI-DSS, GDPR).
  - **Hallucination guardrails**: RAG with strict grounding; responses validated against source documents before serving.


  --- 

  # Scaling a High-Concurrency Ticket Booking System: Key Concepts in Kafka, RabbitMQ, and NoSQL

As a Frontend Architect, you don't need to operate these systems daily, but you **must** understand their role in enabling the scalability, performance, and reliability of the backend. Below are three concise reference guides—each about two pages—covering the critical concepts and how they map to the ticket booking/reservation system.

---

## 1. Apache Kafka – Event Streaming for Scalability & Reliability

### Core Concepts

| Concept | Description | Relevance to Ticketing System |
|---------|-------------|-------------------------------|
| **Topic** | A logical category for messages (e.g., `booking.confirmed`, `payment.authorized`). | Each major event in the booking lifecycle has its own topic, enabling decoupled services to consume only what they need. |
| **Partition** | Each topic is split into ordered, immutable sequences of messages. Partitions enable parallelism. | Booking events can be partitioned by `event_id` or `user_id` to distribute load across many consumers. |
| **Producer** | A client that publishes messages to a topic. | Booking Service publishes events after each state change (e.g., `hold.created`, `payment.completed`). |
| **Consumer** | A client that subscribes to one or more topics and processes messages. | Notification Service consumes `booking.confirmed` to send emails; Analytics Service consumes all events for dashboards. |
| **Consumer Group** | A set of consumers that work together to consume messages from a topic; each partition is assigned to exactly one consumer in the group. | Enables horizontal scaling of consumers (e.g., Notification Service can run multiple instances, each processing a subset of partitions). |
| **Offset** | A unique identifier for each message within a partition. Consumers commit offsets to track which messages have been processed. | Enables **exactly-once** or **at-least-once** delivery semantics, critical for financial transactions to avoid double processing. |
| **Replication** | Each partition can be replicated across multiple brokers to provide high availability. | If a broker fails, another broker can serve the partition, ensuring no data loss and continuous event flow. |
| **ISR (In-Sync Replicas)** | A set of replicas that are fully caught up with the leader. | Producers can be configured to wait for acknowledgment from all ISRs before considering a write successful—ensuring durability. |
| **Retention Policy** | Messages are retained for a configurable period (e.g., 7 days) even after consumption. | Enables **event replay** for debugging, auditing, or rebuilding state (e.g., replaying all `booking.confirmed` events to reconstruct a view). |

### How Kafka Achieves Scalability & Performance

- **Partitioning** – Allows linear scaling: more partitions → more parallelism for both producers and consumers.
- **Sequential I/O** – Writes are append-only, and reads are sequential, making Kafka extremely fast even with high throughput.
- **Zero-copy & Batching** – Producers batch messages, reducing network round-trips; brokers use zero-copy for efficient data transfer.
- **Consumer Groups** – Add more consumers to a group to increase processing throughput without code changes.

### How Kafka Provides Reliability & Consistency

- **Exactly-Once Semantics** – With idempotent producers and transactional APIs, Kafka can guarantee that messages are processed exactly once, even in case of retries or failures.
- **Durable Storage** – Messages are persisted to disk and replicated; no data loss if configured correctly (`acks=all`, `min.insync.replicas=2`).
- **Ordering Guarantee** – Within a partition, messages are strictly ordered—critical for state machines (e.g., a seat must be held before it can be confirmed).

### Kafka in the Ticket Booking Flow

| Step | Kafka's Role |
|------|--------------|
| **Hold Created** | Booking Service publishes `hold.created` to Kafka. |
| **Payment Authorized** | Payment Service publishes `payment.authorized` on Stripe webhook. Booking Service consumes this and updates reservation. |
| **Booking Confirmed** | Booking Service publishes `booking.confirmed`. Notification, Analytics, and Loyalty services consume it asynchronously. |
| **Hold Expired** | A scheduled consumer (or Kafka Streams) listens for hold expiration TTL events and publishes `hold.expired` to restore inventory. |
| **Audit Log** | All events are retained for compliance (e.g., PCI-DSS requires audit trails of all booking changes). |

### Kafka vs. Traditional Databases

- Kafka is not a database; it is an **event log**. It does not serve as the source of truth for current state but as the source for **state evolution**.
- Combined with **event sourcing**, the entire booking state can be rebuilt by replaying events—this enables full audit and debugging.

---

## 2. RabbitMQ – Task Queues for Reliable Asynchronous Processing

### Core Concepts

| Concept | Description | Relevance to Ticketing System |
|---------|-------------|-------------------------------|
| **Message Broker** | A middleware that routes, queues, and delivers messages between producers and consumers. | RabbitMQ handles short-lived, task-oriented messages that require acknowledgments and retries. |
| **Queue** | A buffer that stores messages until they are consumed. | Queues hold tasks like "send email", "generate PDF ticket", "call third-party webhook". |
| **Exchange** | Receives messages from producers and routes them to queues based on routing rules. | Types: `direct`, `topic`, `fanout`, `headers`. For example, a `topic` exchange can route `booking.created` to queues for email, SMS, and analytics. |
| **Binding** | A rule that connects an exchange to a queue, defining which messages go where. | Bindings define the routing keys (e.g., `*.confirmed` routes to the confirmation queue). |
| **Acknowledgement (ACK)** | Consumers send an ACK after processing a message; if not received, the message is re-queued. | Ensures that if a worker fails (e.g., email server down), the task is retried automatically. |
| **Dead Letter Queue (DLQ)** | A queue for messages that cannot be processed (e.g., after max retries). | Failed tasks (e.g., email delivery repeatedly failing) go to DLQ for manual inspection. |
| **TTL (Time-To-Live)** | Messages can expire after a given time. | If a booking confirmation email is not sent within 5 minutes, it might be discarded to avoid late notifications. |
| **Prefetch Count** | Limits the number of unacknowledged messages delivered to a consumer. | Prevents a slow consumer from being overwhelmed; ensures fair distribution of load. |
| **Publisher Confirms** | Producers receive an acknowledgment that the message was successfully routed to a queue. | Guarantees that booking events are not lost on the producer side. |

### How RabbitMQ Enables Scalability & Performance

- **Parallel Workers** – Multiple consumers can read from the same queue; RabbitMQ distributes messages round-robin (or by using `prefetch` to balance based on capacity).
- **Asynchronous & Decoupled** – The booking service does not wait for email or PDF generation to complete; it just publishes the task and returns quickly.
- **Retry & Backoff** – Failed tasks can be re-queued with a delay (via `x-dead-letter-exchange` and `x-message-ttl`) to implement exponential backoff—reducing load on downstream systems.

### How RabbitMQ Ensures Reliability

- **Message Persistence** – Queues and messages can be marked as durable, surviving broker restarts.
- **Delivery Guarantees** – With publisher confirms and consumer ACKs, messages are delivered at least once (idempotent processing is recommended).
- **Dead Letter Handling** – Provides a graceful failure path for tasks that consistently fail, avoiding endless retries.

### RabbitMQ in the Ticket Booking Flow

| Step | RabbitMQ's Role |
|------|-----------------|
| **Post-Booking Email** | A task is queued with `send-email` routing key. A worker sends the email via SendGrid; on failure, it re-queues with a delay. |
| **PDF Ticket Generation** | A queue handles PDF generation; after success, the PDF is stored in S3, and a notification is sent via another queue. |
| **Third-Party Webhooks** | External systems (e.g., partner apps) receive booking updates via webhooks; these are queued to ensure retry on failure. |
| **Inventory Reconciliation** | A worker periodically checks for expired holds and releases inventory—ensuring eventual consistency. |

### RabbitMQ vs. Kafka

| Aspect | RabbitMQ | Kafka |
|--------|----------|-------|
| **Primary Use** | Task queues, RPC, short-lived messages | Event streaming, log aggregation, long-term storage |
| **Consumption Model** | Competing consumers; messages are removed after ACK | Consumer groups; messages persist and can be replayed |
| **Ordering** | Not guaranteed across multiple consumers (unless single consumer) | Guaranteed per partition |
| **Retention** | Messages deleted after consumption | Configurable retention (days/weeks) |
| **Latency** | Very low (milliseconds) | Slightly higher due to batching and persistence |

---

## 3. NoSQL Databases – Enabling High-Speed Reads, Writes, and Caching

NoSQL databases are not a single category. In the ticketing system, we use **different NoSQL technologies for different purposes**.

### Redis – In-Memory Data Store for Low-Latency Operations

#### Core Concepts

| Concept | Description | Relevance to Ticketing System |
|---------|-------------|-------------------------------|
| **Key-Value Store** | Simple `GET`/`SET` operations with sub-millisecond latency. | Distributed locks, session caching, TTL-based holds. |
| **TTL (Time-To-Live)** | Keys automatically expire after a set time. | Seat holds expire after 10 minutes without manual intervention. |
| **Atomic Operations** | Commands like `INCR`, `DECR`, `SETNX` are atomic. | `SETNX` ensures only one process acquires a seat lock. |
| **Lua Scripting** | Scripts execute atomically on the server. | Multi-seat bookings can be handled in a single atomic script. |
| **Pub/Sub** | Lightweight messaging for real-time notifications. | Fan-out inventory updates to multiple connected clients (SSE/WebSocket). |
| **Persistence (AOF/RDB)** | Snapshots or append-only logs for durability. | Prevents data loss in case of Redis restart (critical for active holds). |
| **Cluster Mode** | Data sharded across multiple nodes for horizontal scaling. | Supports millions of concurrent seat holds across many events. |
| **Sentinel** | Provides high availability via automatic failover. | Prevents single points of failure in Redis. |

#### How Redis Enables Scalability & Performance

- **In-Memory Speed** – Redis operates entirely in RAM, achieving sub-millisecond response times—critical for seat selection under heavy load.
- **No Disk I/O** – For most operations (TTL, locks, Pub/Sub), Redis never touches the disk, ensuring consistent low latency.
- **Sharding** – Redis Cluster distributes keys across nodes, allowing the system to scale horizontally as the number of concurrent users grows.
- **Pipelining** – Clients can batch multiple commands in one network round-trip, reducing latency for multi-seat selection.

#### How Redis Provides Reliability

- **Persistence** – AOF (Append-Only File) ensures that even if Redis crashes, the most recent writes can be replayed.
- **Replication** – Master-slave replication provides read replicas for scaling reads (e.g., seat map lookups).
- **Sentinel** – Automatic failover ensures that the system continues to operate if a master node fails.

#### Redis in the Ticket Booking Flow

| Use Case | How Redis Helps |
|----------|-----------------|
| **Seat Hold Lock** | `SETNX` on `seat:event:seatId` with 600s TTL prevents double-booking. |
| **Session Caching** | User sessions stored in Redis to avoid database round-trips for every request. |
| **Real-Time Seat Map** | Redis Pub/Sub pushes availability updates to all connected clients. |
| **Idempotency Keys** | Stores `(idempotency_key, response)` for 24h to prevent duplicate submissions. |
| **Rate Limiting** | Sliding window counters stored in Redis for per-user rate limits. |

---

### Elasticsearch – Full-Text Search & Read Models

#### Core Concepts

| Concept | Description | Relevance to Ticketing System |
|---------|-------------|-------------------------------|
| **Index** | A collection of documents (similar to a table in SQL). | `events`, `venues`, `bookings` indices. |
| **Document** | A JSON object containing fields and values. | An event document includes `name`, `venue`, `date`, `available_seats` (denormalized). |
| **Shard** | Each index is split into shards for horizontal scaling. | Distributes search load across multiple nodes. |
| **Replica** | Copies of shards for high availability and read scalability. | Allows serving search queries from replicas while the primary shards handle writes. |
| **Analyzer** | Text processing (tokenization, stemming) for accurate search. | Enables free-text search on event names, artists, venue names. |
| **Query DSL** | JSON-based query language for complex searches. | Supports filters (date, venue, price range), aggregations (group by genre), and geospatial queries. |
| **Near Real-Time (NRT)** | Documents are searchable within ~1 second of indexing. | Event updates (e.g., seat availability) are reflected quickly. |

#### How Elasticsearch Enables Scalability & Performance

- **Distributed Indexing** – Data is automatically distributed across shards; each node handles only a subset.
- **Parallel Query Execution** – A search query is sent to all relevant shards in parallel, then results are merged.
- **Caching** – Query results and filter caches speed up repeated searches (e.g., popular event listings).
- **Denormalization** – Search indices are optimized for reads; event documents contain pre-computed fields (like available seats count) to avoid joins.

#### Elasticsearch in the Ticket Booking Flow

| Use Case | How Elasticsearch Helps |
|----------|-------------------------|
| **Event Discovery** | Users search by event name, artist, venue, date; Elasticsearch returns relevant results in milliseconds. |
| **Faceted Search** | Aggregations provide filter counts (e.g., "Rock concerts in New York this weekend"). |
| **Availability Snapshot** | The search index contains a cached `available_seats` count, updated asynchronously via Kafka. |
| **Analytics Dashboards** | Elasticsearch can power real-time dashboards for business metrics (e.g., ticket sales by venue). |

---

### NoSQL vs. Relational Databases: Key Trade-Offs for Ticketing

| Aspect | NoSQL (Redis, Elasticsearch) | Relational (PostgreSQL) |
|--------|-------------------------------|-------------------------|
| **Consistency Model** | Eventual (search, caches) / Strong (Redis locks with atomic ops) | Strong ACID (critical for payments and confirmations) |
| **Schema** | Schema-less (Elasticsearch) / Simple key-value (Redis) | Fixed schema with constraints, relationships |
| **Transactions** | Limited (Redis has multi/exec, but not ACID across keys) | Full ACID with `SERIALIZABLE` isolation |
| **Scaling** | Horizontal (sharding) is built-in | Vertical scaling + sharding extensions (Citus) |
| **Use Case** | Caching, real-time locks, full-text search, read models | Source of truth for bookings, payments, inventory |

### Recommendation: Hybrid Persistence

- **PostgreSQL** serves as the **source of truth** for all critical data (bookings, payments, inventory counts) with strong consistency.
- **Redis** handles **high-throughput, low-latency** operations: seat locks, sessions, rate limiting, and caching.
- **Elasticsearch** powers **search and read-only views** that are updated asynchronously via Kafka, providing fast, flexible queries without impacting the transactional database.

This combination ensures that the system achieves:
- **Scalability** – Each component scales independently based on load.
- **Performance** – Reads are served from optimized caches and search indices; writes are handled by the transactional DB.
- **Reliability** – The source of truth remains ACID-compliant; caches and search indexes are eventually consistent and can be rebuilt from events.

---

## Summary Table: Kafka, RabbitMQ, Redis, Elasticsearch in the Booking Flow

| Component | Primary Role | Critical Features |
|-----------|--------------|-------------------|
| **Apache Kafka** | Event streaming backbone | Partitioning, replication, exactly-once semantics, event replay, audit logging. |
| **RabbitMQ** | Task queue & reliable execution | ACKs, dead-letter queues, retries, publisher confirms. |
| **Redis** | In-memory cache & distributed locks | `SETNX`, TTL, Lua scripts, Pub/Sub, persistence via AOF. |
| **Elasticsearch** | Search & read model | Sharding, near real-time indexing, full-text search, aggregations. |

---

