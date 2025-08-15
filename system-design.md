# System Design Interview Questions

## Table of Contents
1. [System Design Fundamentals](#system-design-fundamentals)
2. [Scalability Concepts](#scalability-concepts)
3. [CAP Theorem & Distributed Systems](#cap-theorem--distributed-systems)
4. [Load Balancing](#load-balancing)
5. [Caching Strategies](#caching-strategies)
6. [Database Design & Sharding](#database-design--sharding)
7. [Message Queues & Streaming](#message-queues--streaming)
8. [Common System Design Problems](#common-system-design-problems)
9. [Real-World Architecture Examples](#real-world-architecture-examples)

## System Design Fundamentals

### Q1: What are the key principles of system design?
**Answer:**
1. **Reliability**: System continues to work correctly even when things fail
2. **Scalability**: Ability to handle increased load gracefully
3. **Availability**: Operational time, usually measured in 9s (99.9% = three nines)
4. **Maintainability**: Easy to operate, understand, and modify
5. **Performance**: Response time and throughput
6. **Security**: Protecting data and system from unauthorized access

### Q2: How to approach a system design interview?
**Answer:**
**Step-by-step approach:**
1. **Requirements Gathering** (5-10 mins)
   - Functional requirements
   - Non-functional requirements
   - Scale and constraints
   
2. **Capacity Estimation** (5 mins)
   - Traffic estimates
   - Storage requirements
   - Bandwidth requirements
   
3. **High-Level Design** (10-15 mins)
   - Draw main components
   - Show data flow
   
4. **Detailed Design** (15-20 mins)
   - Deep dive into components
   - Data models
   - Algorithms
   
5. **Scale & Optimize** (10 mins)
   - Bottlenecks
   - Trade-offs
   - Monitoring

### Q3: How to estimate system capacity?
**Answer:**
**Example: Twitter-like system**
```
Users: 500M total, 200M DAU
Tweets: 500M tweets/day
Read:Write ratio: 100:1

Calculations:
- Tweets/second: 500M / (24*3600) ≈ 6000 tweets/sec
- Peak (2x average): 12000 tweets/sec
- Reads/second: 600K reads/sec

Storage (5 years):
- Tweet size: 280 chars = 560 bytes + metadata = 1KB
- Media: 20% tweets have media, avg 200KB
- Daily: 500M * 1KB + 100M * 200KB = 500GB + 20TB ≈ 20TB/day
- 5 years: 20TB * 365 * 5 = 36.5PB

Bandwidth:
- Write: 12000 * 1KB = 12MB/s
- Read: 600K * 1KB = 600MB/s
```

## Scalability Concepts

### Q4: What is horizontal vs vertical scaling?
**Answer:**
**Vertical Scaling (Scale-up):**
- Add more power (CPU, RAM) to existing machine
- Pros: Simple, no code changes, good for ACID transactions
- Cons: Hardware limits, single point of failure, expensive

**Horizontal Scaling (Scale-out):**
- Add more machines to the pool
- Pros: No hardware limit, fault tolerance, cost-effective
- Cons: Complex architecture, data consistency challenges

### Q5: Explain different types of system architectures.
**Answer:**
1. **Monolithic**: Single deployable unit
2. **Service-Oriented (SOA)**: Services communicate through ESB
3. **Microservices**: Small, independent services
4. **Serverless**: Functions as a Service (FaaS)
5. **Event-Driven**: Components communicate through events
6. **Layered**: Organized in hierarchical layers

### Q6: What are the key metrics for system performance?
**Answer:**
- **Response Time**: Time to process a request
- **Throughput**: Number of requests processed per unit time
- **Availability**: Percentage of time system is operational
- **Latency**: Delay before transfer begins
- **Bandwidth**: Maximum data transfer rate
- **QPS (Queries Per Second)**: Request rate

**SLA Availability Levels:**
- 99% = 3.65 days/year downtime
- 99.9% = 8.76 hours/year
- 99.99% = 52.56 minutes/year
- 99.999% = 5.26 minutes/year

## CAP Theorem & Distributed Systems

### Q7: Explain the CAP theorem.
**Answer:**
CAP theorem states a distributed system can guarantee only 2 of 3:
- **Consistency**: All nodes see the same data simultaneously
- **Availability**: System remains operational
- **Partition Tolerance**: System continues despite network failures

**Trade-offs:**
- **CP Systems**: Consistency over Availability (HBase, MongoDB)
- **AP Systems**: Availability over Consistency (Cassandra, DynamoDB)
- **CA Systems**: Not possible in distributed systems with network partitions

### Q8: What is ACID vs BASE?
**Answer:**
**ACID (Traditional RDBMS):**
- **Atomicity**: All or nothing
- **Consistency**: Valid state transitions
- **Isolation**: Concurrent operations don't interfere
- **Durability**: Committed data persists

**BASE (NoSQL):**
- **Basically Available**: System is available
- **Soft State**: State may change over time
- **Eventual Consistency**: Will become consistent eventually

### Q9: Explain different consistency models.
**Answer:**
1. **Strong Consistency**: All reads return most recent write
2. **Eventual Consistency**: System will become consistent over time
3. **Weak Consistency**: No guarantees when all nodes will be consistent
4. **Read-after-Write**: User sees their own writes immediately
5. **Monotonic Read**: Once seen, won't see older version
6. **Causal Consistency**: Causally related operations seen in order

### Q10: What is consensus in distributed systems?
**Answer:**
Consensus algorithms ensure nodes agree on values:

**Paxos**: Complex but proven consensus algorithm
**Raft**: Simpler alternative to Paxos
**Zab**: Used in Zookeeper

**Key concepts:**
- Leader election
- Log replication
- Majority agreement (quorum)

## Load Balancing

### Q11: What are different load balancing algorithms?
**Answer:**
1. **Round Robin**: Requests distributed sequentially
2. **Weighted Round Robin**: Based on server capacity
3. **Least Connections**: Route to server with fewest connections
4. **Least Response Time**: Route to fastest server
5. **IP Hash**: Route based on client IP
6. **Random**: Random server selection
7. **Consistent Hashing**: Minimizes remapping when servers change

### Q12: Explain L4 vs L7 load balancing.
**Answer:**
**L4 (Transport Layer):**
- Works with IP and port
- Faster, less CPU intensive
- Can't inspect application data
- TCP/UDP level

**L7 (Application Layer):**
- Can inspect HTTP headers, URLs
- Content-based routing
- SSL termination
- More CPU intensive

### Q13: How to implement sticky sessions?
**Answer:**
Methods to maintain session affinity:
1. **Cookie-based**: LB sets cookie with server ID
2. **IP-based**: Route based on source IP
3. **Session ID**: Extract from application layer

**Drawbacks:**
- Uneven load distribution
- Difficult to scale
- Session loss on server failure

**Better approach**: Store sessions in distributed cache (Redis)

## Caching Strategies

### Q14: What are different caching strategies?
**Answer:**
1. **Cache-Aside (Lazy Loading)**:
   - Read: Check cache → miss → load from DB → update cache
   - Write: Write to DB → invalidate cache
   
2. **Write-Through**:
   - Write to cache and DB simultaneously
   - Ensures consistency
   - Higher latency
   
3. **Write-Behind (Write-Back)**:
   - Write to cache immediately
   - Async write to DB
   - Risk of data loss
   
4. **Refresh-Ahead**:
   - Proactively refresh before expiration
   - Good for predictable access patterns

### Q15: Where to implement caching?
**Answer:**
1. **Browser Cache**: HTTP headers (Cache-Control, ETag)
2. **CDN**: Static content, geographically distributed
3. **Reverse Proxy**: Nginx, Varnish
4. **Application Cache**: In-memory, Redis, Memcached
5. **Database Cache**: Query result cache

### Q16: How to handle cache invalidation?
**Answer:**
**Strategies:**
1. **TTL (Time To Live)**: Automatic expiration
2. **Event-based**: Invalidate on data change
3. **Manual**: Explicit invalidation
4. **Versioning**: New key for new version

**Patterns:**
```java
// Cache-aside with TTL
public User getUser(String userId) {
    String key = "user:" + userId;
    User user = cache.get(key);
    
    if (user == null) {
        user = database.getUser(userId);
        cache.set(key, user, TTL_SECONDS);
    }
    return user;
}

// Write-through
public void updateUser(User user) {
    database.updateUser(user);
    cache.set("user:" + user.getId(), user);
}
```

## Database Design & Sharding

### Q17: When to use SQL vs NoSQL?
**Answer:**
**SQL (RDBMS):**
- ACID compliance required
- Complex queries and joins
- Structured data with relationships
- Examples: PostgreSQL, MySQL

**NoSQL:**
- **Document**: Semi-structured data (MongoDB)
- **Key-Value**: Simple lookups (Redis, DynamoDB)
- **Column-Family**: Wide columns (Cassandra, HBase)
- **Graph**: Connected data (Neo4j)

### Q18: What is database sharding?
**Answer:**
Sharding splits data across multiple databases.

**Strategies:**
1. **Range-based**: By value range (A-M, N-Z)
2. **Hash-based**: Hash function determines shard
3. **Geographic**: By location
4. **Directory-based**: Lookup service maintains mapping

**Challenges:**
- Joins across shards
- Rebalancing data
- Hot spots
- Maintaining consistency

### Q19: How to handle database replication?
**Answer:**
**Master-Slave Replication:**
- Write to master, read from slaves
- Eventual consistency
- Read scaling

**Master-Master Replication:**
- Write to any master
- Conflict resolution needed
- Better availability

**Implementation considerations:**
- Replication lag
- Failover mechanism
- Data consistency
- Split-brain problem

### Q20: Design a URL shortener (like bit.ly).
**Answer:**
**Requirements:**
- Shorten long URLs
- Redirect to original URL
- 100M URLs/day
- Analytics

**Design:**
```
Components:
1. API Gateway
2. Application Servers
3. Cache Layer (Redis)
4. Database (NoSQL for scale)
5. Analytics Service

Data Model:
ShortURL {
  short_url: string (primary key)
  long_url: string
  created_at: timestamp
  expires_at: timestamp
  click_count: integer
}

Algorithm:
- Base62 encoding (a-z, A-Z, 0-9)
- 7 characters = 62^7 = 3.5 trillion URLs
- Counter-based or hash-based generation

API:
POST /shorten
  Body: { "url": "https://example.com/very/long/url" }
  Response: { "short_url": "https://bit.ly/abc123" }

GET /{short_code}
  Response: 301 Redirect to long URL

Optimizations:
- Cache popular URLs
- Geographic distribution (CDN)
- Rate limiting
- Custom URLs for premium users
```

## Message Queues & Streaming

### Q21: When to use message queues vs event streaming?
**Answer:**
**Message Queues (RabbitMQ, SQS):**
- Point-to-point communication
- Message deleted after consumption
- Task distribution
- Order processing, email sending

**Event Streaming (Kafka, Kinesis):**
- Publish-subscribe model
- Events retained for period
- Multiple consumers
- Real-time analytics, event sourcing

### Q22: Explain different messaging patterns.
**Answer:**
1. **Point-to-Point**: One producer, one consumer
2. **Publish-Subscribe**: One producer, multiple consumers
3. **Request-Reply**: Synchronous communication
4. **Message Router**: Route based on content
5. **Message Translator**: Transform message format

### Q23: How to ensure message delivery?
**Answer:**
**Delivery Guarantees:**
1. **At-most-once**: May lose messages
2. **At-least-once**: May duplicate (idempotency required)
3. **Exactly-once**: No loss or duplication (complex)

**Implementation:**
```java
// Idempotent consumer
public void processMessage(Message message) {
    String idempotencyKey = message.getId();
    
    if (processedMessages.contains(idempotencyKey)) {
        return; // Already processed
    }
    
    try {
        // Process message
        processedMessages.add(idempotencyKey);
        message.acknowledge();
    } catch (Exception e) {
        message.retry();
    }
}
```

## Common System Design Problems

### Q24: Design a distributed cache.
**Answer:**
**Requirements:**
- GET/SET operations
- TTL support
- Eviction policies
- High availability

**Design:**
```
Architecture:
- Consistent hashing for distribution
- Replication for availability
- LRU eviction policy

Components:
1. Cache Servers (nodes)
2. Consistent Hash Ring
3. Replication Manager
4. Client Library

Data Structure:
class CacheNode {
    HashMap<String, CacheEntry> data;
    LinkedList<String> lruList;
    ReplicationManager replicator;
}

class CacheEntry {
    String value;
    long timestamp;
    long ttl;
}

Operations:
- GET: Hash key → Find node → Check TTL → Return value
- SET: Hash key → Find node → Store → Replicate
- DELETE: Hash key → Find node → Delete → Replicate

Consistency:
- Read repair for eventual consistency
- Quorum reads/writes for strong consistency
```

### Q25: Design a rate limiter.
**Answer:**
**Algorithms:**
1. **Token Bucket**: Tokens refill at fixed rate
2. **Leaky Bucket**: Fixed output rate
3. **Fixed Window**: Count in fixed time windows
4. **Sliding Window Log**: Track timestamp of each request
5. **Sliding Window Counter**: Hybrid approach

**Implementation:**
```java
// Token Bucket
public class TokenBucketRateLimiter {
    private final long capacity;
    private final long refillRate;
    private long tokens;
    private long lastRefillTime;
    
    public synchronized boolean allowRequest() {
        refill();
        if (tokens > 0) {
            tokens--;
            return true;
        }
        return false;
    }
    
    private void refill() {
        long now = System.currentTimeMillis();
        long tokensToAdd = (now - lastRefillTime) * refillRate / 1000;
        tokens = Math.min(capacity, tokens + tokensToAdd);
        lastRefillTime = now;
    }
}

// Distributed Rate Limiter with Redis
public class DistributedRateLimiter {
    private RedisTemplate redis;
    
    public boolean allowRequest(String userId, int limit, int window) {
        String key = "rate_limit:" + userId;
        Long count = redis.increment(key);
        
        if (count == 1) {
            redis.expire(key, window, TimeUnit.SECONDS);
        }
        
        return count <= limit;
    }
}
```

### Q26: Design a notification system.
**Answer:**
**Requirements:**
- Multiple channels (push, email, SMS)
- User preferences
- Priority levels
- Delivery guarantees

**Architecture:**
```
Components:
1. API Gateway
2. Notification Service
3. Message Queue (Kafka)
4. Channel Handlers
5. Preference Service
6. Template Service
7. Analytics Service

Flow:
1. Client sends notification request
2. Validate and enrich with user preferences
3. Publish to message queue
4. Channel handlers consume and deliver
5. Track delivery status

Data Model:
Notification {
    id: UUID
    user_id: string
    type: enum (push, email, sms)
    template_id: string
    parameters: map
    priority: enum (high, medium, low)
    status: enum (pending, sent, failed)
    retry_count: integer
}

Reliability:
- Retry with exponential backoff
- Dead letter queue for failed messages
- Idempotency for duplicate prevention
```

### Q27: Design a distributed task scheduler.
**Answer:**
**Requirements:**
- Schedule one-time and recurring tasks
- Distributed execution
- Fault tolerance
- Exactly-once execution

**Design:**
```
Components:
1. Scheduler Service
2. Task Queue (Priority Queue)
3. Worker Pool
4. Lock Service (Zookeeper/Redis)
5. Task Store (Database)

Task Model:
Task {
    id: UUID
    name: string
    payload: json
    schedule: cron_expression
    next_run: timestamp
    status: enum
    retry_policy: object
}

Execution Flow:
1. Scheduler polls due tasks
2. Acquire distributed lock
3. Push to task queue
4. Worker picks up task
5. Execute with timeout
6. Update task status
7. Calculate next run (if recurring)

Fault Tolerance:
- Leader election for scheduler
- Task ownership with locks
- Heartbeat for worker health
- Automatic retry on failure
```

## Real-World Architecture Examples

### Q28: Design WhatsApp/Messenger.
**Answer:**
**Features:**
- 1-to-1 messaging
- Group chat
- Online status
- Read receipts
- Media sharing

**Architecture:**
```
Components:
1. Chat Servers (WebSocket)
2. Media Servers
3. Notification Service
4. Presence Service
5. Database (Messages)
6. Cache (Recent messages)

Message Flow:
1. Client → WebSocket → Chat Server
2. Store in database
3. Check recipient online status
4. If online: Push via WebSocket
5. If offline: Send push notification

Data Model:
Message {
    id: UUID
    sender_id: string
    recipient_id: string
    conversation_id: string
    content: string
    media_url: string
    timestamp: datetime
    status: enum (sent, delivered, read)
}

Optimizations:
- End-to-end encryption
- Message queue for reliability
- CDN for media
- Pagination for message history
```

### Q29: Design YouTube/Netflix.
**Answer:**
**Requirements:**
- Video upload and processing
- Streaming to millions
- Recommendations
- Search

**Architecture:**
```
Upload Pipeline:
1. Upload to storage (S3)
2. Queue processing job
3. Transcode to multiple qualities
4. Generate thumbnails
5. Update metadata
6. Distribute to CDN

Streaming:
- Adaptive bitrate streaming
- CDN for global distribution
- Pre-fetch popular content

Components:
1. Upload Service
2. Video Processing (FFmpeg)
3. Metadata Service
4. CDN (CloudFront)
5. Recommendation Service
6. Search Service (Elasticsearch)

Data Storage:
- Videos: Object storage (S3)
- Metadata: NoSQL (DynamoDB)
- User data: RDBMS
- Search index: Elasticsearch
```

### Q30: Design Uber/Lyft.
**Answer:**
**Core Features:**
- Rider-driver matching
- Real-time location tracking
- Dynamic pricing
- ETA calculation

**Architecture:**
```
Services:
1. User Service
2. Driver Service
3. Location Service
4. Matching Service
5. Pricing Service
6. Trip Service
7. Payment Service
8. Notification Service

Matching Algorithm:
1. Rider requests ride
2. Find nearby drivers (GeoHash/QuadTree)
3. Send request to closest drivers
4. First to accept gets the ride
5. Notify rider

Location Tracking:
- Drivers send location every 4 seconds
- Store in cache (Redis) with TTL
- Use GeoHash for spatial indexing

Data Model:
Trip {
    id: UUID
    rider_id: string
    driver_id: string
    start_location: GeoPoint
    end_location: GeoPoint
    route: LineString
    fare: decimal
    status: enum
    created_at: timestamp
}

Scalability:
- City-based sharding
- WebSocket for real-time updates
- Caching for hot data
- Event-driven architecture
```

## Performance Optimization

### Q31: How to optimize database queries?
**Answer:**
1. **Indexing**: Create appropriate indexes
2. **Query Optimization**: Use EXPLAIN, avoid N+1
3. **Denormalization**: Trade space for speed
4. **Partitioning**: Split large tables
5. **Connection Pooling**: Reuse connections
6. **Caching**: Cache frequent queries
7. **Read Replicas**: Distribute read load

### Q32: How to handle hot partitions?
**Answer:**
**Problem**: Uneven load distribution

**Solutions:**
1. **Add salt to key**: Distribute hot keys
2. **Split hot partitions**: Manual intervention
3. **Caching**: Cache hot data
4. **Request coalescing**: Batch similar requests
5. **Backpressure**: Rate limit hot keys

## Monitoring & Debugging

### Q33: What metrics should be monitored?
**Answer:**
**Golden Signals:**
1. **Latency**: Response time distribution
2. **Traffic**: Requests per second
3. **Errors**: Error rate and types
4. **Saturation**: Resource utilization

**Additional Metrics:**
- Business metrics (conversion, revenue)
- Application metrics (cache hit rate)
- Infrastructure metrics (CPU, memory, disk)

### Q34: How to implement distributed tracing?
**Answer:**
**Components:**
- Trace ID: Unique identifier for request
- Span ID: Individual operation
- Parent Span: Hierarchy of operations

**Tools:**
- Jaeger, Zipkin, AWS X-Ray

**Implementation:**
```java
public class TracingInterceptor {
    public Response intercept(Request request) {
        String traceId = request.getHeader("X-Trace-ID");
        if (traceId == null) {
            traceId = UUID.randomUUID().toString();
        }
        
        Span span = tracer.buildSpan("operation")
            .withTag("trace.id", traceId)
            .start();
            
        try {
            // Forward trace ID to downstream services
            request.addHeader("X-Trace-ID", traceId);
            return next.handle(request);
        } finally {
            span.finish();
        }
    }
}
```

## Interview Tips

### Q35: How to handle trade-offs in system design?
**Answer:**
Always discuss trade-offs:
1. **Consistency vs Availability**
2. **Latency vs Throughput**
3. **Space vs Time**
4. **Simplicity vs Flexibility**
5. **Cost vs Performance**

Example: "For this social media feed, I'd choose eventual consistency over strong consistency because users can tolerate seeing slightly stale data, but the system must remain available."

### Q36: Common mistakes to avoid?
**Answer:**
1. **Over-engineering**: Don't add unnecessary complexity
2. **Ignoring requirements**: Address all functional/non-functional requirements
3. **No capacity planning**: Always estimate scale
4. **Single point of failure**: Ensure redundancy
5. **Ignoring data consistency**: Address CAP trade-offs
6. **No monitoring plan**: Include observability
7. **Security afterthought**: Consider security from start