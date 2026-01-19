# Feature: Spring & Spring Boot Content

## What

Comprehensive interview preparation content covering the Spring ecosystem, from core framework concepts to Spring Boot's opinionated approach to application development. This feature provides in-depth Q&A materials on:

- **Spring Core & IoC Container**: Understanding Inversion of Control, the BeanFactory and ApplicationContext, bean lifecycle, and how Spring manages object creation and dependency wiring through its powerful container.

- **Dependency Injection Patterns**: Constructor injection (recommended), setter injection, and field injection trade-offs; understanding @Autowired, @Qualifier, and @Primary annotations; and why loose coupling through DI improves testability.

- **Spring Boot Auto-Configuration**: How Spring Boot eliminates boilerplate through @EnableAutoConfiguration, starter dependencies, and sensible defaults; customizing auto-configuration and understanding the conditional annotation system.

- **Web Development**: Spring MVC architecture (DispatcherServlet, controllers, view resolvers), building RESTful APIs with @RestController, request/response handling, validation, and exception handling strategies.

- **Data Access**: Spring Data JPA repositories, query methods, custom queries with @Query, transaction management with @Transactional, and connection pooling configuration.

- **Security**: Spring Security authentication and authorization, SecurityFilterChain configuration, JWT integration, OAuth2, and method-level security with @PreAuthorize.

## Why

Spring/Spring Boot dominates enterprise Java development. Interview focus areas include:

1. **Industry Standard**: 70%+ of Java job postings require Spring experience
2. **Architectural Understanding**: Shows you can build production-grade applications
3. **Best Practices**: Spring encourages patterns (DI, AOP) that improve code quality
4. **Full-Stack Knowledge**: From web layer to data access to security

This content helps developers understand Spring's philosophy and make informed decisions about which features to use and how to configure them properly.

## Acceptance Criteria
- [ ] Covers Spring Core concepts and IoC
- [ ] Explains dependency injection patterns
- [ ] Documents Spring Boot auto-configuration
- [ ] Covers common annotations and their usage
- [ ] Explains Spring MVC and REST API development
- [ ] Documents Spring Data JPA patterns
- [ ] Covers Spring Security authentication/authorization
- [ ] Explains AOP concepts and usage
- [ ] Includes microservices patterns with Spring Cloud

## Content Sections

### 1. Spring Core Concepts
Explains what Spring Framework is and its advantages (lightweight, loose coupling, declarative programming). Covers the Spring module ecosystem (Core, Web, Data, Security, Cloud) and how they work together. Foundation for understanding why Spring became the de facto standard.

### 2. Dependency Injection & IoC
Deep dive into IoC principle and its benefits, the Spring container (BeanFactory vs ApplicationContext), and bean definition (XML, annotations, Java config). Covers all injection types with recommendations: constructor injection for required dependencies, setter for optional. Explains @Autowired behavior, @Qualifier for disambiguation, and @Primary for defaults.

### 3. Spring Boot Fundamentals
How Spring Boot simplifies Spring development through auto-configuration, starter dependencies, and embedded servers. Covers @SpringBootApplication (combining @Configuration, @EnableAutoConfiguration, @ComponentScan), application.properties/yml configuration, profiles for environment-specific settings, and the Actuator for production monitoring.

### 4. Annotations
Comprehensive annotation reference: stereotype annotations (@Component, @Service, @Repository, @Controller), configuration (@Configuration, @Bean, @Value, @ConfigurationProperties), web (@RequestMapping, @GetMapping, @PostMapping, @RequestBody, @PathVariable), and scope annotations (@Scope, @RequestScope, @SessionScope).

### 5. Spring MVC & REST
Explains MVC architecture with DispatcherServlet as front controller, handler mappings, and view resolution. Covers building REST APIs: @RestController vs @Controller + @ResponseBody, content negotiation, request/response serialization with Jackson, validation with @Valid and Bean Validation, and global exception handling with @ControllerAdvice.

### 6. Spring Data JPA
Repository pattern implementation with Spring Data: CrudRepository, JpaRepository, PagingAndSortingRepository interfaces. Covers query derivation from method names, @Query for custom JPQL/native queries, Specifications for dynamic queries, and auditing with @CreatedDate/@LastModifiedDate. Explains N+1 problem and solutions (fetch joins, @EntityGraph).

### 7. Spring Security
Security fundamentals: authentication (who you are) vs authorization (what you can do). Covers SecurityFilterChain configuration, UserDetailsService for custom user loading, password encoding, JWT authentication flow, OAuth2 client/resource server setup, and method security (@PreAuthorize, @PostAuthorize, @Secured).

### 8. Spring AOP
Aspect-Oriented Programming concepts: cross-cutting concerns, aspects, join points, pointcuts, and advice types (before, after, around). Practical applications: logging, transaction management, security, caching. Explains proxy-based AOP (JDK dynamic proxies vs CGLIB) and limitations.

### 9. Microservices with Spring
Spring Cloud components for microservices: service discovery (Eureka), client-side load balancing, API Gateway (Spring Cloud Gateway), distributed configuration (Config Server), circuit breakers (Resilience4j), and distributed tracing (Sleuth/Zipkin). Overview of patterns implemented by Spring Cloud.

### 10. Testing in Spring
Testing strategies: @SpringBootTest for integration tests, @WebMvcTest for controller layer, @DataJpaTest for repository layer, @MockBean for mocking dependencies. Covers TestRestTemplate for REST endpoint testing and best practices for test organization.

### 11. Performance & Best Practices
Production optimization: connection pooling (HikariCP), caching with @Cacheable, async processing with @Async, and lazy initialization. Covers monitoring with Actuator endpoints, health checks, and metrics exposure for Prometheus/Grafana.

## Key Concepts to Master

Critical topics for Spring interviews:

- **Bean lifecycle**: Creation, initialization (@PostConstruct), use, destruction (@PreDestroy)
- **Bean scopes**: singleton (default), prototype, request, session - know when to use each
- **@Transactional**: Propagation levels, isolation levels, rollback rules, and proxy limitations
- **Spring Security filter chain**: Understand the order and purpose of security filters
- **Auto-configuration conditions**: @ConditionalOnClass, @ConditionalOnProperty, @ConditionalOnMissingBean

## Study Tips

1. **Build a complete application** - REST API with database, security, and tests
2. **Read auto-configuration classes** - Understand how Spring Boot makes decisions
3. **Debug the container** - Use breakpoints to see bean creation and dependency injection
4. **Know the defaults** - What happens without explicit configuration
5. **Understand proxies** - Why @Transactional doesn't work on private methods or self-calls

## Related
- [Project Intent](project-intent.md)
- [Feature: Microservices](feature-microservices.md)
- [Decision: Content Structure](../decisions/001-content-structure.md)
- [Pattern: Q&A Format](../knowledge/patterns/qa-format.md)
- [Pattern: Code Examples](../knowledge/patterns/code-examples.md)

## Status
- **Created**: 2026-01-19 (Phase: Intent)
- **Updated**: 2026-01-19 - Enhanced content for better study experience
- **Status**: Active (already implemented)
- **File**: `spring-boot.md`
