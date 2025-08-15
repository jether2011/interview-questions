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

**Context & Problem:**
Modern applications face several challenges that single-threaded programming cannot efficiently solve:
- **CPU underutilization**: Single-threaded programs waste CPU cycles during I/O operations (disk reads, network calls, database queries)
- **Poor user experience**: Applications freeze during long-running operations
- **Scalability limitations**: Cannot leverage multi-core processors effectively
- **Resource bottlenecks**: Sequential processing creates artificial constraints

**Deep Dive Answer:**
Multithreading is the concurrent execution of multiple threads within a single program. Each thread represents an independent path of execution with its own:
- **Thread Stack**: Local variables, method parameters, return addresses
- **Program Counter**: Current instruction being executed
- **Thread-local storage**: Variables specific to that thread

Threads share:
- **Heap Memory**: Objects, class instances, arrays
- **Method Area**: Class metadata, static variables, constants
- **Native Method Stacks**: Native code execution context

**Why It's Critical:**

1. **Better CPU Utilization**
   - Problem: CPU sits idle during I/O operations
   - Solution: Other threads can execute while one waits for I/O
   - Real-world impact: Web servers handling 10,000+ concurrent requests

2. **Improved Responsiveness**
   - Problem: UI freezes during computation
   - Solution: Background threads handle heavy operations
   - Real-world impact: Smooth 60 FPS animations while loading data

3. **Parallel Processing**
   - Problem: Single core can't handle big data processing
   - Solution: Divide work across multiple cores
   - Real-world impact: MapReduce processing terabytes of data

4. **Resource Fairness**
   - Problem: One user monopolizes resources
   - Solution: Time-slicing ensures fair resource distribution
   - Real-world impact: Database connection pooling serving multiple clients

```java
// Single-threaded approach - blocks during I/O
public void processFiles(List<File> files) {
    // Problem: If each file takes 1 second to process
    // 100 files = 100 seconds sequential processing
    long startTime = System.currentTimeMillis();
    
    for (File file : files) {
        readFile(file);      // Blocks for ~200ms (disk I/O)
        processData();       // CPU work ~500ms
        writeResults();      // Blocks for ~300ms (disk I/O)
    }
    // Total: 1000ms per file × N files = N seconds
    
    long totalTime = System.currentTimeMillis() - startTime;
    System.out.println("Sequential: " + totalTime + "ms");
}

// Multi-threaded approach - efficient resource usage
public void processFilesParallel(List<File> files) {
    int coreCount = Runtime.getRuntime().availableProcessors();
    ExecutorService executor = Executors.newFixedThreadPool(coreCount);
    CountDownLatch latch = new CountDownLatch(files.size());
    long startTime = System.currentTimeMillis();
    
    files.forEach(file -> 
        executor.submit(() -> {
            try {
                readFile(file);      // Thread A blocks, Thread B runs
                processData();       // Utilizes different CPU cores
                writeResults();      // Can overlap with other threads' I/O
            } finally {
                latch.countDown();
            }
        })
    );
    
    latch.await();
    // Total: ~(N files / core count) seconds for CPU-bound
    // Even better for I/O-bound operations
    
    long totalTime = System.currentTimeMillis() - startTime;
    System.out.println("Parallel: " + totalTime + "ms");
    // Typical speedup: 4-8x for I/O heavy, 2-4x for CPU heavy
    
    executor.shutdown();
}
```

**Key Takeaways:**
- **I/O-bound tasks**: Benefit most from multithreading (10x+ speedup possible)
- **CPU-bound tasks**: Limited by number of cores (max speedup = core count)
- **Thread pool sizing**: I/O-bound = 2×cores, CPU-bound = cores+1
- **Memory overhead**: Each thread costs ~1MB stack space
- **Context switching**: Too many threads (>1000) degrades performance

### Q: What's the difference between process and thread?

**Context & Problem:**
Operating systems need to manage multiple executing programs efficiently. Two fundamental approaches exist:
- **Process-based concurrency**: Complete isolation, high safety
- **Thread-based concurrency**: Shared resources, high performance

Understanding their trade-offs is crucial for system design decisions.

**Deep Dive Answer:**

| Aspect | Process | Thread | Real-World Implication |
|--------|---------|--------|------------------------|
| **Memory** | Independent address space (GB) | Shared memory within process (KB-MB) | Processes: Chrome tabs isolated; Threads: Java web requests share cache |
| **Creation Time** | ~10-100ms (fork/exec) | ~1ms (pthread_create) | Processes: Slow startup; Threads: Quick task spawning |
| **Communication** | IPC (microseconds-milliseconds) | Direct memory (nanoseconds) | Processes: Redis pub/sub; Threads: Shared HashMap |
| **Context Switch** | ~5-10 microseconds | ~1-2 microseconds | High-frequency trading uses threads |
| **Isolation** | Complete (separate page tables) | Partial (shared heap) | Processes: Security boundaries; Threads: Performance |
| **Memory Overhead** | 10-100MB per process | 1MB stack per thread | 1000 processes = 10GB+; 1000 threads = 1GB |
| **Failure Impact** | Isolated (other processes survive) | Cascading (can crash process) | Microservices vs Monoliths |
| **Debugging** | Easier (isolated state) | Harder (shared state races) | Production debugging complexity |

```java
// Process creation - Heavy operation with complete isolation
public class ProcessExample {
    public void demonstrateProcess() throws IOException, InterruptedException {
        // Process creation involves:
        // 1. Fork system call (copy page tables)
        // 2. Exec system call (load new program)
        // 3. New address space allocation
        // 4. Resource descriptor duplication
        
        long startTime = System.nanoTime();
        
        ProcessBuilder pb = new ProcessBuilder("java", "-jar", "worker.jar");
        pb.environment().put("WORKER_ID", "1"); // Separate environment
        Process process = pb.start();
        
        // Inter-process communication via pipes
        try (BufferedWriter writer = new BufferedWriter(
                new OutputStreamWriter(process.getOutputStream()));
             BufferedReader reader = new BufferedReader(
                new InputStreamReader(process.getInputStream()))) {
            
            writer.write("TASK_DATA");
            writer.flush();
            
            String result = reader.readLine(); // Blocking I/O
        }
        
        int exitCode = process.waitFor();
        long elapsed = System.nanoTime() - startTime;
        System.out.println("Process creation + execution: " + 
            TimeUnit.NANOSECONDS.toMillis(elapsed) + "ms");
        // Typical: 50-200ms for JVM process startup
    }
}

// Thread creation - Lightweight with shared memory
public class ThreadExample {
    private volatile int sharedCounter = 0; // Shared heap memory
    private final Object lock = new Object();
    
    public void demonstrateThread() throws InterruptedException {
        long startTime = System.nanoTime();
        
        Thread worker = new Thread(() -> {
            // Thread creation involves:
            // 1. Stack allocation (1MB default)
            // 2. Thread descriptor in kernel
            // 3. Scheduling queue insertion
            // No memory space duplication!
            
            // Direct memory access - no IPC needed
            synchronized (lock) {
                sharedCounter++; // Nanosecond access time
            }
            
            // Can access parent's objects directly
            processSharedData(getSharedList());
        });
        
        worker.start();
        long elapsed = System.nanoTime() - startTime;
        System.out.println("Thread creation: " + 
            TimeUnit.NANOSECONDS.toMicros(elapsed) + "μs");
        // Typical: 100-500 microseconds
        
        worker.join();
    }
    
    private List<String> getSharedList() { 
        return new ArrayList<>(); // Shared heap object
    }
    
    private void processSharedData(List<String> data) {
        // Direct manipulation, no serialization needed
    }
}
```

**Key Decision Factors:**

**Choose Processes When:**
- **Security isolation required**: Different users, untrusted code
- **Fault tolerance critical**: One crash shouldn't affect others
- **Resource limits needed**: CPU/memory quotas per process
- **Language interoperability**: Python calling Java calling C++
- Example: Web browsers, microservices, container orchestration

**Choose Threads When:**
- **Performance critical**: Nanosecond latency requirements
- **Shared state beneficial**: Large caches, connection pools
- **Memory constraints**: Limited RAM available
- **Fine-grained parallelism**: Tight cooperation needed
- Example: Web servers, game engines, database engines

**Hybrid Approach:**
```java
// Modern applications often use both
public class HybridArchitecture {
    // Multiple processes for isolation
    // Each process has thread pool for performance
    
    public void webServerArchitecture() {
        // Master process
        for (int i = 0; i < NUM_WORKERS; i++) {
            // Fork worker processes (isolation)
            Process worker = createWorkerProcess();
            
            // Each worker has thread pool (performance)
            // handles 1000s of connections via threads
        }
    }
}
```

**Performance Metrics Comparison:**
- Process creation: 10-100ms
- Thread creation: 0.1-1ms  
- Process context switch: 5-10μs
- Thread context switch: 1-2μs
- IPC latency: 1-100μs
- Shared memory access: 1-100ns

**Key Takeaways:**
- **Processes**: Safety and isolation at the cost of performance
- **Threads**: Performance and efficiency at the cost of complexity
- **Modern trend**: Microservices (processes) with async/await (lightweight threads)
- **Resource planning**: 1000 threads feasible, 1000 processes problematic
- **Debugging**: Processes easier to debug, threads require specialized tools

---

## Thread Creation and Lifecycle

### Q: What are the different ways to create threads in Java?

**Context & Problem:**
Java's evolution has provided multiple thread creation mechanisms, each solving specific problems:
- **Early Java (1.0)**: Only Thread class - inflexible due to single inheritance
- **Java 1.0+**: Runnable interface - better but no return values
- **Java 5**: Callable and Executors - enterprise-grade concurrency
- **Java 8+**: Lambda expressions and CompletableFuture - functional programming

**Deep Dive Answer:**

**1. Extending Thread Class (Legacy Approach):**

**When to Use:**
- Simple educational examples
- When you need to override other Thread methods
- Legacy codebases

**Problems It Solves:**
- Direct thread manipulation
- Custom thread behavior

**Limitations:**
- Wastes single inheritance slot
- Tight coupling between task and thread management
- Cannot be reused with executors
```java
public class MyThread extends Thread {
    private volatile boolean running = true;
    private final String taskName;
    
    public MyThread(String taskName) {
        super(taskName); // Set thread name
        this.taskName = taskName;
        // Can customize thread properties
        setPriority(Thread.NORM_PRIORITY);
        setDaemon(false);
    }
    
    @Override
    public void run() {
        try {
            System.out.println("Starting: " + taskName);
            while (running) {
                // Task logic
                performWork();
                
                // Check interruption
                if (Thread.interrupted()) {
                    System.out.println("Interrupted: " + taskName);
                    break;
                }
            }
        } catch (Exception e) {
            System.err.println("Error in " + taskName + ": " + e);
        } finally {
            cleanup();
        }
    }
    
    public void stopGracefully() {
        running = false;
        interrupt(); // Wake up if sleeping
    }
    
    private void performWork() {
        // Actual task implementation
    }
    
    private void cleanup() {
        // Resource cleanup
    }
}

// Usage
MyThread thread = new MyThread("DataProcessor");
thread.start();
// ... later
thread.stopGracefully();
thread.join(5000); // Wait max 5 seconds
```

**Real-World Anti-Pattern Example:**
```java
// DON'T DO THIS - wastes inheritance
public class DatabaseWriter extends Thread {
    // Now cannot extend your AbstractDAO class!
    // Forced to use composition instead of inheritance
}
```

**2. Implementing Runnable Interface (Better Approach):**

**When to Use:**
- Task can be executed by any thread
- Need to extend another class
- Want to reuse with different executors
- Separation of concerns (task vs thread management)

**Problems It Solves:**
- Single inheritance limitation
- Better object-oriented design
- Executor framework compatibility

```java
// Full implementation with best practices
public class DataProcessor implements Runnable {
    private final String dataSource;
    private final CountDownLatch startLatch;
    private final AtomicBoolean completed = new AtomicBoolean(false);
    
    public DataProcessor(String dataSource, CountDownLatch startLatch) {
        this.dataSource = dataSource;
        this.startLatch = startLatch;
    }
    
    @Override
    public void run() {
        try {
            // Wait for signal to start
            startLatch.await();
            
            // Process data
            System.out.println(Thread.currentThread().getName() + 
                              " processing " + dataSource);
            
            List<Record> records = fetchData(dataSource);
            processRecords(records);
            
            completed.set(true);
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.err.println("Processing interrupted");
        } catch (Exception e) {
            System.err.println("Processing failed: " + e);
        }
    }
    
    public boolean isCompleted() {
        return completed.get();
    }
    
    private List<Record> fetchData(String source) {
        // Fetch implementation
        return new ArrayList<>();
    }
    
    private void processRecords(List<Record> records) {
        // Processing logic
    }
    
    private static class Record {}
}

// Multiple usage patterns
public class RunnableUsagePatterns {
    public void demonstrateUsage() throws InterruptedException {
        CountDownLatch startSignal = new CountDownLatch(1);
        
        // 1. Direct thread creation
        Runnable task1 = new DataProcessor("db1", startSignal);
        Thread thread1 = new Thread(task1, "Worker-1");
        thread1.start();
        
        // 2. With executor service (preferred)
        ExecutorService executor = Executors.newFixedThreadPool(3);
        executor.execute(new DataProcessor("db2", startSignal));
        
        // 3. Lambda for simple tasks (Java 8+)
        executor.execute(() -> {
            System.out.println("Quick task: " + 
                Thread.currentThread().getName());
        });
        
        // 4. Method reference (Java 8+)
        executor.execute(this::simpleTask);
        
        // Start all tasks
        startSignal.countDown();
        
        executor.shutdown();
    }
    
    private void simpleTask() {
        System.out.println("Method reference task");
    }
}
```

**Key Advantages Over Thread Class:**
- Can extend another class
- Can implement multiple interfaces  
- Same Runnable can be executed by multiple threads
- Works with thread pools
- Better testability (can run synchronously in tests)

**3. Implementing Callable Interface (Return Values & Exceptions):**

**When to Use:**
- Need to return a result
- Need to throw checked exceptions
- Want to track task completion
- Need to cancel tasks

**Problems It Solves:**
- Runnable can't return values
- Runnable can't throw checked exceptions
- No built-in completion tracking

```java
// Comprehensive Callable example
public class DataFetcher implements Callable<DataResult> {
    private final String endpoint;
    private final int timeout;
    private final RetryPolicy retryPolicy;
    
    public DataFetcher(String endpoint, int timeout, RetryPolicy retryPolicy) {
        this.endpoint = endpoint;
        this.timeout = timeout;
        this.retryPolicy = retryPolicy;
    }
    
    @Override
    public DataResult call() throws Exception {
        int attempts = 0;
        Exception lastException = null;
        
        while (attempts < retryPolicy.maxAttempts) {
            try {
                // Check for interruption
                if (Thread.currentThread().isInterrupted()) {
                    throw new InterruptedException("Task cancelled");
                }
                
                // Attempt to fetch data
                HttpResponse response = fetchWithTimeout(endpoint, timeout);
                
                if (response.isSuccessful()) {
                    return new DataResult(
                        response.getData(),
                        attempts + 1,
                        System.currentTimeMillis()
                    );
                }
                
                lastException = new IOException("HTTP " + response.getCode());
                
            } catch (SocketTimeoutException e) {
                lastException = e;
                attempts++;
                
                if (attempts < retryPolicy.maxAttempts) {
                    Thread.sleep(retryPolicy.getBackoffMs(attempts));
                }
            }
        }
        
        // All retries exhausted
        throw new DataFetchException(
            "Failed after " + attempts + " attempts", 
            lastException
        );
    }
    
    private HttpResponse fetchWithTimeout(String url, int timeoutMs) 
            throws SocketTimeoutException {
        // Simulated HTTP call
        return new HttpResponse();
    }
    
    // Result wrapper
    public static class DataResult {
        public final String data;
        public final int attempts;
        public final long timestamp;
        
        public DataResult(String data, int attempts, long timestamp) {
            this.data = data;
            this.attempts = attempts;
            this.timestamp = timestamp;
        }
    }
    
    // Custom exception
    public static class DataFetchException extends Exception {
        public DataFetchException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    // Supporting classes
    static class RetryPolicy {
        final int maxAttempts = 3;
        long getBackoffMs(int attempt) {
            return (long) Math.pow(2, attempt) * 1000; // Exponential backoff
        }
    }
    
    static class HttpResponse {
        boolean isSuccessful() { return Math.random() > 0.3; }
        String getData() { return "data"; }
        int getCode() { return 500; }
    }
}

// Advanced usage patterns
public class CallableUsagePatterns {
    private final ExecutorService executor = Executors.newFixedThreadPool(10);
    
    public void demonstrateCallablePatterns() throws Exception {
        // 1. Single task with timeout
        Future<DataResult> future = executor.submit(
            new DataFetcher("api.example.com", 5000, new RetryPolicy())
        );
        
        try {
            DataResult result = future.get(10, TimeUnit.SECONDS);
            System.out.println("Fetched: " + result.data);
        } catch (TimeoutException e) {
            future.cancel(true); // Interrupt the thread
            System.err.println("Task timed out");
        }
        
        // 2. Multiple tasks with invokeAll
        List<Callable<DataResult>> tasks = Arrays.asList(
            new DataFetcher("api1.com", 3000, new RetryPolicy()),
            new DataFetcher("api2.com", 3000, new RetryPolicy()),
            new DataFetcher("api3.com", 3000, new RetryPolicy())
        );
        
        List<Future<DataResult>> futures = executor.invokeAll(
            tasks, 15, TimeUnit.SECONDS
        );
        
        for (Future<DataResult> f : futures) {
            if (f.isDone() && !f.isCancelled()) {
                try {
                    DataResult r = f.get();
                    System.out.println("Success: " + r.data);
                } catch (ExecutionException e) {
                    System.err.println("Task failed: " + e.getCause());
                }
            }
        }
        
        // 3. First successful result with invokeAny
        try {
            DataResult firstSuccess = executor.invokeAny(tasks);
            System.out.println("First success: " + firstSuccess.data);
        } catch (ExecutionException e) {
            System.err.println("All tasks failed");
        }
    }
}
```

**Callable vs Runnable Decision Matrix:**

| Requirement | Runnable | Callable |
|------------|----------|----------|
| Return value | ❌ | ✅ |
| Checked exceptions | ❌ | ✅ |
| Use with Thread class | ✅ | ❌ |
| Use with ExecutorService | ✅ | ✅ |
| Future tracking | ❌ | ✅ |
| Cancellation support | Basic | Advanced |

**4. Using Thread Pools (Production Best Practice):**

**When to Use:**
- Always in production code
- When managing multiple concurrent tasks
- Need to control resource usage
- Want to reuse threads

**Problems It Solves:**
- Thread creation overhead (1ms per thread)
- Resource exhaustion (OutOfMemoryError)
- Manual thread lifecycle management
- Graceful shutdown complexity

```java
public class ThreadPoolPatterns {
    
    // Production-ready thread pool configuration
    public class OptimizedThreadPool {
        private final ThreadPoolExecutor executor;
        private final ScheduledExecutorService scheduler;
        private final AtomicLong rejectedTasks = new AtomicLong();
        
        public OptimizedThreadPool() {
            int coreSize = Runtime.getRuntime().availableProcessors();
            int maxSize = coreSize * 2;
            
            // Custom thread factory with naming and exception handling
            ThreadFactory factory = new ThreadFactory() {
                private final AtomicInteger counter = new AtomicInteger();
                private final Thread.UncaughtExceptionHandler handler = 
                    (t, e) -> {
                        System.err.println("Uncaught in " + t.getName() + ": " + e);
                        e.printStackTrace();
                    };
                
                @Override
                public Thread newThread(Runnable r) {
                    Thread t = new Thread(r);
                    t.setName("worker-" + counter.incrementAndGet());
                    t.setUncaughtExceptionHandler(handler);
                    t.setDaemon(false);
                    return t;
                }
            };
            
            // Custom rejection handler with metrics
            RejectedExecutionHandler rejectionHandler = (r, executor) -> {
                rejectedTasks.incrementAndGet();
                // Try to run in caller thread as fallback
                if (!executor.isShutdown()) {
                    try {
                        executor.getQueue().put(r); // Block until space available
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                }
            };
            
            // Create the executor
            this.executor = new ThreadPoolExecutor(
                coreSize,                      // Core pool size
                maxSize,                        // Maximum pool size
                60L, TimeUnit.SECONDS,          // Keep-alive time
                new LinkedBlockingQueue<>(1000), // Bounded queue
                factory,                        // Thread factory
                rejectionHandler                // Rejection policy
            );
            
            // Enable core thread timeout for resource efficiency
            executor.allowCoreThreadTimeOut(true);
            
            // Monitoring thread
            this.scheduler = Executors.newSingleThreadScheduledExecutor();
            scheduler.scheduleAtFixedRate(this::logMetrics, 0, 30, TimeUnit.SECONDS);
        }
        
        private void logMetrics() {
            System.out.println(String.format(
                "Pool [active=%d, completed=%d, queued=%d, rejected=%d]",
                executor.getActiveCount(),
                executor.getCompletedTaskCount(),
                executor.getQueue().size(),
                rejectedTasks.get()
            ));
        }
        
        public Future<?> submit(Runnable task) {
            return executor.submit(wrapWithMetrics(task));
        }
        
        private Runnable wrapWithMetrics(Runnable task) {
            return () -> {
                long start = System.nanoTime();
                try {
                    task.run();
                } finally {
                    long duration = System.nanoTime() - start;
                    // Record metrics
                }
            };
        }
        
        public void shutdown() {
            executor.shutdown();
            scheduler.shutdown();
            try {
                if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
                    executor.shutdownNow();
                    if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                        System.err.println("Pool did not terminate");
                    }
                }
            } catch (InterruptedException e) {
                executor.shutdownNow();
                Thread.currentThread().interrupt();
            }
        }
    }
    
    // Different pool types for different workloads
    public void demonstratePoolTypes() {
        // CPU-bound tasks: Limited threads
        ExecutorService cpuBound = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors() + 1
        );
        
        // I/O-bound tasks: More threads
        ExecutorService ioBound = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors() * 2
        );
        
        // Mixed workload: Work-stealing pool
        ExecutorService mixed = Executors.newWorkStealingPool();
        
        // Scheduled tasks
        ScheduledExecutorService scheduled = 
            Executors.newScheduledThreadPool(2);
        
        // Single background thread
        ExecutorService background = Executors.newSingleThreadExecutor();
    }
}
```

**Thread Pool Size Guidelines:**

```java
public class ThreadPoolSizing {
    public static int calculateOptimalThreads(double targetUtilization,
                                               double waitTime,
                                               double computeTime) {
        int cores = Runtime.getRuntime().availableProcessors();
        
        // Little's Law: N = U * C * (1 + W/C)
        // N = number of threads
        // U = target CPU utilization (0-1)
        // C = number of cores
        // W = wait time
        // C = compute time
        
        return (int) (cores * targetUtilization * (1 + waitTime / computeTime));
    }
    
    public static void main(String[] args) {
        // Example: Web server handling requests
        // 50ms database wait, 10ms processing
        int webServerThreads = calculateOptimalThreads(0.8, 50, 10);
        System.out.println("Web server threads: " + webServerThreads);
        // Result: ~38 threads on 8-core machine
        
        // Example: CPU-intensive batch processing
        // 0ms wait, 100ms processing
        int batchThreads = calculateOptimalThreads(0.9, 0, 100);
        System.out.println("Batch processing threads: " + batchThreads);
        // Result: ~7 threads on 8-core machine
    }
}
```

**Key Takeaways:**
- **Always use thread pools** in production
- **Size appropriately**: Too few = underutilization, Too many = overhead
- **Monitor metrics**: Active threads, queue size, rejection rate
- **Graceful shutdown**: Always implement proper shutdown
- **Custom thread factory**: For naming, priority, exception handling
- **Bounded queues**: Prevent memory exhaustion

### Q: Explain thread lifecycle and states in Java

**Context & Problem:**
Threads transition through multiple states during their lifecycle. Understanding these transitions is crucial for:
- Debugging deadlocks and performance issues
- Proper thread coordination
- Resource management
- Performance optimization

**Deep Dive Answer:**

**Thread States and Transitions:**

1. **NEW (Thread Birth)**
   - **What it is**: Thread object created but start() not called
   - **Memory state**: Stack space not yet allocated
   - **CPU state**: Not scheduled
   - **Duration**: Microseconds to indefinite
   - **Common causes**: Object instantiation
   - **Debugging tip**: Threads stuck in NEW indicate start() never called

2. **RUNNABLE (Active Execution)**
   - **What it is**: Thread is executing or ready to execute
   - **Sub-states**: 
     - Running: Currently executing on CPU
     - Ready: Waiting for CPU time slice
   - **Memory state**: Stack allocated, accessing heap
   - **CPU state**: Scheduled or in run queue
   - **Duration**: Nanoseconds to seconds
   - **Transitions from**: NEW (via start()), BLOCKED, WAITING, TIMED_WAITING
   - **Transitions to**: Any other state except NEW

3. **BLOCKED (Monitor Lock Contention)**
   - **What it is**: Waiting to acquire a monitor lock
   - **Memory state**: Stack preserved, cannot access synchronized region
   - **CPU state**: Not scheduled
   - **Duration**: Microseconds to indefinite (deadlock)
   - **Common causes**: synchronized block/method, contention
   - **Performance impact**: High contention = poor scalability
   - **Debugging**: jstack shows "waiting for monitor entry"

4. **WAITING (Indefinite Suspension)**
   - **What it is**: Waiting for another thread's notification
   - **Memory state**: Stack preserved, releases locks
   - **CPU state**: Not scheduled
   - **Duration**: Until notified or interrupted
   - **Common causes**: 
     - Object.wait()
     - Thread.join()
     - LockSupport.park()
   - **Debugging**: jstack shows "waiting on condition"

5. **TIMED_WAITING (Temporary Suspension)**
   - **What it is**: Waiting with timeout
   - **Memory state**: Stack preserved
   - **CPU state**: Not scheduled until timeout or notification
   - **Duration**: Specified timeout or until notified
   - **Common causes**:
     - Thread.sleep(ms)
     - Object.wait(ms)
     - Thread.join(ms)
     - LockSupport.parkNanos()
   - **Use cases**: Polling, timeouts, scheduled tasks

6. **TERMINATED (Thread Death)**
   - **What it is**: run() method completed or exception thrown
   - **Memory state**: Stack deallocated, heap objects may persist
   - **CPU state**: Never scheduled again
   - **Duration**: Permanent
   - **Causes**: Normal completion, uncaught exception
   - **Cleanup**: Thread object eligible for GC when no references

```java
public class ComprehensiveThreadLifecycle {
    
    // Demonstrate all state transitions
    public static void demonstrateAllStates() throws InterruptedException {
        final Object lock = new Object();
        final Object lock2 = new Object();
        
        // Thread to demonstrate all states
        Thread demonstrator = new Thread(() -> {
            try {
                // State: RUNNABLE
                System.out.println("1. Running...");
                
                // State: TIMED_WAITING (sleep)
                Thread.sleep(100);
                
                // State: BLOCKED (waiting for lock)
                synchronized (lock2) {
                    System.out.println("2. Acquired lock2");
                }
                
                // State: WAITING (wait on object)
                synchronized (lock) {
                    System.out.println("3. About to wait");
                    lock.wait();
                    System.out.println("4. Resumed from wait");
                }
                
                // State: RUNNABLE again
                System.out.println("5. Completing...");
                
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "DemoThread");
        
        // Monitor thread to observe states
        Thread monitor = new Thread(() -> {
            Map<Thread.State, Integer> stateCount = new HashMap<>();
            Thread.State lastState = null;
            
            while (demonstrator.getState() != Thread.State.TERMINATED) {
                Thread.State currentState = demonstrator.getState();
                
                // Count state occurrences
                stateCount.merge(currentState, 1, Integer::sum);
                
                // Log state changes
                if (currentState != lastState) {
                    System.out.println(String.format(
                        "[Monitor] State changed: %s -> %s",
                        lastState, currentState
                    ));
                    
                    // Capture stack trace for blocked state
                    if (currentState == Thread.State.BLOCKED) {
                        StackTraceElement[] stack = demonstrator.getStackTrace();
                        if (stack.length > 0) {
                            System.out.println("  Blocked at: " + stack[0]);
                        }
                    }
                    
                    lastState = currentState;
                }
                
                try {
                    Thread.sleep(10); // Poll interval
                } catch (InterruptedException e) {
                    break;
                }
            }
            
            System.out.println("\nState Statistics:");
            stateCount.forEach((state, count) -> 
                System.out.println("  " + state + ": " + count + " observations")
            );
        }, "MonitorThread");
        
        // Execution sequence
        System.out.println("=== Thread Lifecycle Demo ===");
        System.out.println("Initial state: " + demonstrator.getState()); // NEW
        
        // Start monitoring
        monitor.setDaemon(true);
        monitor.start();
        
        // Hold lock2 to cause BLOCKED state
        synchronized (lock2) {
            demonstrator.start(); // -> RUNNABLE
            Thread.sleep(150); // Let it hit the lock2 synchronization
        }
        
        Thread.sleep(200); // Let it enter wait()
        
        // Notify to wake up from WAITING
        synchronized (lock) {
            lock.notify();
        }
        
        // Wait for completion
        demonstrator.join();
        System.out.println("Final state: " + demonstrator.getState()); // TERMINATED
    }
    
    // Real-world debugging example
    public static class ThreadStateDebugger {
        private final ScheduledExecutorService monitor = 
            Executors.newSingleThreadScheduledExecutor();
        
        public void startMonitoring() {
            monitor.scheduleAtFixedRate(() -> {
                Map<Thread.State, List<Thread>> threadsByState = 
                    Thread.getAllStackTraces().keySet().stream()
                        .collect(Collectors.groupingBy(Thread::getState));
                
                System.out.println("\n=== Thread State Report ===");
                System.out.println("Time: " + new Date());
                
                threadsByState.forEach((state, threads) -> {
                    System.out.printf("%s (%d threads):\n", state, threads.size());
                    threads.stream()
                        .limit(3) // Show first 3 threads per state
                        .forEach(t -> System.out.printf("  - %s (daemon=%s)\n", 
                            t.getName(), t.isDaemon()));
                });
                
                // Detect potential issues
                List<Thread> blocked = threadsByState.getOrDefault(
                    Thread.State.BLOCKED, Collections.emptyList()
                );
                if (blocked.size() > 10) {
                    System.out.println("⚠️ WARNING: " + blocked.size() + 
                        " threads BLOCKED - possible contention!");
                }
                
                List<Thread> waiting = threadsByState.getOrDefault(
                    Thread.State.WAITING, Collections.emptyList()
                );
                if (waiting.size() > 50) {
                    System.out.println("⚠️ WARNING: " + waiting.size() + 
                        " threads WAITING - possible thread pool exhaustion!");
                }
                
            }, 0, 5, TimeUnit.SECONDS);
        }
        
        public void stopMonitoring() {
            monitor.shutdown();
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        demonstrateAllStates();
    }
}
```

**State Transition Diagram:**
```
        start()
   NEW ---------> RUNNABLE <--------
                      |              |
                      |              |
    synchronized      |              | notify()
    (blocked)         |              | interrupt()
                      v              | timeout
                  BLOCKED            |
                      |              |
                      |              |
    acquired lock     |              |
                      |              |
                      v              |
                  RUNNABLE           |
                      |              |
    wait()            |              |
    join()            |              |
    park()            v              |
                  WAITING -----------
                      |
    sleep()           |
    wait(timeout)     |
                      v
                TIMED_WAITING -------
                      |
                      |
    run() exits       |
    exception         v
                 TERMINATED
```

**Performance Implications:**

| State | CPU Usage | Memory | Scheduling Overhead | Common Issues |
|-------|-----------|--------|-------------------|---------------|
| NEW | 0% | Minimal | None | Memory leak if not started |
| RUNNABLE | 0-100% | Active | High | CPU bottleneck |
| BLOCKED | 0% | Held | Medium | Lock contention |
| WAITING | 0% | Held | Low | Deadlock risk |
| TIMED_WAITING | 0% | Held | Low | Unnecessary delays |
| TERMINATED | 0% | Released | None | Thread leak if referenced |

**Key Takeaways:**
- **State transitions are one-way**: Can never go back to NEW
- **RUNNABLE ≠ Running**: Thread might be waiting for CPU
- **BLOCKED vs WAITING**: BLOCKED wants lock, WAITING wants notification
- **Monitor tools**: jstack, jconsole, VisualVM for production debugging
- **Performance**: Minimize BLOCKED state through lock-free designs
- **Resource leaks**: Ensure threads reach TERMINATED state

---

## Synchronization Mechanisms

### Q: What is the synchronized keyword and how does it work?

**Context & Problem:**
When multiple threads access shared mutable state, we face:
- **Race Conditions**: Threads overwriting each other's changes
- **Memory Visibility Issues**: Changes not visible across threads
- **Data Corruption**: Partial updates leading to inconsistent state
- **Lost Updates**: Concurrent modifications losing data

**Deep Dive Answer:**

The `synchronized` keyword is Java's built-in mutual exclusion mechanism that ensures:
1. **Atomicity**: Critical sections execute completely without interruption
2. **Visibility**: Changes made by one thread are visible to others
3. **Ordering**: Prevents instruction reordering that could break logic

**How It Works Internally:**
- Every Java object has an intrinsic lock (monitor)
- Threads must acquire the lock before entering synchronized code
- Only one thread can hold a lock at a time
- JVM uses OS-level primitives (futex on Linux, Critical Sections on Windows)
- Modern JVMs optimize with biased locking, lock coarsening, and lock elimination

**Method-level Synchronization:**

```java
public class Counter {
    private int count = 0;
    private long lastModified = 0;
    
    // Instance method synchronization - locks on 'this'
    public synchronized void increment() {
        // Problem: Locks entire object, even for unrelated operations
        count++;
        lastModified = System.currentTimeMillis();
    }
    
    // Equivalent expanded form showing implicit lock
    public void incrementExpanded() {
        // Monitor enter (acquires lock)
        synchronized(this) {
            count++;
            lastModified = System.currentTimeMillis();
        } // Monitor exit (releases lock)
    }
    
    // Static method synchronization - locks on Class object
    public static synchronized void staticMethod() {
        // Equivalent to synchronized(Counter.class)
        // All static synchronized methods share same lock
    }
    
    // Performance comparison
    public void performanceTest() {
        long start = System.nanoTime();
        
        // Synchronized method: ~50-100ns per call under contention
        for (int i = 0; i < 1000000; i++) {
            increment();
        }
        
        long syncTime = System.nanoTime() - start;
        System.out.println("Synchronized: " + syncTime / 1_000_000 + "ms");
    }
}

// Common Anti-pattern: Over-synchronization
public class OverSynchronized {
    // BAD: Synchronizes even read operations
    public synchronized int getCount() {
        return count; // Simple read doesn't need synchronization for primitives
    }
    
    // BAD: Long operations holding lock
    public synchronized void processData() {
        fetchFromDatabase();    // 100ms - holds lock!
        transformData();        // 50ms - holds lock!
        saveToCache();         // 30ms - holds lock!
        // Total: 180ms blocking all other synchronized methods
    }
}
```

**Block-level Synchronization (Best Practice for Fine-grained Control):**

```java
public class BankAccount {
    // Use private final lock objects for encapsulation
    private double balance;
    private final Object balanceLock = new Object();
    private final List<Transaction> history = new ArrayList<>();
    private final Object historyLock = new Object();
    
    // Fine-grained locking example
    public void transfer(BankAccount to, double amount) throws InsufficientFundsException {
        // Acquire locks in consistent order to prevent deadlock
        BankAccount first = System.identityHashCode(this) < System.identityHashCode(to) ? this : to;
        BankAccount second = first == this ? to : this;
        
        synchronized(first.balanceLock) {
            synchronized(second.balanceLock) {
                // Critical section - minimal code
                if (this.balance < amount) {
                    throw new InsufficientFundsException("Balance: " + balance);
                }
                this.balance -= amount;
                to.balance += amount;
            }
        }
        
        // Non-critical operations outside lock
        logTransaction(new Transaction(this, to, amount));
        notifyObservers();
    }
    
    // Separate locks for independent operations
    public double getBalance() {
        synchronized(balanceLock) {
            return balance; // Quick read
        }
    }
    
    public List<Transaction> getHistory() {
        synchronized(historyLock) {
            // Return defensive copy to prevent external modification
            return new ArrayList<>(history);
        }
    }
    
    // Double-checked locking pattern (correct with volatile)
    private volatile ExpensiveObject cached;
    
    public ExpensiveObject getCached() {
        ExpensiveObject result = cached;
        if (result == null) { // First check without lock
            synchronized(this) {
                result = cached;
                if (result == null) { // Second check with lock
                    cached = result = new ExpensiveObject();
                }
            }
        }
        return result;
    }
    
    // Lock splitting for better concurrency
    class OptimizedCounter {
        private long evenCount = 0;
        private long oddCount = 0;
        private final Object evenLock = new Object();
        private final Object oddLock = new Object();
        
        public void increment(int value) {
            if (value % 2 == 0) {
                synchronized(evenLock) {
                    evenCount++;
                }
            } else {
                synchronized(oddLock) {
                    oddCount++;
                }
            }
            // Doubles concurrency by using separate locks
        }
    }
    
    private static class Transaction {
        Transaction(BankAccount from, BankAccount to, double amount) {}
    }
    private static class InsufficientFundsException extends Exception {
        InsufficientFundsException(String msg) { super(msg); }
    }
    private static class ExpensiveObject {}
    private void logTransaction(Transaction t) {}
    private void notifyObservers() {}
}
```

**Synchronization Best Practices:**

| Practice | Reason | Example |
|----------|--------|---------|  
| Use private final locks | Prevents external locking | `private final Object lock = new Object()` |
| Minimize critical sections | Reduces contention | Only sync data access, not I/O |
| Avoid nested locks | Prevents deadlock | Use lock ordering or lock-free designs |
| Don't sync on mutable fields | Lock object could change | Never sync on non-final fields |
| Don't sync on boxed primitives | JVM may reuse instances | `Integer` objects may be cached |
| Use volatile for flags | Lighter than synchronization | `volatile boolean shutdown` |
| Document locking policy | Helps maintenance | `@GuardedBy("lock")` annotations |

### Q: What is volatile and when should you use it?

**Context & Problem:**
Modern processors use multiple optimization techniques that can break multi-threaded code:
- **CPU Caches**: Each core has private L1/L2 caches
- **Store Buffers**: Writes may be delayed
- **Instruction Reordering**: Compilers and CPUs reorder for performance
- **Register Allocation**: Variables may be kept in CPU registers

**Deep Dive Answer:**

`volatile` is a field modifier that provides two guarantees:

1. **Visibility Guarantee**: Changes to volatile variables are immediately visible to all threads
2. **Ordering Guarantee**: Prevents reordering of instructions around volatile access

**How Volatile Works:**
- Bypasses CPU cache (or forces cache coherence)
- Inserts memory barriers (fence instructions)
- Prevents compiler optimizations
- Forces reads from main memory
- Forces writes to main memory

**When to Use Volatile:**

| Use Case | Safe with Volatile | Unsafe with Volatile |
|----------|-------------------|---------------------|
| Status flags | ✅ `volatile boolean running` | ❌ Complex state updates |
| One writer, many readers | ✅ Single writer thread | ❌ Multiple writers |
| Double-checked locking | ✅ With immutable objects | ❌ With mutable objects |
| Simple state publishing | ✅ Reference assignment | ❌ Compound operations |
| Cache invalidation | ✅ Version counters | ❌ Increment operations |

```java
public class VolatileDeepDive {
    
    // Example 1: Status Flag (CORRECT USAGE)
    private volatile boolean shutdown = false;
    
    public void shutdownGracefully() {
        shutdown = true; // Visible to all threads immediately
        // Memory barrier ensures all previous writes are visible
    }
    
    public void workerThread() {
        while (!shutdown) { // Reads from main memory each iteration
            doWork();
            // Without volatile, compiler might optimize to:
            // if (!shutdown) while(true) doWork();
        }
        cleanup();
    }
    
    // Example 2: One Writer, Multiple Readers (CORRECT USAGE)
    private volatile Config config = new Config();
    
    public void updateConfig(Config newConfig) {
        // Single writer thread
        config = newConfig; // Atomic reference assignment
    }
    
    public Config getConfig() {
        // Multiple reader threads
        return config; // Always sees latest reference
    }
    
    // Example 3: INCORRECT USAGE - Compound Operations
    private volatile int counter = 0;
    
    public void incorrectIncrement() {
        counter++; // NOT ATOMIC!
        // Actually three operations:
        // 1. Read counter
        // 2. Add 1
        // 3. Write counter
        // Another thread can interleave between these
    }
    
    // Example 4: Happens-Before Demonstration
    private int x = 0;
    private int y = 0;
    private volatile boolean ready = false;
    
    public void writer() {
        x = 42;         // 1. Normal write
        y = 13;         // 2. Normal write  
        ready = true;   // 3. Volatile write - creates happens-before edge
    }
    
    public void reader() {
        if (ready) {    // 4. Volatile read
            // Guaranteed to see x=42 and y=13
            // Due to happens-before relationship
            assert x == 42;
            assert y == 13;
        }
    }
    
    // Example 5: Double-Checked Locking (CORRECT with volatile)
    private volatile Singleton instance;
    
    public Singleton getInstance() {
        Singleton result = instance; // Local variable for performance
        if (result == null) { // First check (no locking)
            synchronized(this) {
                result = instance;
                if (result == null) { // Second check (with locking)
                    instance = result = new Singleton();
                    // Without volatile, these could be reordered:
                    // 1. Allocate memory
                    // 2. Assign to instance
                    // 3. Initialize object
                    // Another thread might see partially constructed object!
                }
            }
        }
        return result;
    }
    
    // Example 6: Performance Comparison
    public void performanceTest() {
        long start;
        int iterations = 100_000_000;
        
        // Volatile read: ~5-10ns
        start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            boolean b = shutdown;
        }
        System.out.println("Volatile read: " + 
            (System.nanoTime() - start) / iterations + "ns");
        
        // Synchronized read: ~20-50ns
        start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            synchronized(this) {
                boolean b = shutdown;
            }
        }
        System.out.println("Synchronized read: " + 
            (System.nanoTime() - start) / iterations + "ns");
    }
    
    // Example 7: Common Pitfall - Volatile Arrays
    private volatile int[] array = new int[10];
    
    public void arrayPitfall() {
        array[0] = 42; // NOT VOLATILE!
        // Only the array reference is volatile, not elements
        // Other threads might not see this change
        
        // To ensure visibility, need to write to volatile:
        array = array; // Forces memory barrier
        // Or use AtomicIntegerArray
    }
    
    private void doWork() {}
    private void cleanup() {}
    private static class Config {}
    private static class Singleton {}
}
```

**Memory Barrier Effects:**

```java
public class MemoryBarrierExplanation {
    private int a, b, c, d;
    private volatile int v;
    private int x, y, z;
    
    public void demonstrateBarriers() {
        // Thread 1: Writer
        a = 1;  // Normal store
        b = 2;  // Normal store
        c = 3;  // Normal store
        v = 4;  // Volatile store (StoreStore + StoreLoad barrier)
        x = 5;  // Normal store
        y = 6;  // Normal store
        
        // Thread 2: Reader
        int r1 = v;  // Volatile load (LoadLoad + LoadStore barrier)
        // If r1 == 4, guaranteed to see a=1, b=2, c=3
        // NOT guaranteed to see x=5, y=6 (they're after volatile write)
        
        int r2 = a;  // Guaranteed to see 1 if r1 == 4
        int r3 = x;  // Might see 0 or 5
    }
}
```

**Volatile vs Synchronized vs Atomic - Detailed Comparison:**

| Aspect | volatile | synchronized | Atomic Classes |
|--------|----------|--------------|----------------|
| **Mutual Exclusion** | ❌ No | ✅ Yes | ❌ No (but has CAS) |
| **Visibility** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Atomicity** | ✅ Read/Write only | ✅ Full block | ✅ Specific operations |
| **Performance** | Fast (~5ns) | Slower (~50ns) | Fast (~10ns) |
| **Blocking** | ❌ Never blocks | ✅ Can block | ❌ Never blocks |
| **Compound Operations** | ❌ Not safe | ✅ Safe | ✅ Safe (specific) |
| **Memory Overhead** | None | Monitor object | Object wrapper |
| **Use Case** | Flags, references | Critical sections | Counters, updates |
| **Scalability** | Excellent | Poor under contention | Good |
| **Deadlock Risk** | ❌ None | ✅ Possible | ❌ None |

**Decision Tree:**
```
Need thread safety?
├─ No → Regular field
└─ Yes → Need atomicity?
    ├─ No (just visibility) → volatile
    └─ Yes → Compound operations?
        ├─ No (just increment/CAS) → Atomic classes
        └─ Yes → Multiple operations?
            ├─ No → Atomic classes with custom updater
            └─ Yes → Need blocking?
                ├─ No → Lock-free algorithm
                └─ Yes → synchronized or Lock
```

**Real-World Examples:**

```java
public class RealWorldComparison {
    // Use volatile for: Configuration flags
    private volatile boolean debugMode = false;
    
    // Use synchronized for: Complex state transitions
    private State state = State.IDLE;
    public synchronized void transitionState(State newState) {
        if (isValidTransition(state, newState)) {
            State oldState = state;
            state = newState;
            notifyListeners(oldState, newState);
        }
    }
    
    // Use Atomic for: Metrics and counters
    private final AtomicLong requestCount = new AtomicLong();
    private final AtomicReference<Status> status = new AtomicReference<>();
    
    private enum State { IDLE, RUNNING, STOPPED }
    private enum Status {}
    private boolean isValidTransition(State from, State to) { return true; }
    private void notifyListeners(State old, State new) {}
}
```

**Key Takeaways:**
- **volatile**: Lightweight visibility, no compound operations
- **synchronized**: Heavyweight but complete thread safety
- **Atomic**: Middle ground for specific operations
- **Performance**: volatile > Atomic > synchronized (under contention)
- **Complexity**: synchronized > Atomic > volatile
- **Common mistake**: Using volatile for compound operations

---

## Java Memory Model

### Q: Explain the Java Memory Model (JMM) and happens-before relationship

**Context & Problem:**
Modern computer architectures create challenges for concurrent programming:
- **Multiple CPU Cores**: Each with private caches (L1, L2, sometimes L3)
- **Memory Hierarchy**: Registers → L1 → L2 → L3 → Main Memory
- **Performance Optimizations**: Reordering, caching, speculative execution
- **Weak Consistency**: Different cores may see different values

Without a memory model, the same Java program could behave differently on different hardware!

**Deep Dive Answer:**

The Java Memory Model (JMM) is a specification that defines:
1. **How threads interact through memory**
2. **What values a thread can see when reading shared variables**
3. **When changes made by one thread become visible to others**
4. **Which optimizations are legal for compilers and processors**

**Why JMM Matters:**
- **Portability**: Same behavior across different hardware architectures
- **Performance**: Allows aggressive optimizations while maintaining correctness
- **Predictability**: Developers can reason about concurrent behavior
- **Safety**: Prevents data races and ensures memory consistency

**Core Components of JMM:**

1. **Memory Architecture in JVM:**

```java
public class MemoryArchitecture {
    // HEAP MEMORY (Shared across all threads)
    private int instanceField = 0;          // Heap - instance variables
    private static int staticField = 0;     // Heap - method area/metaspace
    private final Object lock = new Object(); // Heap - object instances
    private int[] array = new int[100];     // Heap - arrays
    
    // THREAD STACK (Private to each thread)
    public void demonstrateMemoryLayout() {
        // Stack Frame for this method
        int localVariable = 42;              // Stack - local variables
        long parameter = 100L;               // Stack - method parameters
        
        // Reference on stack, object on heap
        MyObject obj = new MyObject();       // obj reference: Stack
                                             // MyObject instance: Heap
        
        // Primitive array vs Object array
        int[] primitiveArray = new int[10];  // Reference: Stack, Array: Heap
        Object[] objectArray = new Object[10]; // Reference: Stack
                                              // Array: Heap
                                              // Objects in array: Heap
        
        // Method call creates new stack frame
        recursiveMethod(5);                  // Each call adds stack frame
    }
    
    private void recursiveMethod(int depth) {
        if (depth > 0) {
            int localToFrame = depth * 10;   // Each frame has own copy
            recursiveMethod(depth - 1);      // New stack frame
        } // Stack frame popped when method returns
    }
    
    // Cache Line and False Sharing Example
    static class CacheLineDemo {
        // CPU cache works in cache lines (typically 64 bytes)
        private volatile long value1 = 0L;  // 8 bytes
        private volatile long value2 = 0L;  // 8 bytes - same cache line!
        
        // False sharing: Different threads updating value1 and value2
        // cause cache line bouncing between cores
        
        // Solution: Padding to separate cache lines
        private volatile long value3 = 0L;
        private long p1, p2, p3, p4, p5, p6, p7; // Padding (56 bytes)
        private volatile long value4 = 0L;  // Different cache line
    }
    
    private static class MyObject {}
}

// Memory visibility problem demonstration
public class VisibilityProblem {
    private static boolean stopRequested = false; // Shared variable
    
    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int counter = 0;
            while (!stopRequested) {  // May never see update!
                counter++;
                // Without synchronization, compiler may optimize to:
                // if (!stopRequested) while(true) counter++;
            }
            System.out.println("Stopped at: " + counter);
        });
        
        backgroundThread.start();
        Thread.sleep(1000);
        stopRequested = true;  // Main thread writes
        // Without volatile/synchronization, background thread may never stop!
    }
}
```

2. **Happens-Before Guarantees (The Foundation of JMM):**

**What is Happens-Before?**
A happens-before relationship guarantees that memory writes by one statement are visible to another statement.

```java
public class HappensBeforeDeepDive {
    // Example 1: Volatile Variables
    private volatile int v = 0;
    private int a = 0;
    private int b = 0;
    private int c = 0;
    
    public void volatileHappensBefore() {
        // Thread 1: Writer
        a = 1;      // 1: Normal write
        b = 2;      // 2: Normal write
        v = 3;      // 3: Volatile write (release semantics)
        c = 4;      // 4: Normal write
        
        // Thread 2: Reader
        int r1 = v; // 5: Volatile read (acquire semantics)
        if (r1 == 3) {
            // Happens-before edge from line 3 to line 5
            assert a == 1; // ✓ Guaranteed
            assert b == 2; // ✓ Guaranteed
            // c might be 0 or 4 (no guarantee for operations after volatile write)
        }
    }
    
    // Example 2: Synchronized Blocks
    private final Object lock = new Object();
    private int x = 0;
    private int y = 0;
    
    public void synchronizedHappensBefore() {
        // Thread 1
        synchronized (lock) {
            x = 1;
            y = 2;
        } // Monitor exit (release)
        
        // Thread 2
        synchronized (lock) { // Monitor enter (acquire)
            // Guaranteed to see x=1, y=2 if Thread 1 executed first
            assert x == 1;
            assert y == 2;
        }
    }
    
    // Example 3: Thread Start and Join
    class ThreadHappensBefore {
        private int value = 0;
        
        public void demonstrateThreadRules() throws InterruptedException {
            Thread thread = new Thread(() -> {
                // Everything before thread.start() happens-before
                // everything in the thread's run method
                assert value == 42; // Guaranteed if set before start()
                value = 100;
            });
            
            value = 42;
            thread.start(); // Happens-before edge
            
            thread.join();  // Happens-before edge
            // Everything in thread happens-before everything after join()
            assert value == 100; // Guaranteed
        }
    }
    
    // Example 4: Final Fields
    class FinalFieldGuarantee {
        private final int x;
        private final List<String> list;
        
        public FinalFieldGuarantee() {
            x = 42;
            list = new ArrayList<>();
            list.add("initialized");
            // Constructor completion happens-before
            // any access to the object by other threads
        }
        
        // Other threads guaranteed to see x=42 and initialized list
        // (assuming proper publication of the object)
    }
    
    // Example 5: Demonstrating Reordering
    class ReorderingExample {
        private int a = 0, b = 0;
        private int r1 = 0, r2 = 0;
        
        public void demonstratePossibleReordering() {
            // Thread 1
            a = 1;  // Can be reordered
            r1 = b; // Can be reordered
            
            // Thread 2
            b = 1;  // Can be reordered
            r2 = a; // Can be reordered
            
            // Possible outcomes:
            // (r1=0, r2=0) - Each thread reads before other writes
            // (r1=0, r2=1) - Thread 2 completes before Thread 1 reads
            // (r1=1, r2=0) - Thread 1 completes before Thread 2 reads
            // (r1=1, r2=1) - Both threads write before reading
            
            // Without synchronization, all outcomes are legal!
        }
    }
}
```

3. **Memory Barriers and Fences:**

```java
public class MemoryBarrierInDepth {
    // Memory barriers prevent reordering and ensure visibility
    
    private volatile boolean flag = false;
    private int data = 0;
    
    // Types of Memory Barriers:
    // 1. LoadLoad: Prevents reordering of loads
    // 2. LoadStore: Load before subsequent stores
    // 3. StoreLoad: Store before subsequent loads (most expensive)
    // 4. StoreStore: Store before subsequent stores
    
    public void demonstrateBarriers() {
        // Writer thread
        data = 42;          // Normal store
        // StoreStore barrier (prevents reordering with next store)
        flag = true;        // Volatile store
        // StoreLoad barrier (full fence)
        
        // Reader thread
        if (flag) {         // Volatile load
            // LoadLoad barrier (prevents reordering with next load)
            // LoadStore barrier (prevents reordering with stores)
            int localData = data; // Guaranteed to see 42
        }
    }
    
    // Demonstrating with Unsafe (internal JVM class)
    public class UnsafeBarrierExample {
        // Pseudo-code showing what JVM does internally
        public void showInternalBarriers() {
            // Normal write
            data = 42;
            
            // What volatile write actually does:
            // Unsafe.storeFence();     // StoreStore barrier
            // flag = true;
            // Unsafe.fullFence();      // StoreLoad barrier
            
            // What volatile read actually does:
            // boolean local = flag;
            // Unsafe.loadFence();      // LoadLoad + LoadStore barriers
        }
    }
    
    // Real-world example: Lock-free publishing
    class SafePublication {
        private volatile Node head;
        
        class Node {
            final int value;
            final Node next;
            
            Node(int value, Node next) {
                this.value = value;
                this.next = next;
            }
        }
        
        public void publish(int value) {
            // Create new node (all writes happen-before volatile write)
            Node newNode = new Node(value, head);
            // Volatile write ensures safe publication
            head = newNode; // Memory barrier here
            // Any thread reading head will see fully constructed Node
        }
        
        public Node getHead() {
            return head; // Volatile read - sees complete object
        }
    }
    
    // Performance impact of barriers
    public void performanceComparison() {
        long iterations = 100_000_000L;
        long start;
        
        // Without barriers (normal field)
        int normalField = 0;
        start = System.nanoTime();
        for (long i = 0; i < iterations; i++) {
            normalField = (int) i;
        }
        System.out.println("Normal write: " + 
            (System.nanoTime() - start) / iterations + "ns");
        // Typical: 0.3-0.5ns
        
        // With barriers (volatile field)
        start = System.nanoTime();
        for (long i = 0; i < iterations; i++) {
            flag = (i % 2 == 0);
        }
        System.out.println("Volatile write: " + 
            (System.nanoTime() - start) / iterations + "ns");
        // Typical: 5-10ns (10-20x slower)
    }
}
```

### Q: What are the key happens-before rules?

**Context & Problem:**
Without happens-before rules, compilers and CPUs could reorder operations in ways that break program logic. These rules define the minimal guarantees that allow developers to write correct concurrent programs.

**Deep Dive Answer:**

**The Complete Happens-Before Rules:**

1. **Program Order Rule**
   - **What**: Each action in a thread happens-before every subsequent action in that thread
   - **Why**: Preserves sequential consistency within a single thread
   - **Example**: `x = 1; y = 2;` - x assignment happens-before y assignment

2. **Monitor Lock Rule**
   - **What**: Unlock of a monitor happens-before every subsequent lock of that monitor
   - **Why**: Ensures critical sections see previous modifications
   - **Real Impact**: Database connection pools, synchronized collections

3. **Volatile Variable Rule**
   - **What**: Write to volatile happens-before every subsequent read of that volatile
   - **Why**: Provides lightweight synchronization for simple variables
   - **Use Case**: Configuration flags, shutdown signals

4. **Thread Start Rule**
   - **What**: Thread.start() happens-before any action in the started thread
   - **Why**: Ensures thread sees initialization before execution
   - **Common Bug Fixed**: Thread seeing uninitialized fields

5. **Thread Termination Rule**
   - **What**: All actions in thread happen-before Thread.join() returns
   - **Why**: Allows main thread to see worker thread's results
   - **Pattern**: Fork-join parallelism

6. **Thread Interruption Rule**
   - **What**: Thread.interrupt() happens-before interrupted thread detects interruption
   - **Why**: Ensures interrupt flag visibility

7. **Finalizer Rule**
   - **What**: Constructor completion happens-before finalizer start
   - **Why**: Prevents finalizer from seeing partially constructed object

8. **Transitivity Rule**
   - **What**: If A happens-before B, and B happens-before C, then A happens-before C
   - **Why**: Allows chaining of guarantees
   - **Power**: Enables complex synchronization patterns

```java
public class HappensBeforeRulesInAction {
    
    // 1. Program Order Rule
    public void programOrderDemo() {
        int a = 1;      // Action 1
        int b = 2;      // Action 2 (happens-after Action 1)
        int c = a + b;  // Action 3 (happens-after Actions 1 & 2)
        // Within thread: guaranteed order
        // Across threads: no guarantee without synchronization
    }
    
    // 2. Monitor Lock Rule  
    private final Object lock = new Object();
    private int sharedData = 0;
    
    public void monitorLockDemo() {
        // Thread 1
        synchronized (lock) {
            sharedData = 42;
        } // Unlock happens-before...
        
        // Thread 2
        synchronized (lock) { // ...this lock
            assert sharedData == 42; // Guaranteed if Thread 1 ran first
        }
    }
    
    // 3. Volatile Variable Rule
    private volatile int volatileCounter = 0;
    private int normalCounter = 0;
    
    public void volatileRuleDemo() {
        // Thread 1
        normalCounter = 100;     // Normal write
        volatileCounter = 200;   // Volatile write happens-before...
        
        // Thread 2
        if (volatileCounter == 200) { // ...this volatile read
            assert normalCounter == 100; // Guaranteed
        }
    }
    
    // 4. Thread Start Rule
    public void threadStartDemo() {
        final int[] array = new int[1];
        array[0] = 42; // Happens-before thread.start()
        
        Thread worker = new Thread(() -> {
            assert array[0] == 42; // Guaranteed to see 42
            array[0] = 100;
        });
        
        worker.start(); // Happens-before any action in worker
    }
    
    // 5. Thread Join Rule
    public void threadJoinDemo() throws InterruptedException {
        final int[] result = new int[1];
        
        Thread calculator = new Thread(() -> {
            // Complex calculation
            result[0] = 42 * 42;
        }); // All actions happen-before join() returns
        
        calculator.start();
        calculator.join(); // Wait for completion
        
        assert result[0] == 1764; // Guaranteed to see result
    }
    
    // 6. Transitivity in Action
    private volatile boolean phase1Done = false;
    private volatile boolean phase2Done = false;
    private int data1 = 0;
    private int data2 = 0;
    
    public void transitivityDemo() {
        // Thread 1
        data1 = 100;           // Action A
        phase1Done = true;     // Action B (A happens-before B)
        
        // Thread 2
        while (!phase1Done) {} // Action C (B happens-before C)
        data2 = data1 + 50;    // Action D
        phase2Done = true;     // Action E (D happens-before E)
        
        // Thread 3
        while (!phase2Done) {} // Action F (E happens-before F)
        // By transitivity: A happens-before F
        assert data1 == 100;   // Guaranteed
        assert data2 == 150;   // Guaranteed
    }
    
    // 7. Complex Real-World Example
    public class ProducerConsumerWithHappensBefore {
        private final BlockingQueue<Task> queue = new LinkedBlockingQueue<>();
        private volatile boolean shutdown = false;
        
        class Task {
            final String data;
            final long timestamp;
            
            Task(String data) {
                this.data = data;
                this.timestamp = System.nanoTime();
            }
        }
        
        public void producer() throws InterruptedException {
            int taskCount = 0;
            while (!shutdown) { // Volatile read
                Task task = new Task("Task-" + taskCount++);
                queue.put(task); // Happens-before relationship via BlockingQueue
            }
        }
        
        public void consumer() throws InterruptedException {
            while (!shutdown || !queue.isEmpty()) {
                Task task = queue.poll(1, TimeUnit.SECONDS);
                if (task != null) {
                    // Guaranteed to see complete Task object
                    // due to happens-before in BlockingQueue
                    processTask(task);
                }
            }
        }
        
        public void shutdown() {
            shutdown = true; // Volatile write - visible to all threads
        }
        
        private void processTask(Task task) {
            // Process task with full visibility of its fields
        }
    }
}
```

**Practical Implications and Common Pitfalls:**

```java
public class HappensBeforePitfalls {
    
    // PITFALL 1: Assuming happens-before without synchronization
    private int x = 0;
    private boolean ready = false; // NOT volatile!
    
    public void incorrectAssumption() {
        // Thread 1
        x = 42;
        ready = true;
        
        // Thread 2
        if (ready) {
            // x might still be 0! No happens-before relationship
            System.out.println(x); // Could print 0 or 42
        }
    }
    
    // PITFALL 2: Partial synchronization
    private int counter = 0;
    
    public synchronized void increment() {
        counter++;
    }
    
    public int getCount() { // NOT synchronized!
        return counter; // No happens-before, might see stale value
    }
    
    // PITFALL 3: Breaking transitivity chain
    private volatile boolean flag1 = false;
    private boolean flag2 = false; // NOT volatile - breaks chain!
    private volatile boolean flag3 = false;
    private int data = 0;
    
    public void brokenChain() {
        // Thread 1
        data = 42;
        flag1 = true; // Volatile write
        
        // Thread 2
        if (flag1) {
            flag2 = true; // Normal write - no happens-before
        }
        
        // Thread 3
        if (flag2) { // Normal read - no happens-before
            flag3 = true;
            // data might still be 0! Chain is broken
        }
    }
}
```

**Key Takeaways:**
- **Happens-before is transitive**: Chain guarantees together
- **Not just about ordering**: Also about visibility
- **Minimal guarantees**: JVM can do more but not less
- **Design principle**: Use the weakest synchronization that's sufficient
- **Testing difficulty**: Happens-before violations may work on some platforms
- **Performance trade-off**: Stronger guarantees = more overhead

---

## Concurrent Collections

### Q: What are the thread-safe collections in Java?

**Context & Problem:**
Regular collections (ArrayList, HashMap, HashSet) fail catastrophically under concurrent access:
- **ConcurrentModificationException**: Iterator detects structural changes
- **Data Corruption**: Partial updates, lost elements, infinite loops
- **Memory Visibility Issues**: Changes not visible across threads
- **Performance Bottlenecks**: Coarse-grained locking kills scalability

**Deep Dive Answer:**

Java provides three generations of thread-safe collections, each solving different problems:

**1. Legacy Synchronized Collections (Avoid in Modern Code):**

```java
public class LegacyCollectionsAnalysis {
    
    // Synchronized Wrappers - Added in Java 1.2
    public void synchronizedWrappers() {
        List<String> syncList = Collections.synchronizedList(new ArrayList<>());
        Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
        Set<String> syncSet = Collections.synchronizedSet(new HashSet<>());
        
        // How they work: Every method synchronized on the collection itself
        // Equivalent to:
        class SynchronizedListImpl<E> implements List<E> {
            private final List<E> list;
            private final Object mutex;
            
            public boolean add(E e) {
                synchronized (mutex) {
                    return list.add(e);
                }
            }
            // All other methods similarly wrapped
        }
        
        // PROBLEM 1: Iteration still requires external synchronization
        synchronized (syncList) {
            Iterator<String> it = syncList.iterator();
            while (it.hasNext()) {
                // Without this synchronized block: ConcurrentModificationException
                String item = it.next();
            }
        }
        
        // PROBLEM 2: Compound operations aren't atomic
        if (!syncMap.containsKey("key")) { // Check
            // Another thread can insert here!
            syncMap.put("key", 1); // Act - not atomic with check
        }
        
        // PROBLEM 3: Poor performance under contention
        // Every operation locks entire collection
    }
    
    // Vector and Hashtable - Original synchronized collections (Java 1.0)
    public void legacyCollections() {
        Vector<String> vector = new Vector<>();
        Hashtable<String, String> hashtable = new Hashtable<>();
        
        // Why to avoid:
        // 1. Synchronization even when not needed (single-threaded use)
        // 2. Legacy API (Enumeration vs Iterator)
        // 3. Can't remove synchronization overhead
        // 4. Poor performance compared to modern alternatives
        
        // Performance comparison
        long start = System.nanoTime();
        for (int i = 0; i < 1000000; i++) {
            vector.add("item" + i); // Synchronized every call
        }
        long vectorTime = System.nanoTime() - start;
        
        ArrayList<String> arrayList = new ArrayList<>();
        start = System.nanoTime();
        for (int i = 0; i < 1000000; i++) {
            arrayList.add("item" + i); // No synchronization
        }
        long arrayListTime = System.nanoTime() - start;
        
        // Typical: Vector 2-3x slower than ArrayList in single-threaded use
    }
}
```

**2. Modern Concurrent Collections (Java 5+ Best Practices):**

```java
public class ConcurrentCollectionsDeepDive {
    
    // ConcurrentHashMap - The workhorse of concurrent collections
    public void concurrentHashMapDemo() {
        ConcurrentHashMap<String, AtomicInteger> map = new ConcurrentHashMap<>();
        
        // Thread-safe operations without external synchronization
        map.put("key", new AtomicInteger(0));
        
        // Atomic compound operations (Java 8+)
        map.compute("key", (k, v) -> {
            if (v == null) return new AtomicInteger(1);
            v.incrementAndGet();
            return v;
        });
        
        map.computeIfAbsent("newKey", k -> new AtomicInteger(0));
        map.computeIfPresent("key", (k, v) -> {
            v.incrementAndGet();
            return v;
        });
        
        // Merge for aggregation
        map.merge("counter", new AtomicInteger(1), 
            (oldVal, newVal) -> {
                oldVal.addAndGet(newVal.get());
                return oldVal;
            });
        
        // Bulk operations with parallelism threshold
        long sum = map.reduceValuesToLong(
            1, // Parallelism threshold
            v -> v.get(), // Transformer
            0, // Identity
            Long::sum // Reducer
        );
        
        // Safe iteration without ConcurrentModificationException
        map.forEach((k, v) -> System.out.println(k + "=" + v));
    }
    
    // CopyOnWriteArrayList - Optimal for read-heavy workloads
    public void copyOnWriteDemo() {
        CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
        
        // How it works:
        // - Writes create new copy of entire array
        // - Reads work on snapshot without locking
        // - Iterator sees snapshot at creation time
        
        // Write operation - expensive O(n)
        cowList.add("item"); // Copies entire array
        cowList.remove("item"); // Copies entire array
        
        // Read operation - cheap O(1)
        String item = cowList.get(0); // No locking needed
        
        // Safe iteration even during modification
        for (String s : cowList) {
            cowList.add("new"); // Won't affect this iteration
            // Iterator works on snapshot
        }
        
        // Use cases:
        // - Event listeners (rarely modified, frequently iterated)
        // - Configuration lists
        // - Blacklist/whitelist that rarely changes
        
        // When NOT to use:
        // - Frequent modifications (expensive array copying)
        // - Large lists (memory overhead of copying)
    }
    
    // Non-blocking queues for high-performance scenarios
    public void nonBlockingQueuesDemo() {
        // ConcurrentLinkedQueue - Lock-free FIFO queue
        ConcurrentLinkedQueue<Task> taskQueue = new ConcurrentLinkedQueue<>();
        
        // Producer thread - never blocks
        taskQueue.offer(new Task("task1")); // Always succeeds
        
        // Consumer thread - never blocks
        Task task = taskQueue.poll(); // Returns null if empty
        
        // ConcurrentLinkedDeque - Lock-free double-ended queue
        ConcurrentLinkedDeque<String> deque = new ConcurrentLinkedDeque<>();
        deque.addFirst("first");
        deque.addLast("last");
        String first = deque.pollFirst();
        String last = deque.pollLast();
        
        // Performance characteristics:
        // - Lock-free using CAS operations
        // - No blocking, suitable for real-time systems
        // - Unbounded - can cause memory issues
        // - O(1) operations but with CAS retry overhead
    }
    
    // Blocking queues for producer-consumer patterns
    public void blockingQueuesDemo() throws InterruptedException {
        // ArrayBlockingQueue - Bounded, backed by array
        BlockingQueue<String> arrayQueue = new ArrayBlockingQueue<>(100);
        
        // LinkedBlockingQueue - Optionally bounded, backed by linked nodes
        BlockingQueue<String> linkedQueue = new LinkedBlockingQueue<>(1000);
        
        // PriorityBlockingQueue - Unbounded priority queue
        BlockingQueue<Task> priorityQueue = new PriorityBlockingQueue<>();
        
        // SynchronousQueue - Zero capacity, direct handoff
        SynchronousQueue<String> syncQueue = new SynchronousQueue<>();
        
        // DelayQueue - Elements available after delay
        DelayQueue<DelayedTask> delayQueue = new DelayQueue<>();
        
        // LinkedTransferQueue - Combines features
        LinkedTransferQueue<String> transferQueue = new LinkedTransferQueue<>();
        
        // Blocking operations
        arrayQueue.put("item"); // Blocks if full
        String item = arrayQueue.take(); // Blocks if empty
        
        // Timed operations
        boolean offered = arrayQueue.offer("item", 5, TimeUnit.SECONDS);
        String polled = arrayQueue.poll(5, TimeUnit.SECONDS);
        
        // Transfer semantics (LinkedTransferQueue)
        transferQueue.transfer("item"); // Blocks until consumed
        boolean transferred = transferQueue.tryTransfer("item", 1, TimeUnit.SECONDS);
    }
    
    // Concurrent sorted collections
    public void sortedCollectionsDemo() {
        // ConcurrentSkipListMap - Concurrent NavigableMap
        ConcurrentSkipListMap<Integer, String> skipMap = new ConcurrentSkipListMap<>();
        
        // O(log n) operations
        skipMap.put(5, "five");
        skipMap.put(3, "three");
        skipMap.put(7, "seven");
        
        // Navigation operations
        Map.Entry<Integer, String> first = skipMap.firstEntry();
        Map.Entry<Integer, String> last = skipMap.lastEntry();
        Map.Entry<Integer, String> ceiling = skipMap.ceilingEntry(4); // Returns 5
        Map.Entry<Integer, String> floor = skipMap.floorEntry(4); // Returns 3
        
        // Sub-map views
        NavigableMap<Integer, String> subMap = skipMap.subMap(2, 6);
        
        // ConcurrentSkipListSet - Concurrent NavigableSet
        ConcurrentSkipListSet<Integer> skipSet = new ConcurrentSkipListSet<>();
        skipSet.add(5);
        skipSet.add(3);
        
        // How SkipList works:
        // - Probabilistic balanced tree structure
        // - Multiple levels of linked lists
        // - Average O(log n) operations
        // - Lock-free using CAS
    }
    
    private static class Task implements Comparable<Task> {
        String name;
        int priority;
        Task(String name) { this.name = name; }
        public int compareTo(Task other) {
            return Integer.compare(this.priority, other.priority);
        }
    }
    
    private static class DelayedTask implements Delayed {
        private final long delayTime;
        private final String task;
        
        DelayedTask(String task, long delayMs) {
            this.task = task;
            this.delayTime = System.currentTimeMillis() + delayMs;
        }
        
        @Override
        public long getDelay(TimeUnit unit) {
            long diff = delayTime - System.currentTimeMillis();
            return unit.convert(diff, TimeUnit.MILLISECONDS);
        }
        
        @Override
        public int compareTo(Delayed o) {
            return Long.compare(this.delayTime, ((DelayedTask) o).delayTime);
        }
    }
}
```

**Choosing the Right Collection:**

| Use Case | Best Choice | Why |
|----------|-------------|-----|
| General concurrent map | ConcurrentHashMap | Best performance, rich API |
| Read-heavy list | CopyOnWriteArrayList | No locking on reads |
| Write-heavy list | Collections.synchronizedList | Avoid array copying |
| Producer-consumer queue | LinkedBlockingQueue | Balanced performance |
| Task scheduling | DelayQueue | Built-in delay support |
| Priority tasks | PriorityBlockingQueue | Automatic ordering |
| Direct handoff | SynchronousQueue | Zero buffering |
| Sorted concurrent map | ConcurrentSkipListMap | Concurrent navigation |
| High-performance queue | ConcurrentLinkedQueue | Lock-free |

**Performance Characteristics:**

```java
public class CollectionPerformanceComparison {
    public void compareMapPerformance() {
        int threads = 8;
        int operations = 1_000_000;
        
        // Test different map implementations
        Map<Integer, String> hashMap = new HashMap<>();
        Map<Integer, String> syncMap = Collections.synchronizedMap(new HashMap<>());
        Map<Integer, String> concurrentMap = new ConcurrentHashMap<>();
        
        // Measure throughput under contention
        // Typical results (ops/sec):
        // HashMap (unsafe): Data corruption
        // Synchronized Map: ~2M ops/sec
        // ConcurrentHashMap: ~15M ops/sec
    }
}
```

### Q: Explain ConcurrentHashMap internals and improvements in Java 8

**Context & Problem:**
HashMap fails under concurrent access with:
- **Infinite loops**: Resize operation can create circular references
- **Data loss**: Concurrent puts can overwrite each other
- **Memory leaks**: Corrupted internal structure
- **Inconsistent size**: Size counter becomes incorrect

SynchronizedMap solves safety but kills performance with global lock.

**Deep Dive Answer:**

**Evolution of ConcurrentHashMap:**

**Java 7 and Earlier - Segment-Based Locking:**

```java
public class ConcurrentHashMapJava7Internals {
    // Conceptual structure (simplified)
    class Segment<K,V> extends ReentrantLock {
        volatile HashEntry<K,V>[] table;
        volatile int count;
        volatile int modCount;
        
        V put(K key, int hash, V value, boolean onlyIfAbsent) {
            lock(); // Lock only this segment
            try {
                // Locate bucket
                int index = hash & (table.length - 1);
                HashEntry<K,V> first = table[index];
                
                // Search in chain
                for (HashEntry<K,V> e = first; e != null; e = e.next) {
                    if (e.hash == hash && key.equals(e.key)) {
                        V oldValue = e.value;
                        if (!onlyIfAbsent) {
                            e.value = value;
                        }
                        return oldValue;
                    }
                }
                
                // Add new entry
                HashEntry<K,V> newEntry = new HashEntry<>(key, hash, first, value);
                table[index] = newEntry;
                count++;
                return null;
            } finally {
                unlock();
            }
        }
    }
    
    // Default: 16 segments (concurrency level)
    // Each segment: Independent hash table with own lock
    // Maximum concurrent writers: 16 (one per segment)
    // Readers: Lock-free using volatile
    
    // Problems with segment-based approach:
    // 1. Fixed concurrency level (16)
    // 2. Uneven distribution possible
    // 3. Memory overhead of segments
    // 4. Complex resizing logic
}
```

**Java 8+ Revolutionary Improvements:**

```java
public class ConcurrentHashMapJava8Internals {
    
    // New internal structure
    class Node<K,V> {
        final int hash;
        final K key;
        volatile V val;        // Volatile for visibility
        volatile Node<K,V> next;
    }
    
    // Tree nodes for large buckets
    class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;
        boolean red;
    }
    
    // Key improvements demonstration
    public void demonstrateJava8Features() {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        
        // 1. Node-based locking (bin-level synchronization)
        // Instead of segment locks, synchronizes on first node of each bin
        // Allows thousands of concurrent updates to different bins
        
        // 2. Tree bins for collision handling
        // When chain length > 8: Convert to red-black tree
        // When tree size < 6: Convert back to linked list
        // O(log n) worst case instead of O(n)
        
        // 3. Lazy initialization
        // Table initialized on first put
        // Saves memory for empty maps
        
        // 4. Better resizing
        // Multiple threads can help with resizing
        // Transfer work divided among threads
        
        // How put() works in Java 8+
        demonstratePutOperation(map);
        
        // Atomic operations without explicit locking
        demonstrateAtomicOperations(map);
        
        // Parallel bulk operations
        demonstrateBulkOperations(map);
    }
    
    private void demonstratePutOperation(ConcurrentHashMap<String, Integer> map) {
        // Simplified put algorithm
        // 1. If table not initialized, initialize it
        // 2. If bin is empty, CAS new node (no lock needed)
        // 3. If bin is being resized (ForwardingNode), help resize
        // 4. Otherwise, lock the first node and insert
        
        // Performance: No lock for empty bins!
        map.put("key1", 1); // If bin empty: CAS operation, no lock
        
        // Lock only when collision occurs
        map.put("key2", 2); // If collision: lock first node only
    }
    
    private void demonstrateAtomicOperations(ConcurrentHashMap<String, Integer> map) {
        // Compute - atomic read-modify-write
        map.compute("counter", (k, v) -> {
            // This lambda runs atomically
            if (v == null) return 1;
            return v + 1;
        });
        
        // ComputeIfAbsent - lazy initialization
        map.computeIfAbsent("expensive", k -> {
            // Only computed once, even with concurrent calls
            return expensiveComputation();
        });
        
        // ComputeIfPresent - conditional update
        map.computeIfPresent("existing", (k, v) -> {
            return v > 10 ? v * 2 : v;
        });
        
        // Merge - perfect for aggregation
        map.merge("sum", 5, Integer::sum);
        
        // Replace with CAS
        map.replace("key", 1, 2); // CAS-based
    }
    
    private void demonstrateBulkOperations(ConcurrentHashMap<String, Integer> map) {
        // Initialize with data
        for (int i = 0; i < 10000; i++) {
            map.put("key" + i, i);
        }
        
        // Parallel forEach with threshold
        map.forEach(100, // Parallelism threshold
            (k, v) -> {
                // Executed in parallel if size > 100
                System.out.println(Thread.currentThread().getName() + ": " + k);
            });
        
        // Parallel search - stops when found
        String found = map.search(100,
            (k, v) -> v > 5000 ? k : null);
        
        // Parallel reduce
        Integer sum = map.reduce(100,
            (k, v) -> v,           // Transformer
            Integer::sum);         // Reducer
        
        // Parallel reduce with keys and values
        Integer maxValue = map.reduceValues(100,
            Integer::max);
        
        // More accurate size for large maps
        long exactSize = map.mappingCount(); // Better than size()
    }
    
    private int expensiveComputation() {
        return 42;
    }
    
    // Performance comparison
    public void performanceComparison() {
        int threads = 16;
        int operations = 1_000_000;
        
        // Java 7: ~5M ops/sec with 16 segments
        // Java 8: ~15M ops/sec with node-based locking
        // 3x improvement typical
    }
}
```

**Internal Optimizations Deep Dive:**

```java
public class ConcurrentHashMapOptimizations {
    
    // 1. Spread function for better hash distribution
    static final int spread(int h) {
        // XOR higher bits with lower bits
        return (h ^ (h >>> 16)) & 0x7fffffff;
    }
    
    // 2. Table size always power of 2
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= (1 << 30)) ? (1 << 30) : n + 1;
    }
    
    // 3. Special node types
    static class ForwardingNode<K,V> extends Node<K,V> {
        // Used during resizing to indicate bin is being moved
        final Node<K,V>[] nextTable;
        
        ForwardingNode(Node<K,V>[] tab) {
            super(MOVED, null, null, null);
            this.nextTable = tab;
        }
    }
    
    // 4. CounterCells for size tracking
    static final class CounterCell {
        volatile long value;
        // Padded to avoid false sharing
    }
    
    // 5. Transfer operation for resizing
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        // Multiple threads can participate in resizing
        // Each thread claims a range of bins to transfer
        // Uses stride to divide work
    }
    
    private static final int MOVED = -1;    // ForwardingNode
    private static final int TREEBIN = -2;  // TreeBin root
}
```

**Comprehensive Performance Analysis:**

```java
public class ConcurrentHashMapPerformanceAnalysis {
    private static final int THREADS = 16;
    private static final int OPERATIONS = 1_000_000;
    
    public void comprehensiveBenchmark() throws Exception {
        // Different scenarios
        benchmarkPutPerformance();
        benchmarkGetPerformance();
        benchmarkMixedOperations();
        benchmarkComputeOperations();
        benchmarkResizing();
    }
    
    private void benchmarkPutPerformance() throws Exception {
        Map<Integer, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
        ConcurrentHashMap<Integer, Integer> concurrentMap7 = new ConcurrentHashMap<>(16); // Java 7 style
        ConcurrentHashMap<Integer, Integer> concurrentMap8 = new ConcurrentHashMap<>(); // Java 8
        
        // Test with high contention (same keys)
        System.out.println("High Contention (same keys):");
        long syncTime = testMap(syncMap, true);
        long concurrent7Time = testMap(concurrentMap7, true);
        long concurrent8Time = testMap(concurrentMap8, true);
        
        System.out.println("Synchronized: " + syncTime + "ms");
        System.out.println("Concurrent7: " + concurrent7Time + "ms");
        System.out.println("Concurrent8: " + concurrent8Time + "ms");
        // Typical: Concurrent8 3x faster than Synchronized
        
        // Test with low contention (different keys)
        System.out.println("\nLow Contention (different keys):");
        syncTime = testMap(syncMap, false);
        concurrent8Time = testMap(concurrentMap8, false);
        
        System.out.println("Synchronized: " + syncTime + "ms");
        System.out.println("Concurrent8: " + concurrent8Time + "ms");
        // Typical: Concurrent8 10x faster than Synchronized
    }
    
    private long testMap(Map<Integer, Integer> map, boolean highContention) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(THREADS);
        CountDownLatch latch = new CountDownLatch(THREADS);
        
        long start = System.nanoTime();
        
        for (int i = 0; i < THREADS; i++) {
            final int threadId = i;
            executor.submit(() -> {
                for (int j = 0; j < OPERATIONS; j++) {
                    int key = highContention ? 
                        j % 100 : // Same 100 keys
                        threadId * OPERATIONS + j; // Different keys per thread
                    map.put(key, j);
                }
                latch.countDown();
            });
        }
        
        latch.await();
        long elapsed = System.nanoTime() - start;
        executor.shutdown();
        
        return TimeUnit.NANOSECONDS.toMillis(elapsed);
    }
    
    private void benchmarkComputeOperations() {
        ConcurrentHashMap<String, LongAdder> metrics = new ConcurrentHashMap<>();
        
        // Traditional approach - not atomic
        LongAdder counter = metrics.get("requests");
        if (counter == null) {
            counter = new LongAdder();
            LongAdder existing = metrics.putIfAbsent("requests", counter);
            if (existing != null) {
                counter = existing;
            }
        }
        counter.increment();
        
        // Java 8 approach - atomic and cleaner
        metrics.computeIfAbsent("requests", k -> new LongAdder()).increment();
        
        // Performance: computeIfAbsent 2x faster and truly atomic
    }
    
    private void benchmarkResizing() {
        ConcurrentHashMap<Integer, Integer> map = new ConcurrentHashMap<>(16);
        
        // Force multiple resizes
        for (int i = 0; i < 1_000_000; i++) {
            map.put(i, i);
        }
        
        // Java 8: Multiple threads help with resizing
        // Java 7: Single thread does all resizing (bottleneck)
    }
}
```

**Key Takeaways:**

| Aspect | Java 7 | Java 8+ | Improvement |
|--------|---------|---------|-------------|
| Locking | 16 segments | Per-bin | ~10x concurrency |
| Collision handling | Linked list | RB-tree when >8 | O(n) → O(log n) |
| Empty bin insert | Lock segment | CAS | Lock-free |
| Resizing | Single thread | Cooperative | Parallel |
| Memory | Segments overhead | Lazy init | Less memory |
| API | Basic | Compute/merge | Atomic operations |
| Bulk ops | External iteration | Parallel streams | Built-in parallelism |

**When to Use:**
- **High concurrency**: Always prefer ConcurrentHashMap
- **Read-heavy**: ConcurrentHashMap (lock-free reads)
- **Write-heavy**: Still ConcurrentHashMap in Java 8+
- **Small maps**: Regular HashMap with external sync might be OK
- **Compute patterns**: ConcurrentHashMap.compute*() methods

---

## Thread Pools and Executors

### Q: Explain the Executor framework and different types of thread pools

**Context & Problem:**
Direct thread management creates numerous problems:
- **Thread lifecycle overhead**: Creating threads is expensive (~1ms each)
- **Resource exhaustion**: Unlimited threads cause OutOfMemoryError
- **Poor resource management**: Idle threads waste memory
- **No throttling**: Can overwhelm system resources
- **Complex shutdown**: Difficult to track and stop all threads

**Deep Dive Answer:**

The Executor framework (Java 5) revolutionized concurrent programming by decoupling:
1. **Task submission** (what to run)
2. **Task execution** (how/when/where to run)
3. **Thread lifecycle management** (creation, pooling, termination)

```java
public class ExecutorFrameworkDeepDive {
    
    // 1. FixedThreadPool - Predictable resource usage
    public static void fixedThreadPoolAnalysis() {
        int poolSize = Runtime.getRuntime().availableProcessors();
        ExecutorService executor = Executors.newFixedThreadPool(poolSize);
        
        // Internal implementation
        // new ThreadPoolExecutor(
        //     poolSize,                    // core pool size
        //     poolSize,                    // maximum pool size (same as core)
        //     0L, TimeUnit.MILLISECONDS,  // keep-alive time (not used)
        //     new LinkedBlockingQueue<>()  // unbounded queue!
        // );
        
        // STRENGTHS:
        // - Predictable resource usage
        // - Good for CPU-bound tasks
        // - No thread creation overhead after pool fills
        
        // WEAKNESSES:
        // - Unbounded queue can cause memory issues
        // - Can't grow beyond initial size
        // - Tasks wait in queue if all threads busy
        
        // Real-world example: Batch processing
        class BatchProcessor {
            private final ExecutorService executor = 
                Executors.newFixedThreadPool(4);
            
            public void processBatch(List<Record> records) {
                List<Future<Result>> futures = new ArrayList<>();
                
                for (Record record : records) {
                    Future<Result> future = executor.submit(() -> {
                        // CPU-intensive processing
                        return processRecord(record);
                    });
                    futures.add(future);
                }
                
                // Wait for all to complete
                for (Future<Result> future : futures) {
                    try {
                        Result result = future.get();
                        saveResult(result);
                    } catch (Exception e) {
                        handleError(e);
                    }
                }
            }
            
            private Result processRecord(Record r) { return new Result(); }
            private void saveResult(Result r) {}
            private void handleError(Exception e) {}
        }
        
        class Record {}
        class Result {}
    }
    
    // 2. CachedThreadPool - Dynamic sizing for I/O tasks
    public static void cachedThreadPoolAnalysis() {
        ExecutorService executor = Executors.newCachedThreadPool();
        
        // Internal implementation
        // new ThreadPoolExecutor(
        //     0,                           // core pool size (no permanent threads)
        //     Integer.MAX_VALUE,           // maximum pool size (unlimited!)
        //     60L, TimeUnit.SECONDS,       // idle threads terminated after 60s
        //     new SynchronousQueue<>()     // direct handoff, no queuing
        // );
        
        // STRENGTHS:
        // - Scales automatically with load
        // - Reuses threads when possible
        // - Good for I/O-bound tasks
        // - No task queuing (immediate execution)
        
        // WEAKNESSES:
        // - Can create unlimited threads (dangerous!)
        // - Not suitable for CPU-bound tasks
        // - Can overwhelm system resources
        
        // Real-world example: Web server handling requests
        class WebServer {
            private final ExecutorService executor = 
                Executors.newCachedThreadPool();
            
            public void handleRequest(Socket socket) {
                executor.execute(() -> {
                    try {
                        // I/O-bound operation
                        BufferedReader reader = new BufferedReader(
                            new InputStreamReader(socket.getInputStream()));
                        
                        String request = reader.readLine();
                        
                        // Thread might block on database call
                        String response = queryDatabase(request);
                        
                        // Thread might block on network I/O
                        PrintWriter writer = new PrintWriter(
                            socket.getOutputStream());
                        writer.println(response);
                        writer.flush();
                        
                        socket.close();
                    } catch (IOException e) {
                        // Handle error
                    }
                    // Thread returns to pool or terminates after 60s idle
                });
            }
            
            private String queryDatabase(String query) {
                // Simulated database call
                return "response";
            }
        }
        
        // Protection against resource exhaustion
        class BoundedCachedThreadPool {
            public ExecutorService createBounded(int maxThreads) {
                return new ThreadPoolExecutor(
                    0,                          // core pool size
                    maxThreads,                 // maximum pool size (bounded)
                    60L, TimeUnit.SECONDS,      // keep-alive time
                    new SynchronousQueue<>(),   // direct handoff
                    new ThreadPoolExecutor.CallerRunsPolicy() // Fallback
                );
            }
        }
    }
    
    // 3. SingleThreadExecutor - Sequential execution guarantee
    public static void singleThreadExecutorAnalysis() {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        
        // Internal implementation (wrapped for immutability)
        // new FinalizableDelegatedExecutorService(
        //     new ThreadPoolExecutor(
        //         1,                           // core pool size
        //         1,                           // maximum pool size
        //         0L, TimeUnit.MILLISECONDS,  // keep-alive time
        //         new LinkedBlockingQueue<>()  // unbounded queue
        //     )
        // );
        
        // STRENGTHS:
        // - Guarantees sequential execution
        // - Thread replacement on failure
        // - Memory efficiency (single thread)
        // - Implicit happens-before between tasks
        
        // WEAKNESSES:
        // - No parallelism
        // - One slow task blocks all others
        // - Unbounded queue risk
        
        // Real-world example: Event processing
        class EventProcessor {
            private final ExecutorService executor = 
                Executors.newSingleThreadExecutor(r -> {
                    Thread t = new Thread(r, "EventProcessor");
                    t.setDaemon(true);
                    t.setUncaughtExceptionHandler((thread, ex) -> {
                        System.err.println("Event processing failed: " + ex);
                        // Could restart or alert
                    });
                    return t;
                });
            
            private final List<EventListener> listeners = new ArrayList<>();
            
            public void publishEvent(Event event) {
                executor.execute(() -> {
                    // Sequential processing ensures event ordering
                    for (EventListener listener : listeners) {
                        try {
                            listener.onEvent(event);
                        } catch (Exception e) {
                            // One listener failure doesn't affect others
                            handleListenerError(listener, event, e);
                        }
                    }
                });
            }
            
            public void shutdown() {
                executor.shutdown();
                try {
                    if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                        List<Runnable> pending = executor.shutdownNow();
                        System.err.println("Dropped events: " + pending.size());
                    }
                } catch (InterruptedException e) {
                    executor.shutdownNow();
                    Thread.currentThread().interrupt();
                }
            }
            
            private void handleListenerError(EventListener l, Event e, Exception ex) {}
        }
        
        interface EventListener {
            void onEvent(Event event);
        }
        class Event {}
    }
    
    // 4. ScheduledThreadPool - Time-based task execution
    public static void scheduledThreadPoolAnalysis() {
        int corePoolSize = 2;
        ScheduledExecutorService scheduler = 
            Executors.newScheduledThreadPool(corePoolSize);
        
        // Internal: ScheduledThreadPoolExecutor with DelayedWorkQueue
        // Tasks sorted by execution time
        // Leader-follower pattern for efficiency
        
        // Different scheduling strategies
        class SchedulingStrategies {
            private final ScheduledExecutorService scheduler = 
                Executors.newScheduledThreadPool(3);
            
            public void demonstrateSchedulingModes() {
                // 1. One-time delay
                ScheduledFuture<?> future = scheduler.schedule(
                    this::oneTimeTask, 
                    5, TimeUnit.SECONDS
                );
                
                // Can cancel if needed
                if (someCondition()) {
                    future.cancel(false); // Don't interrupt if running
                }
                
                // 2. Fixed rate - starts every period regardless
                // Good for: Regular sampling, metrics collection
                scheduler.scheduleAtFixedRate(
                    this::fixedRateTask,
                    0,     // Initial delay
                    1000,  // Period
                    TimeUnit.MILLISECONDS
                );
                // If task takes 600ms: Starts at 0, 1000, 2000, 3000...
                // Can cause task queue buildup if task is slow!
                
                // 3. Fixed delay - waits between completions
                // Good for: Polling, cleanup tasks
                scheduler.scheduleWithFixedDelay(
                    this::fixedDelayTask,
                    0,     // Initial delay
                    1000,  // Delay after completion
                    TimeUnit.MILLISECONDS
                );
                // If task takes 600ms: Starts at 0, 1600, 3200, 4800...
                // Prevents queue buildup
            }
            
            // Real-world example: Health monitoring
            public void healthMonitoring() {
                AtomicInteger failureCount = new AtomicInteger(0);
                
                ScheduledFuture<?> healthCheck = scheduler.scheduleWithFixedDelay(() -> {
                    try {
                        boolean healthy = checkSystemHealth();
                        
                        if (!healthy) {
                            int failures = failureCount.incrementAndGet();
                            if (failures >= 3) {
                                alertOps("System unhealthy for " + failures + " checks");
                            }
                        } else {
                            failureCount.set(0);
                        }
                    } catch (Exception e) {
                        // Important: Exceptions kill scheduled tasks!
                        System.err.println("Health check failed: " + e);
                        // Task won't run again unless we handle this
                    }
                }, 0, 30, TimeUnit.SECONDS);
                
                // Graceful shutdown
                Runtime.getRuntime().addShutdownHook(new Thread(() -> {
                    healthCheck.cancel(false);
                    scheduler.shutdown();
                }));
            }
            
            // Cron-like scheduling
            public void scheduleDaily(Runnable task, int hour, int minute) {
                long initialDelay = computeDelayToTime(hour, minute);
                
                scheduler.scheduleAtFixedRate(
                    task,
                    initialDelay,
                    TimeUnit.DAYS.toMillis(1),
                    TimeUnit.MILLISECONDS
                );
            }
            
            private void oneTimeTask() {}
            private void fixedRateTask() {}
            private void fixedDelayTask() {}
            private boolean someCondition() { return false; }
            private boolean checkSystemHealth() { return true; }
            private void alertOps(String message) {}
            private long computeDelayToTime(int hour, int minute) { return 0; }
        }
    }
    
    // 5. WorkStealingPool - Modern parallel execution
    public static void workStealingPoolAnalysis() {
        // Uses ForkJoinPool internally
        ExecutorService executor = Executors.newWorkStealingPool();
        // Parallelism = Runtime.availableProcessors()
        
        // How work stealing works:
        // 1. Each thread has its own deque (double-ended queue)
        // 2. Thread pushes/pops from one end (LIFO)
        // 3. Other threads steal from opposite end (FIFO)
        // 4. Reduces contention and improves cache locality
        
        class WorkStealingDemo {
            private final ForkJoinPool pool = ForkJoinPool.commonPool();
            
            // Recursive task that benefits from work stealing
            class MergeSort extends RecursiveAction {
                private final int[] array;
                private final int start, end;
                private static final int THRESHOLD = 1000;
                
                MergeSort(int[] array, int start, int end) {
                    this.array = array;
                    this.start = start;
                    this.end = end;
                }
                
                @Override
                protected void compute() {
                    if (end - start <= THRESHOLD) {
                        // Small enough: sort directly
                        Arrays.sort(array, start, end);
                    } else {
                        // Divide and conquer
                        int mid = (start + end) / 2;
                        
                        MergeSort left = new MergeSort(array, start, mid);
                        MergeSort right = new MergeSort(array, mid, end);
                        
                        // Fork left, compute right, join left
                        left.fork();
                        right.compute();
                        left.join();
                        
                        // Merge results
                        merge(array, start, mid, end);
                    }
                }
                
                private void merge(int[] arr, int start, int mid, int end) {
                    // Merge implementation
                }
            }
            
            public void demonstrateWorkStealing() {
                int[] data = new int[10_000_000];
                // Initialize with random data
                
                long startTime = System.nanoTime();
                
                MergeSort task = new MergeSort(data, 0, data.length);
                pool.invoke(task);
                
                long elapsed = System.nanoTime() - startTime;
                
                // Work stealing statistics
                System.out.println("Pool parallelism: " + pool.getParallelism());
                System.out.println("Steal count: " + pool.getStealCount());
                System.out.println("Queue size: " + pool.getQueuedSubmissionCount());
                
                // Typical: 3-4x speedup on 4-core machine
            }
        }
    }
}
```

**Executor Selection Guide:**

```java
public class ExecutorSelectionGuide {
    
    public ExecutorService selectExecutor(TaskType taskType, 
                                         int expectedLoad,
                                         boolean needsOrdering) {
        
        switch (taskType) {
            case CPU_BOUND:
                // Fixed pool sized to CPU cores
                return Executors.newFixedThreadPool(
                    Runtime.getRuntime().availableProcessors() + 1
                );
                
            case IO_BOUND:
                // Cached pool or larger fixed pool
                if (expectedLoad < 100) {
                    return Executors.newCachedThreadPool();
                } else {
                    // Bounded to prevent resource exhaustion
                    return new ThreadPoolExecutor(
                        10, 200,
                        60L, TimeUnit.SECONDS,
                        new LinkedBlockingQueue<>(1000)
                    );
                }
                
            case SCHEDULED:
                // Scheduled pool for periodic tasks
                return Executors.newScheduledThreadPool(2);
                
            case RECURSIVE:
                // Work-stealing for divide-and-conquer
                return Executors.newWorkStealingPool();
                
            case SEQUENTIAL:
                // Single thread for ordering guarantee
                return Executors.newSingleThreadExecutor();
                
            default:
                // Safe default
                return Executors.newFixedThreadPool(10);
        }
    }
    
    enum TaskType {
        CPU_BOUND, IO_BOUND, SCHEDULED, RECURSIVE, SEQUENTIAL
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

**Context & Problem:**
Deadlocks are one of the most serious concurrency bugs:
- **System hangs**: Application becomes unresponsive
- **Resource starvation**: Critical resources become permanently unavailable
- **Difficult to detect**: May occur only under specific timing conditions
- **Hard to reproduce**: Often happens in production but not testing
- **No automatic recovery**: Requires restart or manual intervention

**Deep Dive Answer:**

**Deadlock Definition:**
A deadlock is a situation where two or more threads are blocked forever, each waiting for resources held by the others.

**Coffman Conditions (All four must be present):**

1. **Mutual Exclusion**
   - Resources cannot be shared simultaneously
   - Example: Database row locks, file handles
   - Why necessary: Some resources inherently non-shareable

2. **Hold and Wait**
   - Thread holds resources while waiting for additional resources
   - Example: Holding lock A while trying to acquire lock B
   - Why it happens: Complex operations need multiple resources

3. **No Preemption**
   - Resources cannot be forcibly taken from threads
   - Example: Can't force thread to release lock
   - Why necessary: Would break consistency guarantees

4. **Circular Wait**
   - Circular chain of threads each waiting for resource held by next
   - Example: T1→R1→T2→R2→T1 (cycle)
   - Why it happens: Inconsistent lock ordering

```java
public class DeadlockScenarios {
    
    // Classic Deadlock Example
    public class ClassicDeadlock {
        private final Object lock1 = new Object();
        private final Object lock2 = new Object();
        
        // Thread 1 execution path
        public void method1() {
            synchronized (lock1) {  // 1. T1 acquires lock1
                System.out.println(Thread.currentThread().getName() + 
                    ": Holding lock1");
                
                try { 
                    Thread.sleep(100); // 2. Simulates work
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                
                System.out.println(Thread.currentThread().getName() + 
                    ": Waiting for lock2");
                    
                synchronized (lock2) { // 3. T1 waits for lock2 (held by T2)
                    System.out.println("Never reached!");
                }
            }
        }
        
        // Thread 2 execution path
        public void method2() {
            synchronized (lock2) {  // 1. T2 acquires lock2
                System.out.println(Thread.currentThread().getName() + 
                    ": Holding lock2");
                
                System.out.println(Thread.currentThread().getName() + 
                    ": Waiting for lock1");
                    
                synchronized (lock1) { // 2. T2 waits for lock1 (held by T1)
                    System.out.println("Never reached!");
                }
            }
        }
        
        // Demonstration with deadlock detection
        public void demonstrateDeadlock() {
            Thread t1 = new Thread(this::method1, "Thread-1");
            Thread t2 = new Thread(this::method2, "Thread-2");
            
            t1.start();
            t2.start();
            
            // Detect deadlock
            try {
                Thread.sleep(1000);
                if (t1.isAlive() && t2.isAlive()) {
                    System.err.println("🛑 DEADLOCK DETECTED!");
                    
                    // Use ThreadMXBean for detailed diagnosis
                    ThreadMXBean bean = ManagementFactory.getThreadMXBean();
                    long[] deadlockedThreadIds = bean.findDeadlockedThreads();
                    
                    if (deadlockedThreadIds != null) {
                        ThreadInfo[] threadInfos = bean.getThreadInfo(deadlockedThreadIds);
                        for (ThreadInfo threadInfo : threadInfos) {
                            System.err.println("Deadlocked thread: " + 
                                threadInfo.getThreadName());
                            System.err.println("  Waiting on: " + 
                                threadInfo.getLockInfo());
                            System.err.println("  Owned locks: " + 
                                Arrays.toString(threadInfo.getLockedMonitors()));
                        }
                    }
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    // Real-world Deadlock: Bank Transfer
    public class BankTransferDeadlock {
        class Account {
            private double balance;
            private final int id;
            
            Account(int id, double balance) {
                this.id = id;
                this.balance = balance;
            }
            
            // DEADLOCK PRONE VERSION
            void transfer(Account from, Account to, double amount) {
                synchronized (from) {  // Lock source account
                    synchronized (to) {  // Lock target account
                        if (from.balance >= amount) {
                            from.balance -= amount;
                            to.balance += amount;
                        }
                    }
                }
            }
            // Problem: transfer(A,B) and transfer(B,A) cause deadlock!
        }
    }
    
    // Database Transaction Deadlock
    public class DatabaseDeadlock {
        // Common in database applications
        public void transaction1(Connection conn) throws SQLException {
            conn.setAutoCommit(false);
            
            // Update order: Table A then Table B
            Statement stmt = conn.createStatement();
            stmt.executeUpdate("UPDATE accounts SET balance = balance - 100 WHERE id = 1");
            Thread.sleep(100); // Simulate processing
            stmt.executeUpdate("UPDATE transactions SET status = 'completed' WHERE id = 1");
            
            conn.commit();
        }
        
        public void transaction2(Connection conn) throws SQLException {
            conn.setAutoCommit(false);
            
            // Update order: Table B then Table A (opposite!)
            Statement stmt = conn.createStatement();
            stmt.executeUpdate("UPDATE transactions SET status = 'pending' WHERE id = 2");
            Thread.sleep(100); // Simulate processing
            stmt.executeUpdate("UPDATE accounts SET balance = balance + 100 WHERE id = 2");
            
            conn.commit();
        }
        // Database detects and breaks deadlock by rolling back one transaction
        
        private void sleep(int ms) {
            try { Thread.sleep(ms); } catch (InterruptedException e) {}
        }
    }
}
```

**Comprehensive Prevention Strategies:**

```java
public class DeadlockPreventionStrategies {
    
    // STRATEGY 1: Lock Ordering (Most Common Solution)
    public class LockOrderingSolution {
        private final Object lock1 = new Object();
        private final Object lock2 = new Object();
        
        // Establish global ordering: lock1 always before lock2
        public void safeMethod1() {
            synchronized (lock1) {
                synchronized (lock2) {
                    // Work with both resources
                }
            }
        }
        
        public void safeMethod2() {
            synchronized (lock1) {  // Same order!
                synchronized (lock2) {
                    // Work with both resources
                }
            }
        }
        
        // Dynamic lock ordering using System.identityHashCode
        public class DynamicOrdering {
            void transfer(Account from, Account to, double amount) {
                // Order locks by identity hash code
                int fromHash = System.identityHashCode(from);
                int toHash = System.identityHashCode(to);
                
                if (fromHash < toHash) {
                    synchronized (from) {
                        synchronized (to) {
                            doTransfer(from, to, amount);
                        }
                    }
                } else if (fromHash > toHash) {
                    synchronized (to) {
                        synchronized (from) {
                            doTransfer(from, to, amount);
                        }
                    }
                } else {
                    // Hash collision - use tie-breaking lock
                    synchronized (tieBreakerLock) {
                        synchronized (from) {
                            synchronized (to) {
                                doTransfer(from, to, amount);
                            }
                        }
                    }
                }
            }
            
            private final Object tieBreakerLock = new Object();
            private void doTransfer(Account from, Account to, double amount) {
                // Transfer logic
            }
        }
        
        class Account {}
    }
    
    // STRATEGY 2: Lock Timeout (tryLock Pattern)
    public class TryLockSolution {
        private final ReentrantLock lock1 = new ReentrantLock();
        private final ReentrantLock lock2 = new ReentrantLock();
        
        public boolean tryLockWithTimeout() throws InterruptedException {
            long startTime = System.nanoTime();
            long timeout = TimeUnit.SECONDS.toNanos(5);
            
            while (System.nanoTime() - startTime < timeout) {
                // Try to acquire both locks
                if (lock1.tryLock()) {
                    try {
                        if (lock2.tryLock()) {
                            try {
                                // Got both locks - do work
                                performWork();
                                return true;
                            } finally {
                                lock2.unlock();
                            }
                        }
                    } finally {
                        lock1.unlock();
                    }
                }
                
                // Back off before retry
                Thread.sleep(50 + ThreadLocalRandom.current().nextInt(50));
            }
            
            // Timeout - possible deadlock avoided
            System.err.println("Failed to acquire locks - possible deadlock avoided");
            return false;
        }
        
        // Advanced: Acquire multiple locks atomically
        public boolean acquireMultipleLocks(List<ReentrantLock> locks, 
                                           long timeout, 
                                           TimeUnit unit) 
                throws InterruptedException {
            
            long startTime = System.nanoTime();
            long timeoutNanos = unit.toNanos(timeout);
            List<ReentrantLock> acquired = new ArrayList<>();
            
            try {
                for (ReentrantLock lock : locks) {
                    long remaining = timeoutNanos - (System.nanoTime() - startTime);
                    if (remaining <= 0 || !lock.tryLock(remaining, TimeUnit.NANOSECONDS)) {
                        // Failed to acquire - release all and return
                        return false;
                    }
                    acquired.add(lock);
                }
                
                // All locks acquired
                performWork();
                return true;
                
            } finally {
                // Release in reverse order
                Collections.reverse(acquired);
                acquired.forEach(ReentrantLock::unlock);
            }
        }
        
        private void performWork() {}
    }
    
    // STRATEGY 3: Deadlock Detection and Recovery
    public class DeadlockDetectionAndRecovery {
        private final ScheduledExecutorService detector = 
            Executors.newSingleThreadScheduledExecutor();
        
        public void startDeadlockMonitoring() {
            detector.scheduleAtFixedRate(() -> {
                ThreadMXBean bean = ManagementFactory.getThreadMXBean();
                long[] deadlockedThreadIds = bean.findDeadlockedThreads();
                
                if (deadlockedThreadIds != null) {
                    handleDeadlock(bean, deadlockedThreadIds);
                }
            }, 0, 5, TimeUnit.SECONDS);
        }
        
        private void handleDeadlock(ThreadMXBean bean, long[] threadIds) {
            ThreadInfo[] threadInfos = bean.getThreadInfo(threadIds, true, true);
            
            // Log detailed information
            StringBuilder report = new StringBuilder("DEADLOCK DETECTED!\n");
            
            for (ThreadInfo info : threadInfos) {
                report.append("\nThread: ").append(info.getThreadName())
                      .append(" (ID: ").append(info.getThreadId()).append(")\n")
                      .append("  State: ").append(info.getThreadState()).append("\n")
                      .append("  Waiting on: ").append(info.getLockInfo()).append("\n")
                      .append("  Lock owner: ").append(info.getLockOwnerName()).append("\n")
                      .append("  Stack trace:\n");
                
                for (StackTraceElement element : info.getStackTrace()) {
                    report.append("    ").append(element).append("\n");
                }
            }
            
            System.err.println(report);
            
            // Recovery strategies
            recoverFromDeadlock(threadInfos);
        }
        
        private void recoverFromDeadlock(ThreadInfo[] deadlockedThreads) {
            // Strategy 1: Interrupt one thread (victim selection)
            ThreadInfo victim = selectVictim(deadlockedThreads);
            if (victim != null) {
                // Find actual Thread object and interrupt
                Thread thread = findThread(victim.getThreadId());
                if (thread != null) {
                    System.err.println("Interrupting victim thread: " + 
                        victim.getThreadName());
                    thread.interrupt();
                }
            }
            
            // Strategy 2: Alert and restart
            alertOperations("Deadlock detected, manual intervention needed");
            
            // Strategy 3: Kill and restart (last resort)
            // System.exit(1); // Let supervisor restart
        }
        
        private ThreadInfo selectVictim(ThreadInfo[] threads) {
            // Select thread with lowest priority or least work done
            return threads.length > 0 ? threads[0] : null;
        }
        
        private Thread findThread(long threadId) {
            for (Thread thread : Thread.getAllStackTraces().keySet()) {
                if (thread.getId() == threadId) {
                    return thread;
                }
            }
            return null;
        }
        
        private void alertOperations(String message) {
            // Send alert to monitoring system
        }
    }
    
    // STRATEGY 4: Lock-Free Algorithms (Avoid Locks Entirely)
    public class LockFreeAlternatives {
        // Use atomic operations instead of locks
        private final AtomicReference<Node> head = new AtomicReference<>();
        
        class Node {
            final int value;
            final AtomicReference<Node> next = new AtomicReference<>();
            
            Node(int value) {
                this.value = value;
            }
        }
        
        // Lock-free stack push
        public void push(int value) {
            Node newNode = new Node(value);
            Node currentHead;
            
            do {
                currentHead = head.get();
                newNode.next.set(currentHead);
            } while (!head.compareAndSet(currentHead, newNode));
            // No locks = no deadlock possible!
        }
    }
    
    // STRATEGY 5: Resource Allocation Graph
    public class ResourceAllocationGraph {
        // Track resource ownership and requests
        private final Map<Thread, Set<Object>> owned = new ConcurrentHashMap<>();
        private final Map<Thread, Set<Object>> requested = new ConcurrentHashMap<>();
        
        public boolean wouldCauseDeadlock(Thread thread, Object resource) {
            // Check if granting resource would create cycle
            // Implementation of cycle detection algorithm
            return detectCycle(thread, resource);
        }
        
        private boolean detectCycle(Thread thread, Object resource) {
            // DFS to detect cycle in wait-for graph
            return false; // Simplified
        }
    }
}
```

**Best Practices Summary:**

```java
public class DeadlockBestPractices {
    // 1. Use higher-level concurrency utilities
    private final ConcurrentHashMap<String, Data> map = new ConcurrentHashMap<>();
    // Instead of: HashMap with synchronized blocks
    
    // 2. Immutable objects don't need locking
    private final ImmutableList<String> list = ImmutableList.of("a", "b");
    
    // 3. Use thread-safe classes
    private final AtomicInteger counter = new AtomicInteger();
    
    // 4. Minimize lock scope
    public void minimizedLocking() {
        Data data = prepareData(); // Outside lock
        
        synchronized (this) {
            // Only critical update inside lock
            updateSharedState(data);
        }
        
        notifyListeners(data); // Outside lock
    }
    
    // 5. Document locking policy
    /**
     * Lock ordering: always acquire locks in order:
     * 1. accountLock
     * 2. transactionLock  
     * 3. auditLock
     */
    public void documentedMethod() {
        // Implementation following documented order
    }
    
    private Data prepareData() { return new Data(); }
    private void updateSharedState(Data d) {}
    private void notifyListeners(Data d) {}
    
    class Data {}
    class ImmutableList<T> {
        static <T> ImmutableList<T> of(T... items) { return new ImmutableList<>(); }
    }
}
```

**Key Takeaways:**
- **Prevention > Detection > Recovery**: Prevent deadlocks by design
- **Lock ordering**: Most effective prevention strategy
- **Timeout**: Good fallback for unpredictable scenarios  
- **Lock-free**: Best performance but complex to implement
- **Testing**: Deadlocks often only appear under load
- **Monitoring**: Production monitoring essential for detection

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

**Context & Problem:**
Concurrency bugs are among the hardest to find and fix because they:
- **Are non-deterministic**: May work 99% of the time
- **Depend on timing**: Thread scheduling affects occurrence
- **Hard to reproduce**: Disappear when debugging
- **Platform-dependent**: Different behavior on different systems
- **Heisenbugs**: Change behavior when observed

**Deep Dive Answer:**

**1. Race Conditions (Most Common Bug):**

**What It Is:**
Occurs when program correctness depends on relative timing of events
```java
public class RaceConditionExamples {
    
    // Example 1: Check-Then-Act Race Condition
    public class CheckThenActBug {
        private Map<String, ExpensiveObject> cache = new HashMap<>();
        
        // BUGGY VERSION - Race condition
        public ExpensiveObject getInstance(String key) {
            ExpensiveObject obj = cache.get(key);      // Check
            if (obj == null) {
                // RACE WINDOW: Another thread can enter here
                obj = new ExpensiveObject(key);        // Create
                cache.put(key, obj);                    // Act
            }
            return obj;
            // Result: Multiple ExpensiveObject instances created!
        }
        
        // FIX 1: Synchronization
        public synchronized ExpensiveObject getInstanceSync(String key) {
            ExpensiveObject obj = cache.get(key);
            if (obj == null) {
                obj = new ExpensiveObject(key);
                cache.put(key, obj);
            }
            return obj;
        }
        
        // FIX 2: ConcurrentHashMap with atomic operation
        private final ConcurrentHashMap<String, ExpensiveObject> safeCache = 
            new ConcurrentHashMap<>();
        
        public ExpensiveObject getInstanceAtomic(String key) {
            return safeCache.computeIfAbsent(key, ExpensiveObject::new);
            // Guaranteed single creation even with concurrent calls
        }
    }
    
    // Example 2: Read-Modify-Write Race Condition
    public class CounterRaceCondition {
        private int count = 0;
        
        // BUGGY VERSION
        public void increment() {
            count++;  // Not atomic!
            // Actually three operations:
            // 1. Read count
            // 2. Add 1
            // 3. Write back
            // Threads can interleave between these
        }
        
        // What happens with 2 threads incrementing 1000 times each:
        // Expected: 2000
        // Actual: 1000-2000 (random due to lost updates)
        
        // FIX 1: Synchronization
        private int syncCount = 0;
        public synchronized void incrementSync() {
            syncCount++;
        }
        
        // FIX 2: AtomicInteger
        private final AtomicInteger atomicCount = new AtomicInteger();
        public void incrementAtomic() {
            atomicCount.incrementAndGet();
        }
        
        // FIX 3: Lock
        private final ReentrantLock lock = new ReentrantLock();
        private int lockCount = 0;
        public void incrementLock() {
            lock.lock();
            try {
                lockCount++;
            } finally {
                lock.unlock();
            }
        }
    }
    
    // Example 3: Collection Modification Race
    public class CollectionRaceCondition {
        private List<String> list = new ArrayList<>();
        
        // BUGGY VERSION
        public void addIfNotPresent(String item) {
            if (!list.contains(item)) {  // Check
                // RACE WINDOW
                list.add(item);           // Act
            }
            // Result: Duplicates possible
        }
        
        // BUGGY VERSION 2: Even iteration is unsafe
        public void iterateAndModify() {
            for (String item : list) {  // Iterator created
                if (item.startsWith("remove")) {
                    list.remove(item);   // ConcurrentModificationException!
                }
            }
        }
        
        // FIX: Thread-safe collection
        private final CopyOnWriteArrayList<String> safeList = 
            new CopyOnWriteArrayList<>();
    }
    
    class ExpensiveObject {
        ExpensiveObject(String key) {}
    }
}
```

**2. Memory Consistency Errors (Visibility Issues):**

**What It Is:**
Threads see inconsistent views of shared memory due to caching and reordering

```java
public class MemoryConsistencyBugs {
    
    // Example 1: Visibility Problem
    public class VisibilityBug {
        private boolean stopRequested = false;
        private int counter = 0;
        
        // BUGGY VERSION
        public void backgroundTask() {
            while (!stopRequested) {  // May never see update!
                counter++;
                // Compiler optimization might turn this into:
                // if (!stopRequested) while(true) counter++;
            }
            System.out.println("Stopped at: " + counter);
        }
        
        public void requestStop() {
            stopRequested = true;  // Other thread might never see this
        }
        
        // FIX 1: Volatile
        private volatile boolean volatileStop = false;
        
        // FIX 2: Synchronization
        private boolean syncStop = false;
        public synchronized boolean shouldStop() {
            return syncStop;
        }
        public synchronized void requestSyncStop() {
            syncStop = true;
        }
        
        // FIX 3: AtomicBoolean
        private final AtomicBoolean atomicStop = new AtomicBoolean(false);
    }
    
    // Example 2: Reordering Problem
    public class ReorderingBug {
        private int x = 0, y = 0;
        private int a = 0, b = 0;
        
        // BUGGY VERSION - Surprising results possible
        public void thread1() {
            a = 1;  // Can be reordered
            x = b;  // Can happen before a = 1
        }
        
        public void thread2() {
            b = 1;  // Can be reordered  
            y = a;  // Can happen before b = 1
        }
        
        // Possible outcomes:
        // (x=0, y=1): Thread 1 runs completely first
        // (x=1, y=0): Thread 2 runs completely first
        // (x=1, y=1): Threads interleave normally
        // (x=0, y=0): SURPRISING! Both reads happen before writes
        //             due to reordering
        
        // FIX: Use volatile or synchronization
        private volatile int vol_x = 0, vol_y = 0;
        private volatile int vol_a = 0, vol_b = 0;
    }
    
    // Example 3: Publication Problem
    public class UnsafePublication {
        private Holder holder;
        
        public void initialize() {
            holder = new Holder(42);  // Not safely published!
        }
        
        class Holder {
            private int value;
            
            public Holder(int value) {
                this.value = value;
            }
            
            public void assertValue() {
                if (value != 42) {
                    throw new AssertionError("Saw partially constructed object!");
                    // This can actually happen!
                }
            }
        }
        
        // FIX: Safe publication
        private volatile Holder safeHolder;  // Volatile ensures safe publication
        private final Holder finalHolder = new Holder(42);  // Final is safe
    }
}
```

**3. Livelock (Threads Alive but Not Progressing):**

**What It Is:**
Threads actively executing but making no progress due to repeated state changes

```java
public class LivelockExamples {
    
    // Example 1: Polite Threads Problem
    public class PoliteLivelock {
        class Person {
            private boolean moving = false;
            private final String name;
            
            Person(String name) {
                this.name = name;
            }
            
            // BUGGY VERSION - Livelock
            public void passCorridor(Person other) {
                while (other.moving) {
                    System.out.println(name + ": You go first");
                    moving = false;
                    Thread.yield();  // Let other thread run
                    moving = true;
                }
                // If both threads execute this, infinite loop!
            }
            
            // FIX 1: Add randomization
            public void passCorridorRandom(Person other) 
                    throws InterruptedException {
                while (other.moving) {
                    moving = false;
                    // Random backoff breaks symmetry
                    Thread.sleep(ThreadLocalRandom.current().nextInt(100));
                    moving = true;
                }
            }
            
            // FIX 2: Priority-based
            public void passCorridorPriority(Person other) {
                // Use consistent ordering (like ID or hash)
                if (name.compareTo(other.name) < 0) {
                    // Lower name has priority
                    while (other.moving) {
                        Thread.yield();
                    }
                } else {
                    moving = false;  // Higher name yields
                }
            }
        }
    }
    
    // Example 2: Resource Allocation Livelock
    public class ResourceLivelock {
        private final Lock lock1 = new ReentrantLock();
        private final Lock lock2 = new ReentrantLock();
        
        // BUGGY VERSION - Can livelock
        public void task1() {
            while (true) {
                if (lock1.tryLock()) {
                    try {
                        if (lock2.tryLock()) {
                            try {
                                // Do work
                                break;
                            } finally {
                                lock2.unlock();
                            }
                        }
                    } finally {
                        lock1.unlock();
                    }
                }
                // Immediately retry - can cause livelock
            }
        }
        
        // FIX: Add backoff
        public void task1Fixed() throws InterruptedException {
            Random random = new Random();
            
            while (true) {
                if (lock1.tryLock()) {
                    try {
                        if (lock2.tryLock()) {
                            try {
                                // Do work
                                break;
                            } finally {
                                lock2.unlock();
                            }
                        }
                    } finally {
                        lock1.unlock();
                    }
                }
                
                // Exponential backoff
                Thread.sleep(random.nextInt(100) * (1 << attempts));
                attempts = Math.min(attempts + 1, 5);
            }
        }
        
        private int attempts = 0;
    }
}
```

**4. Thread Leaks (Resource Exhaustion):**

**What It Is:**
Threads created but never terminated, consuming system resources

```java
public class ThreadLeakExamples {
    
    // Example 1: Unbounded Thread Creation
    public class UnboundedThreadLeak {
        // BUGGY VERSION - Creates unlimited threads
        public void handleRequest(Request request) {
            new Thread(() -> {
                processRequest(request);
            }).start();
            // Problem: Each request creates new thread
            // 10,000 requests = 10,000 threads = ~10GB memory!
        }
        
        // FIX: Use thread pool
        private final ExecutorService executor = 
            Executors.newFixedThreadPool(100);
        
        public void handleRequestFixed(Request request) {
            executor.submit(() -> processRequest(request));
        }
        
        private void processRequest(Request r) {}
    }
    
    // Example 2: Threads Not Terminating
    public class NonTerminatingThreads {
        private volatile boolean running = true;
        private final List<Thread> workers = new ArrayList<>();
        
        // BUGGY VERSION - Threads never stop
        public void startWorkers() {
            for (int i = 0; i < 10; i++) {
                Thread worker = new Thread(() -> {
                    while (true) {  // No termination condition!
                        try {
                            doWork();
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            // Swallowing interrupt - BAD!
                        }
                    }
                });
                worker.start();
                workers.add(worker);
            }
        }
        
        // FIX: Proper termination
        public void startWorkersFixed() {
            for (int i = 0; i < 10; i++) {
                Thread worker = new Thread(() -> {
                    while (running && !Thread.currentThread().isInterrupted()) {
                        try {
                            doWork();
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();  // Restore flag
                            break;  // Exit loop
                        }
                    }
                    System.out.println("Worker terminated gracefully");
                });
                worker.start();
                workers.add(worker);
            }
        }
        
        public void shutdown() {
            running = false;
            workers.forEach(Thread::interrupt);
            
            // Wait for termination
            for (Thread worker : workers) {
                try {
                    worker.join(5000);
                    if (worker.isAlive()) {
                        System.err.println("Worker didn't terminate: " + 
                            worker.getName());
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        }
        
        private void doWork() {}
    }
    
    // Example 3: Timer Thread Leak
    public class TimerLeak {
        // BUGGY VERSION - Timer threads not cleaned up
        public void scheduleTasks() {
            Timer timer = new Timer();  // Creates background thread
            timer.schedule(new TimerTask() {
                @Override
                public void run() {
                    System.out.println("Task executed");
                }
            }, 1000, 1000);
            // Timer thread continues running even after method exits!
        }
        
        // FIX: Proper cleanup
        private Timer timer;
        
        public void scheduleTasksFixed() {
            timer = new Timer(true);  // Daemon thread
            timer.schedule(new TimerTask() {
                @Override
                public void run() {
                    System.out.println("Task executed");
                }
            }, 1000, 1000);
        }
        
        public void cleanup() {
            if (timer != null) {
                timer.cancel();  // Stop timer thread
            }
        }
    }
    
    class Request {}
}
```

**5. Starvation (Thread Never Gets Resources):**

```java
public class StarvationExamples {
    
    // Example: Priority-based starvation
    public class PriorityStarvation {
        private final ReentrantLock lock = new ReentrantLock(true); // Fair lock
        
        public void highPriorityTask() {
            Thread.currentThread().setPriority(Thread.MAX_PRIORITY);
            
            while (true) {
                lock.lock();
                try {
                    // Constantly acquires lock
                    performWork();
                } finally {
                    lock.unlock();
                }
                // Immediately reacquires - starves low priority threads
            }
        }
        
        public void lowPriorityTask() {
            Thread.currentThread().setPriority(Thread.MIN_PRIORITY);
            
            lock.lock();  // May never acquire!
            try {
                performWork();
            } finally {
                lock.unlock();
            }
        }
        
        // FIX: Use fair locking or avoid priorities
        private void performWork() {}
    }
}
```

**Prevention Strategies Summary:**

```java
public class ConcurrencyBugPrevention {
    // 1. Use immutable objects
    private final ImmutableData data = new ImmutableData("value");
    
    // 2. Use thread-safe collections
    private final ConcurrentHashMap<String, String> map = new ConcurrentHashMap<>();
    
    // 3. Use atomic operations
    private final AtomicInteger counter = new AtomicInteger();
    
    // 4. Minimize shared state
    private final ThreadLocal<SimpleDateFormat> dateFormat = 
        ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
    
    // 5. Use higher-level abstractions
    private final CompletableFuture<String> future = new CompletableFuture<>();
    
    // 6. Always handle interruption
    public void interruptibleTask() throws InterruptedException {
        while (!Thread.currentThread().isInterrupted()) {
            // Work
        }
    }
    
    class ImmutableData {
        private final String value;
        ImmutableData(String value) { this.value = value; }
        public String getValue() { return value; }
    }
}
```

**Key Takeaways:**
- **Race conditions**: Use atomic operations or synchronization
- **Memory consistency**: Use volatile or happens-before guarantees
- **Livelock**: Add randomization or backoff
- **Thread leaks**: Always use thread pools and proper shutdown
- **Starvation**: Use fair locks and avoid priority manipulation
- **Testing**: Use tools like jcstress, ThreadSanitizer
- **Prevention**: Design for concurrency from the start

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