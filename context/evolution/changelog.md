# Changelog

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
