# Spring Framework Interview Questions and Answers

## Dependency Injection

### What is IoC container, DI, and inversion of control?

**Answer:**
- **Inversion of Control (IoC):** A design principle where control over program flow is transferred to a framework. Instead of your code calling the framework, the framework calls your code.

- **Dependency Injection (DI):** A specific implementation of IoC where dependencies are "injected" into objects rather than objects creating their dependencies.

- **IoC Container:** The core component of Spring that manages the lifecycle of beans and injects dependencies. It creates objects, wires them together, configures them, and manages their lifecycle from creation to destruction.

**Key benefits:**
- Reduced coupling between classes
- Easier unit testing through mock dependencies
- Modular design
- Easier configuration management

**Example:**
```java
// Without DI - tight coupling
public class UserService {
    private UserRepository userRepository = new MySqlUserRepository();
    
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}

// With DI - loose coupling
public class UserService {
    private final UserRepository userRepository;
    
    // Dependencies injected by Spring
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}
```

### What is the difference between @Inject and @Autowired?

**Answer:**
- **@Autowired:**
  - Spring-specific annotation
  - Supports required attribute to specify if dependency is optional
  - Can be used on fields, constructors, setters, or methods

- **@Inject:**
  - Standard Java annotation (JSR-330)
  - More portable across DI frameworks
  - No built-in required attribute (use @Nullable instead)
  - Can be used on fields, constructors, or methods

**Similarities:**
- Both perform dependency injection
- Both support constructor, setter, and field injection
- Both can be used with @Qualifier

**Example:**
```java
// Using @Autowired
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    @Autowired(required = false)
    private AuditService auditService;
}

// Using @Inject
@Service
public class ProductService {
    @Inject
    private ProductRepository productRepository;
    
    @Inject
    @Named("mainAuditService")
    private AuditService auditService;
}
```

### What are the types of autowiring in Spring?

**Answer:**
Spring supports five types of autowiring:

1. **no (default in XML configuration):**
   - No autowiring
   - Dependencies must be explicitly defined

2. **byName:**
   - Autowires by matching bean property name with bean name in context
   - If a property named "userRepository" exists, Spring looks for a bean named "userRepository"

3. **byType:**
   - Autowires by matching bean property type with bean type in context
   - If a property of type UserRepository exists, Spring looks for a bean of type UserRepository
   - Throws exception if multiple beans of same type exist without qualifiers

4. **constructor:**
   - Similar to byType but applies to constructor arguments
   - Matches constructor argument types with beans in context

5. **autodetect:**
   - Attempts constructor autowiring first, then falls back to byType
   - Deprecated in newer versions of Spring

**Example:**
```java
// XML configuration example
<bean id="userService" class="com.example.UserService" autowire="byName"/>

// Java configuration with @Autowired (byType)
@Service
public class UserService {
    private final UserRepository userRepository;
    
    @Autowired // Constructor injection using byType autowiring
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### Compare setter injection and constructor injection. Which one is better and why?

**Answer:**
**Setter Injection:**
- Dependencies injected via setter methods
- Allows for optional dependencies
- Can resolve circular dependencies
- Dependencies can be changed after object creation
- More flexible but less safe

**Constructor Injection:**
- Dependencies injected via constructor
- Ensures dependencies are available at object creation
- Supports immutability (final fields)
- Prevents circular dependencies
- More explicit about required dependencies

**Which is better?**
Constructor injection is generally preferred because:
1. It enforces required dependencies at object creation time
2. It supports immutability through final fields
3. It makes dependencies explicit in the class signature
4. It ensures the object is always in a valid state
5. It's easier to test with mocks

**Example:**
```java
// Constructor injection (preferred)
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}

// Setter injection
@Service
public class ProductService {
    private ProductRepository productRepository;
    private Optional<NotificationService> notificationService = Optional.empty();
    
    @Autowired
    public void setProductRepository(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    @Autowired(required = false)
    public void setNotificationService(NotificationService notificationService) {
        this.notificationService = Optional.ofNullable(notificationService);
    }
}
```

## Spring Configuration

### How does Spring configuration work?

**Answer:**
Spring configuration is the process of defining beans and their dependencies for the Spring IoC container. It tells Spring:
- What objects (beans) to create
- How to configure these objects
- How to wire dependencies between objects

**Key components:**
1. **Bean definitions:** Specify classes to instantiate as beans
2. **Property values:** Configure bean properties
3. **Dependency references:** Define relationships between beans
4. **Initialization/destruction methods:** Configure lifecycle callbacks
5. **Scope definitions:** Define bean creation policy (singleton, prototype, etc.)

**Configuration approaches:**
1. XML-based configuration
2. Annotation-based configuration
3. Java-based configuration
4. Groovy-based configuration
5. Hybrid approaches combining the above

**Example:**
```java
// Java-based configuration
@Configuration
public class AppConfig {
    @Bean
    public UserRepository userRepository() {
        return new JpaUserRepository();
    }
    
    @Bean
    public UserService userService(UserRepository userRepository) {
        return new UserServiceImpl(userRepository);
    }
}
```

### What are the different ways to configure a Spring application?

**Answer:**
Spring offers multiple configuration approaches:

1. **XML-based configuration:**
   - Traditional approach using XML files
   - Explicit and centralized configuration
   - Example: `applicationContext.xml`
   ```xml
   <beans>
       <bean id="userRepository" class="com.example.JpaUserRepository"/>
       <bean id="userService" class="com.example.UserServiceImpl">
           <constructor-arg ref="userRepository"/>
       </bean>
   </beans>
   ```

2. **Annotation-based configuration:**
   - Uses annotations in Java classes
   - Requires component scanning
   - Examples: `@Component`, `@Service`, `@Repository`, `@Controller`
   ```java
   @Service
   public class UserServiceImpl implements UserService {
       private final UserRepository userRepository;
       
       @Autowired
       public UserServiceImpl(UserRepository userRepository) {
           this.userRepository = userRepository;
       }
   }
   ```

3. **Java-based configuration:**
   - Uses Java classes with `@Configuration` and `@Bean` annotations
   - Type-safe and refactoring-friendly
   ```java
   @Configuration
   public class AppConfig {
       @Bean
       public UserRepository userRepository() {
           return new JpaUserRepository();
       }
       
       @Bean
       public UserService userService() {
           return new UserServiceImpl(userRepository());
       }
   }
   ```

4. **Groovy-based configuration:**
   - Uses Groovy DSL for more concise syntax
   ```groovy
   beans {
       userRepository(JpaUserRepository)
       userService(UserServiceImpl, ref('userRepository'))
   }
   ```

5. **Hybrid configuration:**
   - Combines multiple approaches
   - Example: Java configuration importing XML configuration
   ```java
   @Configuration
   @ImportResource("classpath:applicationContext.xml")
   public class AppConfig {
       @Bean
       public AuditService auditService() {
           return new AuditServiceImpl();
       }
   }
   ```

### What is the difference between BeanFactory and ApplicationContext?

**Answer:**
Both BeanFactory and ApplicationContext are IoC containers in Spring, but ApplicationContext extends BeanFactory with additional features:

**BeanFactory:**
- Basic IoC container
- Lazy initialization of beans
- Explicit bean retrieval using getBean()
- Lightweight and suitable for resource-constrained environments
- Doesn't provide enterprise features out of the box

**ApplicationContext:**
- Advanced IoC container that extends BeanFactory
- Eager initialization of singleton beans by default
- Enterprise features like:
  - Event publication
  - Internationalization (i18n) support
  - Resource access (file, URL, etc.)
  - AOP integration
  - WebApplicationContext for web applications
- Multiple implementations (ClassPathXmlApplicationContext, AnnotationConfigApplicationContext, etc.)

**When to use which:**
- Use ApplicationContext for most applications (recommended)
- Use BeanFactory only in resource-constrained environments where memory is critical

**Example:**
```java
// BeanFactory example
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("beans.xml"));
UserService userService = factory.getBean("userService", UserService.class);

// ApplicationContext example
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
UserService userService = context.getBean("userService", UserService.class);
context.publishEvent(new UserCreatedEvent(user)); // Event publication
```

## Spring Beans

### What is a component and a bean in Spring?

**Answer:**
**Bean:**
- Any object managed by the Spring IoC container
- Created, configured, and managed by Spring
- Defined in configuration (XML, annotations, or Java config)
- Can be any Java class, not necessarily following any specific convention

**Component:**
- A specific type of bean that is auto-detected through component scanning
- Annotated with @Component or a stereotype annotation (@Service, @Repository, @Controller)
- Automatically registered as a bean in the Spring context
- Part of Spring's annotation-based configuration approach

**Relationship:**
- All components are beans, but not all beans are components
- Components are auto-detected, while beans can be explicitly defined
- Components use class-level annotations, while beans can be defined externally

**Example:**
```java
// Component (auto-detected)
@Component
public class UserRepository {
    // Implementation
}

// Bean defined in Java configuration
@Configuration
public class AppConfig {
    @Bean
    public PaymentGateway paymentGateway() {
        PaymentGateway gateway = new PaymentGateway();
        gateway.setApiKey("secret-key");
        return gateway;
    }
}
```

### What is the lifecycle of a Spring bean?

**Answer:**
The Spring bean lifecycle consists of several phases:

1. **Instantiation:**
   - Spring creates an instance of the bean using constructor

2. **Populating Properties:**
   - Dependencies are injected via setter methods or fields

3. **BeanNameAware:**
   - If bean implements BeanNameAware, setBeanName() is called

4. **BeanFactoryAware:**
   - If bean implements BeanFactoryAware, setBeanFactory() is called

5. **ApplicationContextAware:**
   - If bean implements ApplicationContextAware, setApplicationContext() is called

6. **Pre-Initialization (BeanPostProcessor):**
   - postProcessBeforeInitialization() methods are called

7. **InitializingBean:**
   - If bean implements InitializingBean, afterPropertiesSet() is called

8. **Custom Init Method:**
   - Custom init method specified by @Bean(initMethod="...") or XML init-method is called

9. **Post-Initialization (BeanPostProcessor):**
   - postProcessAfterInitialization() methods are called

10. **Bean is ready for use**

11. **Pre-Destruction:**
    - Container shutdown begins

12. **DisposableBean:**
    - If bean implements DisposableBean, destroy() is called

13. **Custom Destroy Method:**
    - Custom destroy method specified by @Bean(destroyMethod="...") or XML destroy-method is called

**Example:**
```java
@Component
public class UserService implements InitializingBean, DisposableBean {
    
    @Autowired
    private UserRepository userRepository;
    
    public UserService() {
        System.out.println("1. Constructor called");
    }
    
    @PostConstruct
    public void postConstruct() {
        System.out.println("3. @PostConstruct method called");
    }
    
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("4. InitializingBean's afterPropertiesSet method called");
    }
    
    public void customInit() {
        System.out.println("5. Custom init method called");
    }
    
    @PreDestroy
    public void preDestroy() {
        System.out.println("6. @PreDestroy method called");
    }
    
    @Override
    public void destroy() throws Exception {
        System.out.println("7. DisposableBean's destroy method called");
    }
    
    public void customDestroy() {
        System.out.println("8. Custom destroy method called");
    }
}

@Configuration
public class AppConfig {
    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public UserService userService() {
        return new UserService();
    }
}
```

### What are the scopes of Spring beans?

**Answer:**
Spring Framework supports six bean scopes:

1. **singleton (default):**
   - Single instance per Spring IoC container
   - Shared instance for all requests
   - Default scope if none specified

2. **prototype:**
   - New instance created each time the bean is requested
   - Not managed after creation (no destruction callbacks)
   - Used for stateful beans

3. **request:**
   - New instance per HTTP request
   - Only valid in web-aware Spring ApplicationContext

4. **session:**
   - New instance per HTTP session
   - Only valid in web-aware Spring ApplicationContext

5. **application:**
   - New instance per ServletContext (per web application)
   - Only valid in web-aware Spring ApplicationContext

6. **websocket:**
   - New instance per WebSocket session
   - Only valid in web-aware Spring ApplicationContext

**Example:**
```java
// Singleton scope (default)
@Component
public class UserService { }

// Prototype scope
@Component
@Scope("prototype")
public class ShoppingCart { }

// Request scope
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestScopedBean { }

// Session scope
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.INTERFACES)
public class UserPreferences implements Serializable { }
```

### What happens if a prototype-scoped bean is defined inside a singleton-scoped bean?

**Answer:**
When a prototype-scoped bean is injected into a singleton-scoped bean, the prototype bean behaves like a singleton because:

1. The singleton bean is created only once
2. During creation, the prototype bean is injected
3. The singleton keeps the same prototype instance forever
4. The prototype's purpose (creating new instances) is defeated

**Solutions:**

1. **Method Injection:**
   ```java
   @Component
   public abstract class UserManager {
       @Lookup
       public abstract ShoppingCart getShoppingCart();
       
       public void processUser(User user) {
           ShoppingCart cart = getShoppingCart(); // New instance each time
           // Use cart
       }
   }
   ```

2. **ObjectFactory/Provider Injection:**
   ```java
   @Component
   public class UserManager {
       @Autowired
       private ObjectFactory<ShoppingCart> cartFactory;
       // OR
       @Autowired
       private Provider<ShoppingCart> cartProvider;
       
       public void processUser(User user) {
           ShoppingCart cart = cartFactory.getObject(); // New instance each time
           // OR
           ShoppingCart cart = cartProvider.get(); // New instance each time
           // Use cart
       }
   }
   ```

3. **Scoped Proxy:**
   ```java
   @Component
   @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
   public class ShoppingCart { }
   
   @Component
   public class UserManager {
       @Autowired
       private ShoppingCart cart; // Actually a proxy
       
       public void processUser(User user) {
           // Each method call on cart goes through proxy and gets a new instance
           cart.addItem(item);
       }
   }
   ```

4. **ApplicationContext Awareness:**
   ```java
   @Component
   public class UserManager implements ApplicationContextAware {
       private ApplicationContext context;
       
       @Override
       public void setApplicationContext(ApplicationContext context) {
           this.context = context;
       }
       
       public void processUser(User user) {
           ShoppingCart cart = context.getBean(ShoppingCart.class); // New instance each time
           // Use cart
       }
   }
   ```

## Dependency Resolution

### What is circular dependency in Spring Framework and how to resolve it?

**Answer:**
**Circular Dependency:** Occurs when bean A depends on bean B, and bean B depends on bean A (directly or indirectly).

**Example:**
```java
@Component
public class BeanA {
    private final BeanB beanB;
    
    @Autowired
    public BeanA(BeanB beanB) {
        this.beanB = beanB;
    }
}

@Component
public class BeanB {
    private final BeanA beanA;
    
    @Autowired
    public BeanB(BeanA beanA) {
        this.beanA = beanA;
    }
}
```

**Problems:**
- With constructor injection, Spring can't instantiate either bean
- Indicates potential design issues in application architecture
- Can lead to startup failures or unexpected behavior

**Solutions:**

1. **Use setter injection:**
   ```java
   @Component
   public class BeanA {
       private BeanB beanB;
       
       @Autowired
       public void setBeanB(BeanB beanB) {
           this.beanB = beanB;
       }
   }
   ```

2. **Use @Lazy annotation:**
   ```java
   @Component
   public class BeanA {
       private final BeanB beanB;
       
       @Autowired
       public BeanA(@Lazy BeanB beanB) {
           this.beanB = beanB;
       }
   }
   ```

3. **Use ApplicationContext:**
   ```java
   @Component
   public class BeanA implements ApplicationContextAware {
       private ApplicationContext context;
       
       @Override
       public void setApplicationContext(ApplicationContext context) {
           this.context = context;
       }
       
       public void doSomething() {
           BeanB beanB = context.getBean(BeanB.class);
           // Use beanB
       }
   }
   ```

4. **Redesign to remove circular dependency:**
   - Extract common functionality to a third bean
   - Use events to decouple beans
   - Use interfaces to break direct dependencies

### What is the use of @Qualifier annotation and @Primary annotation?

**Answer:**
Both annotations help resolve ambiguity when multiple beans of the same type exist, but they work differently:

**@Qualifier:**
- Fine-grained control over which specific bean to inject
- Used at injection point to specify the exact bean by name or qualifier value
- Can be used with @Autowired or @Inject
- Can be custom-defined for more complex qualification

**@Primary:**
- Marks a bean as the primary candidate for autowiring
- Used at bean definition to set a default when multiple beans exist
- Applied when no specific qualifier is provided
- Simplifies configuration when there's a clear "default" implementation

**Example:**
```java
// Multiple implementations of the same interface
@Component
public class EmailNotificationService implements NotificationService { }

@Component
@Primary
public class PushNotificationService implements NotificationService { }

@Component("smsNotifier")
public class SmsNotificationService implements NotificationService { }

// Using @Primary (will inject PushNotificationService)
@Service
public class UserService {
    private final NotificationService notificationService;
    
    @Autowired
    public UserService(NotificationService notificationService) {
        this.notificationService = notificationService; // Gets PushNotificationService
    }
}

// Using @Qualifier with bean name
@Service
public class OrderService {
    private final NotificationService notificationService;
    
    @Autowired
    public OrderService(@Qualifier("smsNotifier") NotificationService notificationService) {
        this.notificationService = notificationService; // Gets SmsNotificationService
    }
}

// Using @Qualifier with custom qualifier
@Component
@Qualifier("async")
public class AsyncEmailService implements EmailService { }

@Component
@Qualifier("sync")
public class SyncEmailService implements EmailService { }

@Service
public class AlertService {
    private final EmailService emailService;
    
    @Autowired
    public AlertService(@Qualifier("async") EmailService emailService) {
        this.emailService = emailService; // Gets AsyncEmailService
    }
}
```

## Stereotypes

### What are stereotype annotations in Spring?

**Answer:**
Stereotype annotations in Spring are annotations that mark classes for auto-detection through component scanning. They indicate the role of the class in the application architecture.

**Core stereotype annotations:**

1. **@Component:**
   - Base stereotype annotation
   - Indicates a general Spring-managed component
   - Parent of all other stereotype annotations

2. **@Service:**
   - Indicates a class belongs to the service layer
   - Contains business logic and orchestrates repositories
   - Functionally identical to @Component but more semantically specific

3. **@Repository:**
   - Indicates a class belongs to the data access layer
   - Provides automatic exception translation for data access exceptions
   - Typically used for DAO implementations

4. **@Controller:**
   - Indicates a class serves as a web controller
   - Handles web requests and returns responses
   - Used in Spring MVC for request mapping

5. **@RestController:**
   - Combines @Controller and @ResponseBody
   - Indicates a class serves as a RESTful web service controller
   - Returns serialized objects directly as response body

**Example:**
```java
@Component
public class EmailValidator {
    public boolean isValid(String email) {
        // Validation logic
        return email != null && email.contains("@");
    }
}

@Repository
public class JpaUserRepository implements UserRepository {
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public User findById(Long id) {
        return entityManager.find(User.class, id);
    }
}

@Service
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;
    
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}

@Controller
public class UserController {
    @GetMapping("/users/{id}")
    public String getUser(@PathVariable Long id, Model model) {
        model.addAttribute("user", userService.getUser(id));
        return "user-details";
    }
}

@RestController
public class UserApiController {
    @GetMapping("/api/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }
}
```

### Can we exchange stereotype annotations with each other? (e.g., replace @Service with @Configuration)

**Answer:**
Technically, you can exchange most stereotype annotations with each other because they all extend from @Component, but it's not recommended for several reasons:

**What works:**
- Replacing @Service, @Repository, or @Controller with @Component
- Replacing @Component with @Service, @Repository, or @Controller
- Using @RestController instead of @Controller with @ResponseBody

**What doesn't work well:**
- Replacing @Configuration with other stereotypes (different behavior)
- Replacing other stereotypes with @Configuration

**Reasons not to exchange stereotypes:**

1. **Semantic clarity:**
   - Stereotypes communicate the role/purpose of the class
   - Makes code more readable and maintainable
   - Helps with architectural understanding

2. **Special behaviors:**
   - @Repository provides exception translation
   - @Controller is detected by request mapping scanners
   - @Configuration processes @Bean methods differently

3. **AOP and pointcuts:**
   - Many aspects target specific stereotypes
   - Changing annotations may break AOP functionality

4. **Future compatibility:**
   - Spring may add more specific behaviors to stereotypes
   - Code may break in future versions if using wrong stereotypes

**Example of potential issues:**
```java
// INCORRECT: Using @Repository for a service class
@Repository  // Wrong stereotype for semantic meaning
public class UserService {
    // Business logic
}

// INCORRECT: Using @Service for a configuration class
@Service  // Won't process @Bean methods correctly
public class DatabaseConfig {
    @Bean
    public DataSource dataSource() {
        // Configuration
    }
}

// CORRECT: Using appropriate stereotypes
@Service
public class UserService {
    // Business logic
}

@Configuration
public class DatabaseConfig {
    @Bean
    public DataSource dataSource() {
        // Configuration
    }
}
```

## Exception Handling

### How is exception handling done in Spring using @ControllerAdvice and @ExceptionHandler?

**Answer:**
Spring provides a robust exception handling mechanism using @ControllerAdvice and @ExceptionHandler annotations:

**@ExceptionHandler:**
- Method-level annotation in controllers
- Handles exceptions thrown from controller methods
- Maps exceptions to specific handler methods
- Limited to the controller where it's defined

**@ControllerAdvice:**
- Class-level annotation
- Creates global exception handlers
- Applies to all controllers or specified subsets
- Centralizes exception handling logic

**Key features:**
1. Consistent error responses across the application
2. Separation of error handling from business logic
3. Custom error responses based on exception type
4. Support for different response formats (JSON, XML, HTML)

**Example:**
```java
// Controller with local exception handler
@Controller
public class UserController {
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        // May throw UserNotFoundException
        return userService.findById(id);
    }
    
    // Local exception handler - only applies to this controller
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleUserNotFound(UserNotFoundException ex) {
        return new ErrorResponse("USER_NOT_FOUND", ex.getMessage());
    }
}

// Global exception handler
@ControllerAdvice
public class GlobalExceptionHandler {
    
    // Handles exceptions from all controllers
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGenericException(Exception ex) {
        return new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred");
    }
    
    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationException(ValidationException ex) {
        return new ErrorResponse("VALIDATION_ERROR", ex.getMessage());
    }
    
    // Handle specific exceptions with custom responses
    @ExceptionHandler(AccessDeniedException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ErrorResponse handleAccessDenied(AccessDeniedException ex) {
        return new ErrorResponse("ACCESS_DENIED", "You don't have permission to access this resource");
    }
    
    // Return different response types based on content negotiation
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ResponseEntity<?> handleResourceNotFound(
            ResourceNotFoundException ex, 
            WebRequest request) {
        
        ErrorResponse errorResponse = new ErrorResponse("NOT_FOUND", ex.getMessage());
        
        if (request.getHeader("Accept").contains("application/json")) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
        } else {
            ModelAndView mav = new ModelAndView("error-page");
            mav.addObject("error", errorResponse);
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body(mav);
        }
    }
}

// Targeting specific controllers or packages
@ControllerAdvice(basePackages = "com.example.api")
public class ApiExceptionHandler {
    // Exception handlers for API controllers only
}

@ControllerAdvice(annotations = RestController.class)
public class RestExceptionHandler {
    // Exception handlers for @RestController classes only
}
```