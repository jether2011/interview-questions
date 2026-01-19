# Java Fundamentals Interview Questions

## Table of Contents
1. [Core Java Concepts](#core-java-concepts)
2. [Collections Framework](#collections-framework)
3. [Equals and HashCode](#equals-and-hashcode)
4. [Multithreading and Concurrency](#multithreading-and-concurrency)
5. [Memory Management and JVM](#memory-management-and-jvm)
6. [Garbage Collection](#garbage-collection)
7. [Java 8+ Features](#java-8-features)
8. [Exception Handling](#exception-handling)
9. [Data Structures](#data-structures)
10. [Performance and Optimization](#performance-and-optimization)

---

## Core Java Concepts

### Q1: What are the main principles of OOP in Java?
**Answer:** 
The four pillars of Object-Oriented Programming are:

- **Encapsulation**: Bundling data (fields) and methods that operate on that data within a single unit (class), while hiding internal implementation details from the outside world through access modifiers.

- **Inheritance**: Mechanism where one class (child/subclass) acquires properties and behaviors from another class (parent/superclass), promoting code reuse and establishing an "is-a" relationship.

- **Polymorphism**: The ability of objects to take multiple forms. Achieved through:
  - **Compile-time (Static)**: Method overloading - same method name, different parameters
  - **Runtime (Dynamic)**: Method overriding - subclass provides specific implementation of parent method

- **Abstraction**: Hiding complex implementation details and exposing only the necessary features. Achieved through abstract classes and interfaces.

```java
// Encapsulation example
public class BankAccount {
    private double balance;  // Hidden from outside
    
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public double getBalance() {
        return balance;
    }
}

// Polymorphism example
public abstract class Animal {
    public abstract void makeSound();  // Abstraction
}

public class Dog extends Animal {  // Inheritance
    @Override
    public void makeSound() {
        System.out.println("Bark!");  // Runtime polymorphism
    }
}
```

### Q2: What is the difference between JDK, JRE, and JVM?
**Answer:**

| Component | Description | Contains |
|-----------|-------------|----------|
| **JVM (Java Virtual Machine)** | Runtime environment that executes Java bytecode | Classloader, Memory areas, Execution engine |
| **JRE (Java Runtime Environment)** | Provides libraries and JVM to run Java applications | JVM + Core libraries + Supporting files |
| **JDK (Java Development Kit)** | Complete development environment | JRE + Compiler (javac) + Debugger + Development tools |

**Relationship**: JDK ⊃ JRE ⊃ JVM

```
JDK
├── JRE
│   ├── JVM
│   │   ├── Class Loader Subsystem
│   │   ├── Runtime Data Areas (Heap, Stack, Method Area)
│   │   └── Execution Engine (Interpreter, JIT Compiler)
│   └── Core Libraries (java.lang, java.util, etc.)
├── Compiler (javac)
├── Debugger (jdb)
├── Documentation generator (javadoc)
└── Other tools (jar, jlink, jconsole)
```

### Q3: Explain the difference between abstract class and interface.
**Answer:**

| Feature | Abstract Class | Interface (Java 8+) |
|---------|---------------|---------------------|
| Methods | Abstract and concrete methods | Abstract, default, static, private (Java 9+) |
| Variables | Instance variables allowed | Only public static final (constants) |
| Constructors | Can have constructors | No constructors |
| Inheritance | Single inheritance only | Multiple inheritance supported |
| Access Modifiers | Any access modifier | Methods public by default |
| Use Case | "Is-a" relationship, shared state | "Can-do" capability, contract definition |

```java
// Abstract class - when you have shared state and behavior
public abstract class Vehicle {
    protected String brand;  // Shared state
    protected int year;
    
    public Vehicle(String brand, int year) {
        this.brand = brand;
        this.year = year;
    }
    
    public void startEngine() {  // Shared behavior
        System.out.println("Engine started");
    }
    
    public abstract void drive();  // Must be implemented
}

// Interface - when you define capabilities
public interface Flyable {
    void fly();
    
    default void land() {  // Default implementation (Java 8+)
        System.out.println("Landing...");
    }
    
    static int getMaxAltitude() {  // Static method (Java 8+)
        return 35000;
    }
}

// A class can extend one class but implement multiple interfaces
public class FlyingCar extends Vehicle implements Flyable, Driveable {
    // Implementation
}
```

**When to use which:**
- Use **abstract class** when classes share state/behavior and have "is-a" relationship
- Use **interface** when unrelated classes need common capability (contract)

### Q4: What is the difference between == and equals()?
**Answer:**

| Aspect | == | equals() |
|--------|-----|----------|
| Comparison type | Reference comparison (memory address) | Content/value comparison |
| For primitives | Compares values | N/A (primitives don't have methods) |
| For objects | Compares if same object in memory | Compares logical equality (when overridden) |
| Can be overridden | No (operator) | Yes (method from Object class) |
| Null safety | Works with null | Throws NullPointerException if called on null |

```java
String s1 = new String("hello");
String s2 = new String("hello");
String s3 = s1;
String s4 = "hello";
String s5 = "hello";

// Reference comparison
System.out.println(s1 == s2);  // false (different objects)
System.out.println(s1 == s3);  // true (same reference)
System.out.println(s4 == s5);  // true (string pool - same reference)

// Content comparison
System.out.println(s1.equals(s2));  // true (same content)
System.out.println(s1.equals(s3));  // true

// Best practice for null-safe comparison
System.out.println(Objects.equals(s1, s2));  // true, handles null safely
```

### Q5: What are the different types of inner classes in Java?
**Answer:**

**1. Member Inner Class (Non-static)**
```java
public class Outer {
    private int x = 10;
    
    class Inner {
        void display() {
            System.out.println(x);  // Can access outer class members
        }
    }
}
// Usage: Outer.Inner inner = new Outer().new Inner();
```

**2. Static Nested Class**
```java
public class Outer {
    private static int x = 10;
    
    static class StaticNested {
        void display() {
            System.out.println(x);  // Can only access static members
        }
    }
}
// Usage: Outer.StaticNested nested = new Outer.StaticNested();
```

**3. Local Inner Class (inside method)**
```java
public class Outer {
    void method() {
        final int localVar = 5;  // Must be final or effectively final
        
        class LocalInner {
            void display() {
                System.out.println(localVar);
            }
        }
        new LocalInner().display();
    }
}
```

**4. Anonymous Inner Class**
```java
// Common for event handlers, callbacks, single-use implementations
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running!");
    }
};

// Can be replaced with lambda (Java 8+)
Runnable lambda = () -> System.out.println("Running!");
```

### Q6: What are access modifiers in Java?
**Answer:**

| Modifier | Same Class | Same Package | Subclass (different package) | Different Package |
|----------|------------|--------------|------------------------------|-------------------|
| `private` | ✅ | ❌ | ❌ | ❌ |
| `default` (no modifier) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**Best Practices:**
- Use most restrictive access level possible (principle of least privilege)
- Make fields `private` and provide getters/setters if needed
- Use `protected` for methods intended for subclass use
- Use `public` only for API methods

### Q7: Explain method overloading vs method overriding.
**Answer:**

| Aspect | Overloading | Overriding |
|--------|-------------|------------|
| Definition | Same method name, different parameters | Same method signature in subclass |
| Binding | Compile-time (static) | Runtime (dynamic) |
| Inheritance | Not required | Required (parent-child) |
| Return type | Can be different | Must be same or covariant |
| Access modifier | Can be different | Cannot be more restrictive |
| Exceptions | Can throw any | Cannot throw broader checked exceptions |
| static methods | Can be overloaded | Cannot be overridden (hidden) |

```java
// Overloading - same class, different parameters
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c) { return a + b + c; }
}

// Overriding - subclass provides specific implementation
public class Animal {
    public void speak() { System.out.println("Some sound"); }
}

public class Dog extends Animal {
    @Override  // Annotation is optional but recommended
    public void speak() { System.out.println("Bark!"); }
}
```

### Q8: What is the final keyword used for?
**Answer:**

**1. Final Variable** - Cannot be reassigned after initialization
```java
final int MAX_SIZE = 100;  // Constant
final List<String> list = new ArrayList<>();  // Reference can't change
list.add("item");  // But object state can be modified!
```

**2. Final Method** - Cannot be overridden in subclasses
```java
public class Parent {
    public final void importantMethod() {
        // Subclasses cannot override this
    }
}
```

**3. Final Class** - Cannot be extended
```java
public final class String {  // String is final in Java
    // No subclasses allowed
}
```

**Why use final:**
- **Immutability**: Create immutable classes
- **Security**: Prevent malicious subclassing
- **Performance**: JVM can optimize final methods/classes
- **Thread safety**: Final fields are safely published

### Q9: What is the static keyword?
**Answer:**

| Static Element | Description |
|----------------|-------------|
| Static variable | Shared across all instances (class-level) |
| Static method | Can be called without object, can't access instance members |
| Static block | Executes once when class is loaded |
| Static class | Only for nested classes |

```java
public class Counter {
    private static int count = 0;  // Shared by all instances
    
    static {  // Static initialization block
        System.out.println("Class loaded");
    }
    
    public Counter() {
        count++;
    }
    
    public static int getCount() {  // Static method
        return count;
        // Cannot use 'this' or instance variables here
    }
}
```

**Important Rules:**
- Static methods cannot access instance variables/methods directly
- Static methods cannot use `this` or `super`
- Static variables are initialized before instance variables
- Static blocks run in order of appearance

---

## Collections Framework

### Q10: Explain the Java Collections hierarchy.
**Answer:**

```
                    Iterable<E>
                        │
                   Collection<E>
            ┌───────────┼───────────┐
           List<E>    Set<E>     Queue<E>
            │           │           │
    ┌───────┼───────┐   │    ┌──────┼──────┐
ArrayList LinkedList  │  PriorityQueue  Deque
  Vector              │                   │
                ┌─────┼─────┐      ArrayDeque
             HashSet TreeSet LinkedHashSet
             
                    Map<K,V> (separate hierarchy)
            ┌───────────┼───────────┐
         HashMap    TreeMap    LinkedHashMap
         Hashtable
```

### Q11: How does HashMap work internally?
**Answer:**

HashMap uses an **array of buckets** (Node<K,V>[] table) with hash-based indexing:

**Put Operation:**
1. Calculate `hashCode()` of key
2. Apply internal hash function: `hash = key.hashCode() ^ (key.hashCode() >>> 16)`
3. Calculate bucket index: `index = hash & (n-1)` where n is array length
4. If bucket is empty, add new Node
5. If bucket has nodes, check for key equality using `equals()`
6. If key exists, update value; otherwise, add to chain

**Collision Handling:**
- **Java 7**: Linked list at each bucket
- **Java 8+**: Linked list converts to **Red-Black Tree** when bucket size > 8 (TREEIFY_THRESHOLD)
  - Converts back to list when size < 6 (UNTREEIFY_THRESHOLD)
  - Tree provides O(log n) instead of O(n) for worst case

```java
// Internal structure (simplified)
static class Node<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;  // For linked list
}

// TreeNode for tree structure (Java 8+)
static final class TreeNode<K,V> extends Node<K,V> {
    TreeNode<K,V> parent;
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;
    boolean red;
}
```

**Key Properties:**
- Default initial capacity: 16
- Default load factor: 0.75
- Resize threshold: capacity × load factor
- When threshold exceeded, capacity doubles

### Q12: What's the difference between HashMap, LinkedHashMap, and TreeMap?
**Answer:**

| Feature | HashMap | LinkedHashMap | TreeMap |
|---------|---------|---------------|---------|
| Ordering | No order guarantee | Insertion/Access order | Sorted (natural/comparator) |
| Null keys | One null key allowed | One null key allowed | Not allowed (for natural ordering) |
| Performance | O(1) average | O(1) average | O(log n) |
| Implementation | Hash table | Hash table + Linked list | Red-Black tree |
| Use case | General purpose | LRU cache, maintain order | Sorted data, range queries |

```java
// HashMap - no order
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("banana", 2);
hashMap.put("apple", 1);
hashMap.put("cherry", 3);
// Iteration order unpredictable

// LinkedHashMap - insertion order
Map<String, Integer> linkedMap = new LinkedHashMap<>();
// Same puts -> iteration order: banana, apple, cherry

// LinkedHashMap - access order (for LRU cache)
Map<String, Integer> lruMap = new LinkedHashMap<>(16, 0.75f, true);
// Most recently accessed moves to end

// TreeMap - sorted by key
Map<String, Integer> treeMap = new TreeMap<>();
// Same puts -> iteration order: apple, banana, cherry (alphabetical)

// TreeMap with custom comparator
Map<String, Integer> reverseMap = new TreeMap<>(Comparator.reverseOrder());
```

### Q13: Explain ArrayList vs LinkedList.
**Answer:**

| Operation | ArrayList | LinkedList |
|-----------|-----------|------------|
| Random access `get(index)` | O(1) | O(n) |
| Add at end `add(e)` | O(1) amortized | O(1) |
| Add at beginning `add(0, e)` | O(n) | O(1) |
| Add at middle `add(i, e)` | O(n) | O(n)* |
| Remove by index | O(n) | O(n)* |
| Remove by value | O(n) | O(n) |
| Memory overhead | Low (contiguous array) | High (node pointers) |
| Cache performance | Excellent (locality) | Poor (scattered nodes) |

*LinkedList: O(1) for remove/add if you have the node reference

```java
// ArrayList - backed by dynamic array
ArrayList<String> arrayList = new ArrayList<>();
// Best for: random access, iteration, add at end
// Avoid: frequent insertions/removals at beginning

// LinkedList - doubly linked list
LinkedList<String> linkedList = new LinkedList<>();
// Best for: frequent add/remove at ends (Deque operations)
// Avoid: random access by index

// When to use which:
// - 95% of cases: Use ArrayList (better cache performance)
// - Queue/Deque operations: Use LinkedList or ArrayDeque
// - Frequent insertions in middle: Consider other data structures
```

### Q14: What is the difference between HashSet and TreeSet?
**Answer:**

| Feature | HashSet | TreeSet |
|---------|---------|---------|
| Ordering | No ordering | Sorted (natural/comparator) |
| Null elements | One null allowed | Not allowed |
| Performance | O(1) average | O(log n) |
| Implementation | HashMap internally | TreeMap internally (Red-Black tree) |
| Comparison | Uses `hashCode()` and `equals()` | Uses `compareTo()` or Comparator |

```java
// HashSet - unordered, fast
Set<String> hashSet = new HashSet<>();
hashSet.add("banana");
hashSet.add("apple");
hashSet.add("cherry");
// Iteration: unpredictable order

// TreeSet - sorted
Set<String> treeSet = new TreeSet<>();
treeSet.add("banana");
treeSet.add("apple");
treeSet.add("cherry");
// Iteration: apple, banana, cherry

// TreeSet with custom comparator
Set<String> reverseSet = new TreeSet<>(Comparator.reverseOrder());

// LinkedHashSet - insertion order preserved
Set<String> linkedSet = new LinkedHashSet<>();
// Iteration: banana, apple, cherry (insertion order)
```

### Q15: What is the time complexity of common operations in Java collections?
**Answer:**

| Collection | add | remove | get | contains | Notes |
|------------|-----|--------|-----|----------|-------|
| **ArrayList** | O(1)* | O(n) | O(1) | O(n) | *amortized, O(n) if resize |
| **LinkedList** | O(1) | O(1)** | O(n) | O(n) | **if node known |
| **HashMap** | O(1)* | O(1)* | O(1)* | O(1)* | *average, O(n) worst |
| **TreeMap** | O(log n) | O(log n) | O(log n) | O(log n) | Red-Black tree |
| **HashSet** | O(1)* | O(1)* | N/A | O(1)* | *average |
| **TreeSet** | O(log n) | O(log n) | N/A | O(log n) | |
| **PriorityQueue** | O(log n) | O(log n) | O(1)*** | O(n) | ***peek only |

### Q16: What are concurrent collections in Java?
**Answer:**

**1. ConcurrentHashMap** - Thread-safe HashMap replacement
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
// No locking for reads, segment-based locking for writes (Java 7)
// CAS operations with fine-grained locking (Java 8+)

// Atomic operations
map.putIfAbsent("key", 1);
map.computeIfAbsent("key", k -> expensiveComputation(k));
map.merge("key", 1, Integer::sum);  // Increment counter
```

**2. CopyOnWriteArrayList** - Thread-safe ArrayList for read-heavy scenarios
```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
// Creates new array copy on every write
// Iterators never throw ConcurrentModificationException
// Best for: many readers, few writers
```

**3. BlockingQueue implementations**
```java
// ArrayBlockingQueue - bounded, FIFO
BlockingQueue<Task> queue = new ArrayBlockingQueue<>(100);
queue.put(task);   // Blocks if full
Task t = queue.take();  // Blocks if empty

// LinkedBlockingQueue - optionally bounded
// PriorityBlockingQueue - priority-based
// DelayQueue - delayed execution
```

**4. ConcurrentSkipListMap/Set** - Thread-safe sorted collections
```java
ConcurrentNavigableMap<String, Integer> skipListMap = new ConcurrentSkipListMap<>();
// O(log n) operations, sorted, thread-safe
```

---

## Equals and HashCode

### Q17: What is the contract between equals() and hashCode()?
**Answer:**

The contract consists of these rules:

1. **Consistency with equals**: If `a.equals(b)` returns true, then `a.hashCode() == b.hashCode()` must be true
2. **Reverse is NOT required**: Two objects with same hashCode may or may not be equal
3. **Consistency**: Multiple invocations of hashCode() on unchanged object must return same value
4. **equals() properties**: Must be reflexive, symmetric, transitive, and consistent

```java
// Correct implementation
public class Employee {
    private String id;
    private String name;
    private String department;
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;  // Same reference
        if (obj == null || getClass() != obj.getClass()) return false;
        Employee other = (Employee) obj;
        return Objects.equals(id, other.id);  // Business key
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(id);  // Same fields as equals
    }
}
```

### Q18: What happens if you override equals() but not hashCode()?
**Answer:**

Breaking the contract causes **HashMap/HashSet to malfunction**:

```java
class Person {
    String name;
    
    Person(String name) { this.name = name; }
    
    @Override
    public boolean equals(Object obj) {
        if (obj instanceof Person) {
            return name.equals(((Person) obj).name);
        }
        return false;
    }
    // hashCode() NOT overridden!
}

// Problem demonstration
Person p1 = new Person("John");
Person p2 = new Person("John");

System.out.println(p1.equals(p2));  // true

Set<Person> set = new HashSet<>();
set.add(p1);
set.add(p2);
System.out.println(set.size());  // 2! (should be 1)

Map<Person, String> map = new HashMap<>();
map.put(p1, "Developer");
System.out.println(map.get(p2));  // null! (should be "Developer")
```

**Why this happens:**
- HashSet/HashMap use hashCode() to find the bucket
- p1 and p2 have different default hashCodes (based on memory address)
- They end up in different buckets
- equals() is never even called!

### Q19: Best practices for implementing equals() and hashCode()
**Answer:**

```java
public class Product {
    private final String sku;        // Business key (immutable)
    private String name;             // Mutable
    private BigDecimal price;        // Mutable
    
@Override
public boolean equals(Object obj) {
        // 1. Self check
    if (this == obj) return true;
        
        // 2. Null check
        if (obj == null) return false;
        
        // 3. Type check (use getClass() for final classes, instanceof for inheritance)
        if (getClass() != obj.getClass()) return false;
        
        // 4. Field comparison (use business key if available)
        Product other = (Product) obj;
        return Objects.equals(sku, other.sku);
}

@Override
public int hashCode() {
        // Use same fields as equals
        return Objects.hash(sku);
    }
}

// Using Lombok (recommended for simplicity)
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class Product {
    @EqualsAndHashCode.Include
    private final String sku;
    private String name;
    private BigDecimal price;
}

// Using Records (Java 14+) - automatic equals/hashCode
public record Product(String sku, String name, BigDecimal price) {}
```

**Best Practices:**
1. Use only **immutable fields** or business keys in equals/hashCode
2. Always override **both** methods together
3. Use `Objects.equals()` and `Objects.hash()` for null safety
4. Consider using IDE generation, Lombok, or Records
5. Use `@EqualsAndHashCode.Include/Exclude` wisely

---

## Multithreading and Concurrency

### Q20: What are the ways to create a thread in Java?
**Answer:**

**1. Extending Thread class**
```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running: " + getName());
}
}
new MyThread().start();
```

**2. Implementing Runnable interface (preferred)**
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable running");
}
}
new Thread(new MyRunnable()).start();

// Lambda version
new Thread(() -> System.out.println("Lambda thread")).start();
```

**3. Using Callable with ExecutorService (returns result)**
```java
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(task);
Integer result = future.get();  // Blocks until complete
executor.shutdown();
```

**4. Virtual Threads (Java 21+)**
```java
// Platform thread (OS thread)
Thread platformThread = Thread.ofPlatform().start(() -> { });

// Virtual thread (lightweight, JVM managed)
Thread virtualThread = Thread.ofVirtual().start(() -> { });

// Using ExecutorService with virtual threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> processRequest());
}
```

**Why Runnable is preferred over Thread:**
- Separation of concerns (task vs execution)
- Class can implement Runnable and extend another class
- Better for thread pools and executors
- More flexible resource management

### Q21: Explain the thread lifecycle in Java.
**Answer:**

```
     start()                  
NEW ─────────> RUNNABLE ◄─────────┐
                  │                │
                  │ waiting for    │ notify()/
                  │ lock          │ notifyAll()
                  ▼                │ 
              BLOCKED              │
                  │                │
                  │ acquired       │
                  │ lock          │
                  ▼                │
              RUNNABLE            │
                  │                │
                  │ wait()/join() │
                  ▼                │
              WAITING ────────────┘
                  │
                  │ wait(timeout)/
                  │ sleep()
                  ▼
           TIMED_WAITING
                  │
                  │ run() completes
                  ▼
             TERMINATED
```

| State | Description |
|-------|-------------|
| NEW | Thread created but not started |
| RUNNABLE | Executing or ready to execute (includes OS ready/running) |
| BLOCKED | Waiting to acquire a monitor lock |
| WAITING | Waiting indefinitely for another thread (wait(), join()) |
| TIMED_WAITING | Waiting for specified time (sleep(), wait(timeout)) |
| TERMINATED | Thread completed execution |

### Q22: What is the difference between synchronized and volatile?
**Answer:**

| Feature | synchronized | volatile |
|---------|--------------|----------|
| Purpose | Mutual exclusion + visibility | Visibility only |
| Atomicity | Guarantees atomicity | Only for read/write of reference |
| Blocking | Blocks other threads | Non-blocking |
| Usage | Methods or blocks | Variables only |
| Performance | Higher overhead | Lower overhead |

```java
// volatile - only visibility guarantee
private volatile boolean running = true;

public void stop() {
    running = false;  // Write is visible to all threads
}

public void run() {
    while (running) {  // Read sees latest value
        // work
    }
}

// volatile NOT sufficient for compound operations
private volatile int count = 0;
count++;  // NOT atomic! (read-modify-write)

// synchronized - mutual exclusion + visibility
private int count = 0;

public synchronized void increment() {
    count++;  // Atomic due to lock
}

// synchronized block for finer control
public void process() {
    // non-critical code
    synchronized (this) {
        count++;
    }
    // non-critical code
}
```

### Q23: Explain deadlock and how to prevent it.
**Answer:**

**Deadlock** occurs when two or more threads are blocked forever, each waiting for the other.

**Four conditions (all must be present):**
1. **Mutual Exclusion**: Resources cannot be shared
2. **Hold and Wait**: Thread holds resource while waiting for another
3. **No Preemption**: Resources cannot be forcibly taken
4. **Circular Wait**: Circular chain of threads waiting

```java
// Deadlock example
Object lock1 = new Object();
Object lock2 = new Object();

Thread t1 = new Thread(() -> {
    synchronized (lock1) {
        Thread.sleep(100);
        synchronized (lock2) {  // Waits for t2 to release lock2
            // work
        }
    }
});

Thread t2 = new Thread(() -> {
    synchronized (lock2) {
        Thread.sleep(100);
        synchronized (lock1) {  // Waits for t1 to release lock1
            // work
        }
    }
});
```

**Prevention Strategies:**

```java
// 1. Lock ordering - always acquire locks in same order
synchronized (lock1) {
    synchronized (lock2) {  // Both threads use this order
        // work
    }
}

// 2. Lock timeout with tryLock
Lock lock1 = new ReentrantLock();
Lock lock2 = new ReentrantLock();

if (lock1.tryLock(1, TimeUnit.SECONDS)) {
    try {
        if (lock2.tryLock(1, TimeUnit.SECONDS)) {
            try {
                // work
            } finally {
                lock2.unlock();
            }
        }
    } finally {
        lock1.unlock();
    }
}

// 3. Use higher-level concurrency utilities
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.compute("key", (k, v) -> v == null ? 1 : v + 1);  // Atomic
```

### Q24: What is the difference between wait() and sleep()?
**Answer:**

| Feature | wait() | sleep() |
|---------|--------|---------|
| Class | Object | Thread |
| Lock | **Releases** monitor lock | **Keeps** lock |
| Context | Must be in synchronized block | Can be called anywhere |
| Wake up | notify(), notifyAll(), timeout | Only timeout or interrupt |
| Purpose | Inter-thread communication | Pausing execution |

```java
// wait() - releases lock, used for signaling
synchronized (sharedObject) {
    while (!condition) {
        sharedObject.wait();  // Releases lock, waits for notify
    }
    // Process when condition is true
}

// Notify waiting threads
synchronized (sharedObject) {
    condition = true;
    sharedObject.notifyAll();  // Wake up all waiting threads
}

// sleep() - keeps lock, just pauses
synchronized (lock) {
    Thread.sleep(1000);  // Still holds lock for 1 second!
}
```

---

## Memory Management and JVM

### Q25: Explain JVM memory structure.
**Answer:**

```
┌─────────────────────────────────────────────────────────────┐
│                         JVM Memory                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    HEAP (Shared)                     │    │
│  │  ┌──────────────┐  ┌────────────────────────────┐   │    │
│  │  │    Young     │  │          Old               │   │    │
│  │  │  Generation  │  │       Generation           │   │    │
│  │  │ ┌────┬────┐  │  │                            │   │    │
│  │  │ │Eden│ S0 │  │  │   Long-lived objects       │   │    │
│  │  │ │    │ S1 │  │  │                            │   │    │
│  │  │ └────┴────┘  │  │                            │   │    │
│  │  └──────────────┘  └────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────┐  ┌──────────────────────────────┐     │
│  │    Metaspace    │  │         Stack (per thread)    │     │
│  │  (Class data)   │  │  ┌─────────────────────────┐  │     │
│  │                 │  │  │ Method frames           │  │     │
│  │  Native memory  │  │  │ - Local variables       │  │     │
│  └─────────────────┘  │  │ - Operand stack         │  │     │
│                       │  │ - Frame data            │  │     │
│  ┌─────────────────┐  │  └─────────────────────────┘  │     │
│  │   PC Register   │  └──────────────────────────────┘     │
│  │  (per thread)   │                                        │
│  └─────────────────┘                                        │
└─────────────────────────────────────────────────────────────┘
```

**Heap Memory:**
- **Young Generation**: New objects allocated here
  - **Eden**: Initial allocation space
  - **Survivor Spaces (S0, S1)**: Objects surviving minor GC
- **Old Generation**: Long-lived objects promoted from Young

**Non-Heap Memory:**
- **Metaspace** (Java 8+): Class metadata, method data (native memory)
- **Stack**: Per-thread, stores frames for method calls
- **PC Register**: Per-thread, current executing instruction
- **Native Method Stack**: Native (JNI) method execution

### Q26: What causes OutOfMemoryError?
**Answer:**

| Error Type | Cause | Solution |
|------------|-------|----------|
| `java.lang.OutOfMemoryError: Java heap space` | Heap full | Increase -Xmx, fix leaks |
| `java.lang.OutOfMemoryError: GC overhead limit exceeded` | GC taking >98% time | Fix leaks, increase heap |
| `java.lang.OutOfMemoryError: Metaspace` | Too many classes loaded | Increase -XX:MaxMetaspaceSize |
| `java.lang.OutOfMemoryError: Direct buffer memory` | NIO direct buffers exhausted | Increase -XX:MaxDirectMemorySize |
| `java.lang.OutOfMemoryError: Unable to create new native thread` | OS thread limit | Reduce thread count, increase OS limits |

```java
// Common memory leak patterns

// 1. Static collections that grow forever
private static final List<Object> cache = new ArrayList<>();
public void addToCache(Object o) {
    cache.add(o);  // Never removed!
}

// 2. Unclosed resources
public void readFile(String path) {
    InputStream is = new FileInputStream(path);
    // If exception occurs, stream never closed
}

// Fix: try-with-resources
public void readFile(String path) {
    try (InputStream is = new FileInputStream(path)) {
        // Auto-closed
    }
}

// 3. Listeners not removed
button.addActionListener(listener);
// Later: forgot to remove listener

// 4. ThreadLocal not cleared
private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<>();
public void process() {
    connectionHolder.set(getConnection());
    // Must call connectionHolder.remove() when done!
}
```

### Q27: What is the difference between Stack and Heap?
**Answer:**

| Aspect | Stack | Heap |
|--------|-------|------|
| Storage | Primitives, references, method frames | Objects, arrays |
| Scope | Thread-specific (each thread has own) | Shared among all threads |
| Size | Limited (typically 1MB default) | Large (configurable) |
| Allocation | LIFO, very fast | Managed by GC |
| Memory management | Automatic (frame popped on method return) | Garbage Collector |
| Error | StackOverflowError | OutOfMemoryError |

```java
public void example() {
    int x = 10;                    // Stack: primitive
    String name = "John";          // Stack: reference to heap
                                   // Heap: String object
    Person p = new Person("John"); // Stack: reference
                                   // Heap: Person object
    
    int[] arr = new int[100];      // Stack: reference
                                   // Heap: array object
}
// When method returns, all stack variables are automatically removed
// Heap objects become eligible for GC when no references exist
```

---

## Garbage Collection

### Q28: Explain different types of garbage collectors.
**Answer:**

| Collector | Characteristics | Use Case |
|-----------|-----------------|----------|
| **Serial GC** | Single-threaded, STW | Small apps, single CPU |
| **Parallel GC** | Multi-threaded STW | Throughput-focused |
| **G1 GC** | Region-based, predictable pauses | Large heaps (>4GB) |
| **ZGC** | Ultra-low latency (<10ms) | Latency-critical apps |
| **Shenandoah** | Concurrent, low-pause | Low-latency requirements |

```bash
# JVM flags for each collector
-XX:+UseSerialGC
-XX:+UseParallelGC
-XX:+UseG1GC           # Default since Java 9
-XX:+UseZGC            # Java 11+
-XX:+UseShenandoahGC   # Java 12+
```

**G1GC (Garbage First) - Default Collector:**
- Divides heap into equal-sized regions
- Prioritizes regions with most garbage ("Garbage First")
- Aims for predictable pause times
- Good for heaps > 4GB

**ZGC - Ultra-Low Latency:**
- Pause times < 10ms regardless of heap size
- Supports heaps up to 16TB
- Concurrent marking and relocation
- Best for latency-critical applications

### Q29: Explain the generational garbage collection hypothesis.
**Answer:**

The **Generational Hypothesis** states that most objects die young.

```
Object Lifecycle:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
█████████████████░░░░░░░░░░░░░░░░░░░░░░░
       ↑                            ↑
   Most objects                Few objects
   die quickly                 live long
```

**How GC uses this:**

1. **Minor GC** (Young Generation):
   - Fast, frequent collection
   - Most objects collected immediately
   - Survivors moved to Survivor spaces
   - After several survivals → promoted to Old Gen

2. **Major GC** (Old Generation):
   - Less frequent, more expensive
   - Collects long-lived objects
   - Can trigger "Stop-the-World" pauses

```java
// GC-friendly code patterns

// 1. Prefer short-lived objects
String result = createTemporaryString();  // GC friendly

// 2. Avoid creating unnecessary objects
// Bad
for (int i = 0; i < 1000; i++) {
    String s = new String("constant");  // 1000 objects!
}

// Good
String s = "constant";  // String pool, single object
for (int i = 0; i < 1000; i++) {
    // use s
}

// 3. Be careful with object pools (can backfire)
// Objects in pools are "old" and expensive to collect
```

### Q30: Explain strong, weak, soft, and phantom references.
**Answer:**

| Reference Type | GC Behavior | Use Case |
|----------------|-------------|----------|
| **Strong** | Never collected while reachable | Normal references |
| **Soft** | Collected before OOM | Memory-sensitive caches |
| **Weak** | Collected at next GC | Caches, canonicalizing mappings |
| **Phantom** | Collected, queued for notification | Resource cleanup tracking |

```java
// Strong - normal reference, prevents GC
Object strong = new Object();

// Weak - collected at any GC if no strong refs
WeakReference<Object> weak = new WeakReference<>(new Object());
Object obj = weak.get();  // May be null if collected

// WeakHashMap - entries removed when key is weakly reachable
Map<Key, Value> cache = new WeakHashMap<>();

// Soft - collected only when memory is needed
SoftReference<byte[]> soft = new SoftReference<>(new byte[10_000_000]);
// Good for caches that should shrink under memory pressure

// Phantom - for tracking object finalization
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantom = new PhantomReference<>(obj, queue);
// phantom.get() always returns null
// Object added to queue after finalization, before memory reclaim
// Used for custom cleanup (alternative to finalizers)
```

---

## Java 8+ Features

### Q31: Explain Lambda Expressions and Functional Interfaces.
**Answer:**

**Functional Interface**: Interface with exactly one abstract method (SAM - Single Abstract Method)

```java
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
    
    // Can have default and static methods
    default int add(int a, int b) { return a + b; }
    static int multiply(int a, int b) { return a * b; }
}

// Lambda expression - concise implementation
Calculator addition = (a, b) -> a + b;
Calculator subtraction = (a, b) -> a - b;

// Using built-in functional interfaces
Predicate<String> isEmpty = s -> s.isEmpty();
Function<String, Integer> toLength = s -> s.length();
Consumer<String> printer = s -> System.out.println(s);
Supplier<Double> randomValue = () -> Math.random();
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// Method references - shorthand for lambdas
Predicate<String> isEmpty2 = String::isEmpty;
Consumer<String> printer2 = System.out::println;
Function<String, Integer> toLength2 = String::length;
Supplier<LocalDate> today = LocalDate::now;
BiFunction<String, String, String> concat = String::concat;
```

### Q32: Explain Stream API operations.
**Answer:**

**Intermediate Operations** (lazy, return Stream):

```java
List<Person> people = getPeople();

// filter - select matching elements
people.stream()
    .filter(p -> p.getAge() > 18)
    
// map - transform elements
    .map(Person::getName)
    
// flatMap - flatten nested structures
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2), Arrays.asList(3, 4));
nested.stream()
    .flatMap(List::stream)  // Stream<Integer>: 1, 2, 3, 4
    
// distinct - remove duplicates
    .distinct()
    
// sorted - sort elements
    .sorted()
    .sorted(Comparator.reverseOrder())
    
// peek - debug/logging (not for modification)
    .peek(System.out::println)
    
// limit/skip - truncation
    .limit(10)
    .skip(5);
```

**Terminal Operations** (trigger execution):

```java
// collect - accumulate results
List<String> names = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());

Map<String, List<Person>> byCity = people.stream()
    .collect(Collectors.groupingBy(Person::getCity));

// forEach - perform action
people.stream().forEach(System.out::println);

// reduce - combine elements
int sum = numbers.stream().reduce(0, Integer::sum);
Optional<Integer> max = numbers.stream().reduce(Integer::max);

// count, min, max
long count = people.stream().count();
Optional<Person> youngest = people.stream()
    .min(Comparator.comparing(Person::getAge));

// anyMatch, allMatch, noneMatch
boolean hasAdult = people.stream().anyMatch(p -> p.getAge() >= 18);

// findFirst, findAny
Optional<Person> first = people.stream()
    .filter(p -> p.getAge() > 30)
    .findFirst();
```

### Q33: What is Optional and how to use it properly?
**Answer:**

**Optional** is a container that may or may not contain a value. Used to avoid null pointer exceptions.

```java
// Creating Optional
Optional<String> empty = Optional.empty();
Optional<String> present = Optional.of("value");      // NPE if null
Optional<String> nullable = Optional.ofNullable(str); // Safe with null

// Bad practices
Optional<User> user = findUser(id);
if (user.isPresent()) {              // Anti-pattern!
    return user.get();
}

// Good practices
// 1. orElse - provide default
String name = optional.orElse("Unknown");

// 2. orElseGet - lazy default (use for expensive operations)
String name = optional.orElseGet(() -> computeDefault());

// 3. orElseThrow - throw exception if empty
User user = optional.orElseThrow(() -> 
    new UserNotFoundException(id));

// 4. map - transform if present
String upperName = optional.map(String::toUpperCase).orElse("");

// 5. flatMap - for nested optionals
Optional<String> city = findUser(id)
    .flatMap(User::getAddress)    // Optional<Address>
    .flatMap(Address::getCity);   // Optional<String>

// 6. filter - conditional
Optional<User> adult = findUser(id)
    .filter(u -> u.getAge() >= 18);

// 7. ifPresent - perform action if present
optional.ifPresent(System.out::println);

// 8. ifPresentOrElse (Java 9+)
optional.ifPresentOrElse(
    System.out::println,
    () -> System.out.println("Not found")
);
```

**When NOT to use Optional:**
- Don't use for fields (memory overhead)
- Don't use for method parameters (just check null)
- Don't use for collections (return empty collection instead)

### Q34: What are Records (Java 14+)?
**Answer:**

**Records** are immutable data carriers with auto-generated methods.

```java
// Traditional class
public class PersonOld {
    private final String name;
    private final int age;
    
    public PersonOld(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    @Override public boolean equals(Object o) { /* ... */ }
    @Override public int hashCode() { /* ... */ }
    @Override public String toString() { /* ... */ }
}

// Record - equivalent to above
public record Person(String name, int age) {}

// Auto-generated:
// - Constructor
// - Accessor methods: name(), age() (not getName()!)
// - equals(), hashCode(), toString()
// - final class (cannot be extended)
// - All fields are final

// Customization
public record Person(String name, int age) {
    // Compact constructor for validation
    public Person {
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
        name = name.trim();  // Can modify parameters
    }
    
    // Additional methods
    public String greeting() {
        return "Hello, " + name;
    }
}

// With generic types
public record Pair<T, U>(T first, U second) {}
```

### Q35: Explain Sealed Classes (Java 17+).
**Answer:**

**Sealed classes** restrict which classes can extend/implement them.

```java
// Sealed class - controls inheritance
public sealed class Shape 
    permits Circle, Rectangle, Triangle {
    // Base implementation
}

// Permitted subclasses must be: final, sealed, or non-sealed
public final class Circle extends Shape {
    private final double radius;
}

public final class Rectangle extends Shape {
    private final double width, height;
}

// non-sealed allows unrestricted extension
public non-sealed class Triangle extends Shape {
    // Any class can extend Triangle
}

// Can also be sealed
public sealed class Polygon extends Shape 
    permits Square, Pentagon {}
```

**Benefits:**
```java
// Exhaustive pattern matching (Java 17+)
public double area(Shape shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.height();
        case Triangle t -> calculateTriangleArea(t);
        // Compiler knows all cases are covered!
    };
}
```

---

## Exception Handling

### Q36: Explain the exception hierarchy in Java.
**Answer:**

```
Throwable
                        │
        ┌───────────────┴───────────────┐
        │                               │
      Error                         Exception
    (unchecked)                         │
        │                   ┌───────────┴───────────┐
        │                   │                       │
   OutOfMemoryError   RuntimeException          Checked
   StackOverflowError   (unchecked)            Exceptions
   VirtualMachineError      │                       │
                            │                   IOException
                   NullPointerException        SQLException
                   IllegalArgumentException    ClassNotFoundException
                   ArrayIndexOutOfBounds       InterruptedException
                   ClassCastException
                   ArithmeticException
```

| Type | Handling | Examples |
|------|----------|----------|
| **Error** | Don't catch | OOM, StackOverflow |
| **Checked Exception** | Must catch or declare | IOException, SQLException |
| **Unchecked Exception** | Optional handling | NullPointerException, IllegalArgumentException |

### Q37: What is try-with-resources?
**Answer:**

Automatic resource management for `AutoCloseable` resources.

```java
// Old way (error-prone)
InputStream is = null;
try {
    is = new FileInputStream("file.txt");
    // use is
} catch (IOException e) {
    // handle
} finally {
    if (is != null) {
        try {
            is.close();
        } catch (IOException e) {
            // Another exception to handle!
        }
    }
}

// Try-with-resources (Java 7+)
try (InputStream is = new FileInputStream("file.txt");
     BufferedReader reader = new BufferedReader(new InputStreamReader(is))) {
    // use resources
} catch (IOException e) {
    // handle - resources automatically closed
}
// Resources closed in reverse order of declaration

// Java 9+ allows effectively final variables
InputStream is = new FileInputStream("file.txt");
try (is) {  // If 'is' is effectively final
    // use
}

// Custom AutoCloseable
public class DatabaseConnection implements AutoCloseable {
    @Override
    public void close() {
        System.out.println("Connection closed");
    }
}
```

### Q38: Best practices for exception handling?
**Answer:**

```java
// 1. Catch specific exceptions, not generic
// Bad
try {
    // code
} catch (Exception e) { }  // Catches everything!

// Good
try {
    // code
} catch (FileNotFoundException e) {
    // Handle missing file
} catch (IOException e) {
    // Handle other I/O issues
}

// 2. Don't swallow exceptions
// Bad
try {
    // code
} catch (Exception e) {
    // Empty - silently failing!
}

// Good
try {
    // code
} catch (Exception e) {
    logger.error("Failed to process", e);
    throw new ServiceException("Processing failed", e);
}

// 3. Use custom exceptions for domain-specific errors
public class InsufficientFundsException extends RuntimeException {
    private final BigDecimal required;
    private final BigDecimal available;
    
    public InsufficientFundsException(BigDecimal required, BigDecimal available) {
        super(String.format("Required: %s, Available: %s", required, available));
        this.required = required;
        this.available = available;
    }
}

// 4. Prefer unchecked exceptions for programming errors
// Use IllegalArgumentException, IllegalStateException, etc.

// 5. Document exceptions in Javadoc
/**
 * Withdraws amount from account.
 * @throws InsufficientFundsException if balance is too low
 * @throws IllegalArgumentException if amount is negative
 */
public void withdraw(BigDecimal amount) { }

// 6. Clean up resources properly
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql)) {
    // Auto-closed
}
```

---

## Data Structures

### Q39: Implement a LRU Cache.
**Answer:**

```java
// Using LinkedHashMap (simplest approach)
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);  // true = access-order
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}

// Usage
LRUCache<String, Integer> cache = new LRUCache<>(3);
cache.put("a", 1);
cache.put("b", 2);
cache.put("c", 3);
cache.get("a");      // Access 'a', moves to end
cache.put("d", 4);   // 'b' evicted (least recently used)

// Manual implementation with HashMap + Doubly Linked List
class LRUCacheManual<K, V> {
    private final int capacity;
    private final Map<K, Node<K, V>> map;
    private final Node<K, V> head, tail;
    
    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V> prev, next;
        
        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
    
    public LRUCacheManual(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        this.head = new Node<>(null, null);
        this.tail = new Node<>(null, null);
        head.next = tail;
        tail.prev = head;
    }
    
    public V get(K key) {
        Node<K, V> node = map.get(key);
        if (node == null) return null;
        moveToHead(node);
        return node.value;
    }
    
    public void put(K key, V value) {
        Node<K, V> node = map.get(key);
        if (node != null) {
            node.value = value;
            moveToHead(node);
        } else {
            node = new Node<>(key, value);
            map.put(key, node);
            addToHead(node);
            if (map.size() > capacity) {
                Node<K, V> removed = removeTail();
                map.remove(removed.key);
            }
        }
    }
    
    private void moveToHead(Node<K, V> node) {
        removeNode(node);
        addToHead(node);
    }
    
    private void addToHead(Node<K, V> node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }
    
    private void removeNode(Node<K, V> node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    private Node<K, V> removeTail() {
        Node<K, V> node = tail.prev;
        removeNode(node);
        return node;
    }
}
```

### Q40: Implement a thread-safe Singleton.
**Answer:**

```java
// 1. Eager Initialization (thread-safe, simple)
public class EagerSingleton {
    private static final EagerSingleton INSTANCE = new EagerSingleton();
    
    private EagerSingleton() {}
    
    public static EagerSingleton getInstance() {
        return INSTANCE;
    }
}

// 2. Lazy Initialization with Double-Checked Locking
public class LazyDCLSingleton {
    private static volatile LazyDCLSingleton instance;  // volatile is crucial!
    
    private LazyDCLSingleton() {}
    
    public static LazyDCLSingleton getInstance() {
        if (instance == null) {                    // First check (no locking)
            synchronized (LazyDCLSingleton.class) {
                if (instance == null) {            // Second check (with lock)
                    instance = new LazyDCLSingleton();
                }
            }
        }
        return instance;
    }
}

// 3. Initialization-on-demand Holder (Bill Pugh Singleton)
public class HolderSingleton {
    private HolderSingleton() {}
    
    private static class Holder {
        static final HolderSingleton INSTANCE = new HolderSingleton();
    }
    
    public static HolderSingleton getInstance() {
        return Holder.INSTANCE;  // Class loading guarantees thread safety
    }
}

// 4. Enum Singleton (Recommended by Joshua Bloch)
public enum EnumSingleton {
    INSTANCE;
    
    private final Connection connection;
    
    EnumSingleton() {
        connection = createConnection();
    }
    
    public Connection getConnection() {
        return connection;
    }
}
// Benefits: Thread-safe, serialization-safe, reflection-proof
```

### Q41: What is the difference between fail-fast and fail-safe iterators?
**Answer:**

| Aspect | Fail-Fast | Fail-Safe |
|--------|-----------|-----------|
| Behavior | Throws ConcurrentModificationException | No exception |
| Collections | ArrayList, HashMap, HashSet | CopyOnWriteArrayList, ConcurrentHashMap |
| Memory | Uses original collection | Uses copy/snapshot |
| Reflection | Shows current state | May show stale data |

```java
// Fail-fast example
List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));
for (String s : list) {
    if (s.equals("b")) {
        list.remove(s);  // ConcurrentModificationException!
    }
}

// Correct ways to remove while iterating
// 1. Use Iterator.remove()
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("b")) {
        it.remove();  // Safe removal
    }
}

// 2. Use removeIf (Java 8+)
list.removeIf(s -> s.equals("b"));

// 3. Collect items to remove
List<String> toRemove = list.stream()
    .filter(s -> s.equals("b"))
    .collect(Collectors.toList());
list.removeAll(toRemove);

// Fail-safe example
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>(list);
for (String s : cowList) {
    if (s.equals("b")) {
        cowList.remove(s);  // No exception (iterates over snapshot)
    }
}
```

---

## Performance and Optimization

### Q42: How to tune JVM performance?
**Answer:**

```bash
# Heap sizing
-Xms2g              # Initial heap size
-Xmx4g              # Maximum heap size
-Xmn1g              # Young generation size

# Garbage collector selection
-XX:+UseG1GC        # G1 (default Java 9+)
-XX:+UseZGC         # ZGC for low latency
-XX:MaxGCPauseMillis=200  # Target pause time for G1

# GC logging (Java 9+)
-Xlog:gc*:file=gc.log:time,uptime:filecount=5,filesize=10m

# Metaspace
-XX:MaxMetaspaceSize=256m

# Thread stack size
-Xss256k

# Performance options
-XX:+UseStringDeduplication  # G1: deduplicate identical strings
-XX:+OptimizeStringConcat    # Optimize string concatenation
```

### Q43: What are Java performance best practices?
**Answer:**

```java
// 1. Use StringBuilder for string concatenation in loops
// Bad
String result = "";
for (String s : strings) {
    result += s;  // Creates new String each iteration
}

// Good
StringBuilder sb = new StringBuilder();
for (String s : strings) {
    sb.append(s);
}
String result = sb.toString();

// 2. Prefer primitives over wrappers
// Bad
Long sum = 0L;
for (int i = 0; i < 1000000; i++) {
    sum += i;  // Boxing/unboxing overhead
}

// Good
long sum = 0L;
for (int i = 0; i < 1000000; i++) {
    sum += i;
}

// 3. Use appropriate collection sizes
// Bad
List<User> users = new ArrayList<>();  // Default capacity 10
// Adding 10000 elements causes multiple resizes

// Good
List<User> users = new ArrayList<>(10000);

// 4. Use lazy initialization for expensive resources
private volatile Connection connection;

public Connection getConnection() {
    if (connection == null) {
        synchronized (this) {
            if (connection == null) {
                connection = createConnection();
            }
        }
    }
    return connection;
}

// 5. Use Stream parallel() wisely
// Good for CPU-bound tasks with large datasets
list.parallelStream()
    .filter(this::expensiveCheck)
    .collect(Collectors.toList());

// Bad for I/O bound or small datasets
// Overhead of parallelization not worth it

// 6. Cache expensive computations
private final Map<String, Result> cache = new ConcurrentHashMap<>();

public Result compute(String key) {
    return cache.computeIfAbsent(key, this::expensiveComputation);
}
```

### Q44: How to diagnose production Java issues?
**Answer:**

```bash
# 1. Thread dump (find deadlocks, thread states)
jstack <pid>
# Or: kill -3 <pid>  (Unix)

# 2. Heap dump (memory analysis)
jmap -dump:format=b,file=heap.hprof <pid>
# Or enable auto dump on OOM:
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/dumps

# 3. JVM statistics
jstat -gc <pid> 1000  # GC stats every second
jstat -gcutil <pid> 1000  # GC utilization

# 4. Native memory tracking
-XX:NativeMemoryTracking=summary
jcmd <pid> VM.native_memory summary

# 5. Flight Recorder (low overhead profiling)
-XX:StartFlightRecording=duration=60s,filename=recording.jfr

# 6. Remote debugging
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```

**Analysis Tools:**
- **VisualVM**: Real-time monitoring, profiling
- **JProfiler/YourKit**: Commercial profilers
- **Eclipse MAT**: Heap dump analysis
- **Java Mission Control**: JFR analysis
- **Async Profiler**: Low-overhead CPU/memory profiling

---

## Interview Tips

1. **Understand concepts deeply**: Don't just memorize - understand WHY things work
2. **Practice coding**: Implement data structures and algorithms by hand
3. **Know trade-offs**: Every design decision has pros and cons
4. **Stay updated**: Keep learning new Java features (Records, Virtual Threads, etc.)
5. **Debug systematically**: Use proper tools and methodologies
6. **Communicate clearly**: Explain your thought process during interviews
7. **Ask clarifying questions**: Don't assume requirements
8. **Draw diagrams**: Visualize data structures, memory, thread interactions
