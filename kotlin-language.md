# Kotlin Language

## Table of Contents
1. [Kotlin vs Java](#kotlin-vs-java)
2. [Type System & Null Safety](#type-system--null-safety)
3. [Classes & Objects](#classes--objects)
4. [Functions & Lambdas](#functions--lambdas)
5. [Scope Functions](#scope-functions)
6. [Collections & Sequences](#collections--sequences)
7. [Coroutines](#coroutines)
8. [Kotlin Flow](#kotlin-flow)
9. [Generics & Advanced Types](#generics--advanced-types)
10. [Java Interoperability](#java-interoperability)
11. [Testing in Kotlin](#testing-in-kotlin)
12. [Best Practices](#best-practices)

---

## Kotlin vs Java

Kotlin is a statically typed JVM language designed to be a more concise, safe, and expressive alternative to Java. It compiles to JVM bytecode, making it fully interoperable with Java — you can call Java libraries from Kotlin and vice versa with no overhead. Kotlin's key improvements over Java: null safety baked into the type system (eliminates NPEs at compile time), data classes that auto-generate boilerplate, extension functions that add behavior to existing classes without inheritance, and coroutines for lightweight concurrency. For backend development, Kotlin integrates seamlessly with Spring Boot and is the language of choice for Android development.

| Feature | Kotlin | Java |
|---|---|---|
| Null safety | Built-in (`?` types) | `Optional` / prone to NPE |
| Data classes | `data class User(val name: String)` | 50+ lines of boilerplate |
| Extension functions | Yes | No |
| Coroutines | Native | `CompletableFuture` / Threads |
| Smart casts | Automatic after `is` check | Manual cast |
| Default parameters | Yes | No (overloads needed) |
| Sealed classes | Yes | `sealed` in Java 17+ |
| Operator overloading | Yes | No |
| String templates | `"Hello $name"` | `String.format(...)` |
| Type inference | Strong | Limited |

Kotlin compiles to JVM bytecode → 100% Java interoperable. No runtime overhead.

---

## Type System & Null Safety

Kotlin's type system distinguishes nullable types (`String?`) from non-null types (`String`) at compile time. Attempting to dereference a nullable reference without a null check is a compile error — not a runtime crash. This eliminates an entire class of `NullPointerException` bugs that plague Java codebases. The three operators `?.` (safe call), `?:` (Elvis/default), and `!!` (non-null assertion — throws NPE if null) give you fine-grained control over null handling. The `!!` operator is a code smell in most contexts — if you find yourself using it, consider whether a design change (returning `Optional` or throwing a specific exception earlier) would be cleaner.

### Nullable Types

```kotlin
var name: String  = "Alice"   // non-null — compiler-enforced
var nick: String? = null      // nullable

// Safe call — returns null if nick is null
val len: Int? = nick?.length

// Elvis operator — provide default
val len: Int = nick?.length ?: 0

// Non-null assertion — throws NPE if null (avoid in production)
val len: Int = nick!!.length

// Smart cast after null check
if (nick != null) {
    println(nick.length)  // nick is String here, not String?
}

// let — execute block only if non-null
nick?.let { println("Nick is $it") }
```

### val vs var

```kotlin
val name = "Alice"         // immutable reference (like Java final)
var count = 0; count = 1   // mutable reference

// val ≠ immutable object — list contents can change
val list = mutableListOf(1, 2, 3)
list.add(4)  // OK — list reference unchanged

// Prefer val; use var only when reassignment is necessary
```

### Type Inference

```kotlin
val name = "John"        // String
val age = 25             // Int
val price = 19.99        // Double
val active = true        // Boolean

fun add(a: Int, b: Int) = a + b  // return type inferred: Int

// Explicit type needed for ambiguity
val number: Long = 42    // without ': Long', would be Int
```

### Nothing Type

```kotlin
// Nothing — function never returns normally (throws or infinite loop)
fun fail(msg: String): Nothing = throw IllegalStateException(msg)

// Useful in when exhaustive:
val x: String = when (status) {
    Status.OK -> "fine"
    Status.ERR -> fail("error")  // compiler knows this branch never returns
}
```

---

## Classes & Objects

Kotlin's class system reduces boilerplate dramatically while adding powerful features. **Data classes** auto-generate `equals`, `hashCode`, `toString`, and `copy` — replacing 50+ lines of Java boilerplate with one line. **Sealed classes** define closed type hierarchies where all subclasses are known at compile time, enabling exhaustive `when` expressions without a default branch (the compiler enforces completeness). **Object declarations** create thread-safe singletons with no boilerplate. **Companion objects** provide static-like behavior while still having access to class internals. Classes are `final` by default in Kotlin — you must explicitly mark them `open` to allow subclassing (a better default than Java's implicit extensibility).

### Data Classes

Compiler generates: `equals`, `hashCode`, `toString`, `copy`, `componentN` functions.

```kotlin
data class User(
    val id: Long,
    val name: String,
    val email: String
)

val u1 = User(1, "Alice", "alice@example.com")
val u2 = u1.copy(name = "Bob")      // copy with changes
val (id, name, email) = u1          // destructuring
```

### Sealed Classes

Restricted class hierarchy — all subclasses known at compile time. Enables exhaustive `when`.

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String, val cause: Throwable? = null) : Result<Nothing>()
    object Loading : Result<Nothing>()
}

fun handle(result: Result<User>) = when (result) {
    is Result.Success -> render(result.data)
    is Result.Error   -> showError(result.message)
    Result.Loading    -> showSpinner()
    // no else needed — sealed class is exhaustive
}
```

### Object & Companion Object

```kotlin
// Singleton (thread-safe by language spec)
object EventBus {
    private val listeners = mutableListOf<EventListener>()
    fun register(l: EventListener) { listeners.add(l) }
    fun post(event: Any) { listeners.forEach { it.onEvent(event) } }
}

// Companion object — static-like members
class User(val name: String) {
    companion object {
        const val MAX_NAME_LENGTH = 50
        fun create(name: String): User {
            require(name.length <= MAX_NAME_LENGTH)
            return User(name)
        }
    }
}
val user = User.create("Alice")
```

### Inheritance & Interfaces

```kotlin
// Classes are final by default — must be open to extend
open class Animal(val name: String) {
    open fun sound() = "..."
}

class Dog(name: String) : Animal(name) {
    override fun sound() = "Woof"
}

// Interface with default implementation
interface Describable {
    val description: String
    fun describe() = "I am: $description"  // default implementation
}

// Multiple interface implementation (no diamond problem — explicit override required on conflict)
interface A { fun foo() = "A" }
interface B { fun foo() = "B" }
class C : A, B {
    override fun foo() = super<A>.foo()  // must resolve conflict explicitly
}
```

### Enum Classes

```kotlin
enum class Status(val code: Int) {
    PENDING(0), ACTIVE(1), INACTIVE(2);

    fun isActive() = this == ACTIVE
}

val s = Status.ACTIVE
println(s.code)         // 1
println(s.isActive())   // true
println(Status.values()) // [PENDING, ACTIVE, INACTIVE]
println(Status.valueOf("PENDING")) // PENDING
```

### Delegation

```kotlin
// Class delegation — implement interface by delegating to another object
interface Logger { fun log(msg: String) }
class ConsoleLogger : Logger { override fun log(msg: String) = println(msg) }

class Service(logger: Logger) : Logger by logger {
    fun doWork() { log("Working...") } // delegates to injected logger
}

// Property delegation — lazy, observable, map
val config: Config by lazy { loadConfig() }   // initialized on first access

var name: String by Delegates.observable("initial") { _, old, new ->
    println("Changed from $old to $new")
}
```

---

## Functions & Lambdas

Kotlin treats functions as first-class citizens — they can be stored in variables, passed as arguments, and returned from other functions. **Extension functions** let you add methods to existing classes (even Java classes like `String` or `List`) without modifying them or using inheritance — enabling fluent, expressive APIs. **Default parameters** eliminate the need for multiple overloaded methods. **Named arguments** make call sites self-documenting. **Inline functions** eliminate the lambda object allocation overhead by copying the function body to the call site — critical for performance-sensitive higher-order functions. **Tail recursion** (`tailrec`) lets the compiler optimize recursive functions into iterative loops.

### Function Flavors

```kotlin
// Regular
fun greet(name: String): String = "Hello, $name"

// Default parameters
fun connect(host: String = "localhost", port: Int = 5432) { ... }
connect()                      // both defaults
connect(port = 8080)           // named parameter — skip host

// Extension function
fun String.wordCount(): Int = split(" ").size
"hello world".wordCount()      // 2

// Infix function
infix fun Int.times(str: String) = str.repeat(this)
val result = 3 times "ha"      // "hahaha"

// Vararg
fun sum(vararg nums: Int) = nums.sum()
sum(1, 2, 3, 4)
```

### Higher-Order Functions & Lambdas

```kotlin
// Higher-order: takes or returns a function
fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T> { ... }

val evens = listOf(1, 2, 3, 4).filter { it % 2 == 0 }

// Lambda syntax
val multiply: (Int, Int) -> Int = { a, b -> a * b }
val double: (Int) -> Int = { it * 2 }  // 'it' for single-param lambdas

// Function reference
val nums = listOf("1", "2", "3")
val ints = nums.map(String::toInt)
```

### Inline Functions

Inline copies the function body + lambda at call site — eliminates lambda object allocation. Useful for performance-sensitive higher-order functions.

```kotlin
inline fun <T> measure(block: () -> T): Pair<T, Long> {
    val start = System.currentTimeMillis()
    val result = block()
    return result to System.currentTimeMillis() - start
}

val (result, ms) = measure { heavyComputation() }
// No lambda object created — block is inlined
```

### Tail Recursion

```kotlin
// tailrec: compiler converts to iterative loop (no stack overflow)
tailrec fun factorial(n: Long, acc: Long = 1): Long =
    if (n <= 1) acc else factorial(n - 1, n * acc)
```

---

## Scope Functions

Scope functions (`let`, `run`, `apply`, `also`, `with`) are Kotlin's way of executing a block of code in the context of an object. They differ along two axes: *how you refer to the object inside the block* (`it` vs `this`) and *what they return* (the lambda result vs the receiver object). Getting them right makes code more readable and idiomatic; getting them wrong adds confusion. The practical rules: use `apply` to configure/build an object (returns the object); use `let` for null-safe operations and transformations (returns the result); use `also` for side effects that don't change the chain (logging, auditing); use `run` when you need both `this` as the receiver AND want to return a result.

| Function | Receiver | Returns | Use Case |
|---|---|---|---|
| `let` | `it` | Lambda result | Null-safe block, transform |
| `run` | `this` | Lambda result | Initialize + compute result |
| `apply` | `this` | Receiver | Configure object |
| `also` | `it` | Receiver | Side effects (logging) |
| `with` | `this` | Lambda result | Multiple operations on object |

```kotlin
// let — null-safe + transform
val upper = name?.let { it.trim().uppercase() } ?: "UNKNOWN"

// apply — builder style configuration
val request = HttpRequest().apply {
    method = "POST"
    url = "https://api.example.com"
    body = json
}

// also — side effects without breaking chain
val user = createUser(dto)
    .also { log.info("Created user: ${it.id}") }
    .also { auditService.record(it) }

// run — compute result using 'this'
val result = transaction.run {
    if (amount > balance) throw InsufficientFundsException()
    balance - amount
}

// with — multiple calls on same object
with(printer) {
    setFont("Arial")
    setSize(12)
    print(document)
}
```

---

## Collections & Sequences

Kotlin's standard library provides a rich collection of functional operators on top of Java's collection types. The key design choice: Kotlin separates **read-only views** (`List`, `Set`, `Map`) from **mutable collections** (`MutableList`, `MutableMap`). This makes intent explicit — a function returning `List<T>` signals that callers shouldn't mutate it. **Sequences** are the lazy counterpart to collections: each element passes through the entire pipeline before the next element begins, avoiding intermediate lists. Use sequences when chaining multiple operations on large collections (10k+ elements) to avoid creating multiple intermediate copies in memory.

### Immutable vs Mutable

```kotlin
// Immutable (read-only views)
val list = listOf(1, 2, 3)
val set  = setOf("a", "b")
val map  = mapOf("key" to "value")

// Mutable
val mList = mutableListOf(1, 2, 3); mList.add(4)
val mMap  = mutableMapOf("a" to 1); mMap["b"] = 2
```

### Collection Operations

```kotlin
val orders = listOf(Order(1, 100.0, "PAID"), Order(2, 50.0, "PENDING"))

// filter, map, flatMap
val paid = orders.filter { it.status == "PAID" }
val totals = orders.map { it.total }
val items = orders.flatMap { it.items }       // flatten nested lists

// groupBy, associate
val byStatus = orders.groupBy { it.status }  // Map<String, List<Order>>
val byId = orders.associateBy { it.id }      // Map<Int, Order>

// reduce, fold
val sum = totals.reduce { acc, v -> acc + v }
val sumWithDefault = totals.fold(0.0) { acc, v -> acc + v }

// find, first, any, all, none
val first = orders.first { it.total > 75.0 }
val exists = orders.any { it.status == "REFUNDED" }  // false

// sorted, sortedBy
val sorted = orders.sortedByDescending { it.total }

// partition — split into two lists
val (paid2, notPaid) = orders.partition { it.status == "PAID" }

// zip
val names = listOf("Alice", "Bob")
val scores = listOf(90, 85)
val paired = names.zip(scores)  // [(Alice, 90), (Bob, 85)]
```

### Sequences (Lazy Evaluation)

```kotlin
// Collections: eager — each operation creates a new list
// Sequences: lazy — operations chained, one pass through data

val result = (1..1_000_000)
    .asSequence()              // convert to lazy sequence
    .filter { it % 2 == 0 }   // not evaluated yet
    .map { it * it }           // not evaluated yet
    .take(10)                  // not evaluated yet
    .toList()                  // terminal op — evaluate everything in ONE pass

// Use sequences for:
// - Large or infinite collections
// - Multiple chained operations (avoids intermediate lists)
// - generateSequence for infinite sequences
val fibs = generateSequence(Pair(0L, 1L)) { (a, b) -> Pair(b, a + b) }
    .map { it.first }
    .take(10)
    .toList()  // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

---

## Coroutines

Coroutines are Kotlin's solution to asynchronous programming — lightweight cooperative threads managed by the Kotlin runtime rather than the OS. A `suspend` function can pause its execution (suspending the coroutine) and release the underlying thread to do other work, resuming later when the result is ready. This makes it possible to write sequential-looking code that is actually non-blocking. The key shift from thread-based concurrency: you think in terms of *coroutines* (thousands of them) sharing a small thread pool, rather than threads (expensive). **Structured concurrency** ensures no coroutine leaks: child coroutines are always linked to a parent scope, and cancellation propagates down the hierarchy.

### Why Coroutines?

- **Threads:** expensive (~1MB stack each), OS-managed, blocking I/O wastes resources
- **Coroutines:** lightweight (few KB), user-space, suspend without blocking threads
- 1 million coroutines on a few threads — not possible with threads

### Core Concepts

```kotlin
// suspend function — can pause without blocking thread
suspend fun fetchUser(id: Long): User = withContext(Dispatchers.IO) {
    userRepository.findById(id)  // IO runs on IO thread pool
}

// launch — fire and forget
val job: Job = scope.launch {
    val user = fetchUser(42)
    updateUI(user)  // runs on caller's context
}

// async — returns a Deferred (future-like)
val deferred: Deferred<User> = scope.async { fetchUser(42) }
val user = deferred.await()  // suspend until result ready

// Parallel execution
val userDeferred    = async { fetchUser(id) }
val ordersDeferred  = async { fetchOrders(id) }
val (user, orders) = Pair(userDeferred.await(), ordersDeferred.await())
// Both run concurrently — not sequentially
```

### Dispatchers

| Dispatcher | Thread Pool | Use For |
|---|---|---|
| `Dispatchers.Main` | Main thread (Android/UI) | UI updates |
| `Dispatchers.IO` | Shared IO pool (64 threads) | DB, network, file I/O |
| `Dispatchers.Default` | CPU core count threads | CPU-intensive, computation |
| `Dispatchers.Unconfined` | Caller's thread | Testing, rarely production |

```kotlin
// withContext — switch dispatcher without new coroutine
suspend fun getUser(id: Long): User = withContext(Dispatchers.IO) {
    db.query("SELECT * FROM users WHERE id = ?", id)
}
```

### Coroutine Scopes

```kotlin
// CoroutineScope — defines lifecycle of coroutines
class OrderService(private val repo: OrderRepository) {
    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.Default)

    fun processOrdersAsync() {
        scope.launch { /* runs in background */ }
    }

    fun shutdown() { scope.cancel() }  // cancels all child coroutines
}

// structured concurrency: parent waits for all children
coroutineScope {              // suspends until all children complete
    launch { task1() }
    launch { task2() }
}  // both tasks finished here
```

### Exception Handling

```kotlin
// launch: uncaught exception → CoroutineExceptionHandler (or crashes)
val handler = CoroutineExceptionHandler { _, e -> log.error("Error", e) }
scope.launch(handler) { risky() }

// async: exception stored in Deferred, thrown on .await()
val result = scope.async { risky() }
try { result.await() } catch (e: Exception) { handle(e) }

// SupervisorJob: child failure doesn't cancel siblings
val supervisor = SupervisorJob()
val scope = CoroutineScope(supervisor)
scope.launch { failImmediately() }  // other coroutines keep running
scope.launch { keepGoing() }

// try-catch in coroutine body (most common)
scope.launch {
    try { riskyOperation() }
    catch (e: IOException) { handleIO(e) }
    finally { cleanup() }  // always runs, even on cancellation
}
```

### Cancellation

```kotlin
val job = scope.launch {
    repeat(1000) { i ->
        ensureActive()           // throws CancellationException if cancelled
        delay(100)               // suspend points check for cancellation automatically
        println("Working $i")
    }
}

job.cancel()                    // cooperative cancellation
job.join()                      // wait for cancellation to complete

// withTimeout
withTimeout(5000) {             // throws TimeoutCancellationException after 5s
    fetchLargeData()
}
val result = withTimeoutOrNull(5000) { fetchLargeData() }  // returns null on timeout
```

---

## Kotlin Flow

Kotlin Flow is the coroutine-native solution for reactive streams — a type-safe, backpressure-aware, asynchronous sequence of values. **Cold flows** (`flow { }`) don't execute until a terminal operator (`collect`) is called — each collector gets its own independent execution. **Hot flows** (`StateFlow`, `SharedFlow`) are always active regardless of collectors, making them suitable for sharing state across multiple subscribers. The distinction matters: cold flow is like a function (execute on demand), hot flow is like a broadcast radio (always transmitting, tune in when ready). Flow integrates naturally with coroutines and supports all standard operators (`map`, `filter`, `flatMapMerge`, `catch`, `onEach`).

Flow is a cold, asynchronous stream of values — like a suspend version of Sequence.

### Cold Flow

```kotlin
// Cold: doesn't produce values until collected
fun orderStream(customerId: Long): Flow<Order> = flow {
    var page = 0
    while (true) {
        val orders = orderRepo.findPage(customerId, page++)
        if (orders.isEmpty()) break
        orders.forEach { emit(it) }  // emit one at a time
    }
}

// Collection (terminal operation — starts execution)
orderStream(42L)
    .filter { it.status == "PAID" }
    .map { it.total }
    .onEach { log.info("Processing $it") }
    .collect { total -> sum += total }

// flowOn — change dispatcher for upstream
flow { emit(db.query()) }
    .flowOn(Dispatchers.IO)       // db query runs on IO dispatcher
    .collect { processOnDefault(it) }
```

### Hot Flows (StateFlow, SharedFlow)

```kotlin
// StateFlow — always has a value, emits to all current collectors
class ViewModel {
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    fun loadData() {
        scope.launch {
            _uiState.value = UiState.Loading
            try {
                _uiState.value = UiState.Success(repo.load())
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message)
            }
        }
    }
}

// SharedFlow — events (no initial value, configurable replay)
val events = MutableSharedFlow<Event>(replay = 0)  // 0 = no replay for late subscribers
events.emit(Event.UserLoggedIn)
events.collect { event -> handleEvent(event) }
```

### Flow Operators

```kotlin
// Transform
flow.map { transform(it) }
flow.filter { condition(it) }
flow.flatMapConcat { innerFlow(it) }   // sequential
flow.flatMapMerge { innerFlow(it) }    // concurrent

// Error handling
flow.catch { e -> emit(defaultValue) }
flow.retry(3) { e -> e is IOException }

// Backpressure
flow.buffer(capacity = 64)      // buffer emissions
flow.conflate()                 // drop intermediate, keep latest (like Rx throttle)
flow.collectLatest { slow() }   // cancel previous collection on new emission

// Combine multiple flows
combine(flow1, flow2) { a, b -> a + b }.collect { ... }
```

---

## Generics & Advanced Types

Kotlin's generics system adds **declaration-site variance** — a significant improvement over Java's use-site variance (`? extends` / `? super`). In Kotlin, you declare variance at the class definition: `out T` means the class only *produces* T (covariant, like `List<T>`); `in T` means it only *consumes* T (contravariant, like `Comparator<T>`). The `reified` keyword (only in `inline` functions) allows accessing the actual type at runtime — eliminating the need for `Class<T>` parameters in many generics-based APIs. **Type aliases** improve readability without runtime overhead. **Value classes** wrap a primitive with a domain-specific type, catching errors at compile time with zero runtime cost.

### Generics: in/out (Variance)

```kotlin
// out (covariant) — can only produce T (return), like Java ? extends T
class Producer<out T>(private val value: T) {
    fun get(): T = value
}
val producer: Producer<Number> = Producer<Int>(42)  // OK — Int is a Number

// in (contravariant) — can only consume T (parameter), like Java ? super T
class Consumer<in T> {
    fun consume(value: T) { println(value) }
}
val consumer: Consumer<Int> = Consumer<Number>()   // OK — Number consumer handles Int

// Reified type parameters (inline functions)
inline fun <reified T> parseJson(json: String): T = objectMapper.readValue(json, T::class.java)
val user: User = parseJson<User>(jsonString)  // T::class available at runtime
```

### Type Aliases

```kotlin
typealias EventHandler = (Event) -> Unit
typealias UserId = Long
typealias OrderMap = Map<UserId, List<Order>>

fun registerHandler(handler: EventHandler) { ... }
```

### Destructuring

```kotlin
data class Point(val x: Int, val y: Int)
val (x, y) = Point(3, 4)

// In lambda
listOf(Pair("Alice", 90), Pair("Bob", 85))
    .forEach { (name, score) -> println("$name: $score") }

// Map entries
for ((key, value) in map) { ... }
```

### Value Classes (Inline Classes)

```kotlin
// Wraps a value with type safety — no runtime overhead (inlined by compiler)
@JvmInline value class UserId(val value: Long)
@JvmInline value class Email(val value: String) {
    init { require(value.contains("@")) }
}

fun getUser(id: UserId): User = ...

// Compile-time safety: can't pass a raw Long where UserId is expected
getUser(UserId(42))  // ✅
getUser(42)          // ❌ compile error
```

---

## Java Interoperability

One of Kotlin's strongest selling points is seamless Java interoperability — you can use any Java library, framework, or codebase from Kotlin with no wrappers or bridges. The interoperability goes both ways: Kotlin code is callable from Java, though some Kotlin features (companion objects, default parameters, top-level functions) require specific annotations (`@JvmStatic`, `@JvmOverloads`, `@JvmField`) to expose a Java-friendly API. The trickiest interop issue is **platform types**: Java doesn't have nullable types, so Kotlin treats Java references as `T!` (unknown nullability) — you must handle potential nulls defensively.

### Calling Java from Kotlin

```kotlin
// Java collections work transparently
val javaList: java.util.ArrayList<String> = ArrayList()
javaList.add("Kotlin")

// Kotlin views Java's T as platform type T! (nullable or non-null — unknown)
val javaString: String! = javaLib.getValue()  // ! = platform type — be careful
val safe: String = javaLib.getValue() ?: ""   // handle potential null
```

### Calling Kotlin from Java

```kotlin
// @JvmStatic — companion function callable as Java static
class Config {
    companion object {
        @JvmStatic fun create(): Config = Config()  // Java: Config.create()
        fun without(): Config = Config()             // Java: Config.Companion.without()
    }
}

// @JvmOverloads — generate Java overloads for default parameters
@JvmOverloads
fun connect(host: String = "localhost", port: Int = 5432) { }
// Generates: connect(), connect(host), connect(host, port) for Java callers

// @JvmField — expose Kotlin property as Java field (no getter/setter)
@JvmField val MAX_SIZE = 100

// Top-level functions → static methods in ClassName.kt → ClassNameKt class
// fun greet() in Greetings.kt → GreetingsKt.greet() in Java
@file:JvmName("Greetings")  // rename generated class
```

### Kotlin + Java Collections

```kotlin
// Kotlin's List<String> compiles to java.util.List<String>
// MutableList<String> is java.util.List with add/remove exposed

// Convert between
val javaList: java.util.List<String> = kotlinList.toMutableList()
val kotlinList: List<String> = javaList.toList()

// Java Streams in Kotlin (works, but Kotlin collections are more idiomatic)
val stream: java.util.stream.Stream<String> = list.stream()
val result = stream.filter { it.startsWith("A") }.collect(toList())
// Prefer: list.filter { it.startsWith("A") }
```

---

## Testing in Kotlin

Kotlin's testing story is excellent. **MockK** is the Kotlin-native mocking library — it understands Kotlin idioms like data classes, extension functions, and suspend functions that Mockito struggles with. The backtick test name syntax (`fun \`create order returns saved order\`()`) makes test names self-documenting English sentences. **`runTest`** from `kotlinx-coroutines-test` provides a controlled time environment for coroutine tests — `advanceUntilIdle()` fast-forwards time so you don't wait for real delays. **`coEvery`/`coVerify`** are the MockK equivalents of `every`/`verify` for suspend functions.

### JUnit 5 + Kotlin

```kotlin
class OrderServiceTest {
    private val repo = mockk<OrderRepository>()
    private val service = OrderService(repo)

    @Test
    fun `create order returns saved order`() {  // backtick names!
        val order = Order(customerId = 1L, total = 100.0)
        every { repo.save(any()) } returns order.copy(id = 42L)

        val result = service.createOrder(order)

        assertThat(result.id).isEqualTo(42L)
        verify { repo.save(order) }
    }

    @Test
    fun `create order with zero total throws exception`() {
        val order = Order(customerId = 1L, total = 0.0)
        assertThrows<IllegalArgumentException> { service.createOrder(order) }
    }
}
```

### MockK (Kotlin-native mocking)

```kotlin
// mockk — regular mock
val repo = mockk<OrderRepository>()
every { repo.findById(1L) } returns Order(1L, "PAID")
every { repo.findById(99L) } throws NotFoundException()

// relaxed mock — returns default values without stubbing
val repo = mockk<OrderRepository>(relaxed = true)

// verify
verify(exactly = 1) { repo.save(any()) }
verify(atLeast = 1) { repo.findByStatus("PAID") }
confirmVerified(repo)

// coEvery / coVerify for suspend functions
coEvery { suspendingRepo.findById(any()) } returns mockUser
coVerify { suspendingRepo.findById(42L) }
```

### Coroutine Testing

```kotlin
class FlowTest {
    @Test
    fun `flow emits expected values`() = runTest {
        val flow = flowOf(1, 2, 3)
        val result = flow.toList()
        assertThat(result).containsExactly(1, 2, 3)
    }

    @Test
    fun `stateflow reflects updates`() = runTest {
        val viewModel = MyViewModel()
        viewModel.load()
        advanceUntilIdle()  // run all pending coroutines
        assertThat(viewModel.uiState.value).isInstanceOf(UiState.Success::class.java)
    }
}
```

---

## Best Practices

These Kotlin best practices distill the key idioms that distinguish experienced Kotlin developers from Java developers writing Kotlin syntax. The core theme is *leveraging the type system* to catch mistakes at compile time rather than runtime: `val` over `var` (immutability prevents state mutation bugs), sealed classes over nullable returns (forces callers to handle error cases), and value classes over raw primitives (prevents passing the wrong ID type). Extension functions keep utilities close to the type they operate on without inheritance. These patterns not only write cleaner code — they communicate your Kotlin fluency clearly in a senior interview.

### Prefer Immutability

```kotlin
// Use val, listOf, mapOf, data class for predictable code
data class Config(val host: String, val port: Int)
val defaults = Config("localhost", 5432)
val prod = defaults.copy(host = "prod.db")  // doesn't mutate defaults
```

### Use Sealed Classes for Exhaustive Handling

```kotlin
// Sealed Result > nullable / exceptions for expected failure cases
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Failure(val error: Throwable) : Result<Nothing>()
}

// Caller forced to handle both cases
when (result) {
    is Result.Success -> process(result.data)
    is Result.Failure -> logError(result.error)
}
```

### Prefer Extension Functions Over Utility Classes

```kotlin
// BAD: StringUtils.isValidEmail(email)
// GOOD:
fun String.isValidEmail(): Boolean = contains("@") && contains(".")
"user@example.com".isValidEmail()
```

### Safe Null Handling Patterns

```kotlin
// Avoid !! — use safe call + Elvis or let
val length = str?.length ?: 0              // default
val result = str?.let { process(it) }      // conditional execution
val user = users.find { it.id == id }
    ?: throw NotFoundException("User $id not found")  // fail fast with message
```

### Coroutine Best Practices

```kotlin
// Always use structured concurrency — don't use GlobalScope in production
class MyService(private val scope: CoroutineScope) {  // inject scope
    fun start() = scope.launch { ... }
}

// Inject dispatchers for testability
class Repository(private val dispatcher: CoroutineDispatcher = Dispatchers.IO) {
    suspend fun load() = withContext(dispatcher) { db.query() }
}
// Test with: Repository(dispatcher = UnconfinedTestDispatcher())

// Use suspend functions instead of returning Deferred directly
suspend fun getUser(id: Long): User = ...   // ✅ caller controls concurrency
fun getUser(id: Long): Deferred<User> = ... // ❌ forces async on caller
```

### Kotlin Idioms Quick Reference

```kotlin
// Create range
(1..10).forEach { print(it) }
(1 until 10)    // excludes 10
(10 downTo 1 step 2).toList()  // [10, 8, 6, 4, 2]

// Check type + smart cast
if (obj is String) println(obj.uppercase())  // obj is String here

// String templates
val msg = "User ${user.name} has ${orders.size} order(s)"

// Multiple assignment (destructuring)
val (first, second) = list

// Swap without temp variable
var a = 1; var b = 2
a = b.also { b = a }

// Execute if not null
user?.let { sendEmail(it.email) }

// Inline condition
val abs = if (n >= 0) n else -n

// run a block and get result
val processed = run {
    val x = compute()
    transform(x)
}
```
