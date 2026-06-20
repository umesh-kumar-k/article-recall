---
aliases:
  - Reservation & Ticket Booking System
---
# Major Components

- Browser/Mobile - React/Angular/Next.js SPA or PWA
- CDN/Edge - Static assets,edge caching, DDos protection
- API Gateway - Auth, rate limiting,routing, SSL termination
- BFF - Aggregates service calls per client type(web/mobile)
- Booking Service - Orchestrates hold -> payment -> confirm flow
- Inventory Service - Manages seat/room/ticket availability
- Payment Service - Integrates with Stripe/Braintree for auth & capture
- Search Service - Elasticsearch powered availability & venue search
- Pricing Service - Dynamic pricing rules & fare calculations
- Notification Service - Email/SMS/push via SendGrid, Twilio, Firebase
- Cache - Redis for distributed locks, sessions, seat maps
- Message Broker - Kafka for event streaming,RabbitMQ for task queues

# Core Concepts

## Inventory Holds

- Redis

  ```
  SET key value EX 600 NX 
  ```

automatically acquires a lock on a seat/ticket with a 10-minute TTL, preventing concurrent holds without a database round-trip

- A background worker(Celery,BullMQ) scans expired holds & restores inventory counts via a compensating transaction.
- Trade-off: Redis is single-source of truth for holds; if Redis fails before the DB is updated,inventory can appear phantom held. Mitigate with persistence (AOF) and Redis Sentinel/Cluster
- Tools: Redis, BullMQ/Celery, Lua scripts for atomic operations

## Two-Phase Reservation

- Hold Phase - Decrement available_count in the DB & write a PENDING reservation row; simultaneously set a Redis TTL key.
- Confirm Phase - On payment success, update reservation status to CONFIRMED & publish a booking.confirmed event to Kafka.
- Release Phase - A Kafka consumer or scheduled job listens for TTL expiry & sets status to EXPIRED, incrementing available_count
- Tools : PostreSQL, Redis, Kafka/RabbitMQ, Temporal.io (workflow orchestration)

## Optimistic Locking(Version Column)

- Each inentory row carries a version integer. Update queries include WHERE id = ? and version = ? zero rows updated means a conflict occured.
- Preferred over timestamp - based optimistic lokcing because distributed clocks drift; version numbers are monotonically reliable.
- On conflict, the client retries with exponential back-off(up to 3 attempts ) before returning a "seat no longer available" error.
- Tools:  Hinermate/Spring Data JPA @Version,SQLAlchemy version_id_col,Prisma middleware

## Pessimistic Locking

- Used for high-contention inventory (e.g last few seats of a sold-out show): SELECT ... FOR UPDATE SKIP LOCKED isolates rows without blocking the entire table.
- SKIP LOCKED (PostreSQL 9.5+) allows parallel workers to process other rows without waiting, useful for queue-style seat claim workers. 
- Trade-off: Higher througput cost under low contention vs optimistic; suitable only at the final confirmation step, not during browsing.
- Tools: PostgreQL SELECT FOR UPDATE SKIP LOCKED , MySQL innoDB row-level locks.

## Idempotency Tokens

- Clients generate a UUID (Idempotency-key header) per booking attempt.  The server stores (idempotency_key -> response) in Redis with a 24 hour TTL
- Retried requests return the cached response without re-executing the payments or hold , preventing duplicate charges.
- Tools: Redis(key storage), API Gateway plugins (Kong request-id plugin) , Stripe's built in idempotency key support.

## Double Booking Prevention

- Database-level : UNIQUE constraint on (seat_id, event_id, status = 'CONFIRMED' ) as the final safety nwt
- Application level: Redis distributed lock (SETNX) per seat acquired before any inventory write
- Trade-Off: DB constraint alone causes hard failures under race conditions, Redis lock provides a softer , earlier rejection with a better UX error message.
- Tools: PostgreQL unique partial indexes, Redis Redlock algorithm

## TTL-Based Holds

- Redis keys with TTL serve as automatic hold expiry without requiring a cron sweep for most cases; Keyspace Notifications (notify-keyspace-events KEx) trigger a Lambda/consumer to reconcile the DB
- For multi seat bookings, a single composite key holds all seats atomically using a Lua script
- Tools: Redis Keyspace notifications, AWS Lambda/Node.js consumer

## Dynamic Pricing 

- A Pricing Service evaluates demand signals (current hold count, time-to-event,historical fill rate) & returns a price per request
- Prices are cached in Redis with a short TTL (30 to 60s) to avoid a per-request DB hits while staying near real time
- Trade-off: Aggressive caching can show stale prices; use short TTL + cache invalidation on demand threshold events
- Tools: Redis, Apache FLink(real-time demand aggregation), custom rules engine or ML model

# API Design & Gateway Patterns

## REST API Best Practices

- Resources modeled as nouns POST /bookings, GET /events/{id}/seats, PATCH /bookings/{id}/confirm
- HTTP status codes used semantically; 202 Accepted for async booking initiation; 409 Conflict for double booking; 412 Precondition Failed for optimistic lock failure
- Tools Open API 3.x (Swagger), Spring Boot, Fast API , Express Js

## Backend for Frontend (BFF)

- Separate BFF instances per client type (web BFF,mobile BFF) aggregate calls to inventory, Pricing and Seat Map services into a single response, reducing chatty round-trips
- Web BFF can perform SSR with Next.js mobile BFF returns a leaner payload without seat map SVG data
- Tools: Node.js/Next.js Apollo Server (if GraphQL BFF), GraphQL Federation

## API Gateway

- Handles cross-cutting concerns; JWT validation, rate limiting, IP allow/block lists, request logging, SSL termination, and canary routing
- Routes /api/search/* to Search Service and /api/bookings/* to the Booking BFF, decoupling client from internal topology
- Tools: Kong,AWS API Gateway, NGINX Plus ,Apigee

## Idempotency at the Gateway

- API Gateway stamps a Request-ID on every inbound request; downstream services propogate it for correlation & idempotent replay.
- Tools: Kong correlation-id plugin , AWS API Gateway $context.requestId 

# Communication Protocols

## HTTP/1.1 vs HTTP/2 vs HTTP/3

- HTTP/2 is the baseline for browser to gateway; multiplexing elimiates head of line blocking for concurrent API calls on the event detail page
- HTTP/3 (QUIC) improves mobile experience on lossy networks(important for Yatra/Agoda mobile users) by removing TCP re-transmission stalls.
- HTTP/1.1 is kept as a fallback for legacy load balancers or internal service mesh hops where HTTP/2 overhead is unnessary
- Tools: Cloudflare (HTTP/3) NGINX (HTTP/2), Envon Proxy

## gRPC vs REST vs GraphQL

- gRPC used for inter service communication (inventory - Booking, Booking - Payment): binary Protobuf enconding is ~ 5 to 7 smaller than JSON;bidirectional streaming supports real-time inventory feed between services
- REST used for public facing browser accessible APIs and third party integrations (webhooks,partnerAPIs)
- GraphQL used at the BFF layer for flexible data aggregation; not exposed directly to public
- Trade-Off: gRPC requires Protobuf schema discipline and is not browser-native (needs gRPC Web Proxy); REST is universally accessible
- Tools: gRPC (Go/Java/Node), Protobuf,Envoy (gRPC-Web Proxy), Apollo GraphQL

## Websockets vs SSE vs Long Polling

- SSE preferred for seat map availability updates; unidirectional server - client, HTTP native, auto-reconnect, works through proxies/CDNs without special config
- WebSockets used for interactive features: live bidding on dynamic priced tickets, multi-user collaborative seat selection.
- Long Polling only as a fallback for environments blocking WebSocket upgrades (some corporate firewalls)
- Trade Off: WebSockets require stateful server connections (complicates horizontal scaling - needs sticky sessions or a Redis pub/sub fan-out layer); SSE is stateless- friendly.
- Tools: Socket.IO (WebSocket + fallback), EventSource API(SSE), Redis Pub/Sub for fan-out.

# Real Time & Async Architecture

## Message Brokers 

### Kafka

- Core event stream booking.initiated, payment.authorized, booking.confirmed, hold.expired topics
- Enables event sourcing, audit log & replay for analytics pipelines (Flink/Spark)
- Trade-Off: Higher operational complexity than RabbitMQ ; suited when ordering guarantees and replay are critical.

### RabbitMQ

- Task queues for bounded-context tasks; email dispatch, PDF ticket generation, post-booking webhooks.
- Dead-letter queues handle retries and failures gracefully
- Trade-off: Simpler ops than Kafka; messages are consumed & deleted (no replay)

### Redis Pub/Sub

- Low -latency fan out for real time status changes to all connected SSE/WebSocket clients
- Trade-off: Fire and Forget - no persistence; message lost if subscriber is down. Suitable only for UI refresh signals, not business critical events

Tools: Apache Kafka, RabbitMQ ,Redis ,Confluent Platform, AWS MSK, CloudAMQP.


## Event Driven Architecture 

- Services communicate via domain events rather than direct calls; Booking Service publishes events consumed by Notification, Analytics, and Loyalty services - fully decoupled
- Outbox pattern (write event to DB outbox table in same transaciton, then relay to Kafka) ensures no lost events between DB commit & broker publish
- Tools : Debezium (CDC for outbox), Kafka Connect

## Event Sourcing

- Reservation aggregate state rebuilt by replaying SeatHeld -> PaymentAuthorized -> BookingConfirmed events ; enables full audit trail required for financial compliance
- Trade-Off: Query complexity increases; requires CQRS read models (projected views) for fast reads.
- Tools: EventStoreDB, Axon Framework, custom Kafka -based sourcing

## Eventual Consistency vs Strong Consistency

- Strong consistency required: Seat hold creation, payment capture, final booking confirmation - use synchronous DB writes with distributed locking.
- Eventual consistency acceptable: Notification delivery, loyalty points, analytics dashboards, search index updates
- Tools; PostgreSQL (strong) Elasticsearch (eventual read model), Kafka (async propogation)

# Front end Specific System Design

## Micro-Frontend Architecture

- Seat selection widget, payment form and booking confirmation can be independently deployed micro frontends composed via Module Federation
- Allows separate teams to own & release the payment UI independently of the seat map
- Tools: Webpack Module Federation, Single SPA , NX Monorepo

## Performance at Scale

 - Code splitting per route; seat-map SVG renderd lazily only when user navigates to selection
 - Critical CSS in lined ; non critical scripts deferred; images served via CDN with WebP/AVIF format negotiation
 - Tools: Next.js , Vite, Lighthout CI, Web Vitals monitoring

## Browser & CDN Caching Strategy

- Static assets (JS, CSS, images): immutable cache with content hash filenames (Cache Control max-age=31536000, immutable)
- Event listing pages: CDN cached with stale while revalidate; purged on inventory update events
- Seat availability: no CDN cache; served directly from origin with short-lived Redis cache
- Tools: CloudFront, Fastly, Cloudflare Pages, Nextjs ISR

## State Management at Scale

- Server state(availability, pricing) managed by React Query/SWR with polling or SSE invalidation - avoids duplicating server state in Redux
- Client only UI state(selected seats,step progress) in Zustand or Context API
- Trade-off: React Query simplifies cache invalidation but requires careful stateTime tuning to avoid showing state seat availability
- Tools: TanStack Query (React Query), Zustand, SWR

## SSR/SSG/ISR 

- SSR : Event detail pages for SEO & first load performance; rendered per request with current pricing.
- ISR: Venue listing pages regenerated every 60 seconds; balances freshness with edge caching
- SSG: Static marketing / FAQ pages; fully CDN-served
- Tools: Next.js (all three modes), Vercel, AWS Amplify

## Optimistic UI 

 - When a user selects a seat, the UI immediately marks it as selected & disables it for others without waiting for the hold API response 
 - On hold failure (seat taken), UI rolls back with a clear error message
 - Tools: TanStack Query onMutate/onError rollback, Immer for state snapshots

# Scalability & Reliablity Patterns

## Load Balancing 

- L6 load balancing at the API Gateway with least connections algorithm; health checks auto remove failed pods
- Sticky sessions only for WebSocket connections (hash by user ID); all other services are stateless
- Tools: AWS ALB,NGINX, HAProxy, Kubernetes Ingress (Traefik)

## Database Scaling

 - Write path : Single primary PostgreSQL for bookings(strong consistency); connection poling via PgBounder
 - Read path: Read replicas for search queries , reporting , seat map reads
 - Sharding: Partition by event_id or venue_id for extreme scale
 - Tools: PostgreSQL , PgBouncer, Citus(sharding), AWS Aurora

## Distributed Caching

 - Redis Cluster for horizontal cache scaling; seat maps and pricing cached with short TTL
 - Cache-Aside pattern: application checks Radis first;on miss, loads from DB & populates cache
 - Tools: Redis Cluster, AWS ElastiCache, Dragonfly (Redis-compatible, higher throughput)

## CQRS

- Command side: Booking/Inventory services handle wirtes with strong consistency
- Query side: Elasticsearch or a separate read DB serves search & seat map queries, updated asynchronously via Kaffa consumers
- Trader off: Increases architectural complexity; justified by the read heavy nature (100:1 read to write ration typical)
- Tools: Elasticsearch, Kafka consumers, Spring Data, MediatR(.NET)

## Circuit Breaker

- Booking Service wraps calls to Payment Service with a circuit breaker; on 5 consecutive failures, it opens and returns a graceful "payment unavailable" error.
- Prevents cascading failure during payment provider outages
- Tools: Resilience4j, Hystric(legacy), Polly(.NET),Istio (service mesh level)

## Saga/ Orchestrator

- The booking flow (Hold -> Pay -> Confirm -> Notify) is a distributed spanning multlple services; a Saga orchestrator manages compensating transactions on failure
- Orchestration Saga: Temportal.io workflow explicitly drives each step & compensation (release hold on payment failure)
- Trade-Off vs Choreopgraphy: Orchestration is easire to reason about and debug; choreography is more loosely coupled but harder to trace
- Tools: Temporal.io, Apache Camel, AWS Step Functions

## Backpressure/Bulkhead

- Bulkhead pattern isolates thread pools per downstream dependency(Payment,Inventory) so a slow payment provider doesn't exhaust threads for inventory calls.
- Kafka consumer groups apply natural backpressure; consumers lag rather than overwhelming the DB
- Tools; Resilience4J Bulkhead, Kafka consumer group lag monitoring (Burrow)

## Observer / Reactive Streams

- Inventory updates pushed reactively to connected clients via SSE;back end uses reactive pipeline to fan out to thousands of subscribers
- Tools: Project Reactor(Spring WebFlux), RxJS (frontend),Vert.x

# Security Architecture

## OAuth2/OIDC 

- Users authenticate via an Identity Provider (Keycloak/Auth0); JWT access tokens (short-lived , 15min) & refresh tokens (HttpOnly cookie)
- Service to Service calls use OAuth2 Client Credentials flow; no user tokens forwarded between internal services
- Tools: Keycloak, Auth0; Okta, AWS Cognito

## Zero Trust Architecture

- Every internal service call is authenticated & authorized; no implicit trust based on network location
- mTLS enforced between all microservices via a service mesh; certificats rotated automatically
- Tools: Istio/Linkerd(mTLS), SPIFFEE/SPIRE(Identity), HashiCorp Vault

## Frontend Security

- Content Security Policy (CSP) headers prevent CSS; Referrer-Policy: strict-origin on all pages
- Payment from isolated in a sandboked iframe(Stripe Elements/ Braintree Drop-in) - card data never touches the application server
- Tools: Stripe Elemetns, Helmet.js (Node CSP Headers), OWASP ZAP(scanning)

## Secrets Management 

- No secrets in source code or environment variables directly; all credentials fetched from a secret manager at runtime
- Dynamic DB credentials rotated automatically; short-lived
- Tools: HashiCorp Valut, AWS Secrets Manager, Azure Key Value

## Api Security 

- Input validation and schema enforment at the API Gateway layer (reject malformed requests early)
- OWASP Top 10 mitigations; parmaterised queries (SQLi), output encoding (XSS), CSRF tokens for state-changing POST requests
- ToolsK Kong (request validation plugin), express -validator , Zod, OWASP Dependency- Check

## Rate Limiting

- Per-user & per IP rate limits enforced at the API Gateway; booking endpoints limited to 5 req/min per user to prevent seat hold abuse
- Sliding window algorithm perferred over fixed window to smooth burst traffic
- Trade-Off: Too aggressive limits frustrate legitimate users during high-demand drops; use token bucket for burst tolerance
- Tools: Kong Rate Limiting plugin, Redis (sliding window counters), AWS WAF

## Optimistic Locking(Security/Integrity)

- Version column prevents a compromised or replayed request from overwriting a newer booking state(e.g replaying a PENDING state over a CONFIRMED booking)
- Combined with JWT list claim validation to reject stale tokens.


# Recommended End to End Booking flow

1. Search - User queries available events/rooms; Search Service queries Elastisearch; CDN cached results returned via BFF with current pricing from Pricing Service
2. Availability - Seat map loaded; Inventory Service returns real-time availability (Redis-backed); SSE channel opened for live seat status updates
3. Seat Selection - User selects seat; optimistic UI marks it as selected immediately
4. Hold - POST/bookings with Idempotency-Key: Booking Service acquires Redis distributed lock on seat, writes PENDING reservation, recrements available_count; TTL set to 10 min
5. Payment Authorization - BFF renders Strip Elements iframe; user submits card details directly to Stripe; Stripe returns  a PaymentIntent client secrret; client confirms payment
6. Payment Capture - Stripe webhook (payment_intent.succeded) received by payment service; event published to Kafka payment.authorized topic
7. Confirm - Booking Service consumes event; updates reservation to CONFIRMED; DB version incremented; Redis hold key deleted. 
8. Notification - Notification Service consumes booking.confirmed; sends email via SendGrid and push via Firebase; PDF ticket generated asynchronously.
9. Analytics - Analytics consumer updates real-time dashboards; Loyalty Service awards points;  Search index refreshed via Debezium CDC


# Key Trade-Offs


| **Decison**        | **Option A**             | **Option B**                    | **Recommendation**                                                                      |
| ------------------ | ------------------------ | ------------------------------- | --------------------------------------------------------------------------------------- |
| Consitency         | Strong (synchronous DB)  | Eventual (async evcents)        | Strong for holds/payments; eventful for notifications/analytics                         |
| Locking            | Optimistic (version col) | Pessimistic (SELECT FOR UPDATE) | Optimistic for general inventory; pessimistic for last seat scenarios                   |
| API Style          | REST                     | GraphQL                         | REST for public APIs; GraphQL at BFF for complex aggregation                            |
| Real-Time          | WebSockets               | SSE                             | SSE for seat availability(unidirectional); WebSokcets for Interactive features          |
| Broker             | Kafka                    | RabbitMQ                        | Kafka for event streaming/audit, RabbitMQ for taks queues                               |
| Cache risk         | High TTL (performance)   | Low TTL(freshness)              | Short TTL (30s) for inventory; long TTL for static content                              |
| Architecture       | Monolith                 | Microservices                   | Microservices with domain boundaries (Booking, Inventory, Payment, Notification)        |
| Saga style         | Orchestration (temporal) | Chorepgraphy (events)           | Orchestration - easier to trace and compensate in financial flows                       |
| Hold expiry        | Redis TTL only           | DB + Redis                      | DB as source of truth; Redis for fast checks; Keyspace Notifications for reconciliation |
| Frontend rendering | CSR(SPA)                 | SSR/ISR (Next.js)               | SSR for event pages(SEO);CSR for seat selection (interactivity)                         |


