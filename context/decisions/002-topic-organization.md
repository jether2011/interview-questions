# Decision: Topic Organization

## Context
The project covers a wide range of senior Java interview topics. The content needed to be organized in a way that allows focused study while maintaining clear boundaries between related but distinct topics.

## Decision
Organize content into 10 separate files by technical domain:
1. `java-fundamentals.md` - Core Java concepts
2. `java-multithreading.md` - Concurrency deep-dive
3. `spring-boot.md` - Spring ecosystem
4. `microservices-patterns.md` - Microservices architecture
5. `system-design.md` - System design principles
6. `distributed-systems-architecture.md` - Distributed computing
7. `docker-kubernetes.md` - Containerization
8. `design-patterns-solid.md` - Design patterns & SOLID
9. `database-caching.md` - Data management
10. `messaging-event-driven.md` - Messaging systems

## Rationale
- **Domain Separation**: Each file covers a distinct technical domain
- **Manageable Size**: Files are comprehensive but not overwhelming
- **Interview Alignment**: Topics align with common interview categories
- **Progressive Learning**: Can study from fundamentals to advanced topics
- **Related Topics Grouped**: e.g., all Spring content in one file

## Alternatives Considered
1. **Single monolithic file**: Rejected - too large to navigate
2. **Many small files (one per question)**: Rejected - loses context, hard to study
3. **By difficulty level**: Rejected - doesn't match interview structure
4. **By company/role type**: Rejected - too specialized, reduces reusability

## Outcomes
*To be documented as project evolves.*

## Related
- [Project Intent](../intent/project-intent.md)
- [Decision: Content Structure](001-content-structure.md)
- All feature files in `intent/`

## Status
- **Created**: 2026-01-19 (Phase: Intent)
- **Status**: Accepted
- **Note**: Documented from existing implementation
