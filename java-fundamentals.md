# Java Fundamentals

## Table of Contents
1. [Tell Me About Yourself](#tell-me-about-yourself)
2. [OOP — 4 Pillars](#oop--4-pillars)
3. [Polymorphism](#polymorphism)
4. [Dynamic Polymorphism & Advantages](#dynamic-polymorphism--advantages)
5. [Loose Coupling](#loose-coupling)
6. [Multiple Inheritance — Why Java Doesn't Support](#multiple-inheritance--why-java-doesnt-support)
7. [Abstract Class vs Interface](#abstract-class-vs-interface)
8. [ArrayList vs LinkedList](#arraylist-vs-linkedlist)
9. [HashSet vs TreeSet](#hashset-vs-treeset)
10. [HashMap Internals](#hashmap-internals)
11. [Checked vs Unchecked Exception](#checked-vs-unchecked-exception)
12. [synchronized](#synchronized)
13. [CountDownLatch](#countdownlatch)
14. [map() vs flatMap()](#map-vs-flatmap)
15. [Two Interfaces Same Method](#two-interfaces-same-method)
16. [Default Method Conflict](#default-method-conflict)
17. [Java 8+ Features](#java-8-features)
18. [equals() & hashCode() Contract](#equals--hashcode-contract)
19. [Immutable Classes](#immutable-classes)
20. [JVM Memory & GC](#jvm-memory--gc)

---

## Tell Me About Yourself

**What interviewers want:** Concise story arc. Tech stack → seniority → impact → what you're looking for.

**Template:**
> "I'm a senior software engineer with X years of experience, primarily building Java/Spring Boot microservices. At [Company], I led [impactful project], which [result — e.g., reduced latency by 40%, handled 10k RPS]. I'm experienced in distributed systems, cloud (AWS), and [blockchain/Kotlin/etc.]. I'm looking for a role where I can [lead technical decisions / work on challenging distributed problems / contribute to a growing product]."

**Tips:** Keep it under 2 minutes. Tailor to the job. End with why this company/role.

---

## OOP — 4 Pillars

### Encapsulation
Bundle data + behavior; hide internal state via access modifiers. Expose only what's needed.

```java
public class BankAccount {
    private BigDecimal balance; // hidden

    public void deposit(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) throw new IllegalArgumentException();
        this.balance = this.balance.add(amount);
    }

    public BigDecimal getBalance() { return balance; } // controlled read
}
```

### Inheritance
Reuse behavior via `extends`. Establishes "is-a" relationship. Enables code reuse but creates tight coupling — prefer composition.

```java
public abstract class Animal {
    protected String name;
    public abstract String makeSound();
    public String describe() { return name + " says " + makeSound(); }
}

public class Dog extends Animal {
    @Override public String makeSound() { return "Woof"; }
}
```

### Polymorphism
One interface, multiple implementations. Resolved at runtime via dynamic dispatch.

```java
List<Animal> animals = List.of(new Dog(), new Cat(), new Bird());
animals.forEach(a -> System.out.println(a.makeSound())); // each behaves differently
```

### Abstraction
Expose **what** a component does, hide **how**. Achieved via interfaces and abstract classes.

```java
public interface PaymentGateway {
    PaymentResult charge(BigDecimal amount, String token);
    void refund(String transactionId);
}
// Stripe, PayPal, etc. implement this — callers don't know which
```

---

## Polymorphism

Two forms:

**Static (Compile-time) — Method Overloading:** same method name, different parameter signatures.
```java
class Calculator {
    int add(int a, int b)           { return a + b; }
    double add(double a, double b)  { return a + b; }
    int add(int a, int b, int c)    { return a + b + c; }
}
```

**Dynamic (Runtime) — Method Overriding:** subclass provides its own implementation; JVM resolves based on actual object type, not reference type.
```java
Animal animal = new Dog();     // reference type: Animal
animal.makeSound();            // calls Dog.makeSound() — resolved at runtime via vtable
```

**Covariant return types (Java 5+):** overriding method can return a subtype.
```java
class Animal { Animal create() { return new Animal(); } }
class Dog extends Animal { @Override Dog create() { return new Dog(); } } // valid
```

---

## Dynamic Polymorphism & Advantages

**Dynamic polymorphism** = method dispatch at runtime using the **virtual method table (vtable)**. The JVM looks up the actual object's vtable, not the reference type's.

**Advantages:**
- **Open/Closed Principle** — add new types without changing existing code. Add `Parrot extends Animal` and all existing loops/collections work automatically.
- **Testability** — swap implementations: `PaymentGateway gw = new MockGateway()`.
- **Extensibility** — frameworks (Spring, Hibernate) rely on this to inject behavior (proxies, decorators).
- **Code reuse** — shared algorithms work on any subtype.

```java
// Same method works for all current AND future implementations
public void processAll(List<Animal> animals) {
    animals.forEach(Animal::makeSound); // no if/else, no switch
}
```

---

## Loose Coupling

**Tight coupling:** a class directly depends on a concrete implementation. Changing the dependency requires changing the caller.

**Loose coupling:** depend on abstractions (interfaces). The caller doesn't know or care which implementation it gets.

```java
// Tight — hard to test, hard to change
class OrderService {
    private MySQLOrderRepository repo = new MySQLOrderRepository(); // hardwired

    public void place(Order order) {
        repo.save(order);
        new SmtpEmailService().send(order.getEmail(), "Order placed");
    }
}

// Loose — testable, swappable
class OrderService {
    private final OrderRepository repo;       // interface
    private final NotificationService notifier; // interface

    public OrderService(OrderRepository repo, NotificationService notifier) {
        this.repo = repo;
        this.notifier = notifier;
    }

    public void place(Order order) {
        repo.save(order);
        notifier.notify(order.getEmail(), "Order placed");
    }
}
```

**Benefits:** unit test with mocks, swap MySQL for MongoDB without touching `OrderService`, comply with Dependency Inversion Principle.

---

## Multiple Inheritance — Why Java Doesn't Support

Java does **not** allow a class to extend more than one class because of the **Diamond Problem**.

```
      A
    /   \
   B     C      B and C both override A.method()
    \   /
      D          Which version does D inherit?
```

The compiler cannot deterministically decide — this creates ambiguity. Languages like C++ allow it but require explicit disambiguation, which adds complexity and bugs.

**Java's solution:** Classes use single inheritance. **Multiple interface implementation** is allowed because interfaces historically had no state and no method bodies (before Java 8 default methods).

**With Java 8 default methods**, if two interfaces declare the same `default` method, the **implementing class must override it** — the compiler forces disambiguation.

---

## Abstract Class vs Interface

| | Abstract Class | Interface |
|---|---|---|
| Instance fields | Yes | No (constants only: `public static final`) |
| Constructor | Yes | No |
| Method types | Abstract + concrete | Abstract + `default` + `static` + `private` (Java 9+) |
| Inheritance | Single (`extends`) | Multiple (`implements`) |
| Access modifiers | Any | `public` by default |
| `extends`/`implements` | Class extends 1 abstract class | Class implements N interfaces |
| When to use | Shared state + partial implementation | Contract / capability |

```java
// Abstract class — share state and partial implementation
abstract class BaseRepository<T, ID> {
    protected final EntityManager em;

    BaseRepository(EntityManager em) { this.em = em; }

    public void save(T entity) { em.persist(entity); } // concrete

    public abstract T findById(ID id);                  // subclass implements
}

// Interface — define a contract
interface Auditable {
    LocalDateTime getCreatedAt();
    LocalDateTime getUpdatedAt();
    default String auditSummary() {
        return "Created: " + getCreatedAt() + ", Updated: " + getUpdatedAt();
    }
}
```

**Rule of thumb:** Start with an interface. Use abstract class only when you need shared state or a non-trivial partial implementation.

---

## ArrayList vs LinkedList

| Operation | ArrayList | LinkedList |
|---|---|---|
| `get(index)` | **O(1)** — direct array offset | O(n) — traverse from head |
| `add(end)` | O(1) amortized — occasional resize | **O(1)** — append to tail |
| `add(middle)` | O(n) — shifts elements right | O(n) to find + **O(1)** to link |
| `remove(middle)` | O(n) — shifts left | O(n) to find + **O(1)** to unlink |
| Memory | Compact — contiguous array | Extra overhead: 2 pointers per node (~48 bytes vs 16) |
| Cache performance | **Excellent** — CPU cache-friendly | Poor — nodes scattered in heap |
| Iterator | Fast | Fast |

**Internal:** `ArrayList` backs a `Object[]` array. When full, creates a new array 1.5× larger and copies. Default capacity: 10.

`LinkedList` is a doubly-linked list. Also implements `Deque` — useful as a queue/stack.

**Default choice: `ArrayList`**. Use `LinkedList` only when you have frequent insertions/deletions at both ends (queue/deque behavior) and no random access.

```java
// When LinkedList makes sense
Deque<Task> taskQueue = new LinkedList<>();
taskQueue.offerFirst(urgentTask);   // O(1) prepend
taskQueue.pollLast();               // O(1) remove from end
```

---

## HashSet vs TreeSet

| | HashSet | TreeSet |
|---|---|---|
| Backed by | `HashMap` | `TreeMap` (Red-Black Tree) |
| Order | None (insertion order not preserved) | **Natural order** or `Comparator` |
| `contains`, `add`, `remove` | **O(1)** average | O(log n) |
| Null | Allows **one** null | Throws `NullPointerException` |
| `first()`, `last()`, `range queries` | Not supported | **Supported** |
| Use when | Fast lookup, no order needed | Sorted iteration, range queries |

**LinkedHashSet** is a middle ground: O(1) operations + **insertion-order** iteration.

```java
// HashSet
Set<String> fastLookup = new HashSet<>(List.of("apple", "banana", "cherry"));
fastLookup.contains("banana"); // O(1)

// TreeSet — sorted
NavigableSet<Integer> sorted = new TreeSet<>(List.of(5, 2, 8, 1));
sorted.first();                // 1
sorted.headSet(5);             // [1, 2]
sorted.subSet(2, 8);           // [2, 5]
```

---

## HashMap Internals

**Structure:** Array of `Node<K,V>[]` (buckets). Each bucket is either a linked list or a Red-Black Tree.

**`put(key, value)` flow:**
1. Compute `hash = key.hashCode()` → mix bits: `(h ^ (h >>> 16))`
2. `index = hash & (capacity - 1)` — bucket index
3. If bucket empty → insert `Node`
4. If bucket has entries → check each with `equals()`
   - Key found → update value
   - Not found → append to chain
5. If chain length ≥ **8** AND capacity ≥ 64 → convert to **Red-Black Tree** (treeify)
6. If chain length ≤ **6** after removal → convert back to linked list (untreeify)
7. If `size > capacity × loadFactor (0.75)` → **resize** (double capacity, rehash)

**Default capacity:** 16. **Load factor:** 0.75 (balances time vs space).

```java
// The contract you MUST follow:
// If a.equals(b) then a.hashCode() == b.hashCode()   → REQUIRED
// If a.hashCode() == b.hashCode() then a.equals(b)   → NOT required (collision OK)

// Example: broken contract causes lost keys
class Point {
    int x, y;
    @Override public boolean equals(Object o) {
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }
    // Missing hashCode! equals says (1,1) == (1,1) but they end up in different buckets
}

// Correct:
@Override public int hashCode() { return Objects.hash(x, y); }
```

**`ConcurrentHashMap`** — thread-safe. Uses segment-level locking (Java 7) or CAS + bin-level synchronization (Java 8+). Does not allow null keys/values.

---

## Checked vs Unchecked Exception

| | Checked | Unchecked |
|---|---|---|
| Extends | `Exception` | `RuntimeException` (also extends `Exception`) |
| Compiler enforces | **Yes** — must catch or declare `throws` | No |
| Represents | External failure (recoverable) | Programming bug (usually unrecoverable) |
| Examples | `IOException`, `SQLException`, `ParseException` | `NullPointerException`, `ArrayIndexOutOfBoundsException`, `IllegalArgumentException` |

```java
// Checked — caller must handle
public String readFile(Path path) throws IOException {
    return Files.readString(path); // IOException is checked
}

// Unchecked — caller's choice to handle
public void setAge(int age) {
    if (age < 0) throw new IllegalArgumentException("Age cannot be negative: " + age);
}
```

**Best practices:**
- Use checked exceptions for recoverable conditions (file not found → ask for another path)
- Use unchecked for programming errors (null reference, invalid argument)
- Wrap checked in custom runtime exceptions in service layers (avoids `throws` pollution)
- Never catch `Exception` or `Throwable` silently

**try-with-resources (Java 7+):** automatically closes `AutoCloseable` resources.
```java
try (InputStream is = new FileInputStream("file.txt");
     BufferedReader reader = new BufferedReader(new InputStreamReader(is))) {
    return reader.readLine();
} // is and reader closed even if exception thrown
```

---

## synchronized

`synchronized` acquires an **intrinsic lock (monitor)** on an object. Only one thread can hold it at a time — all others **block** (enter BLOCKED state).

```java
class Counter {
    private int count = 0;

    // Method-level: lock on 'this'
    public synchronized void increment() { count++; }

    // Block-level: explicit lock object (better granularity)
    private final Object lock = new Object();
    public void decrement() {
        synchronized (lock) { count--; }
    }

    // Static synchronized: lock on Class object
    public static synchronized void staticOp() { ... }
}
```

**How `count++` without sync fails:**
1. Thread A reads `count` = 5 into register
2. Thread B reads `count` = 5 into register
3. Thread A writes `count` = 6
4. Thread B writes `count` = 6  ← lost update! Expected: 7

**synchronized vs Lock:**
- `synchronized` is simpler; lock released automatically
- `ReentrantLock` supports `tryLock(timeout)`, `lockInterruptibly()`, `ReadWriteLock`

```java
ReentrantLock lock = new ReentrantLock();
if (lock.tryLock(100, TimeUnit.MILLISECONDS)) {
    try { doWork(); }
    finally { lock.unlock(); } // ALWAYS release in finally
}
```

---

## CountDownLatch

`CountDownLatch` blocks one or more threads until a count reaches zero. **One-time use** (cannot be reset).

```java
int workerCount = 5;
CountDownLatch latch = new CountDownLatch(workerCount);
ExecutorService executor = Executors.newFixedThreadPool(workerCount);

for (int i = 0; i < workerCount; i++) {
    executor.submit(() -> {
        try {
            performWork(); // each worker does its job
        } finally {
            latch.countDown(); // decrement regardless of outcome
        }
    });
}

latch.await(30, TimeUnit.SECONDS); // main thread waits (with timeout)
System.out.println("All workers finished");
executor.shutdown();
```

**Use cases:** wait for all microservice dependencies to warm up, parallelize initialization tasks, test synchronization barriers.

**`CyclicBarrier`** — reusable; all threads wait for each other at a checkpoint. Good for iterative algorithms (phases).

```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> System.out.println("Phase complete"));
// Each thread calls barrier.await() — all 3 must arrive before any continues
```

---

## map() vs flatMap()

**`map`:** 1-to-1 transformation. Each element → one result. Returns `Stream<R>`.

**`flatMap`:** 1-to-many + **flatten**. Each element → a Stream. All streams concatenated into one. Returns `Stream<R>`.

```java
List<String> sentences = List.of("Hello World", "Java Streams");

// map → Stream<String[]>  (stream of arrays, NOT what we want to count words)
sentences.stream().map(s -> s.split(" ")); // Stream<String[]>

// flatMap → Stream<String> (each array flattened into individual words)
long wordCount = sentences.stream()
    .flatMap(s -> Arrays.stream(s.split(" ")))
    .count(); // 4

// Another example: flatten nested lists
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3, 4), List.of(5));
List<Integer> flat = nested.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList()); // [1, 2, 3, 4, 5]

// Optional.flatMap: avoid nested Optional<Optional<T>>
Optional<String> result = Optional.of("user")
    .flatMap(u -> findUser(u))      // returns Optional<User>
    .flatMap(User::getEmail);       // returns Optional<String>
```

**Rule:** if the mapping function returns `Collection<T>` or `Stream<T>` and you want a flat `Stream<T>`, use `flatMap`.

---

## Two Interfaces Same Method

### If the method is abstract (no default implementation):

```java
interface Flyable { void move(); }
interface Swimmable { void move(); }

class Duck implements Flyable, Swimmable {
    @Override
    public void move() {
        // Only ONE implementation needed — no conflict for abstract methods
        System.out.println("Duck moves");
    }
}
```

No ambiguity — the class simply provides one implementation that satisfies both interfaces.

---

## Default Method Conflict

### If both interfaces provide a `default` implementation:

```java
interface A { default void test() { System.out.println("A"); } }
interface B { default void test() { System.out.println("B"); } }

// Compile error if you don't override:
class C implements A, B {
    @Override
    public void test() {
        A.super.test(); // explicitly choose A's version
        // or B.super.test(); or your own logic
    }
}
```

**Rules Java follows (priority order):**
1. **Class wins** — a concrete method in the class always wins over interface defaults
2. **More specific interface wins** — if interface B extends A and both define the default, B's wins
3. **Must explicitly override** — if neither rule resolves it, compiler error forces you to disambiguate

---

## Java 8+ Features

### Streams
```java
List<Order> highValue = orders.stream()
    .filter(o -> o.getTotal().compareTo(new BigDecimal("1000")) > 0)
    .sorted(Comparator.comparing(Order::getTotal).reversed())
    .limit(10)
    .collect(Collectors.toList());

// Collectors
Map<OrderStatus, List<Order>> byStatus = orders.stream()
    .collect(Collectors.groupingBy(Order::getStatus));

Map<OrderStatus, Long> countByStatus = orders.stream()
    .collect(Collectors.groupingBy(Order::getStatus, Collectors.counting()));
```

### Optional — avoid NullPointerException
```java
Optional<User> user = userRepository.findById(id);
String email = user
    .filter(u -> u.isActive())
    .map(User::getEmail)
    .orElseThrow(() -> new UserNotFoundException(id));
```

### Method References
```java
List<String> names = users.stream()
    .map(User::getName)          // instance method reference
    .sorted(String::compareTo)   // instance method reference
    .collect(Collectors.toList());
```

### Records (Java 16+)
```java
public record Point(double x, double y) {
    // Compact constructor for validation
    public Point {
        if (x < 0 || y < 0) throw new IllegalArgumentException("Negative coordinates");
    }
}
// Automatic: equals, hashCode, toString, getters (x(), y()), constructor
```

### Sealed Classes (Java 17+)
```java
public sealed interface Shape permits Circle, Rectangle, Triangle {}
public record Circle(double radius) implements Shape {}
public record Rectangle(double w, double h) implements Shape {}

// Pattern matching exhaustive switch
double area = switch (shape) {
    case Circle c    -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.w() * r.h();
    case Triangle t  -> t.base() * t.height() / 2;
}; // no default needed — compiler knows all cases
```

### Text Blocks (Java 15+)
```java
String json = """
        {
            "name": "Alice",
            "age": 30
        }
        """;
```

---

## equals() & hashCode() Contract

**Contract:**
- If `a.equals(b)` → `a.hashCode() == b.hashCode()` **(REQUIRED)**
- If `a.hashCode() == b.hashCode()` → `a.equals(b)` **(NOT required — collisions OK)**
- `equals` must be: reflexive, symmetric, transitive, consistent, and `x.equals(null)` = false

```java
public class Order {
    private final Long id;
    private final String customerEmail;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Order)) return false;
        Order other = (Order) o;
        return Objects.equals(id, other.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id); // same fields as equals
    }
}
```

**Common mistake:** override `equals` but not `hashCode` → objects equal by `equals` end up in different HashMap buckets → **can't be found in HashMap/HashSet**.

---

## Immutable Classes

An immutable class cannot be changed after construction. Thread-safe by nature.

```java
public final class Money {             // 1. final class (no subclassing)
    private final Currency currency;   // 2. private final fields
    private final BigDecimal amount;

    public Money(Currency currency, BigDecimal amount) {
        this.currency = currency;
        this.amount = amount.setScale(2, RoundingMode.HALF_UP);
    }

    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) throw new CurrencyMismatchException();
        return new Money(currency, this.amount.add(other.amount)); // 3. return new instance
    }

    // 4. No setters
    // 5. Defensive copy for mutable fields
    public BigDecimal getAmount() { return amount; } // BigDecimal is immutable — OK
}
```

**Rules for immutability:**
1. `final` class (prevent subclass mutation)
2. `private final` fields
3. No setters
4. Initialize all fields in constructor
5. Defensive copy for mutable fields (arrays, dates, collections)
6. Return new instances from "modification" methods

---

## JVM Memory & GC

**JVM Memory Areas:**
```
┌─────────────────────────────────────────────────────┐
│ Heap                                                │
│  ┌──────────────────┐  ┌──────────────────────────┐ │
│  │   Young Gen      │  │       Old Gen             │ │
│  │  Eden│S0│S1      │  │  Long-lived objects       │ │
│  └──────────────────┘  └──────────────────────────┘ │
├─────────────────────────────────────────────────────┤
│ Metaspace (Java 8+) — class metadata, method area  │
├─────────────────────────────────────────────────────┤
│ JVM Stack (per thread) — frames, local vars         │
├─────────────────────────────────────────────────────┤
│ Native Stack, PC Register (per thread)              │
└─────────────────────────────────────────────────────┘
```

**GC Types:**
- **Minor GC** — collects Young Gen (Eden + Survivors). Fast. Frequent.
- **Major GC** — collects Old Gen. Slower.
- **Full GC** — both. Stop-the-world pause. Avoid in production.

**Modern Collectors:**
| Collector | Goal | Use case |
|---|---|---|
| G1 (default Java 9+) | Balanced throughput + latency | General purpose |
| ZGC (Java 15+ production) | < 1ms pauses | Low-latency services |
| Shenandoah | Concurrent compaction | Similar to ZGC |
| Parallel GC | Max throughput | Batch processing |

**Object lifecycle:** Eden → S0/S1 (surviving minor GC → age++) → Old Gen (age > threshold, default 15).

**`-Xms` / `-Xmx`** — initial/max heap. Always set equal in production to avoid resizing pauses.
