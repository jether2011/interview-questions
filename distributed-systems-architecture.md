# Distributed Systems & Architecture Interview Questions

## Table of Contents
1. [Distributed Systems Fundamentals](#distributed-systems-fundamentals)
2. [Architecture Styles](#architecture-styles)
3. [Consistency & Consensus](#consistency--consensus)
4. [Fault Tolerance & Reliability](#fault-tolerance--reliability)
5. [Distributed Computing](#distributed-computing)
6. [Network & Communication](#network--communication)
7. [Distributed Storage](#distributed-storage)
8. [Clock & Ordering](#clock--ordering)
9. [Architecture Principles](#architecture-principles)

## Distributed Systems Fundamentals

### Q1: What are the challenges of distributed systems?
**Answer:**
1. **Network Unreliability**: Messages can be lost, delayed, or duplicated
2. **Partial Failures**: Some components fail while others work
3. **Lack of Global Clock**: No single source of truth for time
4. **Concurrency**: Multiple operations happening simultaneously
5. **Consistency**: Keeping data synchronized across nodes
6. **Scalability**: Adding resources should improve performance
7. **Security**: Multiple attack surfaces
8. **Debugging**: Hard to reproduce issues

### Q2: Explain the Eight Fallacies of Distributed Computing.
**Answer:**
1. **The network is reliable** - Networks fail
2. **Latency is zero** - Network calls take time
3. **Bandwidth is infinite** - Limited bandwidth
4. **The network is secure** - Security threats exist
5. **Topology doesn't change** - Network structure changes
6. **There is one administrator** - Multiple admins
7. **Transport cost is zero** - Network has costs
8. **The network is homogeneous** - Different systems/protocols

### Q3: What is the difference between parallel and distributed computing?
**Answer:**
| Parallel Computing | Distributed Computing |
|-------------------|----------------------|
| Shared memory | Distributed memory |
| Tightly coupled | Loosely coupled |
| Low latency communication | High latency communication |
| Homogeneous systems | Heterogeneous systems |
| Focus on performance | Focus on scalability & availability |

## Architecture Styles

### Q4: Compare different architecture styles.
**Answer:**
**1. Monolithic:**
- Single deployable unit
- Shared memory and resources
- Simple but hard to scale

**2. Service-Oriented (SOA):**
- Services communicate through ESB
- Reusable services
- Heavy middleware

**3. Microservices:**
- Small, independent services
- Decentralized
- Complex but scalable

**4. Serverless:**
- Functions as units
- No server management
- Event-driven

**5. Event-Driven:**
- Loose coupling through events
- Asynchronous communication
- Good for real-time systems

### Q5: What is Hexagonal Architecture (Ports and Adapters)?
**Answer:**
Hexagonal architecture isolates core business logic from external concerns.

```
         [HTTP Adapter] ← [Port]
                ↓
    [Core Business Logic]
                ↑
         [Database Adapter] ← [Port]
```

**Components:**
- **Core**: Business logic
- **Ports**: Interfaces defining operations
- **Adapters**: Implementations for external systems

**Benefits:**
- Testability
- Technology independence
- Clear boundaries

### Q6: Explain Clean Architecture principles.
**Answer:**
**Layers (outside-in):**
1. **Frameworks & Drivers**: UI, Database, External interfaces
2. **Interface Adapters**: Controllers, Presenters, Gateways
3. **Use Cases**: Application business rules
4. **Entities**: Enterprise business rules

**Dependency Rule:** Dependencies point inward only

```java
// Entity (innermost)
public class Order {
    private String id;
    private BigDecimal total;
    // Business rules
}

// Use Case
public class CreateOrderUseCase {
    private OrderRepository repository;
    
    public Order execute(OrderRequest request) {
        Order order = new Order(request);
        return repository.save(order);
    }
}

// Interface Adapter
@RestController
public class OrderController {
    private CreateOrderUseCase createOrder;
    
    @PostMapping("/orders")
    public OrderResponse create(@RequestBody OrderRequest request) {
        Order order = createOrder.execute(request);
        return new OrderResponse(order);
    }
}
```

## Consistency & Consensus

### Q7: Explain different consistency models in detail.
**Answer:**
1. **Linearizability (Strong Consistency)**:
   - Operations appear instantaneous
   - Total order of all operations
   - Most intuitive but expensive

2. **Sequential Consistency**:
   - Operations from each process in program order
   - All processes see same order
   - Weaker than linearizability

3. **Causal Consistency**:
   - Causally related operations ordered
   - Concurrent operations can be seen differently
   
4. **Eventual Consistency**:
   - All replicas converge eventually
   - No ordering guarantees
   - Used in AP systems

5. **Read Your Writes**:
   - Process sees its own writes
   - Others may not see them yet

6. **Monotonic Reads**:
   - Once read, won't see older value
   - Prevents going back in time

### Q8: How does the Raft consensus algorithm work?
**Answer:**
**Raft Components:**
1. **Leader Election**
2. **Log Replication**
3. **Safety**

**States:**
- **Follower**: Passive, responds to leader
- **Candidate**: Wants to become leader
- **Leader**: Handles all client requests

**Process:**
```
1. Timeout triggers election
2. Candidate requests votes
3. Majority votes → becomes leader
4. Leader sends heartbeats
5. Client requests go to leader
6. Leader replicates to followers
7. Majority acknowledgment → commit
```

**Implementation sketch:**
```java
public class RaftNode {
    enum State { FOLLOWER, CANDIDATE, LEADER }
    
    private State state = State.FOLLOWER;
    private int currentTerm = 0;
    private String votedFor = null;
    private List<LogEntry> log = new ArrayList<>();
    
    public void handleTimeout() {
        if (state != State.LEADER) {
            becomeCandidate();
        }
    }
    
    private void becomeCandidate() {
        state = State.CANDIDATE;
        currentTerm++;
        votedFor = myId;
        int votes = 1; // Vote for self
        
        for (Node peer : peers) {
            if (peer.requestVote(currentTerm, myId, lastLogIndex, lastLogTerm)) {
                votes++;
            }
        }
        
        if (votes > peers.size() / 2) {
            becomeLeader();
        }
    }
}
```

### Q9: What is the Byzantine Generals Problem?
**Answer:**
Problem where distributed nodes must agree despite some nodes being faulty or malicious.

**Requirements:**
- Agreement despite Byzantine (arbitrary) failures
- Needs 3f+1 nodes to tolerate f Byzantine nodes

**Solutions:**
- **PBFT** (Practical Byzantine Fault Tolerance)
- **Blockchain** consensus (PoW, PoS)

## Fault Tolerance & Reliability

### Q10: How to design for failure in distributed systems?
**Answer:**
**Principles:**
1. **Assume Failure**: Everything can fail
2. **Fail Fast**: Detect and handle quickly
3. **Graceful Degradation**: Partial functionality better than none
4. **Redundancy**: No single point of failure
5. **Health Checks**: Monitor component health
6. **Circuit Breakers**: Prevent cascade failures
7. **Timeouts**: Don't wait forever
8. **Retries with Backoff**: Handle transient failures

**Implementation:**
```java
@Component
public class ResilientService {
    
    @CircuitBreaker(name = "external-service")
    @Retry(name = "external-service")
    @Bulkhead(name = "external-service")
    public String callExternalService() {
        return externalClient.call();
    }
    
    @Recover
    public String fallback(Exception e) {
        return "Fallback response";
    }
}
```

### Q11: Explain different types of failures in distributed systems.
**Answer:**
1. **Crash Failure**: Node stops working
2. **Omission Failure**: Message lost
3. **Timing Failure**: Response too slow
4. **Byzantine Failure**: Arbitrary/malicious behavior
5. **Network Partition**: Network splits

**Detection Methods:**
- Heartbeats
- Timeouts
- Checksums
- Acknowledgments
- Gossip protocols

### Q12: What is the Split-Brain problem?
**Answer:**
Split-brain occurs when a network partition causes multiple nodes to believe they're the leader.

**Problem:**
- Network partition divides cluster
- Each partition elects a leader
- Data inconsistency and conflicts

**Solutions:**
1. **Quorum-based**: Majority required for operations
2. **Fencing**: Block old leader from resources
3. **STONITH** (Shoot The Other Node In The Head)
4. **Generation Numbers**: Higher generation wins
5. **External Arbitrator**: Third-party decides

## Distributed Computing

### Q13: Explain MapReduce paradigm.
**Answer:**
MapReduce processes large datasets in parallel across distributed nodes.

**Phases:**
1. **Map**: Transform input to key-value pairs
2. **Shuffle**: Group by key
3. **Reduce**: Aggregate values for each key

**Example - Word Count:**
```java
// Mapper
public class WordCountMapper extends Mapper<Long, Text, Text, IntWritable> {
    public void map(Long key, Text value, Context context) {
        String[] words = value.toString().split("\\s+");
        for (String word : words) {
            context.write(new Text(word), new IntWritable(1));
        }
    }
}

// Reducer
public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    public void reduce(Text key, Iterable<IntWritable> values, Context context) {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}
```

### Q14: What is Apache Spark and how does it differ from MapReduce?
**Answer:**
**Differences:**
| MapReduce | Spark |
|-----------|-------|
| Disk-based | In-memory processing |
| Batch only | Batch and streaming |
| Two-stage | DAG execution |
| Slower | 100x faster for in-memory |
| Java-centric | Multiple languages |

**Spark Components:**
- **RDD**: Resilient Distributed Dataset
- **DataFrame**: Structured data
- **Spark SQL**: SQL queries
- **Spark Streaming**: Real-time processing
- **MLlib**: Machine learning
- **GraphX**: Graph processing

### Q15: Explain Actor Model for distributed computing.
**Answer:**
Actor Model treats "actors" as fundamental units of computation.

**Principles:**
- Actors communicate only through messages
- Each actor has private state
- Actors can create other actors
- No shared memory

**Example with Akka:**
```java
public class CounterActor extends AbstractActor {
    private int count = 0;
    
    @Override
    public Receive createReceive() {
        return receiveBuilder()
            .match(Increment.class, msg -> {
                count++;
                sender().tell(count, self());
            })
            .match(GetCount.class, msg -> {
                sender().tell(count, self());
            })
            .build();
    }
}

// Usage
ActorSystem system = ActorSystem.create("MySystem");
ActorRef counter = system.actorOf(Props.create(CounterActor.class));
counter.tell(new Increment(), ActorRef.noSender());
```

## Network & Communication

### Q16: Explain RPC vs REST vs GraphQL.
**Answer:**
**RPC (Remote Procedure Call):**
- Call remote functions like local
- Binary protocols (gRPC, Thrift)
- Strongly typed
- Efficient

**REST:**
- Resource-based
- HTTP methods (GET, POST, PUT, DELETE)
- Stateless
- JSON/XML

**GraphQL:**
- Query language
- Single endpoint
- Client specifies data needs
- Solves over/under-fetching

### Q17: How does gRPC work?
**Answer:**
**Features:**
- HTTP/2 based
- Protocol Buffers
- Bidirectional streaming
- Multiple language support

**Communication Types:**
1. **Unary**: Request-response
2. **Server Streaming**: One request, stream response
3. **Client Streaming**: Stream request, one response
4. **Bidirectional Streaming**: Both stream

**Example:**
```protobuf
service UserService {
    rpc GetUser(UserRequest) returns (User);
    rpc ListUsers(Empty) returns (stream User);
    rpc CreateUsers(stream User) returns (Summary);
    rpc Chat(stream Message) returns (stream Message);
}
```

### Q18: What is Service Mesh?
**Answer:**
Service Mesh provides infrastructure layer for service-to-service communication.

**Components:**
- **Data Plane**: Proxies (Envoy)
- **Control Plane**: Management (Istio, Linkerd)

**Features:**
- Traffic management
- Security (mTLS)
- Observability
- Resilience

**Architecture:**
```
Service A → [Sidecar Proxy] → [Sidecar Proxy] → Service B
                ↑                      ↑
            Control Plane          Control Plane
```

## Distributed Storage

### Q19: Explain Consistent Hashing.
**Answer:**
Consistent hashing minimizes remapping when nodes are added/removed.

**How it works:**
1. Hash nodes and keys to circle (0 to 2^32)
2. Key belongs to first node clockwise
3. Virtual nodes for load distribution

**Benefits:**
- Add/remove node affects only neighbors
- Load balancing with virtual nodes
- Used in Cassandra, DynamoDB

**Implementation:**
```java
public class ConsistentHash<T> {
    private final SortedMap<Integer, T> circle = new TreeMap<>();
    private final int numberOfReplicas;
    private final HashFunction hashFunction;
    
    public void add(T node) {
        for (int i = 0; i < numberOfReplicas; i++) {
            int hash = hashFunction.hash(node.toString() + i);
            circle.put(hash, node);
        }
    }
    
    public T get(Object key) {
        if (circle.isEmpty()) return null;
        
        int hash = hashFunction.hash(key);
        SortedMap<Integer, T> tailMap = circle.tailMap(hash);
        int nodeHash = tailMap.isEmpty() ? 
            circle.firstKey() : tailMap.firstKey();
        return circle.get(nodeHash);
    }
}
```

### Q20: How does distributed file system work (HDFS)?
**Answer:**
**HDFS Architecture:**
- **NameNode**: Metadata, file system namespace
- **DataNode**: Store actual data blocks
- **Block Size**: 128MB default
- **Replication**: Default 3 copies

**Read Process:**
1. Client requests NameNode for block locations
2. NameNode returns DataNode addresses
3. Client reads from closest DataNode

**Write Process:**
1. Client requests NameNode to create file
2. NameNode returns DataNode pipeline
3. Client writes to first DataNode
4. DataNode replicates to others

### Q21: Explain Vector Clocks.
**Answer:**
Vector clocks determine causal ordering of events in distributed systems.

**How it works:**
- Each node maintains vector of logical clocks
- Increment own clock on event
- Include vector with messages
- Merge vectors on receive

**Example:**
```java
public class VectorClock {
    private Map<String, Integer> clock = new HashMap<>();
    
    public void increment(String nodeId) {
        clock.put(nodeId, clock.getOrDefault(nodeId, 0) + 1);
    }
    
    public void update(VectorClock other) {
        for (Map.Entry<String, Integer> entry : other.clock.entrySet()) {
            clock.put(entry.getKey(), 
                Math.max(clock.getOrDefault(entry.getKey(), 0), 
                        entry.getValue()));
        }
    }
    
    public boolean happensBefore(VectorClock other) {
        for (String node : clock.keySet()) {
            if (clock.get(node) > other.clock.getOrDefault(node, 0)) {
                return false;
            }
        }
        return !clock.equals(other.clock);
    }
}
```

## Clock & Ordering

### Q22: What is Lamport Clock?
**Answer:**
Lamport Clock provides partial ordering of events in distributed systems.

**Rules:**
1. Increment clock before any event
2. Include timestamp with messages
3. On receive: clock = max(local_clock, message_clock) + 1

**Limitation:** Can't determine concurrent events

### Q23: Explain the problem of distributed transactions.
**Answer:**
**Challenges:**
- Network failures
- Node failures
- Maintaining ACID properties

**Solutions:**
1. **Two-Phase Commit (2PC)**:
   - Prepare phase
   - Commit phase
   - Blocking protocol

2. **Three-Phase Commit (3PC)**:
   - Add pre-commit phase
   - Non-blocking
   - More complex

3. **Saga Pattern**:
   - Series of local transactions
   - Compensating transactions

## Architecture Principles

### Q24: What are the principles of good software architecture?
**Answer:**
1. **Separation of Concerns**: Different aspects in different modules
2. **Single Responsibility**: One reason to change
3. **Principle of Least Knowledge**: Minimize dependencies
4. **DRY** (Don't Repeat Yourself): Avoid duplication
5. **YAGNI** (You Aren't Gonna Need It): Don't over-engineer
6. **KISS** (Keep It Simple, Stupid): Simplicity is key
7. **Composition over Inheritance**: Prefer composition
8. **Dependency Inversion**: Depend on abstractions

### Q25: Explain the concept of Bounded Context in DDD.
**Answer:**
Bounded Context defines clear boundaries where a domain model applies.

**Characteristics:**
- Own ubiquitous language
- Clear interface with other contexts
- Can have different models for same concept

**Context Mapping Patterns:**
- **Shared Kernel**: Shared model subset
- **Customer/Supplier**: Upstream/downstream relationship
- **Conformist**: Downstream conforms to upstream
- **Anti-corruption Layer**: Translation layer
- **Open Host Service**: Published protocol
- **Published Language**: Well-documented format

### Q26: What is Event-Driven Architecture?
**Answer:**
**Components:**
- **Event Producers**: Generate events
- **Event Routers**: Route events (Event Bus, Broker)
- **Event Consumers**: Process events

**Patterns:**
1. **Event Notification**: Simple notification
2. **Event-Carried State Transfer**: Event contains state
3. **Event Sourcing**: Store events as source of truth
4. **CQRS**: Separate read/write models

**Benefits:**
- Loose coupling
- Scalability
- Flexibility
- Audit trail

**Implementation:**
```java
// Event
public class OrderCreatedEvent {
    private String orderId;
    private String customerId;
    private BigDecimal amount;
    private Instant timestamp;
}

// Producer
@Service
public class OrderService {
    @Autowired
    private EventPublisher publisher;
    
    public Order createOrder(OrderRequest request) {
        Order order = new Order(request);
        orderRepository.save(order);
        
        publisher.publish(new OrderCreatedEvent(order));
        return order;
    }
}

// Consumer
@EventListener
public class InventoryService {
    @Async
    public void handle(OrderCreatedEvent event) {
        // Reserve inventory
        reserveItems(event.getOrderId(), event.getItems());
    }
}
```

### Q27: Explain Reactive Architecture.
**Answer:**
**Reactive Manifesto Principles:**
1. **Responsive**: Timely response
2. **Resilient**: Responsive despite failures
3. **Elastic**: Responsive under varying load
4. **Message Driven**: Asynchronous message passing

**Implementation with Reactive Streams:**
```java
Flux<User> users = userRepository.findAll()
    .filter(user -> user.isActive())
    .map(user -> enrichUser(user))
    .timeout(Duration.ofSeconds(5))
    .retry(3)
    .onErrorResume(e -> Flux.empty());
```

## Performance & Optimization

### Q28: How to optimize distributed system performance?
**Answer:**
1. **Caching**: Multiple levels (CDN, application, database)
2. **Load Balancing**: Distribute load evenly
3. **Async Processing**: Don't block on I/O
4. **Batch Processing**: Reduce round trips
5. **Connection Pooling**: Reuse connections
6. **Data Locality**: Process data where it resides
7. **Compression**: Reduce network transfer
8. **Pagination**: Limit data transfer

### Q29: What is backpressure in distributed systems?
**Answer:**
Backpressure is a mechanism to handle overwhelming load by signaling upstream to slow down.

**Strategies:**
1. **Buffering**: Temporary storage
2. **Dropping**: Discard excess
3. **Throttling**: Rate limiting
4. **Blocking**: Stop accepting

**Implementation:**
```java
// Reactive Streams backpressure
Flux.range(1, 1000000)
    .onBackpressureBuffer(100, 
        dropped -> log.warn("Dropped: " + dropped))
    .publishOn(Schedulers.parallel())
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(10); // Request 10 items
        }
        
        @Override
        protected void hookOnNext(Integer value) {
            // Process value
            request(1); // Request next
        }
    });
```

### Q30: How to handle cascading failures?
**Answer:**
**Prevention:**
1. **Circuit Breakers**: Stop calling failing services
2. **Bulkheads**: Isolate resources
3. **Timeouts**: Don't wait forever
4. **Rate Limiting**: Prevent overload
5. **Load Shedding**: Drop low-priority requests
6. **Graceful Degradation**: Reduced functionality

**Detection:**
- Monitor error rates
- Track latency increases
- Watch for timeout spikes
- Alert on circuit breaker trips