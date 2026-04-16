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

The four pillars are the foundation of object-oriented design. Encapsulation protects internal state from invalid modification. Inheritance shares common behavior through hierarchies (use sparingly — prefer composition). Polymorphism allows the same interface to drive different behaviors at runtime, making code extensible without modification. Abstraction hides implementation details behind contracts, so callers depend on *what* a class does, not *how*. Together, they enable code that's modular, testable, and open for extension.

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

Polymorphism means "many forms" — the same operation behaves differently based on the actual type. Static polymorphism (overloading) is resolved at compile time based on the declared parameter types. Dynamic polymorphism (overriding) is resolved at runtime based on the actual object type — this is what enables the Open/Closed Principle: you can add new behavior (new subclass) without changing existing code that uses the base reference.

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

Coupling measures how much one class knows about another. Tight coupling creates a chain reaction: changing one class forces changes in all dependent classes. Loose coupling breaks this by introducing an abstraction (interface) between collaborators — the caller only knows the contract, not the implementation. This is what makes code testable (inject a mock), swappable (change DB without touching business logic), and independently deployable (microservices communicate via API contracts).

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

Multiple inheritance of classes was deliberately omitted from Java to avoid the **Diamond Problem** — an inherent ambiguity that arises when a class inherits from two parents that both define the same method. C++ allows it but requires explicit disambiguation, which adds cognitive overhead and bugs. Java's solution is clean: a class inherits implementation from *one* parent (via `extends`), but can fulfill multiple *contracts* (via `implements`). Java 8 default methods reopened a limited form of multiple inheritance for behavior, with the rule that the implementing class must explicitly resolve conflicts.

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

Use an **abstract class** when you have shared state (instance fields) or a partial implementation that subclasses should inherit and extend. Use an **interface** when you want to define a capability contract that unrelated classes can implement — a class can implement multiple interfaces but extend only one abstract class. Since Java 8, interfaces can have `default` methods (implementations), blurring the line; the key remaining differences are: interfaces can't have instance state, and a class can only extend one class but implement many interfaces.

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

The answer to "which is faster?" depends entirely on the operation. `ArrayList` is backed by a contiguous array — index access (`get(i)`) is O(1) and CPU cache-friendly because elements are adjacent in memory. `LinkedList` is a doubly-linked list — each node holds data plus two pointers, scattered across the heap. Despite theoretical O(1) insertion at both ends, `LinkedList` is almost always slower in practice due to poor cache locality and pointer-chasing overhead. **Default choice is always `ArrayList`** — only reach for `LinkedList` when you truly need constant-time queue/deque operations at both ends.

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

Both implement `Set` (no duplicates), but with very different trade-offs. `HashSet` uses a `HashMap` internally — O(1) average for add/contains/remove, but no ordering. `TreeSet` uses a Red-Black Tree (`TreeMap`) — O(log n) for all operations but maintains natural sort order, enabling range queries (`headSet`, `tailSet`, `subSet`). A third option, `LinkedHashSet`, provides O(1) operations with **insertion-order iteration** — useful when you need a fast, ordered-by-insertion unique collection. Choose based on whether you need ordering and what kind.

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

HashMap uses a hash table — an array where each slot (bucket) is identified by `hash(key) % capacity`. Collisions (multiple keys mapping to the same bucket) are handled initially with a linked list. When a bucket grows beyond 8 entries, it's converted to a Red-Black Tree for O(log N) worst-case instead of O(N). The load factor (default 0.75) controls when the array is resized — at 75% capacity, the map doubles in size and rehashes all entries. Understanding this helps explain why equals()/hashCode() contracts are critical: a broken hashCode causes keys to land in the wrong bucket, making them unfindable.

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

Java's exception model distinguishes between failures the caller should anticipate and recover from (checked), and programming bugs that represent incorrect usage (unchecked). Checked exceptions force the caller to make an explicit decision — either handle it or propagate it. Unchecked exceptions (RuntimeException subclasses) are not forced on callers because they typically represent bugs that can't be meaningfully handled at the call site. Modern Java practice tends toward unchecked exceptions for most business logic, wrapping checked exceptions when they cross abstraction boundaries.

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

`synchronized` is Java's built-in mutual exclusion mechanism. It guarantees two things: **atomicity** (the block executes as an indivisible unit) and **visibility** (all writes made inside the block are visible to other threads that subsequently acquire the same lock). It's simple to use but inflexible — you can't time out, you can't be interrupted while waiting, and there's no way to have separate read and write locks. For those capabilities, use `java.util.concurrent.locks.ReentrantLock` or `ReadWriteLock`.

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

`CountDownLatch` is a synchronization barrier that lets a thread (typically main) wait until a set of other threads complete their work. Think of it as a countdown timer: initialize it with N, each worker calls `countDown()` when done, and the waiting thread blocks on `await()` until the count hits zero. It's **one-time use** — the count can't be reset. For a reusable barrier where N threads all wait for each other at the same point repeatedly (like parallel processing phases), use `CyclicBarrier` instead.

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

`map` transforms each element to exactly one result — it's a 1-to-1 mapping. `flatMap` transforms each element into a *collection of results* and then **flattens** all those collections into a single stream — it's 1-to-many followed by concatenation. The clearest way to remember: if your mapping function returns `Stream<T>`, `Optional<T>`, or `List<T>` and you don't want a `Stream<Stream<T>>` (nested structure), use `flatMap`. The same concept applies to `Optional.flatMap` (avoids `Optional<Optional<T>>`) and reactor's `flatMap` for async composition.

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

When a class implements two interfaces that declare the same method signature, the outcome depends on whether the methods are abstract or provide `default` implementations. For abstract methods, there's no conflict — the class simply provides one implementation that satisfies both interface contracts simultaneously. For `default` methods, there *is* a conflict because the class would otherwise inherit two implementations; Java forces you to explicitly override and choose (or combine) them. This demonstrates Java's approach to the Diamond Problem for interfaces.

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

Java's default method conflict resolution follows three priority rules, applied in order. First: a concrete class method always wins over any interface default (the class has spoken). Second: a more specific interface wins over a more general one (specialization takes precedence). Third: if neither rule resolves the conflict, the compiler forces you to override the method and choose explicitly using `InterfaceName.super.method()` syntax. This makes the programmer's intent explicit and avoids silent ambiguity.

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

Java 8 was a landmark release that brought functional programming idioms to Java. **Streams** enable declarative data processing pipelines — filter, map, collect — without mutable state or explicit loops. **Lambdas** make anonymous functions first-class values, enabling concise callback syntax. **Optional** provides a type-safe way to express "possibly absent" values, replacing null returns. Later versions added **Records** (Java 16) for concise immutable data carriers, **Sealed Classes** (Java 17) for exhaustive type hierarchies with pattern matching, and **Text Blocks** (Java 15) for multiline strings. Understanding which version introduced what is commonly tested in senior interviews.

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

The `equals()`/`hashCode()` contract is one of the most practically important Java contracts to understand. These two methods must be consistent: if `a.equals(b)` returns true, then `a.hashCode()` **must** equal `b.hashCode()`. The reverse is not required (collisions are acceptable). Violating this contract silently breaks `HashMap`, `HashSet`, and `Hashtable` — objects that are "equal" can't be found because they land in different buckets. The classic bug: override `equals` (to define domain equality) but forget `hashCode` — your objects appear equal by `equals` but HashMap treats them as different keys.

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

Immutable objects are inherently thread-safe because no thread can modify their state — eliminating the entire class of bugs related to shared mutable state. They also make great keys in `HashMap`/`HashSet` (their hash code never changes after insertion), and are safe to share across threads or cache without defensive copying. Java's `String`, `Integer`, `BigDecimal`, `LocalDate`, and all records are immutable. The recipe is consistent: final class, private final fields, no setters, defensive copies of any mutable inputs/outputs, and "modification" methods that return new instances.

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

Understanding JVM memory helps diagnose production issues. `OutOfMemoryError: Java heap space` → heap is full, likely a memory leak or undersized `-Xmx`. `OutOfMemoryError: Metaspace` → too many classes loaded (check for classloader leaks, dynamic proxy generation). `StackOverflowError` → infinite recursion (too many stack frames). GC pauses are the other key concern: long Stop-the-World pauses cause timeouts and latency spikes. G1 is the default from Java 9 and handles most workloads well. ZGC is the choice for latency-critical services (p99 latency under 1ms even with large heaps).

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
