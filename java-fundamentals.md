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

## Core Java Concepts

### Q1: What are the main principles of OOP in Java?
**Answer:** 
- **Encapsulation**: Bundling data and methods that operate on that data within a single unit (class), hiding internal implementation details
- **Inheritance**: Mechanism where one class acquires properties and behaviors from a parent class
- **Polymorphism**: Ability of objects to take multiple forms (method overloading and overriding)
- **Abstraction**: Hiding complex implementation details and showing only essential features

### Q2: What is the difference between JDK, JRE, and JVM?
**Answer:**
- **JVM (Java Virtual Machine)**: Runtime environment that executes Java bytecode
- **JRE (Java Runtime Environment)**: JVM + libraries + other components to run Java applications
- **JDK (Java Development Kit)**: JRE + development tools (compiler, debugger, etc.)

### Q3: Explain the difference between abstract class and interface.
**Answer:**
- **Abstract Class**: 
  - Can have abstract and non-abstract methods
  - Can have constructors and instance variables
  - Supports single inheritance
  - Can have any access modifiers
- **Interface** (Java 8+):
  - All methods are public by default
  - Can have default and static methods
  - Supports multiple inheritance
  - Variables are public, static, and final by default

### Q4: What is the difference between == and equals()?
**Answer:**
- `==` compares references (memory addresses) for objects, values for primitives
- `equals()` compares the content/value of objects (when properly overridden)

### Q5: What are the different types of inner classes in Java?
**Answer:**
- **Member Inner Class**: Non-static class defined at member level
- **Static Nested Class**: Static class defined at member level
- **Local Inner Class**: Class defined inside a method
- **Anonymous Inner Class**: Class without a name, defined and instantiated in one statement

## Collections Framework

### Q6: How does HashMap work internally?
**Answer:**
HashMap uses an array of buckets (Node<K,V>[] table). Each bucket contains a linked list (or tree in Java 8+ for large buckets).
- Uses hashCode() to determine bucket index
- Uses equals() to handle collisions within the same bucket
- Load factor (default 0.75) determines when to resize
- Java 8+ converts linked lists to trees when bucket size > 8 (TREEIFY_THRESHOLD)

### Q7: What's the difference between HashMap and Hashtable?
**Answer:**
| HashMap | Hashtable |
|---------|-----------|
| Not synchronized (not thread-safe) | Synchronized (thread-safe) |
| Allows one null key and multiple null values | No null keys or values |
| Introduced in Java 1.2 | Legacy class (Java 1.0) |
| Better performance | Slower due to synchronization |
| Fail-fast iterator | Enumerator is not fail-fast |

### Q8: Explain ArrayList vs LinkedList.
**Answer:**
| ArrayList | LinkedList |
|-----------|------------|
| Backed by dynamic array | Doubly linked list implementation |
| O(1) for get/set by index | O(n) for get/set by index |
| O(n) for add/remove (except at end) | O(1) for add/remove at beginning/end |
| Better cache locality | Poor cache locality |
| Less memory per element | More memory (stores pointers) |

### Q9: What is the time complexity of common operations in Java collections?
**Answer:**
- **ArrayList**: get O(1), add O(1) amortized, remove O(n), contains O(n)
- **LinkedList**: get O(n), add O(1), remove O(1) at ends, contains O(n)
- **HashMap**: get/put/remove O(1) average, O(n) worst case
- **TreeMap**: get/put/remove O(log n)
- **HashSet**: add/remove/contains O(1) average
- **TreeSet**: add/remove/contains O(log n)

### Q10: What are concurrent collections in Java?
**Answer:**
- **ConcurrentHashMap**: Thread-safe HashMap with segment-based locking
- **CopyOnWriteArrayList**: Thread-safe variant where all mutative operations create a new copy
- **BlockingQueue**: Interface for producer-consumer patterns (ArrayBlockingQueue, LinkedBlockingQueue)
- **ConcurrentSkipListMap**: Thread-safe sorted map
- **ConcurrentLinkedQueue**: Thread-safe unbounded queue

## Equals and HashCode

### Q11: What is the contract between equals() and hashCode()?
**Answer:**
1. If two objects are equal (equals() returns true), they must have the same hash code
2. If two objects have the same hash code, they may or may not be equal
3. hashCode() must return consistent results for the same object
4. equals() must be reflexive, symmetric, transitive, and consistent

### Q12: What happens if you override equals() but not hashCode()?
**Answer:**
Objects that are equal according to equals() may have different hash codes, breaking the contract. This causes:
- HashMap/HashSet malfunction (can't find equal objects)
- Duplicate "equal" objects in HashSet
- Memory leaks in HashMap (objects not properly removed)

### Q13: How to properly implement equals() and hashCode()?
**Answer:**
```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null || getClass() != obj.getClass()) return false;
    MyClass other = (MyClass) obj;
    return Objects.equals(field1, other.field1) && 
           Objects.equals(field2, other.field2);
}

@Override
public int hashCode() {
    return Objects.hash(field1, field2);
}
```

## Multithreading and Concurrency

### Q14: What are the ways to create a thread in Java?
**Answer:**
1. **Extending Thread class**:
```java
class MyThread extends Thread {
    public void run() { /* thread logic */ }
}
```
2. **Implementing Runnable interface**:
```java
class MyRunnable implements Runnable {
    public void run() { /* thread logic */ }
}
```
3. **Using Callable with ExecutorService** (returns result):
```java
Callable<String> task = () -> { return "result"; };
Future<String> future = executor.submit(task);
```
4. **Virtual Threads** (Java 21+):
```java
Thread.ofVirtual().start(() -> { /* logic */ });
```

### Q15: What is the difference between synchronized and volatile?
**Answer:**
- **synchronized**: 
  - Provides mutual exclusion and visibility
  - Blocks other threads
  - Can be used on methods or blocks
  - Guarantees atomicity
- **volatile**:
  - Only provides visibility guarantee
  - Doesn't block threads
  - Only for variables
  - Doesn't guarantee atomicity (except for read/write of reference)

### Q16: Explain deadlock and how to prevent it.
**Answer:**
**Deadlock** occurs when two or more threads are blocked forever, waiting for each other.

**Conditions for deadlock:**
1. Mutual exclusion
2. Hold and wait
3. No preemption
4. Circular wait

**Prevention strategies:**
- Always acquire locks in the same order
- Use tryLock() with timeout
- Avoid nested locks
- Use concurrent collections instead of explicit locking

### Q17: What is ThreadLocal?
**Answer:**
ThreadLocal provides thread-local variables where each thread has its own copy. Common uses:
- User sessions in web applications
- Database connections
- SimpleDateFormat instances (not thread-safe)

```java
ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);
threadLocal.set(42);  // Only for current thread
Integer value = threadLocal.get();  // Gets current thread's value
threadLocal.remove();  // Important to prevent memory leaks
```

### Q18: Explain the thread lifecycle in Java.
**Answer:**
1. **NEW**: Thread created but not started
2. **RUNNABLE**: Thread executing or ready to execute
3. **BLOCKED**: Waiting to acquire a lock
4. **WAITING**: Waiting indefinitely for another thread
5. **TIMED_WAITING**: Waiting for specified time
6. **TERMINATED**: Thread completed execution

### Q19: What is the difference between wait() and sleep()?
**Answer:**
| wait() | sleep() |
|--------|---------|
| Releases lock | Doesn't release lock |
| Must be called from synchronized context | Can be called anywhere |
| Can be awakened by notify()/notifyAll() | Can't be awakened (except interrupt) |
| Instance method of Object | Static method of Thread |
| Used for inter-thread communication | Used for pausing execution |

### Q20: What are Virtual Threads (Project Loom)?
**Answer:**
Virtual threads (Java 21+) are lightweight threads managed by the JVM rather than OS:
- Can create millions of virtual threads
- Ideal for I/O-bound operations
- Automatically yield during blocking operations
- Use same Thread API
- Not suitable for CPU-intensive tasks or synchronized blocks

## Memory Management and JVM

### Q21: Explain JVM memory structure.
**Answer:**
1. **Heap Memory**:
   - **Young Generation**:
     - Eden Space: Where new objects are allocated
     - Survivor Spaces (S0, S1): Objects that survive Eden GC
   - **Old Generation**: Long-lived objects
   - **Metaspace** (Java 8+): Class metadata (replaced PermGen)

2. **Non-Heap Memory**:
   - **Stack**: Method calls, local variables, partial results
   - **Program Counter (PC) Register**: Current executing instruction
   - **Native Method Stack**: Native method execution

### Q22: What is the difference between Stack and Heap?
**Answer:**
| Stack | Heap |
|-------|------|
| Stores local variables and method calls | Stores objects and arrays |
| Thread-specific (each thread has its own) | Shared among all threads |
| LIFO structure | No specific order |
| Fast access | Relatively slower access |
| Size limited | Larger size |
| Automatic memory management | Managed by garbage collector |

### Q23: What causes OutOfMemoryError?
**Answer:**
1. **Java heap space**: Heap is full, can't allocate new objects
2. **GC overhead limit exceeded**: GC taking >98% of time
3. **Metaspace**: Class metadata space exhausted
4. **Direct buffer memory**: NIO direct buffers exhausted
5. **Unable to create new native thread**: OS limit reached

**Solutions:**
- Increase heap size (-Xmx)
- Fix memory leaks
- Optimize data structures
- Use memory profilers

### Q24: What is memory leak in Java? How to detect?
**Answer:**
Memory leak occurs when objects are no longer needed but still referenced, preventing GC from reclaiming memory.

**Common causes:**
- Static collections holding references
- Unclosed resources (streams, connections)
- Listeners not removed
- ThreadLocal variables not cleared

**Detection tools:**
- VisualVM, JProfiler, YourKit
- Heap dumps analysis
- JConsole, Java Mission Control
- Memory profilers

## Garbage Collection

### Q25: Explain different types of garbage collectors.
**Answer:**
1. **Serial GC** (-XX:+UseSerialGC): Single-threaded, suitable for small applications
2. **Parallel GC** (-XX:+UseParallelGC): Multi-threaded, default for server JVMs
3. **G1GC** (-XX:+UseG1GC): Low-latency collector for large heaps
4. **ZGC** (-XX:+UseZGC): Ultra-low latency (<10ms pauses)
5. **Shenandoah** (-XX:+UseShenandoahGC): Concurrent, low-pause collector

### Q26: What is Stop-the-World (STW)?
**Answer:**
STW is when all application threads are paused for GC to run safely. Different collectors minimize STW differently:
- Serial/Parallel: Longer STW pauses
- G1: Predictable pause times
- ZGC/Shenandoah: Concurrent collection, minimal STW

### Q27: Explain strong, weak, soft, and phantom references.
**Answer:**
1. **Strong Reference**: Normal reference, prevents GC
```java
Object obj = new Object();  // Strong reference
```
2. **Weak Reference**: GC can collect if only weak refs exist
```java
WeakReference<Object> weak = new WeakReference<>(obj);
```
3. **Soft Reference**: GC collects before OutOfMemoryError
```java
SoftReference<Object> soft = new SoftReference<>(obj);
```
4. **Phantom Reference**: Enqueued after finalization, before reclamation
```java
PhantomReference<Object> phantom = new PhantomReference<>(obj, queue);
```

## Java 8+ Features

### Q28: What are the main features introduced in Java 8?
**Answer:**
1. **Lambda Expressions**: Functional programming support
2. **Stream API**: Functional operations on collections
3. **Optional**: Avoid null pointer exceptions
4. **Default Methods**: Interface evolution
5. **Method References**: Shorthand for lambdas
6. **CompletableFuture**: Asynchronous programming
7. **New Date/Time API**: java.time package
8. **Functional Interfaces**: @FunctionalInterface

### Q29: Explain Stream API operations.
**Answer:**
**Intermediate Operations** (lazy, return Stream):
- `filter()`: Select elements matching predicate
- `map()`: Transform elements
- `flatMap()`: Flatten nested structures
- `distinct()`: Remove duplicates
- `sorted()`: Sort elements
- `limit()/skip()`: Truncate stream

**Terminal Operations** (trigger processing):
- `collect()`: Accumulate into collection
- `forEach()`: Perform action on each element
- `reduce()`: Combine elements into single result
- `count()`: Count elements
- `anyMatch()/allMatch()/noneMatch()`: Check conditions

### Q30: What are Records (Java 14+)?
**Answer:**
Records are immutable data carriers with automatic implementation of:
- Constructor
- Getters
- equals(), hashCode(), toString()

```java
public record Person(String name, int age) {}
// Automatically gets: name(), age(), equals(), hashCode(), toString()
```

Use cases: DTOs, value objects, immutable data holders

### Q31: Explain Sealed Classes (Java 17+).
**Answer:**
Sealed classes restrict which classes can extend/implement them:
```java
public sealed class Shape permits Circle, Rectangle, Triangle {}
final class Circle extends Shape {}
final class Rectangle extends Shape {}
non-sealed class Triangle extends Shape {}
```

Benefits:
- Exhaustive pattern matching
- Better API design
- Controlled inheritance hierarchy

## Exception Handling

### Q32: Explain the exception hierarchy in Java.
**Answer:**
```
Throwable
├── Error (unchecked)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
└── Exception
    ├── RuntimeException (unchecked)
    │   ├── NullPointerException
    │   ├── IllegalArgumentException
    │   └── IndexOutOfBoundsException
    └── Checked Exceptions
        ├── IOException
        ├── SQLException
        └── ClassNotFoundException
```

### Q33: What is try-with-resources?
**Answer:**
Automatic resource management for AutoCloseable resources:
```java
try (FileReader reader = new FileReader("file.txt");
     BufferedReader br = new BufferedReader(reader)) {
    // Use resources
} catch (IOException e) {
    // Handle exception
}
// Resources automatically closed
```

### Q34: Explain the difference between throw and throws.
**Answer:**
- **throw**: Used to explicitly throw an exception
```java
throw new IllegalArgumentException("Invalid input");
```
- **throws**: Declares exceptions a method might throw
```java
public void readFile() throws IOException {
    // Method that might throw IOException
}
```

## Data Structures

### Q35: Implement a LRU Cache.
**Answer:**
```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);  // true for access-order
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

### Q36: How would you implement a thread-safe Singleton?
**Answer:**
**1. Eager Initialization:**
```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() { return INSTANCE; }
}
```

**2. Double-Checked Locking:**
```java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**3. Enum Singleton (Recommended):**
```java
public enum Singleton {
    INSTANCE;
    public void doSomething() { }
}
```

### Q37: What is the difference between fail-fast and fail-safe iterators?
**Answer:**
**Fail-fast** (ArrayList, HashMap):
- Throws ConcurrentModificationException if collection modified during iteration
- Works on original collection
- Low memory overhead

**Fail-safe** (CopyOnWriteArrayList, ConcurrentHashMap):
- Doesn't throw exception on modification
- Works on copy/snapshot
- Higher memory overhead
- May not reflect latest changes

## Performance and Optimization

### Q38: How to tune JVM performance?
**Answer:**
1. **Heap Size**: -Xms (initial), -Xmx (maximum)
2. **GC Selection**: Choose appropriate collector
3. **GC Logging**: -Xlog:gc* (Java 9+)
4. **Metaspace**: -XX:MaxMetaspaceSize
5. **Thread Stack Size**: -Xss
6. **JIT Compilation**: -XX:CompileThreshold

### Q39: What are best practices for Java performance?
**Answer:**
1. Use StringBuilder for string concatenation in loops
2. Prefer primitives over wrapper classes
3. Use lazy initialization where appropriate
4. Cache expensive computations
5. Use appropriate collection types
6. Minimize synchronization scope
7. Use connection pooling
8. Profile before optimizing

### Q40: How to diagnose production issues?
**Answer:**
1. **Thread Dumps**: jstack or kill -3
2. **Heap Dumps**: jmap or -XX:+HeapDumpOnOutOfMemoryError
3. **GC Logs**: Analyze pause times and frequency
4. **Profilers**: JProfiler, YourKit, VisualVM
5. **APM Tools**: AppDynamics, New Relic
6. **JMX Monitoring**: JConsole, Java Mission Control

## Common Coding Problems

### Q41: Reverse a linked list.
**Answer:**
```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode current = head;
    
    while (current != null) {
        ListNode next = current.next;
        current.next = prev;
        prev = current;
        current = next;
    }
    return prev;
}
```

### Q42: Find duplicate in array of n+1 integers.
**Answer:**
```java
// Floyd's Cycle Detection
public int findDuplicate(int[] nums) {
    int slow = nums[0];
    int fast = nums[0];
    
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);
    
    slow = nums[0];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return fast;
}
```

### Q43: Implement a producer-consumer pattern.
**Answer:**
```java
class ProducerConsumer {
    private BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10);
    
    class Producer implements Runnable {
        public void run() {
            try {
                for (int i = 0; i < 100; i++) {
                    queue.put(i);
                    System.out.println("Produced: " + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    class Consumer implements Runnable {
        public void run() {
            try {
                while (true) {
                    Integer item = queue.take();
                    System.out.println("Consumed: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

## Interview Tips

1. **Understand concepts deeply**: Don't just memorize, understand why things work
2. **Practice coding**: Implement data structures and algorithms
3. **Know trade-offs**: Every design decision has pros and cons
4. **Stay updated**: Keep learning new Java features
5. **Debug systematically**: Use proper tools and methodologies
6. **Communicate clearly**: Explain your thought process
7. **Ask clarifying questions**: Don't assume requirements