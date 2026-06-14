---
aliases:
  - General System Design Topics
---

## Communication Protocols

Topic: 
HTTP/1.1 vs HTTP/2 vs HTTP/3

What to Know: 
Multiplexing, head-of-line blocking, header compression (HPACK/QPACK), QUIC protocol, 0-RRT handshake
Know when HTTP/3 actually helps vs adds complexity

Topic: 
gRPC vs REST vs GraphQL

What to Know: 
Protobuf binary encoding, bidirectional streaming, shema-first design. 
GraphQL N+1 problem, DataLoader, subscriptions. When to pick each

Topic: 
SSE vs WebSockets vs Long Polling vs Short Polling

What to Know: 
SSE = server-push, HTTP, unidirectional, auto-reconnect
WebSocket = full-duplex, stateful connection management overhead
Long Polling = fake push. Puck based on; directionality, connection state,
proxy/firewall constraints

Topic: 
WebRTC 

What to Know:
P2P data channels, ICE/STUN/TURN
Not for LLM - but relevant for real-time collaboration features


## Real-Time & Async Architecture

Topic:
Message Brokers (Kafka vs RabbitMQ vs Redis Pub/Sub)

What to Know:
Kafka: durable log, replay, high-througput, consumer groups, partitions.
RabbitMQ: routing,dead-letter queues, push-based
Redis Pub/Sub: ephemeral, in-memory, no reply



Topic:
Event Driven Architecture

What to Know:
Event producer/consumer decoupling, event schema contracts, event versioning, choregraphy vs orchestration


Topic:
Event Sourcing

What to Know:
Store events not state,rebuild state by replaying.
Append-only log.
Snapshots for performance.

Topic:
CQRS in practice 

What to Know:
Separate read model (optimized for queries) from write model (commands)
Eventual consistency implications


Topic:
Eventual Consistency vs Strong Consistency


What to Know:
CAP theorem, BASE vs ACID, when to choose each, saga pattern for distributed transactions

Topic:
Webhook Design

What to Know:
Retry semantics, idempotency keys, signature verification ( HMAC ) delivery guarantees


## Classic System Design Problems

Problem:
Reservation & Booking System

Core Concepts to Master
Inventory Holds,
2 phase commit alternatives
optimistic locking (version column)
pessimistic locking
idempotency tokens,
double-booking prevention
TTL based holds


Problem:
Notification System

Core Concepts to Master
Fan-out on write vs fan-out on read
priority queues
multi channel (email/SMS/push)
rate limiting per channel
deduplication

Problem:
Rate Limiter

Core Concepts to Master:
Token bucket
leaky bucket
sliding window log
sliding window counter
Distributed rate limiting with Redis

Problem:
URL Shortener/ Unique ID Generator

Core Concepts to Master:
Hash collision handling, 
base62 encoding
Snowflake IDs
distributed counters

Problem:
Search Autocomplete

Core Concepts to Master:
Trie vs inverted index
prefix search
debounce vs throttle on the client
caching hot prefixes

Problem:
File Upload at Scale

Core Concepts to Master:
Chunked upload,
pre-signed URLs(W3)
multipart,
virus scanning,
metadata extraction pipeline

Problem:
Real-Time Dashboard/Feed

Core Concepts to Master:
Push vs Pull
SSW/WebSocket
Time Series Storage
Aggregation Windows
Read replica for queries

Problem:
Authentication System

Core Concepts to Master:
OAuth2/ OIDC flows
JWT vs opaque tokens
refresh token rotation
session management
SSO/SAML for enterprise


## API Design & Gateway Patterns

Topic:
REST API best practices

What to know:
Resource naming, versioning strategies (URI vs header vs content negotiation)
HATEOAS (know it exists, know why its rarely used in practice)
Pagination(cursor vs offset vs keyset)

Topic:
Backend for Frontend (BFF)

What to know:
One BFF per client type(mobile/web)
Aggregates microservices
Owns response shaping

Topic:
API Gateway Paterns

What to know:
Auth offload
rate limiting
request routing
response transformation
circut breaker at gateway layer
Kong, APIM, AWS API Gateway

Topic:
GraphQL in production

What to know:
Persisted queries
depth limiting
cost analysis
DataLoader for N+1 
schema stitching vs federation

Topic:
Idempotency

What to know:
Idempotency keys for POST requests
why it matters for retries
how to implement server-side deduplication


## Frontend Specific System Design

Topic:
Micro Frontend Architecture

What to know:
Module Federation(webpack5)
single-spa
independent deployability
shared dependencies pitfall
routing strategies
cross-MFE comunication

Topic:
Performance at scale

What to know:
Core Web Vitals(LCP,CLS,FIS/INP)
critical rendering path
code splitting
lazy loading
tree shaking
resource hints(preload,prefetch, preconnect)

Topic:
Caching Strategy(browser + CDN)

What to know:
Cache-Control directives
ETag
Service Worker caching strategies(cahce-first, network-first, stale-while-revalidate)
CDN invalidation



Topic:
State Management at Scale

What to know:
NgRx signals architecture
selector memoization
normalizing state(entity adapter)
optimistic updates
state hydration for SSR


Topic:
Server Side Rendering SSR/SSG/ISR

What to know:
Angular Universal
hydration
trade-offs
TTFB vs interactivity
when SSR helps for SEO & LCP


Topic:
Offline-First & PWA

What to know:
Indexed DB, Cache API,
background sync
conflict resolution strategies for offline mutations

Topic:
Design System Architecture

What to know:
Token based themeing
component versioning 
accessibility(WCAG 2.1, AA)
monorepo tooling for shared component libraries


## Scalability & Reliability Patterns


Topic:
Load Balancing

What to know:
L4 vs L7
round robin vs least connections vs consistent hashing
Sticky session - when & why they are a problem

Topic:
Database Scaling

What to know:
Read replicas, sharding strategies (range vs hash)
connection pooling
N+1 in ORMs
index design
when not to normalize


Topic:
Distributed Caching

What to know:
Redis cluster
cache stampede (mutex/probabilistic early expiry)
cache aside vs write through vs write behind

Topic:
Resilience Patterns

What to know:
Retry with exponential backoff + jitter
circuit breaker states(closed / open / half-open)
bulkhead
timeout hierarchy
graceful degradation

Topic:
CDN & Edge architecture

What to know:
Edge caching, cache invalidation & scale
edge functions(Cloudflare Workers, Lambda@Edge)
dynamnic vs static content split


Topic:
Database Transaction Patterns

What to know:
ACID guarantees,
isolation levels(read committed vs repeatable read vs serializable)
optimistic vs pessimistic locking
2-phase locking


## Security Architecture

Topic:
OAuth2/OIDC in depth

What to know:
Authorization Code + PCKE flow
implicit flow
client credentials
token introspection
refresh token strategies

Topic:
Zero Trust Architecture

What to know:
Never trust
always verify
mTLS between services
service mesh basics(Istio,Linkerd)

Topic:
Frontend Security

What to know:
CSP headers, XSS prevention, CSRF tokens vs Same site cookies
CORS configuration, SubResource Integrity(SRI)

Topic:
Secret Management

What to know:
HashiCorp Valut, Azure Key Value, secret rotation, avoiding secrets in client-side code


Topic:
API Security

What to know:
Input  Validation, parameterized queries, 
OWASP API Security top 10
schema validation

