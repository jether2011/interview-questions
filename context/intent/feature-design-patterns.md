# Feature: Design Patterns & SOLID Principles Content

## What

Interview preparation content covering the foundational software design principles and patterns that enable maintainable, extensible, and robust code. This feature provides comprehensive Q&A materials on:

- **SOLID Principles**: The five principles that guide object-oriented design - Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion. Understanding not just what they are, but how to apply them in real code and recognize violations.

- **Creational Patterns**: Patterns that abstract object creation - Singleton (and why it's often an anti-pattern), Factory Method, Abstract Factory, Builder (essential for Java), and Prototype. When each pattern adds value vs adds unnecessary complexity.

- **Structural Patterns**: Patterns that compose objects into larger structures - Adapter (making incompatible interfaces work together), Decorator (adding behavior dynamically), Facade (simplifying complex subsystems), Proxy (controlling access), and Composite (treating individual objects and compositions uniformly).

- **Behavioral Patterns**: Patterns that define how objects communicate - Strategy (interchangeable algorithms), Observer (event handling), Command (encapsulating requests), Template Method (defining algorithm skeletons), and State (behavior based on internal state).

- **Enterprise Patterns**: Patterns common in enterprise Java applications - Repository, Unit of Work, Domain Events, Specification, and DTO/Value Object patterns that structure business logic.

## Why

Design patterns are the shared vocabulary of software engineering:

1. **Communication**: Patterns give names to solutions, enabling concise technical discussions
2. **Proven Solutions**: Each pattern solves a specific problem that has been encountered repeatedly
3. **Interview Essential**: Senior developers are expected to know GoF patterns and when to apply them
4. **Code Quality**: Proper pattern usage leads to more maintainable and testable code

This content helps developers recognize when patterns apply and, equally important, when they add unnecessary complexity.

## Acceptance Criteria
- [ ] Covers all SOLID principles with examples
- [ ] Documents creational patterns (Singleton, Factory, Builder, etc.)
- [ ] Covers structural patterns (Adapter, Decorator, Facade, etc.)
- [ ] Documents behavioral patterns (Strategy, Observer, Command, etc.)
- [ ] Covers enterprise patterns (Repository, Unit of Work, etc.)
- [ ] Documents common anti-patterns to avoid
- [ ] Includes pattern selection guidelines
- [ ] Provides trade-off analysis for pattern choices

## Content Sections

### 1. SOLID Principles

**Single Responsibility Principle (SRP)**: A class should have only one reason to change. Identifying responsibilities, splitting classes appropriately, and avoiding "God classes" that do everything.

**Open/Closed Principle (OCP)**: Software entities should be open for extension but closed for modification. Using abstraction and polymorphism to add new behavior without changing existing code.

**Liskov Substitution Principle (LSP)**: Subtypes must be substitutable for their base types. The Square/Rectangle problem, covariant return types, and behavioral subtyping.

**Interface Segregation Principle (ISP)**: Clients shouldn't depend on interfaces they don't use. Splitting "fat" interfaces into focused ones, and the relationship to SRP.

**Dependency Inversion Principle (DIP)**: High-level modules shouldn't depend on low-level modules; both should depend on abstractions. How this enables testability and flexibility.

### 2. Creational Patterns

**Singleton**: Ensuring a class has only one instance. Implementation approaches: eager, lazy, double-checked locking, enum singleton. Why Singleton is often an anti-pattern (hidden dependencies, testing difficulties) and alternatives (dependency injection).

**Factory Method**: Define an interface for creating objects, letting subclasses decide which class to instantiate. Decouples client code from concrete classes.

**Abstract Factory**: Create families of related objects without specifying concrete classes. When you have multiple product variations (e.g., UI components for different platforms).

**Builder**: Construct complex objects step by step. Essential for Java objects with many optional parameters. Fluent interface design, immutability with builders, and the Lombok @Builder annotation.

**Prototype**: Create new objects by copying existing ones. Java's clone() method, deep vs shallow copy, and when copying is cheaper than construction.

### 3. Structural Patterns

**Adapter**: Convert the interface of a class into another interface clients expect. Class adapter (inheritance) vs object adapter (composition). Integrating legacy code or third-party libraries.

**Decorator**: Attach additional responsibilities to objects dynamically. Java I/O streams as the classic example (BufferedInputStream decorating FileInputStream). Composition over inheritance.

**Facade**: Provide a unified interface to a set of interfaces in a subsystem. Simplifying complex APIs, reducing coupling to subsystem internals.

**Proxy**: Provide a surrogate to control access to another object. Types: virtual proxy (lazy loading), protection proxy (access control), remote proxy (network access). Spring AOP uses proxies extensively.

**Composite**: Compose objects into tree structures to represent part-whole hierarchies. File systems, UI components, organization structures.

**Bridge**: Decouple abstraction from implementation so both can vary independently. When you have multiple dimensions of variation.

### 4. Behavioral Patterns

**Strategy**: Define a family of algorithms, encapsulate each one, and make them interchangeable. Payment processing, sorting algorithms, validation strategies. Replacing conditionals with polymorphism.

**Observer**: Define a one-to-many dependency so that when one object changes state, all dependents are notified. Event handling, MVC architecture, reactive programming foundations.

**Command**: Encapsulate a request as an object, allowing parameterization, queuing, logging, and undo operations. Implementing undo/redo, macro recording, transaction management.

**Template Method**: Define the skeleton of an algorithm in a base class, letting subclasses override specific steps. Framework design, hook methods, and the Hollywood Principle ("don't call us, we'll call you").

**State**: Allow an object to alter its behavior when its internal state changes. State machines, workflow engines, replacing complex conditionals based on state.

**Chain of Responsibility**: Pass a request along a chain of handlers until one handles it. Servlet filters, logging handlers, approval workflows.

**Iterator**: Access elements of a collection sequentially without exposing underlying representation. Java's Iterator and Iterable interfaces, for-each loop support.

### 5. Enterprise Patterns

**Repository**: Mediate between domain and data mapping layers using a collection-like interface. Decoupling business logic from data access, enabling unit testing with in-memory implementations.

**Unit of Work**: Maintain a list of objects affected by a business transaction and coordinate writing out changes. JPA's EntityManager, transaction boundaries.

**Domain Events**: Capture side effects of domain operations as explicit events. Decoupling, eventual consistency, audit trails.

**Specification**: Encapsulate business rules in reusable, combinable objects. Query building, validation rules, business rule engines.

**DTO (Data Transfer Object)**: Objects that carry data between processes. API responses, avoiding exposing domain models, mapping with MapStruct.

**Value Object**: Objects that have no identity, defined only by their attributes. Immutable, equality by value. Money, Address, DateRange examples.

### 6. Anti-Patterns
Common mistakes to recognize and avoid:
- **God Class**: One class that does everything
- **Anemic Domain Model**: Domain objects with only getters/setters, logic elsewhere
- **Service Locator**: Global registry for dependencies (prefer DI)
- **Spaghetti Code**: Tangled control flow, no clear structure
- **Golden Hammer**: Using one pattern/tool for everything
- **Premature Optimization**: Optimizing before measuring

### 7. Pattern Selection & Trade-offs
Guidelines for choosing patterns:
- **Start simple**: Don't use a pattern until you need it
- **YAGNI**: You Aren't Gonna Need It - avoid speculative generality
- **Composition over inheritance**: Prefer object composition to class inheritance
- **Identify the varying part**: Patterns encapsulate what changes
- **Know the trade-offs**: Every pattern has costs (complexity, indirection)

## Key Concepts to Master

Essential pattern knowledge for interviews:

- **Builder pattern**: Be able to implement it for a class with many optional parameters
- **Strategy vs Template Method**: Know when to use each (composition vs inheritance)
- **Singleton pitfalls**: Explain why DI is often better
- **Decorator composition**: Show how decorators can be layered (I/O streams)
- **Repository vs DAO**: Understand the subtle differences

## Study Tips

1. **Implement each pattern** - Writing code solidifies understanding better than reading
2. **Find patterns in frameworks** - Spring (DI, Proxy), Java I/O (Decorator), Collections (Iterator)
3. **Practice pattern recognition** - Given a problem, identify which pattern applies
4. **Know the "bad" examples** - Understand what code looks like WITHOUT the pattern
5. **Explain trade-offs** - Patterns add complexity; know when NOT to use them

## Related
- [Project Intent](project-intent.md)
- [Feature: Java Fundamentals](feature-java-fundamentals.md)
- [Decision: Content Structure](../decisions/001-content-structure.md)
- [Pattern: Q&A Format](../knowledge/patterns/qa-format.md)
- [Pattern: Code Examples](../knowledge/patterns/code-examples.md)

## Status
- **Created**: 2026-01-19 (Phase: Intent)
- **Updated**: 2026-01-19 - Enhanced content for better study experience
- **Status**: Active (already implemented)
- **File**: `design-patterns-solid.md`
