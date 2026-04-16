# Microservices Architecture & Patterns

## Table of Contents
1. [Advantages of Microservices](#advantages-of-microservices)
2. [Service Discovery](#service-discovery)
3. [API Gateway](#api-gateway)
4. [Auto-scaling & New Instance Detection](#auto-scaling--new-instance-detection)
5. [Communication Patterns](#communication-patterns)
6. [Resilience Patterns](#resilience-patterns)
7. [Data Management](#data-management)
8. [Saga Pattern](#saga-pattern)
9. [CQRS & Event Sourcing](#cqrs--event-sourcing)
10. [Distributed Tracing & Observability](#distributed-tracing--observability)
11. [Security in Microservices](#security-in-microservices)

---

## Advantages of Microservices

| Advantage | Detail |
|---|---|
| Independent deployment | Each service has its own CI/CD. Ship `OrderService` without touching `UserService`. |
| Independent scaling | Scale only what's under load. `PaymentService` under Black Friday, not everything. |
| Fault isolation | `InventoryService` crash doesn't take down `OrderService` (with circuit breakers). |
| Technology diversity | Python ML service, Java backend, Node.js BFF — choose the right tool per domain. |
| Team autonomy | Conway's Law: org structure → system architecture. One team, one service, one domain. |
| Smaller codebases | Easier to understand, test, and onboard new devs. |

**Trade-offs (be ready to discuss):**
- Distributed systems complexity (network latency, partial failures)
- Eventual consistency — no simple ACID across services
- Operational overhead — N services = N deployments, N log streams, N dashboards
- Data duplication — no foreign keys across service boundaries
- Testing complexity — integration tests are hard

**When NOT to use microservices:** small teams, MVP, tight data consistency needs, limited DevOps maturity.

---

## Service Discovery

Without service discovery, services hardcode each other's IPs — brittle and unmanageable.

```mermaid
flowchart TD
    OS[Order Service] -->|1 - register host:port on start| SR[(Service Registry\nEureka / Consul)]
    PS[Payment Service] -->|1 - register| SR
    IS[Inventory Service] -->|1 - register| SR
    OS -->|2 - lookup Payment Service| SR
    SR -->|3 - return available instances| OS
    OS -->|4 - call with client-side LB| PS
    SR -->|heartbeat check| OS
    SR -->|heartbeat check| PS
```

### Client-Side Discovery (Netflix Eureka)
```java
// application.yml — register with Eureka
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10
    lease-expiration-duration-in-seconds: 30
spring:
  application:
    name: order-service  # service ID in registry

// Caller uses service name, Spring Cloud LB resolves
@LoadBalanced  // adds RestTemplate interceptor for client-side LB
@Bean
RestTemplate restTemplate() { return new RestTemplate(); }

// Usage
restTemplate.getForObject("http://payment-service/api/payments", PaymentDto.class);
```

### Server-Side Discovery (AWS ALB + ECS, Consul + Envoy)
Client calls stable DNS name → Load Balancer queries registry → routes to healthy instance. Client knows nothing about instances.

**Health checks:** each service exposes `/actuator/health`. Registry removes unhealthy instances automatically.

---

## API Gateway

The API Gateway is the **single entry point** for all external traffic.

```mermaid
flowchart TD
    Mobile[Mobile Client]
    Web[Web Client]
    GW[API Gateway\nSpring Cloud Gateway / Kong / AWS API GW]

    Mobile --> GW
    Web --> GW

    GW -->|Route: /api/orders/**| OS[Order Service]
    GW -->|Route: /api/products/**| PS[Product Service]
    GW -->|Route: /api/users/**| US[User Service]
    GW -->|Route: /api/payments/**| PAY[Payment Service]

    GW -.->|Cross-cutting| Auth[JWT Validation]
    GW -.->|Cross-cutting| RL[Rate Limiting]
    GW -.->|Cross-cutting| Log[Logging & Tracing]
```

**Responsibilities:**
- **Routing** — path-based routing to backend services
- **Authentication** — validate JWT/API keys before forwarding
- **Rate limiting** — protect backends from abuse
- **SSL termination** — one certificate to manage at the edge
- **Request/response transformation** — header manipulation, response aggregation
- **Circuit breaking** — don't forward to unhealthy backends
- **Observability** — centralized access logs, distributed trace injection

```yaml
# Spring Cloud Gateway configuration
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service          # lb:// = load-balanced via Eureka
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=0
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 100
                redis-rate-limiter.burstCapacity: 200

      # Auth filter applied globally
      default-filters:
        - TokenRelay=              # forward JWT downstream
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/fallback
```

---

## Auto-scaling & New Instance Detection

**Question: "5 microservices running. We deploy one more instance of `order-service`. How does the system detect it and route traffic without client-side changes?"**

```mermaid
sequenceDiagram
    participant New as New order-service Instance
    participant Eureka as Service Registry (Eureka)
    participant GW as API Gateway
    participant Client

    New->>Eureka: POST /eureka/apps/ORDER-SERVICE {host, port, health}
    Eureka->>Eureka: Add to registry
    GW->>Eureka: Refresh registry (every 30s by default)
    Eureka->>GW: Updated instance list: [instance1, instance2, instance3]
    Client->>GW: GET /api/orders/123
    GW->>New: Route to new instance (load-balanced automatically)
```

**Step by step:**
1. New instance starts → registers with Eureka (`spring.application.name: order-service`)
2. Sends heartbeat every 10s (configurable)
3. API Gateway periodically refreshes its local cache of instances from Eureka (every 30s)
4. Spring Cloud LoadBalancer distributes requests across all healthy instances
5. Client calls same URL — completely unaware of new instance
6. If new instance fails health check → Eureka removes it → Gateway stops routing to it

**With Spring Cloud Gateway + Discovery Locator:**
```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true                   # auto-creates routes for all Eureka services
          lower-case-service-id: true     # /order-service/** routes auto-created
```
Zero manual configuration needed for new services.

---

## Communication Patterns

```mermaid
flowchart LR
    subgraph Synchronous
        A[Service A] -->|HTTP REST| B[Service B]
        A2[Service A] -->|gRPC| B2[Service B]
    end
    subgraph Asynchronous
        C[Service C] -->|publish| K[(Kafka Topic)]
        K -->|consume| D[Service D]
        K -->|consume| E[Service E]
    end
```

| | REST | gRPC | Kafka |
|---|---|---|---|
| Protocol | HTTP/1.1 or HTTP/2 | HTTP/2 + Protobuf | TCP (own protocol) |
| Coupling | Temporal | Temporal | Decoupled |
| Latency | Medium | Low (~3× faster than REST) | Higher (eventual) |
| Schema | OpenAPI (optional) | `.proto` (enforced) | Avro/JSON Schema Registry |
| Streaming | Limited (SSE/WebSocket) | Native bidirectional | Replayed, retained |
| Use case | Public APIs, CRUD | Internal, high-throughput | Events, fan-out, audit |

**Feign Client (declarative REST):**
```java
@FeignClient(name = "payment-service", fallback = PaymentFallback.class)
public interface PaymentClient {
    @PostMapping("/api/payments")
    PaymentResponse charge(@RequestBody PaymentRequest request);
}

@Component
class PaymentFallback implements PaymentClient {
    @Override
    public PaymentResponse charge(PaymentRequest request) {
        return PaymentResponse.pending("Payment service unavailable");
    }
}
```

---

## Resilience Patterns

### Circuit Breaker

Prevents cascade failures by stopping calls to a failing service.

```mermaid
stateDiagram-v2
    [*] --> Closed: Normal operation
    Closed --> Open: Failure threshold exceeded\n(e.g., 50% failures in 10s)
    Open --> HalfOpen: After wait duration (e.g., 60s)
    HalfOpen --> Closed: Probe request succeeds
    HalfOpen --> Open: Probe request fails
    Open --> Open: All calls fail fast\n(fallback executed)
```

```java
// application.yml (Resilience4j)
resilience4j:
  circuitbreaker:
    instances:
      payment-service:
        sliding-window-size: 10
        minimum-number-of-calls: 5
        failure-rate-threshold: 50
        wait-duration-in-open-state: 60s
        permitted-number-of-calls-in-half-open-state: 3

// Usage
@CircuitBreaker(name = "payment-service", fallbackMethod = "paymentFallback")
@Retry(name = "payment-service", fallbackMethod = "paymentFallback")
@TimeLimiter(name = "payment-service")
public CompletableFuture<PaymentResponse> charge(PaymentRequest request) {
    return CompletableFuture.supplyAsync(() -> paymentClient.charge(request));
}

private CompletableFuture<PaymentResponse> paymentFallback(PaymentRequest req, Throwable t) {
    log.warn("Payment service unavailable, using fallback", t);
    return CompletableFuture.completedFuture(PaymentResponse.queued());
}
```

### Bulkhead

Isolate failures by limiting concurrent calls per dependency.
```java
@Bulkhead(name = "payment-service", type = Bulkhead.Type.THREADPOOL)
public CompletableFuture<PaymentResponse> charge(PaymentRequest req) { ... }
```

### Retry with Exponential Backoff + Jitter
```java
@Retry(name = "external-api")
// application.yml:
# max-attempts: 3
# wait-duration: 500ms
# exponential-backoff-multiplier: 2   → waits 500ms, 1000ms, 2000ms
# randomized-wait-factor: 0.5         → adds jitter (prevents thundering herd)
```

### Timeout
Always set timeouts. A service waiting forever degrades the whole system.
```java
// Feign client timeout
feign:
  client:
    config:
      payment-service:
        connect-timeout: 1000
        read-timeout: 5000
```

---

## Data Management

### Database per Service

```mermaid
flowchart TD
    OS[Order Service] --> ODB[(PostgreSQL\norders, order_items)]
    PS[Product Service] --> PDB[(MongoDB\nproducts, catalog)]
    US[User Service] --> UDB[(MySQL\nusers, profiles)]
    CS[Cart Service] --> CDB[(Redis\ncart data)]
```

**Why separate DBs:**
- Services evolve their schema independently
- Technology choice per use case
- No coupling through shared tables
- Independent scaling (PostgreSQL vs Redis vs Mongo)

**Consequences:**
- No foreign keys across services — reference by ID only
- Cross-service queries require API calls or denormalized read models
- Distributed transactions require Saga (no ACID)

### API Composition for Cross-Service Queries
```java
@Service
public class OrderDashboardService {
    public OrderDashboard getDashboard(Long orderId) {
        // Parallel calls to 3 services
        CompletableFuture<Order> orderFuture = supplyAsync(() -> orderClient.getOrder(orderId));
        CompletableFuture<Customer> custFuture = supplyAsync(() -> customerClient.getCustomer(orderId));
        CompletableFuture<Payment> payFuture = supplyAsync(() -> paymentClient.getPayment(orderId));

        return CompletableFuture.allOf(orderFuture, custFuture, payFuture)
            .thenApply(v -> new OrderDashboard(orderFuture.join(), custFuture.join(), payFuture.join()))
            .join();
    }
}
```

---

## Saga Pattern

Manages distributed transactions across multiple services **without a 2-phase commit lock**.

Each step has a **compensating transaction** (the "undo" operation).

### Choreography (event-driven, decentralized)

```mermaid
sequenceDiagram
    participant OS as Order Service
    participant Kafka
    participant PS as Payment Service
    participant IS as Inventory Service

    OS->>OS: Save order (PENDING)
    OS->>Kafka: OrderCreated
    PS->>PS: Charge payment
    PS->>Kafka: PaymentProcessed
    IS->>IS: Reserve inventory
    IS->>Kafka: InventoryReserved
    OS->>OS: Update order (CONFIRMED)

    Note over OS,IS: On failure: publish compensation events
    IS->>Kafka: InventoryFailed
    PS->>Kafka: PaymentRefunded (compensation)
    OS->>OS: Update order (FAILED)
```

### Orchestration (centralized coordinator)

```mermaid
sequenceDiagram
    participant Saga as Saga Orchestrator
    participant PS as Payment Service
    participant IS as Inventory Service
    participant NS as Notification Service

    Saga->>PS: ProcessPayment
    PS-->>Saga: PaymentSuccess
    Saga->>IS: ReserveInventory
    IS-->>Saga: InventoryFailed
    Saga->>PS: RefundPayment (compensate)
    PS-->>Saga: RefundSuccess
    Saga->>NS: NotifyFailure
```

**Choreography vs Orchestration:**
| | Choreography | Orchestration |
|---|---|---|
| Coupling | Loose — services react to events | Central coordinator |
| Visibility | Hard — flow distributed across services | Easy — single place to debug |
| Complexity | Each service must know compensations | Orchestrator manages complexity |
| Use when | Simple flows, few services | Complex workflows, many steps |

---

## CQRS & Event Sourcing

### CQRS — Command Query Responsibility Segregation

Separate the **write model** (commands) from the **read model** (queries). Different optimizations for each.

```mermaid
flowchart LR
    Client --> GW[API Gateway]
    GW -->|POST commands| WS[Write Service\nOptimized for consistency]
    WS --> WDB[(Write DB\nNormalized PostgreSQL)]
    WDB -->|publish events| MQ[(Kafka)]
    MQ -->|project| RDB[(Read DB\nDenormalized MongoDB / Redis)]
    GW -->|GET queries| RS[Read Service\nOptimized for query speed]
    RS --> RDB
```

**Why:** write model needs strong consistency + validation. Read model needs fast queries for complex joins. They can even be different services with different databases.

### Event Sourcing

Instead of storing current state, store the **full history of events**. State = replay of all events.

```
Events stored:
1. AccountOpened {id: 42, owner: "Alice"}
2. MoneyDeposited {id: 42, amount: 1000}
3. MoneyWithdrawn {id: 42, amount: 200}
4. MoneyDeposited {id: 42, amount: 500}

Current state = replay → balance: 1300
```

```java
// Event store
public class BankAccount {
    private final List<DomainEvent> changes = new ArrayList<>();
    private BigDecimal balance = BigDecimal.ZERO;

    public void deposit(BigDecimal amount) {
        apply(new MoneyDeposited(id, amount)); // apply event
    }

    private void apply(MoneyDeposited event) {
        this.balance = this.balance.add(event.getAmount());
        this.changes.add(event); // store event
    }
}
```

**Benefits:** full audit log, time-travel (reconstruct state at any point), multiple read models from same event stream.  
**Costs:** complexity, eventual consistency, event schema evolution.

---

## Distributed Tracing & Observability

**Three pillars:**
- **Metrics** — aggregated numbers (Prometheus + Grafana)
- **Logs** — timestamped events (ELK stack, CloudWatch)
- **Traces** — full request journey across services (Jaeger, Zipkin, AWS X-Ray)

```mermaid
flowchart LR
    Client -->|traceId: abc123| GW[API Gateway]
    GW -->|traceId: abc123| OS[Order Service]
    OS -->|traceId: abc123| PS[Payment Service]
    OS -->|traceId: abc123| IS[Inventory Service]
    OS -->|traceId: abc123| NS[Notification Service]
    OS --> Zipkin[(Zipkin / Jaeger\nTrace aggregation)]
    PS --> Zipkin
    IS --> Zipkin
```

```yaml
# application.yml — Micrometer + Zipkin
management:
  tracing:
    sampling:
      probability: 1.0    # 100% in dev, 10% in prod
  zipkin:
    tracing:
      endpoint: http://zipkin:9411/api/v2/spans
```

**Key metrics to monitor:** request rate, error rate, latency percentiles (p50/p95/p99), saturation (queue depth, CPU), resource utilization.

**Structured logging:**
```java
log.info("Order created",
    StructuredArguments.kv("orderId", order.getId()),
    StructuredArguments.kv("customerId", order.getCustomerId()),
    StructuredArguments.kv("total", order.getTotal()));
// JSON output: {"level":"INFO","orderId":42,"customerId":7,"total":"150.00",...}
```

---

## Security in Microservices

```mermaid
flowchart LR
    Client -->|HTTPS + JWT| GW[API Gateway\nValidate JWT\nRate limit]
    GW -->|Forward JWT\nor service token| OS[Order Service]
    OS -->|mTLS| PS[Payment Service]
    OS -->|mTLS| IS[Inventory Service]
    GW -.->|Validate with| AS[Auth Server\nKeycloak / Cognito]
```

**Layers:**
1. **Edge (API Gateway):** TLS termination, JWT validation, rate limiting, WAF
2. **Service-to-service (internal):** mTLS (mutual TLS) or service mesh (Istio)
3. **Application:** `@PreAuthorize` + scope validation
4. **Data:** field-level encryption for sensitive data (PII, PAN)

**Secrets management:** never hardcode. Use AWS Secrets Manager, HashiCorp Vault, or Kubernetes Secrets. Rotate regularly.
