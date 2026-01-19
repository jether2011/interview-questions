# Decision: Content Structure

## Context
The project needed a consistent format for presenting interview preparation content that would be easy to study, reference during preparation, and navigate quickly. The content covers complex technical topics that require both explanations and practical examples.

## Decision
Use Markdown format with structured Q&A style for all content files:
- Each topic in its own `.md` file at root level
- Numbered questions with "Q#" prefix
- Answers with "**Answer:**" marker
- Table of Contents with anchor links at top of each file
- Code examples embedded within answers

## Rationale
- **Markdown**: Universal format, renders well on GitHub, easy to read raw
- **Q&A Format**: Mimics actual interview experience, easy to practice
- **Single File Per Topic**: Keeps related content together, easy to navigate
- **Table of Contents**: Quick navigation in large files
- **Embedded Code**: Keeps examples close to explanations

## Alternatives Considered
1. **Wiki-style documentation**: Rejected - harder to maintain, less portable
2. **PDF format**: Rejected - not easily editable, poor code formatting
3. **Interactive website**: Rejected - adds maintenance overhead, reduces portability
4. **Multiple small files per topic**: Rejected - increases navigation complexity

## Outcomes
*To be documented as project evolves.*

## Related
- [Project Intent](../intent/project-intent.md)
- [Feature: Java Fundamentals](../intent/feature-java-fundamentals.md)
- [Feature: Spring Boot](../intent/feature-spring-boot.md)
- [Decision: Topic Organization](002-topic-organization.md)
- [Pattern: Q&A Format](../knowledge/patterns/qa-format.md)

## Status
- **Created**: 2026-01-19 (Phase: Intent)
- **Status**: Accepted
- **Note**: Documented from existing implementation
