# Feature: Distributed Systems & Architecture Content

## What

Interview preparation content covering the theoretical foundations and practical patterns of distributed computing. This feature provides comprehensive Q&A materials on:

- **Distributed Computing Fundamentals**: Understanding what makes distributed systems different from single-machine systems - network unreliability, partial failures, lack of global clock, and the challenges of achieving coordination across machines.

- **Consistency Models**: The spectrum from strong consistency (linearizability) to eventual consistency, and everything in between (sequential, causal, read-your-writes). Understanding what guarantees each model provides and its performance implications.

- **Consensus Algorithms**: How distributed systems agree on values despite failures. Deep understanding of Paxos and Raft algorithms, leader election, and why consensus is the foundation of reliable distributed systems.

- **Fault Tolerance**: Designing systems that continue operating correctly despite hardware failures, network partitions, and software bugs. Replication strategies, failure detection, and recovery mechanisms.

- **Distributed Data Patterns**: Techniques for storing and processing data across multiple nodes - partitioning, replication, distributed transactions, and distributed query processing.

## Why

Distributed systems knowledge separates senior engineers from mid-level developers:

1. **Foundation for Modern Systems**: Cloud computing, microservices, and big data all require distributed systems understanding
2. **Debugging Complex Issues**: Production problems in distributed systems require deep theoretical knowledge
3. **Architecture Decisions**: Choosing between consistency and availability requires understanding the trade-offs
4. **Interview Differentiator**: Deep distributed systems knowledge impresses interviewers at top companies

This content bridges theory and practice, helping developers understand both the "what" and "why" of distributed systems.

## Acceptance Criteria
- [ ] Covers distributed computing fundamentals
- [ ] Explains consistency models (strong, eventual, causal)
- [ ] Documents consensus algorithms (Paxos, Raft)
- [ ] Covers fault tolerance mechanisms
- [ ] Explains architectural principles (SOLID at system level)
- [ ] Documents distributed data patterns
- [ ] Covers partition tolerance strategies
- [ ] Includes real-world examples and trade-offs

## Content Sections

### 1. Distributed Computing Fundamentals
The eight fallacies of distributed computing that every developer learns the hard way: the network is reliable, latency is zero, bandwidth is infinite, etc. Understanding partial failures: some nodes fail while others continue. The lack of global time and why distributed clocks are hard. The two generals problem and why perfect coordination is impossible.

### 2. Consistency Models
**Linearizability (Strong Consistency)**: Operations appear instantaneous and in real-time order. The gold standard but expensive. **Sequential Consistency**: Operations appear in some total order consistent with program order. **Causal Consistency**: Causally related operations appear in order. **Eventual Consistency**: All replicas eventually converge. Understanding the trade-offs: stronger consistency = higher latency and lower availability.

### 3. Consensus Algorithms
Why consensus matters: agreeing on leader, transaction commit, configuration changes. **Paxos**: The original consensus algorithm - proposers, acceptors, learners, and the two-phase protocol. Why Paxos is notoriously difficult to understand and implement. **Raft**: Designed for understandability - leader election, log replication, and safety guarantees. Practical implementations: etcd (Raft), ZooKeeper (Zab, similar to Paxos).

### 4. Fault Tolerance
Types of failures: crash failures (node stops), omission failures (messages lost), Byzantine failures (arbitrary behavior). Failure detection: heartbeats, timeouts, and the phi accrual detector. Replication for fault tolerance: primary-backup, state machine replication. The FLP impossibility result: consensus is impossible in asynchronous systems with even one faulty process.

### 5. Time and Ordering
Why physical clocks fail in distributed systems: clock skew and drift. **Logical Clocks**: Lamport clocks establish partial ordering. **Vector Clocks**: Capture causality between events. **Hybrid Logical Clocks**: Combine physical and logical time for practical systems. How Google Spanner uses TrueTime for global consistency.

### 6. Distributed Transactions
**Two-Phase Commit (2PC)**: Prepare and commit phases, coordinator failure handling. Why 2PC blocks and alternatives. **Three-Phase Commit (3PC)**: Adds pre-commit phase to reduce blocking. **Saga Pattern**: Long-running transactions through compensation. Choosing between strong consistency (2PC) and eventual consistency (Saga) based on requirements.

### 7. Distributed Data Patterns
**Partitioning (Sharding)**: Hash partitioning, range partitioning, consistent hashing. Handling hot spots and rebalancing. **Replication**: Single-leader, multi-leader, leaderless architectures. Quorum reads and writes (W + R > N). Conflict resolution: last-write-wins, vector clocks, CRDTs. **Distributed Queries**: Map-reduce pattern, scatter-gather, parallel query execution.

### 8. Leader Election
Why distributed systems need leaders: coordination, conflict resolution. Election algorithms: bully algorithm, ring algorithm. Challenges: split-brain (multiple leaders), network partitions. Using external coordination services (ZooKeeper, etcd) for reliable election.

### 9. CRDTs (Conflict-free Replicated Data Types)
Data structures that automatically resolve conflicts: G-Counter, PN-Counter, G-Set, OR-Set, LWW-Register. How CRDTs enable eventual consistency without coordination. Use cases: collaborative editing, shopping carts, distributed counters. Trade-offs: limited data types, space overhead.

### 10. Real-World Distributed Systems
Case studies of production systems:
- **Cassandra**: Leaderless replication, tunable consistency, gossip protocol
- **Kafka**: Distributed commit log, partition leadership, consumer groups
- **DynamoDB**: Consistent hashing, vector clocks, sloppy quorum
- **Spanner**: Globally distributed, strongly consistent, TrueTime

## Key Concepts to Master

Critical topics for distributed systems interviews:

- **CAP theorem nuances**: It's not a simple choice; understand PACELC
- **Raft leader election**: Be able to walk through the algorithm step by step
- **Quorum math**: W + R > N and what it guarantees
- **Vector clocks**: How they detect concurrent modifications
- **Consistent hashing**: Virtual nodes and rebalancing on node changes

## Study Tips

1. **Implement simple protocols** - Build a basic Raft or gossip protocol to truly understand
2. **Draw failure scenarios** - What happens when node A fails? When network partitions?
3. **Read the papers** - Lamport's Paxos, Ongaro's Raft, Amazon's Dynamo
4. **Use real systems** - Deploy Cassandra or etcd and observe behavior
5. **Think in terms of guarantees** - What does the system promise? What can go wrong?

## Related
- [Project Intent](project-intent.md)
- [Feature: System Design](feature-system-design.md)
- [Feature: Microservices](feature-microservices.md)
- [Decision: Content Structure](../decisions/001-content-structure.md)
- [Pattern: Q&A Format](../knowledge/patterns/qa-format.md)

## Status
- **Created**: 2026-01-19 (Phase: Intent)
- **Updated**: 2026-01-19 - Enhanced content for better study experience
- **Status**: Active (already implemented)
- **File**: `distributed-systems-architecture.md`
