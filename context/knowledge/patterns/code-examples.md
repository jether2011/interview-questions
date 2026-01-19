# Pattern: Code Examples

## Description
Java code examples embedded within answers to demonstrate practical implementations of concepts. Code blocks use proper Java syntax highlighting and include both "bad" and "good" examples where appropriate.

## When to Use
- When explaining implementation details
- When showing design patterns
- When demonstrating correct vs incorrect approaches
- When illustrating API usage

## Pattern
```markdown
[Explanation text]

```java
// [Comment describing what this code shows]
[Java code with proper formatting]
```

[Additional explanation if needed]
```

### For Good/Bad Comparisons:
```markdown
```java
// Bad: [Why this is problematic]
[Code showing anti-pattern]

// Good: [Why this is better]
[Code showing correct approach]
```
```

## Example

```java
// Bad: Multiple responsibilities
public class Employee {
    private String name;
    private BigDecimal salary;
    
    public void calculatePay() { }
    public void saveToDatabase() { }  // Database responsibility
    public void sendEmail() { }       // Email responsibility
}

// Good: Single responsibility
public class Employee {
    private String name;
    private BigDecimal salary;
    
    public BigDecimal calculatePay() { 
        return salary;
    }
}

public class EmployeeRepository {
    public void save(Employee employee) { }
}

public class EmailService {
    public void sendEmail(Employee employee) { }
}
```

## Code Style Conventions
1. **Proper indentation**: 4 spaces for Java
2. **Descriptive comments**: Explain what the code demonstrates
3. **Complete but concise**: Show enough context without boilerplate
4. **Realistic names**: Use meaningful class/method names
5. **Type annotations**: Always include types in Java code

## Files Using This Pattern
- `java-fundamentals.md` - Collections, streams, OOP examples
- `java-multithreading.md` - Threading, synchronization examples
- `spring-boot.md` - DI, configuration, REST examples
- `microservices-patterns.md` - Pattern implementations
- `design-patterns-solid.md` - All design pattern examples
- `database-caching.md` - Query and caching examples

## Related
- [Decision: Content Structure](../../decisions/001-content-structure.md)
- [Pattern: Q&A Format](qa-format.md)

## Status
- **Created**: 2026-01-19
- **Status**: Active
