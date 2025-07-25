# Design Principles Interview Questions and Answers

## SOLID Principles

### Single Responsibility Principle (SRP)

#### What is the Single Responsibility Principle?

**Answer:**
The Single Responsibility Principle states that a class should have only one reason to change, meaning it should have only one responsibility or job. Each class should focus on a single functionality and encapsulate only the variables and methods that are related to that functionality.

**Key points:**
- A class should have only one responsibility
- A class should have only one reason to change
- High cohesion within the class
- Separation of concerns

#### How does it improve code maintainability?

**Answer:**
SRP improves code maintainability in several ways:

1. **Easier to understand:** Classes with single responsibility are simpler and easier to comprehend
2. **Easier to test:** Focused classes have fewer dependencies and are easier to unit test
3. **Reduced coupling:** Classes are less dependent on each other
4. **Easier to modify:** Changes to one responsibility don't affect other functionalities
5. **Better reusability:** Single-purpose classes can be reused in different contexts

#### Provide an example of SRP in practice

**Answer:**

**Before SRP (Violating the principle):**
```java
// This class has multiple responsibilities
public class Employee {
    private String name;
    private double salary;
    
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    // Responsibility 1: Employee data management
    public String getName() { return name; }
    public double getSalary() { return salary; }
    
    // Responsibility 2: Salary calculation
    public double calculatePay() {
        return salary * 1.1; // 10% bonus
    }
    
    // Responsibility 3: Database operations
    public void save() {
        // Database save logic
        System.out.println("Saving employee to database");
    }
    
    // Responsibility 4: Report generation
    public void generateReport() {
        System.out.println("Employee Report: " + name + " - $" + salary);
    }
}
```

**After SRP (Following the principle):**
```java
// Responsibility 1: Employee data management
public class Employee {
    private String name;
    private double salary;
    
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    public String getName() { return name; }
    public double getSalary() { return salary; }
}

// Responsibility 2: Salary calculation
public class PayrollCalculator {
    public double calculatePay(Employee employee) {
        return employee.getSalary() * 1.1; // 10% bonus
    }
}

// Responsibility 3: Database operations
public class EmployeeRepository {
    public void save(Employee employee) {
        // Database save logic
        System.out.println("Saving employee to database");
    }
}

// Responsibility 4: Report generation
public class EmployeeReportGenerator {
    public void generateReport(Employee employee) {
        System.out.println("Employee Report: " + employee.getName() + " - $" + employee.getSalary());
    }
}
```

### Open/Closed Principle (OCP)

#### What is the Open/Closed Principle?

**Answer:**
The Open/Closed Principle states that software entities (classes, modules, functions) should be open for extension but closed for modification. This means you should be able to add new functionality without changing existing code.

**Key points:**
- Open for extension: New behavior can be added
- Closed for modification: Existing code should not be changed
- Achieved through abstraction and polymorphism
- Promotes code stability and reduces risk of introducing bugs

#### How does it help with code extensibility?

**Answer:**
OCP helps with code extensibility by:

1. **Preventing modification of existing code:** Reduces risk of breaking existing functionality
2. **Enabling new features:** New functionality can be added through inheritance or composition
3. **Promoting abstraction:** Use of interfaces and abstract classes allows for flexible implementations
4. **Supporting polymorphism:** Different implementations can be used interchangeably
5. **Reducing testing overhead:** Existing code doesn't need to be retested when new features are added

#### Provide an example of OCP in practice

**Answer:**

**Before OCP (Violating the principle):**
```java
// This class needs modification every time a new shape is added
public class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Rectangle) {
            Rectangle rectangle = (Rectangle) shape;
            return rectangle.getWidth() * rectangle.getHeight();
        } else if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            return Math.PI * circle.getRadius() * circle.getRadius();
        }
        // Need to modify this method for every new shape
        return 0;
    }
}

class Rectangle {
    private double width, height;
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    public double getWidth() { return width; }
    public double getHeight() { return height; }
}

class Circle {
    private double radius;
    public Circle(double radius) { this.radius = radius; }
    public double getRadius() { return radius; }
}
```

**After OCP (Following the principle):**
```java
// Abstract base class/interface
public abstract class Shape {
    public abstract double calculateArea();
}

// Concrete implementations
public class Rectangle extends Shape {
    private double width, height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return width * height;
    }
}

public class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

// New shape can be added without modifying existing code
public class Triangle extends Shape {
    private double base, height;
    
    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
}

// Calculator class doesn't need modification
public class AreaCalculator {
    public double calculateArea(Shape shape) {
        return shape.calculateArea();
    }
    
    public double calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
                    .mapToDouble(Shape::calculateArea)
                    .sum();
    }
}
```

### Liskov Substitution Principle (LSP)

#### What is the Liskov Substitution Principle?

**Answer:**
The Liskov Substitution Principle states that objects of a superclass should be replaceable with objects of its subclasses without breaking the application. In other words, if S is a subtype of T, then objects of type T may be replaced with objects of type S without altering the correctness of the program.

**Key points:**
- Subtypes must be substitutable for their base types
- Derived classes must not strengthen preconditions
- Derived classes must not weaken postconditions
- Behavioral compatibility is essential
- Maintains the contract defined by the base class

#### How does it ensure proper inheritance hierarchies?

**Answer:**
LSP ensures proper inheritance hierarchies by:

1. **Maintaining behavioral contracts:** Subclasses must honor the behavior expected from the base class
2. **Preventing unexpected behavior:** Substitution should not cause surprises or break functionality
3. **Ensuring semantic correctness:** Inheritance should represent true "is-a" relationships
4. **Promoting polymorphism:** Enables safe use of polymorphic behavior
5. **Validating design decisions:** Forces developers to think about proper inheritance relationships

#### Provide an example of LSP in practice

**Answer:**

**Before LSP (Violating the principle):**
```java
// Base class
public class Bird {
    public void fly() {
        System.out.println("Bird is flying");
    }
}

// Subclass that violates LSP
public class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!");
    }
}

// Client code that breaks when using Penguin
public class BirdWatcher {
    public void makeBirdFly(Bird bird) {
        bird.fly(); // This will throw exception for Penguin
    }
}
```

**After LSP (Following the principle):**
```java
// Base class with common behavior
public abstract class Bird {
    public abstract void move();
    public void eat() {
        System.out.println("Bird is eating");
    }
}

// Interface for flying behavior
public interface Flyable {
    void fly();
}

// Flying bird implementation
public class Sparrow extends Bird implements Flyable {
    @Override
    public void move() {
        fly();
    }
    
    @Override
    public void fly() {
        System.out.println("Sparrow is flying");
    }
}

// Non-flying bird implementation
public class Penguin extends Bird {
    @Override
    public void move() {
        swim();
    }
    
    public void swim() {
        System.out.println("Penguin is swimming");
    }
}

// Client code that works with all birds
public class BirdWatcher {
    public void makeBirdMove(Bird bird) {
        bird.move(); // Works for all bird types
    }
    
    public void makeFlyableFly(Flyable flyable) {
        flyable.fly(); // Only works with flying birds
    }
}
```

### Interface Segregation Principle (ISP)

#### What is the Interface Segregation Principle?

**Answer:**
The Interface Segregation Principle states that no client should be forced to depend on methods it does not use. It suggests that larger interfaces should be split into smaller, more specific interfaces so that clients only need to know about the methods that are relevant to them.

**Key points:**
- Clients should not be forced to depend on interfaces they don't use
- Prefer many specific interfaces over one general-purpose interface
- Reduces coupling between classes
- Increases cohesion within interfaces
- Makes the system more flexible and maintainable

#### How does it prevent interface pollution?

**Answer:**
ISP prevents interface pollution by:

1. **Avoiding fat interfaces:** Large interfaces with many methods are broken down into smaller, focused ones
2. **Reducing unnecessary dependencies:** Clients only depend on methods they actually use
3. **Improving maintainability:** Changes to unused methods don't affect clients
4. **Enhancing flexibility:** Different clients can implement only the interfaces they need
5. **Promoting single responsibility:** Each interface has a specific, well-defined purpose

#### Provide an example of ISP in practice

**Answer:**

**Before ISP (Violating the principle):**
```java
// Fat interface that forces clients to implement methods they don't need
public interface Worker {
    void work();
    void eat();
    void sleep();
    void attendMeeting();
    void writeCode();
    void designSystem();
}

// Robot worker forced to implement irrelevant methods
public class RobotWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Robot is working");
    }
    
    @Override
    public void eat() {
        throw new UnsupportedOperationException("Robots don't eat");
    }
    
    @Override
    public void sleep() {
        throw new UnsupportedOperationException("Robots don't sleep");
    }
    
    @Override
    public void attendMeeting() {
        throw new UnsupportedOperationException("Robots don't attend meetings");
    }
    
    @Override
    public void writeCode() {
        System.out.println("Robot is writing code");
    }
    
    @Override
    public void designSystem() {
        throw new UnsupportedOperationException("Robots don't design systems");
    }
}
```

**After ISP (Following the principle):**
```java
// Segregated interfaces
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

public interface Attendable {
    void attendMeeting();
}

public interface Codeable {
    void writeCode();
}

public interface Designable {
    void designSystem();
}

// Robot worker implements only relevant interfaces
public class RobotWorker implements Workable, Codeable {
    @Override
    public void work() {
        System.out.println("Robot is working");
    }
    
    @Override
    public void writeCode() {
        System.out.println("Robot is writing code");
    }
}

// Human worker implements relevant interfaces
public class HumanWorker implements Workable, Eatable, Sleepable, Attendable, Designable {
    @Override
    public void work() {
        System.out.println("Human is working");
    }
    
    @Override
    public void eat() {
        System.out.println("Human is eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Human is sleeping");
    }
    
    @Override
    public void attendMeeting() {
        System.out.println("Human is attending meeting");
    }
    
    @Override
    public void designSystem() {
        System.out.println("Human is designing system");
    }
}

// Manager class that works with specific interfaces
public class WorkManager {
    public void manageWork(Workable worker) {
        worker.work();
    }
    
    public void organizeMeeting(Attendable attendee) {
        attendee.attendMeeting();
    }
    
    public void assignCoding(Codeable coder) {
        coder.writeCode();
    }
}
```

### Dependency Inversion Principle (DIP)

#### What is the Dependency Inversion Principle?

**Answer:**
The Dependency Inversion Principle states that:
1. High-level modules should not depend on low-level modules. Both should depend on abstractions.
2. Abstractions should not depend on details. Details should depend on abstractions.

This principle promotes loose coupling by ensuring that dependencies flow toward abstractions rather than concrete implementations.

**Key points:**
- Depend on abstractions, not concretions
- High-level modules should not depend on low-level modules
- Both should depend on abstractions
- Promotes loose coupling
- Enables dependency injection

#### How does it promote loose coupling?

**Answer:**
DIP promotes loose coupling by:

1. **Abstracting dependencies:** Using interfaces instead of concrete classes
2. **Inverting control:** Dependencies are injected rather than created internally
3. **Reducing direct dependencies:** Classes depend on abstractions, not implementations
4. **Enabling flexibility:** Different implementations can be swapped easily
5. **Improving testability:** Mock objects can be easily injected for testing
6. **Supporting configuration:** Dependencies can be configured externally

#### Provide an example of DIP in practice

**Answer:**

**Before DIP (Violating the principle):**
```java
// Low-level module
public class MySQLDatabase {
    public void save(String data) {
        System.out.println("Saving data to MySQL database: " + data);
    }
}

// High-level module directly depending on low-level module
public class UserService {
    private MySQLDatabase database; // Direct dependency on concrete class
    
    public UserService() {
        this.database = new MySQLDatabase(); // Tight coupling
    }
    
    public void saveUser(String userData) {
        // Business logic
        String processedData = userData.toUpperCase();
        database.save(processedData);
    }
}
```

**After DIP (Following the principle):**
```java
// Abstraction
public interface Database {
    void save(String data);
}

// Low-level modules implementing the abstraction
public class MySQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving data to MySQL database: " + data);
    }
}

public class PostgreSQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving data to PostgreSQL database: " + data);
    }
}

public class MongoDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving data to MongoDB: " + data);
    }
}

// High-level module depending on abstraction
public class UserService {
    private Database database; // Depends on abstraction
    
    // Dependency injection through constructor
    public UserService(Database database) {
        this.database = database;
    }
    
    public void saveUser(String userData) {
        // Business logic
        String processedData = userData.toUpperCase();
        database.save(processedData);
    }
}

// Usage with dependency injection
public class Application {
    public static void main(String[] args) {
        // Can easily switch between different database implementations
        Database database = new MySQLDatabase();
        UserService userService = new UserService(database);
        userService.saveUser("John Doe");
        
        // Switch to different implementation without changing UserService
        Database mongoDatabase = new MongoDatabase();
        UserService userServiceWithMongo = new UserService(mongoDatabase);
        userServiceWithMongo.saveUser("Jane Smith");
    }
}

// Dependency injection with Spring Framework
@Service
public class UserService {
    private final Database database;
    
    @Autowired
    public UserService(Database database) {
        this.database = database;
    }
    
    public void saveUser(String userData) {
        String processedData = userData.toUpperCase();
        database.save(processedData);
    }
}

@Repository
public class MySQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving data to MySQL database: " + data);
    }
}

## Other Design Principles

### DRY (Don't Repeat Yourself)

#### What is the DRY principle?

**Answer:**
The DRY principle states that "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system." It emphasizes avoiding code duplication by ensuring that each piece of functionality exists in only one place.

**Key points:**
- Avoid code duplication
- Single source of truth for each piece of logic
- Reduce maintenance overhead
- Improve consistency
- Minimize the risk of bugs

#### How does it reduce code duplication?

**Answer:**
DRY reduces code duplication by:

1. **Extracting common functionality:** Moving repeated code into reusable methods or classes
2. **Creating utility functions:** Centralizing common operations
3. **Using inheritance and composition:** Sharing behavior through object-oriented principles
4. **Implementing configuration-driven approaches:** Using configuration files instead of hardcoded values
5. **Utilizing templates and patterns:** Creating reusable code structures

**Examples:**

**Before DRY (Code duplication):**
```java
public class OrderService {
    public void processOnlineOrder(Order order) {
        // Validation logic
        if (order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order must have items");
        }
        if (order.getCustomer() == null) {
            throw new IllegalArgumentException("Order must have a customer");
        }
        
        // Calculate total
        double total = 0;
        for (OrderItem item : order.getItems()) {
            total += item.getPrice() * item.getQuantity();
        }
        order.setTotal(total);
        
        // Process payment
        System.out.println("Processing online payment for: $" + total);
        
        // Save order
        System.out.println("Saving order to database");
    }
    
    public void processInStoreOrder(Order order) {
        // Same validation logic (duplicated)
        if (order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order must have items");
        }
        if (order.getCustomer() == null) {
            throw new IllegalArgumentException("Order must have a customer");
        }
        
        // Same calculation logic (duplicated)
        double total = 0;
        for (OrderItem item : order.getItems()) {
            total += item.getPrice() * item.getQuantity();
        }
        order.setTotal(total);
        
        // Different payment processing
        System.out.println("Processing in-store payment for: $" + total);
        
        // Same save logic (duplicated)
        System.out.println("Saving order to database");
    }
}
```

**After DRY (Eliminating duplication):**
```java
public class OrderService {
    
    public void processOnlineOrder(Order order) {
        validateOrder(order);
        calculateTotal(order);
        processOnlinePayment(order.getTotal());
        saveOrder(order);
    }
    
    public void processInStoreOrder(Order order) {
        validateOrder(order);
        calculateTotal(order);
        processInStorePayment(order.getTotal());
        saveOrder(order);
    }
    
    // Extracted common validation logic
    private void validateOrder(Order order) {
        if (order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order must have items");
        }
        if (order.getCustomer() == null) {
            throw new IllegalArgumentException("Order must have a customer");
        }
    }
    
    // Extracted common calculation logic
    private void calculateTotal(Order order) {
        double total = order.getItems().stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
        order.setTotal(total);
    }
    
    // Specific payment methods
    private void processOnlinePayment(double amount) {
        System.out.println("Processing online payment for: $" + amount);
    }
    
    private void processInStorePayment(double amount) {
        System.out.println("Processing in-store payment for: $" + amount);
    }
    
    // Extracted common save logic
    private void saveOrder(Order order) {
        System.out.println("Saving order to database");
    }
}

### KISS (Keep It Simple, Stupid)

#### What is the KISS principle?

**Answer:**
The KISS principle states that most systems work best if they are kept simple rather than made complicated. It emphasizes simplicity in design and implementation, avoiding unnecessary complexity.

**Key points:**
- Simplicity should be a key goal in design
- Avoid unnecessary complexity
- Simple solutions are easier to understand and maintain
- Complex solutions should only be used when absolutely necessary
- Prefer readable and straightforward code

#### How does it improve code readability?

**Answer:**
KISS improves code readability by:

1. **Using clear and descriptive names:** Variables, methods, and classes have meaningful names
2. **Avoiding over-engineering:** Not adding features or complexity that isn't needed
3. **Writing straightforward logic:** Using simple control structures and avoiding clever tricks
4. **Minimizing nesting:** Keeping code flat and easy to follow
5. **Breaking down complex problems:** Dividing complex tasks into smaller, manageable pieces

### YAGNI (You Aren't Gonna Need It)

#### What is the YAGNI principle?

**Answer:**
YAGNI is a principle that states you should not add functionality until it is actually needed. It discourages developers from implementing features based on speculation about future requirements.

**Key points:**
- Don't implement features until they are actually required
- Avoid speculative development
- Focus on current requirements
- Reduce code complexity and maintenance overhead
- Prevent over-engineering

#### How does it prevent over-engineering?

**Answer:**
YAGNI prevents over-engineering by:

1. **Focusing on current needs:** Only implementing what is required now
2. **Avoiding speculative features:** Not building functionality "just in case"
3. **Reducing complexity:** Keeping the codebase simple and focused
4. **Minimizing maintenance:** Less code means less to maintain and debug
5. **Enabling agility:** Easier to change direction when requirements change

### Law of Demeter (Principle of Least Knowledge)

#### What is the Law of Demeter?

**Answer:**
The Law of Demeter states that a method of an object should only call methods of:
1. The object itself
2. Objects passed as parameters
3. Objects created within the method
4. Direct component objects of the object

It's also known as the "Principle of Least Knowledge" and can be summarized as "don't talk to strangers."

**Key points:**
- Minimize coupling between classes
- Each unit should have limited knowledge about other units
- Only talk to immediate friends, not friends of friends
- Reduces dependencies and improves maintainability

#### How does it reduce coupling between objects?

**Answer:**
The Law of Demeter reduces coupling by:

1. **Limiting object interactions:** Objects only interact with their immediate collaborators
2. **Preventing chain calls:** Avoiding method chaining that creates tight coupling
3. **Encapsulating behavior:** Objects handle their own responsibilities without exposing internals
4. **Reducing knowledge:** Objects don't need to know about the internal structure of other objects
5. **Improving maintainability:** Changes to internal structures don't affect distant objects

## System Design Questions

### URL Shortener

#### How would you design a URL shortener service like TinyURL?

**Answer:**
A URL shortener service requires several key components:

**System Requirements:**
- Shorten long URLs to short URLs
- Redirect short URLs to original URLs
- Handle high read/write traffic
- Provide analytics (optional)
- Custom aliases (optional)

**High-Level Architecture:**
```
[Client] -> [Load Balancer] -> [Web Servers] -> [Application Servers] -> [Database]
                                    |                                      |
                                    v                                      v
                               [Cache Layer]                        [Analytics DB]
```

**Key Components:**

1. **URL Encoding Service:**
```java
public class URLShortenerService {
    private static final String BASE62 = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    private URLRepository urlRepository;
    private CacheService cacheService;
    
    public String shortenURL(String longURL) {
        String existingShortURL = urlRepository.findByLongURL(longURL);
        if (existingShortURL != null) {
            return existingShortURL;
        }
        
        String shortURL = generateShortURL();
        URLMapping mapping = new URLMapping(shortURL, longURL, System.currentTimeMillis());
        urlRepository.save(mapping);
        cacheService.put(shortURL, longURL);
        
        return shortURL;
    }
    
    public String expandURL(String shortURL) {
        String longURL = cacheService.get(shortURL);
        if (longURL != null) {
            return longURL;
        }
        
        URLMapping mapping = urlRepository.findByShortURL(shortURL);
        if (mapping != null) {
            cacheService.put(shortURL, mapping.getLongURL());
            return mapping.getLongURL();
        }
        
        return null;
    }
    
    private String generateShortURL() {
        long counter = getNextCounter();
        return encodeBase62(counter);
    }
    
    private String encodeBase62(long num) {
        StringBuilder sb = new StringBuilder();
        while (num > 0) {
            sb.append(BASE62.charAt((int)(num % 62)));
            num /= 62;
        }
        return sb.reverse().toString();
    }
}
```

#### What are the key components required?

**Answer:**
Key components for a URL shortener:

1. **Web Layer:**
   - Load balancers for traffic distribution
   - Web servers to handle HTTP requests
   - Rate limiting to prevent abuse

2. **Application Layer:**
   - URL encoding/decoding service
   - Validation service
   - Analytics service

3. **Data Layer:**
   - Primary database (MySQL/PostgreSQL)
   - Cache layer (Redis/Memcached)
   - Analytics database (for click tracking)

4. **Supporting Services:**
   - Monitoring and logging
   - Configuration management
   - Security services

#### How would you handle high traffic?

**Answer:**
Strategies for handling high traffic:

1. **Horizontal Scaling:**
   - Multiple application server instances
   - Database sharding based on short URL
   - Read replicas for database

2. **Caching Strategy:**
   - Cache popular URLs in Redis
   - CDN for static content
   - Application-level caching

3. **Database Optimization:**
   - Proper indexing on short_url and long_url
   - Database partitioning
   - Connection pooling

4. **Asynchronous Processing:**
   - Queue-based analytics processing
   - Background cleanup of expired URLs

### API Design

#### What are the key considerations when designing an API?

**Answer:**
Key considerations for API design:

1. **RESTful Design:**
   - Use appropriate HTTP methods (GET, POST, PUT, DELETE)
   - Resource-based URLs
   - Stateless operations
   - Proper HTTP status codes

2. **Consistency:**
   - Consistent naming conventions
   - Uniform response formats
   - Standard error handling

3. **Versioning:**
   - URL versioning (/v1/users)
   - Header versioning
   - Backward compatibility

4. **Documentation:**
   - Clear API documentation
   - Examples and use cases
   - Interactive documentation (Swagger)

5. **Performance:**
   - Pagination for large datasets
   - Filtering and sorting options
   - Caching headers

#### How would you ensure API security?

**Answer:**
API security measures:

1. **Authentication:**
   - JWT tokens
   - OAuth 2.0
   - API keys

2. **Authorization:**
   - Role-based access control (RBAC)
   - Resource-level permissions
   - Scope-based access

3. **Input Validation:**
   - Validate all input parameters
   - Sanitize user input
   - Use parameterized queries

4. **Rate Limiting:**
   - Prevent abuse and DoS attacks
   - Per-user and per-IP limits
   - Graceful degradation

5. **HTTPS:**
   - Encrypt data in transit
   - Use TLS 1.2 or higher
   - Proper certificate management

#### How would you version an API?

**Answer:**
API versioning strategies:

1. **URL Versioning:**
```java
@RequestMapping("/api/v1/users")
@RequestMapping("/api/v2/users")
```

2. **Header Versioning:**
```java
@GetMapping(value = "/users", headers = "API-Version=1")
@GetMapping(value = "/users", headers = "API-Version=2")
```

3. **Parameter Versioning:**
```java
@GetMapping("/users")
public ResponseEntity<?> getUsers(@RequestParam(defaultValue = "1") String version) {
    if ("2".equals(version)) {
        return getV2Users();
    }
    return getV1Users();
}
```

### Caching Strategies

#### How would you design caching for a write-heavy system?

**Answer:**
For write-heavy systems:

1. **Write-Through Cache:**
   - Write to cache and database simultaneously
   - Ensures data consistency
   - Higher write latency

2. **Write-Behind Cache:**
   - Write to cache immediately
   - Asynchronously write to database
   - Better write performance
   - Risk of data loss

3. **Cache Invalidation:**
   - Invalidate cache entries on writes
   - Use TTL for automatic expiration
   - Event-driven cache updates

#### How would you design caching for a read-heavy system?

**Answer:**
For read-heavy systems:

1. **Read-Through Cache:**
   - Cache miss triggers database read
   - Cache is populated automatically
   - Transparent to application

2. **Cache-Aside Pattern:**
   - Application manages cache explicitly
   - More control over caching logic
   - Common pattern for web applications

3. **Multi-Level Caching:**
   - L1: Application cache
   - L2: Distributed cache (Redis)
   - L3: CDN for static content

#### What caching strategies would you use for different scenarios?

**Answer:**
Caching strategies by scenario:

1. **Session Data:**
   - Use Redis with TTL
   - Distributed cache for scalability

2. **Database Query Results:**
   - Query result caching
   - Cache invalidation on data changes

3. **Static Content:**
   - CDN caching
   - Long TTL values
   - Version-based cache busting

4. **Computed Values:**
   - Cache expensive calculations
   - Background refresh for popular items

### Performance Optimization

#### How would you increase the performance of an application?

**Answer:**
Performance optimization strategies:

1. **Database Optimization:**
   - Proper indexing
   - Query optimization
   - Connection pooling
   - Database partitioning

2. **Caching:**
   - Application-level caching
   - Database query caching
   - CDN for static content

3. **Code Optimization:**
   - Algorithm improvements
   - Reduce object creation
   - Use appropriate data structures
   - Lazy loading

4. **Infrastructure:**
   - Horizontal scaling
   - Load balancing
   - Asynchronous processing
   - Microservices architecture

#### What tools would you use to identify performance bottlenecks?

**Answer:**
Performance monitoring tools:

1. **Application Monitoring:**
   - New Relic, AppDynamics
   - Application metrics and traces
   - Real user monitoring

2. **Database Monitoring:**
   - Database-specific tools (MySQL Workbench, pgAdmin)
   - Query execution plans
   - Slow query logs

3. **System Monitoring:**
   - CPU, memory, disk I/O monitoring
   - Network performance
   - JVM monitoring (JVisualVM, JProfiler)

4. **Load Testing:**
   - JMeter, Gatling
   - Stress testing
   - Performance benchmarking

#### What are common performance optimization techniques?

**Answer:**
Common optimization techniques:

1. **Database Level:**
   - Index optimization
   - Query optimization
   - Denormalization where appropriate
   - Read replicas

2. **Application Level:**
   - Object pooling
   - Lazy loading
   - Batch processing
   - Asynchronous operations

3. **Network Level:**
   - Compression
   - CDN usage
   - HTTP/2
   - Connection keep-alive

4. **Memory Management:**
   - Garbage collection tuning
   - Memory leak prevention
   - Efficient data structures
   - Caching strategies
```
```