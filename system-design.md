# System Design

## Table of Contents
1. [Interview Framework](#interview-framework)
2. [Availability, SLA, SLO, SLI](#availability-sla-slo-sli)
3. [Scalability](#scalability)
4. [CAP Theorem & PACELC](#cap-theorem--pacelc)
5. [Load Balancing](#load-balancing)
6. [Caching Strategies](#caching-strategies)
7. [Database Sharding & Replication](#database-sharding--replication)
8. [Consistent Hashing](#consistent-hashing)
9. [Designing a Queryable RESTful API](#designing-a-queryable-restful-api)
10. [Design: URL Shortener](#design-url-shortener)
11. [Design: Rate Limiter](#design-rate-limiter)
12. [Design: Notification System](#design-notification-system)
13. [Design: Chat System](#design-chat-system)

---

## Interview Framework

**5-step framework:**

```
1. CLARIFY REQUIREMENTS (5 min)
   ├── Functional: What must the system do? Core features only (no gold-plating)
   ├── Non-functional: Scale? Latency? Availability? Consistency?
   └── Constraints: Budget? Tech stack? Geographic regions?

2. BACK-OF-ENVELOPE ESTIMATION (3 min)
   ├── Users: DAU, MAU, concurrent users
   ├── Traffic: reads/sec, writes/sec, read:write ratio
   ├── Storage: object size × daily volume × retention
   └── Bandwidth: QPS × average response size

3. HIGH-LEVEL DESIGN (10 min)
   ├── Draw boxes: clients, API gateway, services, DBs, caches
   ├── Data flow: how does a request flow end-to-end?
   └── APIs: list 3-5 key endpoints

4. DEEP DIVE (15 min)
   ├── Pick 2-3 critical components to detail
   ├── Data models and schemas
   ├── Algorithms (e.g., consistent hashing, Bloom filter)
   └── Edge cases and failure modes

5. BOTTLENECKS & SCALE (5 min)
   ├── Where does it break at 10× scale?
   ├── Caching opportunities
   ├── DB sharding
   └── Trade-offs discussed
```

**Always discuss trade-offs.** No perfect design — show you understand the implications.

**Estimation numbers to know:**
| Item | Value |
|---|---|
| Memory access | ~100ns |
| SSD random read | ~100μs |
| Network round-trip (same region) | ~500μs |
| Disk seek | ~10ms |
| 1MB over network | ~10ms |
| QPS one machine can handle (simple) | ~50k-100k HTTP req/s |
| Read from DB (indexed) | ~1ms |

---

## Availability, SLA, SLO, SLI

| Term | Definition |
|---|---|
| **SLA** (Service Level Agreement) | Legal/contractual commitment to customers (e.g., 99.9% uptime) |
| **SLO** (Service Level Objective) | Internal target, stricter than SLA (e.g., 99.95% internally to have buffer) |
| **SLI** (Service Level Indicator) | Actual measured metric (e.g., percentage of successful requests over 30 days) |
| **Error Budget** | 1 - SLO = how much "badness" is allowed. 99.9% SLO = 8.76 hours/year error budget |

| Nines | Downtime/Year | Downtime/Month | Downtime/Week |
|---|---|---|---|
| 99% (two 9s) | 3.65 days | 7.31 hours | 1.68 hours |
| 99.9% (three 9s) | 8.76 hours | 43.83 min | 10.08 min |
| 99.99% (four 9s) | 52.60 min | 4.38 min | 1.01 min |
| 99.999% (five 9s) | 5.26 min | 26.30 sec | 6.05 sec |

**Achieving high availability:**
- Redundancy: active-active (both serve traffic) or active-passive (standby)
- Health checks + automatic failover
- Multi-AZ / multi-region deployment
- Circuit breakers to isolate failures
- Graceful degradation (serve cached/stale data vs complete failure)

---

## Scalability

### Horizontal vs Vertical Scaling

| | Vertical (Scale-Up) | Horizontal (Scale-Out) |
|---|---|---|
| Approach | Bigger machine (more CPU, RAM, SSD) | More machines |
| Limit | Hardware ceiling (~hundreds of cores) | Practically unlimited |
| Cost | Non-linear — very expensive at top end | Linear with commodity hardware |
| Failure mode | Single point of failure | Fault tolerant |
| Complexity | Simple (no code changes) | Requires statelessness, LB, distributed state |
| Best for | Databases, stateful services | Stateless web/API services |

**Making services horizontally scalable:**
1. **Stateless** — no local session data. Store sessions in Redis.
2. **Shared nothing** — each instance reads/writes from same external stores (DB, cache).
3. **Externalized config** — no hardcoded IPs or env-specific code.

```mermaid
flowchart LR
    LB[Load Balancer] --> S1[API Instance 1]
    LB --> S2[API Instance 2]
    LB --> S3[API Instance 3]
    S1 & S2 & S3 --> DB[(Database)]
    S1 & S2 & S3 --> Cache[(Redis)]
```

---

## CAP Theorem & PACELC

**CAP:** A distributed system can guarantee at most 2 of 3: **Consistency, Availability, Partition Tolerance**.

Since network partitions are unavoidable, real systems choose between **CP** or **AP**:

| | CP Systems | AP Systems |
|---|---|---|
| Examples | Zookeeper, etcd, HBase, MongoDB (default) | Cassandra, DynamoDB, CouchDB |
| Partition behavior | Return error (consistent but unavailable) | Return possibly stale data (available but inconsistent) |
| Use case | Leader election, config, financial | Shopping carts, social feeds, DNS |

**PACELC** extends CAP: even without partitions, there's a trade-off between **Latency (L)** and **Consistency (C)**:
- DynamoDB: PA/EL (available + low latency)
- Spanner: PC/EC (consistent at the cost of latency)

**Consistency models (weakest to strongest):**
- **Eventual consistency** — replicas converge given no new writes
- **Monotonic reads** — never see older data than you've seen before
- **Read-your-writes** — you always see your own writes
- **Causal consistency** — causally related operations seen in order
- **Linearizability** — strongest; operations appear instantaneous; real-time ordered

---

## Load Balancing

**Algorithms:**

| Algorithm | Description | Use case |
|---|---|---|
| Round Robin | Cycle through servers in order | Homogeneous servers |
| Weighted Round Robin | More requests to more powerful servers | Heterogeneous servers |
| Least Connections | Route to server with fewest active connections | Long-lived connections |
| IP Hash | Hash client IP → always route to same server | Session stickiness |
| Least Response Time | Route to fastest-responding server | Latency-sensitive |
| Random | Pick randomly | Simple, works well at scale |

**Layers:**
- **L4 (Transport):** Load balance TCP/UDP — fast, no HTTP awareness (AWS NLB)
- **L7 (Application):** Load balance HTTP — content-based routing, SSL termination, WAF (AWS ALB, Nginx)

**Health checks:** active (LB pings `/health`) or passive (monitor for failures). Unhealthy instances removed from rotation.

---

## Caching Strategies

```mermaid
flowchart LR
    App --> Cache[(Cache\nRedis)]
    Cache -->|miss| DB[(Database)]
    DB --> Cache
    Cache --> App
```

### Cache-Aside (Lazy Loading) — Most Common
```
Read: Check cache → if miss: load from DB → store in cache → return
Write: Update DB → invalidate cache (or update cache)
```
```java
public Product getProduct(Long id) {
    return cache.get(id, () -> db.findById(id)); // Spring @Cacheable does this
}
```
**Pros:** only caches data that's actually requested. **Cons:** cache miss = extra latency; stale data possible.

### Write-Through
Write to cache AND DB synchronously. Cache always up-to-date.  
**Pros:** no stale reads. **Cons:** write latency, data not yet requested wastes cache space.

### Write-Behind (Write-Back)
Write to cache; async flush to DB. Fast writes.  
**Pros:** very fast writes. **Cons:** risk data loss if cache dies before flush.

### Read-Through
Cache sits in front; fetches from DB on miss transparently.  
**Pros:** simple app code. **Cons:** cold start problem.

### Cache Eviction Policies
| Policy | Description |
|---|---|
| LRU (default) | Evict least recently used |
| LFU | Evict least frequently used |
| TTL | Time-based expiration |
| FIFO | Evict oldest inserted |

**Cache stampede prevention:** when a cache key expires, N threads all hit DB simultaneously.
```java
// Probabilistic early expiration or mutex-based prevention
String cached = redis.get(key);
if (cached == null) {
    if (redis.setnx("lock:" + key, "1", 5, SECONDS)) { // only one thread re-fills
        try {
            String value = db.fetch(key);
            redis.setex(key, 3600, value);
        } finally { redis.del("lock:" + key); }
    } else {
        Thread.sleep(50); // brief wait, then retry
        return redis.get(key);
    }
}
```

**CDN caching:** static assets (images, JS, CSS) cached at edge locations closest to user. Reduces latency and origin load.

---

## Database Sharding & Replication

### Replication (read scalability + high availability)

```mermaid
flowchart LR
    W[Write] --> Primary[(Primary DB)]
    Primary -->|async/sync replication| R1[(Replica 1)]
    Primary --> R2[(Replica 2)]
    R[Read] --> R1
    R --> R2
```

- **Sync replication:** write confirmed after at least one replica acknowledges → zero data loss, higher write latency
- **Async replication:** write confirmed immediately → lower latency, possible replication lag

### Sharding (write scalability)

Partition data across multiple DB nodes by a **shard key**.

```mermaid
flowchart TD
    App --> Router[Shard Router]
    Router -->|user_id % 3 = 0| S0[(Shard 0\nusers 0-33%)]
    Router -->|user_id % 3 = 1| S1[(Shard 1\nusers 34-66%)]
    Router -->|user_id % 3 = 2| S2[(Shard 2\nusers 67-100%)]
```

**Sharding strategies:**
| Strategy | Description | Pros/Cons |
|---|---|---|
| Range sharding | `user_id 0-999 → Shard 0` | Simple; can hotspot if data not uniform |
| Hash sharding | `hash(key) % N` | Even distribution; no range queries |
| Directory/Lookup | Mapping table: key → shard | Flexible; lookup table is bottleneck |
| Geo sharding | Route by geography | Low latency for regional data |

**Sharding challenges:**
- Cross-shard joins — expensive/impossible
- Distributed transactions — complex
- Rebalancing when adding shards — requires data migration
- Non-uniform key distribution → hotspots

---

## Consistent Hashing

Minimizes remapping when nodes are added/removed (vs modulo hashing where `hash % N` remaps almost everything).

```
Ring: 0 ────────────────── 2^32-1
             S1    S2    S3
              ↑     ↑     ↑   (servers on ring)
      K1   K2    K3   K4     (keys map to next server clockwise)
```

Adding a server: only the keys between it and its predecessor remap (1/N keys on average).  
Removing a server: only its keys move to successor.

**Virtual nodes:** each server gets multiple positions on the ring → better load distribution.

Used in: Cassandra, DynamoDB, Riak, CDN routing.

---

## Designing a Queryable RESTful API

**My approach when designing a queryable REST API:**

### 1. Resource-Oriented URL Design
```
GET    /api/v1/orders              → list orders (paginated)
GET    /api/v1/orders/{id}         → get single order
POST   /api/v1/orders              → create order
PUT    /api/v1/orders/{id}         → full update
PATCH  /api/v1/orders/{id}         → partial update
DELETE /api/v1/orders/{id}         → delete order
GET    /api/v1/orders/{id}/items   → sub-resources
```
- Nouns, not verbs. Resources, not actions.
- Plural names (`/orders` not `/order`)
- Hierarchical for sub-resources

### 2. Filtering, Sorting, Pagination — Query Parameters

```
GET /api/v1/orders?
    status=PENDING                 → filter
    &customerId=42                 → filter
    &totalMin=100&totalMax=1000    → range filter
    &createdAfter=2024-01-01       → date filter
    &sort=createdAt,desc           → sort (field,direction)
    &page=0&size=20                → pagination (0-indexed)
    &fields=id,status,total        → sparse fieldsets
```

**Never return unbounded lists.** Always paginate with a reasonable default and max limit.

### 3. Response Envelope (Consistent Structure)

```json
{
  "data": [
    { "id": 1, "status": "PENDING", "total": 150.00 }
  ],
  "meta": {
    "page": 0,
    "size": 20,
    "totalElements": 150,
    "totalPages": 8
  },
  "links": {
    "self": "/api/v1/orders?page=0&size=20",
    "next": "/api/v1/orders?page=1&size=20",
    "prev": null
  }
}
```

### 4. Error Responses — RFC 7807 Problem Details
```json
{
  "type": "https://api.company.com/errors/validation-error",
  "status": 400,
  "title": "Validation Error",
  "detail": "order.items must not be empty",
  "traceId": "abc-123-def",
  "timestamp": "2024-01-15T10:30:00Z",
  "errors": [
    { "field": "items", "message": "must not be empty" }
  ]
}
```

### 5. Versioning Strategy

- **URL versioning:** `/api/v1/orders` — most common, explicit, easy to route
- **Header versioning:** `Accept: application/vnd.company.v1+json` — clean URLs, harder to test in browser
- **Query param:** `/api/orders?version=1` — messy

**Breaking vs non-breaking changes:**
- Non-breaking (safe): add optional field, add new endpoint, expand enum
- Breaking (new version): remove field, rename field, change type, change semantics

### 6. Performance for Queryable APIs

```mermaid
flowchart LR
    Client -->|GET /orders?status=PENDING| GW[API Gateway\nRate limit\nAuth]
    GW --> API[Order Service]
    API -->|check| Cache[(Redis\nTTL-based cache)]
    Cache -->|miss| DB[(PostgreSQL)]
    DB -->|indexed scan| API
    API --> Client
```

- **DB indexes on all filterable fields**: `status`, `customerId`, `createdAt`, composite indexes for common filter combos
- **Redis caching** for popular queries (short TTL + invalidation on write)
- **Projections** — SELECT only needed columns (avoid `SELECT *`)
- **Cursor-based pagination** for large datasets (avoids `OFFSET N` performance degradation)
- **Rate limiting** per API key
- **ETags** for conditional GET (304 Not Modified)

### 7. Spring Boot Implementation

```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    @GetMapping
    public ResponseEntity<Page<OrderResponse>> list(
            @RequestParam(required = false) OrderStatus status,
            @RequestParam(required = false) Long customerId,
            @RequestParam(required = false) @DateTimeFormat(iso = ISO.DATE) LocalDate createdAfter,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") @Max(100) int size,
            @RequestParam(defaultValue = "createdAt,desc") String sort) {

        Sort sortSpec = parseSortParam(sort);
        Pageable pageable = PageRequest.of(page, size, sortSpec);

        Specification<Order> spec = Specification.where(null);
        if (status != null) spec = spec.and(OrderSpecs.hasStatus(status));
        if (customerId != null) spec = spec.and(OrderSpecs.hasCustomer(customerId));
        if (createdAfter != null) spec = spec.and(OrderSpecs.createdAfter(createdAfter));

        return ResponseEntity.ok(orderRepo.findAll(spec, pageable).map(orderMapper::toResponse));
    }
}
```

---

## Design: URL Shortener

**Requirements:** shorten URLs, redirect short URLs, analytics (optional). 100M URLs/day.

```mermaid
flowchart LR
    Client -->|POST /shorten {url}| API[API Service]
    API -->|store mapping| DB[(Key-Value Store\nDynamoDB / Redis)]
    API -->|return https://sho.rt/abc123| Client

    Browser -->|GET /abc123| API
    API -->|lookup abc123| DB
    DB -->|original URL| API
    API -->|301 Redirect| Browser
```

**Key design decisions:**
1. **Short code generation:** base62 encoding of auto-incremented ID or random 7 chars
   - `hash(url)` → collision risk → not ideal
   - `ID → base62(ID)` → predictable, easy to distribute
   - Random 7 chars (62^7 ≈ 3.5 trillion) → safe
2. **301 vs 302 redirect:** 301 (permanent, browser caches) vs 302 (temporary, every click hits server for analytics)
3. **Storage:** ~500 bytes per mapping × 100M/day × 30 days = ~1.5 TB/month
4. **Read optimization:** cache hot short codes in Redis (80/20 rule)

---

## Design: Rate Limiter

**Goal:** limit requests per user/IP/API key to protect backend.

**Algorithms:**

| Algorithm | Allows Bursts? | Memory | Accuracy |
|---|---|---|---|
| Token Bucket | Yes | Low | Good |
| Leaky Bucket | No (smoothed) | Low | Good |
| Fixed Window Counter | Yes (at boundaries) | Very low | Approximate |
| Sliding Window Log | No | High | Exact |
| Sliding Window Counter | Partially | Low | Good |

```mermaid
flowchart LR
    Client -->|Request + API key| LB[Load Balancer]
    LB --> RL[Rate Limiter\nmiddleware]
    RL -->|check/decrement| Redis[(Redis\nAtomic counters)]
    Redis -->|allowed| Service[Backend Service]
    Redis -->|429 Too Many Requests| Client
```

**Token Bucket in Redis (atomic Lua script):**
```lua
-- Arguments: key, capacity, refill_rate, cost
local tokens = tonumber(redis.call('GET', KEYS[1])) or ARGV[1]
local now = tonumber(ARGV[2])
-- refill tokens based on elapsed time
tokens = math.min(ARGV[1], tokens + (now - last_time) * ARGV[2])
if tokens >= ARGV[3] then
    tokens = tokens - ARGV[3]
    redis.call('SET', KEYS[1], tokens)
    return 1  -- allowed
else
    return 0  -- rejected
end
```

**Distributed rate limiting:** all gateway instances share Redis counters → consistent limits across nodes.

---

## Design: Notification System

**Types:** push (mobile), email, SMS, in-app. Scale: 10M notifications/day.

```mermaid
flowchart TD
    API[Notification API] -->|publish| Kafka[(Kafka\nnotifications topic)]
    Kafka --> Worker[Notification Workers\nConsumer Group]
    Worker -->|route by type| Push[APNs / FCM\nPush Service]
    Worker --> Email[SMTP / SendGrid]
    Worker --> SMS[Twilio / SNS]
    Worker --> DB[(DB\nnotification log)]
```

**Key considerations:**
- **Priority queues:** critical alerts vs marketing (separate Kafka topics/partitions)
- **Deduplication:** idempotency key prevents sending same notification twice
- **Rate limiting per user:** max 3 SMS/hour per user
- **Retry with backoff:** failed deliveries retry with exponential backoff
- **Dead letter queue:** after N retries, move to DLQ for manual inspection
- **User preferences:** respect opt-out, do-not-disturb hours, channel preference

---

## Design: Chat System

**Requirements:** 1-to-1 and group chat, online status, message history.

```mermaid
flowchart TD
    Client -->|WebSocket| WSS[WebSocket Server]
    Client2 -->|WebSocket| WSS2[WebSocket Server]
    WSS -->|publish| Kafka[(Kafka)]
    Kafka --> WSS2
    WSS -->|store| MsgDB[(Message DB\nCassandra)]
    WSS -->|update presence| Cache[(Redis\nOnline Status)]
```

**Key design decisions:**
1. **WebSocket** for real-time; fallback to long polling
2. **Message delivery:** publisher → Kafka → subscriber's server → WebSocket
3. **Message storage:** Cassandra for write-heavy time-series (messages by conversation + time)
4. **Online status:** Redis with TTL (heartbeat every 5s updates TTL)
5. **Message ordering:** Snowflake ID (timestamp + machine ID + sequence) — time-sortable, globally unique
6. **Group chat fan-out:** for large groups (>1000 members), don't fan-out at send time — pull on read
7. **Read receipts:** separate event stream, async
8. **End-to-end encryption:** keys managed on client, server never has plaintext
