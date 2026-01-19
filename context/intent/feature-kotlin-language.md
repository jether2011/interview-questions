# Feature: Kotlin Language Content

## What

Comprehensive interview preparation content covering Kotlin programming language for senior developer positions, especially focused on Android and backend development. This feature provides in-depth Q&A materials based on 100 interview questions from [devinterview.io](https://devinterview.io/questions/web-and-mobile-development/kotlin-interview-questions) covering:

- **Kotlin Fundamentals**: Core language features including type inference, null safety, extension functions, and how Kotlin interoperates with Java.

- **Control Flow and Error Handling**: Understanding `when` expressions, Elvis operator, smart casts, null safety mechanisms, and exception handling in Kotlin.

- **Classes and Objects**: Data classes, sealed classes, companion objects, inheritance, properties vs fields, and object expressions.

- **Functions and Lambdas**: Higher-order functions, inline functions, lambda expressions, scope functions (`with`, `apply`, `let`, `run`, `also`), and tail recursion.

- **Collections and Functional Constructs**: Lists, sets, maps, lazy evaluation, sequences, and functional operations like `map`, `flatMap`, `filter`.

- **Coroutines and Concurrency**: Suspend functions, coroutine scopes, launch/async, structured concurrency, exception handling in coroutines.

- **Java Interoperability**: Calling Java from Kotlin and vice versa, annotations (`@JvmStatic`, `@JvmOverloads`, `@JvmField`, `@JvmName`).

- **Advanced Topics**: Delegation, dependency injection, generics (reified types, variance), type aliasing, destructuring declarations, inline classes.

- **Android Specific**: Android development with Kotlin, LiveData, ViewModel, coroutine scopes in Android, Kotlin Android Extensions.

- **DSL and Meta-Programming**: Creating domain-specific languages, reflection, and scope functions for DSL building.

- **Multiplatform Development**: Kotlin Multiplatform Mobile (KMM), Kotlin/Native, shared business logic, cross-platform limitations.

- **Testing and Tooling**: Testing frameworks, mocking dependencies, REPL, and best practices for testable code.

- **Asynchronous Flow**: Kotlin Flow, backpressure handling, cold vs hot streams, callback-to-suspend conversions.

## Why

Kotlin has become the preferred language for Android development (officially recommended by Google) and is gaining strong traction in backend development. Interviewers use these questions to:

1. **Assess Modern Language Proficiency**: Kotlin expertise demonstrates awareness of modern development practices and language features
2. **Evaluate Concurrency Knowledge**: Coroutines are fundamental to Kotlin development; understanding them shows advanced async programming skills
3. **Test Java Interoperability**: Most projects have mixed Java/Kotlin codebases; interop knowledge is essential
4. **Verify Android Competency**: For mobile positions, Kotlin is now the standard; deep knowledge separates senior from junior developers
5. **Assess Functional Programming**: Kotlin's functional features (lambdas, higher-order functions) test modern programming paradigms understanding

This content helps developers prepare for interviews at top tech companies where Kotlin is used for Android, backend (Ktor, Spring), or multiplatform development.

## Acceptance Criteria

- [ ] Covers all 100 questions from devinterview.io Kotlin interview questions
- [ ] Questions organized by topic/difficulty level
- [ ] Includes Kotlin code examples with syntax highlighting
- [ ] Covers Kotlin Fundamentals (10 questions)
- [ ] Covers Control Flow and Error Handling (8 questions)
- [ ] Covers Classes and Objects (9 questions)
- [ ] Covers Functions and Lambdas (7 questions)
- [ ] Covers Collections and Functional Constructs (6 questions)
- [ ] Covers Coroutines and Concurrency (8 questions)
- [ ] Covers Java Interoperability (5 questions)
- [ ] Covers Advanced Topics (7 questions)
- [ ] Covers Android Specific topics (7 questions)
- [ ] Covers DSL and Meta-Programming (4 questions)
- [ ] Covers Multiplatform Development (5 questions)
- [ ] Covers Testing and Tooling (4 questions)
- [ ] Covers Asynchronous Flow (4 questions)
- [ ] Covers Project Setup (4 questions)
- [ ] Covers Best Practices (4 questions)
- [ ] Covers Future Directions (4 questions)
- [ ] Covers Miscellaneous topics (4 questions)
- [ ] Includes comparison tables for Java vs Kotlin concepts
- [ ] Follows Q&A format pattern

## Content Sections

### 1. Kotlin Fundamentals (Questions 1-10)
Core language features including what Kotlin is and its Java interoperability, improvements over Java for Android development, basic types, `val` vs `var` differences, singleton creation with `object`, type inference rules, `Unit` type, string interpolation, and extension functions.

### 2. Control Flow and Error Handling (Questions 11-18)
Kotlin's `if` expressions (vs Java statements), `when` expressions for pattern matching, null safety and Elvis operator (`?:`), smart casts for automatic type conversions, custom getters/setters, exception handling differences from Java, and the `Nothing` type in control flow.

### 3. Classes and Objects (Questions 19-27)
Class creation, primary vs secondary constructors, data classes (automatic `equals`/`hashCode`/`copy`), inheritance with `open` keyword, sealed classes for restricted hierarchies, properties vs backing fields, object expressions (anonymous classes), companion objects, and enum definitions.

### 4. Functions and Lambdas (Questions 28-34)
Function definitions, higher-order functions (functions as parameters/return values), inline functions for performance, lambda syntax and usage, scope functions (`with`, `apply`, `let`, `run`, `also`), tail recursive functions with `tailrec`, and default/named parameters.

### 5. Collections and Functional Constructs (Questions 35-40)
Using lists, sets, and maps (mutable vs immutable), `map` vs `flatMap` operations, lazy collection evaluation, iteration methods (`forEach`, `forEachIndexed`, iterator), sequences for large data processing, and collection transformation patterns.

### 6. Coroutines and Concurrency (Questions 41-48)
Coroutines vs threads (lightweight concurrency), launching coroutines with `launch` and `async`, suspend functions and continuation, coroutine context and dispatchers, scopes (`GlobalScope`, `CoroutineScope`, `viewModelScope`), cancellation and exception handling, and threading restrictions.

### 7. Java Interoperability (Questions 49-53)
Achieving seamless Kotlin-Java interoperability, calling Kotlin from Java code, using Java annotations in Kotlin, JVM annotations (`@JvmStatic`, `@JvmOverloads`, `@JvmField`, `@JvmName`), and using Java Streams in Kotlin.

### 8. Advanced Topics (Questions 54-60)
Delegation pattern with `by` keyword, dependency injection approaches (Koin, Dagger/Hilt), type aliases for complex types, generics with reified types and variance (`in`/`out`), `vararg` vs array, destructuring declarations, and inline/value classes.

### 9. Android Specific (Questions 61-67)
Non-null view properties in Android, benefits of Kotlin for Android development, Kotlin Android Extensions (now deprecated, use ViewBinding), configuration change handling, LiveData usage, replacing AsyncTask with coroutines, and `lifecycleScope`/`viewModelScope`.

### 10. DSL and Meta-Programming (Questions 68-71)
Creating domain-specific languages in Kotlin, extension functions for DSL building, reflection capabilities and use cases, and how scope functions improve DSL writing.

### 11. Multiplatform Development (Questions 72-76)
Kotlin Multiplatform Mobile (KMM) for shared code, Kotlin/Native for iOS and native targets, sharing business logic across platforms, current limitations, and build system/tooling support.

### 12. Testing and Tooling (Questions 77-80)
Testing frameworks (JUnit, MockK, Kotest), mocking dependencies with MockK, best practices for testable Kotlin code, and Kotlin REPL for interactive development.

### 13. Asynchronous Flow (Questions 81-84)
Kotlin Flow vs coroutines, backpressure handling with `buffer`/`conflate`, cold flows vs hot channels/SharedFlow, and converting callbacks to suspend functions.

### 14. Project Setup (Questions 85-88)
Setting up Kotlin projects with Gradle, Kotlin script (`.kts`) files, dependency management, and following Kotlin coding style guide.

### 15. Best Practices (Questions 89-92)
Naming and organizing files, avoiding nullability pitfalls, performance optimization, and effective use of scope functions.

### 16. Future Directions (Questions 93-96)
Coroutines evolution, upcoming Kotlin features, backend development adoption, and impact of Google's official support.

### 17. Miscellaneous (Questions 97-100)
JSON serialization/deserialization (kotlinx.serialization, Gson, Moshi), memory management and garbage collection, server-side Kotlin benefits, and common Kotlin idioms.

## Key Concepts to Master

These are the topics interviewers most frequently ask about:

- **Null Safety**: Nullable types (`?`), Elvis operator (`?:`), safe calls (`?.`), not-null assertion (`!!`)
- **Coroutines**: `suspend` functions, `launch` vs `async`, structured concurrency, cancellation
- **Data Classes**: Automatic `equals`/`hashCode`/`toString`/`copy`, destructuring
- **Extension Functions**: Adding methods to existing classes without inheritance
- **Scope Functions**: When to use `let`, `run`, `with`, `apply`, `also`
- **Sealed Classes**: Exhaustive `when` expressions, state modeling
- **Java Interoperability**: Calling Java from Kotlin and vice versa

## Study Tips

1. **Compare with Java** - Most interviewers will ask how Kotlin improves on Java concepts
2. **Practice coroutines** - Write actual concurrent code; understanding comes from practice
3. **Know the "why"** - Explain *why* Kotlin has certain features (null safety solves NPEs)
4. **Code in the REPL** - Quick experimentation helps solidify understanding
5. **Android + Backend** - Know both use cases, as many companies use Kotlin for both

## Related

- [Project Intent](project-intent.md)
- [Decision: Kotlin Implementation](../decisions/004-kotlin-implementation.md)
- [Pattern: Q&A Format](../knowledge/patterns/qa-format.md)
- [Pattern: Code Examples](../knowledge/patterns/code-examples.md)
- [Pattern: Comparison Tables](../knowledge/patterns/comparison-tables.md)

## Status

- **Created**: 2026-01-19 (Phase: Intent)
- **Implemented**: 2026-01-19 (Phase: Build)
- **Status**: Active (implemented)
- **File**: `kotlin-language.md` (4,580 lines, 100 questions)
