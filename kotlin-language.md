# Kotlin Language Interview Questions

## Table of Contents
1. [Kotlin Fundamentals](#kotlin-fundamentals)
2. [Control Flow and Error Handling](#control-flow-and-error-handling)
3. [Classes and Objects](#classes-and-objects)
4. [Functions and Lambdas](#functions-and-lambdas)
5. [Collections and Functional Constructs](#collections-and-functional-constructs)
6. [Coroutines and Concurrency](#coroutines-and-concurrency)
7. [Java Interoperability](#java-interoperability)
8. [Advanced Topics](#advanced-topics)
9. [Android Specific](#android-specific)
10. [DSL and Meta-Programming](#dsl-and-meta-programming)
11. [Multiplatform Development](#multiplatform-development)
12. [Testing and Tooling](#testing-and-tooling)
13. [Asynchronous Flow](#asynchronous-flow)
14. [Project and Environment Setup](#project-and-environment-setup)
15. [Best Practices](#best-practices)
16. [Future Directions](#future-directions)
17. [Miscellaneous](#miscellaneous)

---

## Kotlin Fundamentals

### Q1: What is Kotlin and how does it interoperate with Java?
**Answer:**
Kotlin is a modern, statically-typed programming language developed by JetBrains that runs on the JVM. It's designed to be fully interoperable with Java while being more concise and safer.

**Key characteristics:**
- **100% Java Interoperability**: Kotlin compiles to JVM bytecode, allowing seamless integration with existing Java code
- **Null Safety**: Built-in null safety prevents NullPointerException at compile time
- **Concise Syntax**: Reduces boilerplate code significantly compared to Java
- **Functional Programming**: First-class support for lambdas, higher-order functions, and immutability

```kotlin
// Kotlin calling Java
val list = ArrayList<String>()  // Java class
list.add("Hello")

// Java calling Kotlin
// In Kotlin file:
fun greet(name: String) = "Hello, $name"

// In Java file:
// String greeting = KotlinFileKt.greet("World");
```

| Feature | Kotlin | Java |
|---------|--------|------|
| Null Safety | Built-in (`?` types) | Optional class (Java 8+) |
| Data Classes | One line | 50+ lines of boilerplate |
| Extension Functions | Supported | Not available |
| Coroutines | Native support | CompletableFuture/Threads |
| Smart Casts | Automatic | Manual casting required |

### Q2: How does Kotlin improve upon Java for Android development?
**Answer:**
Kotlin has become the preferred language for Android development (officially recommended by Google since 2019) due to several improvements:

**1. Null Safety**
```kotlin
// Kotlin prevents NPE at compile time
var name: String = "John"    // Cannot be null
var nickname: String? = null  // Nullable type

// Safe call operator
val length = nickname?.length  // Returns null if nickname is null
```

**2. Concise Syntax**
```kotlin
// Data class in Kotlin (1 line)
data class User(val name: String, val age: Int)

// Equivalent Java requires: constructor, getters, setters, equals, hashCode, toString (~50 lines)
```

**3. Extension Functions**
```kotlin
// Add functionality to existing classes
fun String.addExclamation() = "$this!"
println("Hello".addExclamation())  // "Hello!"
```

**4. Coroutines for Async Operations**
```kotlin
// Clean async code without callbacks
lifecycleScope.launch {
    val user = fetchUser()  // Suspends, doesn't block
    updateUI(user)
}
```

**5. Kotlin Android Extensions & ViewBinding**
```kotlin
// Direct view access (deprecated, use ViewBinding)
// binding.textView.text = "Hello"
```

### Q3: What are the basic types in Kotlin?
**Answer:**
Kotlin has the following basic types, all of which are objects (no primitive types at the language level):

**Numbers:**
```kotlin
val byte: Byte = 127           // 8 bits
val short: Short = 32767       // 16 bits
val int: Int = 2147483647      // 32 bits
val long: Long = 9223372036854775807L  // 64 bits
val float: Float = 3.14f       // 32 bits
val double: Double = 3.14159   // 64 bits
```

**Characters and Strings:**
```kotlin
val char: Char = 'A'
val string: String = "Hello, Kotlin"
val multiline: String = """
    Line 1
    Line 2
""".trimIndent()
```

**Booleans:**
```kotlin
val isTrue: Boolean = true
val isFalse: Boolean = false
```

**Arrays:**
```kotlin
val intArray: IntArray = intArrayOf(1, 2, 3)
val stringArray: Array<String> = arrayOf("a", "b", "c")
```

| Type | Size | Range |
|------|------|-------|
| Byte | 8 bits | -128 to 127 |
| Short | 16 bits | -32768 to 32767 |
| Int | 32 bits | -2^31 to 2^31-1 |
| Long | 64 bits | -2^63 to 2^63-1 |
| Float | 32 bits | IEEE 754 |
| Double | 64 bits | IEEE 754 |

### Q4: Explain the difference between val and var in Kotlin.
**Answer:**
`val` and `var` are used to declare variables with different mutability characteristics:

| Keyword | Mutability | Equivalent in Java |
|---------|------------|-------------------|
| `val` | Immutable (read-only) | `final` variable |
| `var` | Mutable (read-write) | Regular variable |

```kotlin
// val - cannot be reassigned
val name = "John"
// name = "Jane"  // Compilation error!

// var - can be reassigned
var age = 25
age = 26  // OK

// Note: val doesn't mean the object is immutable
val list = mutableListOf(1, 2, 3)
list.add(4)  // OK - modifying the list content
// list = mutableListOf(5, 6)  // Error - cannot reassign
```

**Best Practices:**
- Prefer `val` over `var` for immutability and thread safety
- Use `var` only when reassignment is necessary
- For truly immutable collections, use `listOf()`, `setOf()`, `mapOf()`

```kotlin
// Immutable collection (cannot add/remove)
val immutableList = listOf(1, 2, 3)

// Mutable collection with immutable reference
val mutableList = mutableListOf(1, 2, 3)
```

### Q5: How do you create a singleton in Kotlin?
**Answer:**
Kotlin provides the `object` keyword for creating singletons with thread-safe lazy initialization:

```kotlin
// Singleton using object declaration
object DatabaseConnection {
    val url = "jdbc:mysql://localhost:3306/db"
    
    fun connect() {
        println("Connected to $url")
    }
}

// Usage
DatabaseConnection.connect()
```

**Comparison with Java Singleton:**
```kotlin
// Kotlin - 1 line
object Logger {
    fun log(message: String) = println(message)
}

// Java equivalent - multiple lines with double-checked locking
/*
public class Logger {
    private static volatile Logger instance;
    private Logger() {}
    public static Logger getInstance() {
        if (instance == null) {
            synchronized (Logger.class) {
                if (instance == null) {
                    instance = new Logger();
                }
            }
        }
        return instance;
    }
    public void log(String message) {
        System.out.println(message);
    }
}
*/
```

**Companion Object for Static-like Members:**
```kotlin
class MyClass {
    companion object {
        const val TAG = "MyClass"
        fun create(): MyClass = MyClass()
    }
}

// Usage
val tag = MyClass.TAG
val instance = MyClass.create()
```

### Q6: What are the Kotlin type inference rules?
**Answer:**
Kotlin has powerful type inference that allows the compiler to automatically determine types:

```kotlin
// Type inference for variables
val name = "John"           // Inferred as String
val age = 25                // Inferred as Int
val price = 19.99           // Inferred as Double
val isActive = true         // Inferred as Boolean

// Type inference for functions
fun add(a: Int, b: Int) = a + b  // Return type inferred as Int

// Type inference for lambdas
val numbers = listOf(1, 2, 3)
val doubled = numbers.map { it * 2 }  // it inferred as Int
```

**When explicit types are required:**
```kotlin
// 1. Function parameters always need explicit types
fun greet(name: String) { }  // String is required

// 2. Public API return types (recommended)
fun calculate(): Int = 42

// 3. Late-initialized properties
lateinit var service: UserService

// 4. When inference is ambiguous
val number: Long = 42  // Without : Long, would be Int
```

**Type inference limitations:**
```kotlin
// Cannot infer from multiple expressions
val result = if (condition) "text" else 42  // Inferred as Any

// Collections need element type or inference
val emptyList = emptyList<String>()  // Type parameter required
val inferredList = listOf("a", "b")  // Inferred as List<String>
```

### Q7: Can Kotlin code be executed without a main function?
**Answer:**
For standalone JVM applications, a `main` function is required as the entry point. However, there are scenarios where Kotlin code executes without an explicit `main`:

**1. Standard Entry Point:**
```kotlin
fun main() {
    println("Hello, World!")
}

// Or with command-line arguments
fun main(args: Array<String>) {
    println("Arguments: ${args.joinToString()}")
}
```

**2. Kotlin Script (.kts files):**
```kotlin
// script.kts - executed directly without main
println("This is a script!")
val result = 2 + 2
println("Result: $result")
```

**3. Kotlin REPL:**
```kotlin
// Interactive mode - no main needed
>>> val x = 10
>>> println(x * 2)
20
```

**4. Android/Frameworks:**
```kotlin
// Android - Activity lifecycle, no main needed
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Entry point managed by Android framework
    }
}
```

**5. Test Classes:**
```kotlin
class MyTest {
    @Test
    fun testSomething() {
        // JUnit manages execution
    }
}
```

### Q8: What is the purpose of the Unit type in Kotlin?
**Answer:**
`Unit` is Kotlin's equivalent to Java's `void`, representing a function that doesn't return a meaningful value. Unlike `void`, `Unit` is an actual type with a singleton instance.

```kotlin
// Explicit Unit return type (optional)
fun printMessage(message: String): Unit {
    println(message)
}

// Implicit Unit (preferred style)
fun printMessage(message: String) {
    println(message)
}

// Unit is a real object
val unit: Unit = Unit
println(unit)  // Prints: kotlin.Unit
```

**Differences from Java void:**

| Feature | Kotlin Unit | Java void |
|---------|-------------|-----------|
| Is a type | Yes | No (keyword) |
| Can be used as generic | Yes | No (use Void wrapper) |
| Has a value | Yes (singleton) | No |
| Can be returned | Yes | N/A |

```kotlin
// Unit as generic type parameter
fun <T> execute(action: () -> T): T = action()

// Works with Unit-returning lambdas
execute { println("Hello") }  // Returns Unit

// Java would require:
// Callable<Void> callable = () -> { System.out.println("Hello"); return null; };
```

### Q9: How do you perform string interpolation in Kotlin?
**Answer:**
Kotlin supports string interpolation using the `$` symbol for variables and `${}` for expressions:

```kotlin
val name = "John"
val age = 30

// Simple variable interpolation
println("Hello, $name!")  // Hello, John!

// Expression interpolation
println("In 5 years, you'll be ${age + 5}")  // In 5 years, you'll be 35

// Accessing properties
data class Person(val name: String, val age: Int)
val person = Person("Alice", 25)
println("Name: ${person.name}, Age: ${person.age}")

// Function calls in interpolation
println("Name uppercase: ${name.uppercase()}")

// Escaping dollar sign
println("Price: \$99.99")  // Price: $99.99
```

**Raw strings (triple quotes):**
```kotlin
val json = """
    {
        "name": "$name",
        "age": $age
    }
""".trimIndent()

// With trimMargin for custom margin
val text = """
    |First line
    |Second line
    |Third line
""".trimMargin()
```

### Q10: What are extension functions in Kotlin?
**Answer:**
Extension functions allow adding new functions to existing classes without modifying their source code or using inheritance:

```kotlin
// Adding a function to String class
fun String.addExclamation(): String {
    return "$this!"
}

println("Hello".addExclamation())  // Hello!

// Extension with receiver
fun Int.isEven(): Boolean = this % 2 == 0
println(4.isEven())  // true

// Extension on nullable types
fun String?.orEmpty(): String = this ?: ""
val nullString: String? = null
println(nullString.orEmpty())  // ""
```

**Extension properties:**
```kotlin
val String.lastChar: Char
    get() = this[length - 1]

println("Kotlin".lastChar)  // n
```

**Important characteristics:**
- Extensions are resolved **statically** (at compile time)
- They don't actually modify the class
- Member functions take precedence over extensions

```kotlin
open class Shape
class Rectangle : Shape()

fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"

fun printName(s: Shape) {
    println(s.getName())  // Always prints "Shape" - static resolution
}

printName(Rectangle())  // Prints: Shape
```

**Common use cases:**
```kotlin
// Utility extensions
fun <T> List<T>.secondOrNull(): T? = if (size >= 2) this[1] else null

// Android extensions
fun Context.toast(message: String) {
    Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
}

// DSL building
fun StringBuilder.appendLine(line: String) {
    append(line).append("\n")
}
```

---

## Control Flow and Error Handling

### Q11: How are if expressions used in Kotlin as compared to Java?
**Answer:**
In Kotlin, `if` is an **expression** that returns a value, unlike Java where it's a statement:

```kotlin
// Kotlin - if as expression
val max = if (a > b) a else b

// Java equivalent
// int max = (a > b) ? a : b;  // Must use ternary
// or
// int max;
// if (a > b) { max = a; } else { max = b; }
```

**Key differences:**

| Feature | Kotlin | Java |
|---------|--------|------|
| Returns value | Yes | No (use ternary) |
| Ternary operator | Not needed | `? :` |
| Block expressions | Last expression is result | N/A |

```kotlin
// Block expressions
val result = if (score >= 90) {
    println("Excellent!")
    "A"  // Last expression is the return value
} else if (score >= 80) {
    println("Good!")
    "B"
} else {
    println("Keep trying!")
    "C"
}

// Replacing ternary operator
val status = if (isActive) "Active" else "Inactive"

// In function return
fun getDiscount(isMember: Boolean) = if (isMember) 0.2 else 0.0
```

### Q12: Explain when expressions in Kotlin.
**Answer:**
`when` is Kotlin's powerful replacement for Java's `switch`, supporting pattern matching and expressions:

```kotlin
// Basic when expression
val result = when (x) {
    1 -> "One"
    2 -> "Two"
    3, 4 -> "Three or Four"  // Multiple values
    in 5..10 -> "Between 5 and 10"  // Range
    else -> "Unknown"
}

// When without argument (replaces if-else chain)
val grade = when {
    score >= 90 -> "A"
    score >= 80 -> "B"
    score >= 70 -> "C"
    else -> "F"
}
```

**Advanced patterns:**
```kotlin
// Type checking with smart cast
fun describe(obj: Any): String = when (obj) {
    is String -> "String of length ${obj.length}"
    is Int -> "Integer: $obj"
    is List<*> -> "List with ${obj.size} items"
    else -> "Unknown type"
}

// Capturing subject in variable
when (val response = fetchData()) {
    is Success -> println(response.data)
    is Error -> println(response.message)
}

// When as statement (no else required if exhaustive)
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
}

fun handle(result: Result) = when (result) {
    is Result.Success -> println(result.data)
    is Result.Error -> println(result.message)
    // No else needed - sealed class is exhaustive
}
```

### Q13: How does Kotlin handle null safety and what is the Elvis operator?
**Answer:**
Kotlin's type system distinguishes between nullable and non-nullable types to prevent NullPointerException:

```kotlin
// Non-nullable (cannot be null)
var name: String = "John"
// name = null  // Compilation error!

// Nullable (can be null)
var nickname: String? = null
nickname = "Johnny"  // OK
```

**Safe call operator (`?.`):**
```kotlin
val length = nickname?.length  // Returns null if nickname is null
// Chaining
val city = user?.address?.city
```

**Elvis operator (`?:`):**
```kotlin
// Provides default value when null
val displayName = nickname ?: "Guest"

// With throw or return
val name = nickname ?: throw IllegalArgumentException("Name required")
val name = nickname ?: return
```

**Not-null assertion (`!!`):**
```kotlin
// Throws NullPointerException if null (use sparingly!)
val length = nickname!!.length
```

**Safe casts:**
```kotlin
val str: String? = obj as? String  // Returns null if cast fails
```

**let for null checks:**
```kotlin
nickname?.let { 
    println("Nickname is $it")
}
```

| Operator | Purpose | Example |
|----------|---------|---------|
| `?` | Nullable type | `String?` |
| `?.` | Safe call | `name?.length` |
| `?:` | Elvis (default) | `name ?: "default"` |
| `!!` | Not-null assertion | `name!!` |
| `as?` | Safe cast | `obj as? String` |

### Q14: What is a "smart cast" in Kotlin?
**Answer:**
Smart cast is Kotlin's ability to automatically cast types after a type check, eliminating the need for explicit casting:

```kotlin
fun process(obj: Any) {
    if (obj is String) {
        // obj is automatically cast to String here
        println(obj.length)  // No explicit cast needed
        println(obj.uppercase())
    }
}

// Works with when expressions
fun describe(obj: Any): String = when (obj) {
    is Int -> "Integer: ${obj * 2}"  // Smart cast to Int
    is String -> "String: ${obj.uppercase()}"  // Smart cast to String
    is List<*> -> "List size: ${obj.size}"  // Smart cast to List
    else -> "Unknown"
}
```

**Smart cast with null checks:**
```kotlin
fun printLength(str: String?) {
    if (str != null) {
        // str is smart cast to non-nullable String
        println(str.length)
    }
    
    // Also works with return/throw
    str ?: return
    println(str.length)  // str is non-nullable here
}
```

**Limitations (when smart cast doesn't work):**
```kotlin
class Example {
    var mutableProperty: String? = null
    
    fun test() {
        if (mutableProperty != null) {
            // Cannot smart cast - property could change between check and use
            // println(mutableProperty.length)  // Error!
            
            // Solution: use local variable
            val local = mutableProperty
            if (local != null) {
                println(local.length)  // Works!
            }
        }
    }
}
```

### Q15: How do you implement a custom getter and setter in Kotlin?
**Answer:**
Kotlin properties can have custom accessors defined inline:

```kotlin
class Person {
    // Property with custom getter
    val isAdult: Boolean
        get() = age >= 18
    
    // Property with custom getter and setter
    var age: Int = 0
        get() = field  // 'field' is the backing field
        set(value) {
            if (value >= 0) {
                field = value
            }
        }
    
    // Property with private setter
    var name: String = ""
        private set
    
    // Computed property (no backing field)
    val info: String
        get() = "$name is $age years old"
}
```

**Backing field (`field`):**
```kotlin
var counter: Int = 0
    set(value) {
        if (value >= 0) {
            field = value  // Use 'field' to access backing field
            // counter = value  // Would cause infinite recursion!
        }
    }
```

**Late initialization:**
```kotlin
class Service {
    // For non-nullable properties initialized later
    lateinit var repository: Repository
    
    fun initialize() {
        repository = Repository()
    }
    
    fun isInitialized() = ::repository.isInitialized
}
```

**Lazy initialization:**
```kotlin
class Config {
    // Computed only when first accessed
    val settings: Map<String, String> by lazy {
        loadSettings()  // Expensive operation
    }
}
```

### Q16: Describe exception handling in Kotlin.
**Answer:**
Kotlin uses try-catch-finally for exception handling, similar to Java, but with key differences:

```kotlin
// Basic try-catch
try {
    val result = riskyOperation()
} catch (e: IOException) {
    println("IO Error: ${e.message}")
} catch (e: Exception) {
    println("Error: ${e.message}")
} finally {
    cleanup()
}
```

**Try as an expression:**
```kotlin
val result = try {
    parseInt(input)
} catch (e: NumberFormatException) {
    0  // Default value
}

// Or with null
val number: Int? = try {
    parseInt(input)
} catch (e: NumberFormatException) {
    null
}
```

**Key differences from Java:**

| Feature | Kotlin | Java |
|---------|--------|------|
| Checked exceptions | No | Yes |
| Try is expression | Yes | No |
| Throw is expression | Yes (returns Nothing) | No |

```kotlin
// No checked exceptions - no forced try-catch
fun readFile(path: String): String {
    return File(path).readText()  // IOException not required to be caught
}

// Throw as expression
val value = map[key] ?: throw IllegalArgumentException("Key not found")
```

**Use function for resource management:**
```kotlin
// Equivalent to try-with-resources
File("data.txt").bufferedReader().use { reader ->
    println(reader.readText())
}  // Automatically closes reader
```

### Q17: What are the differences between throw, try, catch, and finally in Kotlin versus other languages?
**Answer:**

**throw - Returns Nothing type:**
```kotlin
// throw is an expression in Kotlin
val result = name ?: throw IllegalArgumentException("Name required")

// Nothing type indicates function never returns normally
fun fail(message: String): Nothing {
    throw IllegalStateException(message)
}

// Useful for type inference
val x = if (condition) value else fail("Error")  // x is type of value
```

**try - Is an expression:**
```kotlin
// Can assign result of try
val parsed = try {
    input.toInt()
} catch (e: NumberFormatException) {
    -1
}
```

**catch - No checked exceptions:**
```kotlin
// Kotlin doesn't have checked exceptions
// Java forces: throws IOException
fun readData(): String {
    return URL("http://example.com").readText()
    // No need to declare or catch IOException
}
```

**finally - Same as Java:**
```kotlin
var connection: Connection? = null
try {
    connection = openConnection()
    // use connection
} finally {
    connection?.close()  // Always executed
}
```

**Comparison table:**

| Construct | Kotlin | Java |
|-----------|--------|------|
| `throw` | Expression (returns `Nothing`) | Statement |
| `try` | Expression (returns value) | Statement |
| Checked exceptions | Not enforced | Enforced |
| `finally` | Same behavior | Same behavior |

### Q18: How does Kotlin's Nothing type work in control flow?
**Answer:**
`Nothing` is a type with no instances, representing computations that never complete normally:

```kotlin
// Functions that never return
fun fail(message: String): Nothing {
    throw IllegalStateException(message)
}

fun infinite(): Nothing {
    while (true) {
        // Never terminates
    }
}
```

**Type system integration:**
```kotlin
// Nothing is subtype of all types
val result: String = if (condition) {
    "success"
} else {
    fail("error")  // Returns Nothing, compatible with String
}

// Elvis operator with throw
val name: String = nullableName ?: throw IllegalArgumentException()
// throw returns Nothing, so entire expression is String

// With return
fun process(value: String?): String {
    val nonNull = value ?: return "default"
    return nonNull.uppercase()
}
```

**Nothing vs Unit vs Any:**

| Type | Purpose | Has instances |
|------|---------|---------------|
| `Any` | Supertype of all non-null types | Yes (all objects) |
| `Unit` | No meaningful return value | Yes (singleton) |
| `Nothing` | Never returns normally | No |

```kotlin
// Nothing? can only hold null
val nothing: Nothing? = null
// val x: Nothing = ???  // Cannot create instance

// Useful in generic contexts
fun <T> emptyList(): List<T> = object : List<Nothing> { /* ... */ } as List<T>
```


---

## Classes and Objects

### Q19: How do you create classes in Kotlin?
**Answer:**
Kotlin classes are declared using the `class` keyword with concise syntax:

```kotlin
// Simple class
class Person

// Class with properties in primary constructor
class Person(val name: String, var age: Int)

// Class with body
class Person(val name: String) {
    var age: Int = 0
    
    fun greet() = println("Hello, I'm $name")
}

// Class with init block
class Person(name: String) {
    val upperName: String
    
    init {
        upperName = name.uppercase()
        println("Person created: $upperName")
    }
}
```

**Visibility modifiers:**
```kotlin
class Example {
    public val a = 1      // Visible everywhere (default)
    private val b = 2     // Visible only in this class
    protected val c = 3   // Visible in this class and subclasses
    internal val d = 4    // Visible in the same module
}
```

### Q20: Explain primary and secondary constructors in Kotlin.
**Answer:**

**Primary Constructor:**
```kotlin
// Primary constructor in class header
class Person(val name: String, var age: Int)

// With default values
class Person(val name: String = "Unknown", var age: Int = 0)

// With init block for initialization logic
class Person(val name: String) {
    val isValid: Boolean
    
    init {
        require(name.isNotBlank()) { "Name cannot be blank" }
        isValid = true
    }
}
```

**Secondary Constructors:**
```kotlin
class Person {
    val name: String
    var age: Int
    
    // Primary constructor
    constructor(name: String, age: Int) {
        this.name = name
        this.age = age
    }
    
    // Secondary constructor - must delegate to primary
    constructor(name: String) : this(name, 0)
}

// With both primary and secondary
class Person(val name: String) {
    var age: Int = 0
    var email: String = ""
    
    constructor(name: String, age: Int) : this(name) {
        this.age = age
    }
    
    constructor(name: String, age: Int, email: String) : this(name, age) {
        this.email = email
    }
}
```

### Q21: What are data classes in Kotlin?
**Answer:**
Data classes automatically generate common methods for holding data:

```kotlin
data class User(val name: String, val age: Int, val email: String)

// Automatically generates:
// - equals() / hashCode()
// - toString() -> "User(name=John, age=30, email=john@email.com)"
// - componentN() functions for destructuring
// - copy() function
```

**Usage:**
```kotlin
val user = User("John", 30, "john@email.com")

// toString()
println(user)  // User(name=John, age=30, email=john@email.com)

// copy() with modifications
val olderUser = user.copy(age = 31)

// Destructuring
val (name, age, email) = user

// equals()
val user2 = User("John", 30, "john@email.com")
println(user == user2)  // true
```

**Requirements:**
- Primary constructor must have at least one parameter
- Parameters must be `val` or `var`
- Cannot be abstract, open, sealed, or inner

**Comparison with Java:**
```kotlin
// Kotlin: 1 line
data class User(val name: String, val age: Int)

// Java equivalent: ~50 lines with constructor, getters, equals, hashCode, toString
```

### Q22: How does inheritance work in Kotlin?
**Answer:**
Classes in Kotlin are `final` by default. Use `open` to allow inheritance:

```kotlin
// Must be 'open' to be extended
open class Animal(val name: String) {
    open fun makeSound() {
        println("Some sound")
    }
}

// Extending a class
class Dog(name: String, val breed: String) : Animal(name) {
    override fun makeSound() {
        println("Bark!")
    }
}
```

**Abstract classes:**
```kotlin
abstract class Shape {
    abstract fun area(): Double
    
    fun describe() = println("This is a shape with area ${area()}")
}

class Circle(val radius: Double) : Shape() {
    override fun area() = Math.PI * radius * radius
}
```

**Interface implementation:**
```kotlin
interface Drawable {
    fun draw()
    fun getColor(): String = "Black"  // Default implementation
}

class Square : Shape(), Drawable {
    override fun area() = /* ... */
    override fun draw() = /* ... */
}
```

**Calling superclass:**
```kotlin
open class Parent {
    open fun greet() = println("Hello from Parent")
}

class Child : Parent() {
    override fun greet() {
        super.greet()
        println("Hello from Child")
    }
}
```

### Q23: What are sealed classes in Kotlin?
**Answer:**
Sealed classes restrict class hierarchies to a limited set of types, enabling exhaustive `when` expressions:

```kotlin
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val exception: Exception) : Result()
    object Loading : Result()
}

// Exhaustive when - no else needed
fun handle(result: Result): String = when (result) {
    is Result.Success -> "Data: ${result.data}"
    is Result.Error -> "Error: ${result.exception.message}"
    is Result.Loading -> "Loading..."
    // Compiler ensures all cases are covered
}
```

**Benefits:**
- Compile-time exhaustiveness checking
- Safe state modeling
- Restricted inheritance hierarchy

**Sealed interfaces (Kotlin 1.5+):**
```kotlin
sealed interface ApiResponse
data class SuccessResponse(val body: String) : ApiResponse
data class ErrorResponse(val code: Int, val message: String) : ApiResponse
```

**Use cases:**
```kotlin
// State management
sealed class UiState {
    object Loading : UiState()
    data class Success(val items: List<Item>) : UiState()
    data class Error(val message: String) : UiState()
}

// Network responses
sealed class NetworkResult<out T> {
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Failure(val error: Throwable) : NetworkResult<Nothing>()
}
```

### Q24: Explain how properties and fields differ in Kotlin.
**Answer:**
In Kotlin, you work with **properties** (not fields directly). Fields are implementation details:

```kotlin
class Person {
    // This is a property
    var name: String = "John"
        get() = field.uppercase()  // 'field' is the backing field
        set(value) {
            field = value.trim()
        }
}

// Properties without backing fields (computed)
class Rectangle(val width: Int, val height: Int) {
    val area: Int
        get() = width * height  // Computed on each access, no backing field
}
```

**Backing field:**
```kotlin
var counter: Int = 0
    set(value) {
        if (value >= 0) field = value  // 'field' accesses backing field
        // counter = value  // ERROR: infinite recursion!
    }
```

**Backing property pattern:**
```kotlin
class Example {
    private var _items: MutableList<String> = mutableListOf()
    
    val items: List<String>
        get() = _items  // Expose as immutable
    
    fun addItem(item: String) {
        _items.add(item)
    }
}
```

### Q25: What is object expression and when do you use it?
**Answer:**
Object expressions create anonymous objects (similar to Java anonymous classes):

```kotlin
// Anonymous object implementing interface
val clickListener = object : View.OnClickListener {
    override fun onClick(view: View) {
        println("Clicked!")
    }
}

// Anonymous object extending class
val comparator = object : Comparator<String> {
    override fun compare(s1: String, s2: String): Int {
        return s1.length - s2.length
    }
}

// Anonymous object with multiple supertypes
interface A { fun a() }
interface B { fun b() }

val obj = object : A, B {
    override fun a() = println("A")
    override fun b() = println("B")
}
```

**Object expression without supertype:**
```kotlin
fun getPoint() = object {
    val x = 10
    val y = 20
}

// Usage in local/private scope
private fun createPair() = object {
    val first = 1
    val second = 2
}
val pair = createPair()
println("${pair.first}, ${pair.second}")  // 1, 2
```

### Q26: What are companion objects in Kotlin?
**Answer:**
Companion objects provide static-like functionality in Kotlin:

```kotlin
class MyClass {
    companion object {
        const val TAG = "MyClass"
        
        fun create(): MyClass = MyClass()
        
        @JvmStatic  // For Java interop
        fun staticMethod() = println("Called statically")
    }
}

// Usage
val tag = MyClass.TAG
val instance = MyClass.create()
MyClass.staticMethod()
```

**Named companion object:**
```kotlin
class User private constructor(val name: String) {
    companion object Factory {
        fun create(name: String): User = User(name)
    }
}

// Usage
val user = User.Factory.create("John")
// or
val user = User.create("John")
```

**Companion object implementing interface:**
```kotlin
interface Factory<T> {
    fun create(): T
}

class Product {
    companion object : Factory<Product> {
        override fun create() = Product()
    }
}

// Can be passed where Factory is expected
fun <T> buildItem(factory: Factory<T>): T = factory.create()
val product = buildItem(Product)  // Companion object as argument
```

### Q27: How do you define an enum in Kotlin?
**Answer:**

```kotlin
// Simple enum
enum class Direction {
    NORTH, SOUTH, EAST, WEST
}

// Enum with properties
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF);
    
    fun containsRed() = (rgb and 0xFF0000) != 0
}

// Enum implementing interface
enum class Status : Describable {
    PENDING {
        override fun describe() = "Waiting..."
    },
    APPROVED {
        override fun describe() = "Approved!"
    },
    REJECTED {
        override fun describe() = "Rejected!"
    }
}

interface Describable {
    fun describe(): String
}
```

**Using enums:**
```kotlin
val direction = Direction.NORTH

// Enum properties
println(direction.name)     // "NORTH"
println(direction.ordinal)  // 0

// Iteration
Direction.values().forEach { println(it) }

// Parse from string
val parsed = Direction.valueOf("NORTH")

// In when expression
val message = when (direction) {
    Direction.NORTH -> "Going up"
    Direction.SOUTH -> "Going down"
    Direction.EAST -> "Going right"
    Direction.WEST -> "Going left"
}
```

---

## Functions and Lambdas

### Q28: How do you define functions in Kotlin?
**Answer:**

```kotlin
// Standard function
fun add(a: Int, b: Int): Int {
    return a + b
}

// Single-expression function
fun add(a: Int, b: Int) = a + b

// Function with default parameters
fun greet(name: String, greeting: String = "Hello") {
    println("$greeting, $name!")
}

// Function with named arguments
greet(name = "John", greeting = "Hi")
greet(name = "Jane")  // Uses default greeting

// Vararg parameters
fun printAll(vararg messages: String) {
    messages.forEach { println(it) }
}
printAll("Hello", "World", "!")

// Spread operator for arrays
val array = arrayOf("a", "b", "c")
printAll(*array)
```

**Local functions:**
```kotlin
fun processUser(user: User) {
    fun validate(value: String, name: String) {
        require(value.isNotBlank()) { "$name cannot be blank" }
    }
    
    validate(user.name, "Name")
    validate(user.email, "Email")
    // Process user...
}
```

### Q29: What is a higher-order function in Kotlin?
**Answer:**
A higher-order function takes functions as parameters or returns a function:

```kotlin
// Function as parameter
fun operateOnNumbers(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

// Usage
val sum = operateOnNumbers(5, 3) { x, y -> x + y }
val product = operateOnNumbers(5, 3) { x, y -> x * y }

// Function returning a function
fun multiplier(factor: Int): (Int) -> Int {
    return { number -> number * factor }
}

val triple = multiplier(3)
println(triple(4))  // 12
```

**Common higher-order functions:**
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

// map - transform each element
val doubled = numbers.map { it * 2 }  // [2, 4, 6, 8, 10]

// filter - keep matching elements
val evens = numbers.filter { it % 2 == 0 }  // [2, 4]

// reduce - combine elements
val sum = numbers.reduce { acc, n -> acc + n }  // 15

// fold - reduce with initial value
val sumPlus10 = numbers.fold(10) { acc, n -> acc + n }  // 25

// forEach - iterate with side effects
numbers.forEach { println(it) }
```

### Q30: What is the purpose of inline functions?
**Answer:**
Inline functions copy the function body and lambda code directly to the call site, eliminating the overhead of function calls and lambda object creation:

```kotlin
// Without inline: creates lambda object each call
fun measure(block: () -> Unit) {
    val start = System.currentTimeMillis()
    block()
    println("Time: ${System.currentTimeMillis() - start}ms")
}

// With inline: no lambda object created
inline fun measureInline(block: () -> Unit) {
    val start = System.currentTimeMillis()
    block()
    println("Time: ${System.currentTimeMillis() - start}ms")
}
```

**Benefits:**
- No lambda object allocation
- Enables non-local returns
- Enables reified type parameters

```kotlin
// Non-local return (only with inline)
inline fun findFirst(list: List<Int>, predicate: (Int) -> Boolean): Int? {
    for (item in list) {
        if (predicate(item)) return item
    }
    return null
}

// Reified type parameters
inline fun <reified T> isType(value: Any): Boolean {
    return value is T  // Only possible with reified
}

println(isType<String>("hello"))  // true
println(isType<Int>("hello"))     // false
```

**noinline and crossinline:**
```kotlin
inline fun execute(
    inlinedLambda: () -> Unit,
    noinline notInlinedLambda: () -> Unit,  // Won't be inlined
    crossinline noReturnLambda: () -> Unit  // Inlined but no non-local return
) {
    inlinedLambda()
    notInlinedLambda()
    thread { 
        crossinline()  // Safe to use in different context
    }
}
```

### Q31: How do you use lambdas in Kotlin?
**Answer:**

```kotlin
// Lambda syntax
val sum: (Int, Int) -> Int = { a, b -> a + b }

// Type inference
val multiply = { a: Int, b: Int -> a * b }

// Single parameter uses 'it'
val double: (Int) -> Int = { it * 2 }

// Lambda with receiver
val greet: String.() -> String = { "Hello, $this!" }
println("World".greet())  // Hello, World!
```

**Trailing lambda syntax:**
```kotlin
// When lambda is last parameter, can be outside parentheses
listOf(1, 2, 3).map { it * 2 }

// If only parameter, parentheses can be omitted
run { println("Hello") }

// Multiple lambdas - only last can be trailing
listOf(1, 2, 3).fold(0, { acc, n -> acc + n })
// or
listOf(1, 2, 3).fold(0) { acc, n -> acc + n }
```

**Destructuring in lambdas:**
```kotlin
val map = mapOf("a" to 1, "b" to 2)
map.forEach { (key, value) ->
    println("$key = $value")
}

// With data classes
data class Person(val name: String, val age: Int)
listOf(Person("John", 30)).forEach { (name, age) ->
    println("$name is $age")
}
```

### Q32: Explain the use of with and apply in Kotlin.
**Answer:**
Kotlin provides scope functions for cleaner code:

**`with` - call multiple methods on an object:**
```kotlin
val result = with(StringBuilder()) {
    append("Hello")
    append(" ")
    append("World")
    toString()  // Returns this value
}
// result = "Hello World"
```

**`apply` - configure an object and return it:**
```kotlin
val person = Person().apply {
    name = "John"
    age = 30
    email = "john@email.com"
}
// Returns the configured person object
```

**All scope functions comparison:**

| Function | Object ref | Return value | Use case |
|----------|-----------|--------------|----------|
| `let` | `it` | Lambda result | Null checks, transformations |
| `run` | `this` | Lambda result | Object configuration + compute |
| `with` | `this` | Lambda result | Grouping function calls |
| `apply` | `this` | Context object | Object configuration |
| `also` | `it` | Context object | Side effects |

```kotlin
// let - for nullable and transformations
val length = name?.let { it.length } ?: 0

// run - configuration + result
val service = Service().run {
    port = 8080
    start()
    this  // or return computed value
}

// also - for side effects
val numbers = mutableListOf(1, 2, 3).also {
    println("Before adding: $it")
}.also {
    it.add(4)
}
```

### Q33: What are tail recursive functions and how do you define one in Kotlin?
**Answer:**
Tail recursion optimizes recursive calls to prevent stack overflow by converting recursion to iteration:

```kotlin
// Regular recursion (can stack overflow)
fun factorial(n: Long): Long {
    return if (n <= 1) 1 else n * factorial(n - 1)
}

// Tail recursive (optimized)
tailrec fun factorialTailRec(n: Long, accumulator: Long = 1): Long {
    return if (n <= 1) accumulator
    else factorialTailRec(n - 1, n * accumulator)
}
```

**Requirements for `tailrec`:**
- Function must call itself as the **last** operation
- Cannot be used in try/catch/finally blocks
- Currently only supported for JVM

```kotlin
// Valid tail recursion
tailrec fun findFixPoint(x: Double = 1.0): Double {
    return if (Math.abs(x - Math.cos(x)) < 1e-10) x
    else findFixPoint(Math.cos(x))
}

// NOT tail recursive (operation after recursive call)
fun sumNotTailRec(n: Int): Int {
    return if (n <= 0) 0
    else n + sumNotTailRec(n - 1)  // Addition after call
}

// Tail recursive version
tailrec fun sumTailRec(n: Int, acc: Int = 0): Int {
    return if (n <= 0) acc
    else sumTailRec(n - 1, acc + n)
}
```

### Q34: What are default and named parameters in Kotlin?
**Answer:**

**Default parameters:**
```kotlin
fun greet(
    name: String,
    greeting: String = "Hello",
    punctuation: String = "!"
) {
    println("$greeting, $name$punctuation")
}

greet("John")              // Hello, John!
greet("John", "Hi")        // Hi, John!
greet("John", "Hey", "?")  // Hey, John?
```

**Named parameters:**
```kotlin
// Skip parameters with defaults using names
greet(name = "John", punctuation = "?")  // Hello, John?

// Reorder parameters
greet(punctuation = "!!!", greeting = "Welcome", name = "John")
```

**Benefits over Java overloading:**
```kotlin
// Kotlin: 1 function with defaults
fun createUser(
    name: String,
    age: Int = 0,
    email: String = "",
    isActive: Boolean = true
) = User(name, age, email, isActive)

// Java would need multiple overloads:
// User createUser(String name)
// User createUser(String name, int age)
// User createUser(String name, int age, String email)
// User createUser(String name, int age, String email, boolean isActive)
```

**With @JvmOverloads for Java interop:**
```kotlin
class Builder @JvmOverloads constructor(
    val width: Int = 100,
    val height: Int = 100,
    val color: String = "black"
)
// Generates overloaded constructors for Java
```


---

## Collections and Functional Constructs

### Q35: How do you use lists, sets, and maps in Kotlin?
**Answer:**

**Lists:**
```kotlin
// Immutable list
val immutableList = listOf(1, 2, 3)
// immutableList.add(4)  // Error: no add method

// Mutable list
val mutableList = mutableListOf(1, 2, 3)
mutableList.add(4)

// ArrayList
val arrayList = arrayListOf(1, 2, 3)

// Empty list with type
val emptyList = emptyList<String>()

// List operations
val first = immutableList.first()
val last = immutableList.last()
val element = immutableList[1]
```

**Sets:**
```kotlin
// Immutable set (no duplicates)
val immutableSet = setOf(1, 2, 2, 3)  // [1, 2, 3]

// Mutable set
val mutableSet = mutableSetOf(1, 2, 3)
mutableSet.add(4)
mutableSet.remove(1)

// LinkedHashSet (maintains insertion order)
val linkedSet = linkedSetOf(3, 1, 2)

// HashSet
val hashSet = hashSetOf(1, 2, 3)
```

**Maps:**
```kotlin
// Immutable map
val immutableMap = mapOf("a" to 1, "b" to 2)
val value = immutableMap["a"]  // 1

// Mutable map
val mutableMap = mutableMapOf("a" to 1)
mutableMap["b"] = 2
mutableMap.remove("a")

// HashMap
val hashMap = hashMapOf("a" to 1)

// Iteration
for ((key, value) in immutableMap) {
    println("$key -> $value")
}
```

### Q36: What is the difference between map and flatMap in Kotlin?
**Answer:**

**`map` - transforms each element:**
```kotlin
val numbers = listOf(1, 2, 3)
val doubled = numbers.map { it * 2 }  // [2, 4, 6]

val words = listOf("hello", "world")
val lengths = words.map { it.length }  // [5, 5]
```

**`flatMap` - transforms and flattens:**
```kotlin
val nested = listOf(listOf(1, 2), listOf(3, 4))
val flattened = nested.flatMap { it }  // [1, 2, 3, 4]

// More useful example
val sentences = listOf("Hello World", "Kotlin Programming")
val words = sentences.flatMap { it.split(" ") }
// ["Hello", "World", "Kotlin", "Programming"]

// Processing nested data
data class Department(val employees: List<String>)
val departments = listOf(
    Department(listOf("Alice", "Bob")),
    Department(listOf("Charlie"))
)
val allEmployees = departments.flatMap { it.employees }
// ["Alice", "Bob", "Charlie"]
```

**Comparison:**
| Operation | Input | Output |
|-----------|-------|--------|
| `map { }` | List<T> | List<R> |
| `flatMap { }` | List<T> | List<R> (flattened) |

```kotlin
// Visual difference
val numbers = listOf(1, 2, 3)

// map produces List<List<Int>>
val mapped = numbers.map { listOf(it, it * 10) }
// [[1, 10], [2, 20], [3, 30]]

// flatMap produces List<Int>
val flatMapped = numbers.flatMap { listOf(it, it * 10) }
// [1, 10, 2, 20, 3, 30]
```

### Q37: Explain lazy collection evaluation in Kotlin.
**Answer:**
Lazy evaluation delays computation until the result is actually needed:

```kotlin
// Eager evaluation (immediate)
val eagerResult = listOf(1, 2, 3, 4, 5)
    .map { println("map $it"); it * 2 }
    .filter { println("filter $it"); it > 4 }
    .first()
// Processes ALL elements through map, THEN all through filter

// Lazy evaluation with sequences
val lazyResult = listOf(1, 2, 3, 4, 5)
    .asSequence()
    .map { println("map $it"); it * 2 }
    .filter { println("filter $it"); it > 4 }
    .first()
// Processes element by element until first match found
```

**Lazy property delegation:**
```kotlin
class ExpensiveResource {
    val data: String by lazy {
        println("Computing...")
        loadExpensiveData()  // Only called once, when first accessed
    }
}

val resource = ExpensiveResource()
println("Created")  // "Created" printed
println(resource.data)  // "Computing..." then data printed
println(resource.data)  // Just data (cached)
```

**Thread-safe lazy modes:**
```kotlin
// Default: synchronized (thread-safe)
val sync by lazy { computeValue() }

// Publication: multiple threads may compute, first wins
val pub by lazy(LazyThreadSafetyMode.PUBLICATION) { computeValue() }

// None: no synchronization (single-threaded use only)
val none by lazy(LazyThreadSafetyMode.NONE) { computeValue() }
```

### Q38: What are the different ways to iterate over a collection in Kotlin?
**Answer:**

```kotlin
val list = listOf("a", "b", "c")

// 1. for loop
for (item in list) {
    println(item)
}

// 2. forEach
list.forEach { println(it) }

// 3. forEachIndexed
list.forEachIndexed { index, item ->
    println("$index: $item")
}

// 4. Iterator
val iterator = list.iterator()
while (iterator.hasNext()) {
    println(iterator.next())
}

// 5. Indices
for (i in list.indices) {
    println(list[i])
}

// 6. withIndex
for ((index, value) in list.withIndex()) {
    println("$index: $value")
}

// 7. Range with step
for (i in 0 until list.size step 2) {
    println(list[i])
}
```

**Map iteration:**
```kotlin
val map = mapOf("a" to 1, "b" to 2)

// Destructuring
for ((key, value) in map) {
    println("$key -> $value")
}

// forEach
map.forEach { (key, value) ->
    println("$key -> $value")
}

// Entries
map.entries.forEach { entry ->
    println("${entry.key} -> ${entry.value}")
}
```

### Q39: What are sequences in Kotlin and when should you use them?
**Answer:**
Sequences are lazily evaluated collections that process elements one at a time:

```kotlin
// Creating sequences
val seq1 = sequenceOf(1, 2, 3)
val seq2 = listOf(1, 2, 3).asSequence()
val seq3 = generateSequence(1) { it + 1 }  // Infinite!

// Sequence operations are lazy
val result = (1..1_000_000)
    .asSequence()
    .map { it * 2 }
    .filter { it % 3 == 0 }
    .take(10)
    .toList()  // Terminal operation triggers evaluation
```

**When to use sequences:**

| Use Sequences | Use Collections |
|---------------|-----------------|
| Large data sets | Small data sets |
| Multiple chained operations | Single operation |
| Only need subset of results | Need all results |
| Expensive intermediate operations | Simple operations |

```kotlin
// Sequence - efficient for large data with early exit
val firstLarge = (1..1_000_000)
    .asSequence()
    .map { it * 2 }
    .first { it > 1000 }  // Stops early

// Collection - more overhead, processes everything
val firstLargeList = (1..1_000_000)
    .map { it * 2 }  // Creates intermediate list of 1M elements
    .first { it > 1000 }
```

### Q40: How do you transform a collection by applying a function to each element in Kotlin?
**Answer:**

**Basic transformations:**
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

// map - transform each element
val doubled = numbers.map { it * 2 }  // [2, 4, 6, 8, 10]

// mapIndexed - transform with index
val indexed = numbers.mapIndexed { i, n -> "$i:$n" }
// ["0:1", "1:2", "2:3", "3:4", "4:5"]

// mapNotNull - transform and filter nulls
val results = numbers.mapNotNull { if (it > 2) it else null }
// [3, 4, 5]
```

**Filtering:**
```kotlin
val evens = numbers.filter { it % 2 == 0 }  // [2, 4]
val odds = numbers.filterNot { it % 2 == 0 }  // [1, 3, 5]
val notNull = listOf(1, null, 2).filterNotNull()  // [1, 2]
```

**Aggregation:**
```kotlin
val sum = numbers.reduce { acc, n -> acc + n }  // 15
val product = numbers.fold(1) { acc, n -> acc * n }  // 120
val sumOf = numbers.sumOf { it * 2 }  // 30
```

**Grouping and partitioning:**
```kotlin
// groupBy
val grouped = numbers.groupBy { if (it % 2 == 0) "even" else "odd" }
// {odd=[1, 3, 5], even=[2, 4]}

// partition
val (evens, odds) = numbers.partition { it % 2 == 0 }
// evens=[2, 4], odds=[1, 3, 5]
```

**Chaining transformations:**
```kotlin
val result = numbers
    .filter { it > 2 }
    .map { it * 10 }
    .sorted()
    .take(2)
// [30, 40]
```

---

## Coroutines and Concurrency

### Q41: What are coroutines in Kotlin and how do they compare to threads?
**Answer:**
Coroutines are lightweight, suspendable computations that enable asynchronous programming without blocking threads:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        delay(1000)  // Suspends, doesn't block
        println("World!")
    }
    println("Hello,")
}
// Output: Hello, World!
```

**Comparison:**

| Feature | Coroutines | Threads |
|---------|------------|---------|
| Memory | Very lightweight (~few KB) | Heavy (~1MB stack) |
| Switching | Fast (no OS involvement) | Slow (OS context switch) |
| Count | Millions possible | Limited (thousands) |
| Blocking | Suspends (non-blocking) | Blocks entire thread |
| Cancellation | Cooperative, structured | Interrupt-based |

```kotlin
// Can run millions of coroutines
runBlocking {
    repeat(100_000) {
        launch {
            delay(1000)
            print(".")
        }
    }
}

// Same with threads would crash
// repeat(100_000) { thread { Thread.sleep(1000) } }  // OutOfMemoryError
```

### Q42: How do you launch a coroutine?
**Answer:**

**Main coroutine builders:**
```kotlin
import kotlinx.coroutines.*

// 1. launch - fire and forget (returns Job)
val job = GlobalScope.launch {
    delay(1000)
    println("Launched!")
}

// 2. async - returns a result (returns Deferred<T>)
val deferred = GlobalScope.async {
    delay(1000)
    "Result"
}
val result = deferred.await()

// 3. runBlocking - blocks current thread (for main/tests)
fun main() = runBlocking {
    launch { println("Child coroutine") }
}
```

**Coroutine scopes:**
```kotlin
// GlobalScope - lives for app lifetime (avoid in production)
GlobalScope.launch { }

// CoroutineScope - structured concurrency
class MyService : CoroutineScope {
    private val job = Job()
    override val coroutineContext = Dispatchers.Default + job
    
    fun doWork() = launch {
        // work
    }
    
    fun cleanup() = job.cancel()
}

// coroutineScope - suspending scope
suspend fun loadData() = coroutineScope {
    val data1 = async { fetchData1() }
    val data2 = async { fetchData2() }
    data1.await() + data2.await()
}
```

### Q43: Explain the structure of a coroutine with a launch and async example.
**Answer:**

**launch - for side effects:**
```kotlin
fun main() = runBlocking {
    println("Start")
    
    val job = launch {
        println("Coroutine started")
        delay(1000)
        println("Coroutine completed")
    }
    
    println("Launched")
    job.join()  // Wait for completion
    println("End")
}
// Output:
// Start
// Launched
// Coroutine started
// Coroutine completed
// End
```

**async - for concurrent results:**
```kotlin
fun main() = runBlocking {
    val time = measureTimeMillis {
        val one = async { 
            delay(1000)
            13 
        }
        val two = async { 
            delay(1000)
            29 
        }
        println("Result: ${one.await() + two.await()}")
    }
    println("Completed in $time ms")  // ~1000ms, not 2000ms
}
```

**Structured concurrency:**
```kotlin
suspend fun fetchUserData(userId: Int): UserData = coroutineScope {
    // Both run concurrently
    val profile = async { fetchProfile(userId) }
    val friends = async { fetchFriends(userId) }
    
    // If either fails, the other is cancelled
    UserData(profile.await(), friends.await())
}
```

### Q44: What is a suspend function in Kotlin?
**Answer:**
A `suspend` function can pause execution without blocking and resume later:

```kotlin
// Suspend function - can only be called from coroutine or another suspend function
suspend fun fetchUser(): User {
    delay(1000)  // Suspends, doesn't block
    return User("John")
}

// Cannot call directly from regular function
// fun main() { fetchUser() }  // Error!

// Must call from coroutine
fun main() = runBlocking {
    val user = fetchUser()  // OK
}
```

**How suspension works:**
```kotlin
suspend fun example() {
    println("Before delay")
    delay(1000)  // Suspension point - execution pauses here
    println("After delay")  // Resumes after delay
}
```

**Creating suspend functions:**
```kotlin
// Wrapping callback APIs
suspend fun fetchData(): String = suspendCoroutine { continuation ->
    someCallbackApi { result ->
        continuation.resume(result)
    }
}

// With cancellation support
suspend fun fetchDataCancellable(): String = suspendCancellableCoroutine { cont ->
    val call = api.fetch()
    cont.invokeOnCancellation { call.cancel() }
    call.enqueue { result ->
        cont.resume(result)
    }
}
```

### Q45: Explain the context of coroutines and how do you manage it.
**Answer:**
Coroutine context is a set of elements that define coroutine behavior:

```kotlin
// Context elements
val context = Dispatchers.Default +  // Dispatcher
              CoroutineName("MyCoroutine") +  // Name
              Job()  // Job for cancellation

launch(context) {
    println("Running in ${coroutineContext[CoroutineName]}")
}
```

**Dispatchers:**
```kotlin
launch(Dispatchers.Main) {
    // Main thread (UI)
}

launch(Dispatchers.IO) {
    // Background I/O operations
}

launch(Dispatchers.Default) {
    // CPU-intensive work
}

launch(Dispatchers.Unconfined) {
    // Starts in caller thread, resumes in suspending thread
}
```

**Switching context:**
```kotlin
suspend fun fetchAndDisplay() {
    val data = withContext(Dispatchers.IO) {
        fetchData()  // Runs on IO dispatcher
    }
    withContext(Dispatchers.Main) {
        displayData(data)  // Runs on Main dispatcher
    }
}
```

**Custom dispatcher:**
```kotlin
val customDispatcher = Executors.newFixedThreadPool(4).asCoroutineDispatcher()
launch(customDispatcher) {
    // Runs on custom thread pool
}
customDispatcher.close()  // Don't forget to close
```

### Q46: What are coroutine scopes and why are they important?
**Answer:**
Coroutine scopes provide structured concurrency - managing coroutine lifecycles:

```kotlin
// CoroutineScope interface
class MyService : CoroutineScope {
    private val job = SupervisorJob()
    override val coroutineContext = Dispatchers.Default + job
    
    fun doWork() {
        launch {
            // Child coroutine
        }
    }
    
    fun cleanup() {
        job.cancel()  // Cancels all children
    }
}
```

**Built-in scopes:**
```kotlin
// GlobalScope - application lifetime (avoid)
GlobalScope.launch { }

// runBlocking - blocks thread (tests/main)
runBlocking { }

// coroutineScope - suspends, doesn't block
suspend fun work() = coroutineScope {
    launch { }
}

// supervisorScope - children failures don't affect siblings
suspend fun work() = supervisorScope {
    launch { throw Exception() }  // Doesn't cancel siblings
    launch { println("Still running") }
}
```

**Android scopes:**
```kotlin
// ViewModel scope - cancelled when ViewModel cleared
class MyViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {
            val data = fetchData()
            _state.value = data
        }
    }
}

// Lifecycle scope - cancelled when lifecycle destroyed
class MyFragment : Fragment() {
    fun loadData() {
        lifecycleScope.launch {
            val data = fetchData()
            updateUI(data)
        }
    }
}
```

### Q47: How do you cancel a coroutine and handle exceptions in coroutines?
**Answer:**

**Cancellation:**
```kotlin
val job = launch {
    repeat(1000) { i ->
        println("Job: $i")
        delay(500)  // Checks for cancellation
    }
}

delay(1300)
job.cancel()  // Cancel the coroutine
job.join()    // Wait for cancellation to complete
// Or: job.cancelAndJoin()
```

**Cooperative cancellation:**
```kotlin
val job = launch {
    while (isActive) {  // Check cancellation status
        // CPU-intensive work
    }
}

// Or use yield()
val job = launch {
    while (true) {
        yield()  // Suspends and checks for cancellation
        // work
    }
}
```

**Exception handling:**
```kotlin
// CoroutineExceptionHandler
val handler = CoroutineExceptionHandler { _, exception ->
    println("Caught: $exception")
}

GlobalScope.launch(handler) {
    throw RuntimeException("Error!")
}

// try-catch within coroutine
launch {
    try {
        riskyOperation()
    } catch (e: Exception) {
        println("Caught: $e")
    }
}

// SupervisorJob for independent children
val supervisor = SupervisorJob()
val scope = CoroutineScope(supervisor)

scope.launch {
    throw Exception()  // Doesn't affect sibling
}
scope.launch {
    println("Still running")
}
```

### Q48: Can coroutines be used on any thread or are there restrictions?
**Answer:**
Coroutines can run on any thread, controlled by Dispatchers:

```kotlin
// Main thread (UI)
launch(Dispatchers.Main) {
    // Must be used for UI updates on Android
    textView.text = "Updated"
}

// Background threads
launch(Dispatchers.IO) {
    // File/network operations
    val data = readFile()
}

launch(Dispatchers.Default) {
    // CPU-intensive (number of threads = CPU cores)
    val result = calculatePrimes()
}

// Unconfined - starts in caller, resumes anywhere
launch(Dispatchers.Unconfined) {
    println("Started in ${Thread.currentThread().name}")
    delay(100)
    println("Resumed in ${Thread.currentThread().name}")  // Different thread!
}
```

**Thread confinement:**
```kotlin
// Single-threaded context
val singleThread = newSingleThreadContext("MyThread")
launch(singleThread) {
    // Always runs on same thread
}
singleThread.close()

// Main-safe functions
suspend fun fetchAndUpdate() {
    val data = withContext(Dispatchers.IO) {
        fetchData()  // Background
    }
    // Back on original dispatcher (Main if called from Main)
    updateUI(data)
}
```

---

## Java Interoperability

### Q49: How is Kotlin-Java interoperability achieved?
**Answer:**
Kotlin compiles to JVM bytecode, enabling seamless interop:

**Calling Java from Kotlin:**
```kotlin
// Java class
// public class JavaHelper {
//     public static String format(String text) { return text.toUpperCase(); }
//     public void process(List<String> items) { ... }
// }

// Kotlin usage
val result = JavaHelper.format("hello")
val helper = JavaHelper()
helper.process(listOf("a", "b"))
```

**Kotlin features in Java:**
```kotlin
// Kotlin file: Utils.kt
package com.example

fun greet(name: String) = "Hello, $name"
val version = "1.0"

// Java usage
// String greeting = UtilsKt.greet("World");
// String ver = UtilsKt.getVersion();
```

**Handling platform types:**
```kotlin
// Java returns platform type (String! - unknown nullability)
val javaString = JavaClass.getString()  // Type is String!

// Explicitly handle nullability
val safe: String = javaString ?: "default"
val nullable: String? = javaString
```

### Q50: Can you call Kotlin code from Java?
**Answer:**
Yes, with some considerations:

```kotlin
// Kotlin file: UserService.kt
class UserService {
    fun getUser(id: Int): User = User("John")
    
    companion object {
        @JvmStatic
        fun create(): UserService = UserService()
    }
}

fun topLevelFunction() = "Hello"
val topLevelProperty = "World"
```

**Java usage:**
```java
// Class methods
UserService service = new UserService();
User user = service.getUser(1);

// Companion object
UserService service2 = UserService.Companion.create();
// Or with @JvmStatic:
UserService service3 = UserService.create();

// Top-level functions (file: UserServiceKt)
String result = UserServiceKt.topLevelFunction();
String prop = UserServiceKt.getTopLevelProperty();
```

### Q51: Can Java annotations be used in Kotlin? How?
**Answer:**
Yes, Java annotations work in Kotlin with special syntax for some cases:

```kotlin
// Standard Java annotations
@Override
@Deprecated("Use newMethod instead")
@SuppressWarnings("unchecked")
class Example

// Annotation with array parameter
@RequestMapping(value = ["path1", "path2"])
fun endpoint() { }

// Use-site targets
class Example(
    @field:JsonProperty("user_name") // Apply to Java field
    @get:JsonProperty("user_name")   // Apply to getter
    @param:JsonProperty("user_name") // Apply to constructor parameter
    val userName: String
)

// File-level annotations
@file:JvmName("Utils")
package com.example
```

**Common annotation targets:**
| Target | Applies to |
|--------|-----------|
| `@field:` | Java field |
| `@get:` | Property getter |
| `@set:` | Property setter |
| `@param:` | Constructor parameter |
| `@property:` | Kotlin property |
| `@receiver:` | Extension receiver |
| `@file:` | File class |

### Q52: What is the @JvmStatic, @JvmOverloads, @JvmField, and @JvmName annotation and when do you use them?
**Answer:**

**@JvmStatic - generate static method:**
```kotlin
class Utils {
    companion object {
        @JvmStatic
        fun helper() = "Static-like"
    }
}

// Java: Utils.helper()  // Works like static
// Without @JvmStatic: Utils.Companion.helper()
```

**@JvmOverloads - generate overloaded methods:**
```kotlin
@JvmOverloads
fun greet(name: String, greeting: String = "Hello", punct: String = "!") {
    println("$greeting, $name$punct")
}

// Generates for Java:
// void greet(String name, String greeting, String punct)
// void greet(String name, String greeting)
// void greet(String name)
```

**@JvmField - expose as field:**
```kotlin
class Config {
    @JvmField
    val VERSION = "1.0"
    
    companion object {
        @JvmField
        val INSTANCE = Config()
    }
}

// Java: String v = config.VERSION;  // Direct field access
// Without: String v = config.getVERSION();  // Getter access
```

**@JvmName - rename for Java:**
```kotlin
@file:JvmName("StringUtils")
package utils

@JvmName("isEmptyString")
fun String.isEmpty() = this.length == 0

// Java: StringUtils.isEmptyString(str)
```

### Q53: How do you use Java Streams in Kotlin?
**Answer:**
You can use Java Streams, but Kotlin collections are often preferred:

```kotlin
import java.util.stream.Collectors

// Using Java Streams in Kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val result = numbers.stream()
    .filter { it > 2 }
    .map { it * 2 }
    .collect(Collectors.toList())

// Equivalent Kotlin (preferred)
val kotlinResult = numbers
    .filter { it > 2 }
    .map { it * 2 }
```

**When to use Streams vs Kotlin collections:**

| Use Java Streams | Use Kotlin Collections |
|-----------------|----------------------|
| Existing Java codebase | New Kotlin code |
| Parallel processing needed | Sequential processing |
| Interop with Java APIs | Pure Kotlin project |

```kotlin
// Parallel streams
val parallelResult = numbers.parallelStream()
    .filter { it > 2 }
    .map { expensiveOperation(it) }
    .collect(Collectors.toList())

// Kotlin sequences for lazy evaluation
val sequenceResult = numbers.asSequence()
    .filter { it > 2 }
    .map { it * 2 }
    .toList()
```


---

## Advanced Topics

### Q54: What is the role of delegation in Kotlin?
**Answer:**
Delegation is a design pattern where an object handles requests by delegating to another object:

**Class delegation:**
```kotlin
interface Printer {
    fun print(message: String)
}

class ConsolePrinter : Printer {
    override fun print(message: String) = println(message)
}

// Delegate implementation to another object
class LoggingPrinter(private val printer: Printer) : Printer by printer {
    override fun print(message: String) {
        println("[LOG] Printing...")
        printer.print(message)  // Can also use delegated method
    }
}
```

**Property delegation:**
```kotlin
class Example {
    // lazy - computed on first access
    val lazyValue: String by lazy {
        println("Computing...")
        "Hello"
    }
    
    // observable - callback on change
    var observed: String by Delegates.observable("initial") { _, old, new ->
        println("Changed from $old to $new")
    }
    
    // vetoable - can reject changes
    var vetoed: Int by Delegates.vetoable(0) { _, _, new ->
        new >= 0  // Only accept non-negative
    }
    
    // Map delegation
    val map = mapOf("name" to "John", "age" to 30)
    val name: String by map
    val age: Int by map
}
```

**Custom delegate:**
```kotlin
class Trimmed : ReadWriteProperty<Any?, String> {
    private var value: String = ""
    
    override fun getValue(thisRef: Any?, property: KProperty<*>) = value
    override fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        this.value = value.trim()
    }
}

class Form {
    var name: String by Trimmed()
}
```

### Q55: How do you manage dependency injection in Kotlin?
**Answer:**

**Manual DI:**
```kotlin
// Constructor injection
class UserRepository(private val database: Database)

class UserService(private val repository: UserRepository)

// Factory pattern
object ServiceFactory {
    fun createUserService(): UserService {
        val db = Database()
        val repo = UserRepository(db)
        return UserService(repo)
    }
}
```

**Koin (popular Kotlin DI):**
```kotlin
// Define modules
val appModule = module {
    single { Database() }
    single { UserRepository(get()) }
    factory { UserService(get()) }
}

// Start Koin
startKoin {
    modules(appModule)
}

// Inject
class MyActivity : AppCompatActivity() {
    private val userService: UserService by inject()
}
```

**Hilt/Dagger (Android):**
```kotlin
@HiltAndroidApp
class MyApplication : Application()

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject lateinit var userService: UserService
}

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideUserService(repo: UserRepository): UserService {
        return UserService(repo)
    }
}
```

### Q56: What is type aliasing in Kotlin and why would you use it?
**Answer:**
Type aliases provide alternative names for existing types:

```kotlin
// Simplify complex types
typealias UserMap = Map<String, List<User>>
typealias Callback = (String, Int) -> Boolean

fun process(users: UserMap, callback: Callback) {
    // Much more readable than:
    // fun process(users: Map<String, List<User>>, callback: (String, Int) -> Boolean)
}

// Clarify intent
typealias UserId = Int
typealias Username = String

fun findUser(id: UserId): Username? = // ...

// Function types
typealias ClickHandler = (View) -> Unit
typealias Predicate<T> = (T) -> Boolean

// Generic type aliases
typealias StringList = List<String>
typealias NodeSet = Set<Network.Node>
```

**Limitations:**
```kotlin
typealias UserId = Int

val id: UserId = 123
val regularInt: Int = id  // OK - same type at runtime

// Type aliases don't create new types
fun process(id: UserId) { }
fun process(id: Int) { }  // Error: same signature
```

### Q57: How are generics handled in Kotlin compared to Java?
**Answer:**

**Declaration-site variance:**
```kotlin
// Kotlin: variance declared at class level
interface Source<out T> {  // Covariant (producer)
    fun get(): T
}

interface Sink<in T> {  // Contravariant (consumer)
    fun put(item: T)
}

// Java: use-site variance only
// interface Source<T> { T get(); }
// Source<? extends Number> source;
```

**Reified type parameters (with inline):**
```kotlin
// Kotlin: access type at runtime
inline fun <reified T> isInstance(value: Any): Boolean {
    return value is T
}

inline fun <reified T> parseJson(json: String): T {
    return Gson().fromJson(json, T::class.java)
}

// Java: type erasure, cannot do this without passing Class<T>
```

**Star projection (wildcard equivalent):**
```kotlin
// Kotlin
fun processAny(list: List<*>) {  // Unknown type
    val item: Any? = list[0]  // Returns Any?
}

// Java equivalent
// void processAny(List<?> list)
```

**Comparison:**

| Feature | Kotlin | Java |
|---------|--------|------|
| Variance | Declaration-site (`out`/`in`) | Use-site (`? extends`/`? super`) |
| Reified | Supported (inline) | Not supported |
| Wildcard | Star projection (`*`) | `?` |
| Type bounds | `where T : A, T : B` | `<T extends A & B>` |

### Q58: What is the difference between a vararg and an array in Kotlin?
**Answer:**

**vararg - variable arguments:**
```kotlin
fun printAll(vararg messages: String) {
    for (msg in messages) {
        println(msg)
    }
}

// Call with multiple arguments
printAll("Hello", "World", "!")

// Pass array using spread operator
val array = arrayOf("a", "b", "c")
printAll(*array)

// Mix spread with regular args
printAll("Start", *array, "End")
```

**Array - fixed-size collection:**
```kotlin
val array: Array<String> = arrayOf("a", "b", "c")
val intArray: IntArray = intArrayOf(1, 2, 3)  // Primitive array

// Array creation
val nulls: Array<String?> = arrayOfNulls(5)
val generated = Array(5) { i -> "Item $i" }
```

**Key differences:**

| Feature | vararg | Array |
|---------|--------|-------|
| Declaration | Parameter modifier | Type |
| Usage | Multiple args at call site | Single array argument |
| Null elements | Based on type | Based on type |
| Spread | Required to pass array | N/A |
| Position | Usually last parameter | Anywhere |

```kotlin
// vararg must be last (or use named params)
fun example(vararg items: String, last: Int)  // OK with named
example("a", "b", last = 1)

// Only one vararg allowed
// fun invalid(vararg a: Int, vararg b: String)  // Error
```

### Q59: Explain destructuring declarations in Kotlin.
**Answer:**
Destructuring breaks objects into multiple variables:

```kotlin
// Data class destructuring
data class Person(val name: String, val age: Int)
val person = Person("John", 30)
val (name, age) = person  // Destructures into two variables

// Pair and Triple
val pair = Pair("key", 123)
val (key, value) = pair

val triple = Triple(1, 2, 3)
val (a, b, c) = triple
```

**In loops:**
```kotlin
val map = mapOf("a" to 1, "b" to 2)
for ((key, value) in map) {
    println("$key -> $value")
}

val list = listOf(Person("Alice", 25), Person("Bob", 30))
for ((name, age) in list) {
    println("$name is $age")
}
```

**In lambdas:**
```kotlin
map.forEach { (key, value) ->
    println("$key: $value")
}

list.filter { (_, age) -> age > 25 }  // Underscore to ignore
```

**Custom destructuring:**
```kotlin
class Point(val x: Int, val y: Int) {
    operator fun component1() = x
    operator fun component2() = y
}

val point = Point(10, 20)
val (x, y) = point
```

### Q60: How do you create and use an inline class in Kotlin?
**Answer:**
Inline classes (now called value classes) wrap a value without runtime overhead:

```kotlin
@JvmInline
value class UserId(val id: Int)

@JvmInline
value class Email(val value: String) {
    init {
        require(value.contains("@")) { "Invalid email" }
    }
    
    val domain: String
        get() = value.substringAfter("@")
}

// Usage
fun findUser(id: UserId): User? = // ...
fun sendEmail(email: Email) = // ...

val userId = UserId(123)
val email = Email("user@example.com")

// At runtime, these are just Int and String (no wrapper object)
```

**Benefits:**
- Type safety without allocation overhead
- Prevents mixing up same primitive types
- Can have properties and functions

**Restrictions:**
- Only one property in primary constructor
- Cannot have backing fields
- Cannot participate in class hierarchy

```kotlin
// Type safety example
@JvmInline value class Meters(val value: Double)
@JvmInline value class Feet(val value: Double)

fun travel(distance: Meters) { }

val m = Meters(100.0)
val f = Feet(100.0)

travel(m)  // OK
// travel(f)  // Compile error! Different types
```

---

## Android Specific

### Q61: How do you ensure non-null view properties in Android with Kotlin?
**Answer:**

**ViewBinding (recommended):**
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // Non-null view access
        binding.textView.text = "Hello"
        binding.button.setOnClickListener { }
    }
}
```

**Lazy initialization:**
```kotlin
class MyFragment : Fragment(R.layout.fragment_my) {
    private val textView: TextView by lazy {
        requireView().findViewById(R.id.textView)
    }
}
```

**lateinit:**
```kotlin
class MyActivity : AppCompatActivity() {
    private lateinit var textView: TextView
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        textView = findViewById(R.id.textView)
    }
}
```

**Avoid:**
```kotlin
// Don't use !! on views
val text = findViewById<TextView>(R.id.textView)!!  // Risky

// Better: use safe call or binding
val text = findViewById<TextView>(R.id.textView)
text?.text = "Safe"
```

### Q62: What are the benefits of using Kotlin in Android development?
**Answer:**

| Benefit | Example |
|---------|---------|
| **Null Safety** | Prevents NPE at compile time |
| **Concise Syntax** | Data classes, string templates |
| **Coroutines** | Async without callbacks |
| **Extension Functions** | Cleaner View manipulation |
| **DSL Support** | Anko, Jetpack Compose |

```kotlin
// 1. Null safety
val name: String? = intent.getStringExtra("name")
textView.text = name ?: "Guest"

// 2. Concise data classes
data class User(val id: Int, val name: String)

// 3. Coroutines for async
lifecycleScope.launch {
    val user = fetchUser()
    updateUI(user)
}

// 4. Extension functions
fun View.show() { visibility = View.VISIBLE }
fun View.hide() { visibility = View.GONE }

// 5. Apply for configuration
val textView = TextView(context).apply {
    text = "Hello"
    textSize = 16f
    setTextColor(Color.BLACK)
}
```

### Q63: Explain how to use Kotlin Android Extensions.
**Answer:**
**Note:** Kotlin Android Extensions is deprecated. Use ViewBinding instead.

```kotlin
// OLD (deprecated):
// import kotlinx.android.synthetic.main.activity_main.*
// textView.text = "Hello"

// NEW (ViewBinding):
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        binding.textView.text = "Hello"
    }
}

// Enable in build.gradle
// android {
//     buildFeatures {
//         viewBinding true
//     }
// }
```

### Q64: How do you handle configuration changes in Android using Kotlin?
**Answer:**

**ViewModel (recommended):**
```kotlin
class MainViewModel : ViewModel() {
    private val _data = MutableLiveData<String>()
    val data: LiveData<String> = _data
    
    fun loadData() {
        viewModelScope.launch {
            _data.value = repository.fetchData()
        }
    }
}

class MainActivity : AppCompatActivity() {
    private val viewModel: MainViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        viewModel.data.observe(this) { data ->
            textView.text = data
        }
        
        if (savedInstanceState == null) {
            viewModel.loadData()
        }
    }
}
```

**SavedStateHandle:**
```kotlin
class MyViewModel(private val state: SavedStateHandle) : ViewModel() {
    var searchQuery: String?
        get() = state["query"]
        set(value) { state["query"] = value }
}
```

### Q65: What is an Android LiveData and how do you use it in Kotlin?
**Answer:**
LiveData is a lifecycle-aware observable data holder:

```kotlin
class UserViewModel : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user
    
    fun updateUser(user: User) {
        _user.value = user  // Main thread
        // _user.postValue(user)  // Background thread
    }
}

// Observing in Activity/Fragment
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        viewModel.user.observe(this) { user ->
            // Update UI - only called when lifecycle is STARTED/RESUMED
            nameTextView.text = user.name
        }
    }
}
```

**Transformations:**
```kotlin
val userName: LiveData<String> = Transformations.map(user) { 
    it.name.uppercase()
}

val userDetails: LiveData<Details> = Transformations.switchMap(userId) { id ->
    repository.getUserDetails(id)
}
```

### Q66: How does Kotlin improve working with Android's concurrency APIs like AsyncTask?
**Answer:**
Coroutines replace AsyncTask with cleaner, safer code:

```kotlin
// OLD: AsyncTask (deprecated)
// class FetchTask : AsyncTask<Void, Void, String>() {
//     override fun doInBackground(vararg params: Void?): String {
//         return fetchData()
//     }
//     override fun onPostExecute(result: String) {
//         textView.text = result
//     }
// }

// NEW: Coroutines
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            val data = withContext(Dispatchers.IO) {
                fetchData()  // Background
            }
            textView.text = data  // Main thread
        }
    }
}

// In ViewModel
class MyViewModel : ViewModel() {
    fun loadData() = viewModelScope.launch {
        try {
            _loading.value = true
            val result = withContext(Dispatchers.IO) {
                repository.fetchData()
            }
            _data.value = result
        } catch (e: Exception) {
            _error.value = e.message
        } finally {
            _loading.value = false
        }
    }
}
```

### Q67: What is the purpose of coroutineScope or lifecycleScope in Android with Kotlin?
**Answer:**

**lifecycleScope - tied to Activity/Fragment lifecycle:**
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Cancelled when activity is destroyed
        lifecycleScope.launch {
            val data = fetchData()
            updateUI(data)
        }
        
        // Launch when lifecycle reaches state
        lifecycleScope.launchWhenStarted {
            // Only runs when STARTED
        }
        
        // Repeat on lifecycle
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                flow.collect { updateUI(it) }
            }
        }
    }
}
```

**viewModelScope - tied to ViewModel lifecycle:**
```kotlin
class MyViewModel : ViewModel() {
    init {
        // Cancelled when ViewModel is cleared
        viewModelScope.launch {
            loadInitialData()
        }
    }
    
    fun refresh() = viewModelScope.launch {
        _data.value = repository.fetchData()
    }
}
```

**Comparison:**

| Scope | Lifecycle | Use Case |
|-------|-----------|----------|
| `lifecycleScope` | Activity/Fragment | UI operations |
| `viewModelScope` | ViewModel | Data operations |
| `GlobalScope` | Application | Avoid in Android |

---

## DSL and Meta-Programming

### Q68: What is a domain-specific language (DSL) in Kotlin, and how would you create one?
**Answer:**
A DSL is a specialized language for a specific domain. Kotlin's features enable clean DSL creation:

```kotlin
// HTML DSL example
fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}

class HTML {
    private val children = mutableListOf<Element>()
    
    fun head(init: Head.() -> Unit) {
        children.add(Head().apply(init))
    }
    
    fun body(init: Body.() -> Unit) {
        children.add(Body().apply(init))
    }
}

// Usage
val page = html {
    head {
        title("My Page")
    }
    body {
        h1("Welcome")
        p("This is a paragraph")
    }
}
```

**Gradle Kotlin DSL:**
```kotlin
// build.gradle.kts
plugins {
    kotlin("jvm") version "1.9.0"
}

dependencies {
    implementation("org.jetbrains.kotlin:kotlin-stdlib")
    testImplementation("junit:junit:4.13.2")
}
```

### Q69: How do Kotlin extension functions facilitate DSL creation?
**Answer:**
Extension functions add methods to existing types, enabling fluent APIs:

```kotlin
// String builder DSL
fun buildString(action: StringBuilder.() -> Unit): String {
    return StringBuilder().apply(action).toString()
}

val result = buildString {
    append("Hello")
    append(" ")
    append("World")
}

// Route DSL
class Router {
    fun get(path: String, handler: () -> String) { }
    fun post(path: String, handler: () -> String) { }
}

fun routing(init: Router.() -> Unit): Router {
    return Router().apply(init)
}

val routes = routing {
    get("/users") { "List users" }
    post("/users") { "Create user" }
}
```

### Q70: What are the common use-cases for Kotlin's reflective capabilities?
**Answer:**

```kotlin
import kotlin.reflect.full.*

// Class reflection
val kClass = User::class
println(kClass.simpleName)  // "User"
println(kClass.members)     // Properties and functions

// Property reflection
data class User(val name: String, var age: Int)
val user = User("John", 30)

val nameProperty = User::name
println(nameProperty.get(user))  // "John"

val ageProperty = User::age
ageProperty.set(user, 31)

// Function reflection
fun greet(name: String) = "Hello, $name"
val func = ::greet
println(func("World"))  // "Hello, World"

// Annotations
@Target(AnnotationTarget.PROPERTY)
annotation class Validate(val regex: String)

data class Form(@Validate("[a-z]+") val field: String)

Form::class.memberProperties.forEach { prop ->
    prop.findAnnotation<Validate>()?.let {
        println("${prop.name} validates with ${it.regex}")
    }
}
```

**Use cases:**
- Dependency injection frameworks
- Serialization/deserialization
- ORM mapping
- Testing frameworks

### Q71: How does Kotlin's apply, let, run, with, and also improve DSL writing?
**Answer:**

```kotlin
// apply - configure object, return it
val person = Person().apply {
    name = "John"    // this.name
    age = 30         // this.age
}

// let - transform, return result
val length = name?.let {
    println("Name: $it")
    it.length
}

// run - execute block, return result
val result = person.run {
    "$name is $age years old"  // Access with this
}

// with - same as run, different syntax
val description = with(person) {
    "$name is $age years old"
}

// also - side effects, return original
val numbers = mutableListOf(1, 2, 3).also {
    println("Created list: $it")
}
```

**DSL example combining scope functions:**
```kotlin
class Configuration {
    var host: String = ""
    var port: Int = 0
    var database: DatabaseConfig? = null
    
    fun database(init: DatabaseConfig.() -> Unit) {
        database = DatabaseConfig().apply(init)
    }
}

class DatabaseConfig {
    var name: String = ""
    var user: String = ""
}

fun config(init: Configuration.() -> Unit) = Configuration().apply(init)

// DSL usage
val cfg = config {
    host = "localhost"
    port = 8080
    database {
        name = "mydb"
        user = "admin"
    }
}
```

---

## Multiplatform Development

### Q72: What is Kotlin Multiplatform Mobile (KMM)?
**Answer:**
KMM enables sharing code between Android and iOS applications:

```
project/
├── shared/
│   ├── commonMain/    # Shared Kotlin code
│   ├── androidMain/   # Android-specific
│   └── iosMain/       # iOS-specific
├── androidApp/        # Android application
└── iosApp/            # iOS application
```

**Shared code example:**
```kotlin
// commonMain - shared business logic
expect class Platform() {
    val name: String
}

class Greeting {
    fun greet(): String = "Hello, ${Platform().name}!"
}

// androidMain
actual class Platform actual constructor() {
    actual val name: String = "Android ${android.os.Build.VERSION.SDK_INT}"
}

// iosMain
actual class Platform actual constructor() {
    actual val name: String = UIDevice.currentDevice.systemName()
}
```

### Q73: Explain how Kotlin/Native works and what it offers for cross-platform development.
**Answer:**
Kotlin/Native compiles Kotlin to native binaries without JVM:

**Targets:**
- iOS (arm64, x64)
- macOS
- Linux
- Windows
- WebAssembly

```kotlin
// Native-specific code
fun main() {
    // Direct memory access
    memScoped {
        val buffer = allocArray<ByteVar>(1024)
        // Use native memory
    }
}

// Interop with C
// cinterop generates Kotlin bindings for C libraries
```

**Benefits:**
- No JVM required
- Native performance
- iOS deployment
- Embedded systems

### Q74: How can shared business logic be developed with Kotlin Multiplatform?
**Answer:**

```kotlin
// commonMain/kotlin/SharedRepository.kt
class UserRepository(private val api: UserApi, private val db: UserDatabase) {
    suspend fun getUser(id: String): User {
        return db.getUser(id) ?: api.fetchUser(id).also {
            db.saveUser(it)
        }
    }
}

// commonMain/kotlin/UserApi.kt
expect class UserApi() {
    suspend fun fetchUser(id: String): User
}

// androidMain/kotlin/UserApi.kt
actual class UserApi actual constructor() {
    actual suspend fun fetchUser(id: String): User {
        return retrofitService.getUser(id)
    }
}

// iosMain/kotlin/UserApi.kt
actual class UserApi actual constructor() {
    actual suspend fun fetchUser(id: String): User {
        // Use iOS networking
    }
}
```

### Q75: What are the limitations of Kotlin Multiplatform?
**Answer:**

| Limitation | Description |
|------------|-------------|
| **iOS Debugging** | Limited compared to Android |
| **Library Ecosystem** | Fewer multiplatform libraries |
| **Concurrency** | Different models (coroutines vs GCD) |
| **Binary Size** | iOS binaries can be large |
| **Learning Curve** | Need to understand expect/actual |

```kotlin
// Platform differences require abstractions
expect fun platformLog(message: String)

// May need different implementations
expect class DateTime {
    fun now(): Long
}

// Some APIs not available cross-platform
// File I/O, networking, UI - need platform-specific code
```

### Q76: Discuss the build system and tooling support for Kotlin Multiplatform projects.
**Answer:**

**Gradle configuration:**
```kotlin
// build.gradle.kts
plugins {
    kotlin("multiplatform") version "1.9.0"
    id("com.android.library")
}

kotlin {
    android()
    ios()
    
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.0")
            }
        }
        val androidMain by getting
        val iosMain by getting
    }
}
```

**IDE Support:**
- Android Studio with KMM plugin
- Xcode for iOS
- Fleet (JetBrains) for full KMM support

**Testing:**
```kotlin
// commonTest
class SharedTest {
    @Test
    fun testGreeting() {
        val greeting = Greeting()
        assertTrue(greeting.greet().contains("Hello"))
    }
}
```

---

## Testing and Tooling

### Q77: What testing frameworks are available for Kotlin?
**Answer:**

**JUnit (most common):**
```kotlin
import org.junit.Test
import org.junit.Assert.*

class CalculatorTest {
    @Test
    fun `addition works correctly`() {
        assertEquals(4, Calculator.add(2, 2))
    }
}
```

**Kotest (Kotlin-native):**
```kotlin
class MySpec : StringSpec({
    "strings should concatenate" {
        "hello" + " world" shouldBe "hello world"
    }
    
    "lists should contain element" {
        listOf(1, 2, 3) shouldContain 2
    }
})
```

**MockK (Kotlin mocking):**
```kotlin
@Test
fun `test with mock`() {
    val repository = mockk<UserRepository>()
    every { repository.getUser(1) } returns User("John")
    
    val service = UserService(repository)
    assertEquals("John", service.getUserName(1))
    
    verify { repository.getUser(1) }
}
```

### Q78: How do you mock dependencies in Kotlin unit tests?
**Answer:**

**MockK:**
```kotlin
// Create mock
val mock = mockk<Repository>()

// Define behavior
every { mock.getData() } returns "data"
every { mock.saveData(any()) } just Runs
coEvery { mock.fetchAsync() } returns "async data"

// Verify calls
verify { mock.getData() }
verify(exactly = 2) { mock.saveData(any()) }

// Capture arguments
val slot = slot<String>()
every { mock.saveData(capture(slot)) } just Runs
mock.saveData("test")
assertEquals("test", slot.captured)
```

**Relaxed mocks:**
```kotlin
val relaxedMock = mockk<Service>(relaxed = true)
// All methods return default values
```

**Spy (partial mock):**
```kotlin
val spy = spyk(RealClass())
every { spy.specificMethod() } returns "mocked"
// Other methods call real implementation
```

### Q79: What are some of the best practices for writing testable Kotlin code?
**Answer:**

```kotlin
// 1. Constructor injection
class UserService(
    private val repository: UserRepository,
    private val validator: Validator
) {
    fun createUser(user: User): Result<User> {
        if (!validator.isValid(user)) return Result.failure(ValidationError())
        return Result.success(repository.save(user))
    }
}

// 2. Interface-based design
interface UserRepository {
    fun save(user: User): User
    fun find(id: Int): User?
}

// 3. Pure functions
fun calculateTotal(items: List<Item>): Double {
    return items.sumOf { it.price * it.quantity }
}

// 4. Avoid static/global state
// Bad: object with mutable state
// Good: injectable dependencies

// 5. Small, focused functions
fun User.isEligible(): Boolean = age >= 18 && isActive
```

### Q80: How does the Kotlin REPL work and what can it be used for?
**Answer:**
REPL (Read-Eval-Print Loop) allows interactive Kotlin execution:

```bash
# Start REPL
$ kotlinc

>>> val x = 10
>>> println(x * 2)
20

>>> fun greet(name: String) = "Hello, $name"
>>> greet("World")
res1: kotlin.String = Hello, World

>>> data class User(val name: String)
>>> User("John")
res2: User = User(name=John)
```

**Use cases:**
- Learning Kotlin syntax
- Quick prototyping
- Testing code snippets
- Exploring APIs

**In IntelliJ:**
- Tools → Kotlin → Kotlin REPL
- Scratch files for more complex experiments

---

## Asynchronous Flow

### Q81: What is a Flow in Kotlin and how does it differ from a coroutine?
**Answer:**
Flow is a cold asynchronous stream that emits multiple values:

```kotlin
// Flow - emits multiple values over time
fun numbersFlow(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

// Collecting the flow
runBlocking {
    numbersFlow().collect { value ->
        println(value)
    }
}
```

**Differences:**

| Feature | Flow | Coroutine (suspend) |
|---------|------|-------------------|
| Values | Multiple | Single |
| Execution | Cold (on collect) | Eager (on call) |
| Cancellation | On collector cancel | On scope cancel |
| Use case | Streams, events | Single async result |

```kotlin
// Suspend function - single value
suspend fun fetchUser(): User = api.getUser()

// Flow - multiple values
fun observeUsers(): Flow<User> = flow {
    while (true) {
        emit(api.getUser())
        delay(1000)
    }
}
```

### Q82: How would you handle backpressure in Kotlin?
**Answer:**
Backpressure occurs when producer is faster than consumer:

```kotlin
// buffer - buffer emissions
flow
    .buffer(capacity = 64)
    .collect { process(it) }

// conflate - skip intermediate values
flow
    .conflate()
    .collect { processLatest(it) }

// collectLatest - cancel previous collection
flow.collectLatest { value ->
    // Cancels if new value arrives before completion
    heavyProcessing(value)
}

// Sample/debounce
flow
    .sample(100)  // Emit every 100ms
    .collect { }

flow
    .debounce(300)  // Wait for 300ms silence
    .collect { }
```

### Q83: Show an example of a cold flow versus hot channels in Kotlin.
**Answer:**

**Cold Flow (starts fresh for each collector):**
```kotlin
val coldFlow = flow {
    println("Flow started")
    emit(1)
    emit(2)
}

// Each collect starts the flow from beginning
coldFlow.collect { println(it) }  // Prints: Flow started, 1, 2
coldFlow.collect { println(it) }  // Prints: Flow started, 1, 2
```

**Hot SharedFlow/StateFlow:**
```kotlin
// SharedFlow - hot stream
val sharedFlow = MutableSharedFlow<Int>()

// Collectors receive only new emissions
launch { sharedFlow.collect { println("A: $it") } }
launch { sharedFlow.collect { println("B: $it") } }

sharedFlow.emit(1)  // Both receive
sharedFlow.emit(2)  // Both receive

// StateFlow - always has current value
val stateFlow = MutableStateFlow(0)
println(stateFlow.value)  // 0

launch { stateFlow.collect { println(it) } }
stateFlow.value = 1  // Collector receives
```

### Q84: How do you convert a callback-based API to Kotlin suspend function?
**Answer:**

```kotlin
// Callback-based API
interface Callback {
    fun onSuccess(result: String)
    fun onError(error: Exception)
}

fun fetchData(callback: Callback)

// Convert to suspend function
suspend fun fetchDataSuspend(): String = suspendCoroutine { continuation ->
    fetchData(object : Callback {
        override fun onSuccess(result: String) {
            continuation.resume(result)
        }
        override fun onError(error: Exception) {
            continuation.resumeWithException(error)
        }
    })
}

// With cancellation support
suspend fun fetchDataCancellable(): String = suspendCancellableCoroutine { cont ->
    val call = api.fetchData(object : Callback {
        override fun onSuccess(result: String) {
            cont.resume(result)
        }
        override fun onError(error: Exception) {
            cont.resumeWithException(error)
        }
    })
    
    cont.invokeOnCancellation {
        call.cancel()
    }
}
```

---

## Project and Environment Setup

### Q85: How do you set up a Kotlin project using Gradle?
**Answer:**

**build.gradle.kts:**
```kotlin
plugins {
    kotlin("jvm") version "1.9.0"
    application
}

group = "com.example"
version = "1.0-SNAPSHOT"

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.0")
    testImplementation(kotlin("test"))
}

tasks.test {
    useJUnitPlatform()
}

kotlin {
    jvmToolchain(17)
}

application {
    mainClass.set("MainKt")
}
```

**settings.gradle.kts:**
```kotlin
rootProject.name = "my-project"
```

### Q86: What is the Kotlin script (.kts) file and how is it used?
**Answer:**
`.kts` files are Kotlin scripts executed directly without compilation:

**build.gradle.kts (Gradle build script):**
```kotlin
plugins {
    kotlin("jvm") version "1.9.0"
}
```

**Custom script (example.kts):**
```kotlin
#!/usr/bin/env kotlin

import java.io.File

val files = File(".").listFiles()
files?.forEach { println(it.name) }

// Run with: kotlin example.kts
```

**Use cases:**
- Gradle build files
- Automation scripts
- Quick prototyping
- CI/CD scripts

### Q87: How do you manage Kotlin project dependencies effectively?
**Answer:**

**Version catalogs (recommended):**
```kotlin
// gradle/libs.versions.toml
[versions]
kotlin = "1.9.0"
coroutines = "1.7.0"

[libraries]
kotlin-stdlib = { module = "org.jetbrains.kotlin:kotlin-stdlib", version.ref = "kotlin" }
coroutines-core = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "coroutines" }

[plugins]
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }

// build.gradle.kts
dependencies {
    implementation(libs.kotlin.stdlib)
    implementation(libs.coroutines.core)
}
```

**buildSrc for shared config:**
```kotlin
// buildSrc/src/main/kotlin/Dependencies.kt
object Versions {
    const val kotlin = "1.9.0"
}

object Deps {
    const val kotlinStdlib = "org.jetbrains.kotlin:kotlin-stdlib:${Versions.kotlin}"
}
```

### Q88: What is Kotlin style guide and why should you follow it?
**Answer:**
Official Kotlin coding conventions ensure consistency:

```kotlin
// Naming conventions
class UserService          // PascalCase for classes
fun fetchData()           // camelCase for functions
val userName: String      // camelCase for properties
const val MAX_SIZE = 100  // SCREAMING_SNAKE_CASE for constants

// Formatting
class Example(
    val name: String,
    val age: Int
) {
    fun doSomething() {
        if (condition) {
            // code
        }
    }
}

// When to use expression body
fun square(x: Int) = x * x  // Good for simple expressions

// Trailing comma
data class User(
    val name: String,
    val age: Int,  // Trailing comma
)
```

**Tools:**
- ktlint - linter/formatter
- detekt - static analysis
- IDE formatting (Ctrl+Alt+L)

---

## Best Practices

### Q89: What are the recommended conventions for naming and organizing Kotlin files?
**Answer:**

```kotlin
// File naming
UserService.kt           // Class file
Extensions.kt            // Multiple extensions
StringExtensions.kt      // Grouped extensions

// Package structure
com.example.app/
├── data/
│   ├── repository/
│   └── model/
├── domain/
│   └── usecase/
└── presentation/
    ├── viewmodel/
    └── ui/

// File organization
// 1. Single class per file (usually)
// 2. Multiple related top-level functions OK
// 3. Extensions can be grouped

// Naming conventions
interface UserRepository     // Interface without I prefix
class UserRepositoryImpl    // Implementation with Impl suffix
fun String.toSlug()         // Extension descriptive name
```

### Q90: How do you avoid common pitfalls with Kotlin's nullability?
**Answer:**

```kotlin
// 1. Avoid !! (not-null assertion)
// Bad
val name = user.name!!

// Good
val name = user.name ?: "default"
val name = user.name ?: return
val name = requireNotNull(user.name) { "Name required" }

// 2. Use safe calls and Elvis
val length = name?.length ?: 0

// 3. Use let for null checks
name?.let { nonNullName ->
    process(nonNullName)
}

// 4. Prefer non-null types
class User(
    val name: String,  // Not String?
    val email: String
)

// 5. Handle platform types carefully
val javaString: String = javaMethod()!!  // Be explicit

// 6. Use lateinit for lifecycle-initialized
class Activity {
    lateinit var presenter: Presenter
    
    fun isInitialized() = ::presenter.isInitialized
}
```

### Q91: What are some best practices to optimize Kotlin code for performance?
**Answer:**

```kotlin
// 1. Use sequences for large collections with multiple operations
val result = list.asSequence()
    .filter { it > 0 }
    .map { it * 2 }
    .take(10)
    .toList()

// 2. Use inline functions for lambdas
inline fun measure(block: () -> Unit) {
    val start = System.currentTimeMillis()
    block()
    println("Time: ${System.currentTimeMillis() - start}")
}

// 3. Avoid unnecessary object creation
// Bad
fun isEven(n: Int) = n % 2 == 0
list.filter(::isEven)  // Creates function reference each time

// Good
list.filter { it % 2 == 0 }

// 4. Use primitive arrays for performance
val intArray = IntArray(1000)  // Not Array<Int>

// 5. Use const for compile-time constants
const val MAX_SIZE = 1000  // Inlined at compile time
val maxSize = 1000  // Creates getter

// 6. Prefer value classes for wrappers
@JvmInline
value class UserId(val id: Int)
```

### Q92: How do you effectively use scope functions?
**Answer:**

```kotlin
// let - null checks and transformations
val result = nullableValue?.let {
    transform(it)
}

// apply - object configuration
val person = Person().apply {
    name = "John"
    age = 30
}

// run - compute with object
val description = person.run {
    "$name is $age years old"
}

// also - side effects
val numbers = mutableListOf(1, 2).also {
    println("Initial: $it")
}

// with - multiple operations on object
with(textView) {
    text = "Hello"
    visibility = View.VISIBLE
    setOnClickListener { }
}

// Decision guide:
// Need return value? -> let, run, with
// Need the object back? -> apply, also
// Reference with 'it'? -> let, also
// Reference with 'this'? -> apply, run, with
```

---

## Future Directions

### Q93: Discuss how coroutines have evolved in Kotlin and what the future might hold.
**Answer:**
Coroutines evolution:
- **1.1**: Experimental coroutines
- **1.3**: Stable coroutines, structured concurrency
- **1.4**: SharedFlow, StateFlow
- **1.5**: Improved cancellation
- **1.6+**: Performance improvements, better debugging

**Future directions:**
- Better debugging tools
- More integration with Project Loom (virtual threads)
- Enhanced multiplatform support
- Improved Flow operators

### Q94: What upcoming features are projected for Kotlin that developers should be aware of?
**Answer:**

- **Context receivers** (stable soon)
- **K2 compiler** - faster compilation
- **Static extensions**
- **Collection literals**
- **Improved multiplatform**
- **Better Java interop**

```kotlin
// Context receivers (experimental)
context(LoggingContext, TransactionContext)
fun performOperation() {
    log("Starting...")
    transaction {
        // operation
    }
}
```

### Q95: How is Kotlin being adopted for backend development, and why would you choose it over traditional Java frameworks?
**Answer:**

**Popular frameworks:**
- **Ktor** - Kotlin-native async framework
- **Spring Boot** - Full Kotlin support
- **Micronaut** - Compile-time DI
- **Quarkus** - Cloud-native

**Benefits over Java:**
```kotlin
// Ktor example
fun main() {
    embeddedServer(Netty, port = 8080) {
        routing {
            get("/") {
                call.respondText("Hello, World!")
            }
            
            get("/users/{id}") {
                val id = call.parameters["id"]
                call.respond(userService.getUser(id))
            }
        }
    }.start(wait = true)
}
```

| Benefit | Description |
|---------|-------------|
| Conciseness | Less boilerplate |
| Null safety | Fewer NPEs |
| Coroutines | Native async |
| DSLs | Clean configurations |
| Interop | Use existing Java libraries |

### Q96: What impact does Google's official support for Kotlin have on its adoption?
**Answer:**

**Timeline:**
- 2017: Google announces Kotlin support for Android
- 2019: Kotlin-first for Android development
- 2023+: Jetpack Compose (Kotlin-only)

**Impact:**
- Massive adoption in Android community
- More Kotlin-first libraries
- Better tooling support
- Industry standard for Android
- Growing backend adoption

---

## Miscellaneous

### Q97: How do you serialize and deserialize JSON in Kotlin?
**Answer:**

**kotlinx.serialization (recommended):**
```kotlin
@Serializable
data class User(val name: String, val age: Int)

val json = Json.encodeToString(User("John", 30))
// {"name":"John","age":30}

val user = Json.decodeFromString<User>(json)
```

**Gson:**
```kotlin
val gson = Gson()
val json = gson.toJson(User("John", 30))
val user = gson.fromJson(json, User::class.java)
```

**Moshi:**
```kotlin
val moshi = Moshi.Builder().add(KotlinJsonAdapterFactory()).build()
val adapter = moshi.adapter(User::class.java)
val json = adapter.toJson(User("John", 30))
val user = adapter.fromJson(json)
```

### Q98: Discuss how Kotlin manages memory and garbage collection.
**Answer:**
Kotlin on JVM uses JVM garbage collection:

```kotlin
// Same GC as Java (G1, ZGC, etc.)
// Kotlin-specific considerations:

// 1. Avoid unnecessary object creation
val list = List(1000) { it }  // Creates objects

// 2. Use primitive arrays
val intArray = IntArray(1000)  // No boxing

// 3. Inline classes avoid allocation
@JvmInline
value class UserId(val id: Int)  // No runtime object

// 4. Sequences for lazy evaluation
sequence.filter { }.map { }.toList()  // Intermediate collections avoided

// Kotlin/Native uses its own memory manager
// - Automatic memory management (ARC-like + cycle collector)
// - No garbage collection pauses
```

### Q99: What are the benefits of using Kotlin for server-side development?
**Answer:**

| Benefit | Description |
|---------|-------------|
| **Coroutines** | Efficient async I/O |
| **Null safety** | Fewer runtime errors |
| **Conciseness** | Less code to maintain |
| **Java interop** | Use existing libraries |
| **Type safety** | Catch errors at compile time |
| **DSLs** | Clean configuration |

```kotlin
// Spring Boot with Kotlin
@RestController
class UserController(private val userService: UserService) {
    
    @GetMapping("/users/{id}")
    suspend fun getUser(@PathVariable id: Long): User {
        return userService.findById(id)
    }
}

// Exposed (Kotlin SQL DSL)
Users.select { Users.age greater 18 }
    .map { it[Users.name] }
```

### Q100: Explain some common Kotlin idioms for handling common programming tasks.
**Answer:**

```kotlin
// 1. Default values instead of overloads
fun connect(host: String, port: Int = 80) { }

// 2. String templates
val message = "Hello, $name! You have ${items.size} items."

// 3. Safe calls chain
val city = user?.address?.city ?: "Unknown"

// 4. When for complex conditionals
val result = when {
    x < 0 -> "negative"
    x == 0 -> "zero"
    else -> "positive"
}

// 5. Single-expression functions
fun double(x: Int) = x * 2

// 6. Data classes for DTOs
data class Point(val x: Int, val y: Int)

// 7. Extension functions for utilities
fun String.isEmail() = contains("@")

// 8. Scope functions for configuration
val config = Config().apply {
    debug = true
    timeout = 30
}

// 9. Destructuring
val (name, age) = person

// 10. Collection operations
val adults = people.filter { it.age >= 18 }
                   .sortedBy { it.name }
                   .map { it.name }
```

