# Microservices Architecture & Patterns Interview Questions

## Table of Contents
1. [Microservices Fundamentals](#microservices-fundamentals)
2. [Communication Patterns](#communication-patterns)
3. [Data Management Patterns](#data-management-patterns)
4. [Saga Pattern](#saga-pattern)
5. [Event Sourcing & CQRS](#event-sourcing--cqrs)
6. [Service Discovery & Registry](#service-discovery--registry)
7. [API Gateway Pattern](#api-gateway-pattern)
8. [Circuit Breaker & Resilience](#circuit-breaker--resilience)
9. [Distributed Tracing & Monitoring](#distributed-tracing--monitoring)
10. [Security Patterns](#security-patterns)
11. [Deployment & DevOps](#deployment--devops)

## Microservices Fundamentals

### Q1: What are microservices and their characteristics?
**Answer:**
Microservices is an architectural style where applications are built as a collection of small, autonomous services.

**Key Characteristics:**
1. **Single Responsibility**: Each service has one business capability
2. **Autonomous**: Services are independently deployable
3. **Decentralized**: No single point of control
4. **Fault Isolation**: Failure in one service doesn't cascade
5. **Technology Agnostic**: Different tech stacks possible
6. **Smart Endpoints**: Business logic in services, not middleware
7. **Design for Failure**: Expect and handle failures gracefully

### Q2: What are the advantages and disadvantages of microservices?
**Answer:**
**Advantages:**
- Independent deployment and scaling
- Technology diversity
- Fault isolation
- Better team autonomy
- Easier to understand individual services
- Supports continuous delivery

**Disadvantages:**
- Distributed system complexity
- Network latency
- Data consistency challenges
- Operational overhead
- Testing complexity
- Service coordination difficulties

### Q3: When should you use microservices?
**Answer:**
**Good fit when:**
- Large, complex applications
- Multiple teams working independently
- Different parts need different scaling
- Need for technology diversity
- Rapid feature development required
- High availability requirements

**Not suitable when:**
- Small, simple applications
- Small team (< 5 developers)
- Tight data consistency requirements
- Limited DevOps capabilities
- Proof of concept or MVP stage

### Q4: What is Domain-Driven Design (DDD) in microservices?
**Answer:**
DDD helps identify service boundaries through:
- **Bounded Context**: Clear boundaries where a model applies
- **Ubiquitous Language**: Common vocabulary within context
- **Aggregates**: Cluster of domain objects
- **Domain Events**: Something that happened in the domain
- **Context Mapping**: Relationships between bounded contexts

```java
// Example: E-commerce bounded contexts
OrderService (Order Context)
├── Order Aggregate
├── OrderItem Entity
└── Payment Value Object

InventoryService (Inventory Context)
├── Product Aggregate
├── Stock Entity
└── Location Value Object

CustomerService (Customer Context)
├── Customer Aggregate
├── Address Value Object
└── Preferences Entity
```

## Communication Patterns

### Q5: What are different communication patterns in microservices?
**Answer:**
**1. Synchronous Communication:**
- REST APIs (HTTP/HTTPS)
- GraphQL
- gRPC
- WebSockets

**2. Asynchronous Communication:**
- Message Queues (RabbitMQ, AWS SQS)
- Event Streaming (Kafka, AWS Kinesis)
- Pub/Sub (Redis Pub/Sub, Google Pub/Sub)

**3. Service Mesh:**
- Istio, Linkerd, Consul Connect
- Handles service-to-service communication

### Q6: Explain REST vs gRPC vs GraphQL for microservices.
**Answer:**
| Aspect | REST | gRPC | GraphQL |
|--------|------|------|---------|
| Protocol | HTTP/1.1, HTTP/2 | HTTP/2 | HTTP |
| Data Format | JSON, XML | Protocol Buffers | JSON |
| Schema | OpenAPI/Swagger | Proto files | SDL |
| Communication | Request/Response | Unary, Streaming | Query/Mutation |
| Performance | Good | Excellent | Good |
| Use Case | Public APIs | Internal services | Flexible queries |

### Q7: How to handle service versioning?
**Answer:**
**Strategies:**
1. **URL Versioning**: `/api/v1/users`, `/api/v2/users`
2. **Header Versioning**: `Accept-Version: v1`
3. **Query Parameter**: `/api/users?version=1`
4. **Content Negotiation**: `Accept: application/vnd.api.v1+json`

**Best Practices:**
- Maintain backward compatibility
- Deprecate gradually
- Use semantic versioning
- Document breaking changes
- Support multiple versions temporarily

### Q8: What is the Backend for Frontend (BFF) pattern?
**Answer:**
BFF creates separate backend services for different frontend applications:

```
Mobile App → Mobile BFF → Microservices
Web App → Web BFF → Microservices
Desktop App → Desktop BFF → Microservices
```

**Benefits:**
- Optimized for specific frontend needs
- Reduces over-fetching/under-fetching
- Simplifies frontend logic
- Better security control

## Data Management Patterns

### Q9: How to manage data in microservices?
**Answer:**
**Database per Service Pattern:**
- Each service owns its database
- No direct database access between services
- Data exchange through APIs or events

**Challenges:**
- Distributed transactions
- Data consistency
- Query complexity
- Data duplication

**Solutions:**
- Saga pattern for transactions
- Event sourcing for audit
- CQRS for query optimization
- API composition for joins

### Q10: What is the Shared Database anti-pattern?
**Answer:**
Multiple services sharing the same database is an anti-pattern because:
- Creates tight coupling
- Prevents independent deployment
- Limits technology choices
- Makes scaling difficult
- Schema changes affect multiple services

**Exception:** Legacy system migration where temporary sharing is necessary.

### Q11: How to handle distributed queries?
**Answer:**
**1. API Composition:**
```java
@Service
public class OrderDetailsService {
    public OrderDetails getOrderDetails(Long orderId) {
        Order order = orderService.getOrder(orderId);
        Customer customer = customerService.getCustomer(order.getCustomerId());
        List<Product> products = productService.getProducts(order.getProductIds());
        return new OrderDetails(order, customer, products);
    }
}
```

**2. CQRS with Materialized Views:**
```java
// Command side updates multiple services
// Query side reads from denormalized view
@EventHandler
public void on(OrderPlacedEvent event) {
    OrderView view = new OrderView();
    view.setOrderId(event.getOrderId());
    view.setCustomerName(event.getCustomerName());
    view.setProducts(event.getProducts());
    orderViewRepository.save(view);
}
```

**3. Data Replication:**
- Services maintain local copies of required data
- Updated through events
- Eventually consistent

## Saga Pattern

### Q12: What is the Saga pattern?
**Answer:**
Saga manages distributed transactions across multiple services by breaking them into a series of local transactions.

**Types:**
1. **Choreography**: Services coordinate through events
2. **Orchestration**: Central orchestrator manages workflow

### Q13: Implement Choreography-based Saga.
**Answer:**
```java
// Order Service
@Service
public class OrderService {
    @Autowired
    private EventPublisher eventPublisher;
    
    public void createOrder(Order order) {
        // Local transaction
        orderRepository.save(order);
        
        // Publish event
        eventPublisher.publish(new OrderCreatedEvent(order));
    }
    
    @EventHandler
    public void handle(PaymentFailedEvent event) {
        // Compensating transaction
        Order order = orderRepository.findById(event.getOrderId());
        order.setStatus(OrderStatus.CANCELLED);
        orderRepository.save(order);
    }
}

// Payment Service
@Service
public class PaymentService {
    @EventHandler
    public void handle(OrderCreatedEvent event) {
        try {
            // Process payment
            Payment payment = processPayment(event);
            eventPublisher.publish(new PaymentSuccessEvent(payment));
        } catch (Exception e) {
            eventPublisher.publish(new PaymentFailedEvent(event.getOrderId()));
        }
    }
}

// Inventory Service
@Service
public class InventoryService {
    @EventHandler
    public void handle(PaymentSuccessEvent event) {
        try {
            // Reserve inventory
            reserveInventory(event.getOrderId());
            eventPublisher.publish(new InventoryReservedEvent(event.getOrderId()));
        } catch (Exception e) {
            eventPublisher.publish(new InventoryFailedEvent(event.getOrderId()));
        }
    }
    
    @EventHandler
    public void handle(PaymentFailedEvent event) {
        // Compensating transaction - release inventory if reserved
        releaseInventory(event.getOrderId());
    }
}
```

### Q14: Implement Orchestration-based Saga.
**Answer:**
```java
@Service
public class OrderSagaOrchestrator {
    
    @Autowired
    private OrderService orderService;
    @Autowired
    private PaymentService paymentService;
    @Autowired
    private InventoryService inventoryService;
    
    public void processOrder(OrderRequest request) {
        SagaTransaction saga = new SagaTransaction();
        
        try {
            // Step 1: Create Order
            Order order = orderService.createOrder(request);
            saga.addCompensation(() -> orderService.cancelOrder(order.getId()));
            
            // Step 2: Process Payment
            Payment payment = paymentService.processPayment(order);
            saga.addCompensation(() -> paymentService.refund(payment.getId()));
            
            // Step 3: Reserve Inventory
            Reservation reservation = inventoryService.reserve(order.getItems());
            saga.addCompensation(() -> inventoryService.release(reservation.getId()));
            
            // Step 4: Confirm Order
            orderService.confirmOrder(order.getId());
            
        } catch (Exception e) {
            // Execute compensating transactions in reverse order
            saga.compensate();
            throw new OrderProcessingException("Order processing failed", e);
        }
    }
}

class SagaTransaction {
    private Stack<Runnable> compensations = new Stack<>();
    
    public void addCompensation(Runnable compensation) {
        compensations.push(compensation);
    }
    
    public void compensate() {
        while (!compensations.isEmpty()) {
            try {
                compensations.pop().run();
            } catch (Exception e) {
                // Log and continue compensating
            }
        }
    }
}
```

## Event Sourcing & CQRS

### Q15: What is Event Sourcing?
**Answer:**
Event Sourcing stores the state of an application as a sequence of events rather than current state.

**Benefits:**
- Complete audit log
- Temporal queries
- Event replay
- Debugging capabilities

**Implementation:**
```java
@Entity
public class Event {
    @Id
    private String eventId;
    private String aggregateId;
    private String eventType;
    private String eventData;
    private Instant timestamp;
    private Long version;
}

@Service
public class AccountAggregate {
    private String accountId;
    private BigDecimal balance;
    private List<Event> uncommittedEvents = new ArrayList<>();
    
    public void debit(BigDecimal amount) {
        if (balance.compareTo(amount) < 0) {
            throw new InsufficientFundsException();
        }
        applyEvent(new AccountDebitedEvent(accountId, amount));
    }
    
    public void credit(BigDecimal amount) {
        applyEvent(new AccountCreditedEvent(accountId, amount));
    }
    
    private void applyEvent(Event event) {
        // Apply event to current state
        if (event instanceof AccountDebitedEvent) {
            balance = balance.subtract(((AccountDebitedEvent) event).getAmount());
        } else if (event instanceof AccountCreditedEvent) {
            balance = balance.add(((AccountCreditedEvent) event).getAmount());
        }
        uncommittedEvents.add(event);
    }
    
    public static AccountAggregate loadFromHistory(List<Event> events) {
        AccountAggregate aggregate = new AccountAggregate();
        events.forEach(aggregate::applyEvent);
        return aggregate;
    }
}
```

### Q16: What is CQRS pattern?
**Answer:**
CQRS (Command Query Responsibility Segregation) separates read and write operations into different models.

**Components:**
- **Command Model**: Handles writes, enforces business rules
- **Query Model**: Optimized for reads
- **Event Store**: Source of truth
- **Projections**: Read models built from events

```java
// Command Side
@Service
public class OrderCommandService {
    @Autowired
    private EventStore eventStore;
    
    public void createOrder(CreateOrderCommand command) {
        Order order = new Order(command);
        order.validate();
        
        List<Event> events = order.getUncommittedEvents();
        eventStore.save(events);
        eventBus.publish(events);
    }
}

// Query Side
@Service
public class OrderQueryService {
    @Autowired
    private OrderViewRepository repository;
    
    @EventHandler
    public void on(OrderCreatedEvent event) {
        OrderView view = new OrderView();
        view.setOrderId(event.getOrderId());
        view.setCustomerName(event.getCustomerName());
        view.setTotal(event.getTotal());
        repository.save(view);
    }
    
    public OrderView findOrder(String orderId) {
        return repository.findById(orderId);
    }
    
    public List<OrderView> findOrdersByCustomer(String customerId) {
        return repository.findByCustomerId(customerId);
    }
}
```

### Q17: How to handle eventual consistency in CQRS?
**Answer:**
**Strategies:**
1. **UI Feedback**: Show pending state
2. **Polling**: Check for updates
3. **WebSockets**: Push updates to client
4. **Versioning**: Track read model versions
5. **Compensation**: Handle inconsistencies

```java
@RestController
public class OrderController {
    
    @PostMapping("/orders")
    public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) {
        String orderId = UUID.randomUUID().toString();
        
        // Send command
        commandGateway.send(new CreateOrderCommand(orderId, request));
        
        // Return immediately with status
        return ResponseEntity.accepted()
            .location(URI.create("/orders/" + orderId))
            .body(new OrderResponse(orderId, "PENDING"));
    }
    
    @GetMapping("/orders/{id}")
    public ResponseEntity<OrderView> getOrder(@PathVariable String id) {
        Optional<OrderView> order = queryService.findOrder(id);
        
        if (order.isEmpty()) {
            // Check if command is still processing
            if (commandStore.isPending(id)) {
                return ResponseEntity.status(HttpStatus.ACCEPTED)
                    .header("Retry-After", "2")
                    .build();
            }
            return ResponseEntity.notFound().build();
        }
        
        return ResponseEntity.ok(order.get());
    }
}
```

## Service Discovery & Registry

### Q18: What is Service Discovery?
**Answer:**
Service Discovery automatically detects services and their network locations in a distributed system.

**Types:**
1. **Client-Side Discovery**: Client queries registry and load balances
2. **Server-Side Discovery**: Load balancer queries registry

**Popular Solutions:**
- Netflix Eureka
- Consul
- Kubernetes Service Discovery
- AWS Cloud Map

### Q19: Implement Service Discovery with Eureka.
**Answer:**
**Eureka Server:**
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

// application.yml
server:
  port: 8761
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

**Service Registration:**
```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

// application.yml
spring:
  application:
    name: order-service
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${random.value}
```

**Service Discovery:**
```java
@Service
public class PaymentService {
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @Autowired
    @LoadBalanced
    private RestTemplate restTemplate;
    
    // Option 1: Using DiscoveryClient
    public Order getOrder(String orderId) {
        List<ServiceInstance> instances = 
            discoveryClient.getInstances("order-service");
        
        if (instances.isEmpty()) {
            throw new ServiceUnavailableException("Order service not available");
        }
        
        ServiceInstance instance = instances.get(0);
        String url = instance.getUri() + "/orders/" + orderId;
        return restTemplate.getForObject(url, Order.class);
    }
    
    // Option 2: Using LoadBalanced RestTemplate
    public Order getOrderWithLoadBalancing(String orderId) {
        return restTemplate.getForObject(
            "http://order-service/orders/" + orderId, 
            Order.class
        );
    }
}
```

## API Gateway Pattern

### Q20: What is an API Gateway?
**Answer:**
API Gateway is a single entry point for all client requests, handling:
- Request routing
- Authentication/authorization
- Rate limiting
- Request/response transformation
- Load balancing
- Circuit breaking
- Monitoring

### Q21: Implement API Gateway with Spring Cloud Gateway.
**Answer:**
```java
@SpringBootApplication
@EnableDiscoveryClient
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // Route to order service
            .route("order-service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .rewritePath("/api/orders/(?<segment>.*)", "/orders/${segment}")
                    .addRequestHeader("X-Gateway-Request", "true")
                    .circuitBreaker(c -> c
                        .setName("orderServiceCB")
                        .setFallbackUri("forward:/fallback/orders"))
                    .retry(config -> config.setRetries(3))
                )
                .uri("lb://order-service"))
            
            // Route to customer service
            .route("customer-service", r -> r
                .path("/api/customers/**")
                .filters(f -> f
                    .rewritePath("/api/customers/(?<segment>.*)", "/customers/${segment}")
                    .requestRateLimiter(c -> c
                        .setRateLimiter(redisRateLimiter())
                        .setKeyResolver(userKeyResolver()))
                )
                .uri("lb://customer-service"))
            .build();
    }
    
    @Bean
    public RedisRateLimiter redisRateLimiter() {
        return new RedisRateLimiter(10, 20); // 10 requests per second, burst 20
    }
    
    @Bean
    KeyResolver userKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest().getHeaders().getFirst("X-User-Id")
        );
    }
    
    // Global filters
    @Component
    public class AuthenticationFilter implements GlobalFilter, Ordered {
        @Autowired
        private JwtUtil jwtUtil;
        
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, 
                                GatewayFilterChain chain) {
            ServerHttpRequest request = exchange.getRequest();
            
            if (!request.getHeaders().containsKey("Authorization")) {
                return onError(exchange, "No Authorization header", 
                             HttpStatus.UNAUTHORIZED);
            }
            
            String token = request.getHeaders().get("Authorization").get(0);
            
            if (!jwtUtil.validateToken(token)) {
                return onError(exchange, "Invalid token", 
                             HttpStatus.UNAUTHORIZED);
            }
            
            return chain.filter(exchange);
        }
        
        @Override
        public int getOrder() {
            return -100;
        }
        
        private Mono<Void> onError(ServerWebExchange exchange, 
                                  String err, HttpStatus httpStatus) {
            ServerHttpResponse response = exchange.getResponse();
            response.setStatusCode(httpStatus);
            return response.setComplete();
        }
    }
}
```

## Circuit Breaker & Resilience

### Q22: What is the Circuit Breaker pattern?
**Answer:**
Circuit Breaker prevents cascading failures by monitoring service calls and breaking the circuit when failures exceed threshold.

**States:**
1. **Closed**: Normal operation, requests pass through
2. **Open**: Requests fail immediately without calling service
3. **Half-Open**: Limited requests to test service recovery

### Q23: Implement Circuit Breaker with Resilience4j.
**Answer:**
```java
@Service
public class OrderService {
    
    @CircuitBreaker(name = "order-service", fallbackMethod = "fallbackGetOrder")
    @Retry(name = "order-service")
    @Bulkhead(name = "order-service")
    @TimeLimiter(name = "order-service")
    public CompletableFuture<Order> getOrder(String orderId) {
        return CompletableFuture.supplyAsync(() -> 
            orderClient.getOrder(orderId)
        );
    }
    
    public CompletableFuture<Order> fallbackGetOrder(String orderId, Exception ex) {
        // Return cached or default response
        return CompletableFuture.completedFuture(
            new Order(orderId, "Fallback Order", OrderStatus.UNKNOWN)
        );
    }
}

// Configuration
@Configuration
public class ResilienceConfig {
    
    @Bean
    public CircuitBreakerConfig circuitBreakerConfig() {
        return CircuitBreakerConfig.custom()
            .slidingWindowType(SlidingWindowType.COUNT_BASED)
            .slidingWindowSize(10)
            .minimumNumberOfCalls(5)
            .failureRateThreshold(50)
            .waitDurationInOpenState(Duration.ofSeconds(10))
            .permittedNumberOfCallsInHalfOpenState(3)
            .slowCallRateThreshold(50)
            .slowCallDurationThreshold(Duration.ofSeconds(2))
            .build();
    }
    
    @Bean
    public RetryConfig retryConfig() {
        return RetryConfig.custom()
            .maxAttempts(3)
            .waitDuration(Duration.ofMillis(500))
            .retryExceptions(IOException.class, TimeoutException.class)
            .ignoreExceptions(BusinessException.class)
            .build();
    }
    
    @Bean
    public BulkheadConfig bulkheadConfig() {
        return BulkheadConfig.custom()
            .maxConcurrentCalls(10)
            .maxWaitDuration(Duration.ofMillis(100))
            .build();
    }
}
```

### Q24: What is the Bulkhead pattern?
**Answer:**
Bulkhead isolates resources to prevent failure in one area from affecting others.

**Types:**
1. **Semaphore Bulkhead**: Limits concurrent calls
2. **Thread Pool Bulkhead**: Isolates calls in separate thread pool

```java
@Service
public class PaymentService {
    
    // Semaphore bulkhead
    @Bulkhead(name = "payment-service", type = Bulkhead.Type.SEMAPHORE)
    public Payment processPayment(PaymentRequest request) {
        return paymentProcessor.process(request);
    }
    
    // Thread pool bulkhead
    @Bulkhead(name = "external-service", type = Bulkhead.Type.THREADPOOL)
    public CompletableFuture<String> callExternalService() {
        return CompletableFuture.supplyAsync(() -> 
            externalClient.call()
        );
    }
}
```

## Distributed Tracing & Monitoring

### Q25: How to implement distributed tracing?
**Answer:**
**Using Spring Cloud Sleuth and Zipkin:**
```java
// Dependencies
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>

// Configuration
spring:
  sleuth:
    sampler:
      probability: 1.0 # Sample all requests
  zipkin:
    base-url: http://localhost:9411

// Custom spans
@Service
public class OrderService {
    @Autowired
    private Tracer tracer;
    
    public Order processOrder(OrderRequest request) {
        Span span = tracer.nextSpan().name("process-order");
        
        try (Tracer.SpanInScope ws = tracer.withSpanInScope(span.start())) {
            span.tag("order.id", request.getOrderId());
            span.tag("customer.id", request.getCustomerId());
            
            // Process order
            Order order = createOrder(request);
            
            span.event("Order created");
            
            return order;
        } finally {
            span.end();
        }
    }
}
```

### Q26: What metrics should be monitored in microservices?
**Answer:**
**Golden Signals:**
1. **Latency**: Response time
2. **Traffic**: Requests per second
3. **Errors**: Error rate
4. **Saturation**: Resource utilization

**Implementation with Micrometer:**
```java
@RestController
public class OrderController {
    private final MeterRegistry meterRegistry;
    private final Counter orderCounter;
    private final Timer orderTimer;
    
    public OrderController(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.orderCounter = Counter.builder("orders.created")
            .description("Number of orders created")
            .register(meterRegistry);
        this.orderTimer = Timer.builder("order.processing.time")
            .description("Order processing time")
            .register(meterRegistry);
    }
    
    @PostMapping("/orders")
    public Order createOrder(@RequestBody OrderRequest request) {
        return orderTimer.recordCallable(() -> {
            Order order = orderService.create(request);
            orderCounter.increment();
            
            // Custom metrics
            meterRegistry.gauge("orders.pending", 
                orderService.getPendingCount());
            
            return order;
        });
    }
}
```

## Security Patterns

### Q27: How to implement security in microservices?
**Answer:**
**Security Layers:**
1. **Edge Security**: API Gateway authentication
2. **Service-to-Service**: mTLS, service mesh
3. **Token Propagation**: JWT, OAuth2
4. **Secret Management**: Vault, AWS Secrets Manager

**Implementation:**
```java
// API Gateway Security
@Component
public class JwtAuthenticationFilter extends AbstractGatewayFilterFactory<Config> {
    
    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            
            if (!request.getHeaders().containsKey("Authorization")) {
                return onError(exchange, "Missing authorization", 
                             HttpStatus.UNAUTHORIZED);
            }
            
            String token = request.getHeaders().get("Authorization").get(0);
            
            try {
                Claims claims = validateToken(token);
                
                // Add user context to headers
                ServerHttpRequest modifiedRequest = exchange.getRequest()
                    .mutate()
                    .header("X-User-Id", claims.getSubject())
                    .header("X-User-Roles", claims.get("roles", String.class))
                    .build();
                
                return chain.filter(exchange.mutate()
                    .request(modifiedRequest).build());
                    
            } catch (Exception e) {
                return onError(exchange, "Invalid token", 
                             HttpStatus.UNAUTHORIZED);
            }
        };
    }
}

// Service-level Security
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/internal/**").hasIpAddress("10.0.0.0/8")
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt())
            .csrf().disable();
            
        return http.build();
    }
}
```

### Q28: What is the Token Relay pattern?
**Answer:**
Token Relay passes authentication tokens through the service chain:

```java
@Component
public class TokenRelayGatewayFilterFactory 
       extends AbstractGatewayFilterFactory<Object> {
    
    @Autowired
    private TokenRelayService tokenRelayService;
    
    @Override
    public GatewayFilter apply(Object config) {
        return (exchange, chain) -> {
            return tokenRelayService.getAccessToken()
                .map(token -> {
                    ServerHttpRequest request = exchange.getRequest()
                        .mutate()
                        .header("Authorization", "Bearer " + token)
                        .build();
                    return exchange.mutate().request(request).build();
                })
                .defaultIfEmpty(exchange)
                .flatMap(chain::filter);
        };
    }
}
```

## Deployment & DevOps

### Q29: What are deployment strategies for microservices?
**Answer:**
**1. Blue-Green Deployment:**
- Two identical environments (blue/green)
- Switch traffic after deployment
- Easy rollback

**2. Canary Deployment:**
- Gradual rollout to subset of users
- Monitor metrics
- Increase traffic progressively

**3. Rolling Deployment:**
- Update instances one by one
- Zero downtime
- Slower rollout

**Implementation with Kubernetes:**
```yaml
# Canary Deployment
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order
  ports:
    - port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: order
      version: stable
  template:
    metadata:
      labels:
        app: order
        version: stable
    spec:
      containers:
      - name: order
        image: order-service:1.0
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service-canary
spec:
  replicas: 1  # 10% traffic
  selector:
    matchLabels:
      app: order
      version: canary
  template:
    metadata:
      labels:
        app: order
        version: canary
    spec:
      containers:
      - name: order
        image: order-service:2.0
```

### Q30: How to handle configuration management?
**Answer:**
**Externalized Configuration:**
```java
// Spring Cloud Config Server
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

// Config Client
@SpringBootApplication
@RefreshScope
public class OrderServiceApplication {
    @Value("${order.processing.timeout:30}")
    private int processingTimeout;
    
    @Value("${order.max.items:100}")
    private int maxItems;
}

// Dynamic refresh
@RestController
public class ConfigController {
    @PostMapping("/actuator/refresh")
    public void refresh() {
        // Triggers configuration refresh
    }
}
```

## Best Practices & Anti-Patterns

### Q31: What are microservices best practices?
**Answer:**
1. **Design for failure**: Implement circuit breakers, retries, timeouts
2. **Decentralized data**: Each service owns its data
3. **Smart endpoints, dumb pipes**: Business logic in services
4. **Design around business capabilities**: Not technical layers
5. **Automate everything**: CI/CD, testing, monitoring
6. **Version APIs carefully**: Backward compatibility
7. **Monitor everything**: Logs, metrics, traces
8. **Security at every layer**: Zero trust approach

### Q32: What are common microservices anti-patterns?
**Answer:**
1. **Distributed Monolith**: Services too tightly coupled
2. **Chatty Services**: Too many inter-service calls
3. **Shared Database**: Multiple services sharing database
4. **Synchronous Communication Everywhere**: No async messaging
5. **Missing Service Boundaries**: Wrong service decomposition
6. **Inadequate Testing**: Not testing failure scenarios
7. **Manual Deployment**: No automation
8. **Ignoring Data Consistency**: No strategy for eventual consistency