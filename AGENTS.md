# AGENTS.md

## Setup Commands
- **Install**: No installation required (Markdown files only)
- **Dev**: Open files in any Markdown editor or IDE
- **Build**: N/A (documentation project)
- **Test**: N/A (documentation project)

## Code Style
- Use Markdown format for all content
- Follow Q&A pattern from `@context/knowledge/patterns/qa-format.md`
- Include Java code examples with syntax highlighting
- Use comparison tables for "vs" questions
- Include Table of Contents with anchor links for navigation
- Follow patterns from `@context/knowledge/patterns/`

## Context Files to Load

Before starting any work, load relevant context:
- `@context/intent/project-intent.md` (always)
- `@context/intent/feature-*.md` (for specific topic)
- `@context/decisions/*.md` (relevant decisions)
- `@context/knowledge/patterns/*.md` (patterns to follow)

## Project Structure
```
root/
├── AGENTS.md                              # This file
├── README.md                              # Project overview
├── LICENSE                                # MIT License
├── context/
│   ├── .context-mesh-framework.md         # Framework rules
│   ├── intent/
│   │   ├── project-intent.md              # Project intent
│   │   └── feature-*.md                   # Feature intents
│   ├── decisions/
│   │   ├── 001-content-structure.md       # Content format decision
│   │   └── 002-topic-organization.md      # Topic organization decision
│   ├── knowledge/
│   │   ├── patterns/
│   │   │   ├── qa-format.md               # Q&A format pattern
│   │   │   ├── code-examples.md           # Code examples pattern
│   │   │   └── comparison-tables.md       # Comparison tables pattern
│   │   └── anti-patterns/                 # (empty)
│   ├── agents/                            # (empty)
│   └── evolution/
│       └── changelog.md                   # Change history
├── java-fundamentals.md                   # Core Java content
├── java-multithreading.md                 # Concurrency content
├── spring-boot.md                         # Spring content
├── microservices-patterns.md              # Microservices content
├── system-design.md                       # System design content
├── distributed-systems-architecture.md    # Distributed systems content
├── docker-kubernetes.md                   # Container content
├── design-patterns-solid.md               # Design patterns content
├── database-caching.md                    # Database content
└── messaging-event-driven.md              # Messaging content
```

## AI Agent Rules

### Always
- Load context before implementing
- Follow Q&A format pattern for new questions
- Use Java code examples with proper formatting
- Include Table of Contents updates when adding questions
- Follow decisions from `@context/decisions/`
- Use patterns from `@context/knowledge/patterns/`
- Update context after any changes

### Never
- Ignore documented patterns (Q&A format, code examples)
- Mix content between topic files
- Use anti-patterns from `@context/knowledge/anti-patterns/`
- Leave context stale after changes
- Add questions without proper formatting

### After Any Changes (Critical)
AI must update Context Mesh after changes:
- Update relevant feature intent if content added
- Add outcomes to decision files if approach changed
- Update `changelog.md`
- Create `learning-*.md` if significant insights

## Content Guidelines

### Adding New Questions
1. Follow Q&A Format pattern
2. Number questions sequentially
3. Include code examples when applicable
4. Add comparison tables for "vs" questions
5. Update Table of Contents

### Adding New Topics
1. Create new `.md` file at root
2. Create corresponding `feature-*.md` in context
3. Update `project-intent.md` with new feature
4. Update `changelog.md`

### Code Examples
- Use proper Java syntax highlighting
- Include "good" and "bad" examples where appropriate
- Add comments explaining the code
- Keep examples concise but complete

## Definition of Done (Build Phase)

Before completing any content addition:
- [ ] Question follows Q&A format pattern
- [ ] Code examples included (if applicable)
- [ ] Table of Contents updated
- [ ] Feature intent updated (if new section)
- [ ] Changelog updated
- [ ] Patterns followed consistently

---

**Note**: This is a documentation project for interview preparation. The primary "code" is Markdown content following consistent patterns.
