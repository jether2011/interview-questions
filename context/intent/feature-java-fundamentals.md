# Feature: Java Fundamentals Content

## What

Comprehensive interview preparation content covering the essential Java concepts that form the foundation of all senior Java developer interviews. This feature provides in-depth Q&A materials on:

- **Object-Oriented Programming (OOP)**: The four pillars that define Java's programming paradigm - encapsulation, inheritance, polymorphism, and abstraction - with practical examples showing when and how to apply each principle.

- **Java Platform Architecture**: Understanding the relationship between JDK, JRE, and JVM, how bytecode execution works, and why Java achieves platform independence through "write once, run anywhere."

- **Collections Framework**: Deep exploration of Java's data structures including internal implementations of HashMap, ArrayList, LinkedList, TreeMap, and HashSet, along with their time complexities and appropriate use cases.

- **Memory Management**: How the JVM manages memory through heap and stack, understanding object lifecycle, and the role of garbage collection in automatic memory management.

- **Modern Java Features**: Java 8+ innovations including lambda expressions, Stream API, Optional class, functional interfaces, and the new Date/Time API that have transformed how Java code is written.

## Why

Java fundamentals are the bedrock of every senior developer interview. Interviewers use these questions to:

1. **Assess Deep Understanding**: Surface-level knowledge won't pass senior interviews - you need to explain *why* things work, not just *what* they do
2. **Evaluate Problem-Solving**: Understanding internals (like HashMap collision handling) shows you can debug complex issues
3. **Test Experience**: Senior developers should know trade-offs (ArrayList vs LinkedList) from real project experience
4. **Verify Code Quality**: Knowledge of equals/hashCode contract and exception handling shows you write production-quality code

This content helps developers move beyond memorization to true comprehension, enabling confident explanations during interviews.

## Acceptance Criteria
- [ ] Covers core OOP concepts (encapsulation, inheritance, polymorphism, abstraction)
- [ ] Explains collections framework with internal implementations
- [ ] Documents equals/hashCode contract and best practices
- [ ] Covers memory management and JVM architecture
- [ ] Explains garbage collection algorithms and tuning
- [ ] Documents Java 8+ features (lambdas, streams, Optional, etc.)
- [ ] Covers exception handling best practices
- [ ] Includes code examples for each concept

## Content Sections

### 1. Core Java Concepts
Covers the four pillars of OOP with Java-specific implementations, the difference between JDK/JRE/JVM and their roles, abstract classes vs interfaces (including Java 8+ default methods), access modifiers and their scopes, and the five types of inner classes. This section establishes the vocabulary and mental models needed for all other topics.

### 2. Collections Framework
Deep dive into Java's collection hierarchy starting with HashMap internals (bucket array, hash function, collision handling with linked lists and tree conversion at threshold 8). Compares ArrayList vs LinkedList performance characteristics, covers TreeMap's red-black tree implementation, and explains when to use each collection type based on access patterns and performance requirements.

### 3. Equals and HashCode
Explains the critical contract between equals() and hashCode() methods, why violating it breaks HashMap/HashSet behavior, and best practices for implementing both methods. Covers the difference between == (reference comparison) and equals() (value comparison), with common interview gotchas.

### 4. Multithreading and Concurrency (Overview)
Introduction to thread creation, synchronization basics, and thread safety concepts. This section provides foundational knowledge that connects to the dedicated multithreading feature for deeper exploration of concurrent programming.

### 5. Memory Management and JVM
Covers JVM architecture including class loader subsystem, runtime data areas (heap, stack, method area, PC registers), and execution engine. Explains how objects are allocated, the difference between stack and heap memory, and how to identify memory leaks.

### 6. Garbage Collection
Explores GC algorithms (Serial, Parallel, CMS, G1, ZGC), generational hypothesis (Young/Old generations), GC roots and reachability, and tuning parameters. Covers how to analyze GC logs and optimize for latency vs throughput.

### 7. Java 8+ Features
Comprehensive coverage of lambda expressions and functional interfaces, Stream API operations (intermediate vs terminal), Optional for null-safe programming, new Date/Time API (LocalDate, LocalDateTime, ZonedDateTime), and default/static methods in interfaces. Includes practical examples showing before/after Java 8 code.

### 8. Exception Handling
Covers checked vs unchecked exceptions, try-with-resources and AutoCloseable, exception hierarchy, best practices for creating custom exceptions, and common anti-patterns to avoid. Explains when to catch, when to propagate, and how to handle exceptions in multi-threaded code.

### 9. Data Structures
Reviews fundamental data structures (arrays, linked lists, trees, graphs) and their Java implementations. Covers Big O complexity analysis for common operations, helping developers choose the right structure for their use case.

## Key Concepts to Master

These are the topics interviewers most frequently ask about:

- **HashMap internals**: Be ready to explain buckets, hash collisions, load factor (0.75), and tree conversion
- **equals/hashCode contract**: Know why both must be overridden together and what breaks if you don't
- **Garbage Collection**: Understand generational GC and be able to discuss G1 vs ZGC trade-offs
- **Stream API**: Know the difference between intermediate and terminal operations, and lazy evaluation
- **Memory model**: Explain stack vs heap, and how objects are referenced

## Study Tips

1. **Practice explaining out loud** - Senior interviews require verbal explanation, not just code
2. **Draw diagrams** - HashMap buckets, GC generations, and collection hierarchies are easier to explain visually
3. **Know the "why"** - For every concept, understand the problem it solves
4. **Code examples** - Be ready to write equals/hashCode, Stream operations, and lambda expressions
5. **Real-world scenarios** - Connect concepts to actual problems you've solved

## Related
- [Project Intent](project-intent.md)
- [Decision: Content Structure](../decisions/001-content-structure.md)
- [Pattern: Q&A Format](../knowledge/patterns/qa-format.md)
- [Pattern: Code Examples](../knowledge/patterns/code-examples.md)
- [Pattern: Comparison Tables](../knowledge/patterns/comparison-tables.md)

## Status
- **Created**: 2026-01-19 (Phase: Intent)
- **Updated**: 2026-01-19 - Enhanced content for better study experience
- **Status**: Active (already implemented)
- **File**: `java-fundamentals.md`
