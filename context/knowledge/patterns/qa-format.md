# Pattern: Q&A Format

## Description
A consistent question-and-answer format used throughout all interview preparation content files. This pattern structures technical explanations as interview-style questions with comprehensive answers.

## When to Use
- When adding new interview questions to any topic file
- When explaining technical concepts that could be asked in interviews
- When documenting best practices or comparisons

## Pattern
```markdown
### Q[NUMBER]: [Question text in natural interview style]
**Answer:**
[Comprehensive answer with:]
- Key points as bullet lists when applicable
- Technical explanations
- Trade-offs and considerations

[Optional: Code example]

[Optional: Comparison table for "vs" questions]
```

## Example

```markdown
### Q1: What are the main principles of OOP in Java?
**Answer:** 
- **Encapsulation**: Bundling data and methods that operate on that data within a single unit (class), hiding internal implementation details
- **Inheritance**: Mechanism where one class acquires properties and behaviors from a parent class
- **Polymorphism**: Ability of objects to take multiple forms (method overloading and overriding)
- **Abstraction**: Hiding complex implementation details and showing only essential features
```

```markdown
### Q7: What's the difference between HashMap and Hashtable?
**Answer:**
| HashMap | Hashtable |
|---------|-----------|
| Not synchronized (not thread-safe) | Synchronized (thread-safe) |
| Allows one null key and multiple null values | No null keys or values |
| Introduced in Java 1.2 | Legacy class (Java 1.0) |
| Better performance | Slower due to synchronization |
```

## Files Using This Pattern
- `java-fundamentals.md` - All 40+ questions
- `java-multithreading.md` - Concurrency questions
- `spring-boot.md` - Spring framework questions
- `microservices-patterns.md` - Architecture questions
- `system-design.md` - Design questions
- `distributed-systems-architecture.md` - Distributed systems questions
- `docker-kubernetes.md` - Container questions
- `design-patterns-solid.md` - Pattern questions
- `database-caching.md` - Database questions
- `messaging-event-driven.md` - Messaging questions

## Related
- [Decision: Content Structure](../../decisions/001-content-structure.md)
- [Pattern: Code Examples](code-examples.md)
- [Pattern: Comparison Tables](comparison-tables.md)

## Status
- **Created**: 2026-01-19
- **Status**: Active
