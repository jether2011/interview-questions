# Database & Caching Interview Questions

## Table of Contents
1. [Database Fundamentals](#database-fundamentals)
2. [SQL & Query Optimization](#sql--query-optimization)
3. [NoSQL Databases](#nosql-databases)
4. [Database Design & Normalization](#database-design--normalization)
5. [Transactions & ACID](#transactions--acid)
6. [Indexing & Performance](#indexing--performance)
7. [Caching Fundamentals](#caching-fundamentals)
8. [Caching Strategies](#caching-strategies)
9. [Distributed Caching](#distributed-caching)
10. [Cache Implementation](#cache-implementation)

## Database Fundamentals

### Q1: Compare SQL vs NoSQL databases.
**Answer:**

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| Data Model | Tables with rows and columns | Document, Key-Value, Graph, Column-Family |
| Schema | Fixed schema | Dynamic/Flexible schema |
| Scaling | Vertical (Scale-up) | Horizontal (Scale-out) |
| ACID | Full ACID compliance | Eventually consistent (usually) |
| Query Language | SQL | Database-specific |
| Relationships | Foreign keys, JOINs | Embedded documents or references |
| Use Cases | Complex queries, transactions | Big data, real-time, flexible data |

### Q2: When to use which type of database?
**Answer:**

**RDBMS (PostgreSQL, MySQL):**
- Complex relationships between data
- ACID transactions required
- Structured data with fixed schema
- Complex queries with JOINs
- Financial applications

**Document Store (MongoDB, CouchDB):**
- Semi-structured data
- Rapid development
- Content management
- Catalogs

**Key-Value (Redis, DynamoDB):**
- Session storage
- Caching
- Real-time recommendations
- Gaming leaderboards

**Column-Family (Cassandra, HBase):**
- Time-series data
- Write-heavy workloads
- Distributed data
- IoT applications

**Graph (Neo4j, Amazon Neptune):**
- Social networks
- Recommendation engines
- Fraud detection
- Network topology

### Q3: Explain database isolation levels.
**Answer:**

**1. Read Uncommitted:**
- Lowest isolation
- Dirty reads possible
- No locks when reading

**2. Read Committed:**
- No dirty reads
- Non-repeatable reads possible
- Default in PostgreSQL, Oracle

**3. Repeatable Read:**
- No dirty or non-repeatable reads
- Phantom reads possible
- Default in MySQL InnoDB

**4. Serializable:**
- Highest isolation
- No anomalies
- Performance impact

```sql
-- Setting isolation level
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Example of isolation issues
-- Transaction 1
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Not committed yet

-- Transaction 2 (READ UNCOMMITTED)
SELECT balance FROM accounts WHERE id = 1; -- Sees uncommitted change (dirty read)

-- Transaction 1
ROLLBACK; -- Change is rolled back

-- Transaction 2's read was incorrect
```

## SQL & Query Optimization

### Q4: How to optimize SQL queries?
**Answer:**

**1. Use appropriate indexes:**
```sql
-- Create index on frequently queried columns
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_order_date ON orders(created_at);

-- Composite index for multiple columns
CREATE INDEX idx_user_name ON users(last_name, first_name);
```

**2. Avoid SELECT *:**
```sql
-- Bad
SELECT * FROM users WHERE active = true;

-- Good
SELECT id, name, email FROM users WHERE active = true;
```

**3. Use EXPLAIN to analyze queries:**
```sql
EXPLAIN ANALYZE SELECT u.name, COUNT(o.id)
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;
```

**4. Optimize JOINs:**
```sql
-- Ensure join columns are indexed
-- Use INNER JOIN when possible (faster than OUTER JOIN)
-- Join smaller tables first
```

**5. Use appropriate data types:**
```sql
-- Use INT for IDs instead of VARCHAR
-- Use appropriate string lengths
-- Use DATE/TIMESTAMP for dates, not strings
```

### Q5: Explain different types of JOINs.
**Answer:**

```sql
-- Sample tables
-- users: id, name
-- orders: id, user_id, amount

-- INNER JOIN: Returns matching records from both tables
SELECT u.name, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: All from left table, matching from right
SELECT u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
-- Users without orders will have NULL for amount

-- RIGHT JOIN: All from right table, matching from left
SELECT u.name, o.amount
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN: All records from both tables
SELECT u.name, o.amount
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- CROSS JOIN: Cartesian product
SELECT u.name, p.product_name
FROM users u
CROSS JOIN products p;

-- SELF JOIN: Table joined with itself
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id;
```

### Q6: What are window functions?
**Answer:**
Window functions perform calculations across a set of rows related to the current row.

```sql
-- ROW_NUMBER(): Assigns sequential number
SELECT 
    name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rank
FROM employees;

-- RANK() and DENSE_RANK()
SELECT 
    name,
    score,
    RANK() OVER (ORDER BY score DESC) as rank,
    DENSE_RANK() OVER (ORDER BY score DESC) as dense_rank
FROM students;
-- RANK: 1,2,2,4 (skips 3)
-- DENSE_RANK: 1,2,2,3 (no skip)

-- LAG() and LEAD()
SELECT 
    date,
    sales,
    LAG(sales, 1) OVER (ORDER BY date) as previous_day_sales,
    LEAD(sales, 1) OVER (ORDER BY date) as next_day_sales
FROM daily_sales;

-- Running total
SELECT 
    date,
    amount,
    SUM(amount) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM transactions;

-- Moving average
SELECT 
    date,
    price,
    AVG(price) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg_3
FROM stock_prices;
```

## NoSQL Databases

### Q7: Explain MongoDB data modeling.
**Answer:**

**Embedding vs Referencing:**

```javascript
// Embedding (Denormalized)
{
  _id: ObjectId("..."),
  name: "John Doe",
  addresses: [
    {
      type: "home",
      street: "123 Main St",
      city: "Boston"
    },
    {
      type: "work",
      street: "456 Office Blvd",
      city: "Cambridge"
    }
  ]
}

// Referencing (Normalized)
// User document
{
  _id: ObjectId("user123"),
  name: "John Doe",
  address_ids: [ObjectId("addr1"), ObjectId("addr2")]
}

// Address documents
{
  _id: ObjectId("addr1"),
  user_id: ObjectId("user123"),
  type: "home",
  street: "123 Main St",
  city: "Boston"
}
```

**When to embed:**
- 1-to-1 relationships
- 1-to-few relationships
- Data always accessed together
- Data rarely changes

**When to reference:**
- 1-to-many with many
- Many-to-many relationships
- Large documents
- Frequently changing data

### Q8: Explain Cassandra's data model.
**Answer:**

Cassandra uses a wide-column store model:

```sql
-- Create keyspace (database)
CREATE KEYSPACE my_app
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 3
};

-- Create table with partition key and clustering key
CREATE TABLE user_activities (
    user_id UUID,
    activity_time timestamp,
    activity_type text,
    details text,
    PRIMARY KEY (user_id, activity_time)
) WITH CLUSTERING ORDER BY (activity_time DESC);

-- Partition key: user_id (determines node)
-- Clustering key: activity_time (sorts within partition)

-- Efficient queries
SELECT * FROM user_activities WHERE user_id = ?;
SELECT * FROM user_activities WHERE user_id = ? AND activity_time > ?;

-- Inefficient (requires ALLOW FILTERING)
SELECT * FROM user_activities WHERE activity_type = ?;
```

**Key Concepts:**
- **Partition Key**: Determines data distribution
- **Clustering Key**: Sorts data within partition
- **Denormalization**: Duplicate data for query patterns
- **Write-optimized**: Designed for high write throughput

## Database Design & Normalization

### Q9: Explain database normalization forms.
**Answer:**

**1NF (First Normal Form):**
- Atomic values (no repeating groups)
- Each column contains single value

```sql
-- Violates 1NF
CREATE TABLE orders (
    order_id INT,
    products VARCHAR(255) -- "Product1,Product2,Product3"
);

-- Satisfies 1NF
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT
);
```

**2NF (Second Normal Form):**
- Must be in 1NF
- No partial dependencies on composite key

```sql
-- Violates 2NF (category depends only on product_id)
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_category VARCHAR(50),
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- Satisfies 2NF
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_category VARCHAR(50)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

**3NF (Third Normal Form):**
- Must be in 2NF
- No transitive dependencies

```sql
-- Violates 3NF (city depends on zip_code, not on customer_id)
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    zip_code VARCHAR(10),
    city VARCHAR(50)
);

-- Satisfies 3NF
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    zip_code VARCHAR(10)
);

CREATE TABLE zip_codes (
    zip_code VARCHAR(10) PRIMARY KEY,
    city VARCHAR(50)
);
```

### Q10: When to denormalize?
**Answer:**

**Reasons to denormalize:**
1. **Performance**: Reduce JOINs
2. **Read-heavy workloads**: Optimize for queries
3. **Reporting**: Pre-computed aggregates
4. **Caching**: Materialized views

**Examples:**
```sql
-- Normalized
SELECT o.order_id, o.order_date, c.name, c.email
FROM orders o
JOIN customers c ON o.customer_id = c.id;

-- Denormalized (customer info in orders)
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE,
    customer_id INT,
    customer_name VARCHAR(100),  -- Denormalized
    customer_email VARCHAR(100)  -- Denormalized
);

-- Materialized view for reporting
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT 
    DATE_TRUNC('month', order_date) as month,
    SUM(total_amount) as total_sales,
    COUNT(*) as order_count
FROM orders
GROUP BY DATE_TRUNC('month', order_date);
```

## Transactions & ACID

### Q11: Explain ACID properties with examples.
**Answer:**

**Atomicity:** All or nothing
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- If any fails, both are rolled back
COMMIT;
```

**Consistency:** Valid state transitions
```sql
-- Constraint ensures consistency
ALTER TABLE accounts ADD CONSTRAINT positive_balance 
CHECK (balance >= 0);

-- This will fail if it violates constraint
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
```

**Isolation:** Concurrent transactions don't interfere
```sql
-- Transaction 1
BEGIN;
SELECT balance FROM accounts WHERE id = 1; -- 1000

-- Transaction 2
BEGIN;
UPDATE accounts SET balance = 900 WHERE id = 1;
COMMIT;

-- Transaction 1 (REPEATABLE READ)
SELECT balance FROM accounts WHERE id = 1; -- Still 1000
COMMIT;
```

**Durability:** Committed data persists
```sql
-- After COMMIT, data survives system crash
BEGIN;
INSERT INTO important_logs (message) VALUES ('Critical event');
COMMIT; -- Data is persisted to disk
```

### Q12: How to handle distributed transactions?
**Answer:**

**Two-Phase Commit (2PC):**
```java
// Coordinator
public class TransactionCoordinator {
    public boolean executeDistributedTransaction() {
        // Phase 1: Prepare
        boolean allPrepared = participants.stream()
            .allMatch(p -> p.prepare());
        
        if (!allPrepared) {
            // Phase 2: Abort
            participants.forEach(p -> p.abort());
            return false;
        }
        
        // Phase 2: Commit
        participants.forEach(p -> p.commit());
        return true;
    }
}
```

**Saga Pattern (Compensation-based):**
```java
public class OrderSaga {
    public void processOrder(Order order) {
        try {
            // Step 1
            Payment payment = paymentService.charge(order);
            
            // Step 2
            Inventory inventory = inventoryService.reserve(order);
            
            // Step 3
            Shipment shipment = shippingService.schedule(order);
            
        } catch (Exception e) {
            // Compensate in reverse order
            shippingService.cancel(shipment);
            inventoryService.release(inventory);
            paymentService.refund(payment);
        }
    }
}
```

## Indexing & Performance

### Q13: Explain different types of database indexes.
**Answer:**

**1. B-Tree Index:**
- Default in most databases
- Good for equality and range queries
- Ordered data structure

```sql
CREATE INDEX idx_users_age ON users(age);
-- Efficient for: WHERE age = 25, WHERE age > 25
```

**2. Hash Index:**
- Only equality comparisons
- O(1) lookup
- No range queries

```sql
CREATE INDEX idx_users_email ON users USING HASH (email);
-- Efficient for: WHERE email = 'user@example.com'
-- Not for: WHERE email LIKE 'user%'
```

**3. Bitmap Index:**
- Low cardinality columns
- Efficient for AND/OR operations
- Common in data warehouses

```sql
CREATE BITMAP INDEX idx_users_status ON users(status);
-- Good for: status IN ('active', 'inactive', 'pending')
```

**4. Full-Text Index:**
- Text search
- Language-aware

```sql
CREATE FULLTEXT INDEX idx_articles_content ON articles(title, content);
SELECT * FROM articles WHERE MATCH(title, content) AGAINST('database');
```

**5. Composite Index:**
```sql
CREATE INDEX idx_users_name ON users(last_name, first_name);
-- Efficient for:
-- WHERE last_name = 'Smith'
-- WHERE last_name = 'Smith' AND first_name = 'John'
-- Not efficient for: WHERE first_name = 'John'
```

### Q14: How to identify and fix slow queries?
**Answer:**

**1. Enable slow query log:**
```sql
-- MySQL
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;

-- PostgreSQL
ALTER SYSTEM SET log_min_duration_statement = 1000; -- ms
```

**2. Use EXPLAIN:**
```sql
EXPLAIN (ANALYZE, BUFFERS) 
SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;
```

**3. Common optimizations:**
```sql
-- Add missing index
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Rewrite subquery as JOIN
-- Bad
SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders WHERE total > 100);

-- Good
SELECT DISTINCT u.* FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.total > 100;

-- Use covering index
CREATE INDEX idx_orders_covering ON orders(user_id, total, status);

-- Partition large tables
CREATE TABLE orders_2024 PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

## Caching Fundamentals

### Q15: What are different caching levels?
**Answer:**

**1. Browser Cache:**
- HTTP headers (Cache-Control, ETag)
- Local storage
- Service workers

**2. CDN (Content Delivery Network):**
- Geographic distribution
- Static content
- Edge caching

**3. Application Cache:**
- In-memory (local)
- Process-level caching
- Framework caching (Spring Cache)

**4. Distributed Cache:**
- Redis, Memcached
- Shared across instances
- Session storage

**5. Database Cache:**
- Query result cache
- Buffer pool
- Materialized views

```java
// Multi-level caching
@Service
public class UserService {
    // L1 Cache: Local
    private final Map<Long, User> localCache = new ConcurrentHashMap<>();
    
    // L2 Cache: Distributed
    @Autowired
    private RedisTemplate<String, User> redisTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    public User getUser(Long id) {
        // Check L1
        User user = localCache.get(id);
        if (user != null) return user;
        
        // Check L2
        String key = "user:" + id;
        user = redisTemplate.opsForValue().get(key);
        if (user != null) {
            localCache.put(id, user);
            return user;
        }
        
        // Load from database
        user = userRepository.findById(id).orElse(null);
        if (user != null) {
            redisTemplate.opsForValue().set(key, user, 1, TimeUnit.HOURS);
            localCache.put(id, user);
        }
        
        return user;
    }
}
```

## Caching Strategies

### Q16: Explain different caching strategies.
**Answer:**

**1. Cache-Aside (Lazy Loading):**
```java
public User getUser(Long id) {
    String key = "user:" + id;
    User user = cache.get(key);
    
    if (user == null) {
        user = database.findById(id);
        if (user != null) {
            cache.set(key, user, TTL);
        }
    }
    return user;
}

public void updateUser(User user) {
    database.save(user);
    cache.delete("user:" + user.getId());
}
```

**2. Write-Through:**
```java
public void saveUser(User user) {
    cache.set("user:" + user.getId(), user);
    database.save(user);  // Synchronous
}
```

**3. Write-Behind (Write-Back):**
```java
public class WriteBackCache {
    private final Map<String, Object> cache = new ConcurrentHashMap<>();
    private final Queue<WriteOperation> writeQueue = new LinkedBlockingQueue<>();
    
    public void set(String key, Object value) {
        cache.put(key, value);
        writeQueue.offer(new WriteOperation(key, value));
    }
    
    @Scheduled(fixedDelay = 5000)
    public void flush() {
        List<WriteOperation> batch = new ArrayList<>();
        writeQueue.drainTo(batch, 100);
        
        if (!batch.isEmpty()) {
            database.batchSave(batch);
        }
    }
}
```

**4. Refresh-Ahead:**
```java
public class RefreshAheadCache {
    @Scheduled(fixedRate = 60000)
    public void refreshPopularItems() {
        List<String> popularKeys = getPopularKeys();
        
        for (String key : popularKeys) {
            if (isAboutToExpire(key)) {
                Object freshData = database.load(key);
                cache.set(key, freshData, TTL);
            }
        }
    }
}
```

### Q17: How to handle cache invalidation?
**Answer:**

**1. TTL-based:**
```java
cache.setex("user:123", 3600, userData);  // Expires in 1 hour
```

**2. Event-based:**
```java
@EventListener
public void handleUserUpdated(UserUpdatedEvent event) {
    cache.delete("user:" + event.getUserId());
    cache.delete("user:email:" + event.getEmail());
}
```

**3. Tag-based invalidation:**
```java
public class TaggedCache {
    private Map<String, Set<String>> tagToKeys = new ConcurrentHashMap<>();
    
    public void set(String key, Object value, String... tags) {
        cache.set(key, value);
        for (String tag : tags) {
            tagToKeys.computeIfAbsent(tag, k -> new HashSet<>()).add(key);
        }
    }
    
    public void invalidateTag(String tag) {
        Set<String> keys = tagToKeys.get(tag);
        if (keys != null) {
            keys.forEach(cache::delete);
            tagToKeys.remove(tag);
        }
    }
}
```

**4. Versioning:**
```java
public class VersionedCache {
    private AtomicLong version = new AtomicLong(0);
    
    public void set(String key, Object value) {
        long v = version.get();
        cache.set(key + ":" + v, value);
        cache.set(key + ":version", v);
    }
    
    public Object get(String key) {
        Long v = cache.get(key + ":version");
        if (v != null) {
            return cache.get(key + ":" + v);
        }
        return null;
    }
    
    public void invalidateAll() {
        version.incrementAndGet();
    }
}
```

## Distributed Caching

### Q18: How does Redis work as a cache?
**Answer:**

**Redis Data Structures:**
```bash
# Strings
SET user:123 '{"name":"John","age":30}'
GET user:123
SETEX session:abc 3600 "user123"  # With TTL

# Hashes
HSET user:123 name "John" age 30
HGET user:123 name
HGETALL user:123

# Lists (queues)
LPUSH queue:emails "email1"
RPOP queue:emails

# Sets
SADD user:123:friends 456 789
SMEMBERS user:123:friends

# Sorted Sets (leaderboards)
ZADD leaderboard 100 "user1" 95 "user2"
ZREVRANGE leaderboard 0 9 WITHSCORES

# Pub/Sub
PUBLISH user:123:notifications "New message"
SUBSCRIBE user:123:notifications
```

**Redis Persistence:**
1. **RDB**: Point-in-time snapshots
2. **AOF**: Append-only file log
3. **Hybrid**: RDB + AOF

**Redis Cluster:**
```java
@Configuration
public class RedisConfig {
    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        RedisClusterConfiguration clusterConfig = 
            new RedisClusterConfiguration()
                .clusterNode("redis-node1", 6379)
                .clusterNode("redis-node2", 6379)
                .clusterNode("redis-node3", 6379);
        return new LettuceConnectionFactory(clusterConfig);
    }
}
```

### Q19: How to implement distributed cache?
**Answer:**

**Consistent Hashing for Distribution:**
```java
public class DistributedCache {
    private ConsistentHash<CacheNode> consistentHash;
    
    public void put(String key, Object value) {
        CacheNode node = consistentHash.get(key);
        node.put(key, value);
        
        // Replicate to N-1 next nodes
        List<CacheNode> replicas = consistentHash.getNextNodes(key, replicationFactor - 1);
        for (CacheNode replica : replicas) {
            replica.putReplica(key, value);
        }
    }
    
    public Object get(String key) {
        CacheNode node = consistentHash.get(key);
        Object value = node.get(key);
        
        if (value == null && readRepair) {
            // Try replicas
            List<CacheNode> replicas = consistentHash.getNextNodes(key, replicationFactor - 1);
            for (CacheNode replica : replicas) {
                value = replica.get(key);
                if (value != null) {
                    node.put(key, value);  // Repair
                    break;
                }
            }
        }
        
        return value;
    }
}
```

**Cache Coherence:**
```java
public class CacheCoordinator {
    private final EventBus eventBus;
    
    public void invalidate(String key) {
        // Local invalidation
        localCache.remove(key);
        
        // Broadcast to other nodes
        eventBus.publish(new CacheInvalidationEvent(key));
    }
    
    @EventListener
    public void handleInvalidation(CacheInvalidationEvent event) {
        localCache.remove(event.getKey());
    }
}
```

## Cache Implementation

### Q20: Implement an LRU cache.
**Answer:**

```java
public class LRUCache<K, V> {
    private final int capacity;
    private final Map<K, Node<K, V>> map;
    private final DoublyLinkedList<K, V> dll;
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        this.dll = new DoublyLinkedList<>();
    }
    
    public V get(K key) {
        Node<K, V> node = map.get(key);
        if (node == null) {
            return null;
        }
        
        // Move to front (most recently used)
        dll.moveToFront(node);
        return node.value;
    }
    
    public void put(K key, V value) {
        Node<K, V> node = map.get(key);
        
        if (node != null) {
            // Update existing
            node.value = value;
            dll.moveToFront(node);
        } else {
            // Add new
            if (map.size() >= capacity) {
                // Evict LRU
                Node<K, V> lru = dll.removeLast();
                map.remove(lru.key);
            }
            
            node = new Node<>(key, value);
            dll.addFirst(node);
            map.put(key, node);
        }
    }
    
    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V> prev;
        Node<K, V> next;
        
        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
    
    private static class DoublyLinkedList<K, V> {
        private Node<K, V> head;
        private Node<K, V> tail;
        
        DoublyLinkedList() {
            head = new Node<>(null, null);
            tail = new Node<>(null, null);
            head.next = tail;
            tail.prev = head;
        }
        
        void addFirst(Node<K, V> node) {
            node.next = head.next;
            node.prev = head;
            head.next.prev = node;
            head.next = node;
        }
        
        void remove(Node<K, V> node) {
            node.prev.next = node.next;
            node.next.prev = node.prev;
        }
        
        void moveToFront(Node<K, V> node) {
            remove(node);
            addFirst(node);
        }
        
        Node<K, V> removeLast() {
            Node<K, V> last = tail.prev;
            remove(last);
            return last;
        }
    }
}

// Using LinkedHashMap
public class SimpleLRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public SimpleLRUCache(int capacity) {
        super(capacity, 0.75f, true);  // true for access-order
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

### Q21: How to monitor cache performance?
**Answer:**

**Key Metrics:**
```java
public class CacheMetrics {
    private final AtomicLong hits = new AtomicLong();
    private final AtomicLong misses = new AtomicLong();
    private final AtomicLong evictions = new AtomicLong();
    private final Histogram latency = new Histogram();
    
    public double getHitRate() {
        long total = hits.get() + misses.get();
        return total == 0 ? 0 : (double) hits.get() / total;
    }
    
    public void recordGet(String key, long startTime, boolean hit) {
        if (hit) {
            hits.incrementAndGet();
        } else {
            misses.incrementAndGet();
        }
        latency.record(System.nanoTime() - startTime);
    }
    
    @Scheduled(fixedRate = 60000)
    public void reportMetrics() {
        log.info("Cache Stats - Hit Rate: {}%, Hits: {}, Misses: {}, Evictions: {}",
            getHitRate() * 100, hits.get(), misses.get(), evictions.get());
        log.info("Latency - p50: {}ms, p99: {}ms",
            latency.getPercentile(50), latency.getPercentile(99));
    }
}
```

**Cache Warming:**
```java
@Component
public class CacheWarmer {
    @EventListener(ApplicationReadyEvent.class)
    public void warmCache() {
        // Load frequently accessed data
        List<User> popularUsers = userRepository.findTop100ByOrderByAccessCount();
        popularUsers.forEach(user -> 
            cache.put("user:" + user.getId(), user));
        
        // Load configuration data
        List<Config> configs = configRepository.findAll();
        configs.forEach(config -> 
            cache.put("config:" + config.getKey(), config.getValue()));
    }
}
```

### Q22: Cache best practices?
**Answer:**

1. **Choose appropriate TTL**: Balance freshness vs performance
2. **Use cache levels**: L1 (local) + L2 (distributed)
3. **Monitor hit rates**: Target > 80% for effective caching
4. **Implement circuit breakers**: Protect against cache failures
5. **Use async loading**: Don't block on cache misses
6. **Compress large values**: Reduce memory usage
7. **Implement cache warming**: Pre-load critical data
8. **Use consistent key naming**: "entity:id:field"
9. **Handle thundering herd**: Jitter, request coalescing
10. **Plan for cache failure**: Graceful degradation