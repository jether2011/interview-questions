# Senior Java Interview Preparation Guide

A concise, dense reference for senior Java/backend developers targeting international positions. Answers are direct, code is minimal, architecture topics use Mermaid diagrams.

## Content Files

| Topic | File | Key Areas |
|---|---|---|
| Java Fundamentals | [java-fundamentals.md](./java-fundamentals.md) | OOP pillars, polymorphism, collections, HashMap internals, exceptions, streams |
| Multithreading | [java-multithreading.md](./java-multithreading.md) | Thread contention, 10k RPS strategies, race conditions, locks, CompletableFuture |
| Spring Boot | [spring-boot.md](./spring-boot.md) | @Autowired, Bean lifecycle, @Transactional, JPA, OAuth2, Docker, EC2 |
| Microservices | [microservices-patterns.md](./microservices-patterns.md) | Service Discovery, API Gateway, Saga, CQRS, Circuit Breaker |
| System Design | [system-design.md](./system-design.md) | CAP, caching, sharding, RESTful API design, URL shortener, rate limiter |
| Distributed Systems | [distributed-systems-architecture.md](./distributed-systems-architecture.md) | CAP/PACELC, Raft, fault tolerance, Hexagonal Architecture |
| Docker & Kubernetes | [docker-kubernetes.md](./docker-kubernetes.md) | Dockerfile, K8s architecture, HPA, probes, rolling updates |
| Design Patterns | [design-patterns-solid.md](./design-patterns-solid.md) | SOLID, Singleton, Strategy, Observer, Repository, Decorator |
| Database & Caching | [database-caching.md](./database-caching.md) | SQL/NoSQL, indexes, isolation levels, sharding, Redis |
| Messaging & EDA | [messaging-event-driven.md](./messaging-event-driven.md) | Kafka, delivery guarantees, Outbox pattern, choreography vs orchestration |
| Solidity & Blockchain | [solidity-blockchain.md](./solidity-blockchain.md) | EVM E2E transactions, revert behavior, smart contract testing, gas |
| Kotlin | [kotlin-language.md](./kotlin-language.md) | Null safety, coroutines, Flow, Java interop, Android |

## Topics Covered (Interview Q&A)

### Core Java & OOP
- 4 pillars of OOP
- Polymorphism (static vs dynamic)
- Why Java doesn't support multiple inheritance
- Abstract Class vs Interface
- Diamond problem / default method conflict
- ArrayList vs LinkedList, HashSet vs TreeSet
- HashMap internals (buckets, treeify, load factor)
- Checked vs Unchecked exceptions
- synchronized, CountDownLatch
- map() vs flatMap()

### Spring Boot & Backend
- What is Spring Boot (auto-configuration, starters, embedded server)
- Spring Boot App vs Microservice
- @Autowired resolution (type → qualifier → name)
- Bean lifecycle (@PostConstruct, @PreDestroy)
- @Transactional (propagation, rollback, proxy gotcha)
- JPA / Hibernate (EntityManager states, N+1)
- Spring Security + OAuth2 (authentication vs authorization)
- Docker containerization + multi-stage Dockerfile
- EC2 deployment

### Microservices & Architecture
- Advantages and trade-offs of microservices
- Service Discovery (Eureka, Consul)
- API Gateway (routing, auth, rate limiting)
- Auto-detection of new instances (Spring Cloud Gateway)
- Circuit Breaker (state machine)
- Saga pattern (choreography + orchestration)
- CQRS and Event Sourcing
- Database per Service

### Concurrency
- Thread contention (definition, symptoms, fixes)
- Handle 10,000 RPS (WebFlux, Virtual Threads, async, scaling)
- Race conditions in distributed environments (optimistic lock, pessimistic lock, Redis lock, CAS, idempotency)
- Deadlocks
- volatile vs AtomicInteger

### System Design
- Interview framework (5 steps)
- Availability SLAs
- CAP theorem and PACELC
- Consistent hashing
- Caching strategies (cache-aside, write-through, write-behind)
- Designing a queryable RESTful API
- URL shortener and Rate limiter design

### Blockchain
- EVM transaction lifecycle E2E (Web3j)
- Transaction revert behavior
- Testing smart contracts (Testcontainers, Foundry, Hardhat fork)
- Gas optimization
- Reentrancy attack and CEI pattern

## How to Use

Study one topic at a time. For each question, read the answer then close the file and explain it in your own words. If you can't — re-read and try again.

For architecture questions, draw the diagram on paper before looking at the Mermaid source.

## License

MIT — see [LICENSE](./LICENSE)

## Author

Jether Rodrigues do Nascimento
