# Java Multithreading & Concurrency - Deep Dive

## Table of Contents
1. [Thread Fundamentals](#thread-fundamentals)
2. [Thread Creation and Lifecycle](#thread-creation-and-lifecycle)
3. [Synchronization Mechanisms](#synchronization-mechanisms)
4. [Java Memory Model](#java-memory-model)
5. [Concurrent Collections](#concurrent-collections)
6. [Thread Pools and Executors](#thread-pools-and-executors)
7. [Locks and Advanced Synchronization](#locks-and-advanced-synchronization)
8. [Atomic Operations](#atomic-operations)
9. [Thread Communication](#thread-communication)
10. [Deadlocks and Thread Safety](#deadlocks-and-thread-safety)
11. [Fork/Join Framework](#forkjoin-framework)
12. [CompletableFuture and Async Programming](#completablefuture-and-async-programming)
13. [Best Practices and Patterns](#best-practices-and-patterns)
14. [Performance Optimization](#performance-optimization)
15. [Common Interview Questions](#common-interview-questions)

---

## Thread Fundamentals

### Q: What is multithreading and why is it important?

**Answer:**
Multithreading is the concurrent execution of multiple threads within a single program. Each thread represents an independent path of execution with its own call stack but shares the process's memory space.

**Importance:**
- **Better CPU Utilization**: Prevents idle CPU time during I/O operations
- **Improved Responsiveness**: UI remains responsive while performing background tasks
- **Parallel Processing**: Utilizes multiple CPU cores for computation-intensive tasks
- **Resource Fairness**: Multiple users/requests can be served simultaneously

```java
// Single-threaded approach - blocks during I/O
public void processFiles(List<File> files) {
    for (File file : files) {
        readFile(file);      // Blocks
        processData();       // CPU waits during I/O
        writeResults();      // Sequential processing
    }
}

// Multi-threaded approach - efficient resource usage
public void processFilesParallel(List<File> files) {
    ExecutorService executor = Executors.newFixedThreadPool(4);
    files.forEach(file -> 
        executor.submit(() -> {
            readFile(file);
            processData();
            writeResults();
        })
    );
}
```

### Q: What's the difference between process and thread?

**Answer:**

| Aspect | Process | Thread |
|--------|---------|--------|
| Memory | Independent memory space | Shared memory within process |
| Creation | Heavy, OS-level operation | Lightweight, faster creation |
| Communication | IPC (pipes, sockets) | Direct memory access |
| Isolation | Complete isolation | Shares resources |
| Overhead | High memory and time | Low overhead |
| Failure Impact | Isolated failures | Can affect entire process |

```java
// Process creation (heavy)
ProcessBuilder pb = new ProcessBuilder("java", "-jar", "app.jar");
Process process = pb.start();

// Thread creation (lightweight)
Thread thread = new Thread(() -> {
    // Shares memory with parent
    sharedVariable++;
});
thread.start();
```

---

## Thread Creation and Lifecycle

### Q: What are the different ways to create threads in Java?

**Answer:**

**1. Extending Thread Class:**
```java
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread: " + Thread.currentThread().getName());
    }
}

// Usage
MyThread thread = new MyThread();
thread.start();
```

**2. Implementing Runnable Interface (Preferred):**
```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable: " + Thread.currentThread().getName());
    }
}

// Usage
Thread thread = new Thread(new MyRunnable());
thread.start();

// Lambda expression
Thread thread = new Thread(() -> 
    System.out.println("Lambda: " + Thread.currentThread().getName())
);
```

**3. Implementing Callable Interface:**
```java
public class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        // Can return value and throw checked exceptions
        return ThreadLocalRandom.current().nextInt(100);
    }
}

// Usage with ExecutorService
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(new MyCallable());
Integer result = future.get(); // Blocks until complete
```

**4. Using Thread Pools:**
```java
ExecutorService executor = Executors.newFixedThreadPool(5);

// Submit Runnable
executor.execute(() -> System.out.println("Task executed"));

// Submit Callable
Future<String> future = executor.submit(() -> "Result");
```

### Q: Explain thread lifecycle and states in Java

**Answer:**

Thread states in Java:

1. **NEW**: Thread created but not started
2. **RUNNABLE**: Executing or ready to execute
3. **BLOCKED**: Waiting for monitor lock
4. **WAITING**: Waiting indefinitely for another thread
5. **TIMED_WAITING**: Waiting for specified period
6. **TERMINATED**: Execution completed

```java
public class ThreadLifecycleDemo {
    public static void main(String[] args) throws InterruptedException {
        Object lock = new Object();
        
        Thread thread = new Thread(() -> {
            synchronized (lock) {
                try {
                    lock.wait(); // WAITING state
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });
        
        System.out.println("State after creation: " + thread.getState()); // NEW
        
        thread.start();
        Thread.sleep(100);
        System.out.println("State after start: " + thread.getState()); // WAITING
        
        synchronized (lock) {
            lock.notify(); // Wake up waiting thread
        }
        
        thread.join();
        System.out.println("State after completion: " + thread.getState()); // TERMINATED
    }
}
```

---

## Synchronization Mechanisms

### Q: What is the synchronized keyword and how does it work?

**Answer:**

The `synchronized` keyword provides exclusive access to shared resources, preventing race conditions through mutual exclusion.

**Method-level Synchronization:**
```java
public class Counter {
    private int count = 0;
    
    // Entire method is synchronized
    public synchronized void increment() {
        count++;
    }
    
    // Equivalent to:
    public void increment() {
        synchronized(this) {
            count++;
        }
    }
    
    // Static method synchronization
    public static synchronized void staticMethod() {
        // Locks on Class object
    }
}
```

**Block-level Synchronization (Preferred for fine-grained control):**
```java
public class BankAccount {
    private double balance;
    private final Object lock = new Object(); // Dedicated lock object
    
    public void transfer(BankAccount to, double amount) {
        // Only synchronize critical section
        synchronized(lock) {
            if (balance >= amount) {
                balance -= amount;
                to.deposit(amount);
            }
        }
        
        // Non-critical operations outside synchronized block
        logTransaction(amount);
    }
    
    // Avoid nested locks to prevent deadlock
    public void deposit(double amount) {
        synchronized(lock) {
            balance += amount;
        }
    }
}
```

### Q: What is volatile and when should you use it?

**Answer:**

`volatile` ensures visibility of changes across threads and prevents instruction reordering, but doesn't provide atomicity.

**Use Cases:**
1. **Status flags**
2. **One writer, multiple readers**
3. **Double-checked locking**

```java
public class VolatileExample {
    // Without volatile, changes might not be visible to other threads
    private volatile boolean running = true;
    private volatile int counter = 0; // Still not thread-safe for increment!
    
    public void stopProcessing() {
        running = false; // Immediately visible to all threads
    }
    
    public void process() {
        while (running) { // Always reads from main memory
            // Process data
        }
    }
    
    // Double-checked locking pattern
    private volatile Singleton instance;
    
    public Singleton getInstance() {
        if (instance == null) {
            synchronized(this) {
                if (instance == null) {
                    instance = new Singleton(); // volatile prevents reordering
                }
            }
        }
        return instance;
    }
}
```

**volatile vs synchronized:**
- `volatile`: Only visibility guarantee, no mutual exclusion
- `synchronized`: Both visibility and mutual exclusion
- `volatile` is lighter but limited to simple read/write operations

---

## Java Memory Model

### Q: Explain the Java Memory Model (JMM) and happens-before relationship

**Answer:**

The JMM defines how threads interact through memory and what behaviors are allowed in concurrent execution.

**Key Concepts:**

1. **Thread Stack vs Heap:**
```java
public class MemoryModel {
    private int sharedVar = 0;        // Heap (shared)
    private static int staticVar = 0; // Heap (shared)
    
    public void method() {
        int localVar = 0;              // Thread stack (private)
        MyObject obj = new MyObject();  // Reference in stack, object in heap
    }
}
```

2. **Happens-Before Guarantees:**
```java
public class HappensBeforeExample {
    private volatile int v = 0;
    private int a = 0;
    private int b = 0;
    
    // Thread 1
    public void writer() {
        a = 1;      // 1
        b = 2;      // 2
        v = 3;      // 3 - volatile write
    }
    
    // Thread 2
    public void reader() {
        int r1 = v; // 4 - volatile read
        // If r1 == 3, then guaranteed:
        // a == 1 and b == 2 (happens-before relationship)
        int r2 = a; // 5
        int r3 = b; // 6
    }
}
```

3. **Memory Barriers:**
```java
public class MemoryBarrierExample {
    private volatile boolean flag = false;
    private int data = 0;
    
    // Writer thread
    public void write() {
        data = 42;          // Happens-before volatile write
        flag = true;        // Volatile write - memory barrier
    }
    
    // Reader thread
    public void read() {
        if (flag) {         // Volatile read - memory barrier
            // Guaranteed to see data = 42
            System.out.println(data);
        }
    }
}
```

### Q: What are the key happens-before rules?

**Answer:**

1. **Program Order Rule**: Each action in a thread happens-before every subsequent action in that thread
2. **Monitor Lock Rule**: Unlock happens-before subsequent lock
3. **Volatile Variable Rule**: Write happens-before subsequent read
4. **Thread Start Rule**: Thread.start() happens-before any action in the started thread
5. **Thread Join Rule**: Actions in thread happen-before Thread.join() returns
6. **Transitivity**: If A happens-before B, and B happens-before C, then A happens-before C

```java
public class HappensBeforeRules {
    private int x = 0;
    private volatile boolean ready = false;
    
    // Demonstrates multiple happens-before relationships
    public void demonstrate() throws InterruptedException {
        Thread thread = new Thread(() -> {
            x = 42;              // 1
            ready = true;        // 2 - volatile write
        });
        
        thread.start();          // Thread start rule
        
        while (!ready) {         // 3 - volatile read
            Thread.yield();
        }
        
        System.out.println(x);   // 4 - Guaranteed to print 42
        
        thread.join();           // Thread join rule
        // All actions in thread happened-before this point
    }
}
```

---

## Concurrent Collections

### Q: What are the thread-safe collections in Java?

**Answer:**

**1. Legacy Synchronized Collections:**
```java
// Synchronized wrappers - coarse-grained locking
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
Set<String> syncSet = Collections.synchronizedSet(new HashSet<>());

// Vector and Hashtable - legacy, avoid using
Vector<String> vector = new Vector<>();
Hashtable<String, String> hashtable = new Hashtable<>();
```

**2. Concurrent Collections (Preferred):**
```java
// ConcurrentHashMap - segment-based locking
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("key", 1);
concurrentMap.compute("key", (k, v) -> v == null ? 1 : v + 1); // Atomic operation

// CopyOnWriteArrayList - for read-heavy scenarios
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
cowList.add("item"); // Creates new copy of underlying array

// ConcurrentLinkedQueue - non-blocking queue
ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();
queue.offer("item");
String item = queue.poll();

// BlockingQueue implementations
BlockingQueue<String> blockingQueue = new LinkedBlockingQueue<>(100);
blockingQueue.put("item"); // Blocks if full
String taken = blockingQueue.take(); // Blocks if empty

// ConcurrentSkipListMap - sorted, concurrent NavigableMap
ConcurrentSkipListMap<Integer, String> skipListMap = new ConcurrentSkipListMap<>();
skipListMap.put(1, "one");
```

### Q: Explain ConcurrentHashMap internals and improvements in Java 8

**Answer:**

**Java 7 and earlier - Segment-based locking:**
```java
// Divided into 16 segments by default
// Each segment is independently locked
// Allows 16 concurrent writers
```

**Java 8+ improvements:**
```java
public class ConcurrentHashMapDemo {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        
        // 1. Node-based locking (fine-grained)
        // Uses synchronized blocks on first node of bucket
        
        // 2. Red-Black tree for large buckets
        // Converts to tree when bucket size > 8
        
        // 3. Atomic operations
        map.compute("key", (k, v) -> v == null ? 1 : v + 1);
        map.merge("key", 1, Integer::sum);
        
        // 4. Parallel operations with custom parallelism
        long count = map.mappingCount(); // More accurate than size()
        
        // 5. Bulk operations
        map.forEach(1, (k, v) -> System.out.println(k + "=" + v));
        
        // 6. Search operations
        String result = map.search(1, (k, v) -> v > 10 ? k : null);
        
        // 7. Reduction operations
        Integer sum = map.reduce(1, 
            (k, v) -> v,
            Integer::sum);
    }
}
```

**Performance Comparison:**
```java
// Benchmark example
public class MapPerformanceTest {
    private static final int THREADS = 16;
    private static final int OPERATIONS = 1_000_000;
    
    public static void testConcurrentHashMap() {
        ConcurrentHashMap<Integer, Integer> map = new ConcurrentHashMap<>();
        ExecutorService executor = Executors.newFixedThreadPool(THREADS);
        
        long start = System.currentTimeMillis();
        
        for (int i = 0; i < THREADS; i++) {
            executor.submit(() -> {
                for (int j = 0; j < OPERATIONS; j++) {
                    map.put(ThreadLocalRandom.current().nextInt(1000), j);
                }
            });
        }
        
        executor.shutdown();
        // ConcurrentHashMap: ~2x faster than synchronized HashMap
    }
}
```

---

## Thread Pools and Executors

### Q: Explain the Executor framework and different types of thread pools

**Answer:**

The Executor framework decouples task submission from execution mechanics:

```java
public class ExecutorTypes {
    
    // 1. Fixed Thread Pool - bounded number of threads
    public static void fixedThreadPoolExample() {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        // Core threads: 5, Max threads: 5
        // Queue: LinkedBlockingQueue (unbounded)
        // Use case: Known workload, controlled resource usage
        
        for (int i = 0; i < 10; i++) {
            executor.submit(() -> {
                System.out.println(Thread.currentThread().getName());
            });
        }
    }
    
    // 2. Cached Thread Pool - creates threads as needed
    public static void cachedThreadPoolExample() {
        ExecutorService executor = Executors.newCachedThreadPool();
        // Core threads: 0, Max threads: Integer.MAX_VALUE
        // Queue: SynchronousQueue
        // Threads idle for 60s are terminated
        // Use case: Short-lived async tasks
    }
    
    // 3. Single Thread Executor - single worker thread
    public static void singleThreadExecutorExample() {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        // Guarantees sequential execution
        // Use case: Tasks that must run sequentially
    }
    
    // 4. Scheduled Thread Pool - scheduled/periodic tasks
    public static void scheduledThreadPoolExample() {
        ScheduledExecutorService scheduler = 
            Executors.newScheduledThreadPool(3);
        
        // Schedule once after delay
        scheduler.schedule(() -> System.out.println("Delayed task"), 
            5, TimeUnit.SECONDS);
        
        // Schedule periodic task
        scheduler.scheduleAtFixedRate(() -> System.out.println("Periodic"), 
            0, 2, TimeUnit.SECONDS);
        
        // Schedule with fixed delay between executions
        scheduler.scheduleWithFixedDelay(() -> System.out.println("Fixed delay"), 
            0, 3, TimeUnit.SECONDS);
    }
    
    // 5. Work Stealing Pool (Java 8+) - ForkJoinPool
    public static void workStealingPoolExample() {
        ExecutorService executor = Executors.newWorkStealingPool();
        // Creates pool with parallelism = available processors
        // Each thread has its own deque
        // Idle threads steal work from busy threads
        // Use case: Recursive, divide-and-conquer algorithms
    }
}
```

### Q: How do you properly configure a custom ThreadPoolExecutor?

**Answer:**

```java
public class CustomThreadPoolConfiguration {
    
    public static ThreadPoolExecutor createOptimalPool() {
        int corePoolSize = Runtime.getRuntime().availableProcessors();
        int maximumPoolSize = corePoolSize * 2;
        long keepAliveTime = 60L;
        TimeUnit unit = TimeUnit.SECONDS;
        
        // Choose queue strategy based on requirements
        BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(1000);
        // ArrayBlockingQueue - bounded, FIFO
        // LinkedBlockingQueue - optionally bounded
        // SynchronousQueue - direct handoff
        // PriorityBlockingQueue - priority-based
        
        // Custom thread factory for naming and daemon status
        ThreadFactory threadFactory = new ThreadFactory() {
            private final AtomicInteger counter = new AtomicInteger(0);
            
            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r);
                thread.setName("CustomPool-Thread-" + counter.incrementAndGet());
                thread.setDaemon(false);
                thread.setPriority(Thread.NORM_PRIORITY);
                return thread;
            }
        };
        
        // Rejection policies
        RejectedExecutionHandler rejectionHandler = 
            new ThreadPoolExecutor.CallerRunsPolicy();
        // CallerRunsPolicy - caller thread executes task
        // AbortPolicy - throws RejectedExecutionException
        // DiscardPolicy - silently discards
        // DiscardOldestPolicy - discards oldest unhandled
        
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            corePoolSize,
            maximumPoolSize,
            keepAliveTime,
            unit,
            workQueue,
            threadFactory,
            rejectionHandler
        );
        
        // Allow core threads to timeout
        executor.allowCoreThreadTimeOut(true);
        
        // Pre-start core threads for immediate availability
        executor.prestartAllCoreThreads();
        
        return executor;
    }
    
    // Monitoring and management
    public static void monitorThreadPool(ThreadPoolExecutor executor) {
        ScheduledExecutorService monitor = 
            Executors.newSingleThreadScheduledExecutor();
        
        monitor.scheduleAtFixedRate(() -> {
            System.out.println("Pool Size: " + executor.getPoolSize());
            System.out.println("Active Threads: " + executor.getActiveCount());
            System.out.println("Completed Tasks: " + executor.getCompletedTaskCount());
            System.out.println("Total Tasks: " + executor.getTaskCount());
            System.out.println("Queue Size: " + executor.getQueue().size());
        }, 0, 5, TimeUnit.SECONDS);
    }
}
```

---

## Locks and Advanced Synchronization

### Q: Explain ReentrantLock and how it differs from synchronized

**Answer:**

ReentrantLock provides more flexibility than synchronized:

```java
public class ReentrantLockExample {
    private final ReentrantLock lock = new ReentrantLock(true); // fair lock
    private int counter = 0;
    
    // Basic usage - equivalent to synchronized
    public void increment() {
        lock.lock();
        try {
            counter++;
        } finally {
            lock.unlock(); // Must be in finally block
        }
    }
    
    // Try lock with timeout
    public boolean tryIncrement() throws InterruptedException {
        if (lock.tryLock(5, TimeUnit.SECONDS)) {
            try {
                counter++;
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false;
    }
    
    // Interruptible lock acquisition
    public void interruptibleOperation() throws InterruptedException {
        lock.lockInterruptibly(); // Can be interrupted while waiting
        try {
            // Critical section
        } finally {
            lock.unlock();
        }
    }
    
    // Condition variables for complex coordination
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();
    private final Queue<String> queue = new LinkedList<>();
    private final int capacity = 10;
    
    public void produce(String item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await(); // Release lock and wait
            }
            queue.offer(item);
            notEmpty.signal(); // Wake up one consumer
        } finally {
            lock.unlock();
        }
    }
    
    public String consume() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();
            }
            String item = queue.poll();
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

**Comparison Table:**

| Feature | synchronized | ReentrantLock |
|---------|-------------|---------------|
| Implicit locking | Yes | No (explicit lock/unlock) |
| Try lock with timeout | No | Yes |
| Interruptible | No | Yes |
| Fairness option | No | Yes |
| Multiple conditions | No | Yes |
| Lock status query | No | Yes |
| Performance | Good | Slightly better under contention |

### Q: What is ReadWriteLock and when should you use it?

**Answer:**

ReadWriteLock allows multiple concurrent readers but exclusive writers:

```java
public class ReadWriteLockExample {
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();
    private Map<String, String> cache = new HashMap<>();
    
    // Multiple threads can read simultaneously
    public String get(String key) {
        readLock.lock();
        try {
            return cache.get(key);
        } finally {
            readLock.unlock();
        }
    }
    
    // Exclusive write access
    public void put(String key, String value) {
        writeLock.lock();
        try {
            cache.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }
    
    // Write lock can be downgraded to read lock
    public String putAndGet(String key, String value) {
        writeLock.lock();
        try {
            cache.put(key, value);
            // Downgrade to read lock before releasing write lock
            readLock.lock();
        } finally {
            writeLock.unlock();
        }
        
        try {
            return cache.get(key);
        } finally {
            readLock.unlock();
        }
    }
}

// StampedLock (Java 8+) - optimistic reading
public class StampedLockExample {
    private final StampedLock stampedLock = new StampedLock();
    private double x, y;
    
    public void move(double deltaX, double deltaY) {
        long stamp = stampedLock.writeLock();
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            stampedLock.unlockWrite(stamp);
        }
    }
    
    public double distanceFromOrigin() {
        // Try optimistic read first
        long stamp = stampedLock.tryOptimisticRead();
        double currentX = x;
        double currentY = y;
        
        // Check if values are still valid
        if (!stampedLock.validate(stamp)) {
            // Upgrade to read lock
            stamp = stampedLock.readLock();
            try {
                currentX = x;
                currentY = y;
            } finally {
                stampedLock.unlockRead(stamp);
            }
        }
        
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
}
```

---

## Atomic Operations

### Q: Explain atomic classes and their use cases

**Answer:**

Atomic classes provide lock-free thread-safe operations using low-level atomic hardware primitives:

```java
public class AtomicOperationsExample {
    
    // 1. AtomicInteger/Long/Boolean
    private AtomicInteger counter = new AtomicInteger(0);
    private AtomicLong sequenceNumber = new AtomicLong(0);
    private AtomicBoolean flag = new AtomicBoolean(false);
    
    public void demonstrateAtomicInteger() {
        // Atomic increment
        int newValue = counter.incrementAndGet(); // ++counter
        int oldValue = counter.getAndIncrement(); // counter++
        
        // Atomic addition
        int result = counter.addAndGet(5);
        
        // Compare and set
        int expected = 10;
        int update = 20;
        boolean success = counter.compareAndSet(expected, update);
        
        // Update with function (Java 8+)
        counter.updateAndGet(x -> x * 2);
        counter.accumulateAndGet(5, (x, y) -> x + y);
    }
    
    // 2. AtomicReference for object references
    private AtomicReference<String> atomicString = 
        new AtomicReference<>("initial");
    
    public void updateReference() {
        String oldValue = atomicString.getAndSet("new value");
        
        // CAS operation for objects
        atomicString.compareAndSet("expected", "updated");
        
        // Update with function
        atomicString.updateAndGet(s -> s.toUpperCase());
    }
    
    // 3. AtomicStampedReference - solves ABA problem
    private AtomicStampedReference<Integer> stampedRef = 
        new AtomicStampedReference<>(100, 0);
    
    public void demonstrateStampedReference() {
        int[] stampHolder = new int[1];
        Integer value = stampedRef.get(stampHolder);
        int stamp = stampHolder[0];
        
        // Update with new stamp
        stampedRef.compareAndSet(value, value + 100, stamp, stamp + 1);
    }
    
    // 4. AtomicArray classes
    private AtomicIntegerArray atomicArray = new AtomicIntegerArray(10);
    
    public void demonstrateAtomicArray() {
        atomicArray.set(0, 100);
        int oldValue = atomicArray.getAndIncrement(0);
        atomicArray.compareAndSet(0, 101, 200);
    }
    
    // 5. Field updaters for memory efficiency
    private static final AtomicIntegerFieldUpdater<AtomicOperationsExample> 
        UPDATER = AtomicIntegerFieldUpdater.newUpdater(
            AtomicOperationsExample.class, "volatileField");
    
    private volatile int volatileField = 0;
    
    public void updateField() {
        UPDATER.incrementAndGet(this);
    }
    
    // 6. LongAdder/DoubleAdder for high contention (Java 8+)
    private LongAdder adder = new LongAdder();
    
    public void demonstrateLongAdder() {
        // Better performance under high contention
        adder.increment();
        adder.add(10);
        long sum = adder.sum(); // Not strongly consistent
    }
}
```

### Q: How do atomic operations work internally?

**Answer:**

Atomic operations use Compare-And-Swap (CAS) at the hardware level:

```java
public class CASInternals {
    
    // Simplified CAS implementation concept
    public class SimplifiedAtomicInteger {
        private volatile int value;
        
        // Pseudocode for CAS operation
        public final boolean compareAndSet(int expect, int update) {
            // This is done atomically at hardware level
            // Using CPU instruction like CMPXCHG on x86
            if (value == expect) {
                value = update;
                return true;
            }
            return false;
        }
        
        // Increment implementation using CAS loop
        public final int incrementAndGet() {
            int current, next;
            do {
                current = value;
                next = current + 1;
            } while (!compareAndSet(current, next));
            return next;
        }
    }
    
    // Memory barriers and happens-before
    public void demonstrateMemoryEffects() {
        AtomicInteger atomic = new AtomicInteger(0);
        
        // Thread 1
        int x = 1;
        atomic.set(2); // Release semantics - stores before this are visible
        
        // Thread 2
        int y = atomic.get(); // Acquire semantics
        // If y == 2, then x == 1 is guaranteed to be visible
    }
}
```

---

## Thread Communication

### Q: Explain wait(), notify(), and notifyAll()

**Answer:**

These methods enable thread coordination through monitor locks:

```java
public class WaitNotifyExample {
    private final Object lock = new Object();
    private boolean condition = false;
    
    // Producer thread
    public void produce() {
        synchronized (lock) {
            // Do some work
            condition = true;
            lock.notify(); // Wake up one waiting thread
            // lock.notifyAll(); // Wake up all waiting threads
        }
    }
    
    // Consumer thread
    public void consume() throws InterruptedException {
        synchronized (lock) {
            while (!condition) { // Always use while, not if
                lock.wait(); // Releases lock and waits
            }
            // Condition is true, proceed
            condition = false;
        }
    }
    
    // Classic Producer-Consumer pattern
    public class BoundedBuffer<T> {
        private final Queue<T> queue = new LinkedList<>();
        private final int capacity;
        
        public BoundedBuffer(int capacity) {
            this.capacity = capacity;
        }
        
        public synchronized void put(T item) throws InterruptedException {
            while (queue.size() == capacity) {
                wait(); // Buffer full, wait for space
            }
            queue.offer(item);
            notifyAll(); // Notify consumers
        }
        
        public synchronized T take() throws InterruptedException {
            while (queue.isEmpty()) {
                wait(); // Buffer empty, wait for items
            }
            T item = queue.poll();
            notifyAll(); // Notify producers
            return item;
        }
    }
}
```

### Q: What are CountDownLatch, CyclicBarrier, and Semaphore?

**Answer:**

**1. CountDownLatch - One-time synchronization:**
```java
public class CountDownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        int numberOfWorkers = 5;
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(numberOfWorkers);
        
        // Start worker threads
        for (int i = 0; i < numberOfWorkers; i++) {
            new Thread(() -> {
                try {
                    startSignal.await(); // Wait for start signal
                    doWork();
                    doneSignal.countDown(); // Signal completion
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }).start();
        }
        
        // Start all workers
        startSignal.countDown();
        
        // Wait for all workers to complete
        doneSignal.await();
        System.out.println("All workers completed");
    }
    
    private static void doWork() {
        // Simulate work
    }
}
```

**2. CyclicBarrier - Reusable synchronization point:**
```java
public class CyclicBarrierExample {
    public static void main(String[] args) {
        int numberOfThreads = 4;
        CyclicBarrier barrier = new CyclicBarrier(numberOfThreads, () -> {
            // Barrier action - executed when all threads reach barrier
            System.out.println("All threads reached barrier, proceeding...");
        });
        
        for (int i = 0; i < numberOfThreads; i++) {
            new Thread(() -> {
                try {
                    // Phase 1
                    doPhase1Work();
                    barrier.await(); // Wait for all threads
                    
                    // Phase 2
                    doPhase2Work();
                    barrier.await(); // Reuse barrier
                    
                } catch (InterruptedException | BrokenBarrierException e) {
                    Thread.currentThread().interrupt();
                }
            }).start();
        }
    }
    
    private static void doPhase1Work() { }
    private static void doPhase2Work() { }
}
```

**3. Semaphore - Resource pool management:**
```java
public class SemaphoreExample {
    private final Semaphore semaphore = new Semaphore(3); // 3 permits
    private final Set<Connection> connections = new HashSet<>();
    
    // Connection pool with limited resources
    public Connection acquireConnection() throws InterruptedException {
        semaphore.acquire(); // Blocks if no permits available
        synchronized (connections) {
            Connection conn = createConnection();
            connections.add(conn);
            return conn;
        }
    }
    
    public void releaseConnection(Connection conn) {
        synchronized (connections) {
            connections.remove(conn);
        }
        semaphore.release(); // Return permit
    }
    
    // Try acquire with timeout
    public Connection tryAcquireConnection(long timeout, TimeUnit unit) 
            throws InterruptedException {
        if (semaphore.tryAcquire(timeout, unit)) {
            return createConnection();
        }
        return null;
    }
    
    private Connection createConnection() {
        return new Connection();
    }
    
    class Connection { }
}
```

---

## Deadlocks and Thread Safety

### Q: What is deadlock and how do you prevent it?

**Answer:**

Deadlock occurs when threads are blocked forever, waiting for each other. Four conditions must be present:
1. **Mutual Exclusion**: Resources cannot be shared
2. **Hold and Wait**: Thread holds resources while waiting for others
3. **No Preemption**: Resources cannot be forcibly taken
4. **Circular Wait**: Circular chain of threads waiting for resources

```java
public class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    // Deadlock scenario
    public void method1() {
        synchronized (lock1) {
            System.out.println("Thread 1: Holding lock 1");
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            
            synchronized (lock2) {
                System.out.println("Thread 1: Holding lock 1 & 2");
            }
        }
    }
    
    public void method2() {
        synchronized (lock2) {
            System.out.println("Thread 2: Holding lock 2");
            
            synchronized (lock1) { // Opposite order - deadlock!
                System.out.println("Thread 2: Holding lock 2 & 1");
            }
        }
    }
}
```

**Prevention Strategies:**

```java
public class DeadlockPrevention {
    
    // 1. Lock Ordering - Always acquire locks in same order
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void safeMethod1() {
        synchronized (lock1) {
            synchronized (lock2) {
                // Always lock1 then lock2
            }
        }
    }
    
    public void safeMethod2() {
        synchronized (lock1) {
            synchronized (lock2) {
                // Same order as method1
            }
        }
    }
    
    // 2. Lock Timeout - Use tryLock with timeout
    private final ReentrantLock rLock1 = new ReentrantLock();
    private final ReentrantLock rLock2 = new ReentrantLock();
    
    public boolean tryLockBoth() throws InterruptedException {
        boolean lock1Acquired = false;
        boolean lock2Acquired = false;
        
        try {
            lock1Acquired = rLock1.tryLock(5, TimeUnit.SECONDS);
            lock2Acquired = rLock2.tryLock(5, TimeUnit.SECONDS);
            
            if (lock1Acquired && lock2Acquired) {
                // Do work
                return true;
            }
        } finally {
            if (lock2Acquired) rLock2.unlock();
            if (lock1Acquired) rLock1.unlock();
        }
        return false;
    }
    
    // 3. Deadlock Detection and Recovery
    public void detectDeadlock() {
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        long[] deadlockedThreadIds = threadMXBean.findDeadlockedThreads();
        
        if (deadlockedThreadIds != null) {
            ThreadInfo[] threadInfos = threadMXBean.getThreadInfo(deadlockedThreadIds);
            for (ThreadInfo threadInfo : threadInfos) {
                System.out.println("Deadlocked thread: " + threadInfo.getThreadName());
            }
        }
    }
}
```

### Q: What makes a class thread-safe?

**Answer:**

Thread-safety ensures correct behavior when accessed by multiple threads:

```java
// 1. Immutability - Inherently thread-safe
public final class ImmutableClass {
    private final String name;
    private final int value;
    private final List<String> list;
    
    public ImmutableClass(String name, int value, List<String> list) {
        this.name = name;
        this.value = value;
        // Defensive copy
        this.list = new ArrayList<>(list);
    }
    
    public List<String> getList() {
        // Return defensive copy
        return new ArrayList<>(list);
    }
}

// 2. Synchronization
public class SynchronizedClass {
    private int counter;
    private final Object lock = new Object();
    
    public void increment() {
        synchronized (lock) {
            counter++;
        }
    }
}

// 3. Thread Confinement
public class ThreadConfinedClass {
    // Each thread has its own copy
    private static ThreadLocal<SimpleDateFormat> dateFormat = 
        ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
    
    public String formatDate(Date date) {
        return dateFormat.get().format(date);
    }
}

// 4. Lock-free algorithms using atomic operations
public class LockFreeStack<T> {
    private AtomicReference<Node<T>> top = new AtomicReference<>();
    
    private static class Node<T> {
        final T item;
        Node<T> next;
        
        Node(T item) {
            this.item = item;
        }
    }
    
    public void push(T item) {
        Node<T> newHead = new Node<>(item);
        Node<T> oldHead;
        do {
            oldHead = top.get();
            newHead.next = oldHead;
        } while (!top.compareAndSet(oldHead, newHead));
    }
    
    public T pop() {
        Node<T> oldHead;
        Node<T> newHead;
        do {
            oldHead = top.get();
            if (oldHead == null) {
                return null;
            }
            newHead = oldHead.next;
        } while (!top.compareAndSet(oldHead, newHead));
        return oldHead.item;
    }
}
```

---

## Fork/Join Framework

### Q: Explain the Fork/Join framework and its use cases

**Answer:**

Fork/Join is designed for recursive divide-and-conquer algorithms:

```java
public class ForkJoinExample {
    
    // RecursiveTask returns a result
    static class FibonacciTask extends RecursiveTask<Long> {
        private final long n;
        
        FibonacciTask(long n) {
            this.n = n;
        }
        
        @Override
        protected Long compute() {
            if (n <= 1) {
                return n;
            }
            
            // Fork subtasks
            FibonacciTask f1 = new FibonacciTask(n - 1);
            f1.fork(); // Execute asynchronously
            
            FibonacciTask f2 = new FibonacciTask(n - 2);
            
            // Compute f2 directly and join f1
            return f2.compute() + f1.join();
        }
    }
    
    // RecursiveAction doesn't return a result
    static class MergeSortTask extends RecursiveAction {
        private final int[] array;
        private final int start;
        private final int end;
        private static final int THRESHOLD = 10;
        
        MergeSortTask(int[] array, int start, int end) {
            this.array = array;
            this.start = start;
            this.end = end;
        }
        
        @Override
        protected void compute() {
            if (end - start <= THRESHOLD) {
                // Sequential sort for small arrays
                Arrays.sort(array, start, end);
                return;
            }
            
            int mid = (start + end) / 2;
            
            // Fork both subtasks
            MergeSortTask left = new MergeSortTask(array, start, mid);
            MergeSortTask right = new MergeSortTask(array, mid, end);
            
            invokeAll(left, right); // Fork and join both
            
            // Merge results
            merge(array, start, mid, end);
        }
        
        private void merge(int[] array, int start, int mid, int end) {
            // Merge implementation
        }
    }
    
    // Parallel stream operations use Fork/Join
    public static void parallelStreamExample() {
        List<Integer> numbers = IntStream.range(0, 1000000)
            .boxed()
            .collect(Collectors.toList());
        
        // Uses Fork/Join pool internally
        long sum = numbers.parallelStream()
            .filter(n -> n % 2 == 0)
            .mapToLong(n -> n * n)
            .sum();
        
        // Custom Fork/Join pool
        ForkJoinPool customPool = new ForkJoinPool(4);
        try {
            long result = customPool.submit(() ->
                numbers.parallelStream()
                    .reduce(0, Integer::sum)
            ).get();
        } catch (InterruptedException | ExecutionException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public static void main(String[] args) {
        ForkJoinPool pool = new ForkJoinPool();
        
        // Submit task
        FibonacciTask task = new FibonacciTask(20);
        Long result = pool.invoke(task);
        System.out.println("Fibonacci(20) = " + result);
        
        // Work stealing demonstration
        System.out.println("Parallelism: " + pool.getParallelism());
        System.out.println("Pool size: " + pool.getPoolSize());
        System.out.println("Steal count: " + pool.getStealCount());
    }
}
```

---

## CompletableFuture and Async Programming

### Q: Explain CompletableFuture and asynchronous programming patterns

**Answer:**

CompletableFuture enables composable asynchronous programming:

```java
public class CompletableFutureExample {
    
    // Basic async operations
    public void basicOperations() {
        // Create completed future
        CompletableFuture<String> completed = 
            CompletableFuture.completedFuture("Result");
        
        // Async computation
        CompletableFuture<Integer> future = CompletableFuture
            .supplyAsync(() -> {
                // Long running operation
                return computeValue();
            });
        
        // With custom executor
        ExecutorService executor = Executors.newFixedThreadPool(10);
        CompletableFuture<Integer> futureWithExecutor = 
            CompletableFuture.supplyAsync(() -> computeValue(), executor);
    }
    
    // Chaining and composition
    public void chainingOperations() {
        CompletableFuture<String> future = CompletableFuture
            .supplyAsync(() -> "Hello")
            .thenApply(s -> s + " World")           // Transform result
            .thenApply(String::toUpperCase)         // Chain transformations
            .thenCompose(s -> CompletableFuture     // Flat map
                .supplyAsync(() -> s + "!"));
        
        // Async vs sync variants
        future.thenApply(s -> s.length());          // Same thread
        future.thenApplyAsync(s -> s.length());     // Different thread
    }
    
    // Combining multiple futures
    public void combiningFutures() {
        CompletableFuture<String> future1 = 
            CompletableFuture.supplyAsync(() -> "Hello");
        CompletableFuture<String> future2 = 
            CompletableFuture.supplyAsync(() -> "World");
        
        // Combine two futures
        CompletableFuture<String> combined = future1
            .thenCombine(future2, (s1, s2) -> s1 + " " + s2);
        
        // Wait for both
        CompletableFuture<Void> both = 
            CompletableFuture.allOf(future1, future2);
        
        // Wait for any
        CompletableFuture<Object> any = 
            CompletableFuture.anyOf(future1, future2);
    }
    
    // Exception handling
    public void exceptionHandling() {
        CompletableFuture<Integer> future = CompletableFuture
            .supplyAsync(() -> {
                if (Math.random() > 0.5) {
                    throw new RuntimeException("Failed");
                }
                return 42;
            })
            .handle((result, ex) -> {
                if (ex != null) {
                    System.out.println("Error: " + ex.getMessage());
                    return -1;
                }
                return result;
            })
            .exceptionally(ex -> {
                // Only called on exception
                return 0;
            });
    }
    
    // Practical example: Parallel API calls
    public CompletableFuture<User> getUserDetails(String userId) {
        CompletableFuture<UserInfo> userInfo = 
            CompletableFuture.supplyAsync(() -> fetchUserInfo(userId));
        
        CompletableFuture<List<Order>> orders = 
            CompletableFuture.supplyAsync(() -> fetchUserOrders(userId));
        
        CompletableFuture<PaymentInfo> payment = 
            CompletableFuture.supplyAsync(() -> fetchPaymentInfo(userId));
        
        return CompletableFuture.allOf(userInfo, orders, payment)
            .thenApply(v -> new User(
                userInfo.join(),
                orders.join(),
                payment.join()
            ));
    }
    
    // Timeout handling
    public void timeoutHandling() {
        CompletableFuture<String> future = CompletableFuture
            .supplyAsync(() -> slowOperation())
            .orTimeout(5, TimeUnit.SECONDS)           // Java 9+
            .completeOnTimeout("Default", 5, TimeUnit.SECONDS); // Java 9+
    }
    
    private int computeValue() { return 42; }
    private UserInfo fetchUserInfo(String id) { return null; }
    private List<Order> fetchUserOrders(String id) { return null; }
    private PaymentInfo fetchPaymentInfo(String id) { return null; }
    private String slowOperation() { return "Result"; }
    
    class User {
        User(UserInfo info, List<Order> orders, PaymentInfo payment) {}
    }
    class UserInfo {}
    class Order {}
    class PaymentInfo {}
}
```

---

## Best Practices and Patterns

### Q: What are the best practices for concurrent programming?

**Answer:**

```java
public class ConcurrencyBestPractices {
    
    // 1. Minimize shared mutable state
    public class MinimizeSharedState {
        // Bad - shared mutable state
        private List<String> sharedList = new ArrayList<>();
        
        // Good - immutable or thread-local
        private final List<String> immutableList = 
            Collections.unmodifiableList(Arrays.asList("a", "b"));
        private final ThreadLocal<List<String>> threadLocalList = 
            ThreadLocal.withInitial(ArrayList::new);
    }
    
    // 2. Use high-level concurrency utilities
    public class UseHighLevelUtilities {
        // Bad - manual thread management
        public void badApproach() {
            Thread thread = new Thread(() -> doWork());
            thread.start();
        }
        
        // Good - executor service
        private final ExecutorService executor = 
            Executors.newCachedThreadPool();
        
        public void goodApproach() {
            executor.submit(() -> doWork());
        }
        
        private void doWork() {}
    }
    
    // 3. Document thread-safety
    @ThreadSafe
    public class DocumentedThreadSafeClass {
        @GuardedBy("this")
        private int counter;
        
        public synchronized void increment() {
            counter++;
        }
    }
    
    // 4. Use immutable objects
    public final class ImmutableData {
        private final String name;
        private final int value;
        private final List<String> items;
        
        public ImmutableData(String name, int value, List<String> items) {
            this.name = name;
            this.value = value;
            this.items = Collections.unmodifiableList(new ArrayList<>(items));
        }
        
        // Only getters, no setters
        public String getName() { return name; }
        public int getValue() { return value; }
        public List<String> getItems() { 
            return Collections.unmodifiableList(items); 
        }
    }
    
    // 5. Avoid nested locks
    public class AvoidNestedLocks {
        private final Lock lock1 = new ReentrantLock();
        private final Lock lock2 = new ReentrantLock();
        
        // Bad - nested locks can cause deadlock
        public void badMethod() {
            lock1.lock();
            try {
                lock2.lock();
                try {
                    // Work
                } finally {
                    lock2.unlock();
                }
            } finally {
                lock1.unlock();
            }
        }
        
        // Good - single lock or lock-free
        private final Lock singleLock = new ReentrantLock();
        
        public void goodMethod() {
            singleLock.lock();
            try {
                // Work
            } finally {
                singleLock.unlock();
            }
        }
    }
    
    // 6. Handle InterruptedException properly
    public class HandleInterruption {
        public void properInterruptHandling() throws InterruptedException {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                // Restore interrupted status
                Thread.currentThread().interrupt();
                // Clean up and propagate
                throw e;
            }
        }
    }
    
    // 7. Use volatile carefully
    public class VolatileUsage {
        // Good - simple flag
        private volatile boolean shutdown = false;
        
        // Bad - compound operations not atomic
        private volatile int counter = 0;
        
        public void badIncrement() {
            counter++; // NOT atomic even with volatile
        }
        
        // Good - use atomic classes for compound operations
        private final AtomicInteger atomicCounter = new AtomicInteger(0);
        
        public void goodIncrement() {
            atomicCounter.incrementAndGet();
        }
    }
}
```

---

## Performance Optimization

### Q: How do you optimize concurrent applications?

**Answer:**

```java
public class ConcurrencyOptimization {
    
    // 1. Choose appropriate thread pool size
    public ExecutorService optimalThreadPool() {
        int cpuCores = Runtime.getRuntime().availableProcessors();
        
        // CPU-bound tasks
        int cpuBoundPoolSize = cpuCores + 1;
        
        // I/O-bound tasks (use Little's Law)
        // Pool size = cores * (1 + wait time / service time)
        int ioBoundPoolSize = cpuCores * 2;
        
        return new ThreadPoolExecutor(
            cpuCores,
            ioBoundPoolSize,
            60L, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(1000)
        );
    }
    
    // 2. Reduce lock contention
    public class LockContention {
        // Bad - coarse-grained locking
        private final Map<String, Integer> map = new HashMap<>();
        
        public synchronized void badUpdate(String key, int value) {
            // Locks entire map for all operations
            map.put(key, value);
        }
        
        // Good - fine-grained locking with ConcurrentHashMap
        private final ConcurrentHashMap<String, Integer> concurrentMap = 
            new ConcurrentHashMap<>();
        
        public void goodUpdate(String key, int value) {
            // Only locks the specific bucket
            concurrentMap.put(key, value);
        }
        
        // Better - lock striping
        private final int stripes = 16;
        private final Object[] locks = new Object[stripes];
        private final Map<String, Integer>[] maps = new HashMap[stripes];
        
        {
            for (int i = 0; i < stripes; i++) {
                locks[i] = new Object();
                maps[i] = new HashMap<>();
            }
        }
        
        public void stripedUpdate(String key, int value) {
            int stripe = key.hashCode() & (stripes - 1);
            synchronized (locks[stripe]) {
                maps[stripe].put(key, value);
            }
        }
    }
    
    // 3. Use lock-free data structures
    public class LockFreeOptimization {
        // Lock-free counter using AtomicLong
        private final AtomicLong counter = new AtomicLong();
        
        // Lock-free queue
        private final ConcurrentLinkedQueue<String> queue = 
            new ConcurrentLinkedQueue<>();
        
        // High-contention counter using LongAdder
        private final LongAdder adder = new LongAdder();
        
        public void increment() {
            adder.increment(); // Better than AtomicLong under contention
        }
    }
    
    // 4. Batch operations
    public class BatchProcessing {
        private final BlockingQueue<Task> queue = 
            new LinkedBlockingQueue<>();
        
        // Process items in batches
        public void processBatch() throws InterruptedException {
            List<Task> batch = new ArrayList<>();
            queue.drainTo(batch, 100); // Get up to 100 items
            
            if (!batch.isEmpty()) {
                processBatchedTasks(batch);
            }
        }
        
        private void processBatchedTasks(List<Task> tasks) {
            // Process all tasks together - more efficient
        }
    }
    
    // 5. Use thread-local storage
    public class ThreadLocalOptimization {
        // Expensive object creation
        private static final ThreadLocal<SimpleDateFormat> dateFormat = 
            ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
        
        public String format(Date date) {
            // No synchronization needed
            return dateFormat.get().format(date);
        }
    }
    
    // 6. Apply Amdahl's Law
    public void amdahlsLaw() {
        // Speedup = 1 / (S + P/N)
        // S = sequential portion
        // P = parallel portion
        // N = number of processors
        
        double sequentialPortion = 0.1; // 10% sequential
        int processors = 8;
        
        double speedup = 1.0 / (sequentialPortion + 
            (1 - sequentialPortion) / processors);
        
        System.out.println("Maximum speedup: " + speedup);
        // Output: ~4.7x (not 8x despite 8 processors)
    }
    
    class Task {}
}
```

### Q: How do you measure and profile concurrent application performance?

**Answer:**

```java
public class ConcurrencyProfiling {
    
    // 1. Basic timing measurement
    public void measureExecutionTime() {
        long startTime = System.nanoTime();
        
        // Execute concurrent operation
        performConcurrentOperation();
        
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        
        System.out.println("Execution time: " + 
            TimeUnit.NANOSECONDS.toMillis(duration) + " ms");
    }
    
    // 2. JMH benchmarking
    @Benchmark
    @BenchmarkMode(Mode.Throughput)
    @OutputTimeUnit(TimeUnit.MILLISECONDS)
    public void benchmarkMethod() {
        // Method to benchmark
    }
    
    // 3. Thread pool monitoring
    public void monitorThreadPool(ThreadPoolExecutor executor) {
        ScheduledExecutorService monitor = 
            Executors.newSingleThreadScheduledExecutor();
        
        monitor.scheduleAtFixedRate(() -> {
            System.out.println("Active threads: " + executor.getActiveCount());
            System.out.println("Pool size: " + executor.getPoolSize());
            System.out.println("Queue size: " + executor.getQueue().size());
            System.out.println("Completed tasks: " + 
                executor.getCompletedTaskCount());
            
            // Calculate utilization
            double utilization = (double) executor.getActiveCount() / 
                executor.getMaximumPoolSize();
            System.out.println("Utilization: " + 
                String.format("%.2f%%", utilization * 100));
        }, 0, 1, TimeUnit.SECONDS);
    }
    
    // 4. Custom performance metrics
    public class PerformanceMetrics {
        private final AtomicLong totalRequests = new AtomicLong();
        private final AtomicLong totalLatency = new AtomicLong();
        private final LongAdder concurrentRequests = new LongAdder();
        
        public void recordRequest(long latencyNanos) {
            totalRequests.incrementAndGet();
            totalLatency.addAndGet(latencyNanos);
        }
        
        public void startRequest() {
            concurrentRequests.increment();
        }
        
        public void endRequest() {
            concurrentRequests.decrement();
        }
        
        public void printMetrics() {
            long requests = totalRequests.get();
            long latency = totalLatency.get();
            
            System.out.println("Total requests: " + requests);
            System.out.println("Average latency: " + 
                (requests > 0 ? latency / requests : 0) + " ns");
            System.out.println("Concurrent requests: " + 
                concurrentRequests.sum());
        }
    }
    
    private void performConcurrentOperation() {}
}
```

---

## Common Interview Questions

### Q: Implement a thread-safe Singleton pattern

**Answer:**

```java
// 1. Eager initialization (thread-safe by JVM)
public class EagerSingleton {
    private static final EagerSingleton INSTANCE = new EagerSingleton();
    
    private EagerSingleton() {
        // Prevent reflection attack
        if (INSTANCE != null) {
            throw new IllegalStateException("Instance already created");
        }
    }
    
    public static EagerSingleton getInstance() {
        return INSTANCE;
    }
}

// 2. Lazy initialization with double-checked locking
public class DoubleCheckedSingleton {
    private static volatile DoubleCheckedSingleton instance;
    
    private DoubleCheckedSingleton() {}
    
    public static DoubleCheckedSingleton getInstance() {
        if (instance == null) {
            synchronized (DoubleCheckedSingleton.class) {
                if (instance == null) {
                    instance = new DoubleCheckedSingleton();
                }
            }
        }
        return instance;
    }
}

// 3. Bill Pugh Solution (recommended)
public class BillPughSingleton {
    private BillPughSingleton() {}
    
    private static class SingletonHelper {
        private static final BillPughSingleton INSTANCE = 
            new BillPughSingleton();
    }
    
    public static BillPughSingleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}

// 4. Enum Singleton (Joshua Bloch's recommendation)
public enum EnumSingleton {
    INSTANCE;
    
    public void doSomething() {
        // Business logic
    }
}
```

### Q: Implement a custom blocking queue

**Answer:**

```java
public class CustomBlockingQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final Object lock = new Object();
    
    public CustomBlockingQueue(int capacity) {
        this.capacity = capacity;
    }
    
    public void put(T item) throws InterruptedException {
        synchronized (lock) {
            while (queue.size() == capacity) {
                lock.wait(); // Wait for space
            }
            queue.offer(item);
            lock.notifyAll(); // Notify consumers
        }
    }
    
    public T take() throws InterruptedException {
        synchronized (lock) {
            while (queue.isEmpty()) {
                lock.wait(); // Wait for items
            }
            T item = queue.poll();
            lock.notifyAll(); // Notify producers
            return item;
        }
    }
    
    // Using ReentrantLock and Conditions (better approach)
    public class BetterBlockingQueue<T> {
        private final Queue<T> queue = new LinkedList<>();
        private final int capacity;
        private final ReentrantLock lock = new ReentrantLock();
        private final Condition notFull = lock.newCondition();
        private final Condition notEmpty = lock.newCondition();
        
        public BetterBlockingQueue(int capacity) {
            this.capacity = capacity;
        }
        
        public void put(T item) throws InterruptedException {
            lock.lock();
            try {
                while (queue.size() == capacity) {
                    notFull.await();
                }
                queue.offer(item);
                notEmpty.signal();
            } finally {
                lock.unlock();
            }
        }
        
        public T take() throws InterruptedException {
            lock.lock();
            try {
                while (queue.isEmpty()) {
                    notEmpty.await();
                }
                T item = queue.poll();
                notFull.signal();
                return item;
            } finally {
                lock.unlock();
            }
        }
    }
}
```

### Q: Implement a rate limiter

**Answer:**

```java
public class RateLimiter {
    // Token bucket algorithm
    private final int maxTokens;
    private final long refillPeriodMillis;
    private final AtomicInteger availableTokens;
    private final AtomicLong lastRefillTime;
    
    public RateLimiter(int maxTokens, long refillPeriodMillis) {
        this.maxTokens = maxTokens;
        this.refillPeriodMillis = refillPeriodMillis;
        this.availableTokens = new AtomicInteger(maxTokens);
        this.lastRefillTime = new AtomicLong(System.currentTimeMillis());
    }
    
    public boolean tryAcquire() {
        refillTokens();
        return availableTokens.getAndUpdate(tokens -> 
            tokens > 0 ? tokens - 1 : tokens) > 0;
    }
    
    private void refillTokens() {
        long now = System.currentTimeMillis();
        long lastRefill = lastRefillTime.get();
        long timeSinceLastRefill = now - lastRefill;
        
        if (timeSinceLastRefill > refillPeriodMillis) {
            int tokensToAdd = (int)(timeSinceLastRefill / refillPeriodMillis);
            if (tokensToAdd > 0) {
                availableTokens.updateAndGet(tokens -> 
                    Math.min(maxTokens, tokens + tokensToAdd));
                lastRefillTime.compareAndSet(lastRefill, now);
            }
        }
    }
}

// Sliding window rate limiter
public class SlidingWindowRateLimiter {
    private final int maxRequests;
    private final long windowSizeMillis;
    private final ConcurrentLinkedDeque<Long> requestTimes = 
        new ConcurrentLinkedDeque<>();
    
    public SlidingWindowRateLimiter(int maxRequests, long windowSizeMillis) {
        this.maxRequests = maxRequests;
        this.windowSizeMillis = windowSizeMillis;
    }
    
    public synchronized boolean tryAcquire() {
        long now = System.currentTimeMillis();
        long windowStart = now - windowSizeMillis;
        
        // Remove old requests outside the window
        while (!requestTimes.isEmpty() && requestTimes.peekFirst() < windowStart) {
            requestTimes.pollFirst();
        }
        
        if (requestTimes.size() < maxRequests) {
            requestTimes.addLast(now);
            return true;
        }
        
        return false;
    }
}
```

### Q: Implement a thread pool from scratch

**Answer:**

```java
public class SimpleThreadPool {
    private final BlockingQueue<Runnable> taskQueue;
    private final List<WorkerThread> threads;
    private volatile boolean shutdown = false;
    
    public SimpleThreadPool(int poolSize, int queueSize) {
        taskQueue = new LinkedBlockingQueue<>(queueSize);
        threads = new ArrayList<>(poolSize);
        
        for (int i = 0; i < poolSize; i++) {
            WorkerThread worker = new WorkerThread();
            worker.start();
            threads.add(worker);
        }
    }
    
    public boolean submit(Runnable task) {
        if (shutdown) {
            throw new IllegalStateException("Thread pool is shutdown");
        }
        return taskQueue.offer(task);
    }
    
    public void shutdown() {
        shutdown = true;
        threads.forEach(Thread::interrupt);
    }
    
    public void awaitTermination(long timeout, TimeUnit unit) 
            throws InterruptedException {
        long deadline = System.currentTimeMillis() + unit.toMillis(timeout);
        
        for (WorkerThread thread : threads) {
            long remaining = deadline - System.currentTimeMillis();
            if (remaining <= 0) break;
            thread.join(remaining);
        }
    }
    
    private class WorkerThread extends Thread {
        @Override
        public void run() {
            while (!shutdown) {
                try {
                    Runnable task = taskQueue.take();
                    task.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                } catch (Exception e) {
                    // Log exception and continue
                    System.err.println("Task execution failed: " + e);
                }
            }
        }
    }
}
```

### Q: What are common concurrency bugs and how to avoid them?

**Answer:**

**1. Race Conditions:**
```java
// Bug: Non-atomic check-then-act
public class RaceConditionBug {
    private Map<String, String> map = new HashMap<>();
    
    // Bug - race condition
    public void buggyPut(String key, String value) {
        if (!map.containsKey(key)) {
            map.put(key, value); // Another thread might add between check and put
        }
    }
    
    // Fix - use atomic operation
    private ConcurrentHashMap<String, String> safeMap = new ConcurrentHashMap<>();
    
    public void safePut(String key, String value) {
        safeMap.putIfAbsent(key, value); // Atomic operation
    }
}
```

**2. Memory Consistency Errors:**
```java
// Bug: Missing synchronization
public class MemoryConsistencyBug {
    private boolean ready = false;
    private int number = 0;
    
    // Writer thread
    public void writer() {
        number = 42;
        ready = true; // May be reordered or not visible
    }
    
    // Reader thread
    public void reader() {
        if (ready) {
            System.out.println(number); // Might print 0!
        }
    }
    
    // Fix - use volatile
    private volatile boolean safeReady = false;
    private int safeNumber = 0;
    
    public void safeWriter() {
        safeNumber = 42;
        safeReady = true; // Volatile write creates happens-before
    }
}
```

**3. Livelock:**
```java
// Bug: Threads keep yielding to each other
public class LivelockExample {
    static class Polite {
        private boolean yielding = false;
        
        public void pass(Polite other) {
            while (other.yielding) {
                // Both threads keep yielding forever
                yielding = true;
                yielding = false;
            }
        }
    }
    
    // Fix - add randomization or priority
    public void fixedPass(Polite other) throws InterruptedException {
        while (other.yielding) {
            yielding = true;
            Thread.sleep(ThreadLocalRandom.current().nextInt(10));
            yielding = false;
        }
    }
}
```

**4. Thread Leaks:**
```java
// Bug: Threads not properly terminated
public class ThreadLeakBug {
    // Bug - creates new thread for each task
    public void leakyMethod() {
        new Thread(() -> {
            while (true) {
                // Infinite loop without termination
                doWork();
            }
        }).start();
    }
    
    // Fix - use thread pool and proper shutdown
    private final ExecutorService executor = Executors.newFixedThreadPool(10);
    
    public void fixedMethod() {
        executor.submit(() -> doWork());
    }
    
    public void shutdown() {
        executor.shutdown();
        try {
            if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
    
    private void doWork() {}
}
```

---

## Advanced Topics Summary

### Key Takeaways:

1. **Thread Safety**: Achieved through synchronization, immutability, or thread confinement
2. **Memory Model**: Understanding happens-before relationships is crucial
3. **Performance**: Profile, measure, and optimize based on actual bottlenecks
4. **High-Level Abstractions**: Prefer ExecutorService, CompletableFuture over raw threads
5. **Testing**: Concurrent code requires special testing strategies (stress testing, race detection)
6. **Debugging**: Use thread dumps, profilers, and concurrent testing frameworks

### Common Pitfalls to Avoid:

- Not handling InterruptedException properly
- Using synchronized when volatile is sufficient
- Creating threads instead of using thread pools
- Ignoring happens-before guarantees
- Not documenting thread-safety guarantees
- Over-synchronization leading to poor performance
- Under-synchronization leading to race conditions

### Tools for Concurrent Programming:

- **Static Analysis**: SpotBugs, ErrorProne
- **Dynamic Analysis**: ThreadSanitizer, Intel VTune
- **Testing**: JCStress, ConcurrentUnit
- **Profiling**: JProfiler, YourKit, async-profiler
- **Monitoring**: JMX, Micrometer, custom metrics

This comprehensive guide covers the essential aspects of Java multithreading and concurrency that are crucial for senior Java developer interviews and real-world applications.