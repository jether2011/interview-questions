# Pattern: Comparison Tables

## Description
Markdown tables used to compare similar technologies, approaches, or concepts side-by-side. This pattern provides quick visual reference for understanding differences between related items.

## When to Use
- When comparing two or more similar technologies
- When explaining trade-offs between approaches
- When answering "What's the difference between X and Y?" questions
- When showing pros/cons of different options

## Pattern
```markdown
| [Option A] | [Option B] |
|------------|------------|
| [Characteristic 1 of A] | [Characteristic 1 of B] |
| [Characteristic 2 of A] | [Characteristic 2 of B] |
| [Characteristic 3 of A] | [Characteristic 3 of B] |
```

### For Multiple Options:
```markdown
| Feature | [Option A] | [Option B] | [Option C] |
|---------|------------|------------|------------|
| [Feature 1] | [A's value] | [B's value] | [C's value] |
| [Feature 2] | [A's value] | [B's value] | [C's value] |
```

## Example

```markdown
### Q7: What's the difference between HashMap and Hashtable?
**Answer:**
| HashMap | Hashtable |
|---------|-----------|
| Not synchronized (not thread-safe) | Synchronized (thread-safe) |
| Allows one null key and multiple null values | No null keys or values |
| Introduced in Java 1.2 | Legacy class (Java 1.0) |
| Better performance | Slower due to synchronization |
| Fail-fast iterator | Enumerator is not fail-fast |
```

```markdown
### Q9: What is the time complexity of common operations in Java collections?
**Answer:**
| Collection | get | add | remove | contains |
|------------|-----|-----|--------|----------|
| ArrayList | O(1) | O(1)* | O(n) | O(n) |
| LinkedList | O(n) | O(1) | O(1)** | O(n) |
| HashMap | O(1)*** | O(1)*** | O(1)*** | O(1)*** |
| TreeMap | O(log n) | O(log n) | O(log n) | O(log n) |

*amortized **at ends ***average case
```

## Table Style Conventions
1. **Header row**: Always include clear column headers
2. **Alignment**: Left-align text, keep consistent
3. **Concise cells**: Keep cell content brief and scannable
4. **Footnotes**: Use asterisks for additional notes
5. **Consistent terminology**: Use same terms across comparison

## Files Using This Pattern
- `java-fundamentals.md` - Collections comparisons
- `java-multithreading.md` - Lock types, executor comparisons
- `database-caching.md` - SQL vs NoSQL, cache strategies
- `messaging-event-driven.md` - Kafka vs RabbitMQ
- `microservices-patterns.md` - Communication pattern comparisons

## Related
- [Decision: Content Structure](../../decisions/001-content-structure.md)
- [Pattern: Q&A Format](qa-format.md)

## Status
- **Created**: 2026-01-19
- **Status**: Active
