# System Design Interview Questions

## Table of Contents
1. [System Design Fundamentals](#system-design-fundamentals)
2. [Scalability Concepts](#scalability-concepts)
3. [CAP Theorem & Consistency](#cap-theorem--consistency)
4. [Load Balancing](#load-balancing)
5. [Caching Strategies](#caching-strategies)
6. [Database Design & Sharding](#database-design--sharding)
7. [Content Delivery Networks](#content-delivery-networks)
8. [Message Queues & Async Processing](#message-queues--async-processing)
9. [Common System Design Problems](#common-system-design-problems)
10. [Estimation & Capacity Planning](#estimation--capacity-planning)

---

## System Design Fundamentals

### Q1: What are the key principles of system design?
**Answer:**

| Principle | Description | Metrics |
|-----------|-------------|---------|
| **Reliability** | System works correctly even when faults occur | MTBF, MTTR |
| **Availability** | System is operational when needed | Uptime % (99.9%, 99.99%) |
| **Scalability** | Handle increased load gracefully | QPS, response time under load |
| **Performance** | Fast response times, high throughput | Latency (p50, p95, p99), throughput |
| **Maintainability** | Easy to operate, debug, and modify | Deployment frequency, incident resolution time |
| **Security** | Protect data and systems | Compliance, breach incidents |

**Availability SLA Examples:**

| SLA | Downtime/Year | Downtime/Month | Downtime/Week |
|-----|---------------|----------------|---------------|
| 99% (two 9s) | 3.65 days | 7.31 hours | 1.68 hours |
| 99.9% (three 9s) | 8.76 hours | 43.83 min | 10.08 min |
| 99.99% (four 9s) | 52.60 min | 4.38 min | 1.01 min |
| 99.999% (five 9s) | 5.26 min | 26.30 sec | 6.05 sec |

### Q2: How to approach a system design interview?
**Answer:**

**Framework (RESHADED):**

```
Step 1: REQUIREMENTS (5-10 min)
├── Functional: What should the system do?
├── Non-functional: Scale, latency, availability?
├── Constraints: Budget, timeline, team size?
└── Out of scope: What are we NOT building?

Step 2: ESTIMATION (5 min)
├── Users: DAU, MAU, peak concurrent
├── Traffic: QPS (read/write ratio)
├── Storage: Data size × retention
└── Bandwidth: Traffic × data size

Step 3: HIGH-LEVEL DESIGN (10-15 min)
├── Draw major components
├── Show data flow
├── Identify APIs
└── Choose databases

Step 4: DETAILED DESIGN (15-20 min)
├── Deep dive on 2-3 critical components
├── Data models and schemas
├── Algorithms and data structures
└── API contracts

Step 5: SCALE & OPTIMIZE (10 min)
├── Identify bottlenecks
├── Add caching, CDN, load balancing
├── Database scaling (sharding, replication)
└── Discuss trade-offs

Step 6: MONITORING & OPERATIONS
├── Logging and metrics
├── Alerting
├── Deployment strategy
└── Disaster recovery
```

### Q3: What questions should you ask at the start?
**Answer:**

**Functional Requirements:**
- What are the core features? (MVP vs full product)
- Who are the users? (B2C, B2B, internal)
- What actions can users perform?
- What data needs to be stored?

**Non-functional Requirements:**
- What's the expected scale? (users, requests/second)
- What's the acceptable latency? (real-time vs batch)
- What's the availability requirement? (99.9%? 99.99%?)
- Consistency requirements? (strong vs eventual)

**Constraints:**
- Is this a new system or adding to existing?
- Any technology preferences or restrictions?
- Geographic distribution requirements?
- Compliance/security requirements (GDPR, HIPAA)?

---

## Scalability Concepts

### Q4: What is horizontal vs vertical scaling?
**Answer:**

| Aspect | Vertical (Scale-up) | Horizontal (Scale-out) |
|--------|---------------------|------------------------|
| **Method** | Bigger machine (CPU, RAM) | More machines |
| **Limit** | Hardware ceiling | Practically unlimited |
| **Complexity** | Simple, no code changes | Complex, distributed systems |
| **Cost** | Expensive at high end | Cost-effective |
| **Downtime** | Usually required | Can scale live |
| **Failure** | Single point of failure | Fault tolerant |
| **Best for** | Databases, stateful apps | Stateless web servers |

```
Vertical Scaling:
┌─────────────────────┐
│  Before     After   │
│  ┌─────┐   ┌─────┐  │
│  │ 4GB │   │32GB │  │
│  │ 2CPU│   │16CPU│  │
│  └─────┘   └─────┘  │
└─────────────────────┘

Horizontal Scaling:
┌─────────────────────────────────┐
│  Before          After          │
│  ┌─────┐   ┌─────┐┌─────┐┌─────┐│
│  │ App │   │ App ││ App ││ App ││
│  └─────┘   └─────┘└─────┘└─────┘│
└─────────────────────────────────┘
```

### Q5: What makes a system scalable?
**Answer:**

**Key Properties:**

1. **Statelessness**: No session data on servers
```java
// Bad: Stateful
public class ShoppingCart {
    private static Map<String, List<Item>> carts = new HashMap<>();
}

// Good: Stateless (store in Redis)
public class ShoppingCartService {
    private RedisTemplate<String, Cart> redis;
    
    public Cart getCart(String userId) {
        return redis.opsForValue().get("cart:" + userId);
    }
}
```

2. **Loose Coupling**: Independent components
3. **Asynchronous Processing**: Queue work for later
4. **Data Partitioning**: Distribute data across nodes
5. **Caching**: Reduce database load
6. **Load Balancing**: Distribute traffic evenly

### Q6: What is consistent hashing?
**Answer:**

Consistent hashing minimizes remapping when nodes are added/removed.

**Traditional Hashing Problem:**
```
hash(key) % N servers
If N changes: almost all keys remap (bad for caches!)
```

**Consistent Hashing:**
```
Ring: 0 ────────────────────────── 2^32-1
            ↓
      ┌─────────────┐
     /               \
    S1    S2    S3    S4    ← Servers mapped to ring
    │     │     │     │
    K1    K2    K3    K4    ← Keys go to next server clockwise
```

**Implementation:**
```java
public class ConsistentHash<T> {
    private final TreeMap<Long, T> ring = new TreeMap<>();
    private final int virtualNodes;
    
    public ConsistentHash(int virtualNodes) {
        this.virtualNodes = virtualNodes;
    }
    
    public void addNode(T node) {
        for (int i = 0; i < virtualNodes; i++) {
            long hash = hash(node.toString() + i);
            ring.put(hash, node);
        }
    }
    
    public void removeNode(T node) {
        for (int i = 0; i < virtualNodes; i++) {
            long hash = hash(node.toString() + i);
            ring.remove(hash);
        }
    }
    
    public T getNode(String key) {
        if (ring.isEmpty()) return null;
        long hash = hash(key);
        // Find first node at or after hash
        Map.Entry<Long, T> entry = ring.ceilingEntry(hash);
        if (entry == null) {
            entry = ring.firstEntry();  // Wrap around
        }
        return entry.getValue();
    }
    
    private long hash(String key) {
        return Hashing.murmur3_128().hashString(key, UTF_8).asLong();
    }
}
```

**Benefits:**
- Add server: only K/N keys remap (K=keys, N=servers)
- Remove server: only keys from that server remap
- Virtual nodes ensure even distribution

---

## CAP Theorem & Consistency

### Q7: Explain the CAP theorem.
**Answer:**

CAP theorem states a distributed system can guarantee only **2 of 3**:

```
         Consistency
             ╱╲
            ╱  ╲
           ╱ CP ╲      ← Choose 2
          ╱──────╲
         ╱   CA   ╲    ← Not possible with partitions
        ╱──────────╲
       ╱     AP     ╲
      ╱──────────────╲
Availability ──────── Partition Tolerance
```

**Definitions:**
- **Consistency (C)**: All nodes see the same data at the same time
- **Availability (A)**: Every request gets a response (success/failure)
- **Partition Tolerance (P)**: System works despite network failures

**Real-world choice (since P is required):**

| System Type | Choice | Examples | Use Case |
|-------------|--------|----------|----------|
| CP | Consistency + Partition Tolerance | ZooKeeper, HBase, MongoDB (default) | Banking, inventory |
| AP | Availability + Partition Tolerance | Cassandra, DynamoDB, Couchbase | Social media, product catalogs |

### Q8: What is PACELC theorem?
**Answer:**

PACELC extends CAP to address normal operations:

```
P(artition)  →  A(vailability) vs C(onsistency)
E(lse)       →  L(atency) vs C(onsistency)
```

**During Partition:** Choose Availability or Consistency
**Else (normal):** Choose Latency or Consistency

| System | P: A vs C | E: L vs C |
|--------|-----------|-----------|
| DynamoDB | A | L |
| Cassandra | A | L |
| MongoDB | C | C |
| PNUTS | A | C |
| VoltDB | C | C |

### Q9: Explain different consistency models.
**Answer:**

| Model | Guarantee | Example |
|-------|-----------|---------|
| **Strong/Linearizable** | Reads always return latest write | Single-node DB, ZooKeeper |
| **Sequential** | All operations appear in some total order | Distributed transactions |
| **Causal** | Causally related operations ordered correctly | Social feeds |
| **Read-your-writes** | User always sees their own writes | User sessions |
| **Eventual** | Given no updates, all reads eventually return same value | DNS, Cassandra |

```java
// Strong Consistency
// User A writes, User B immediately sees update
write(x, 1)  // User A
read(x) → 1  // User B (guaranteed)

// Eventual Consistency
// User A writes, User B may see stale data temporarily
write(x, 1)  // User A
read(x) → 0  // User B (old value)
// ... some time passes ...
read(x) → 1  // User B (eventually consistent)
```

### Q10: What is quorum in distributed systems?
**Answer:**

**Quorum** ensures consistency by requiring minimum nodes to agree.

```
N = Total replicas
W = Write quorum (nodes that must acknowledge write)
R = Read quorum (nodes that must respond to read)

Strong consistency when: W + R > N
```

**Examples:**
```
N=3, W=2, R=2: Strong consistency (2+2 > 3)
N=3, W=1, R=1: Eventual consistency, fast (1+1 < 3)
N=3, W=3, R=1: Strong consistency, slow writes
N=3, W=1, R=3: Strong consistency, slow reads
```

**Implementation:**
```java
public class QuorumReplication {
    private List<Node> replicas;
    private int W, R;
    
    public void write(String key, String value) {
        int acks = 0;
        for (Node replica : replicas) {
            try {
                replica.write(key, value);
                acks++;
            } catch (Exception e) {
                // Node failed
            }
        }
        if (acks < W) {
            throw new WriteFailedException("Quorum not reached");
        }
    }
    
    public String read(String key) {
        List<VersionedValue> responses = new ArrayList<>();
        for (Node replica : replicas) {
            try {
                responses.add(replica.read(key));
                if (responses.size() >= R) break;
            } catch (Exception e) {
                // Node failed
            }
        }
        if (responses.size() < R) {
            throw new ReadFailedException("Quorum not reached");
        }
        // Return value with highest version
        return responses.stream()
            .max(Comparator.comparing(VersionedValue::getVersion))
            .map(VersionedValue::getValue)
            .orElse(null);
    }
}
```

---

## Load Balancing

### Q11: What are different load balancing algorithms?
**Answer:**

| Algorithm | Description | Best For |
|-----------|-------------|----------|
| **Round Robin** | Sequential rotation | Homogeneous servers, similar requests |
| **Weighted Round Robin** | Based on server capacity | Heterogeneous servers |
| **Least Connections** | Route to least busy server | Varying request duration |
| **Least Response Time** | Route to fastest server | Performance-critical apps |
| **IP Hash** | Consistent by client IP | Session affinity needed |
| **Random** | Random selection | Simple, even distribution |
| **Consistent Hashing** | Minimal remapping | Caching, distributed systems |

```
Round Robin:
Request 1 → Server A
Request 2 → Server B
Request 3 → Server C
Request 4 → Server A  (cycles back)

Least Connections:
Server A: 5 connections
Server B: 3 connections  ← Next request goes here
Server C: 7 connections
```

### Q12: What is the difference between L4 and L7 load balancing?
**Answer:**

| Aspect | L4 (Transport) | L7 (Application) |
|--------|----------------|------------------|
| **Layer** | TCP/UDP | HTTP/HTTPS |
| **Decision basis** | IP, port | URL, headers, cookies, content |
| **SSL termination** | No (pass-through) | Yes |
| **Content-based routing** | No | Yes |
| **Performance** | Faster | Slower |
| **Features** | Basic load balancing | URL routing, auth, caching |
| **Examples** | HAProxy (L4 mode), AWS NLB | Nginx, HAProxy (L7), AWS ALB |

```
L4 Load Balancing:
┌─────────┐     ┌─────┐     ┌─────────┐
│ Client  │────▶│ LB  │────▶│ Server  │
│ IP:Port │     │     │     │ IP:Port │
└─────────┘     └─────┘     └─────────┘
                  │ TCP connection forwarded

L7 Load Balancing:
┌─────────┐     ┌─────┐     ┌─────────┐
│ Client  │────▶│ LB  │────▶│ Server A│ ← /api/*
│ HTTP Req│     │     │────▶│ Server B│ ← /images/*
└─────────┘     └─────┘     │ Server C│ ← /static/*
                  │ HTTP routing by URL
```

### Q13: How to implement health checks?
**Answer:**

```java
// Simple HTTP health check
@RestController
public class HealthController {
    
    @GetMapping("/health")
    public ResponseEntity<Health> health() {
        return ResponseEntity.ok(
            Health.builder()
                .status(Status.UP)
                .withDetail("database", checkDatabase())
                .withDetail("cache", checkCache())
                .withDetail("memory", getMemoryStatus())
                .build()
        );
    }
}

// Load balancer configuration (Nginx)
upstream backend {
    server backend1.example.com:8080 weight=5;
    server backend2.example.com:8080 weight=3;
    server backend3.example.com:8080 backup;
}

server {
    location / {
        proxy_pass http://backend;
        
        # Health check
        health_check interval=5s fails=3 passes=2 uri=/health;
    }
}
```

**Health Check Types:**
1. **Liveness**: Is the process running?
2. **Readiness**: Is it ready to receive traffic?
3. **Startup**: Has it finished initializing?

---

## Caching Strategies

### Q14: What are different caching patterns?
**Answer:**

**1. Cache-Aside (Lazy Loading)**
```java
public User getUser(String userId) {
    // Check cache first
    User user = cache.get("user:" + userId);
    if (user != null) {
        return user;  // Cache hit
    }
    
    // Cache miss: load from database
    user = database.findUser(userId);
    
    // Populate cache
    cache.set("user:" + userId, user, Duration.ofMinutes(30));
    
    return user;
}

// Write: invalidate cache
public void updateUser(User user) {
    database.updateUser(user);
    cache.delete("user:" + user.getId());  // Invalidate
}
```

**2. Write-Through**
```java
public void updateUser(User user) {
    // Write to cache AND database synchronously
    cache.set("user:" + user.getId(), user);
    database.updateUser(user);
}
```

**3. Write-Behind (Write-Back)**
```java
public void updateUser(User user) {
    // Write to cache immediately
    cache.set("user:" + user.getId(), user);
    
    // Queue async write to database
    writeQueue.add(new WriteOperation(user));
}

// Background worker
@Scheduled(fixedRate = 1000)
public void flushWrites() {
    List<WriteOperation> ops = writeQueue.drain();
    database.batchUpdate(ops);
}
```

**4. Refresh-Ahead**
```java
public User getUser(String userId) {
    CacheEntry entry = cache.getWithMetadata("user:" + userId);
    
    if (entry != null) {
        // If close to expiry, refresh in background
        if (entry.isCloseToExpiry()) {
            executor.submit(() -> refreshCache(userId));
        }
        return entry.getValue();
    }
    
    return loadAndCache(userId);
}
```

### Q15: What is cache stampede and how to prevent it?
**Answer:**

**Problem:** Cache expires → many concurrent requests → all hit database → database overwhelmed

```
Time 0: Cache expires
Time 0: Request 1 → Cache miss → DB query
Time 0: Request 2 → Cache miss → DB query
Time 0: Request 3 → Cache miss → DB query
...100 requests all hit DB simultaneously!
```

**Solutions:**

**1. Locking (Mutex)**
```java
public User getUser(String userId) {
    String key = "user:" + userId;
    User user = cache.get(key);
    
    if (user == null) {
        String lockKey = "lock:" + key;
        
        if (cache.setIfAbsent(lockKey, "1", Duration.ofSeconds(30))) {
            try {
                // Only this thread rebuilds cache
                user = database.findUser(userId);
                cache.set(key, user, Duration.ofMinutes(30));
            } finally {
                cache.delete(lockKey);
            }
        } else {
            // Another thread is rebuilding, wait and retry
            Thread.sleep(50);
            return getUser(userId);  // Retry
        }
    }
    return user;
}
```

**2. Probabilistic Early Expiration**
```java
public User getUser(String userId) {
    CacheEntry entry = cache.getWithExpiry("user:" + userId);
    
    if (entry != null) {
        long ttl = entry.getTimeToLive();
        long delta = entry.getComputeTime();  // Time to rebuild
        double beta = 1.0;  // Tuning parameter
        
        // Random early expiration
        double random = -delta * beta * Math.log(Math.random());
        if (random >= ttl) {
            // Refresh proactively
            return refreshCache(userId);
        }
        return entry.getValue();
    }
    
    return loadAndCache(userId);
}
```

**3. Background Refresh**
```java
@Scheduled(fixedRate = 60000)
public void refreshPopularItems() {
    List<String> hotKeys = analytics.getTopAccessedKeys(100);
    for (String key : hotKeys) {
        refreshIfExpiringSoon(key);
    }
}
```

### Q16: Where should you implement caching?
**Answer:**

```
┌─────────────────────────────────────────────────────────────┐
│                      Caching Layers                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐                                            │
│  │   Browser   │  HTTP Cache (Cache-Control, ETag)          │
│  └──────┬──────┘                                            │
│         │                                                    │
│  ┌──────▼──────┐                                            │
│  │     CDN     │  Static assets, API responses              │
│  └──────┬──────┘                                            │
│         │                                                    │
│  ┌──────▼──────┐                                            │
│  │   Gateway   │  Rate limiting, auth tokens                │
│  └──────┬──────┘                                            │
│         │                                                    │
│  ┌──────▼──────┐                                            │
│  │Application  │  Redis/Memcached (session, objects)        │
│  └──────┬──────┘                                            │
│         │                                                    │
│  ┌──────▼──────┐                                            │
│  │  Database   │  Query cache, buffer pool                  │
│  └─────────────┘                                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Database Design & Sharding

### Q17: When should you shard a database?
**Answer:**

**Shard when:**
- Single database can't handle write load
- Data doesn't fit in one machine's storage
- Query latency degrades with data size
- Read replicas can't solve write bottlenecks

**Sharding Strategies:**

| Strategy | Description | Pros | Cons |
|----------|-------------|------|------|
| **Hash-based** | hash(key) % N | Even distribution | Hard to add shards |
| **Range-based** | By value ranges | Range queries efficient | Hot spots possible |
| **Geographic** | By location | Data locality | Uneven distribution |
| **Directory-based** | Lookup table | Flexible | Lookup is SPOF |

```java
// Hash-based sharding
public int getShardId(String userId) {
    return Math.abs(userId.hashCode() % numShards);
}

// Range-based sharding
public int getShardId(LocalDate date) {
    // Shard by month
    return date.getYear() * 12 + date.getMonthValue();
}

// Consistent hashing (best for scaling)
public String getShardId(String key) {
    return consistentHash.getNode(key).getId();
}
```

### Q18: How to choose a shard key?
**Answer:**

**Good Shard Key Properties:**
1. **High Cardinality**: Many unique values (not boolean!)
2. **Even Distribution**: Avoid hot spots
3. **Query Patterns**: Support common queries without cross-shard joins
4. **Immutable**: Shouldn't change (resharding is expensive)

```
✅ Good shard keys:
- user_id (for user-centric apps)
- order_id (for e-commerce)
- device_id (for IoT)

❌ Bad shard keys:
- country_code (uneven: US >> Liechtenstein)
- created_date (hot spot on current date)
- status (low cardinality)
```

**Example: E-commerce**
```sql
-- Shard by customer_id
-- Good: Customer queries are single-shard
SELECT * FROM orders WHERE customer_id = 123;

-- Bad: Cross-shard scatter-gather query
SELECT * FROM orders WHERE created_at > '2024-01-01';
```

### Q19: How to handle cross-shard queries?
**Answer:**

**Strategies:**

**1. Scatter-Gather**
```java
public List<Order> searchOrders(SearchCriteria criteria) {
    List<Future<List<Order>>> futures = new ArrayList<>();
    
    // Query all shards in parallel
    for (Shard shard : shards) {
        futures.add(executor.submit(() -> 
            shard.search(criteria)
        ));
    }
    
    // Gather and merge results
    List<Order> results = new ArrayList<>();
    for (Future<List<Order>> future : futures) {
        results.addAll(future.get());
    }
    
    // Sort and limit
    return results.stream()
        .sorted(criteria.getSort())
        .limit(criteria.getLimit())
        .collect(toList());
}
```

**2. Denormalization**
```
Store duplicate data to avoid joins
User shard: {user_id, name, email}
Order shard: {order_id, user_id, user_name}  ← Denormalized
```

**3. Global Tables**
```
Small, rarely-changing tables replicated to all shards
- Countries
- Currencies
- Product categories
```

### Q20: What is database replication?
**Answer:**

**Single-Leader (Master-Slave):**
```
        ┌─────────┐
        │ Primary │ ← All writes
        │ (Master)│
        └────┬────┘
     ┌───────┼───────┐
     ▼       ▼       ▼
┌────────┐┌────────┐┌────────┐
│Replica1││Replica2││Replica3│ ← Reads distributed
└────────┘└────────┘└────────┘
```

**Multi-Leader:**
```
┌─────────┐          ┌─────────┐
│Primary 1│◀────────▶│Primary 2│
│(Region A)│ sync    │(Region B)│
└────┬────┘          └────┬────┘
     ▼                    ▼
 Replicas             Replicas
```

**Leaderless (Dynamo-style):**
```
Client writes to multiple nodes
Quorum determines consistency
```

**Replication Lag:**
```java
// Problem: User writes, then reads from replica
user.setName("New Name");
userService.save(user);  // Goes to primary

// Reads from replica that hasn't synced yet
User loaded = userService.findById(id);  // May return old name!

// Solution: Read-your-writes consistency
// Route user's reads to primary after recent writes
if (userSession.hasRecentWrites()) {
    return primary.findById(id);
} else {
    return replica.findById(id);
}
```

---

## Content Delivery Networks

### Q21: How do CDNs work?
**Answer:**

```
Without CDN:
User (Tokyo) ───────────────────────────▶ Origin (New York)
                   ~200ms latency

With CDN:
User (Tokyo) ───▶ Edge (Tokyo) ───cache──▶ Origin (New York)
                   ~20ms latency
```

**CDN Functions:**
1. **Caching**: Store content at edge locations
2. **Load Distribution**: Reduce origin load
3. **DDoS Protection**: Absorb attack traffic
4. **SSL Termination**: Handle TLS at edge
5. **Compression**: gzip/brotli at edge

**Cache Headers:**
```http
Cache-Control: public, max-age=31536000, immutable
ETag: "abc123"
Vary: Accept-Encoding
```

### Q22: What is the difference between push and pull CDN?
**Answer:**

| Aspect | Push CDN | Pull CDN |
|--------|----------|----------|
| **Upload** | You upload to CDN | CDN fetches on demand |
| **Best for** | Static content, large files | Dynamic, frequently updated |
| **Staleness** | You control updates | May serve stale on miss |
| **Cost** | Pay for storage | Pay for bandwidth |
| **Examples** | Video platforms | News websites |

```
Push CDN:
1. You upload file to CDN
2. CDN distributes to all edges
3. Users request from nearest edge

Pull CDN:
1. User requests file
2. Edge checks cache → miss
3. Edge fetches from origin
4. Edge caches and serves
5. Subsequent requests served from cache
```

### Q23: How to invalidate CDN cache?
**Answer:**

**Strategies:**

**1. Cache Busting (versioned URLs)**
```html
<!-- Old -->
<script src="/app.js"></script>

<!-- New (change URL on deployment) -->
<script src="/app.js?v=2"></script>
<!-- Or -->
<script src="/app-abc123.js"></script>
```

**2. API Invalidation**
```java
// CloudFront invalidation
cloudFront.createInvalidation(new CreateInvalidationRequest()
    .withDistributionId(distributionId)
    .withInvalidationBatch(new InvalidationBatch()
        .withPaths(new Paths()
            .withItems("/images/*", "/css/*")
            .withQuantity(2))
        .withCallerReference(UUID.randomUUID().toString())));
```

**3. Short TTL + Revalidation**
```http
Cache-Control: public, max-age=60, stale-while-revalidate=300
```

---

## Message Queues & Async Processing

### Q24: When to use message queues?
**Answer:**

**Use Cases:**
1. **Decoupling**: Services communicate without direct dependency
2. **Load Leveling**: Handle traffic spikes
3. **Reliability**: Guaranteed delivery
4. **Async Processing**: Background jobs

```
Synchronous (without queue):
User ──▶ API ──▶ Email Service ──▶ Response
         │
         └─ If email service down, request fails

Asynchronous (with queue):
User ──▶ API ──▶ Queue ──▶ Email Worker
         │         │
         └─ Response immediately
                   └─ Email sent later (even if worker was down)
```

### Q25: What are different delivery guarantees?
**Answer:**

| Guarantee | Description | How to Achieve |
|-----------|-------------|----------------|
| **At-most-once** | May lose messages | Fire and forget |
| **At-least-once** | May duplicate | Ack after processing |
| **Exactly-once** | No loss, no duplicates | Idempotency + transactions |

```java
// At-least-once with idempotency
public void processMessage(Message message) {
    String messageId = message.getId();
    
    // Check if already processed
    if (processedMessages.contains(messageId)) {
        message.acknowledge();
        return;  // Skip duplicate
    }
    
    try {
        // Process message
        doWork(message.getPayload());
        
        // Mark as processed
        processedMessages.add(messageId);
        
        // Acknowledge
        message.acknowledge();
    } catch (Exception e) {
        // Don't acknowledge - will be redelivered
        message.nack();
    }
}
```

### Q26: How to handle poison messages?
**Answer:**

```java
public void processWithRetry(Message message) {
    int retryCount = message.getRetryCount();
    int maxRetries = 3;
    
    try {
        process(message);
        message.acknowledge();
    } catch (Exception e) {
        if (retryCount >= maxRetries) {
            // Send to Dead Letter Queue
            dlq.send(message);
            message.acknowledge();
            alerting.notify("Message sent to DLQ: " + message.getId());
        } else {
            // Retry with backoff
            long delay = (long) Math.pow(2, retryCount) * 1000;
            message.retry(delay);
        }
    }
}
```

---

## Common System Design Problems

### Q27: Design a URL Shortener (like bit.ly)
**Answer:**

**Requirements:**
- Shorten long URLs
- Redirect to original
- 100M URLs/day, 1000:1 read/write ratio
- Analytics

**Estimation:**
```
Write: 100M/day = 1,160/sec, peak 2,300/sec
Read: 100B/day = 1.16M/sec, peak 2.3M/sec
Storage (5 years): 100M × 365 × 5 × 500 bytes = 91 TB
```

**Design:**
```
┌──────────┐     ┌──────────────┐     ┌─────────────┐
│  Client  │────▶│ Load Balancer │────▶│ App Servers │
└──────────┘     └──────────────┘     └──────┬──────┘
                                             │
                       ┌─────────────────────┼─────────────────────┐
                       ▼                     ▼                     ▼
                 ┌──────────┐          ┌──────────┐          ┌──────────┐
                 │  Redis   │          │ Database │          │ Analytics│
                 │  Cache   │          │ (NoSQL)  │          │ (Kafka)  │
                 └──────────┘          └──────────┘          └──────────┘
```

**URL Generation:**
```java
// Base62 encoding: a-z, A-Z, 0-9 = 62 characters
// 7 characters = 62^7 = 3.5 trillion combinations

public class UrlShortener {
    private static final String ALPHABET = 
        "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    
    // Counter-based (distributed counter with ZooKeeper)
    public String shorten(String longUrl) {
        long id = counter.getNextId();  // Distributed counter
        String shortCode = toBase62(id);
        database.save(new UrlMapping(shortCode, longUrl));
        return "https://short.url/" + shortCode;
    }
    
    private String toBase62(long num) {
        StringBuilder sb = new StringBuilder();
        while (num > 0) {
            sb.append(ALPHABET.charAt((int)(num % 62)));
            num /= 62;
        }
        return sb.reverse().toString();
    }
    
    // Redirect with caching
    public String resolve(String shortCode) {
        String longUrl = cache.get(shortCode);
        if (longUrl == null) {
            longUrl = database.findByShortCode(shortCode);
            cache.set(shortCode, longUrl, Duration.ofHours(24));
        }
        analytics.recordClick(shortCode);  // Async
        return longUrl;
    }
}
```

### Q28: Design a Rate Limiter
**Answer:**

**Algorithms:**

**1. Token Bucket**
```java
public class TokenBucket {
    private final long capacity;
    private final double refillRatePerSecond;
    private double tokens;
    private long lastRefillTime;
    
    public synchronized boolean tryAcquire() {
        refill();
        if (tokens >= 1) {
            tokens--;
            return true;
        }
        return false;
    }
    
    private void refill() {
        long now = System.currentTimeMillis();
        double tokensToAdd = (now - lastRefillTime) / 1000.0 * refillRatePerSecond;
        tokens = Math.min(capacity, tokens + tokensToAdd);
        lastRefillTime = now;
    }
}
```

**2. Sliding Window (Redis)**
```java
public class SlidingWindowRateLimiter {
    private RedisTemplate<String, String> redis;
    
    public boolean isAllowed(String userId, int limit, int windowSeconds) {
        String key = "rate:" + userId;
        long now = System.currentTimeMillis();
        long windowStart = now - windowSeconds * 1000;
        
        // Remove old entries
        redis.opsForZSet().removeRangeByScore(key, 0, windowStart);
        
        // Count current window
        Long count = redis.opsForZSet().count(key, windowStart, now);
        
        if (count < limit) {
            // Add this request
            redis.opsForZSet().add(key, UUID.randomUUID().toString(), now);
            redis.expire(key, windowSeconds, TimeUnit.SECONDS);
            return true;
        }
        return false;
    }
}
```

### Q29: Design Twitter Feed
**Answer:**

**Approach: Fan-out on Write vs Fan-out on Read**

| Approach | Description | Pros | Cons |
|----------|-------------|------|------|
| **Fan-out on Write** | Pre-compute feeds when tweet posted | Fast reads | Slow writes, storage |
| **Fan-out on Read** | Compute feed on request | Fast writes | Slow reads |
| **Hybrid** | Fan-out for normal users, fan-in for celebrities | Balanced | Complex |

```java
// Fan-out on Write
public void postTweet(String userId, String content) {
    Tweet tweet = new Tweet(userId, content);
    tweetStore.save(tweet);
    
    // Get followers (async)
    List<String> followers = userService.getFollowers(userId);
    
    for (String followerId : followers) {
        // Add to each follower's feed cache
        feedCache.prepend("feed:" + followerId, tweet.getId());
    }
}

public List<Tweet> getFeed(String userId) {
    // Simply read from pre-computed cache
    List<String> tweetIds = feedCache.getRange("feed:" + userId, 0, 100);
    return tweetStore.findByIds(tweetIds);
}

// Hybrid for celebrities (>1M followers)
public void postTweet(String userId, String content) {
    Tweet tweet = new Tweet(userId, content);
    tweetStore.save(tweet);
    
    if (userService.getFollowerCount(userId) < 10000) {
        // Fan-out normally
        fanOutToFollowers(userId, tweet);
    } else {
        // Celebrity: followers will pull on read
        celebrityTweets.add("celebrity:" + userId, tweet.getId());
    }
}

public List<Tweet> getFeed(String userId) {
    // Get pre-computed feed
    List<String> tweetIds = feedCache.getRange("feed:" + userId, 0, 100);
    
    // Merge with celebrity tweets (fan-in on read)
    List<String> celebrities = userService.getFollowedCelebrities(userId);
    for (String celebrity : celebrities) {
        tweetIds.addAll(celebrityTweets.getRecent("celebrity:" + celebrity));
    }
    
    // Sort by time and return
    return tweetStore.findByIds(tweetIds)
        .stream()
        .sorted(comparing(Tweet::getCreatedAt).reversed())
        .limit(100)
        .collect(toList());
}
```

### Q30: Design a Chat Application (WhatsApp)
**Answer:**

**Requirements:**
- 1-on-1 and group messaging
- Online presence
- Read receipts
- Media sharing
- End-to-end encryption

**Architecture:**
```
┌────────────┐                                    ┌────────────┐
│  Client A  │◀──WebSocket──▶┌──────────────┐◀───│  Client B  │
└────────────┘               │ Chat Gateway │     └────────────┘
                             │   (Stateful) │
                             └──────┬───────┘
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         ▼                          ▼                          ▼
   ┌───────────┐            ┌──────────────┐           ┌───────────┐
   │  Presence │            │   Message    │           │  Media    │
   │  Service  │            │   Service    │           │  Service  │
   └───────────┘            └──────────────┘           └───────────┘
         │                          │                          │
         ▼                          ▼                          ▼
   ┌───────────┐            ┌──────────────┐           ┌───────────┐
   │   Redis   │            │   Cassandra  │           │    S3     │
   │ (Status)  │            │  (Messages)  │           │  (Files)  │
   └───────────┘            └──────────────┘           └───────────┘
```

**Message Flow:**
```java
// Send message
public void sendMessage(Message message) {
    // 1. Store in database
    messageStore.save(message);
    
    // 2. Check recipient online status
    String recipientId = message.getRecipientId();
    boolean isOnline = presenceService.isOnline(recipientId);
    
    if (isOnline) {
        // 3a. Send via WebSocket
        WebSocketSession session = sessionManager.getSession(recipientId);
        session.sendMessage(message);
    } else {
        // 3b. Send push notification
        pushService.sendNotification(recipientId, message);
    }
    
    // 4. Send delivery receipt to sender
    sendDeliveryReceipt(message.getSenderId(), message.getId());
}

// Online presence with heartbeat
public class PresenceService {
    private RedisTemplate<String, String> redis;
    
    public void heartbeat(String userId) {
        redis.opsForValue().set(
            "presence:" + userId, 
            "online",
            Duration.ofSeconds(30)  // TTL
        );
    }
    
    public boolean isOnline(String userId) {
        return redis.hasKey("presence:" + userId);
    }
}
```

---

## Estimation & Capacity Planning

### Q31: What numbers should you memorize?
**Answer:**

**Latency Numbers:**
```
L1 cache reference:                   0.5 ns
L2 cache reference:                     7 ns
Main memory reference:                100 ns
SSD random read:                   16,000 ns (16 μs)
HDD seek:                       2,000,000 ns (2 ms)
Round trip within datacenter:     500,000 ns (0.5 ms)
Round trip CA → Netherlands:  150,000,000 ns (150 ms)
```

**Throughput:**
```
Read 1 MB sequentially from memory:     250,000 ns (0.25 ms)
Read 1 MB sequentially from SSD:      1,000,000 ns (1 ms)
Read 1 MB sequentially from HDD:     20,000,000 ns (20 ms)
```

**Powers of Two:**
```
2^10 = 1 KB (1,024 bytes)
2^20 = 1 MB (1,048,576 bytes)
2^30 = 1 GB
2^40 = 1 TB
```

**Traffic:**
```
1 million requests/day = 12 requests/second
1 billion requests/day = 12,000 requests/second
```

### Q32: How to estimate storage for Twitter?
**Answer:**

```
Given:
- 500M users
- 200M DAU
- Each user tweets 2 times/day average
- Tweet: 140 chars = 280 bytes + metadata = 500 bytes
- 20% tweets have media (avg 500KB)
- Retention: 10 years

Calculations:

Daily tweets:
= 200M × 2 = 400M tweets/day

Daily storage (text):
= 400M × 500 bytes = 200 GB/day

Daily storage (media):
= 400M × 20% × 500KB = 40 TB/day

Total daily:
≈ 40 TB/day (media dominates)

10-year storage:
= 40 TB × 365 × 10 = 146 PB

With 3x replication:
= 438 PB
```

### Q33: How to estimate bandwidth?
**Answer:**

```
Given:
- 1M users
- Each user loads feed 10 times/day
- Each feed request: 20 tweets × 1KB + 10 images × 100KB = 1 MB

Daily bandwidth:
= 1M × 10 × 1MB = 10 TB/day
= 116 MB/sec average
= 350 MB/sec peak (3x average)

With CDN:
- 80% served from CDN
- Origin bandwidth: 70 MB/sec peak
```

### Q34: How to use Little's Law?
**Answer:**

**Little's Law**: L = λ × W

Where:
- L = Average number of items in system
- λ = Arrival rate (items/time)
- W = Average time in system

```
Example: Database connection pool sizing

Given:
- 1000 requests/second
- Each query takes 50ms

Connections needed:
L = 1000/sec × 0.05 sec = 50 connections

With safety margin (2x):
Pool size = 100 connections
```

---

## Interview Tips

### Q35: How to handle trade-offs?
**Answer:**

**Always acknowledge trade-offs:**

```
"There's a trade-off between consistency and availability here.
For a banking system, I'd choose strong consistency because 
showing incorrect balance is unacceptable. For social media likes,
eventual consistency is fine since users can tolerate briefly 
seeing an old count."
```

**Common trade-offs:**
1. Consistency vs Availability (CAP)
2. Latency vs Throughput
3. Read optimization vs Write optimization
4. Space vs Time (denormalization)
5. Complexity vs Flexibility

### Q36: Common mistakes to avoid?
**Answer:**

1. **Diving into details too fast**: Understand requirements first
2. **Over-engineering**: Start simple, add complexity as needed
3. **Single point of failure**: Always ask "what if X fails?"
4. **Ignoring scale**: Design for expected load, discuss scaling
5. **Not discussing trade-offs**: Every decision has pros/cons
6. **Forgetting monitoring**: Include logging, metrics, alerting
7. **Security as afterthought**: Consider from the start

### Q37: How to structure your communication?
**Answer:**

```
1. REPEAT the problem back
   "So we're designing a URL shortener that handles 100M URLs/day..."

2. ASK clarifying questions
   "Should we support custom URLs? What's the URL expiration policy?"

3. ESTIMATE scale
   "That's about 1,160 writes/sec, 1.16M reads/sec..."

4. DRAW and EXPLAIN high-level design
   "Here's the overall architecture..."

5. DEEP DIVE on key components
   "Let me explain how the ID generation works..."

6. DISCUSS trade-offs and alternatives
   "We could use a counter vs hash approach..."

7. ADDRESS scale and reliability
   "For high availability, we'd add redundancy here..."
```
