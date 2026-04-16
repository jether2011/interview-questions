# Project Intent: Senior Java Interview Preparation Guide

## What
A concise, dense interview reference for senior Java/backend developers targeting international positions. Content is organized by topic, written in English, with minimal code examples and Mermaid diagrams for architecture topics.

## Why
Interview preparation materials tend to be either too shallow (no depth) or too verbose (hard to skim before an interview). This project aims for the sweet spot: direct answers that demonstrate real understanding, not memorization.

## Design Principles
1. **Direct answers** — no filler, no hedging
2. **Minimal code** — only what's needed to explain the concept
3. **Comparison tables** — for "vs" questions
4. **Mermaid diagrams** — for architecture, flow, and state topics
5. **Interview-ready** — every Q maps to a real interview question

## Current Features
- [Java Fundamentals](feature-java-fundamentals.md) — OOP, Collections, Exceptions, Streams
- [Multithreading](feature-multithreading.md) — Synchronization, thread contention, 10k RPS, race conditions
- [Spring Boot](feature-spring-boot.md) — DI, Bean lifecycle, @Transactional, JPA, OAuth2, Docker, EC2
- [Microservices](feature-microservices.md) — Service Discovery, API Gateway, Saga, CQRS, resilience
- [System Design](feature-system-design.md) — CAP, caching, sharding, RESTful API design
- [Distributed Systems](feature-distributed-systems.md) — Consensus, fault tolerance, Hexagonal Architecture
- [Docker & Kubernetes](feature-docker-kubernetes.md) — Containers, K8s architecture, HPA, probes
- [Design Patterns](feature-design-patterns.md) — SOLID, GoF, enterprise patterns
- [Database & Caching](feature-database-caching.md) — SQL/NoSQL, indexes, isolation, Redis
- [Messaging & EDA](feature-messaging.md) — Kafka, delivery guarantees, Outbox pattern, EDA principles
- [Solidity & Blockchain](feature-solidity-blockchain.md) — EVM transactions E2E, revert, testing, Solidity
- [Kotlin Language](feature-kotlin-language.md) — Coroutines, null safety, Android, Java interop

## Key Topics Added (2026-04-15)
- How to manage transaction E2E on EVM Blockchain (Web3j + nonce + gas + receipt)
- How to test API integration with Blockchain smart contracts (Testcontainers + Foundry)
- What happens when a transaction is reverted
- How to design a queryable RESTful API
- What happens with Java thread contention
- How to handle 10,000 requests in a Java microservice
- How to avoid race conditions in distributed environments

## Status
- **Status**: Active
- **Last updated**: 2026-04-15
- **Format**: Markdown, English, Mermaid diagrams
