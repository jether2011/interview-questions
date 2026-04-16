# Java Fundamentals

## Table of Contents
1. [OOP Pillars](#oop-pillars)
2. [Polymorphism](#polymorphism)
3. [Interfaces & Abstract Classes](#interfaces--abstract-classes)
4. [Collections](#collections)
5. [Exception Handling](#exception-handling)
6. [Concurrency Basics](#concurrency-basics)
7. [Streams & Functional](#streams--functional)
8. [HashMap Internals](#hashmap-internals)

---

## OOP Pillars

### What are the 4 pillars of OOP?

**Encapsulation** — hide state, expose behavior via public API.  
**Inheritance** — reuse behavior via `extends` (is-a relationship).  
**Polymorphism** — one interface, multiple implementations.  
**Abstraction** — expose only what's needed; hide complexity.

```java
public class BankAccount {
    private double balance; // encapsulation

    public void deposit(double amount) { balance += amount; }
    public double getBalance() { return balance; }
}
```

---

## Polymorphism

### Explain Polymorphism with an example.

Two forms:
- **Static (compile-time)** — method overloading (same name, different params)
- **Dynamic (runtime)** — method overriding (subclass redefines parent method)

```java
// Static
int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }

// Dynamic
class Animal { String sound() { return "..."; } }
class Dog extends Animal { @Override String sound() { return "Woof"; } }

Animal a = new Dog();
a.sound(); // "Woof" — resolved at runtime
```

### What is Dynamic Polymorphism and its advantages?

Dynamic dispatch: the JVM resolves the method at **runtime** based on the actual object type, not the reference type.

**Advantages:**
- Open/Closed Principle: add new types without changing existing code
- Testability: swap implementations via interfaces
- Flexibility: `List<Animal>` holds Dogs, Cats, Birds

### Why do we need Loose Coupling?

Tightly coupled code is hard to test, change, and scale.  
Loose coupling via interfaces/DI means:
- Components are **independently testable** (mock the dependency)
- One change doesn't cascade across the codebase
- Services can be swapped without touching callers

```java
// Tight coupling — hard to test
class OrderService {
    EmailService email = new EmailService(); // hardwired
}

// Loose coupling — injectable, testable
class OrderService {
    private final NotificationService notifier;
    OrderService(NotificationService notifier) { this.notifier = notifier; }
}
```

---

## Interfaces & Abstract Classes

### Why doesn't Java support multiple inheritance with classes?

The **Diamond Problem**: if two parent classes define the same method, the compiler can't decide which one to inherit.

```
      A.move()
     /        \
    B           C    ← both override move()
     \        /
        D          ← which move() does D get?
```

Java solves this by allowing only single class inheritance but **multiple interface implementation**. If two interfaces have the same `default` method, the implementing class **must override it** explicitly.

### Difference between Abstract Class and Interface?

| | Abstract Class | Interface |
|---|---|---|
| State | Can have instance fields | No instance state (constants only) |
| Constructor | Yes | No |
| Methods | Abstract + concrete | Abstract + `default` + `static` |
| Inheritance | Single (`extends`) | Multiple (`implements`) |
| Use when | Shared base behavior + state | Contract / capability definition |

**Rule of thumb:** use `interface` by default; use `abstract class` when you need to share state or a partial implementation.

### What if two interfaces have the same `default` method `test()`?

```java
interface A { default void test() { System.out.println("A"); } }
interface B { default void test() { System.out.println("B"); } }

class C implements A, B {
    @Override
    public void test() {
        A.super.test(); // disambiguate explicitly
    }
}
```

The compiler forces you to override. Not doing so is a **compile error**.

---

## Collections

### Difference between ArrayList and LinkedList?

| | ArrayList | LinkedList |
|---|---|---|
| Backed by | Dynamic array | Doubly-linked list |
| Random access | O(1) | O(n) |
| Insert/delete (middle) | O(n) — shifts elements | O(1) — pointer update |
| Memory | Compact | Extra overhead (prev/next pointers) |
| Use when | Reads dominate | Frequent inserts/deletes |

**Default choice: `ArrayList`** — cache-friendly, better performance for most use cases.

### HashSet vs TreeSet?

| | HashSet | TreeSet |
|---|---|---|
| Order | None | Natural / Comparator order |
| Performance | O(1) avg | O(log n) |
| Nulls | One null allowed | No null (NullPointerException) |
| Backed by | HashMap | TreeMap (Red-Black tree) |
| Use when | Fast lookups | Sorted iteration needed |

---

## HashMap Internals

### What is HashMap? How does it work internally?

HashMap stores key-value pairs in an array of **buckets**. The bucket index is derived from `key.hashCode()`.

**Steps on `put(key, value)`:**
1. Compute `hash = key.hashCode()` → spread with bit operations
2. `index = hash & (capacity - 1)` → bucket slot
3. If bucket empty → insert new `Node`
4. If collision → chain via linked list; if chain length ≥ 8 → convert to **Red-Black Tree** (Java 8+)
5. If `size > capacity * loadFactor (0.75)` → **resize** (double capacity, rehash all)

```java
// Critical contract: if a.equals(b), then a.hashCode() == b.hashCode()
// Violation → objects lost in wrong bucket

class Key {
    int id;
    @Override public boolean equals(Object o) { ... }
    @Override public int hashCode() { return Objects.hash(id); }
}
```

**Collision resolution:** chaining (linked list → tree). Java 8+ treeifies when chain > 8 nodes, untreeifies when < 6.

---

## Exception Handling

### What is a Checked Exception?

Checked exceptions must be declared (`throws`) or caught. They represent **recoverable** conditions the caller should handle.

```java
// Checked — caller must handle or propagate
public String readFile(String path) throws IOException { ... }

// Unchecked (RuntimeException) — programming errors, not required to catch
throw new IllegalArgumentException("invalid input");
```

| | Checked | Unchecked |
|---|---|---|
| Extends | `Exception` | `RuntimeException` |
| Compile check | Yes | No |
| Examples | `IOException`, `SQLException` | `NullPointerException`, `IllegalArgumentException` |
| Use when | External failure (file, network) | Programming bugs |

---

## Concurrency Basics

### How does `synchronized` work?

`synchronized` acquires a **monitor lock** on the object (or class for `static`). Only one thread can hold it at a time — others block.

```java
class Counter {
    private int count = 0;

    public synchronized void increment() { count++; } // method-level lock

    public void decrement() {
        synchronized (this) { count--; } // block-level lock
    }
}
```

**Cost:** context switching + cache coherency. Prefer `ReentrantLock` or `java.util.concurrent` atomics for fine-grained control.

### Explain CountDownLatch in multithreading.

`CountDownLatch` lets one or more threads **wait** until a set of operations completes.

```java
CountDownLatch latch = new CountDownLatch(3); // 3 workers

Runnable worker = () -> {
    doWork();
    latch.countDown(); // decrement counter
};

new Thread(worker).start();
new Thread(worker).start();
new Thread(worker).start();

latch.await(); // main thread blocks until count = 0
System.out.println("All workers done");
```

Use case: wait for all microservice dependencies to warm up before accepting traffic.

---

## Streams & Functional

### `map()` vs `flatMap()`?

`map()` — 1-to-1 transformation. Returns `Stream<T>`.  
`flatMap()` — 1-to-many + **flatten**. Returns `Stream<T>` from `Stream<Stream<T>>`.

```java
List<String> words = List.of("hello world", "foo bar");

// map → Stream<String[]>
words.stream().map(s -> s.split(" "));

// flatMap → Stream<String> (flattened)
words.stream()
     .flatMap(s -> Arrays.stream(s.split(" ")))
     .forEach(System.out::println);
// hello, world, foo, bar
```

**Rule:** if your mapping function returns a collection/stream, use `flatMap`.
