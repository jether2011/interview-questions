# Senior Java/Backend Interview Preparation Guide

A dense, direct reference for senior Java/backend developers targeting international positions. Content is structured by theme, uses Mermaid diagrams for architecture, and provides concise code examples that explain concepts fast.

## Content Index

| Topic | File | Key Areas |
|---|---|---|
| Java Fundamentals | [java-fundamentals.md](./java-fundamentals.md) | OOP pillars, polymorphism, collections, HashMap internals, exceptions, Streams, Java 8-21 |
| Multithreading | [java-multithreading.md](./java-multithreading.md) | Thread contention, 10k RPS strategies, race conditions, locks, CompletableFuture, Virtual Threads |
| Spring Boot | [spring-boot.md](./spring-boot.md) | Auto-configuration, @Autowired, Bean lifecycle, @Transactional, JPA, OAuth2, Docker, EC2 |
| Microservices | [microservices-patterns.md](./microservices-patterns.md) | Service Discovery, API Gateway, Circuit Breaker, Saga, CQRS, Event Sourcing |
| System Design | [system-design.md](./system-design.md) | CAP/PACELC, caching, sharding, consistent hashing, RESTful API design, rate limiter |
| Distributed Systems | [distributed-systems-architecture.md](./distributed-systems-architecture.md) | Raft/Paxos, vector clocks, CRDT, fault tolerance, DDD, Hexagonal Architecture |
| Docker & Kubernetes | [docker-kubernetes.md](./docker-kubernetes.md) | Docker internals, multi-stage builds, K8s architecture, HPA, probes, Helm, service mesh |
| Design Patterns | [design-patterns-solid.md](./design-patterns-solid.md) | SOLID deep-dive, all GoF patterns with code, enterprise patterns |
| Database & Caching | [database-caching.md](./database-caching.md) | SQL/NoSQL, B-tree indexes, MVCC, isolation levels, sharding, Redis data structures |
| Messaging & EDA | [messaging-event-driven.md](./messaging-event-driven.md) | Kafka internals, delivery guarantees, Outbox, schema evolution, EDA patterns |
| Solidity & Blockchain | [solidity-blockchain.md](./solidity-blockchain.md) | EVM E2E (Web3j), revert behavior, Testcontainers/Foundry/Hardhat testing, gas, security |
| Kotlin | [kotlin-language.md](./kotlin-language.md) | Null safety, coroutines, Flow, sealed classes, generics, Java interop, testing |

## Interview Topics Coverage

### Core Java & OOP (Q1–Q16)
- 4 OOP pillars with code examples
- Static vs dynamic polymorphism, vtable dispatch
- Diamond problem & multiple inheritance
- Abstract class vs Interface (Java 8+ default methods)
- ArrayList vs LinkedList Big-O, HashMap internals (buckets, treeify at 8, load factor 0.75)
- equals()/hashCode() contract, immutable class rules
- Checked vs unchecked exceptions, try-with-resources
- map() vs flatMap(), Optional, Records, Sealed Classes

### Multithreading & Concurrency (Q17–Q22)
- Thread lifecycle, synchronized, ReentrantLock, ReadWriteLock
- Thread contention: symptoms, jstack diagnosis, solutions
- JMM happens-before, volatile vs AtomicInteger vs LongAdder
- CountDownLatch, CyclicBarrier, Semaphore, Phaser
- ThreadPoolExecutor, CompletableFuture async composition
- **Handle 10,000 RPS**: WebFlux, Virtual Threads (Java 21), async 202, HikariCP tuning, horizontal scaling
- **Race conditions in distributed systems**: optimistic lock, pessimistic lock, Redisson distributed lock, Redis atomic, idempotency keys, event sourcing

### Spring Boot (Q23–Q30)
- Auto-configuration, starters, Spring Boot vs Microservice
- @Autowired resolution (type → qualifier → name), constructor injection
- Bean lifecycle (8 steps), @Transactional propagation + 3 gotchas
- JPA entity states, N+1 fix (JOIN FETCH, @EntityGraph, @BatchSize)
- Spring Security, OAuth2 authorization code flow (Mermaid), JWT
- Docker multi-stage Dockerfile, EC2 deployment

### Microservices Architecture (Q31–Q40)
- Advantages and trade-offs table
- Service Discovery (Eureka), API Gateway (Spring Cloud Gateway)
- Circuit Breaker state machine (Resilience4j), Bulkhead, Retry+jitter
- Communication: REST vs gRPC vs Kafka comparison
- Saga: choreography vs orchestration (both Mermaid sequences)
- CQRS + Event Sourcing, DB per service

### System Design
- 5-step framework with estimation numbers
- CAP theorem, PACELC, consistency models
- Caching strategies (cache-aside, write-through, write-behind, stampede prevention)
- Database sharding (range, hash, directory, consistent hashing)
- **Queryable RESTful API design** (resource URLs, filtering, pagination, versioning)
- URL shortener, Rate limiter (token bucket), Notification system, Chat system

### Distributed Systems
- Raft consensus algorithm, Paxos comparison
- Vector clocks, Lamport clocks, HLC (Google Spanner TrueTime)
- CRDTs, Bloom filters, HyperLogLog
- 2PC vs Saga, TCC
- Hexagonal architecture, Clean Architecture, DDD
- Anti-patterns: distributed monolith, shared DB, chatty services

### Blockchain & Solidity
- EVM transaction lifecycle E2E with Web3j Java code
- Transaction revert: require vs revert vs assert + gas behavior
- Smart contract testing: Testcontainers+Ganache JUnit5, Foundry (fuzz tests), Hardhat TypeScript
- Gas optimization (7 techniques, operation cost table)
- Security: reentrancy CEI pattern, tx.origin vs msg.sender
- Proxy/upgradeability pattern with delegatecall

## Study Method

1. **Theme-by-theme**: read one file per session, don't skip around
2. **Close and explain**: after reading an answer, close the file and explain it aloud — if you can't, re-read
3. **Draw first**: for architecture questions, sketch the diagram on paper before looking at the Mermaid source
4. **Code from memory**: close the file and write the code example from scratch
5. **Teach it**: explain concepts as if interviewing someone else

## Format Conventions

- Tables for comparison — fast to scan
- Mermaid diagrams for sequence flows and architecture
- Concise code (explains the concept, not production-ready verbosity)
- No obvious statements — if it's basic knowledge, it's skipped

## Author

Jether Rodrigues do Nascimento — [jetherrodrigues@gmail.com](mailto:jetherrodrigues@gmail.com)
