# Feature: Messaging & Event-Driven Architecture Content

## What

Interview preparation content covering asynchronous communication patterns and event-driven architecture essential for building scalable, loosely-coupled systems. This feature provides comprehensive Q&A materials on:

- **Message Queue Fundamentals**: Understanding the role of message queues in distributed systems - decoupling producers from consumers, handling traffic spikes, ensuring reliable message delivery, and enabling asynchronous processing.

- **Apache Kafka**: Deep dive into Kafka's architecture - topics, partitions, consumer groups, brokers, and ZooKeeper/KRaft. Understanding Kafka's log-based design, how it achieves high throughput, and when to choose Kafka over traditional message queues.

- **RabbitMQ**: AMQP protocol, exchange types (direct, fanout, topic, headers), queues, bindings, and message routing. Understanding acknowledgments, persistence, and RabbitMQ's reliability guarantees.

- **Event-Driven Architecture**: Designing systems around events rather than commands. Event notifications vs event-carried state transfer vs event sourcing. Benefits (loose coupling, scalability) and challenges (eventual consistency, debugging).

- **Delivery Guarantees**: Understanding at-most-once, at-least-once, and exactly-once semantics. How different systems achieve these guarantees and the trade-offs involved.

- **Error Handling Patterns**: Dead letter queues, retry strategies, poison message handling, and compensating transactions for maintaining system reliability.

## Why

Messaging is fundamental to modern distributed systems:

1. **Scalability**: Async processing enables handling variable loads without blocking
2. **Resilience**: Queues provide buffers that absorb failures and traffic spikes
3. **Microservices Communication**: Events enable loose coupling between services
4. **Interview Focus**: Messaging patterns are common in system design interviews

This content helps developers understand when to use messaging, which technology fits their needs, and how to handle the challenges of asynchronous communication.

## Acceptance Criteria
- [ ] Covers message queue fundamentals
- [ ] Explains Apache Kafka architecture and patterns
- [ ] Documents RabbitMQ concepts and exchange types
- [ ] Covers event-driven architecture patterns
- [ ] Explains message delivery guarantees (at-least-once, exactly-once)
- [ ] Documents event sourcing integration
- [ ] Covers dead letter queues and error handling
- [ ] Includes comparison of messaging technologies

## Content Sections

### 1. Message Queue Fundamentals
Why message queues exist: decoupling, load leveling, reliable delivery. **Producers**: Applications that send messages. **Consumers**: Applications that receive messages. **Queue**: Buffer that stores messages until consumed. **Broker**: Server that manages queues and routes messages.

Key concepts: message persistence, acknowledgments, message ordering, competing consumers pattern. When to use queues: async processing, workload distribution, system integration.

### 2. Apache Kafka Architecture
Kafka's unique design: distributed commit log, not a traditional queue.

**Core Components**:
- **Topics**: Named streams of records, divided into partitions
- **Partitions**: Ordered, immutable sequence of records, enable parallelism
- **Brokers**: Servers that store data and serve clients
- **ZooKeeper/KRaft**: Cluster coordination (KRaft is the new controller)
- **Producers**: Write to topic partitions (key-based or round-robin)
- **Consumer Groups**: Parallel consumption, partition assignment

**Key Features**: High throughput, horizontal scalability, message retention (replay capability), exactly-once semantics.

### 3. Kafka Patterns and Best Practices
**Partitioning strategy**: Choosing partition keys for even distribution and ordering requirements.

**Consumer group design**: One partition per consumer, rebalancing behavior, static membership.

**Exactly-once semantics**: Idempotent producers, transactional messaging, read-committed isolation.

**Schema management**: Avro schemas, Schema Registry, compatibility modes (backward, forward, full).

**Kafka Streams**: Stream processing library, KTables, joins, windowing.

**Kafka Connect**: Connectors for integrating with databases, file systems, cloud services.

### 4. RabbitMQ Concepts
AMQP model: producers, exchanges, bindings, queues, consumers.

**Exchange Types**:
- **Direct**: Route by exact routing key match
- **Fanout**: Broadcast to all bound queues (pub/sub)
- **Topic**: Route by routing key pattern matching (wildcards)
- **Headers**: Route by message headers

**Message Flow**: Producer → Exchange → (binding) → Queue → Consumer

**Reliability Features**: Message acknowledgments (manual vs auto), persistent messages, durable queues, publisher confirms.

### 5. RabbitMQ Patterns
**Work queues**: Distribute tasks among multiple workers, prefetch limit for fair dispatch.

**Publish/Subscribe**: Fanout exchange for broadcasting to all subscribers.

**Routing**: Direct exchange for selective routing based on routing key.

**Topics**: Flexible routing with patterns (audit.*, *.error).

**RPC**: Request-reply pattern using correlation IDs and reply-to queues.

**Priority queues**: Process urgent messages first.

### 6. Event-Driven Architecture Patterns
**Event Notification**: Minimal events (just IDs), consumers query for details. Simple but can cause load on source.

**Event-Carried State Transfer**: Events contain all data needed, no callbacks required. Enables autonomy but larger payloads.

**Event Sourcing**: Store state as sequence of events, not current state. Full audit trail, temporal queries, event replay. Complexity cost.

**CQRS with Events**: Commands produce events, events update read models. Separate optimization of reads and writes.

### 7. Delivery Guarantees
**At-most-once**: Fire and forget. No retries, possible message loss. Highest throughput, lowest latency.

**At-least-once**: Retry until acknowledged. Possible duplicates. Most common choice.

**Exactly-once**: Each message processed exactly once. Requires idempotent consumers or transactional support. Highest complexity.

**Achieving exactly-once**:
- Idempotent consumers: Processing same message twice has same effect
- Deduplication: Track processed message IDs
- Transactions: Kafka transactions, outbox pattern

### 8. Error Handling Patterns
**Dead Letter Queue (DLQ)**: Messages that fail processing go to separate queue for analysis. Prevents blocking, enables manual intervention.

**Retry Strategies**:
- Immediate retry: For transient failures
- Exponential backoff: Increasing delays between retries
- Retry with delay queue: Move to delayed queue, retry later

**Poison Message Handling**: Messages that always fail. Detect (retry count), isolate (DLQ), alert, investigate.

**Circuit Breaker**: Stop consuming when downstream is failing. Prevent cascading failures.

**Compensating Transactions**: Undo effects of failed operations in saga patterns.

### 9. Kafka vs RabbitMQ Comparison
| Aspect | Kafka | RabbitMQ |
|--------|-------|----------|
| Model | Distributed log | Message broker |
| Ordering | Per partition | Per queue |
| Retention | Configurable (time/size) | Until consumed |
| Replay | Yes (consumer offset) | No (deleted after ack) |
| Throughput | Very high | High |
| Latency | Higher | Lower |
| Use case | Event streaming, logs | Task queues, RPC |

**Choose Kafka when**: Event streaming, log aggregation, high throughput, replay needed.

**Choose RabbitMQ when**: Complex routing, RPC, low latency, traditional messaging.

### 10. Production Considerations
**Monitoring**: Consumer lag, queue depth, broker health, message rates.

**Scaling**:
- Kafka: Add partitions, add consumers (up to partition count)
- RabbitMQ: Clustering, federation, shovel for distribution

**Security**: Authentication (SASL), authorization (ACLs), encryption (TLS).

**Disaster Recovery**: Replication, multi-region deployment, backup strategies.

## Key Concepts to Master

Essential topics for messaging interviews:

- **Kafka partition strategy**: Explain how partitioning affects ordering and parallelism
- **Consumer group rebalancing**: What happens when consumers join/leave
- **RabbitMQ exchange types**: Know when to use each type
- **Exactly-once semantics**: Explain the challenges and solutions
- **Dead letter queue design**: How to implement and monitor

## Study Tips

1. **Set up locally** - Run Kafka and RabbitMQ, produce and consume messages
2. **Simulate failures** - Kill consumers, brokers; observe recovery behavior
3. **Measure throughput** - Understand the performance characteristics
4. **Implement patterns** - Build a saga, implement retry with DLQ
5. **Design systems** - Practice choosing messaging for system design problems

## Related
- [Project Intent](project-intent.md)
- [Feature: Microservices](feature-microservices.md)
- [Feature: Distributed Systems](feature-distributed-systems.md)
- [Decision: Content Structure](../decisions/001-content-structure.md)
- [Pattern: Q&A Format](../knowledge/patterns/qa-format.md)

## Status
- **Created**: 2026-01-19 (Phase: Intent)
- **Updated**: 2026-01-19 - Enhanced content for better study experience
- **Status**: Active (already implemented)
- **File**: `messaging-event-driven.md`
