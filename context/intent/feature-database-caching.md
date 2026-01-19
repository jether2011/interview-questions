# Feature: Database & Caching Content

## What

Interview preparation content covering data persistence and caching strategies essential for building high-performance applications. This feature provides comprehensive Q&A materials on:

- **SQL Database Fundamentals**: Relational database concepts, normalization, SQL query optimization, joins, subqueries, and window functions. Understanding execution plans and how databases process queries.

- **NoSQL Databases**: Understanding when to choose NoSQL over relational databases. Document stores (MongoDB), key-value stores (Redis), wide-column stores (Cassandra), and graph databases (Neo4j) - their strengths, weaknesses, and appropriate use cases.

- **Transaction Management**: ACID properties, isolation levels (Read Uncommitted, Read Committed, Repeatable Read, Serializable), and their trade-offs. Understanding phenomena like dirty reads, non-repeatable reads, and phantom reads.

- **Indexing Strategies**: How indexes work (B-tree, B+ tree, hash indexes), when to create indexes, covering indexes, composite indexes, and the cost of over-indexing. Reading execution plans to verify index usage.

- **Caching Patterns**: Cache-aside, read-through, write-through, write-behind patterns. Cache invalidation strategies, TTL management, and handling cache stampede.

- **Redis Deep Dive**: Redis data structures (strings, lists, sets, sorted sets, hashes, streams), persistence options (RDB, AOF), clustering, and common use cases beyond simple caching.

## Why

Data management is at the heart of every application:

1. **Performance Impact**: Database queries are often the slowest part of an application
2. **Scalability**: Proper caching and database design enable systems to handle growth
3. **Interview Focus**: Data-related questions appear in almost every senior interview
4. **Real-World Problem Solving**: Most production issues involve data in some way

This content helps developers make informed decisions about data storage and retrieval strategies.

## Acceptance Criteria
- [ ] Covers SQL database fundamentals and advanced queries
- [ ] Explains NoSQL databases and use cases
- [ ] Documents query optimization techniques
- [ ] Covers transaction management and isolation levels
- [ ] Explains indexing strategies (B-tree, hash, etc.)
- [ ] Documents caching patterns (cache-aside, write-through, etc.)
- [ ] Covers Redis data structures and use cases
- [ ] Includes performance tuning guidelines

## Content Sections

### 1. SQL Fundamentals
Core relational concepts: tables, rows, columns, primary keys, foreign keys. Normalization forms (1NF, 2NF, 3NF, BCNF) and when to denormalize for performance. SQL basics: SELECT, INSERT, UPDATE, DELETE, and the importance of WHERE clauses.

### 2. Advanced SQL
**Joins**: INNER, LEFT/RIGHT OUTER, FULL OUTER, CROSS joins - when to use each. **Subqueries**: Correlated vs non-correlated, EXISTS vs IN performance. **Window Functions**: ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD, running totals, and moving averages. **CTEs**: Common Table Expressions for readable, maintainable queries. **Set Operations**: UNION, INTERSECT, EXCEPT.

### 3. Query Optimization
Reading execution plans: table scans vs index scans vs index seeks. Identifying slow queries with EXPLAIN/EXPLAIN ANALYZE. Common optimizations: avoiding SELECT *, using appropriate indexes, rewriting subqueries as joins, limiting result sets. Query hints and when to use them (rarely). Statistics and how the query optimizer uses them.

### 4. Transactions and ACID
**Atomicity**: All-or-nothing execution. **Consistency**: Database moves from one valid state to another. **Isolation**: Concurrent transactions don't interfere. **Durability**: Committed data survives failures.

**Isolation Levels**:
- Read Uncommitted: Dirty reads possible (rarely used)
- Read Committed: Default in PostgreSQL, no dirty reads
- Repeatable Read: Default in MySQL InnoDB, consistent reads
- Serializable: Full isolation, lowest concurrency

Phenomena: dirty reads, non-repeatable reads, phantom reads, and which isolation levels prevent each.

### 5. Indexing Strategies
**B-tree indexes**: The default, good for equality and range queries. **Hash indexes**: O(1) lookup, only equality queries. **Covering indexes**: Include all columns needed, avoiding table lookups. **Composite indexes**: Column order matters (leftmost prefix rule). **Partial indexes**: Index subset of rows for specific queries.

Index trade-offs: faster reads, slower writes, storage overhead. When NOT to index: low cardinality columns, frequently updated columns, small tables.

### 6. NoSQL Databases
**Document Stores (MongoDB)**: Flexible schema, JSON documents, nested data. Good for content management, catalogs, user profiles.

**Key-Value Stores (Redis, DynamoDB)**: Simple get/put operations, extreme performance. Good for sessions, caching, real-time data.

**Wide-Column Stores (Cassandra, HBase)**: Column families, high write throughput, no joins. Good for time-series, IoT data, activity logs.

**Graph Databases (Neo4j)**: Nodes and relationships, traversal queries. Good for social networks, recommendations, fraud detection.

Choosing between SQL and NoSQL: data model fit, consistency requirements, query patterns, scale requirements.

### 7. Caching Patterns
**Cache-Aside (Lazy Loading)**: Application checks cache, loads from DB on miss, populates cache. Most common pattern. Stale data risk.

**Read-Through**: Cache sits in front of DB, transparently loads on miss. Simplifies application code.

**Write-Through**: Writes go to cache and DB synchronously. Consistency guaranteed, but write latency increased.

**Write-Behind (Write-Back)**: Writes go to cache, async flush to DB. Low write latency, but data loss risk.

**Cache Invalidation**: TTL-based (simple but can be stale), event-based (complex but consistent), version-based (optimistic approach).

### 8. Redis Deep Dive
**Data Structures**:
- Strings: Simple values, counters (INCR), bit operations
- Lists: Queues (LPUSH/RPOP), stacks (LPUSH/LPOP), capped lists
- Sets: Unique values, set operations (intersection, union)
- Sorted Sets: Leaderboards, rate limiting, priority queues
- Hashes: Object storage, partial updates
- Streams: Event streaming, consumer groups

**Persistence**: RDB (point-in-time snapshots), AOF (append-only file), or both.

**Clustering**: Redis Cluster for horizontal scaling, hash slots, replica failover.

**Use Cases**: Session storage, rate limiting, leaderboards, pub/sub messaging, distributed locks.

### 9. Cache Stampede and Solutions
The problem: cache expires, many concurrent requests all try to rebuild it, overwhelming the database.

**Solutions**:
- Locking: Only one request rebuilds, others wait
- Probabilistic early expiration: Random early refresh
- Background refresh: Separate process refreshes before expiration
- Circuit breaker: Limit DB requests during rebuilding

### 10. Performance Tuning
**Database level**: Connection pooling (HikariCP), query caching, buffer pool sizing, slow query logging.

**Application level**: Batch operations, lazy loading vs eager loading (N+1 problem), read replicas for read scaling.

**Monitoring**: Query performance metrics, cache hit rates, connection pool utilization. Tools: pgAdmin, MySQL Workbench, Redis CLI, APM solutions.

## Key Concepts to Master

Critical topics for database/caching interviews:

- **Index selection**: Given a query, design the optimal index
- **Isolation levels**: Explain trade-offs, especially Read Committed vs Repeatable Read
- **N+1 problem**: Identify it, explain solutions (batch loading, join fetch)
- **Cache invalidation**: Explain why it's hard and strategies to handle it
- **Redis data structure selection**: Choose the right structure for a use case

## Study Tips

1. **Write queries** - Practice SQL on real datasets (LeetCode, HackerRank SQL problems)
2. **Read execution plans** - Use EXPLAIN on your queries, understand what you see
3. **Set up Redis locally** - Experiment with data structures, TTL, persistence
4. **Design schemas** - Practice normalizing, then denormalizing for specific queries
5. **Calculate trade-offs** - Cache hit rate impact on response time, index cost on writes

## Related
- [Project Intent](project-intent.md)
- [Feature: System Design](feature-system-design.md)
- [Decision: Content Structure](../decisions/001-content-structure.md)
- [Pattern: Q&A Format](../knowledge/patterns/qa-format.md)

## Status
- **Created**: 2026-01-19 (Phase: Intent)
- **Updated**: 2026-01-19 - Enhanced content for better study experience
- **Status**: Active (already implemented)
- **File**: `database-caching.md`
