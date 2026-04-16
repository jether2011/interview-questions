# AGENTS.md

## Setup
No installation required. All content is Markdown.

## Content Style Rules

### Writing Principles
- **Direct answers** — no filler, no "It's important to note that..."
- **Minimal code** — only what illustrates the concept; keep it under 20 lines
- **English only** — all content in English
- **No hallucination** — only write what you know is technically correct

### Format Conventions
- Questions as `###` headings (natural language, not numbered)
- Code blocks with language tag: ` ```java ` / ` ```solidity ` / ` ```yaml `
- Comparison tables for "vs" questions
- Mermaid diagrams for architecture, flow, state machine, sequence topics
- Short, scannable answers — not paragraphs of prose

### When to Use Mermaid
- Sequence diagrams: auth flows, distributed transactions, event flows
- Flowcharts: architecture components, decision trees, request paths
- State diagrams: circuit breaker states, lifecycle stages
- Always prefer Mermaid over ASCII art

## Context Files to Load

Before any work:
- `@context/intent/project-intent.md` (always)
- `@context/intent/feature-*.md` (for the specific topic)
- `@context/evolution/changelog.md` (to understand recent changes)

## Project Structure

```
root/
├── AGENTS.md                              # This file
├── README.md                              # Project overview
├── context/
│   ├── intent/
│   │   ├── project-intent.md
│   │   └── feature-*.md
│   ├── decisions/
│   │   └── 00N-*.md
│   ├── knowledge/patterns/
│   └── evolution/
│       └── changelog.md
├── java-fundamentals.md
├── java-multithreading.md
├── spring-boot.md
├── microservices-patterns.md
├── system-design.md
├── distributed-systems-architecture.md
├── docker-kubernetes.md
├── design-patterns-solid.md
├── database-caching.md
├── messaging-event-driven.md
├── solidity-blockchain.md
└── kotlin-language.md
```

## AI Agent Rules

### Always
- Read context files before making changes
- Keep code examples minimal and correct
- Use Mermaid for architectural diagrams
- Update changelog.md after any changes
- Add new Q&A to the correct file (don't mix topics)

### Never
- Write verbose explanations — be direct
- Include hallucinated or uncertain content
- Use numbered questions (use natural `###` headings)
- Add emojis or decorative formatting
- Skip updating context files after changes

### After Any Changes
1. Update `context/evolution/changelog.md`
2. Update `context/intent/feature-<topic>.md` if new content added
3. Update `README.md` if new files or significant restructuring

## Definition of Done

- [ ] Answer is direct and technically correct
- [ ] Code example is minimal and compiles
- [ ] Mermaid diagrams used where appropriate
- [ ] No verbose preamble or filler
- [ ] Changelog updated
