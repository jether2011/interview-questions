# Messaging & Event-Driven Architecture Interview Questions

## Table of Contents
1. [Messaging Fundamentals](#messaging-fundamentals)
2. [Message Queue Patterns](#message-queue-patterns)
3. [Apache Kafka](#apache-kafka)
4. [RabbitMQ](#rabbitmq)
5. [Event-Driven Architecture](#event-driven-architecture)
6. [Event Sourcing](#event-sourcing)
7. [Message Delivery Guarantees](#message-delivery-guarantees)
8. [Integration Patterns](#integration-patterns)
9. [Real-World Implementations](#real-world-implementations)

## Messaging Fundamentals

### Q1: What is the difference between Message Queue and Pub/Sub?
**Answer:**

**Message Queue (Point-to-Point):**
- One producer, one consumer
- Message consumed by single consumer
- Message deleted after consumption
- Work distribution pattern

```java
// Producer
queue.send("order-processing", orderMessage);

// Consumer
Message msg = queue.receive("order-processing");
processOrder(msg);
// Message is removed from queue
```

**Pub/Sub (Publish-Subscribe):**
- One publisher, multiple subscribers
- Message delivered to all subscribers
- Each subscriber gets own copy
- Broadcasting pattern

```java
// Publisher
topic.publish("order-events", orderCreatedEvent);

// Subscribers
subscription1.subscribe("order-events", msg -> updateInventory(msg));
subscription2.subscribe("order-events", msg -> sendEmail(msg));
subscription3.subscribe("order-events", msg -> updateAnalytics(msg));
```

### Q2: When to use synchronous vs asynchronous communication?
**Answer:**

**Synchronous:**
- Real-time response required
- Simple request-response
- Data consistency critical
- User-facing operations

```java
// Synchronous - REST call
@GetMapping("/user/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);  // Blocks until response
}
```

**Asynchronous:**
- Long-running operations
- Fire-and-forget scenarios
- Decoupling services
- Better fault tolerance

```java
// Asynchronous - Message queue
@PostMapping("/orders")
public ResponseEntity<String> createOrder(@RequestBody Order order) {
    String orderId = UUID.randomUUID().toString();
    messageQueue.send(new OrderCreatedEvent(orderId, order));
    return ResponseEntity.accepted()
        .body("Order " + orderId + " is being processed");
}
```

### Q3: Explain different messaging patterns.
**Answer:**

**1. Request-Reply:**
```java
// Send request with correlation ID
Message request = new Message("GetUserDetails");
request.setCorrelationId("123");
request.setReplyTo("reply-queue");
sender.send("user-service", request);

// Wait for reply
Message reply = receiver.receive("reply-queue", correlationId);
```

**2. Message Router:**
```java
public class MessageRouter {
    public void route(Message message) {
        switch(message.getType()) {
            case "ORDER":
                orderQueue.send(message);
                break;
            case "PAYMENT":
                paymentQueue.send(message);
                break;
            case "SHIPPING":
                shippingQueue.send(message);
                break;
        }
    }
}
```

**3. Message Filter:**
```java
public class MessageFilter {
    public void filter(Message message) {
        if (message.getPriority() > 5) {
            highPriorityQueue.send(message);
        } else if (isValid(message)) {
            normalQueue.send(message);
        }
        // Discard invalid or low priority
    }
}
```

**4. Aggregator:**
```java
public class OrderAggregator {
    private Map<String, List<Message>> messages = new ConcurrentHashMap<>();
    
    public void aggregate(Message message) {
        String correlationId = message.getCorrelationId();
        messages.computeIfAbsent(correlationId, k -> new ArrayList<>())
                .add(message);
        
        if (isComplete(correlationId)) {
            Order completeOrder = combineMessages(messages.get(correlationId));
            outputQueue.send(completeOrder);
            messages.remove(correlationId);
        }
    }
}
```

## Message Queue Patterns

### Q4: Explain competing consumers pattern.
**Answer:**
Multiple consumers process messages from same queue for load distribution.

```java
// RabbitMQ Example
@Component
public class OrderProcessor {
    
    @RabbitListener(queues = "orders", concurrency = "5-10")
    public void processOrder(Order order) {
        // Multiple instances of this method run concurrently
        // Each message processed by only one consumer
        orderService.process(order);
    }
}

// Kafka Example (Consumer Group)
@Component
public class KafkaOrderConsumer {
    
    @KafkaListener(
        topics = "orders",
        groupId = "order-processing-group",
        concurrency = "3"
    )
    public void consume(Order order) {
        // Partitions distributed among consumers in group
        processOrder(order);
    }
}
```

### Q5: How to implement priority queue?
**Answer:**

**RabbitMQ Priority Queue:**
```java
@Configuration
public class RabbitConfig {
    
    @Bean
    public Queue priorityQueue() {
        return QueueBuilder.durable("priority-queue")
            .maxPriority(10)  // 0-10 priority levels
            .build();
    }
}

// Producer
public void sendWithPriority(Message message, int priority) {
    rabbitTemplate.convertAndSend("priority-queue", message, msg -> {
        msg.getMessageProperties().setPriority(priority);
        return msg;
    });
}
```

**Custom Priority Queue:**
```java
public class PriorityMessageQueue {
    private final PriorityBlockingQueue<PriorityMessage> queue = 
        new PriorityBlockingQueue<>(100, 
            Comparator.comparing(PriorityMessage::getPriority).reversed());
    
    public void send(Message message, int priority) {
        queue.offer(new PriorityMessage(message, priority));
    }
    
    public Message receive() throws InterruptedException {
        PriorityMessage pm = queue.take();
        return pm.getMessage();
    }
    
    static class PriorityMessage {
        private final Message message;
        private final int priority;
        private final long timestamp = System.currentTimeMillis();
        
        // Getters...
    }
}
```

### Q6: Explain dead letter queue (DLQ).
**Answer:**
DLQ handles messages that can't be processed successfully.

```java
// RabbitMQ DLQ Configuration
@Configuration
public class DeadLetterConfig {
    
    @Bean
    public Queue mainQueue() {
        return QueueBuilder.durable("main-queue")
            .deadLetterExchange("dlx")
            .deadLetterRoutingKey("failed")
            .ttl(60000)  // Message TTL
            .maxLength(10000)  // Max queue length
            .build();
    }
    
    @Bean
    public Queue deadLetterQueue() {
        return new Queue("dead-letter-queue");
    }
    
    @Bean
    public DirectExchange deadLetterExchange() {
        return new DirectExchange("dlx");
    }
    
    @Bean
    public Binding deadLetterBinding() {
        return BindingBuilder
            .bind(deadLetterQueue())
            .to(deadLetterExchange())
            .with("failed");
    }
}

// Consumer with retry and DLQ
@Component
public class MessageConsumer {
    
    @RabbitListener(queues = "main-queue")
    public void process(Message message) {
        try {
            processMessage(message);
        } catch (Exception e) {
            // After max retries, message goes to DLQ
            throw new AmqpRejectAndDontRequeueException("Failed to process", e);
        }
    }
    
    @RabbitListener(queues = "dead-letter-queue")
    public void processDLQ(Message message) {
        // Handle failed messages
        alertingService.notifyFailure(message);
        failedMessageRepository.save(message);
    }
}
```

## Apache Kafka

### Q7: Explain Kafka architecture.
**Answer:**

**Components:**
1. **Producer**: Publishes messages to topics
2. **Consumer**: Subscribes to topics
3. **Broker**: Kafka server storing messages
4. **Topic**: Category of messages
5. **Partition**: Ordered, immutable sequence
6. **Zookeeper/KRaft**: Cluster coordination

**Key Concepts:**
```java
// Producer
@Component
public class KafkaProducer {
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    public void send(String topic, String key, Object message) {
        ProducerRecord<String, Object> record = 
            new ProducerRecord<>(topic, key, message);
        
        // Async send with callback
        kafkaTemplate.send(record).addCallback(
            result -> log.info("Sent message to partition {} offset {}",
                result.getRecordMetadata().partition(),
                result.getRecordMetadata().offset()),
            ex -> log.error("Failed to send message", ex)
        );
    }
}

// Consumer
@Component
public class KafkaConsumer {
    
    @KafkaListener(
        topics = "orders",
        groupId = "order-processing",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void consume(
        ConsumerRecord<String, Order> record,
        @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
        @Header(KafkaHeaders.OFFSET) long offset
    ) {
        log.info("Consumed from partition {} offset {}", partition, offset);
        processOrder(record.value());
    }
}
```

### Q8: How does Kafka ensure message ordering?
**Answer:**

**Partition-level ordering:**
```java
// Messages with same key go to same partition
producer.send("orders", order.getUserId(), order);  // userId as key

// All orders for a user are in order within partition
```

**Global ordering (single partition):**
```java
// Create topic with single partition
kafka-topics.sh --create --topic global-orders --partitions 1

// All messages globally ordered but no parallelism
```

**Preserve ordering with parallelism:**
```java
@Component
public class OrderedProcessor {
    private final Map<String, Queue<Order>> userQueues = new ConcurrentHashMap<>();
    private final Map<String, Lock> userLocks = new ConcurrentHashMap<>();
    
    @KafkaListener(topics = "orders")
    public void consume(Order order) {
        String userId = order.getUserId();
        Lock lock = userLocks.computeIfAbsent(userId, k -> new ReentrantLock());
        
        lock.lock();
        try {
            // Process orders for each user sequentially
            processOrder(order);
        } finally {
            lock.unlock();
        }
    }
}
```

### Q9: Explain Kafka consumer groups.
**Answer:**

Consumer groups enable parallel consumption with load balancing:

```java
// Configuration
@Configuration
public class KafkaConsumerConfig {
    
    @Bean
    public ConsumerFactory<String, Object> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-consumer-group");
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);  // Manual commit
        return new DefaultKafkaConsumerFactory<>(props);
    }
}

// Multiple consumers in same group
@Component
public class ConsumerGroup {
    
    // Consumer 1 - gets partitions 0,1
    @KafkaListener(
        topics = "orders",
        groupId = "order-group",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void consumer1(Order order) {
        processOrder(order);
    }
    
    // Consumer 2 - gets partitions 2,3
    @KafkaListener(
        topics = "orders",
        groupId = "order-group",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void consumer2(Order order) {
        processOrder(order);
    }
}

// Manual offset management
@KafkaListener(topics = "orders")
public void consume(Order order, Acknowledgment ack) {
    try {
        processOrder(order);
        ack.acknowledge();  // Commit offset
    } catch (Exception e) {
        // Don't acknowledge - message will be redelivered
    }
}
```

### Q10: How to handle Kafka transactions?
**Answer:**

```java
// Producer Transactions
@Component
public class TransactionalProducer {
    @Autowired
    private KafkaTransactionManager transactionManager;
    
    @Transactional
    public void sendInTransaction(List<Order> orders) {
        for (Order order : orders) {
            kafkaTemplate.send("orders", order);
            
            // If this fails, all messages in transaction are rolled back
            if (order.getAmount() > 10000) {
                auditService.audit(order);
            }
        }
    }
}

// Exactly-once semantics
@Configuration
public class KafkaConfig {
    @Bean
    public ProducerFactory<String, Object> producerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "tx-id");
        props.put(ProducerConfig.ACKS_CONFIG, "all");
        return new DefaultKafkaProducerFactory<>(props);
    }
}

// Consumer reading only committed messages
@Bean
public ConsumerFactory<String, Object> consumerFactory() {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");
    return new DefaultKafkaConsumerFactory<>(props);
}
```

## RabbitMQ

### Q11: Explain RabbitMQ exchange types.
**Answer:**

**1. Direct Exchange:**
```java
// Routes by exact routing key match
@Bean
public DirectExchange directExchange() {
    return new DirectExchange("direct-exchange");
}

@Bean
public Binding bindingDirect() {
    return BindingBuilder
        .bind(queue())
        .to(directExchange())
        .with("routing.key");
}

// Send
rabbitTemplate.convertAndSend("direct-exchange", "routing.key", message);
```

**2. Topic Exchange:**
```java
// Pattern matching with * (one word) and # (zero or more words)
@Bean
public TopicExchange topicExchange() {
    return new TopicExchange("topic-exchange");
}

@Bean
public Binding bindingTopic() {
    return BindingBuilder
        .bind(queue())
        .to(topicExchange())
        .with("order.*.created");  // Matches order.us.created, order.eu.created
}
```

**3. Fanout Exchange:**
```java
// Broadcasts to all bound queues
@Bean
public FanoutExchange fanoutExchange() {
    return new FanoutExchange("fanout-exchange");
}

@Bean
public Binding bindingFanout1() {
    return BindingBuilder.bind(queue1()).to(fanoutExchange());
}

@Bean
public Binding bindingFanout2() {
    return BindingBuilder.bind(queue2()).to(fanoutExchange());
}
```

**4. Headers Exchange:**
```java
// Routes based on message headers
@Bean
public HeadersExchange headersExchange() {
    return new HeadersExchange("headers-exchange");
}

@Bean
public Binding bindingHeaders() {
    Map<String, Object> headers = new HashMap<>();
    headers.put("format", "pdf");
    headers.put("type", "report");
    
    return BindingBuilder
        .bind(queue())
        .to(headersExchange())
        .whereAll(headers).match();
}
```

### Q12: How to implement RPC with RabbitMQ?
**Answer:**

```java
// Server (RPC Provider)
@Component
public class RpcServer {
    
    @RabbitListener(queues = "rpc-queue")
    public String processRpcRequest(String request) {
        // Process request and return response
        return "Response for: " + request;
    }
}

// Client (RPC Caller)
@Component
public class RpcClient {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public String callRpc(String request) {
        // Synchronous RPC call
        String response = (String) rabbitTemplate.convertSendAndReceive(
            "rpc-exchange",
            "rpc-routing-key",
            request,
            message -> {
                message.getMessageProperties().setExpiration("5000");
                message.getMessageProperties().setCorrelationId(UUID.randomUUID().toString());
                return message;
            }
        );
        
        if (response == null) {
            throw new TimeoutException("RPC call timed out");
        }
        
        return response;
    }
}

// Async RPC
public CompletableFuture<String> callRpcAsync(String request) {
    return CompletableFuture.supplyAsync(() -> {
        AsyncRabbitTemplate asyncTemplate = new AsyncRabbitTemplate(rabbitTemplate);
        
        RabbitConverterFuture<String> future = asyncTemplate.convertSendAndReceive(
            "rpc-exchange",
            "rpc-routing-key",
            request
        );
        
        try {
            return future.get(5, TimeUnit.SECONDS);
        } catch (Exception e) {
            throw new RuntimeException("RPC failed", e);
        }
    });
}
```

## Event-Driven Architecture

### Q13: What are the principles of event-driven architecture?
**Answer:**

**Key Principles:**
1. **Loose Coupling**: Services communicate through events
2. **Asynchronous**: Non-blocking communication
3. **Event-First Thinking**: Model domain as events
4. **Eventual Consistency**: Accept temporary inconsistency
5. **Choreography over Orchestration**: No central coordinator

**Implementation:**
```java
// Event Definition
public class OrderCreatedEvent {
    private String orderId;
    private String customerId;
    private BigDecimal amount;
    private Instant timestamp;
    // Getters, setters
}

// Event Publisher
@Component
public class OrderService {
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    @Transactional
    public Order createOrder(OrderRequest request) {
        Order order = new Order(request);
        orderRepository.save(order);
        
        // Publish event
        eventPublisher.publishEvent(new OrderCreatedEvent(order));
        
        return order;
    }
}

// Event Consumers
@Component
public class InventoryService {
    @EventListener
    @Async
    public void handleOrderCreated(OrderCreatedEvent event) {
        reserveInventory(event.getOrderId(), event.getItems());
    }
}

@Component
public class EmailService {
    @EventListener
    @Async
    public void handleOrderCreated(OrderCreatedEvent event) {
        sendOrderConfirmationEmail(event.getCustomerId(), event.getOrderId());
    }
}

@Component
public class AnalyticsService {
    @EventListener
    @Async
    public void handleOrderCreated(OrderCreatedEvent event) {
        updateOrderMetrics(event);
    }
}
```

### Q14: Explain event choreography vs orchestration.
**Answer:**

**Choreography (Distributed):**
```java
// Each service knows what to do when events occur
// No central coordinator

// Order Service
public void createOrder(Order order) {
    orderRepository.save(order);
    eventBus.publish(new OrderCreatedEvent(order));
}

// Payment Service listens and publishes
@EventListener
public void handleOrderCreated(OrderCreatedEvent event) {
    Payment payment = processPayment(event.getOrderId());
    eventBus.publish(new PaymentProcessedEvent(payment));
}

// Inventory Service listens to payment
@EventListener
public void handlePaymentProcessed(PaymentProcessedEvent event) {
    reserveInventory(event.getOrderId());
    eventBus.publish(new InventoryReservedEvent(event.getOrderId()));
}

// Shipping Service completes the flow
@EventListener
public void handleInventoryReserved(InventoryReservedEvent event) {
    scheduleShipping(event.getOrderId());
}
```

**Orchestration (Centralized):**
```java
// Central orchestrator manages the flow
@Component
public class OrderOrchestrator {
    
    public void processOrder(Order order) {
        try {
            // Step 1: Create order
            orderService.createOrder(order);
            
            // Step 2: Process payment
            PaymentResult payment = paymentService.processPayment(order);
            if (!payment.isSuccessful()) {
                throw new PaymentFailedException();
            }
            
            // Step 3: Reserve inventory
            InventoryResult inventory = inventoryService.reserve(order);
            if (!inventory.isAvailable()) {
                paymentService.refund(payment);
                throw new OutOfStockException();
            }
            
            // Step 4: Schedule shipping
            shippingService.schedule(order);
            
        } catch (Exception e) {
            // Centralized error handling and compensation
            compensate(order, e);
        }
    }
}
```

## Event Sourcing

### Q15: Explain Event Sourcing pattern.
**Answer:**

Event Sourcing stores all changes as a sequence of events:

```java
// Events
public abstract class DomainEvent {
    private String aggregateId;
    private Instant timestamp = Instant.now();
    private Long version;
}

public class AccountCreatedEvent extends DomainEvent {
    private String accountNumber;
    private String owner;
}

public class MoneyDepositedEvent extends DomainEvent {
    private BigDecimal amount;
}

public class MoneyWithdrawnEvent extends DomainEvent {
    private BigDecimal amount;
}

// Event Store
@Repository
public class EventStore {
    @Autowired
    private EventRepository repository;
    
    public void save(DomainEvent event) {
        EventEntity entity = new EventEntity();
        entity.setAggregateId(event.getAggregateId());
        entity.setEventType(event.getClass().getSimpleName());
        entity.setEventData(serialize(event));
        entity.setTimestamp(event.getTimestamp());
        entity.setVersion(getNextVersion(event.getAggregateId()));
        
        repository.save(entity);
    }
    
    public List<DomainEvent> getEvents(String aggregateId) {
        return repository.findByAggregateIdOrderByVersion(aggregateId)
            .stream()
            .map(this::deserialize)
            .collect(Collectors.toList());
    }
}

// Aggregate
public class BankAccount {
    private String accountId;
    private BigDecimal balance = BigDecimal.ZERO;
    private List<DomainEvent> uncommittedEvents = new ArrayList<>();
    
    // Command handler
    public void deposit(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        
        applyEvent(new MoneyDepositedEvent(accountId, amount));
    }
    
    // Apply event to state
    private void apply(MoneyDepositedEvent event) {
        this.balance = this.balance.add(event.getAmount());
    }
    
    // Load from event history
    public static BankAccount fromEvents(List<DomainEvent> events) {
        BankAccount account = new BankAccount();
        events.forEach(account::apply);
        return account;
    }
}

// CQRS Read Model
@Component
public class AccountProjection {
    
    @EventHandler
    public void on(AccountCreatedEvent event) {
        AccountView view = new AccountView();
        view.setAccountId(event.getAggregateId());
        view.setOwner(event.getOwner());
        view.setBalance(BigDecimal.ZERO);
        accountViewRepository.save(view);
    }
    
    @EventHandler
    public void on(MoneyDepositedEvent event) {
        AccountView view = accountViewRepository.findById(event.getAggregateId());
        view.setBalance(view.getBalance().add(event.getAmount()));
        view.setLastModified(event.getTimestamp());
        accountViewRepository.save(view);
    }
}
```

### Q16: What are the benefits and challenges of Event Sourcing?
**Answer:**

**Benefits:**
1. **Complete Audit Trail**: Every change is recorded
2. **Time Travel**: Reconstruct state at any point
3. **Event Replay**: Rebuild projections, fix bugs
4. **Analytics**: Rich event data for analysis
5. **Debugging**: See exactly what happened

**Challenges:**
1. **Complexity**: Steeper learning curve
2. **Event Schema Evolution**: Handling event versioning
3. **Storage**: Can grow large over time
4. **GDPR Compliance**: Deleting user data
5. **Eventual Consistency**: Read models may lag

**Solutions:**
```java
// Event Versioning
public interface EventUpgrader {
    boolean canUpgrade(String eventType, int version);
    DomainEvent upgrade(String eventData);
}

// Snapshotting for performance
@Component
public class SnapshotService {
    
    @Scheduled(fixedDelay = 60000)
    public void createSnapshots() {
        List<String> activeAggregates = getActiveAggregates();
        
        for (String aggregateId : activeAggregates) {
            List<DomainEvent> events = eventStore.getEventsSince(aggregateId, lastSnapshot);
            
            if (events.size() > 100) {  // Snapshot every 100 events
                Aggregate aggregate = Aggregate.fromEvents(events);
                Snapshot snapshot = new Snapshot(aggregateId, aggregate.getState());
                snapshotStore.save(snapshot);
            }
        }
    }
}
```

## Message Delivery Guarantees

### Q17: Explain different message delivery guarantees.
**Answer:**

**1. At-Most-Once (Fire and Forget):**
```java
// Message may be lost, never duplicated
public void sendAtMostOnce(Message message) {
    try {
        messageQueue.send(message);
        // Don't retry on failure
    } catch (Exception e) {
        log.error("Failed to send message", e);
        // Message is lost
    }
}
```

**2. At-Least-Once (Retry Until Success):**
```java
// Message never lost, may be duplicated
@Retryable(maxAttempts = 3, backoff = @Backoff(delay = 1000))
public void sendAtLeastOnce(Message message) {
    messageQueue.send(message);
    // Retry on failure - may cause duplicates
}

// Consumer must be idempotent
@KafkaListener(topics = "orders")
public void consume(Order order) {
    if (orderRepository.existsById(order.getId())) {
        return;  // Already processed
    }
    processOrder(order);
}
```

**3. Exactly-Once (Transactional):**
```java
// Kafka exactly-once
@Transactional
public void sendExactlyOnce(List<Message> messages) {
    for (Message message : messages) {
        kafkaTemplate.send("topic", message);
    }
    // All messages committed atomically
}

// Two-phase processing
public class ExactlyOnceProcessor {
    
    @Transactional
    public void process(Message message) {
        // Phase 1: Check if already processed
        if (processedMessages.contains(message.getId())) {
            return;
        }
        
        // Phase 2: Process and mark as done
        doProcess(message);
        processedMessages.add(message.getId());
    }
}
```

### Q18: How to ensure idempotency?
**Answer:**

**Idempotency Key:**
```java
@Service
public class IdempotentService {
    private final Set<String> processedKeys = new ConcurrentHashMap<>();
    
    public void process(Message message) {
        String idempotencyKey = message.getIdempotencyKey();
        
        if (!processedKeys.add(idempotencyKey)) {
            log.info("Message already processed: {}", idempotencyKey);
            return;
        }
        
        try {
            // Process message
            doProcess(message);
        } catch (Exception e) {
            processedKeys.remove(idempotencyKey);  // Allow retry
            throw e;
        }
    }
}

// Database-backed idempotency
@Entity
public class ProcessedMessage {
    @Id
    private String messageId;
    private Instant processedAt;
    private String result;
}

@Service
public class DatabaseIdempotency {
    
    @Transactional
    public void process(Message message) {
        // Use database constraint to ensure uniqueness
        ProcessedMessage processed = new ProcessedMessage();
        processed.setMessageId(message.getId());
        processed.setProcessedAt(Instant.now());
        
        try {
            processedMessageRepository.save(processed);
            // If save succeeds, this is first time processing
            doProcess(message);
        } catch (DataIntegrityViolationException e) {
            // Already processed
            log.info("Duplicate message: {}", message.getId());
        }
    }
}
```

## Integration Patterns

### Q19: Explain Saga pattern for distributed transactions.
**Answer:**

```java
// Choreography-based Saga
@Component
public class OrderSaga {
    
    @SagaOrchestrationStart
    public void handle(CreateOrderCommand command) {
        // Start saga
        Order order = new Order(command);
        orderRepository.save(order);
        
        eventBus.publish(new OrderCreatedEvent(order));
    }
    
    @SagaOrchestrationEnd
    @EventHandler
    public void handle(OrderCompletedEvent event) {
        // Saga completed successfully
        updateOrderStatus(event.getOrderId(), Status.COMPLETED);
    }
    
    @EventHandler
    public void handle(PaymentFailedEvent event) {
        // Compensate
        eventBus.publish(new CancelOrderCommand(event.getOrderId()));
    }
}

// State Machine-based Saga
public class OrderStateMachine {
    
    enum State {
        CREATED, PAYMENT_PENDING, PAYMENT_COMPLETED, 
        INVENTORY_RESERVED, SHIPPED, COMPLETED, CANCELLED
    }
    
    @Autowired
    private StateMachineFactory<State, Event> factory;
    
    public void processOrder(String orderId) {
        StateMachine<State, Event> sm = factory.getStateMachine(orderId);
        
        sm.start();
        sm.sendEvent(Event.ORDER_CREATED);
        
        // State transitions trigger actions
    }
    
    @OnTransition(source = "CREATED", target = "PAYMENT_PENDING")
    public void processPayment() {
        paymentService.charge(order);
    }
    
    @OnTransition(source = "PAYMENT_COMPLETED", target = "INVENTORY_RESERVED")
    public void reserveInventory() {
        inventoryService.reserve(order);
    }
}
```

### Q20: How to implement Circuit Breaker for messaging?
**Answer:**

```java
@Component
public class MessageCircuitBreaker {
    private final CircuitBreaker circuitBreaker;
    
    public MessageCircuitBreaker() {
        this.circuitBreaker = CircuitBreaker.ofDefaults("messaging");
        circuitBreaker.getEventPublisher()
            .onStateTransition(event -> 
                log.info("Circuit breaker state transition: {}", event));
    }
    
    public void sendMessage(Message message) {
        Supplier<Void> decoratedSupplier = CircuitBreaker
            .decorateSupplier(circuitBreaker, () -> {
                messageQueue.send(message);
                return null;
            });
        
        Try.ofSupplier(decoratedSupplier)
            .recover(throwable -> {
                // Circuit open - use fallback
                fallbackQueue.send(message);
                return null;
            });
    }
}

// Resilience4j Configuration
@Configuration
public class CircuitBreakerConfig {
    
    @Bean
    public CircuitBreakerConfigCustomizer customize() {
        return CircuitBreakerConfigCustomizer
            .of("messaging", builder -> builder
                .slidingWindowSize(10)
                .minimumNumberOfCalls(5)
                .failureRateThreshold(50)
                .waitDurationInOpenState(Duration.ofSeconds(30))
                .permittedNumberOfCallsInHalfOpenState(3)
                .automaticTransitionFromOpenToHalfOpenEnabled(true)
            );
    }
}
```

## Real-World Implementations

### Q21: Design a notification system using messaging.
**Answer:**

```java
// Notification Service Architecture
@Component
public class NotificationOrchestrator {
    
    @KafkaListener(topics = "notifications")
    public void handleNotification(NotificationRequest request) {
        // Determine channels based on user preferences
        UserPreferences prefs = userService.getPreferences(request.getUserId());
        
        if (prefs.isEmailEnabled()) {
            emailQueue.send(new EmailNotification(request));
        }
        
        if (prefs.isPushEnabled()) {
            pushQueue.send(new PushNotification(request));
        }
        
        if (prefs.isSmsEnabled() && request.getPriority() == HIGH) {
            smsQueue.send(new SmsNotification(request));
        }
    }
}

// Email processor with batching
@Component
public class EmailProcessor {
    private final List<EmailNotification> batch = new ArrayList<>();
    
    @RabbitListener(queues = "email-queue")
    public void process(EmailNotification notification) {
        batch.add(notification);
        
        if (batch.size() >= 100) {
            sendBatch();
        }
    }
    
    @Scheduled(fixedDelay = 5000)
    public void sendBatch() {
        if (!batch.isEmpty()) {
            emailService.sendBatch(new ArrayList<>(batch));
            batch.clear();
        }
    }
}

// Priority-based SMS processing
@Component
public class SmsProcessor {
    
    @RabbitListener(queues = "sms-high-priority")
    public void processHighPriority(SmsNotification notification) {
        smsService.sendImmediate(notification);
    }
    
    @RabbitListener(queues = "sms-normal-priority", concurrency = "1")
    public void processNormalPriority(SmsNotification notification) {
        // Rate limited processing
        rateLimiter.acquire();
        smsService.send(notification);
    }
}
```

### Q22: Implement event-driven order processing system.
**Answer:**

```java
// Order Aggregate
@Component
public class OrderAggregate {
    
    @CommandHandler
    public void handle(CreateOrderCommand command) {
        // Validate
        if (command.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order must have items");
        }
        
        // Create order
        Order order = new Order(command);
        orderRepository.save(order);
        
        // Publish event
        eventBus.publish(OrderCreatedEvent.builder()
            .orderId(order.getId())
            .customerId(command.getCustomerId())
            .items(command.getItems())
            .totalAmount(order.calculateTotal())
            .timestamp(Instant.now())
            .build());
    }
}

// Payment Service
@Component
public class PaymentEventHandler {
    
    @EventHandler
    @Async
    public void on(OrderCreatedEvent event) {
        try {
            PaymentResult result = paymentGateway.charge(
                event.getCustomerId(),
                event.getTotalAmount()
            );
            
            if (result.isSuccessful()) {
                eventBus.publish(new PaymentCompletedEvent(
                    event.getOrderId(),
                    result.getTransactionId()
                ));
            } else {
                eventBus.publish(new PaymentFailedEvent(
                    event.getOrderId(),
                    result.getFailureReason()
                ));
            }
        } catch (Exception e) {
            eventBus.publish(new PaymentFailedEvent(
                event.getOrderId(),
                e.getMessage()
            ));
        }
    }
}

// Inventory Service
@Component
public class InventoryEventHandler {
    
    @EventHandler
    public void on(PaymentCompletedEvent event) {
        String orderId = event.getOrderId();
        Order order = orderRepository.findById(orderId);
        
        try {
            for (OrderItem item : order.getItems()) {
                inventoryService.reserve(item.getProductId(), item.getQuantity());
            }
            
            eventBus.publish(new InventoryReservedEvent(orderId));
            
        } catch (InsufficientInventoryException e) {
            // Compensate
            eventBus.publish(new RefundRequestedEvent(
                orderId,
                event.getTransactionId()
            ));
        }
    }
}

// Shipping Service
@Component
public class ShippingEventHandler {
    
    @EventHandler
    public void on(InventoryReservedEvent event) {
        Shipment shipment = shippingService.scheduleShipment(event.getOrderId());
        
        eventBus.publish(new ShipmentScheduledEvent(
            event.getOrderId(),
            shipment.getTrackingNumber(),
            shipment.getEstimatedDelivery()
        ));
    }
}

// Order Status Updater
@Component
public class OrderStatusUpdater {
    
    @EventHandler
    public void on(OrderCreatedEvent event) {
        updateStatus(event.getOrderId(), OrderStatus.PENDING_PAYMENT);
    }
    
    @EventHandler
    public void on(PaymentCompletedEvent event) {
        updateStatus(event.getOrderId(), OrderStatus.PAYMENT_RECEIVED);
    }
    
    @EventHandler
    public void on(InventoryReservedEvent event) {
        updateStatus(event.getOrderId(), OrderStatus.PREPARING);
    }
    
    @EventHandler
    public void on(ShipmentScheduledEvent event) {
        updateStatus(event.getOrderId(), OrderStatus.SHIPPED);
    }
    
    private void updateStatus(String orderId, OrderStatus status) {
        Order order = orderRepository.findById(orderId);
        order.setStatus(status);
        order.setLastModified(Instant.now());
        orderRepository.save(order);
    }
}
```

### Q23: Best practices for messaging systems?
**Answer:**

1. **Message Design:**
   - Keep messages small and focused
   - Use schemas (Avro, Protobuf)
   - Version your messages
   - Include correlation IDs

2. **Error Handling:**
   - Implement retry with exponential backoff
   - Use dead letter queues
   - Monitor failed messages
   - Set appropriate timeouts

3. **Performance:**
   - Batch processing when possible
   - Use async processing
   - Implement backpressure
   - Monitor queue depths

4. **Reliability:**
   - Choose appropriate delivery guarantee
   - Implement idempotency
   - Use transactions when needed
   - Regular health checks

5. **Monitoring:**
   - Track message rates
   - Monitor latency
   - Alert on queue buildup
   - Log correlation IDs

6. **Security:**
   - Encrypt sensitive data
   - Use authentication/authorization
   - Audit message access
   - Validate message content