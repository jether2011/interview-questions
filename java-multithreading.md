# Java Multithreading & Concurrency

## Table of Contents
1. [Thread Lifecycle](#thread-lifecycle)
2. [synchronized & Locks](#synchronized--locks)
3. [Thread Contention](#thread-contention)
4. [CountDownLatch & CyclicBarrier](#countdownlatch--cyclicbarrier)
5. [Executor Framework](#executor-framework)
6. [CompletableFuture](#completablefuture)
7. [Handle 10,000 Requests in a Java Microservice](#handle-10000-requests-in-a-java-microservice)
8. [Race Conditions in Distributed Environments](#race-conditions-in-distributed-environments)
9. [Deadlocks](#deadlocks)
10. [Atomic & Volatile](#atomic--volatile)

---

## Thread Lifecycle

```
NEW → RUNNABLE → (BLOCKED | WAITING | TIMED_WAITING) → TERMINATED
```

- **BLOCKED** — waiting to acquire a `synchronized` lock
- **WAITING** — `wait()`, `join()`, `LockSupport.park()`
- **TIMED_WAITING** — `sleep(ms)`, `wait(ms)`, `tryLock(timeout)`

---

## synchronized & Locks

```java
// Method-level (locks on 'this')
public synchronized void increment() { count++; }

// Block-level (explicit lock object — better granularity)
private final Object lock = new Object();
public void increment() {
    synchronized (lock) { count++; }
}

// ReentrantLock — same semantics, more control
private final ReentrantLock lock = new ReentrantLock();
public void increment() {
    lock.lock();
    try { count++; }
    finally { lock.unlock(); }
}
```

**ReentrantLock advantages over `synchronized`:**
- `tryLock(timeout)` — avoid indefinite blocking
- `lockInterruptibly()` — cancel waiting thread
- `ReentrantReadWriteLock` — allow concurrent reads, exclusive writes

---

## Thread Contention

**Thread contention** occurs when multiple threads compete for the same resource (lock, I/O, CPU). It causes threads to **block**, increasing latency and reducing throughput.

**Symptoms:**
- High CPU with low throughput
- Thread dumps showing `BLOCKED` or `WAITING` on the same monitor
- Profiler shows lock contention hotspots

**Causes and fixes:**

| Cause | Fix |
|---|---|
| Coarse-grained `synchronized` | Use `ConcurrentHashMap`, `ReadWriteLock`, or segment locks |
| Long critical sections | Minimize work inside locks; move I/O outside |
| Hot singleton | Stripe the lock or use lock-free data structures |
| Connection pool exhaustion | Tune pool size; use async I/O |

```java
// Bad: single lock for all keys
synchronized Map<String, Value> map = new HashMap<>();

// Good: ConcurrentHashMap uses segment-level locking
ConcurrentHashMap<String, Value> map = new ConcurrentHashMap<>();

// Good: ReadWriteLock — reads don't block each other
ReadWriteLock rwLock = new ReentrantReadWriteLock();
rwLock.readLock().lock();   // multiple readers concurrently
rwLock.writeLock().lock();  // exclusive write
```

**Diagnosis:** use `jstack <pid>` or `VisualVM` to spot threads blocked on the same monitor address.

---

## CountDownLatch & CyclicBarrier

```java
// CountDownLatch: wait for N events to complete (one-time use)
CountDownLatch latch = new CountDownLatch(3);
executorService.submit(() -> { doWork(); latch.countDown(); });
latch.await(); // blocks until count = 0

// CyclicBarrier: N threads wait for each other at a checkpoint (reusable)
CyclicBarrier barrier = new CyclicBarrier(3, () -> System.out.println("All ready"));
executorService.submit(() -> { prepare(); barrier.await(); proceed(); });
```

---

## Executor Framework

```java
// CPU-bound: fixed pool = number of cores
ExecutorService cpu = Executors.newFixedThreadPool(
    Runtime.getRuntime().availableProcessors()
);

// I/O-bound: cached pool or virtual threads (Java 21)
ExecutorService io = Executors.newVirtualThreadPerTaskExecutor(); // Java 21

// Scheduled tasks
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
scheduler.scheduleAtFixedRate(task, 0, 10, TimeUnit.SECONDS);
```

Always shut down executors: `executor.shutdown()` → `awaitTermination()` → `shutdownNow()` if needed.

---

## CompletableFuture

Composable async pipeline without blocking threads.

```java
CompletableFuture
    .supplyAsync(() -> fetchUser(id))          // run on ForkJoinPool
    .thenApplyAsync(user -> fetchOrders(user)) // chain non-blocking
    .thenCombine(fetchPromotions(), (orders, promos) -> merge(orders, promos))
    .exceptionally(ex -> fallback())
    .thenAccept(result -> respond(result));
```

**Key methods:**
- `thenApply` / `thenApplyAsync` — transform result
- `thenCombine` — merge two futures
- `thenCompose` — flat-map (when the function returns another future)
- `allOf` — wait for all; `anyOf` — wait for first

---

## Handle 10,000 Requests in a Java Microservice

**The bottleneck is usually threads, not CPU.** Classic thread-per-request model doesn't scale to 10k concurrent requests.

### Strategy 1: Non-blocking I/O (Reactive)

```java
// Spring WebFlux — reactive stack, no thread blocking
@GetMapping("/orders/{id}")
public Mono<Order> getOrder(@PathVariable Long id) {
    return orderRepository.findById(id)  // non-blocking DB call
        .switchIfEmpty(Mono.error(new NotFoundException()));
}
```

A single thread handles thousands of requests via event loop (Netty). No thread sits idle waiting for I/O.

### Strategy 2: Virtual Threads (Java 21 — Project Loom)

```java
// application.properties — Spring Boot 3.2+
spring.threads.virtual.enabled=true

// Or explicit
try (ExecutorService exec = Executors.newVirtualThreadPerTaskExecutor()) {
    for (Request req : requests) {
        exec.submit(() -> handleRequest(req));
    }
}
```

Virtual threads are lightweight (KB vs MB for platform threads). JVM can run millions of them. They block without holding OS threads.

### Strategy 3: Tune the Thread Pool + Connection Pool

```yaml
# application.properties (Tomcat)
server.tomcat.threads.max=400
server.tomcat.accept-count=200
spring.datasource.hikari.maximum-pool-size=50
```

### Strategy 4: Async Processing

Move work off the request thread. Return `202 Accepted` immediately; process in background.

```java
@PostMapping("/orders")
public ResponseEntity<Void> createOrder(@RequestBody OrderRequest req) {
    queue.publish(req);                    // fast, non-blocking
    return ResponseEntity.accepted().build(); // 202 immediately
}
```

### Strategy 5: Horizontal Scaling + Load Balancer

10k requests distributed across 5 instances = 2k per instance. Much more manageable.

---

## Race Conditions in Distributed Environments

**Race condition:** two processes read-modify-write shared state concurrently, producing incorrect results.

### Problem Example

```
Thread A: reads balance = 100
Thread B: reads balance = 100
Thread A: writes balance = 80  (deducted 20)
Thread B: writes balance = 70  (deducted 30 — from stale 100!)
Expected: 50. Actual: 70.
```

### Solutions

**1. Optimistic Locking (database version column)**

```java
@Entity
public class Account {
    @Version
    private Long version;  // JPA checks version on update
    private BigDecimal balance;
}

// If version changed since read → throws OptimisticLockException → retry
```

**2. Pessimistic Locking**

```java
@Query("SELECT a FROM Account a WHERE a.id = :id")
@Lock(LockModeType.PESSIMISTIC_WRITE)
Account findByIdForUpdate(@Param("id") Long id);
// SELECT ... FOR UPDATE — DB row lock
```

**3. Distributed Lock (Redis)**

```java
// Redisson
RLock lock = redisson.getLock("account:" + accountId);
lock.lock(5, TimeUnit.SECONDS);
try {
    // critical section — only one node at a time
    accountService.debit(accountId, amount);
} finally {
    lock.unlock();
}
```

**4. Atomic Operations / Compare-And-Swap**

```java
// Redis atomic: DECRBY returns new value atomically
Long newBalance = redis.opsForValue().decrement("balance:" + id, amount);
if (newBalance < 0) {
    redis.opsForValue().increment("balance:" + id, amount); // rollback
    throw new InsufficientFundsException();
}
```

**5. Event-Driven with Idempotency Key**

Each operation carries a unique `idempotencyKey`. Duplicate requests return the same stored result without re-executing.

```java
if (idempotencyStore.exists(key)) return idempotencyStore.get(key);
Result result = processOperation();
idempotencyStore.save(key, result, TTL_1H);
return result;
```

**Choice guide:**
- Low contention: **optimistic locking**
- High contention or critical: **pessimistic locking** or **distributed lock**
- Simple counters: **atomic Redis commands**
- Financial/blockchain: **event sourcing + idempotency**

---

## Deadlocks

A deadlock occurs when two threads each hold a lock the other needs.

```java
// Thread A locks A then tries B
// Thread B locks B then tries A → deadlock

// Fix: consistent lock ordering
synchronized (Math.min(idA, idB) == idA ? resourceA : resourceB) {
    synchronized (Math.min(idA, idB) == idA ? resourceB : resourceA) {
        transfer();
    }
}

// Or: use tryLock with timeout
if (lock1.tryLock(100, MILLISECONDS)) {
    if (lock2.tryLock(100, MILLISECONDS)) { ... }
    else lock1.unlock();
}
```

**Prevention:** lock ordering, timeouts, lock-free data structures, reduce lock granularity.

---

## Atomic & Volatile

```java
// volatile: visibility guarantee (no caching in registers/CPU cache)
// Does NOT guarantee atomicity for compound operations (check-then-act)
private volatile boolean running = true;

// AtomicInteger: lock-free, CAS-based atomicity
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();           // atomic
counter.compareAndSet(expected, update); // CAS
```

**Rule:** `volatile` for flags/status; `AtomicXxx` for counters; `synchronized`/`Lock` for compound operations on multiple variables.
