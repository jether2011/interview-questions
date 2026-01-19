# Feature: System Design Content

## What

Interview preparation content covering the art and science of designing large-scale distributed systems. This feature provides comprehensive Q&A materials on:

- **Scalability Fundamentals**: Understanding horizontal vs vertical scaling, stateless design principles, and how to scale each layer of an application (web, application, database, cache) independently.

- **CAP Theorem & Consistency**: The fundamental trade-offs in distributed systems between Consistency, Availability, and Partition tolerance. Understanding when to choose CP vs AP systems and their real-world implications.

- **Load Balancing**: Distributing traffic across servers using different algorithms (round-robin, least connections, IP hash), Layer 4 vs Layer 7 load balancing, health checks, and global load balancing for multi-region deployments.

- **Caching Strategies**: Multi-level caching (browser, CDN, application, database), caching patterns (cache-aside, write-through, write-behind), cache invalidation strategies, and handling cache stampede.

- **Database Scaling**: Replication for read scaling, sharding for write scaling, choosing partition keys, handling cross-shard queries, and the trade-offs of different sharding strategies.

- **Real-World System Design**: Structured approaches to designing systems like URL shorteners, social media feeds, chat applications, and video streaming platforms.

## Why

System design interviews are mandatory for senior positions. This content matters because:

1. **Senior Role Requirement**: Every FAANG-level and senior position includes system design rounds
2. **Architecture Ownership**: Shows you can make high-level technical decisions
3. **Scale Understanding**: Demonstrates experience with production systems under load
4. **Trade-off Analysis**: Proves you can balance competing requirements (cost, performance, reliability)

This content provides frameworks for approaching any system design problem systematically.

## Acceptance Criteria
- [ ] Covers scalability fundamentals (horizontal/vertical)
- [ ] Explains CAP theorem and implications
- [ ] Documents load balancing strategies
- [ ] Covers caching architectures and patterns
- [ ] Explains database sharding techniques
- [ ] Includes real-world system design examples
- [ ] Documents trade-offs for design decisions
- [ ] Covers estimation and capacity planning

## Content Sections

### 1. Scalability Fundamentals
Defines scalability and why it matters. Vertical scaling (bigger machines): simple but limited by hardware ceiling. Horizontal scaling (more machines): complex but unlimited growth potential. Stateless design principles that enable horizontal scaling. Identifying bottlenecks: CPU-bound vs I/O-bound vs memory-bound systems. The universal scalability law and why linear scaling is hard.

### 2. CAP Theorem
The fundamental theorem of distributed systems: you can only guarantee two of Consistency, Availability, and Partition tolerance. Since network partitions are unavoidable, the real choice is CP (consistent but may be unavailable) vs AP (available but may be inconsistent). Examples: banking (CP) vs social media likes (AP). PACELC extension: during normal operation, choose between Latency and Consistency.

### 3. Load Balancing
Why load balancers are essential for scalability and reliability. Algorithms: round-robin (simple, even distribution), weighted round-robin (for heterogeneous servers), least connections (for varying request durations), IP hash (for session affinity). Layer 4 (TCP/UDP) vs Layer 7 (HTTP) load balancing trade-offs. Health checks and graceful removal of unhealthy servers. Global load balancing with DNS and GeoDNS.

### 4. Caching Strategies
The caching pyramid: browser cache, CDN, reverse proxy, application cache, database cache. Cache-aside pattern (application manages cache), write-through (sync write to cache and DB), write-behind (async write to DB), refresh-ahead (proactive refresh). Cache invalidation challenges: TTL-based, event-based, version-based. Handling cache stampede with locking and probabilistic early expiration. When NOT to cache: highly personalized or frequently changing data.

### 5. Database Sharding
When replication isn't enough and you need to partition data. Sharding strategies: hash-based (even distribution), range-based (for range queries), directory-based (flexible but adds complexity). Choosing a shard key: high cardinality, even distribution, query patterns. Cross-shard query challenges and denormalization strategies. Resharding: the painful process of changing your sharding scheme.

### 6. Database Replication
Master-slave (primary-replica) for read scaling: write to master, read from replicas. Replication lag and its implications for consistency. Master-master for write availability: conflict resolution challenges. Synchronous vs asynchronous replication trade-offs. Failover strategies and split-brain prevention.

### 7. Content Delivery Networks (CDN)
How CDNs reduce latency by serving content from edge locations. Push vs pull CDN strategies. Cache headers for CDN control (Cache-Control, ETag, Vary). Origin shield to reduce origin load. CDN for dynamic content: edge computing and personalization at the edge.

### 8. Message Queues & Async Processing
Decoupling components with message queues for reliability and scalability. Use cases: background jobs, event processing, inter-service communication. Queue vs topic (pub-sub) patterns. Handling failures: dead letter queues, retry policies. Exactly-once vs at-least-once delivery trade-offs.

### 9. Real-World System Designs
Structured approach to common interview problems:
- **URL Shortener**: Hash generation, collision handling, analytics, caching
- **Twitter/Social Feed**: Fan-out on write vs fan-out on read, timeline generation
- **Chat Application**: WebSocket connections, message storage, presence detection
- **Video Streaming**: Encoding pipeline, adaptive bitrate, CDN distribution
- **Rate Limiter**: Token bucket, sliding window algorithms, distributed rate limiting

### 10. Estimation & Capacity Planning
Back-of-envelope calculations for interviews. Key numbers to memorize: QPS calculations, storage requirements, bandwidth needs. Little's Law for queue sizing. The "powers of two" table for quick math. Example: estimating storage for 1 billion photos.

## Key Concepts to Master

Essential topics for system design interviews:

- **Consistent hashing**: How it enables adding/removing nodes with minimal redistribution
- **Database indexing**: B-tree indexes, covering indexes, when indexes hurt performance
- **Connection pooling**: Why it matters and how to size pools
- **Back-pressure**: Handling systems that can't keep up with input rate
- **Idempotency**: Designing operations that can safely be retried

## Study Tips

1. **Practice the framework** - Requirements → High-level design → Deep dive → Bottlenecks
2. **Know your numbers** - Memorize latency, throughput, and storage order of magnitudes
3. **Draw as you talk** - System design is visual; practice whiteboard/diagramming
4. **Ask clarifying questions** - Scope the problem before designing
5. **Discuss trade-offs** - There's no perfect design; show you understand the costs

## Related
- [Project Intent](project-intent.md)
- [Feature: Distributed Systems](feature-distributed-systems.md)
- [Feature: Database & Caching](feature-database-caching.md)
- [Decision: Content Structure](../decisions/001-content-structure.md)
- [Pattern: Q&A Format](../knowledge/patterns/qa-format.md)

## Status
- **Created**: 2026-01-19 (Phase: Intent)
- **Updated**: 2026-01-19 - Enhanced content for better study experience
- **Status**: Active (already implemented)
- **File**: `system-design.md`
