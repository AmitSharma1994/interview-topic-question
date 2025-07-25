# Spring Boot Interview Questions and Answers

## Core Concepts

### What is Spring Boot and how is it different from Spring Framework?

**Answer:**
- **Spring Boot** is a project built on top of the Spring Framework that simplifies the setup and development of Spring applications.
- **Key differences:**
  - Spring Boot provides auto-configuration while Spring Framework requires explicit configuration
  - Spring Boot includes embedded servers (Tomcat, Jetty, Undertow) while Spring Framework requires separate server setup
  - Spring Boot uses opinionated defaults with "starter" dependencies while Spring Framework requires manual dependency management
  - Spring Boot minimizes XML configuration and boilerplate code compared to Spring Framework

### What are the key features of Spring Boot?

**Answer:**
- **Auto-configuration:** Automatically configures application based on dependencies
- **Standalone:** Creates self-contained applications with embedded servers
- **Opinionated defaults:** Provides sensible default configurations
- **Spring Boot Starters:** Simplified dependency management
- **Actuator:** Production-ready features for monitoring and management
- **Spring Boot CLI:** Command-line tool for rapid prototyping
- **DevTools:** Development-time features like hot reloading
- **Externalized configuration:** Easy configuration via properties files, YAML, environment variables
- **Production-ready:** Metrics, health checks, and externalized configuration

### List common annotations used in Spring Boot

**Answer:**
- **@SpringBootApplication:** Combines @Configuration, @EnableAutoConfiguration, and @ComponentScan
- **@RestController:** Creates RESTful web services returning domain objects
- **@RequestMapping:** Maps HTTP requests to handler methods
- **@Autowired:** Injects dependencies automatically
- **@Service:** Marks a class as service layer component
- **@Repository:** Marks a class as data access component
- **@Component:** Marks a class as Spring component
- **@Configuration:** Indicates a class defines bean configurations
- **@EnableAutoConfiguration:** Enables Spring Boot's auto-configuration
- **@ComponentScan:** Configures component scanning directives
- **@Value:** Injects values from properties files
- **@Profile:** Configures when beans are loaded based on environment
- **@Conditional:** Conditionally creates beans based on conditions

## Auto-Configuration

### What is Auto-Configuration in Spring Boot?

**Answer:**
- Auto-configuration is a feature that automatically configures beans based on:
  - Classpath dependencies (JARs) available
  - Existing beans in the application context
  - Environment properties
- It reduces boilerplate configuration by providing sensible defaults
- Implemented through conditional annotations like @ConditionalOnClass, @ConditionalOnMissingBean
- Located in spring-boot-autoconfigure.jar
- Can be enabled with @EnableAutoConfiguration or @SpringBootApplication

**Example:**
```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### What is the similarity and difference between IoC container and Auto-Configuration?

**Answer:**
- **Similarities:**
  - Both manage object creation and lifecycle
  - Both reduce manual configuration
  - Both support dependency injection

- **Differences:**
  - IoC container is a core Spring Framework concept for managing beans
  - Auto-configuration is a Spring Boot feature that pre-configures the IoC container
  - IoC requires explicit bean definitions while auto-configuration provides defaults
  - IoC is about dependency injection while auto-configuration is about default configurations

### How to exclude beans from Auto-Configuration?

**Answer:**
There are several ways to exclude auto-configuration:

1. **Using @SpringBootApplication annotation:**
```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class MyApplication {
    // ...
}
```

2. **Using @EnableAutoConfiguration annotation:**
```java
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})
@Configuration
public class MyConfiguration {
    // ...
}
```

3. **Using properties file:**
```properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

## Configuration

### What configuration can be kept in application.properties?

**Answer:**
- **Server properties:** port, context path, SSL settings
- **Database configurations:** URL, username, password, connection pool
- **Logging settings:** Log levels, patterns, file output
- **Security properties:** User credentials, OAuth settings
- **Actuator configurations:** Endpoints, exposure settings
- **Custom application properties:** Any application-specific properties
- **Spring MVC settings:** View resolvers, static resource paths
- **Hibernate/JPA properties:** Dialect, DDL mode, naming strategy

**Example:**
```properties
# Server properties
server.port=8080
server.servlet.context-path=/myapp

# Database properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=user
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update

# Logging properties
logging.level.org.springframework=INFO
logging.level.com.myapp=DEBUG

# Custom properties
app.feature.enabled=true
app.cache.timeout=3600
```

### How to override the embedded Tomcat server settings?

**Answer:**
Tomcat server settings can be overridden through:

1. **application.properties:**
```properties
# Server port
server.port=8090

# Connection timeout
server.connection-timeout=5000

# Max HTTP header size
server.max-http-header-size=8KB

# Max HTTP post size
server.tomcat.max-swallow-size=2MB

# Thread pool settings
server.tomcat.max-threads=200
server.tomcat.min-spare-threads=10

# Access logs
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.directory=logs
server.tomcat.accesslog.pattern=%t %a "%r" %s (%D ms)
```

2. **Programmatically with WebServerFactoryCustomizer:**
```java
@Component
public class TomcatCustomizer implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {
    @Override
    public void customize(TomcatServletWebServerFactory factory) {
        factory.setPort(8090);
        factory.addConnectorCustomizers(connector -> {
            connector.setMaxPostSize(2000000); // 2MB
            connector.setAttribute("maxThreads", 200);
        });
    }
}
```

### How to change server port in Spring Boot?

**Answer:**
There are multiple ways to change the server port:

1. **In application.properties:**
```properties
server.port=8090
```

2. **In application.yml:**
```yaml
server:
  port: 8090
```

3. **Using environment variables:**
```
java -jar myapp.jar --server.port=8090
```

4. **Programmatically:**
```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApplication.class);
        app.setDefaultProperties(Collections.singletonMap("server.port", "8090"));
        app.run(args);
    }
}
```

5. **Using WebServerFactoryCustomizer:**
```java
@Component
public class ServerPortCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        factory.setPort(8090);
    }
}
```

## Annotations

### What is the difference between @Component, @Service, and @Bean?

**Answer:**
- **@Component:**
  - Class-level annotation marking a class as a Spring component
  - Automatically detected through classpath scanning
  - Base stereotype annotation for any Spring-managed component

- **@Service:**
  - Specialization of @Component for service layer
  - Functionally identical to @Component but more semantically specific
  - Indicates the class holds business logic

- **@Bean:**
  - Method-level annotation, not class-level
  - Explicitly declares a single bean, rather than letting Spring auto-detect
  - Must be used within @Configuration classes
  - Gives more control over bean creation and initialization

**Example:**
```java
// Using @Component
@Component
public class UserRepository {
    // Implementation
}

// Using @Service
@Service
public class UserService {
    private final UserRepository repository;
    
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
    // Business logic
}

// Using @Bean
@Configuration
public class AppConfig {
    @Bean
    public PaymentProcessor paymentProcessor() {
        PaymentProcessor processor = new PaymentProcessor();
        processor.setMerchantId("merchant123");
        return processor;
    }
}
```

### How to get values from query parameters, path parameters, headers, and request body?

**Answer:**
- **Query Parameters:** Use @RequestParam
- **Path Parameters:** Use @PathVariable
- **Headers:** Use @RequestHeader
- **Request Body:** Use @RequestBody

**Example:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // Path parameter
    @GetMapping("/{id}")
    public User getUserById(@PathVariable("id") Long userId) {
        return userService.findById(userId);
    }
    
    // Query parameters
    @GetMapping("/search")
    public List<User> searchUsers(
            @RequestParam("name") String name,
            @RequestParam(value = "age", required = false) Integer age) {
        return userService.searchUsers(name, age);
    }
    
    // Request headers
    @GetMapping("/info")
    public String getUserAgent(@RequestHeader("User-Agent") String userAgent) {
        return "Your User-Agent: " + userAgent;
    }
    
    // Request body
    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }
    
    // Multiple types combined
    @PutMapping("/{id}")
    public User updateUser(
            @PathVariable("id") Long id,
            @RequestBody User user,
            @RequestHeader("Authorization") String token) {
        // Validate token
        // Update user
        return userService.update(id, user);
    }
}
```

## Developer Tools

### What are Spring Boot DevTools?

**Answer:**
Spring Boot DevTools is a set of tools that enhances the development experience by providing:

- **Automatic restart:** Restarts the application when classpath changes
- **LiveReload:** Automatically refreshes the browser when resources change
- **Property defaults:** Development-specific property defaults
- **Remote development support:** Remote debugging and updates
- **Development-only features:** Features disabled in production

**How to add DevTools:**
```xml
<!-- Maven -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

### How do they help in development?

**Answer:**
1. **Faster development cycle:**
   - Automatic restart is faster than manual restart
   - Only restarts when necessary (changes to classes)
   - Uses two classloaders to improve restart time

2. **Enhanced productivity:**
   - LiveReload eliminates manual browser refreshes
   - Property defaults optimized for development

3. **Improved debugging:**
   - Remote debugging and development support
   - Global settings for all projects

4. **Development-specific configurations:**
   - Disables template caching
   - Enables debug logging for web groups
   - Excludes static resources from triggering restarts

## Actuator

### What is Spring Boot Actuator?

**Answer:**
Spring Boot Actuator is a sub-project of Spring Boot that adds production-ready features to applications:

- **Monitoring:** Health checks, metrics, and information about the application
- **Management:** Endpoints to manage and interact with the application
- **Auditing:** Tracking events and changes in the application
- **HTTP tracing:** Information about HTTP requests and responses

**How to add Actuator:**
```xml
<!-- Maven -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### How are endpoints exposed in Actuator?

**Answer:**
Actuator endpoints can be exposed via:

1. **HTTP (web):** Accessible via REST endpoints
2. **JMX:** Accessible via JMX beans

**Configuration options:**
```properties
# Enable all endpoints
management.endpoints.web.exposure.include=*

# Enable specific endpoints
management.endpoints.web.exposure.include=health,info,metrics

# Exclude specific endpoints
management.endpoints.web.exposure.exclude=env,beans

# Base path for actuator endpoints (default is /actuator)
management.endpoints.web.base-path=/management

# Enable JMX exposure
management.endpoints.jmx.exposure.include=*
```

**Common endpoints:**
- `/actuator/health`: Application health information
- `/actuator/info`: Application information
- `/actuator/metrics`: Application metrics
- `/actuator/env`: Environment properties
- `/actuator/loggers`: Logger configuration
- `/actuator/mappings`: Request mapping information
- `/actuator/beans`: Spring beans information

### How to change log level using Actuator?

**Answer:**
Log levels can be changed at runtime using the `/actuator/loggers` endpoint:

1. **View current logger configuration:**
   - GET request to `/actuator/loggers/{logger-name}`
   - Example: `GET /actuator/loggers/org.springframework`

2. **Change log level:**
   - POST request to `/actuator/loggers/{logger-name}`
   - With JSON body: `{"configuredLevel": "DEBUG"}`
   - Example: 
   ```
   POST /actuator/loggers/org.springframework.web
   Content-Type: application/json
   
   {"configuredLevel": "DEBUG"}
   ```

**Code example:**
```java
// Using RestTemplate to change log level
RestTemplate restTemplate = new RestTemplate();
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);

Map<String, String> body = Collections.singletonMap("configuredLevel", "DEBUG");
HttpEntity<Map<String, String>> entity = new HttpEntity<>(body, headers);

restTemplate.postForEntity(
    "http://localhost:8080/actuator/loggers/org.springframework.web",
    entity,
    Void.class
);
```

## Exception Handling

### How is exception handling done in Spring Boot?

**Answer:**
Spring Boot provides several ways to handle exceptions:

1. **@ExceptionHandler:** Method-level annotation to handle exceptions in a specific controller

2. **@ControllerAdvice/@RestControllerAdvice:** Global exception handling across multiple controllers

3. **HandlerExceptionResolver:** Interface for programmatic exception resolution

4. **ErrorController:** Custom implementation to handle errors at the Spring Boot level

5. **application.properties configuration:**
```properties
server.error.include-message=always
server.error.include-binding-errors=always
server.error.include-stacktrace=on_param
server.error.include-exception=false
```

**Example using @ExceptionHandler:**
```java
@RestController
public class ProductController {
    
    @GetMapping("/products/{id}")
    public Product getProduct(@PathVariable Long id) {
        // Logic that might throw exceptions
        if (productNotFound) {
            throw new ProductNotFoundException(id);
        }
        return product;
    }
    
    @ExceptionHandler(ProductNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleProductNotFound(ProductNotFoundException ex) {
        return new ErrorResponse("PRODUCT_NOT_FOUND", ex.getMessage());
    }
}
```

**Example using @RestControllerAdvice:**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ProductNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleProductNotFound(ProductNotFoundException ex) {
        return new ErrorResponse("PRODUCT_NOT_FOUND", ex.getMessage());
    }
    
    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(ValidationException ex) {
        return new ErrorResponse("VALIDATION_FAILED", ex.getMessage());
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGenericException(Exception ex) {
        return new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred");
    }
}
```

### How would you handle error responses in a RESTful API to ensure they are informative and user-friendly?

**Answer:**
1. **Create consistent error response structure:**
```java
public class ErrorResponse {
    private String code;
    private String message;
    private List<String> details;
    private LocalDateTime timestamp;
    
    // Constructors, getters, setters
}
```

2. **Use appropriate HTTP status codes:**
   - 400 Bad Request: Client-side validation errors
   - 401 Unauthorized: Authentication required
   - 403 Forbidden: Insufficient permissions
   - 404 Not Found: Resource not found
   - 409 Conflict: Resource conflict
   - 422 Unprocessable Entity: Validation errors
   - 500 Internal Server Error: Server-side errors

3. **Include detailed error information:**
   - Error code (for client-side error handling)
   - Human-readable message
   - Field-level validation errors
   - Timestamp
   - Request ID for tracing

4. **Implement global exception handling:**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationExceptions(MethodArgumentNotValidException ex) {
        List<String> details = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.toList());
            
        return new ErrorResponse(
            "VALIDATION_FAILED",
            "Validation failed",
            details,
            LocalDateTime.now()
        );
    }
    
    // Other exception handlers
}
```

5. **Log errors appropriately:**
   - Log detailed information for server-side errors
   - Include stack traces for unexpected exceptions
   - Use request IDs to correlate logs with responses

6. **Consider internationalization:**
   - Support error messages in multiple languages
   - Use message codes instead of hardcoded strings

## Transactions

### How to manage transactions in Spring Boot?

**Answer:**
Spring Boot provides several ways to manage transactions:

1. **Declarative transaction management with @Transactional:**
```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Transactional
    public User createUser(User user) {
        // All operations within this method will be in a transaction
        User savedUser = userRepository.save(user);
        createUserProfile(savedUser);
        return savedUser;
    }
    
    private void createUserProfile(User user) {
        // This is part of the same transaction
        // If an exception occurs here, the user save will be rolled back
    }
}
```

2. **Programmatic transaction management:**
```java
@Service
public class OrderService {
    
    private final TransactionTemplate transactionTemplate;
    private final OrderRepository orderRepository;
    
    public OrderService(PlatformTransactionManager transactionManager, OrderRepository orderRepository) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
        this.orderRepository = orderRepository;
    }
    
    public Order createOrder(Order order) {
        return transactionTemplate.execute(status -> {
            try {
                Order savedOrder = orderRepository.save(order);
                processPayment(order);
                return savedOrder;
            } catch (PaymentException e) {
                status.setRollbackOnly();
                throw e;
            }
        });
    }
}
```

3. **Configuration in application.properties:**
```properties
# Transaction timeout (in seconds)
spring.transaction.default-timeout=60

# Default transaction propagation
spring.transaction.default-propagation=required
```

### How do transactions work internally in Spring Boot?

**Answer:**
Spring Boot transactions work through the following components and process:

1. **PlatformTransactionManager:**
   - Core interface for transaction management
   - Different implementations for different data sources (JDBC, JPA, etc.)
   - Handles begin, commit, and rollback operations

2. **TransactionDefinition:**
   - Defines transaction properties (isolation, propagation, timeout, etc.)
   - Created from @Transactional annotation or programmatically

3. **TransactionStatus:**
   - Represents current transaction state
   - Allows checking if transaction is completed, new, or rollback-only

4. **AOP (Aspect-Oriented Programming):**
   - @Transactional uses Spring AOP to create proxies
   - Transaction advice intercepts method calls
   - Begins transaction before method execution
   - Commits or rolls back after method completion

5. **Transaction synchronization:**
   - Coordinates resources within a transaction
   - Manages connection acquisition and release
   - Handles transaction callbacks

### What is Spring Boot transaction propagation?

**Answer:**
Transaction propagation defines how transactions relate to each other when methods are called within an existing transaction context:

1. **REQUIRED (default):**
   - Uses current transaction if exists, creates new one if not
   - Most common propagation type

2. **SUPPORTS:**
   - Uses current transaction if exists, executes non-transactionally if not
   - For operations that don't require transactions but can use one

3. **MANDATORY:**
   - Uses current transaction, throws exception if none exists
   - For operations that must be part of existing transaction

4. **REQUIRES_NEW:**
   - Always creates new transaction, suspends current if exists
   - For operations that need isolation from existing transaction

5. **NOT_SUPPORTED:**
   - Executes non-transactionally, suspends current if exists
   - For operations that shouldn't run in a transaction

6. **NEVER:**
   - Executes non-transactionally, throws exception if transaction exists
   - For operations that must not run in a transaction

7. **NESTED:**
   - Creates nested transaction if current exists, new transaction if not
   - Uses savepoints for partial rollback capability

**Example:**
```java
@Service
public class PaymentService {
    
    @Transactional(propagation = Propagation.REQUIRED)
    public void processPayment(Order order) {
        // Uses existing transaction or creates new one
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logPaymentAttempt(Order order) {
        // Always creates new transaction
        // If this fails, it won't affect the parent transaction
    }
    
    @Transactional(propagation = Propagation.NESTED)
    public void applyDiscount(Order order) {
        // Creates nested transaction
        // If this fails, only this part can be rolled back
    }
}
```

### What are isolation levels in Spring Boot transactions?

**Answer:**
Transaction isolation levels control how changes made by one transaction are visible to other concurrent transactions:

1. **DEFAULT:**
   - Uses the default isolation level of the underlying database

2. **READ_UNCOMMITTED:**
   - Lowest isolation level
   - Allows dirty reads, non-repeatable reads, and phantom reads
   - One transaction can see uncommitted changes from another transaction
   - Highest performance, lowest data integrity

3. **READ_COMMITTED:**
   - Prevents dirty reads
   - Allows non-repeatable reads and phantom reads
   - Only sees data that has been committed
   - Default for many databases (PostgreSQL, SQL Server, Oracle)

4. **REPEATABLE_READ:**
   - Prevents dirty reads and non-repeatable reads
   - Allows phantom reads
   - Same query returns same results within a transaction
   - Default for MySQL/MariaDB

5. **SERIALIZABLE:**
   - Highest isolation level
   - Prevents dirty reads, non-repeatable reads, and phantom reads
   - Transactions are completely isolated
   - Lowest performance, highest data integrity

**Example:**
```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public List<Product> getAvailableProducts() {
    // This method will only see committed data
    return productRepository.findByAvailableTrue();
}

@Transactional(isolation = Isolation.SERIALIZABLE)
public void transferFunds(Account from, Account to, BigDecimal amount) {
    // Highest isolation for financial transactions
    from.withdraw(amount);
    to.deposit(amount);
    accountRepository.save(from);
    accountRepository.save(to);
}
```

## Caching

### How to implement cache in Spring Boot?

**Answer:**
Spring Boot provides a caching abstraction that can be implemented in several ways:

1. **Enable caching with @EnableCaching:**
```java
@SpringBootApplication
@EnableCaching
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

2. **Use caching annotations:**
```java
@Service
public class ProductService {
    
    private final ProductRepository repository;
    
    public ProductService(ProductRepository repository) {
        this.repository = repository;
    }
    
    @Cacheable(value = "products", key = "#id")
    public Product getProductById(Long id) {
        // This result will be cached
        return repository.findById(id).orElseThrow();
    }
    
    @CachePut(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        // Updates the cache after updating the product
        return repository.save(product);
    }
    
    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        // Removes the product from cache when deleted
        repository.deleteById(id);
    }
    
    @CacheEvict(value = "products", allEntries = true)
    public void clearProductCache() {
        // Clears the entire cache
    }
}
```

3. **Configure cache manager in application.properties:**
```properties
# Simple cache configuration
spring.cache.type=simple

# Caffeine cache configuration
spring.cache.cache-names=products,categories
spring.cache.caffeine.spec=maximumSize=500,expireAfterAccess=600s

# Redis cache configuration
spring.cache.type=redis
spring.redis.host=localhost
spring.redis.port=6379
```

### What cache providers can be used with Spring Boot?

**Answer:**
Spring Boot supports multiple cache providers:

1. **Simple (ConcurrentHashMap):**
   - Default in-memory caching
   - No additional dependencies required
   - Limited features, suitable for development

2. **Caffeine:**
   - High-performance, in-memory caching library
   - Replacement for Guava cache
   - Features include size-based eviction, time-based eviction, statistics

3. **EhCache:**
   - Popular open-source Java cache
   - Supports disk persistence, distributed caching
   - Extensive configuration options

4. **Redis:**
   - Distributed, in-memory data store
   - Persistence options
   - Supports complex data structures

5. **Hazelcast:**
   - Distributed computing and caching platform
   - Clustering and data partitioning
   - Near-cache capabilities

6. **Infinispan:**
   - Distributed in-memory key/value data store
   - Advanced features like transactions, events, queries
   - Embedded or client-server modes

**Configuration example for Caffeine:**
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(500)
            .expireAfterAccess(10, TimeUnit.MINUTES)
            .recordStats());
        return cacheManager;
    }
}
```

### How to increase the timeout of cache data?

**Answer:**
Cache timeout (expiration) can be configured in several ways depending on the cache provider:

1. **Using properties file (for supported providers):**
```properties
# Caffeine cache with expiration
spring.cache.caffeine.spec=maximumSize=500,expireAfterWrite=30m,expireAfterAccess=15m

# Redis cache with expiration
spring.cache.redis.time-to-live=3600000
```

2. **Programmatic configuration:**
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        // Caffeine configuration
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .expireAfterWrite(30, TimeUnit.MINUTES)
            .expireAfterAccess(15, TimeUnit.MINUTES));
        return cacheManager;
    }
    
    // Alternative: Redis configuration
    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30));
            
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .withCacheConfiguration("products", RedisCacheConfiguration
                .defaultCacheConfig()
                .entryTtl(Duration.ofHours(1)))
            .build();
    }
}
```

3. **Using @Cacheable with custom expiration:**
```java
// For Redis cache
@Cacheable(value = "products", key = "#id", cacheManager = "redisCacheManager")
public Product getProduct(Long id) {
    return productRepository.findById(id).orElse(null);
}
```

### What are cache policies in Spring Boot?

**Answer:**
Cache policies control how the cache behaves in terms of storage, eviction, and expiration:

1. **Eviction policies:**
   - **Size-based:** Remove entries when cache reaches maximum size
   - **Time-based:** Remove entries after a certain time (TTL)
   - **Usage-based:** Remove least recently/frequently used entries
   - **Reference-based:** Remove entries based on garbage collection (weak/soft references)

2. **Cache loading strategies:**
   - **Lazy loading:** Load data into cache on first request
   - **Eager loading:** Pre-populate cache at startup
   - **Refresh ahead:** Automatically refresh before expiration

3. **Write policies:**
   - **Write-through:** Update cache and database simultaneously
   - **Write-behind:** Update cache immediately, database asynchronously
   - **Cache-aside:** Update database directly, invalidate cache

4. **Consistency policies:**
   - **Synchronous updates:** Ensure cache and database consistency
   - **Eventual consistency:** Allow temporary inconsistency for performance

**Example with Caffeine:**
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        
        // Products cache - time-based eviction
        Cache<Object, Object> productsCache = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(1, TimeUnit.HOURS)
            .build();
            
        // User cache - size and usage-based eviction
        Cache<Object, Object> userCache = Caffeine.newBuilder()
            .maximumSize(10000)
            .expireAfterAccess(30, TimeUnit.MINUTES)
            .build();
            
        // Register custom caches
        cacheManager.registerCustomCache("products", productsCache);
        cacheManager.registerCustomCache("users", userCache);
        
        return cacheManager;
    }
}
```

## Security

### How would you secure a Spring Boot application against common vulnerabilities?

**Answer:**
To secure a Spring Boot application against common vulnerabilities:

1. **Input validation and output encoding:**
   - Validate all input data (size, format, range, type)
   - Use Bean Validation (JSR-380) for model validation
   - Encode output to prevent XSS attacks
   - Use prepared statements to prevent SQL injection

2. **Authentication and authorization:**
   - Implement proper authentication mechanisms
   - Use Spring Security for access control
   - Apply principle of least privilege
   - Implement proper session management

3. **HTTPS and transport security:**
   - Force HTTPS for all communications
   - Configure proper TLS/SSL settings
   - Use HTTP security headers

4. **CSRF protection:**
   - Enable CSRF protection (on by default in Spring Security)
   - Use CSRF tokens for state-changing operations

5. **Secure dependencies:**
   - Keep dependencies updated
   - Use dependency vulnerability scanning
   - Remove unused dependencies

6. **Secure configuration:**
   - Externalize sensitive configuration
   - Use environment variables or secure vaults for secrets
   - Apply proper file permissions

7. **Logging and monitoring:**
   - Log security events
   - Implement audit trails
   - Monitor for suspicious activities

**Example configurations:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            .and()
            .headers()
                .contentSecurityPolicy("default-src 'self'; frame-ancestors 'none'")
                .and()
                .frameOptions().deny()
                .xssProtection().block(true)
                .and()
            .authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .antMatchers("/api/**").authenticated()
                .anyRequest().authenticated()
            .and()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.ALWAYS)
                .invalidSessionUrl("/login?invalid")
                .maximumSessions(1);
    }
}
```

```properties
# HTTPS configuration
server.ssl.enabled=true
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=${KEYSTORE_PASSWORD}
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=tomcat

# HTTP security headers
server.servlet.session.cookie.secure=true
server.servlet.session.cookie.http-only=true
server.servlet.session.cookie.same-site=strict
```

### How would you implement authentication and authorization in a Spring Boot application?

**Answer:**
Implementing authentication and authorization in Spring Boot:

1. **Add Spring Security dependency:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

2. **Basic authentication configuration:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user").password("{bcrypt}$2a$10$...").roles("USER")
            .and()
            .withUser("admin").password("{bcrypt}$2a$10$...").roles("USER", "ADMIN");
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/home").permitAll()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
            .logout()
                .permitAll();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

3. **Database authentication:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    private DataSource dataSource;
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.jdbcAuthentication()
            .dataSource(dataSource)
            .usersByUsernameQuery(
                "select username, password, enabled from users where username = ?")
            .authoritiesByUsernameQuery(
                "select username, authority from authorities where username = ?");
    }
}
```

4. **Custom UserDetailsService:**
```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));
            
        return org.springframework.security.core.userdetails.User
            .withUsername(user.getUsername())
            .password(user.getPassword())
            .authorities(mapRolesToAuthorities(user.getRoles()))
            .accountExpired(!user.isActive())
            .accountLocked(!user.isActive())
            .credentialsExpired(!user.isActive())
            .disabled(!user.isActive())
            .build();
    }
    
    private Collection<? extends GrantedAuthority> mapRolesToAuthorities(Collection<Role> roles) {
        return roles.stream()
            .map(role -> new SimpleGrantedAuthority(role.getName()))
            .collect(Collectors.toList());
    }
}
```

5. **Method-level security:**
```java
@Configuration
@EnableGlobalMethodSecurity(
    prePostEnabled = true,
    securedEnabled = true,
    jsr250Enabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
}

@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public List<User> getAllUsers() {
        // Only accessible to admins
        return userRepository.findAll();
    }
    
    @PostAuthorize("returnObject.username == authentication.name or hasRole('ADMIN')")
    public User getUser(Long id) {
        // Only accessible to the user themselves or admins
        return userRepository.findById(id).orElseThrow();
    }
    
    @Secured({"ROLE_USER", "ROLE_ADMIN"})
    public void updateUser(User user) {
        // Accessible to users and admins
    }
    
    @RolesAllowed("ADMIN")
    public void deleteUser(Long id) {
        // Only accessible to admins
    }
}
```

6. **JWT authentication:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    private JwtTokenProvider tokenProvider;
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authorizeRequests()
                .antMatchers("/auth/**").permitAll()
                .anyRequest().authenticated()
            .and()
            .apply(new JwtConfigurer(tokenProvider));
    }
}
```

7. **OAuth2/OIDC authentication:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .oauth2Login()
                .loginPage("/login")
                .defaultSuccessUrl("/home")
                .and()
            .authorizeRequests()
                .antMatchers("/", "/login").permitAll()
                .anyRequest().authenticated();
    }
}
```

```properties
# OAuth2 client configuration
spring.security.oauth2.client.registration.google.client-id=${GOOGLE_CLIENT_ID}
spring.security.oauth2.client.registration.google.client-secret=${GOOGLE_CLIENT_SECRET}
spring.security.oauth2.client.registration.google.scope=openid,profile,email
```