# Design Patterns & SOLID

## Table of Contents
1. [SOLID Principles](#solid-principles)
2. [Creational Patterns](#creational-patterns)
3. [Structural Patterns](#structural-patterns)
4. [Behavioral Patterns](#behavioral-patterns)
5. [Enterprise Patterns](#enterprise-patterns)

---

## SOLID Principles

| Principle | Rule | Violation sign |
|---|---|---|
| **S**RP | One class, one reason to change | Class handles 5 different concerns |
| **O**CP | Extend via new code, not modifying old | Every new feature needs editing the same class |
| **L**SP | Subtypes work wherever the base type is used | Override throws `UnsupportedOperationException` |
| **I**SP | Small, focused interfaces | Interface forces implementing unused methods |
| **D**IP | Depend on abstractions, not concretions | `new ConcreteRepo()` inside a service class |

```java
// DIP example
// Bad
class OrderService { private MySQLOrderRepo repo = new MySQLOrderRepo(); }

// Good
class OrderService {
    private final OrderRepository repo;           // interface
    OrderService(OrderRepository repo) { this.repo = repo; }
}
```

---

## Creational Patterns

### Singleton

```java
public class Config {
    private static volatile Config instance;
    private Config() {}

    public static Config getInstance() {
        if (instance == null) {
            synchronized (Config.class) {
                if (instance == null) instance = new Config(); // double-checked locking
            }
        }
        return instance;
    }
}
// In Spring: @Bean is singleton scope by default — use that instead
```

### Builder

```java
Order order = Order.builder()
    .customerId(42L)
    .items(List.of(item1, item2))
    .status(PENDING)
    .build();
```

Use when constructors have many optional parameters. Lombok `@Builder` generates this automatically.

### Factory Method

```java
interface NotificationService { void send(String msg); }
class EmailService implements NotificationService { ... }
class SmsService implements NotificationService { ... }

class NotificationFactory {
    static NotificationService create(String type) {
        return switch (type) {
            case "EMAIL" -> new EmailService();
            case "SMS"   -> new SmsService();
            default -> throw new IllegalArgumentException(type);
        };
    }
}
```

---

## Structural Patterns

### Adapter

Wraps an incompatible interface so it works with existing code.

```java
// External library returns LegacyAddress, but our system uses Address
class AddressAdapter implements Address {
    private final LegacyAddress legacy;
    AddressAdapter(LegacyAddress l) { this.legacy = l; }

    @Override public String getCity() { return legacy.getCityName(); }
}
```

### Decorator

Add behavior dynamically without subclassing.

```java
// Base
interface Logger { void log(String msg); }

// Decorator: adds timestamps
class TimestampLogger implements Logger {
    private final Logger delegate;
    TimestampLogger(Logger delegate) { this.delegate = delegate; }

    @Override public void log(String msg) {
        delegate.log(Instant.now() + " " + msg);
    }
}
```

Spring's `@Transactional`, `@Cacheable`, and AOP proxies are decorators at the framework level.

### Proxy

Controls access to an object. Spring uses JDK dynamic proxies or CGLIB for AOP and `@Transactional`.

---

## Behavioral Patterns

### Strategy

Define a family of algorithms, make them interchangeable.

```java
@FunctionalInterface
interface SortStrategy { void sort(List<Integer> data); }

class SortService {
    private SortStrategy strategy;
    void setStrategy(SortStrategy s) { this.strategy = s; }
    void sort(List<Integer> data) { strategy.sort(data); }
}

// Usage
sortService.setStrategy(Collections::sort);
sortService.setStrategy(data -> data.sort(Comparator.reverseOrder()));
```

### Observer

```java
// Java built-in: ApplicationEvent / ApplicationListener in Spring
@Component
class OrderCreatedListener {
    @EventListener
    public void handle(OrderCreatedEvent event) {
        emailService.sendConfirmation(event.getOrder());
    }
}

// Publishing
applicationEventPublisher.publishEvent(new OrderCreatedEvent(order));
```

### Command

Encapsulate a request as an object — enables undo, queuing, logging.

```java
interface Command { void execute(); }

record TransferCommand(Account from, Account to, BigDecimal amount) implements Command {
    public void execute() { from.debit(amount); to.credit(amount); }
}

// Queue commands, execute later, or undo
```

---

## Enterprise Patterns

### Repository

Abstracts the data layer. Domain code talks to an interface; infrastructure provides the implementation.

```java
public interface OrderRepository {
    Order findById(Long id);
    List<Order> findByStatus(OrderStatus status);
    Order save(Order order);
}
```

### Unit of Work

Tracks all changes during a transaction and commits them together (JPA `EntityManager` implements this pattern).

### Specification

Encapsulate query criteria as reusable objects.

```java
Specification<Order> pending = (root, query, cb) -> cb.equal(root.get("status"), PENDING);
Specification<Order> highValue = (root, query, cb) -> cb.gt(root.get("total"), 1000);

orderRepo.findAll(pending.and(highValue));
```
