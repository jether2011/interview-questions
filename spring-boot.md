# Spring & Spring Boot Interview Questions

## Table of Contents
1. [Spring Core Concepts](#spring-core-concepts)
2. [Dependency Injection & IoC](#dependency-injection--ioc)
3. [Spring Boot Fundamentals](#spring-boot-fundamentals)
4. [Annotations](#annotations)
5. [Spring MVC & REST](#spring-mvc--rest)
6. [Spring Data JPA](#spring-data-jpa)
7. [Spring Security](#spring-security)
8. [Spring AOP](#spring-aop)
9. [Microservices with Spring](#microservices-with-spring)
10. [Testing in Spring](#testing-in-spring)
11. [Performance & Best Practices](#performance--best-practices)

## Spring Core Concepts

### Q1: What is Spring Framework?
**Answer:**
Spring is a comprehensive framework for enterprise Java development that provides:
- **Inversion of Control (IoC)** container
- **Aspect-Oriented Programming (AOP)** support
- **Data Access** abstraction
- **Transaction Management**
- **MVC Framework** for web applications
- **Security Framework**
- **Integration** with other frameworks

### Q2: What are the advantages of Spring Framework?
**Answer:**
1. **Lightweight**: Uses POJO-based programming
2. **Loose Coupling**: Through dependency injection
3. **Declarative Programming**: Via annotations and XML
4. **AOP Support**: Separation of cross-cutting concerns
5. **Transaction Management**: Declarative transaction handling
6. **Testing**: Easy unit and integration testing
7. **Integration**: Works well with other frameworks

### Q3: What are the different modules in Spring Framework?
**Answer:**
- **Core Container**: Core, Beans, Context, SpEL
- **Data Access/Integration**: JDBC, ORM, OXM, JMS, Transactions
- **Web**: WebSocket, Servlet, Web, Portlet
- **AOP and Instrumentation**: AOP, Aspects, Instrumentation
- **Test**: Unit and integration testing support

## Dependency Injection & IoC

### Q4: Explain Inversion of Control (IoC) and Dependency Injection (DI).
**Answer:**
**IoC**: Design principle where control of object creation and lifecycle is inverted from the application to a container/framework.

**DI**: Implementation of IoC where dependencies are injected into objects rather than objects creating their dependencies.

**Types of DI:**
1. **Constructor Injection**: Dependencies via constructor
2. **Setter Injection**: Dependencies via setter methods
3. **Field Injection**: Direct field injection (not recommended)

```java
// Constructor Injection (Recommended)
@Component
public class UserService {
    private final UserRepository repository;
    
    @Autowired
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}

// Setter Injection
@Component
public class UserService {
    private UserRepository repository;
    
    @Autowired
    public void setRepository(UserRepository repository) {
        this.repository = repository;
    }
}

// Field Injection (Not Recommended)
@Component
public class UserService {
    @Autowired
    private UserRepository repository;
}
```

### Q5: What is the Spring IoC container?
**Answer:**
The IoC container is responsible for:
- Instantiating and configuring objects (beans)
- Managing bean lifecycle
- Assembling dependencies between beans
- Providing beans to the application

Two types of containers:
1. **BeanFactory**: Basic container, lazy initialization
2. **ApplicationContext**: Advanced container with more features (preferred)

### Q6: What are the different types of ApplicationContext?
**Answer:**
1. **ClassPathXmlApplicationContext**: Loads from classpath XML
2. **FileSystemXmlApplicationContext**: Loads from file system XML
3. **AnnotationConfigApplicationContext**: Java-based configuration
4. **WebApplicationContext**: Web application specific

```java
// Annotation-based
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

// XML-based
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

### Q7: Explain Bean lifecycle in Spring.
**Answer:**
1. **Instantiation**: Container creates bean instance
2. **Populate Properties**: Dependency injection occurs
3. **setBeanName()**: If BeanNameAware implemented
4. **setBeanFactory()**: If BeanFactoryAware implemented
5. **setApplicationContext()**: If ApplicationContextAware implemented
6. **@PostConstruct** or afterPropertiesSet(): Custom initialization
7. **Bean Ready**: Bean is ready for use
8. **@PreDestroy** or destroy(): Custom destruction
9. **Bean Destroyed**: Container shuts down

### Q8: What are the different bean scopes in Spring?
**Answer:**
1. **singleton** (default): One instance per Spring container
2. **prototype**: New instance for each request
3. **request**: One instance per HTTP request (web only)
4. **session**: One instance per HTTP session (web only)
5. **application**: One instance per ServletContext (web only)
6. **websocket**: One instance per WebSocket session

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    // New instance created each time
}
```

## Spring Boot Fundamentals

### Q9: What is Spring Boot and its advantages?
**Answer:**
Spring Boot is an opinionated framework that simplifies Spring application development.

**Advantages:**
1. **Auto-configuration**: Automatic configuration based on dependencies
2. **Starter Dependencies**: Pre-configured dependency templates
3. **Embedded Servers**: Tomcat, Jetty, or Undertow
4. **Production Ready**: Metrics, health checks, externalized config
5. **No XML Configuration**: Annotation-based configuration
6. **Spring Boot CLI**: Command-line tool for quick prototyping

### Q10: Explain @SpringBootApplication annotation.
**Answer:**
`@SpringBootApplication` combines three annotations:
1. **@Configuration**: Marks class as configuration source
2. **@EnableAutoConfiguration**: Enables auto-configuration
3. **@ComponentScan**: Enables component scanning

```java
@SpringBootApplication
// Equivalent to:
// @Configuration
// @EnableAutoConfiguration
// @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### Q11: How does Spring Boot auto-configuration work?
**Answer:**
1. **@EnableAutoConfiguration** triggers auto-configuration
2. Scans classpath for libraries
3. Uses `@Conditional` annotations to determine what to configure
4. Loads configuration from `spring-boot-autoconfigure` JAR
5. Can be customized or disabled via properties

```properties
# Disable specific auto-configuration
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

### Q12: What are Spring Boot Starters?
**Answer:**
Starters are dependency descriptors that include all required dependencies for a functionality.

Common starters:
- `spring-boot-starter-web`: Web applications
- `spring-boot-starter-data-jpa`: JPA with Hibernate
- `spring-boot-starter-security`: Spring Security
- `spring-boot-starter-test`: Testing dependencies
- `spring-boot-starter-actuator`: Production monitoring

## Annotations

### Q13: Explain common Spring annotations.
**Answer:**

**Core Annotations:**
- `@Component`: Generic stereotype for any Spring-managed component
- `@Service`: Specialization for service layer
- `@Repository`: Specialization for persistence layer
- `@Controller`: Specialization for presentation layer
- `@Configuration`: Java-based configuration class

**Dependency Injection:**
- `@Autowired`: Automatic dependency injection
- `@Qualifier`: Specify which bean to inject
- `@Primary`: Preferred bean when multiple candidates
- `@Value`: Inject values from properties

**Web Annotations:**
- `@RestController`: @Controller + @ResponseBody
- `@RequestMapping`: Map HTTP requests
- `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`
- `@RequestBody`: Bind request body to method parameter
- `@ResponseBody`: Bind return value to response body
- `@PathVariable`: Extract values from URI
- `@RequestParam`: Extract query parameters

### Q14: What is the difference between @Component, @Service, @Repository, and @Controller?
**Answer:**
All are specializations of `@Component` with semantic differences:

| Annotation | Layer | Purpose | Additional Features |
|------------|-------|---------|-------------------|
| @Component | Any | Generic component | Basic Spring bean |
| @Service | Service | Business logic | No additional features |
| @Repository | Persistence | Data access | Exception translation |
| @Controller | Presentation | Web controllers | Request handling |

### Q15: Explain @Conditional annotations.
**Answer:**
Conditional annotations control bean registration based on conditions:

```java
@Configuration
public class ConditionalConfig {
    
    @Bean
    @ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
    public FeatureService featureService() {
        return new FeatureService();
    }
    
    @Bean
    @ConditionalOnMissingBean(DataSource.class)
    public DataSource dataSource() {
        return new EmbeddedDataSource();
    }
    
    @Bean
    @ConditionalOnClass(name = "com.mysql.jdbc.Driver")
    public DataSource mysqlDataSource() {
        return new MySQLDataSource();
    }
}
```

## Spring MVC & REST

### Q16: How does Spring MVC work?
**Answer:**
1. **DispatcherServlet** receives request
2. **HandlerMapping** determines controller
3. **HandlerAdapter** invokes controller method
4. Controller returns **ModelAndView**
5. **ViewResolver** resolves view name
6. View renders response

### Q17: How to create RESTful web services in Spring Boot?
**Answer:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        User saved = userService.save(user);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(saved.getId())
            .toUri();
        return ResponseEntity.created(location).body(saved);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, 
                                          @Valid @RequestBody User user) {
        return userService.update(id, user)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Q18: How to handle exceptions in Spring Boot?
**Answer:**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
            ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(
            MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            errors
        );
        return ResponseEntity.badRequest().body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal server error",
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                           .body(error);
    }
}
```

### Q19: How to implement versioning in REST APIs?
**Answer:**
**1. URI Versioning:**
```java
@RestController
public class UserController {
    @GetMapping("/api/v1/users")
    public List<UserV1> getUsersV1() { }
    
    @GetMapping("/api/v2/users")
    public List<UserV2> getUsersV2() { }
}
```

**2. Request Parameter Versioning:**
```java
@GetMapping(value = "/users", params = "version=1")
public List<UserV1> getUsersV1() { }

@GetMapping(value = "/users", params = "version=2")
public List<UserV2> getUsersV2() { }
```

**3. Header Versioning:**
```java
@GetMapping(value = "/users", headers = "X-API-VERSION=1")
public List<UserV1> getUsersV1() { }

@GetMapping(value = "/users", headers = "X-API-VERSION=2")
public List<UserV2> getUsersV2() { }
```

**4. Content Negotiation:**
```java
@GetMapping(value = "/users", 
           produces = "application/vnd.company.app-v1+json")
public List<UserV1> getUsersV1() { }

@GetMapping(value = "/users", 
           produces = "application/vnd.company.app-v2+json")
public List<UserV2> getUsersV2() { }
```

## Spring Data JPA

### Q20: What is Spring Data JPA?
**Answer:**
Spring Data JPA provides repository support for JPA, simplifying data access layer implementation:
- Repository interfaces with CRUD operations
- Query method generation from method names
- Custom queries with @Query
- Pagination and sorting support
- Auditing capabilities

### Q21: Explain different types of Spring Data repositories.
**Answer:**
```java
// Base repository (no methods)
Repository<T, ID>

// CRUD operations
CrudRepository<T, ID>
  - save(), findById(), findAll(), delete()

// Pagination and sorting
PagingAndSortingRepository<T, ID>
  - findAll(Sort), findAll(Pageable)

// JPA specific (flushing, batch operations)
JpaRepository<T, ID>
  - flush(), saveAndFlush(), deleteInBatch()
```

### Q22: How to create custom queries in Spring Data JPA?
**Answer:**
**1. Query Methods:**
```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Derived query
    List<User> findByEmailAndActive(String email, boolean active);
    
    // Query by example
    List<User> findByEmailContainingIgnoreCase(String email);
}
```

**2. @Query Annotation:**
```java
public interface UserRepository extends JpaRepository<User, Long> {
    // JPQL
    @Query("SELECT u FROM User u WHERE u.email = ?1")
    User findByEmail(String email);
    
    // Native SQL
    @Query(value = "SELECT * FROM users WHERE email = ?1", 
           nativeQuery = true)
    User findByEmailNative(String email);
    
    // Named parameters
    @Query("SELECT u FROM User u WHERE u.firstName = :firstName")
    List<User> findByFirstName(@Param("firstName") String firstName);
    
    // Modifying queries
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
    void deactivateUsersNotLoggedInSince(@Param("date") LocalDate date);
}
```

### Q23: How to implement pagination and sorting?
**Answer:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserRepository userRepository;
    
    @GetMapping
    public Page<User> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "ASC") String direction) {
        
        Sort.Direction sortDirection = Sort.Direction.fromString(direction);
        Pageable pageable = PageRequest.of(page, size, 
                                          Sort.by(sortDirection, sortBy));
        return userRepository.findAll(pageable);
    }
}
```

### Q24: Explain JPA entity relationships.
**Answer:**
```java
// One-to-One
@Entity
public class User {
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id")
    private Profile profile;
}

// One-to-Many / Many-to-One
@Entity
public class Department {
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Employee> employees;
}

@Entity
public class Employee {
    @ManyToOne
    @JoinColumn(name = "dept_id")
    private Department department;
}

// Many-to-Many
@Entity
public class Student {
    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses;
}
```

## Spring Security

### Q25: How does Spring Security work?
**Answer:**
Spring Security uses a filter chain to intercept requests:
1. **DelegatingFilterProxy**: Delegates to Spring Security filter chain
2. **FilterChainProxy**: Manages security filter chain
3. **Security Filters**: Authentication, authorization, CSRF, etc.

Key components:
- **Authentication**: Who are you?
- **Authorization**: What can you do?
- **Principal**: Currently authenticated user
- **GrantedAuthority**: Permissions/roles
- **SecurityContext**: Holds authentication information

### Q26: How to configure Spring Security?
**Answer:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login?logout")
                .permitAll()
            )
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            );
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

### Q27: How to implement JWT authentication?
**Answer:**
```java
@Component
public class JwtTokenProvider {
    
    @Value("${jwt.secret}")
    private String jwtSecret;
    
    @Value("${jwt.expiration}")
    private int jwtExpiration;
    
    public String generateToken(Authentication authentication) {
        UserPrincipal userPrincipal = (UserPrincipal) authentication.getPrincipal();
        Date expiryDate = new Date(System.currentTimeMillis() + jwtExpiration);
        
        return Jwts.builder()
                .setSubject(userPrincipal.getUsername())
                .setIssuedAt(new Date())
                .setExpiration(expiryDate)
                .signWith(SignatureAlgorithm.HS512, jwtSecret)
                .compact();
    }
    
    public String getUsernameFromToken(String token) {
        Claims claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token)
                .getBody();
        return claims.getSubject();
    }
    
    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }
}

@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Autowired
    private JwtTokenProvider tokenProvider;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                  HttpServletResponse response,
                                  FilterChain filterChain) 
                                  throws ServletException, IOException {
        String token = getTokenFromRequest(request);
        
        if (token != null && tokenProvider.validateToken(token)) {
            String username = tokenProvider.getUsernameFromToken(token);
            // Set authentication in context
        }
        
        filterChain.doFilter(request, response);
    }
}
```

## Spring AOP

### Q28: Explain Aspect-Oriented Programming in Spring.
**Answer:**
AOP enables separation of cross-cutting concerns from business logic.

**Key Concepts:**
- **Aspect**: Module containing cross-cutting concern
- **Join Point**: Point in program execution (method call, exception)
- **Advice**: Action taken at join point
- **Pointcut**: Expression matching join points
- **Target Object**: Object being advised
- **Weaving**: Linking aspects with objects

### Q29: What are different types of advice?
**Answer:**
```java
@Aspect
@Component
public class LoggingAspect {
    
    // Before advice
    @Before("@annotation(Loggable)")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Before: " + joinPoint.getSignature());
    }
    
    // After returning advice
    @AfterReturning(pointcut = "@annotation(Loggable)", 
                   returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("Returned: " + result);
    }
    
    // After throwing advice
    @AfterThrowing(pointcut = "@annotation(Loggable)", 
                  throwing = "error")
    public void logAfterThrowing(JoinPoint joinPoint, Throwable error) {
        System.out.println("Exception: " + error);
    }
    
    // After (finally) advice
    @After("@annotation(Loggable)")
    public void logAfter(JoinPoint joinPoint) {
        System.out.println("After: " + joinPoint.getSignature());
    }
    
    // Around advice
    @Around("@annotation(Timed)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) 
                                 throws Throwable {
        long start = System.currentTimeMillis();
        Object proceed = joinPoint.proceed();
        long executionTime = System.currentTimeMillis() - start;
        System.out.println(joinPoint.getSignature() + 
                         " executed in " + executionTime + "ms");
        return proceed;
    }
}
```

### Q30: How to write pointcut expressions?
**Answer:**
```java
@Aspect
@Component
public class SecurityAspect {
    
    // Match specific method
    @Pointcut("execution(public * com.example.service.UserService.findById(..))")
    public void findByIdMethod() {}
    
    // Match all methods in a package
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void allServiceMethods() {}
    
    // Match methods with annotation
    @Pointcut("@annotation(org.springframework.security.access.prepost.PreAuthorize)")
    public void securedMethods() {}
    
    // Match classes with annotation
    @Pointcut("@within(org.springframework.stereotype.Service)")
    public void allServiceClasses() {}
    
    // Combine pointcuts
    @Pointcut("allServiceMethods() && securedMethods()")
    public void securedServiceMethods() {}
    
    // Match method arguments
    @Pointcut("execution(* *..find*(Long))")
    public void findMethodsWithLongParam() {}
}
```

## Microservices with Spring

### Q31: How to create microservices with Spring Boot?
**Answer:**
Key components for microservices:
1. **Service Discovery**: Eureka, Consul
2. **API Gateway**: Spring Cloud Gateway
3. **Configuration**: Spring Cloud Config
4. **Circuit Breaker**: Resilience4j
5. **Tracing**: Sleuth + Zipkin
6. **Message Bus**: RabbitMQ, Kafka

```java
// Eureka Server
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

// Microservice with Eureka Client
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

// application.yml
spring:
  application:
    name: user-service
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

### Q32: How to implement inter-service communication?
**Answer:**
**1. REST Template:**
```java
@Configuration
public class RestTemplateConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class OrderService {
    @Autowired
    private RestTemplate restTemplate;
    
    public User getUser(Long userId) {
        return restTemplate.getForObject(
            "http://user-service/users/" + userId, 
            User.class
        );
    }
}
```

**2. Feign Client:**
```java
@FeignClient(name = "user-service")
public interface UserClient {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable("id") Long id);
}

@Service
public class OrderService {
    @Autowired
    private UserClient userClient;
    
    public User getUser(Long userId) {
        return userClient.getUser(userId);
    }
}
```

### Q33: How to implement Circuit Breaker pattern?
**Answer:**
Using Resilience4j:
```java
@Service
public class UserService {
    
    @CircuitBreaker(name = "user-service", fallbackMethod = "getDefaultUser")
    @Retry(name = "user-service")
    @Bulkhead(name = "user-service")
    public User getUser(Long id) {
        // Call to external service
        return userClient.getUser(id);
    }
    
    public User getDefaultUser(Long id, Exception ex) {
        // Fallback method
        return new User(id, "Default User");
    }
}

// application.yml
resilience4j:
  circuitbreaker:
    instances:
      user-service:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10000
        permitted-number-of-calls-in-half-open-state: 3
  retry:
    instances:
      user-service:
        max-attempts: 3
        wait-duration: 1000
  bulkhead:
    instances:
      user-service:
        max-concurrent-calls: 10
```

## Testing in Spring

### Q34: How to write unit tests in Spring Boot?
**Answer:**
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testFindById() {
        // Given
        User user = new User(1L, "John Doe");
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));
        
        // When
        Optional<User> result = userService.findById(1L);
        
        // Then
        assertTrue(result.isPresent());
        assertEquals("John Doe", result.get().getName());
        verify(userRepository, times(1)).findById(1L);
    }
}
```

### Q35: How to write integration tests?
**Answer:**
```java
@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(locations = "classpath:application-test.properties")
class UserControllerIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    @Transactional
    @Rollback
    void testCreateUser() throws Exception {
        String userJson = "{\"name\":\"John Doe\",\"email\":\"john@example.com\"}";
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(userJson))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.name").value("John Doe"))
                .andExpect(jsonPath("$.email").value("john@example.com"));
        
        assertEquals(1, userRepository.count());
    }
    
    @Test
    void testGetUser() throws Exception {
        User user = userRepository.save(new User("Jane Doe", "jane@example.com"));
        
        mockMvc.perform(get("/api/users/{id}", user.getId()))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("Jane Doe"));
    }
}
```

### Q36: How to test JPA repositories?
**Answer:**
```java
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void testFindByEmail() {
        // Given
        User user = new User("John Doe", "john@example.com");
        entityManager.persistAndFlush(user);
        
        // When
        Optional<User> found = userRepository.findByEmail("john@example.com");
        
        // Then
        assertTrue(found.isPresent());
        assertEquals("John Doe", found.get().getName());
    }
    
    @Test
    void testCustomQuery() {
        // Setup test data
        entityManager.persist(new User("Active User", "active@example.com", true));
        entityManager.persist(new User("Inactive User", "inactive@example.com", false));
        entityManager.flush();
        
        // Test
        List<User> activeUsers = userRepository.findByActive(true);
        assertEquals(1, activeUsers.size());
        assertEquals("Active User", activeUsers.get(0).getName());
    }
}
```

## Performance & Best Practices

### Q37: How to optimize Spring Boot application performance?
**Answer:**
1. **JVM Tuning**: Appropriate heap size, GC tuning
2. **Connection Pooling**: HikariCP configuration
3. **Caching**: Spring Cache abstraction
4. **Lazy Loading**: Lazy initialization of beans
5. **Database Optimization**: Indexes, query optimization
6. **Async Processing**: @Async for non-blocking operations
7. **Response Compression**: Enable GZIP compression

```java
// Caching
@Service
public class UserService {
    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        return userRepository.findById(id).orElse(null);
    }
    
    @CacheEvict(value = "users", key = "#user.id")
    public void updateUser(User user) {
        userRepository.save(user);
    }
}

// Async processing
@Service
@EnableAsync
public class EmailService {
    @Async
    public CompletableFuture<String> sendEmail(String to) {
        // Send email asynchronously
        return CompletableFuture.completedFuture("Email sent");
    }
}
```

### Q38: What are Spring Boot best practices?
**Answer:**
1. **Use constructor injection** over field injection
2. **Follow RESTful conventions** for APIs
3. **Use profiles** for environment-specific configuration
4. **Implement proper exception handling**
5. **Use validation annotations** (@Valid, @NotNull)
6. **Enable actuator** for monitoring
7. **Externalize configuration**
8. **Write comprehensive tests**
9. **Use logging levels appropriately**
10. **Document APIs with OpenAPI/Swagger**

### Q39: How to monitor Spring Boot applications?
**Answer:**
**Spring Boot Actuator:**
```java
// pom.xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

// application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true

// Custom health indicator
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        if (isDatabaseHealthy()) {
            return Health.up()
                .withDetail("database", "Available")
                .build();
        }
        return Health.down()
            .withDetail("database", "Not Available")
            .build();
    }
}
```

### Q40: How to handle transactions in Spring?
**Answer:**
```java
@Service
@Transactional
public class TransferService {
    
    @Transactional(propagation = Propagation.REQUIRED)
    public void transfer(Long fromAccount, Long toAccount, BigDecimal amount) {
        Account from = accountRepository.findById(fromAccount).orElseThrow();
        Account to = accountRepository.findById(toAccount).orElseThrow();
        
        from.debit(amount);
        to.credit(amount);
        
        accountRepository.save(from);
        accountRepository.save(to);
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void auditTransfer(TransferDetails details) {
        // Always create new transaction for audit
        auditRepository.save(new AuditLog(details));
    }
    
    @Transactional(readOnly = true)
    public Account getAccount(Long id) {
        return accountRepository.findById(id).orElseThrow();
    }
    
    @Transactional(rollbackFor = Exception.class, 
                  noRollbackFor = BusinessException.class)
    public void complexOperation() {
        // Specific rollback rules
    }
}
```

**Transaction Propagation Levels:**
- **REQUIRED**: Use existing or create new (default)
- **REQUIRES_NEW**: Always create new transaction
- **SUPPORTS**: Use existing if available
- **MANDATORY**: Must run within existing transaction
- **NOT_SUPPORTED**: Suspend current transaction
- **NEVER**: Must not run within transaction
- **NESTED**: Create savepoint in existing transaction