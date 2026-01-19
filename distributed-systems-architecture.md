# Distributed Systems & Architecture Interview Questions

## Table of Contents
1. [Distributed Systems Fundamentals](#distributed-systems-fundamentals)
2. [Architecture Styles](#architecture-styles)
3. [Consistency Models](#consistency-models)
4. [Consensus Algorithms](#consensus-algorithms)
5. [Fault Tolerance & Reliability](#fault-tolerance--reliability)
6. [Distributed Computing](#distributed-computing)
7. [Network & Communication](#network--communication)
8. [Distributed Storage](#distributed-storage)
9. [Clock & Ordering](#clock--ordering)
10. [Distributed Transactions](#distributed-transactions)

---

## Distributed Systems Fundamentals

### Q1: What are the challenges of distributed systems?
**Answer:**

| Challenge | Description | Mitigation |
|-----------|-------------|------------|
| **Network Unreliability** | Messages lost, delayed, duplicated | Retries, acknowledgments, idempotency |
| **Partial Failures** | Some components fail while others work | Health checks, circuit breakers |
| **Lack of Global Clock** | No single source of truth for time | Logical clocks, NTP |
| **Concurrency** | Multiple operations simultaneously | Locks, CAS, conflict resolution |
| **Consistency** | Keeping data synchronized | Consensus protocols, replication |
| **Scalability** | Handling increased load | Horizontal scaling, partitioning |
| **Security** | Multiple attack surfaces | Encryption, authentication |
| **Debugging** | Hard to reproduce issues | Distributed tracing, logging |

### Q2: Explain the Eight Fallacies of Distributed Computing.
**Answer:**

The fallacies describe false assumptions developers make about networks:

1. **The network is reliable**
   - Reality: Networks fail, packets drop, cables get cut
   - Mitigation: Timeouts, retries, circuit breakers

2. **Latency is zero**
   - Reality: Network calls take 0.5ms (datacenter) to 150ms (cross-continent)
   - Mitigation: Caching, async operations, batching

3. **Bandwidth is infinite**
   - Reality: Limited bandwidth, especially at scale
   - Mitigation: Compression, pagination, CDNs

4. **The network is secure**
   - Reality: Man-in-the-middle, eavesdropping
   - Mitigation: TLS, authentication, authorization

5. **Topology doesn't change**
   - Reality: Servers added/removed, network restructured
   - Mitigation: Service discovery, load balancers

6. **There is one administrator**
   - Reality: Multiple teams, vendors, cloud providers
   - Mitigation: Clear interfaces, contracts

7. **Transport cost is zero**
   - Reality: Serialization, deserialization, network costs
   - Mitigation: Efficient protocols (gRPC), batching

8. **The network is homogeneous**
   - Reality: Different protocols, versions, platforms
   - Mitigation: Standard protocols, API versioning

### Q3: What is the difference between parallel and distributed computing?
**Answer:**

| Aspect | Parallel Computing | Distributed Computing |
|--------|-------------------|----------------------|
| **Memory** | Shared memory | Distributed memory |
| **Coupling** | Tightly coupled | Loosely coupled |
| **Communication** | Low latency (ns) | High latency (ms) |
| **Systems** | Homogeneous | Heterogeneous |
| **Goal** | Performance | Scalability, availability |
| **Failures** | All or nothing | Partial failures |
| **Clock** | Shared clock | No shared clock |
| **Examples** | Multi-core CPU, GPU | Microservices, cloud |

```
Parallel Computing:
┌────────────────────────────┐
│        Shared Memory       │
│  ┌────┐ ┌────┐ ┌────┐     │
│  │CPU1│ │CPU2│ │CPU3│     │
│  └────┘ └────┘ └────┘     │
└────────────────────────────┘

Distributed Computing:
┌──────┐      ┌──────┐      ┌──────┐
│Node 1│◀────▶│Node 2│◀────▶│Node 3│
│Memory│      │Memory│      │Memory│
└──────┘      └──────┘      └──────┘
     Network Communication
```

---

## Architecture Styles

### Q4: Compare different architecture styles.
**Answer:**

| Style | Description | Pros | Cons | Use Case |
|-------|-------------|------|------|----------|
| **Monolithic** | Single deployable unit | Simple, easy to test | Hard to scale, tight coupling | Small teams, MVPs |
| **SOA** | Services via ESB | Reusable services | Heavy middleware | Enterprise integration |
| **Microservices** | Small independent services | Scalable, flexible | Complex operations | Large-scale apps |
| **Serverless** | Functions as units | No ops, pay-per-use | Cold starts, vendor lock-in | Event processing |
| **Event-Driven** | Events as communication | Loose coupling | Eventual consistency | Real-time systems |

### Q5: What is Hexagonal Architecture (Ports and Adapters)?
**Answer:**

Hexagonal architecture isolates core business logic from external concerns.

```
                    ┌─────────────────────────────────┐
                    │         External World          │
                    │  [REST API]  [CLI]  [Message]   │
                    └─────────────┬───────────────────┘
                                  │
                    ┌─────────────▼───────────────────┐
                    │        Driving Adapters         │
                    │  (HTTP Controller, Consumer)    │
                    └─────────────┬───────────────────┘
                                  │
          ┌───────────────────────▼───────────────────────┐
          │                   Ports                        │
          │    ┌─────────────────────────────────────┐    │
          │    │         Application Core            │    │
          │    │    ┌─────────────────────────┐     │    │
          │    │    │   Domain (Entities,      │     │    │
          │    │    │   Business Rules)        │     │    │
          │    │    └─────────────────────────┘     │    │
          │    │    Use Cases / Application Services│    │
          │    └─────────────────────────────────────┘    │
          └───────────────────────┬───────────────────────┘
                                  │
                    ┌─────────────▼───────────────────┐
                    │        Driven Adapters          │
                    │   (Database, External APIs)     │
                    └─────────────┬───────────────────┘
                                  │
                    ┌─────────────▼───────────────────┐
                    │      External Systems           │
                    │  [PostgreSQL]  [Redis]  [S3]    │
                    └─────────────────────────────────┘
```

```java
// Port (Interface)
public interface OrderRepository {
    Order findById(String id);
    void save(Order order);
}

// Driven Adapter (Database implementation)
@Repository
public class PostgresOrderRepository implements OrderRepository {
    private final JdbcTemplate jdbc;
    
    @Override
    public Order findById(String id) {
        return jdbc.queryForObject("SELECT * FROM orders WHERE id = ?", 
            Order.class, id);
    }
}

// Application Core (Use Case)
public class CreateOrderUseCase {
    private final OrderRepository repository;  // Uses port, not adapter
    
    public Order execute(CreateOrderCommand command) {
        Order order = Order.create(command);
        repository.save(order);
        return order;
    }
}

// Driving Adapter (REST Controller)
@RestController
public class OrderController {
    private final CreateOrderUseCase createOrder;
    
    @PostMapping("/orders")
    public ResponseEntity<OrderResponse> create(@RequestBody OrderRequest request) {
        Order order = createOrder.execute(request.toCommand());
        return ResponseEntity.created(uri).body(OrderResponse.from(order));
    }
}
```

### Q6: Explain Clean Architecture principles.
**Answer:**

**The Dependency Rule**: Dependencies must point inward only. Inner layers know nothing about outer layers.

```
┌─────────────────────────────────────────────────────────────┐
│  FRAMEWORKS & DRIVERS (outermost)                           │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  INTERFACE ADAPTERS                                   │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │  APPLICATION BUSINESS RULES (Use Cases)         │  │  │
│  │  │  ┌───────────────────────────────────────────┐  │  │  │
│  │  │  │  ENTERPRISE BUSINESS RULES (Entities)     │  │  │  │
│  │  │  │           Core Domain                      │  │  │  │
│  │  │  └───────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Layer Responsibilities:**

| Layer | Responsibility | Examples |
|-------|---------------|----------|
| **Entities** | Business objects and rules | Order, Product, Price calculation |
| **Use Cases** | Application-specific rules | CreateOrder, CancelOrder |
| **Interface Adapters** | Convert data formats | Controllers, Presenters, Gateways |
| **Frameworks** | External tools | Spring, Hibernate, REST |

---

## Consistency Models

### Q7: Explain different consistency models in detail.
**Answer:**

```
Strongest ──────────────────────────────────────▶ Weakest

Linearizable → Sequential → Causal → Eventual
     │              │           │         │
     │              │           │         └─ All replicas converge eventually
     │              │           └─ Causally related ops ordered
     │              └─ Single total order visible to all
     └─ Real-time order preserved
```

**1. Linearizability (Strongest)**
```
Client A: Write(x=1) ───────────────────────────────▶
                              │ (instant visibility)
Client B:              Read(x) → returns 1
```
- All operations appear to happen atomically at some point
- Real-time guarantee: if op1 completes before op2 starts, op1 precedes op2

**2. Sequential Consistency**
```
Client A: Write(x=1) ─────▶ Write(x=2) ───────────▶
Client B:           Read(x)=1     Read(x)=2
```
- All clients see same order of operations
- Order respects program order, not necessarily real-time

**3. Causal Consistency**
```
Client A: Write(x=1) ─────▶ Write(y=2, depends on x=1) ───▶
Client B: Read(y)=2 implies Read(x)=1 (must see cause)
```
- Causally related operations are seen in order
- Concurrent operations may be seen in any order

**4. Eventual Consistency**
```
Client A: Write(x=1) ─────────────────────────────▶
                      │
                      └─ Propagation delay (could be seconds)
                              │
Client B:              Read(x)=0 ... Read(x)=1 (eventually)
```
- Given no new updates, all replicas eventually converge
- No ordering guarantees

### Q8: What is Read-Your-Writes consistency?
**Answer:**

Guarantees a user always sees their own writes, even if reading from different replicas.

```java
public class ReadYourWritesSession {
    private Map<String, Long> lastWriteTimestamps = new HashMap<>();
    
    public void recordWrite(String key, long timestamp) {
        lastWriteTimestamps.put(key, timestamp);
    }
    
    public String read(String key) {
        Long lastWrite = lastWriteTimestamps.get(key);
        
        if (lastWrite != null) {
            // Route to replica that has seen our write
            Replica replica = findReplicaWithTimestamp(key, lastWrite);
            return replica.read(key);
        }
        
        // No recent write, any replica is fine
        return anyReplica.read(key);
    }
}
```

**Implementation strategies:**
1. **Sticky sessions**: Route user to same replica
2. **Write-through cache**: User reads from cache containing their writes
3. **Timestamp tracking**: Only read from replicas that have caught up

---

## Consensus Algorithms

### Q9: How does the Raft consensus algorithm work?
**Answer:**

Raft is designed to be understandable. It decomposes consensus into three sub-problems:

**1. Leader Election**
```
             Election Timeout
Follower ──────────────────────▶ Candidate
    ▲                                │
    │    Heartbeat         RequestVote (majority)
    │        │                       │
    └────────┼───────────────────────▼
             │                    Leader
             └────────────────────────┘
                   Heartbeats
```

**2. Log Replication**
```
Leader:   [Entry1] [Entry2] [Entry3]
              │        │        │
              ▼        ▼        ▼
Follower1: [Entry1] [Entry2] [Entry3]  ✓ Committed
Follower2: [Entry1] [Entry2]           Catching up
Follower3: [Entry1] [Entry2] [Entry3]  ✓ Committed
```

**3. Safety**
- Only leader with complete log can win election
- Committed entries survive leader crashes

```java
public class RaftNode {
    enum State { FOLLOWER, CANDIDATE, LEADER }
    
    private State state = State.FOLLOWER;
    private int currentTerm = 0;
    private String votedFor = null;
    private List<LogEntry> log = new ArrayList<>();
    private int commitIndex = 0;
    
    // Follower timeout triggers election
    public void onElectionTimeout() {
        becomeCandidate();
    }
    
    private void becomeCandidate() {
        state = State.CANDIDATE;
        currentTerm++;
        votedFor = myId;
        
        int votes = 1; // Vote for self
        
        RequestVoteArgs args = new RequestVoteArgs(
            currentTerm, myId, 
            getLastLogIndex(), getLastLogTerm());
        
        for (Node peer : peers) {
            CompletableFuture.supplyAsync(() -> peer.requestVote(args))
                .thenAccept(reply -> {
                    if (reply.voteGranted) {
                        synchronized(this) {
                            votes++;
                            if (votes > (peers.size() + 1) / 2) {
                                becomeLeader();
                            }
                        }
                    }
                });
        }
    }
    
    private void becomeLeader() {
        state = State.LEADER;
        // Initialize leader state
        for (Node peer : peers) {
            nextIndex.put(peer, log.size());
            matchIndex.put(peer, 0);
        }
        // Start sending heartbeats
        sendHeartbeats();
    }
    
    // Log replication
    public boolean appendEntries(AppendEntriesArgs args) {
        if (args.term < currentTerm) {
            return false; // Reject old leader
        }
        
        resetElectionTimeout();
        
        // Check log consistency
        if (args.prevLogIndex > 0 && 
            (log.size() < args.prevLogIndex || 
             log.get(args.prevLogIndex - 1).term != args.prevLogTerm)) {
            return false; // Log inconsistent
        }
        
        // Append new entries
        log.addAll(args.entries);
        
        // Update commit index
        if (args.leaderCommit > commitIndex) {
            commitIndex = Math.min(args.leaderCommit, log.size());
        }
        
        return true;
    }
}
```

### Q10: How does Paxos work?
**Answer:**

Paxos achieves consensus in unreliable networks. It has three roles:

**Roles:**
- **Proposer**: Proposes values
- **Acceptor**: Votes on proposals
- **Learner**: Learns the chosen value

**Two Phases:**

```
Phase 1 (Prepare):
Proposer                  Acceptors
    │                         │
    │ ── Prepare(n) ────────▶ │  "I want to propose with number n"
    │                         │
    │ ◀── Promise(n, v?) ──── │  "I promise not to accept < n,
    │                         │   here's highest I've accepted"

Phase 2 (Accept):
    │                         │
    │ ── Accept(n, v) ───────▶│  "Please accept value v with number n"
    │                         │
    │ ◀── Accepted(n, v) ───  │  "I've accepted"
    │                         │
If majority accepted, value is chosen!
```

**Key Properties:**
- **Safety**: Only one value can be chosen
- **Liveness**: Eventually makes progress (with reasonable assumptions)

### Q11: What is the Byzantine Generals Problem?
**Answer:**

The problem asks: How can distributed nodes agree on a value when some nodes may be faulty or malicious?

```
General 1 (Loyal):     "Attack at dawn"
General 2 (Traitor):   Tells 1 "Attack", tells 3 "Retreat"
General 3 (Loyal):     "What should we do?"
```

**Requirements:**
- All loyal generals must agree
- Need **3f + 1** nodes to tolerate **f** Byzantine failures

**Solutions:**

**1. PBFT (Practical Byzantine Fault Tolerance)**
```
Client → Primary → Pre-prepare → Prepare → Commit → Reply to Client
                      │             │          │
                      └─────────────┴──────────┘
                        Requires 2f+1 agreements
```

**2. Blockchain Consensus**
- Proof of Work (Bitcoin)
- Proof of Stake (Ethereum 2.0)

---

## Fault Tolerance & Reliability

### Q12: How to design for failure in distributed systems?
**Answer:**

**Principles:**

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| **Assume Failure** | Everything will fail eventually | Design for it |
| **Fail Fast** | Detect and handle failures quickly | Timeouts, health checks |
| **Graceful Degradation** | Reduced functionality > no functionality | Feature flags, fallbacks |
| **Redundancy** | No single point of failure | Replication |
| **Isolation** | Failure in one area doesn't spread | Bulkheads |

```java
@Component
public class ResilientService {
    
    // Circuit Breaker: Stop calling failing service
    @CircuitBreaker(name = "inventory", fallbackMethod = "inventoryFallback")
    public InventoryResponse checkInventory(String productId) {
        return inventoryClient.check(productId);
    }
    
    // Fallback when circuit is open
    public InventoryResponse inventoryFallback(String productId, Exception e) {
        log.warn("Inventory service unavailable, returning cached data");
        return cachedInventory.get(productId);
    }
    
    // Retry with exponential backoff
    @Retry(name = "payment", maxAttempts = 3, backoff = @Backoff(delay = 1000, multiplier = 2))
    public PaymentResult processPayment(Payment payment) {
        return paymentGateway.process(payment);
    }
    
    // Bulkhead: Isolate resources
    @Bulkhead(name = "reporting", type = Bulkhead.Type.THREADPOOL)
    public Report generateReport(ReportRequest request) {
        return reportingService.generate(request);  // Won't starve other operations
    }
    
    // Timeout: Don't wait forever
    @TimeLimiter(name = "search", cancelRunningFuture = true)
    public CompletableFuture<SearchResults> search(String query) {
        return CompletableFuture.supplyAsync(() -> searchService.search(query));
    }
}

// Configuration
resilience4j:
  circuitbreaker:
    instances:
      inventory:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
  retry:
    instances:
      payment:
        maxAttempts: 3
        waitDuration: 1s
  bulkhead:
    instances:
      reporting:
        maxConcurrentCalls: 10
        maxWaitDuration: 100ms
```

### Q13: What is the Split-Brain problem?
**Answer:**

Split-brain occurs when a network partition causes nodes to operate independently, each believing it's the leader.

```
Before partition:
[Node A] ◀────▶ [Node B] ◀────▶ [Node C]
  Leader          Follower        Follower

After partition:
[Node A]    ✗    [Node B] ◀────▶ [Node C]
  Leader         New Leader!      Follower
                 (believes A is dead)

Both accept writes → Data divergence!
```

**Solutions:**

**1. Quorum-based (majority required)**
```java
public boolean canAcceptWrites() {
    int reachableNodes = countReachableNodes();
    int totalNodes = clusterSize;
    
    // Only accept writes if we have majority
    return reachableNodes > totalNodes / 2;
}
```

**2. Fencing with Generation Numbers**
```java
public class FencedLeader {
    private int generation;
    private String leaderId;
    
    public boolean isValidLeader(String id, int gen) {
        if (gen > this.generation) {
            // New leader with higher generation wins
            this.generation = gen;
            this.leaderId = id;
            return true;
        }
        return id.equals(this.leaderId);
    }
}
```

**3. External Arbitrator (Witness)**
```
[Node A]────▶[Witness]◀────[Node B]
             (ZooKeeper)
             
Only node that can reach witness becomes leader
```

### Q14: What is Gossip Protocol?
**Answer:**

Gossip protocol spreads information through random peer-to-peer communication, like how rumors spread.

```
Initial state:
Node A: knows [update1]
Node B: knows []
Node C: knows []
Node D: knows []

Round 1: A gossips to B
Node A: knows [update1]
Node B: knows [update1]  ← received
Node C: knows []
Node D: knows []

Round 2: A gossips to D, B gossips to C
Node A: knows [update1]
Node B: knows [update1]
Node C: knows [update1]  ← received from B
Node D: knows [update1]  ← received from A

O(log N) rounds to reach all nodes!
```

```java
public class GossipProtocol {
    private final Map<String, NodeState> nodeStates = new ConcurrentHashMap<>();
    private final int gossipFanout = 3;  // Number of peers per round
    
    @Scheduled(fixedRate = 1000)  // Gossip every second
    public void gossip() {
        List<Node> peers = selectRandomPeers(gossipFanout);
        GossipMessage message = buildGossipMessage();
        
        for (Node peer : peers) {
            try {
                GossipMessage reply = peer.exchange(message);
                mergeState(reply);
            } catch (Exception e) {
                markPotentiallyFailed(peer);
            }
        }
    }
    
    private List<Node> selectRandomPeers(int count) {
        List<Node> allPeers = new ArrayList<>(knownPeers);
        Collections.shuffle(allPeers);
        return allPeers.subList(0, Math.min(count, allPeers.size()));
    }
    
    private void mergeState(GossipMessage message) {
        for (Map.Entry<String, NodeState> entry : message.getStates().entrySet()) {
            nodeStates.merge(entry.getKey(), entry.getValue(), 
                (old, new) -> new.timestamp > old.timestamp ? new : old);
        }
    }
}
```

**Use Cases:**
- **Cassandra**: Cluster membership, failure detection
- **Consul**: Service discovery
- **Amazon S3**: Replication state

---

## Distributed Computing

### Q15: Explain MapReduce paradigm.
**Answer:**

MapReduce processes large datasets in parallel across distributed nodes.

```
Input Data
    │
    ├──▶ Mapper 1 ──▶ (key1, value1)
    │                      │
    ├──▶ Mapper 2 ──▶ (key2, value2) ──▶ Shuffle & Sort ──▶ Reducer 1
    │                      │                                    │
    └──▶ Mapper 3 ──▶ (key1, value3)                       Reducer 2
                                                               │
                                                           Output
```

**Example: Word Count**
```java
// Mapper: emit (word, 1) for each word
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
    
    public void map(LongWritable key, Text value, Context context) 
            throws IOException, InterruptedException {
        StringTokenizer tokenizer = new StringTokenizer(value.toString());
        while (tokenizer.hasMoreTokens()) {
            word.set(tokenizer.nextToken());
            context.write(word, one);  // Emit (word, 1)
        }
    }
}

// Reducer: sum counts for each word
public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    
    public void reduce(Text key, Iterable<IntWritable> values, Context context)
            throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}
```

### Q16: What is Apache Spark and how does it differ from MapReduce?
**Answer:**

| Feature | MapReduce | Spark |
|---------|-----------|-------|
| **Processing** | Disk-based (write intermediate to disk) | In-memory (RDD cached in RAM) |
| **Speed** | Slower (disk I/O) | 10-100x faster |
| **Model** | Map → Shuffle → Reduce | DAG of transformations |
| **Processing** | Batch only | Batch + Streaming |
| **Languages** | Java | Java, Scala, Python, R |
| **Fault Tolerance** | Re-execute failed tasks | Lineage-based recovery |

```scala
// Spark word count (much simpler!)
val textFile = spark.read.textFile("hdfs://...")
val counts = textFile
    .flatMap(line => line.split(" "))
    .map(word => (word, 1))
    .reduceByKey(_ + _)
counts.saveAsTextFile("hdfs://output")

// Spark Streaming example
val lines = spark.readStream
    .format("kafka")
    .option("kafka.bootstrap.servers", "host:9092")
    .option("subscribe", "topic")
    .load()

val wordCounts = lines
    .selectExpr("CAST(value AS STRING)")
    .flatMap(_.split(" "))
    .groupBy("value")
    .count()

wordCounts.writeStream
    .outputMode("complete")
    .format("console")
    .start()
```

### Q17: Explain Actor Model for distributed computing.
**Answer:**

The Actor Model treats "actors" as the fundamental unit of computation.

**Principles:**
- Actors communicate **only** through asynchronous messages
- Each actor has **private state** (no shared memory)
- Actors can **create** other actors
- Processing is **single-threaded** within an actor

```java
// Akka Actor example
public class CounterActor extends AbstractActor {
    private int count = 0;
    
    // Define message types
    static class Increment {}
    static class Decrement {}
    static class GetCount {}
    static class CountResult {
        final int count;
        CountResult(int count) { this.count = count; }
    }
    
    @Override
    public Receive createReceive() {
        return receiveBuilder()
            .match(Increment.class, msg -> {
                count++;
            })
            .match(Decrement.class, msg -> {
                count--;
            })
            .match(GetCount.class, msg -> {
                sender().tell(new CountResult(count), self());
            })
            .build();
    }
}

// Usage
ActorSystem system = ActorSystem.create("counter-system");
ActorRef counter = system.actorOf(Props.create(CounterActor.class), "counter");

// Send messages (fire-and-forget)
counter.tell(new Increment(), ActorRef.noSender());
counter.tell(new Increment(), ActorRef.noSender());

// Ask pattern (request-response)
CompletionStage<Object> future = Patterns.ask(counter, new GetCount(), Duration.ofSeconds(5));
future.thenAccept(result -> {
    CountResult count = (CountResult) result;
    System.out.println("Count: " + count.count);  // Count: 2
});
```

---

## Network & Communication

### Q18: Explain RPC vs REST vs GraphQL.
**Answer:**

| Aspect | RPC (gRPC) | REST | GraphQL |
|--------|------------|------|---------|
| **Paradigm** | Action-oriented | Resource-oriented | Query-oriented |
| **Protocol** | HTTP/2, custom | HTTP 1.1/2 | HTTP |
| **Format** | Protocol Buffers | JSON/XML | JSON |
| **Contract** | .proto files | OpenAPI/Swagger | Schema |
| **Typing** | Strongly typed | Loosely typed | Strongly typed |
| **Endpoints** | Many procedures | Many resources | Single endpoint |
| **Fetching** | Fixed response | Fixed response | Client specifies |

```protobuf
// gRPC - Define service in .proto
service UserService {
    rpc GetUser (GetUserRequest) returns (User);
    rpc ListUsers (Empty) returns (stream User);  // Server streaming
    rpc CreateUsers (stream User) returns (Summary);  // Client streaming
    rpc Chat (stream Message) returns (stream Message);  // Bidirectional
}

message User {
    string id = 1;
    string name = 2;
    string email = 3;
}
```

```graphql
# GraphQL - Client specifies what data it needs
query {
    user(id: "123") {
        name
        email
        orders(last: 5) {
            id
            total
        }
    }
}
```

### Q19: What is Service Mesh?
**Answer:**

Service Mesh provides infrastructure for service-to-service communication with features like traffic management, security, and observability.

```
┌─────────────────────────────────────────────────────────────────┐
│                         Control Plane                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Config     │  │   Service    │  │   Policy     │          │
│  │   Server     │  │   Discovery  │  │   Engine     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│            │                │                │                   │
│            └────────────────┴────────────────┘                   │
│                            │                                     │
└────────────────────────────┼─────────────────────────────────────┘
                             │ Configuration
                             ▼
┌──────────────────────────────────────────────────────────────────┐
│                         Data Plane                                │
│                                                                   │
│  ┌──────────────────┐          ┌──────────────────┐             │
│  │   Service A      │          │   Service B      │             │
│  │  ┌───────────┐   │          │  ┌───────────┐   │             │
│  │  │   App     │   │          │  │   App     │   │             │
│  │  └─────┬─────┘   │          │  └─────┬─────┘   │             │
│  │        │         │          │        │         │             │
│  │  ┌─────▼─────┐   │          │  ┌─────▼─────┐   │             │
│  │  │  Sidecar  │◀──┼──────────┼──▶  Sidecar  │   │             │
│  │  │  (Envoy)  │   │  mTLS    │  │  (Envoy)  │   │             │
│  │  └───────────┘   │          │  └───────────┘   │             │
│  └──────────────────┘          └──────────────────┘             │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

**Features:**
- **Traffic Management**: Load balancing, retries, circuit breaking
- **Security**: mTLS, authorization policies
- **Observability**: Distributed tracing, metrics, logs
- **Examples**: Istio, Linkerd, Consul Connect

---

## Distributed Storage

### Q20: How does Consistent Hashing work?
**Answer:**

```
Hash Ring (0 to 2^32-1):

                    0
                    │
         ┌──────────┴──────────┐
        /                       \
    Node A                    Node C
       │                         │
       │    ┌────────────┐      │
       │    │            │      │
       ├────│    Keys    │──────┤
       │    │            │      │
       │    └────────────┘      │
       │                         │
    Node B                    Node D
        \                       /
         └──────────┬──────────┘
                    │
                   2^32

Key "user:123" hashes to position between Node B and Node C
→ Belongs to Node C (first node clockwise)
```

**Virtual Nodes:**
```
Physical Node A → Virtual nodes: A0, A1, A2, A3
Physical Node B → Virtual nodes: B0, B1, B2, B3

Ring distribution:
... A0 ... B1 ... A2 ... B0 ... A1 ... B2 ... A3 ... B3 ...

Benefits:
- More even key distribution
- Gradual load transfer when node added/removed
```

### Q21: What are CRDTs (Conflict-free Replicated Data Types)?
**Answer:**

CRDTs automatically resolve conflicts without coordination, enabling eventual consistency.

**Types:**
1. **G-Counter** (Grow-only counter)
2. **PN-Counter** (Positive-Negative counter)
3. **G-Set** (Grow-only set)
4. **OR-Set** (Observed-Remove set)
5. **LWW-Register** (Last-Writer-Wins register)

```java
// G-Counter: Each node tracks its own increments
public class GCounter {
    private final String nodeId;
    private final Map<String, Long> counters = new ConcurrentHashMap<>();
    
    public GCounter(String nodeId) {
        this.nodeId = nodeId;
    }
    
    public void increment() {
        counters.merge(nodeId, 1L, Long::sum);
    }
    
    public long value() {
        return counters.values().stream().mapToLong(Long::longValue).sum();
    }
    
    // Merge with another replica - take max for each node
    public void merge(GCounter other) {
        for (Map.Entry<String, Long> entry : other.counters.entrySet()) {
            counters.merge(entry.getKey(), entry.getValue(), Math::max);
        }
    }
}

// OR-Set: Add with unique tags, remove by tag
public class ORSet<E> {
    private final Map<E, Set<UUID>> elements = new ConcurrentHashMap<>();
    private final Set<UUID> tombstones = ConcurrentHashMap.newKeySet();
    
    public void add(E element) {
        elements.computeIfAbsent(element, k -> ConcurrentHashMap.newKeySet())
                .add(UUID.randomUUID());  // Unique tag
    }
    
    public void remove(E element) {
        Set<UUID> tags = elements.get(element);
        if (tags != null) {
            tombstones.addAll(tags);  // Mark all current tags as removed
        }
    }
    
    public Set<E> value() {
        Set<E> result = new HashSet<>();
        for (Map.Entry<E, Set<UUID>> entry : elements.entrySet()) {
            Set<UUID> activeTags = new HashSet<>(entry.getValue());
            activeTags.removeAll(tombstones);
            if (!activeTags.isEmpty()) {
                result.add(entry.getKey());
            }
        }
        return result;
    }
}
```

---

## Clock & Ordering

### Q22: What is Lamport Clock?
**Answer:**

Lamport Clock provides **partial ordering** of events in distributed systems.

**Rules:**
1. Increment clock before each event
2. Include timestamp with messages
3. On receive: `clock = max(local_clock, msg_clock) + 1`

```java
public class LamportClock {
    private long counter = 0;
    
    // Local event
    public synchronized long tick() {
        return ++counter;
    }
    
    // Send message
    public synchronized long send() {
        return ++counter;
    }
    
    // Receive message
    public synchronized long receive(long messageTimestamp) {
        counter = Math.max(counter, messageTimestamp) + 1;
        return counter;
    }
}
```

**Limitation:** Cannot determine if events are concurrent.

### Q23: Explain Vector Clocks.
**Answer:**

Vector clocks track causality and detect concurrent events.

```
Node A: [A:1, B:0, C:0]  → [A:2, B:0, C:0]
                              │
                              │ send msg to B
                              ▼
Node B: [A:0, B:1, C:0]  → [A:2, B:2, C:0]  (merge: max each component + 1)
                              │
                              │ send msg to C
                              ▼
Node C: [A:0, B:0, C:1]  → [A:2, B:2, C:2]
```

```java
public class VectorClock {
    private final String nodeId;
    private final Map<String, Integer> clock = new ConcurrentHashMap<>();
    
    public VectorClock(String nodeId) {
        this.nodeId = nodeId;
    }
    
    public Map<String, Integer> increment() {
        clock.merge(nodeId, 1, Integer::sum);
        return new HashMap<>(clock);
    }
    
    public void merge(Map<String, Integer> other) {
        for (Map.Entry<String, Integer> entry : other.entrySet()) {
            clock.merge(entry.getKey(), entry.getValue(), Math::max);
        }
        clock.merge(nodeId, 1, Integer::sum);  // Increment after merge
    }
    
    // Determine ordering
    public Ordering compare(Map<String, Integer> other) {
        boolean thisIsLess = false;
        boolean thisIsGreater = false;
        
        Set<String> allNodes = new HashSet<>();
        allNodes.addAll(clock.keySet());
        allNodes.addAll(other.keySet());
        
        for (String node : allNodes) {
            int thisValue = clock.getOrDefault(node, 0);
            int otherValue = other.getOrDefault(node, 0);
            
            if (thisValue < otherValue) thisIsLess = true;
            if (thisValue > otherValue) thisIsGreater = true;
        }
        
        if (thisIsLess && !thisIsGreater) return Ordering.BEFORE;
        if (thisIsGreater && !thisIsLess) return Ordering.AFTER;
        if (thisIsLess && thisIsGreater) return Ordering.CONCURRENT;
        return Ordering.EQUAL;
    }
    
    enum Ordering { BEFORE, AFTER, CONCURRENT, EQUAL }
}
```

---

## Distributed Transactions

### Q24: Explain Two-Phase Commit (2PC).
**Answer:**

2PC is a protocol to achieve atomicity across multiple nodes.

```
Coordinator                     Participants
     │                             │
     │ ──── Prepare ─────────────▶ │  Phase 1: Prepare
     │                             │
     │ ◀─── Vote Yes/No ────────── │
     │                             │
     │ ──── Commit/Abort ────────▶ │  Phase 2: Commit
     │                             │
     │ ◀─── Acknowledgment ─────── │
     │                             │
```

**Problems:**
1. **Blocking**: Participants wait if coordinator fails
2. **Single point of failure**: Coordinator
3. **Performance**: Multiple round trips

```java
public class TwoPhaseCommitCoordinator {
    private final List<Participant> participants;
    
    public boolean executeTransaction(Transaction tx) {
        // Phase 1: Prepare
        List<CompletableFuture<Boolean>> votes = participants.stream()
            .map(p -> CompletableFuture.supplyAsync(() -> p.prepare(tx)))
            .collect(Collectors.toList());
        
        boolean allVotedYes = votes.stream()
            .allMatch(f -> {
                try {
                    return f.get(30, TimeUnit.SECONDS);
                } catch (Exception e) {
                    return false;
                }
            });
        
        // Phase 2: Commit or Abort
        if (allVotedYes) {
            participants.forEach(p -> p.commit(tx));
            return true;
        } else {
            participants.forEach(p -> p.rollback(tx));
            return false;
        }
    }
}
```

### Q25: What is the Saga Pattern?
**Answer:**

Saga handles distributed transactions as a sequence of local transactions with compensating actions.

```
Forward transactions:          Compensating actions:
T1: Create Order         ──▶   C1: Cancel Order
T2: Reserve Inventory    ──▶   C2: Release Inventory
T3: Process Payment      ──▶   C3: Refund Payment
T4: Ship Order           ──▶   C4: Cancel Shipment

Success path:
T1 → T2 → T3 → T4 → Done!

Failure at T3:
T1 → T2 → T3(fails) → C2 → C1 → Rolled back!
```

**Orchestration (Centralized):**
```java
public class OrderSaga {
    public void execute(OrderRequest request) {
        try {
            Order order = orderService.create(request);
            inventoryService.reserve(order.getItems());
            paymentService.charge(order.getPayment());
            shippingService.ship(order);
        } catch (InventoryException e) {
            orderService.cancel(order);
            throw e;
        } catch (PaymentException e) {
            inventoryService.release(order.getItems());
            orderService.cancel(order);
            throw e;
        } catch (ShippingException e) {
            paymentService.refund(order.getPayment());
            inventoryService.release(order.getItems());
            orderService.cancel(order);
            throw e;
        }
    }
}
```

**Choreography (Event-driven):**
```java
// Order Service
@EventListener
public void onOrderCreated(OrderCreatedEvent event) {
    eventPublisher.publish(new OrderCreatedEvent(event.getOrder()));
}

// Inventory Service
@EventListener
public void onOrderCreated(OrderCreatedEvent event) {
    try {
        inventoryService.reserve(event.getItems());
        eventPublisher.publish(new InventoryReservedEvent(event.getOrderId()));
    } catch (Exception e) {
        eventPublisher.publish(new InventoryFailedEvent(event.getOrderId()));
    }
}

// Payment Service
@EventListener
public void onInventoryReserved(InventoryReservedEvent event) {
    try {
        paymentService.charge(event.getPayment());
        eventPublisher.publish(new PaymentCompletedEvent(event.getOrderId()));
    } catch (Exception e) {
        eventPublisher.publish(new PaymentFailedEvent(event.getOrderId()));
    }
}

// Compensation handler
@EventListener
public void onPaymentFailed(PaymentFailedEvent event) {
    inventoryService.release(event.getOrderId());
    orderService.cancel(event.getOrderId());
}
```

---

## Performance & Optimization

### Q26: What is backpressure in distributed systems?
**Answer:**

Backpressure is a mechanism to handle overwhelming load by signaling producers to slow down.

```
Without backpressure:
Producer ──[100/s]──▶ Consumer [can handle 50/s]
                           │
                           ▼
                      Buffer overflow → Data loss or OOM

With backpressure:
Producer ◀──[slow down signal]── Consumer
     │                               │
     │ ──[50/s]──────────────────▶  │
     │                               │
     └───────── Stable system ───────┘
```

**Strategies:**
1. **Buffering**: Temporary storage (risk: buffer overflow)
2. **Dropping**: Discard excess (risk: data loss)
3. **Throttling**: Rate limiting (risk: increased latency)
4. **Blocking**: Stop accepting (risk: deadlock)

```java
// Reactive Streams backpressure
Flux.range(1, 1000000)
    .onBackpressureBuffer(100)  // Buffer up to 100
    .publishOn(Schedulers.parallel())
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(10);  // Request only 10 items initially
        }
        
        @Override
        protected void hookOnNext(Integer value) {
            process(value);
            request(1);  // Request next when ready
        }
    });
```

### Q27: How to handle cascading failures?
**Answer:**

```
Without protection:
Service A → Service B → Service C
    │           │           │
    │           │           └─ C fails
    │           └─ B blocked waiting for C
    └─ A blocked waiting for B
All services fail!

With circuit breaker:
Service A → Service B → [Circuit Breaker] → Service C
    │           │               │               │
    │           │               │               └─ C fails
    │           │               └─ Opens circuit, returns fallback
    │           └─ Gets fallback, continues working
    └─ Works normally
System degrades gracefully!
```

**Protection layers:**

| Pattern | Purpose | Implementation |
|---------|---------|----------------|
| **Circuit Breaker** | Stop calling failing services | Resilience4j, Hystrix |
| **Timeout** | Don't wait forever | HTTP client timeout |
| **Bulkhead** | Isolate resources | Thread pools, semaphores |
| **Rate Limiting** | Prevent overload | Token bucket, sliding window |
| **Retry with Backoff** | Handle transient failures | Exponential backoff |
| **Load Shedding** | Drop low-priority requests | Priority queues |

---

## Interview Tips

### Q28: Common distributed systems interview questions
**Answer:**

1. **Design a distributed cache** (consistent hashing, replication)
2. **Design a distributed lock** (ZooKeeper, Redis)
3. **Design a leader election system** (Raft, Paxos)
4. **Handle data consistency in microservices** (Saga, eventual consistency)
5. **Design a rate limiter** (token bucket, distributed coordination)

### Q29: Key concepts to remember
**Answer:**

| Concept | Key Point |
|---------|-----------|
| CAP Theorem | Can only guarantee 2 of 3 (usually CP or AP) |
| Consistency | Strong vs eventual—understand trade-offs |
| Consensus | Raft for understandability, Paxos for theory |
| Failures | Design for failure, not against it |
| Time | No global clock—use logical clocks |
| State | Stateless services scale better |
| Communication | Async > sync for scalability |
