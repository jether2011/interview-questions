# Changelog

## [2026-04-15] - Second Comprehensive Rewrite: Maximum Depth + Kotlin Update

### Summary
All 12 content files received comprehensive rewrites targeting 600-900 lines each. Philosophy: maximum summarized information, concise code, tables for comparison, Mermaid diagrams for architecture. Kotlin file restructured from verbose Q&A format to thematic sections.

### Changed (second-pass comprehensive rewrite)

**messaging-event-driven.md**
- Added: Kafka producer reliability settings (YAML), consumer group rebalancing, Kafka transactions (exactly-once)
- Added: Schema Registry flow (Mermaid sequence), schema evolution rules table (backward/forward/full)
- Added: Dead Letter Queue (DLQ) config + DefaultErrorHandler, Event Sourcing with OrderAggregate code
- Added: Correlation ID propagation across Kafka headers, back-pressure consumer scaling guide
- Added: inter-service communication comparison (REST/gRPC/Kafka), batch consumer pattern

**distributed-systems-architecture.md**
- Added: 8 key metrics table (latency, throughput, availability, etc.)
- Added: Raft log replication flow (step-by-step), quorum formula (R + W > N)
- Added: Lamport clock, Vector clock, HLC, Google Spanner TrueTime, why not NTP
- Added: CRDTs (G-Counter, PN-Counter, LWW, OR-Set), Bloom filters, HyperLogLog
- Added: SWIM protocol, Phi Accrual failure detection
- Added: DDD key concepts table (Aggregate, Bounded Context, Value Object), Aggregate root code
- Added: Strangler Fig pattern, distributed systems anti-patterns table

**database-caching.md**
- Added: B-Tree vs Hash index internals, PostgreSQL index types table (GIN, GiST, BRIN, Partial)
- Added: MVCC deep-dive (xmin/xmax row headers, snapshot isolation mechanism)
- Added: HikariCP pool sizing formula and YAML config
- Added: Redis data structures with use cases table, sliding window rate limiting with Sorted Set
- Added: Redis Cluster vs Sentinel comparison, Redisson distributed lock with Watchdog explanation
- Added: Database selection flowchart (Mermaid), N+1 fix examples (@EntityGraph, @BatchSize)

**docker-kubernetes.md**
- Added: Container vs VM comparison table, Linux primitives (namespaces, cgroups, OverlayFS, seccomp)
- Added: Docker network modes table, Volumes vs Bind Mounts
- Added: StatefulSet YAML with volumeClaimTemplates, guarantees explained
- Added: NetworkPolicy YAML (micro-segmentation), Ingress YAML with TLS + cert-manager
- Added: Pod scheduling controls (nodeAffinity, podAntiAffinity, taints/tolerations)
- Added: Helm commands + chart structure, Istio VirtualService for canary deployments
- Added: Deployment strategies table (Rolling, Blue/Green, Canary, Recreate) + Mermaid

**design-patterns-solid.md**
- Added: SOLID with full code examples (SRP bad/good, OCP Strategy pattern, LSP Rectangle/Square)
- Added: ISP with Worker/Robot example, DIP with injection comparison
- Added: Abstract Factory, Prototype patterns with code
- Added: Composite pattern (pricing components), Facade pattern
- Added: Chain of Responsibility, State machine pattern, Template Method
- Added: Event Sourcing aggregate, CQRS command/query separation code
- Added: Pattern selection flowchart (Mermaid), "When to Use What" table

**kotlin-language.md**
- Restructured: replaced 100 verbose Q&As with 12 thematic sections
- Kept: all key content — null safety operators, sealed classes, coroutines, Flow
- Added: scope functions comparison table with use cases
- Added: Value classes (inline classes) for type safety
- Added: MockK testing patterns, coroutine testing with runTest
- Added: Kotlin idioms quick reference section

### Previous Rewrite
## [2026-04-15] - Major Content Rewrite: Conciseness & New Topics

### Philosophy Change
All content files rewritten from **verbose Q&A books** to **dense, interview-focused reference sheets**. Code examples are now minimal and explanatory, not exhaustive. Every answer is direct — no filler.

### Changed (all 10 main content files rewritten)

**java-fundamentals.md**
- Rewrote all OOP sections: Polymorphism, Loose Coupling, Abstract vs Interface, Multiple Inheritance
- Added: Diamond Problem explanation, interface default method conflict resolution
- Tightened all code examples to < 20 lines
- Added: ArrayList vs LinkedList, HashSet vs TreeSet, HashMap internals (treeify, load factor), Checked vs Unchecked exceptions, CountDownLatch, map() vs flatMap()

**spring-boot.md**
- Rewrote: What is Spring Boot, @Autowired internals (3-step resolution order), Bean lifecycle, @Transactional (gotchas: proxy bypass, propagation, rollbackFor)
- Added: Spring Boot App vs Microservice comparison table, OAuth2 sequence diagram (Mermaid), Docker containerization with multi-stage Dockerfile, EC2 deployment diagram

**microservices-patterns.md**
- Added Mermaid diagrams throughout (service discovery flow, API gateway, circuit breaker state machine, saga choreography/orchestration, CQRS read/write separation)
- Rewrote service discovery, API gateway with auto-detection explanation
- Added: how Spring Cloud Gateway discovers new instances automatically

**system-design.md**
- Added: "Designing a Queryable RESTful API" (my approach) — resource URLs, filtering/pagination, response envelope, versioning, error format, HATEOAS, performance considerations
- Added Mermaid diagrams: horizontal scaling, CAP theorem, sharding, RESTful API flow, URL shortener, rate limiter
- Rewrote all sections to be reference-first, explanation-second

**java-multithreading.md**
- Added: "Thread Contention" — definition, symptoms, causes/fixes table, diagnosis with jstack
- Added: "Handle 10,000 Requests in a Java Microservice" — 5 strategies (reactive/WebFlux, virtual threads Java 21, thread pool tuning, async processing, horizontal scaling)
- Added: "Race Conditions in Distributed Environments" — 5 solutions (optimistic locking, pessimistic locking, Redis distributed lock, atomic CAS, idempotency key)
- Tightened all existing examples

**solidity-blockchain.md**
- Added: "EVM Transaction Lifecycle E2E" — full Mermaid sequence diagram, Java/Web3j code for nonce management, gas estimation, signing, receipt parsing, key management
- Added: "What Happens When a Transaction is Reverted?" — Mermaid flowchart, require/revert/assert comparison, Java handling of receipt.isStatusOK()
- Added: "Testing API Integration with Smart Contracts" — 3 strategies: Testcontainers+Ganache (Java), Foundry unit tests (Solidity), Hardhat fork
- Rewrote all sections removing hallucinated/incorrect content

**distributed-systems-architecture.md**
- Added Mermaid state diagram for Raft consensus
- Added PACELC extension to CAP
- Added Bulkhead pattern with code
- Added Hexagonal Architecture diagram
- Trimmed verbose explanations

**messaging-event-driven.md**
- Added Mermaid diagrams: queue/pubsub/stream comparison, Kafka cluster topology, choreography vs orchestration
- Added: Commands vs Events table, Outbox pattern diagram, Claim Check pattern, Content-Based Router
- Added: EDA design principles section (events as facts, schema evolution, governance, coupling spectrum)

**database-caching.md**
- Added: SQL vs NoSQL decision guide, covering index example, ACID definition, isolation levels table, sharding strategies comparison
- Tightened Redis examples to actionable code

**docker-kubernetes.md**
- Added Mermaid diagram: Kubernetes control plane + nodes
- Added: HPA YAML, rolling update strategy, liveness/readiness probe YAML
- Explained `UseContainerSupport` JVM flag
- Rewrote Dockerfile with layer caching explanation

**design-patterns-solid.md**
- Rewrote all patterns to be minimal and concrete
- Added Specification pattern
- SOLID table with "violation sign" column for quick recognition

### Context Files
- Updated `changelog.md` (this file)
- Updated `project-intent.md`

### Source Materials Used
- Java Challengers book (Rafael del Nero)
- Java Interview Cheat Sheet (Rafael del Nero)
- Java Algorithms Interview Challenger (Rafael del Nero)
- Java Systems Design Interview Challenger (Rafael del Nero)
- EDA Visuals (boyney.io)
- Interview with Bunny system design reference

---

## Previous History

### [2026-01-19] - Added Kotlin Language Feature
### [2026-01-19] - Added Solidity & Blockchain Feature
### [2026-01-19] - Major Content Expansion
### [2026-01-19] - Enhanced Feature Documentation
### [2026-01-19] - Context Mesh Added
