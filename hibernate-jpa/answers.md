# Hibernate and JPA Interview Questions and Answers

## Core Concepts

### What is ORM (Object-Relational Mapping)?

**Answer:**
ORM (Object-Relational Mapping) is a programming technique that maps objects in object-oriented programming languages to tables in relational databases. It provides a bridge between the object-oriented world and the relational database world.

**Key concepts:**
- **Object-to-Table mapping:** Classes map to database tables
- **Attribute-to-Column mapping:** Object properties map to table columns
- **Relationship mapping:** Object associations map to foreign key relationships
- **Automatic SQL generation:** ORM frameworks generate SQL queries automatically
- **Database abstraction:** Code becomes database-independent

**Benefits of ORM:**
- **Productivity:** Reduces boilerplate JDBC code
- **Maintainability:** Object-oriented approach is easier to maintain
- **Database independence:** Switch databases with minimal code changes
- **Automatic SQL generation:** No need to write SQL queries manually
- **Caching:** Built-in caching mechanisms improve performance
- **Lazy loading:** Load data only when needed

**ORM mapping example:**
```java
// Java class
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "username")
    private String username;
    
    @Column(name = "email")
    private String email;
    
    // Constructors, getters, setters
}

// Corresponding database table
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL
);
```

**Without ORM (JDBC):**
```java
// Manual JDBC code
String sql = "SELECT id, username, email FROM users WHERE id = ?";
PreparedStatement stmt = connection.prepareStatement(sql);
stmt.setLong(1, userId);
ResultSet rs = stmt.executeQuery();

if (rs.next()) {
    User user = new User();
    user.setId(rs.getLong("id"));
    user.setUsername(rs.getString("username"));
    user.setEmail(rs.getString("email"));
    return user;
}
```

**With ORM (Hibernate/JPA):**
```java
// Simple ORM code
User user = entityManager.find(User.class, userId);
```

### What is Hibernate and how does it relate to JPA?

**Answer:**
Hibernate is a popular Java ORM framework that implements the JPA (Java Persistence API) specification.

**JPA (Java Persistence API):**
- **Specification:** Standard API for ORM in Java
- **Vendor-neutral:** Not tied to any specific implementation
- **Annotations:** Provides standard annotations for mapping
- **EntityManager:** Standard interface for persistence operations
- **JPQL:** Java Persistence Query Language

**Hibernate:**
- **Implementation:** Concrete implementation of JPA specification
- **Additional features:** Provides features beyond JPA specification
- **Mature framework:** Existed before JPA specification
- **Native API:** Has its own API alongside JPA compliance

**Relationship:**
```java
// JPA standard annotations and interfaces
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // JPA EntityManager
    EntityManager entityManager;
    User user = entityManager.find(User.class, 1L);
}

// Hibernate-specific features
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE) // Hibernate-specific
public class User {
    @Id
    @GenericGenerator(name = "uuid", strategy = "uuid2") // Hibernate-specific
    @GeneratedValue(generator = "uuid")
    private String id;
    
    // Hibernate Session (native API)
    Session session = sessionFactory.getCurrentSession();
    User user = session.get(User.class, 1L);
}
```

**JPA vs Hibernate features:**
| Feature | JPA | Hibernate |
|---------|-----|-----------|
| **Standard** | Yes | Implementation |
| **Portability** | High | Hibernate-specific |
| **Caching** | Basic | Advanced (2nd level cache) |
| **Query Language** | JPQL | JPQL + HQL |
| **Lazy Loading** | Basic | Advanced strategies |
| **Custom Types** | Limited | Extensive support |

### What are the advantages of using Hibernate?

**Answer:**
Hibernate provides numerous advantages for Java persistence:

**1. Productivity Benefits:**
- **Reduced boilerplate code:** Eliminates repetitive JDBC code
- **Automatic SQL generation:** No need to write SQL queries manually
- **Object-oriented approach:** Work with objects instead of SQL
- **Rapid development:** Faster application development

**2. Performance Benefits:**
- **Lazy loading:** Load data only when accessed
- **Caching:** First-level and second-level caching
- **Connection pooling:** Efficient database connection management
- **Batch processing:** Optimize database operations

**3. Maintenance Benefits:**
- **Database independence:** Switch databases easily
- **Schema evolution:** Handle database schema changes
- **Centralized mapping:** All mapping in one place
- **Type safety:** Compile-time checking

**4. Advanced Features:**
- **Automatic dirty checking:** Detects and saves only changed data
- **Cascade operations:** Automatic related entity operations
- **Inheritance mapping:** Support for object inheritance
- **Custom types:** Define custom data types

**Code examples:**

**Lazy Loading:**
```java
@Entity
public class Department {
    @OneToMany(fetch = FetchType.LAZY, mappedBy = "department")
    private List<Employee> employees; // Loaded only when accessed
}

// Usage
Department dept = session.get(Department.class, 1L); // Department loaded
List<Employee> employees = dept.getEmployees(); // Employees loaded now
```

**Caching:**
```java
// First-level cache (automatic)
User user1 = session.get(User.class, 1L); // Database hit
User user2 = session.get(User.class, 1L); // Cache hit (same session)

// Second-level cache
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    // Entity cached across sessions
}
```

**Automatic Dirty Checking:**
```java
// Hibernate automatically detects changes
User user = session.get(User.class, 1L);
user.setEmail("newemail@example.com"); // Just modify object
// No explicit save() call needed - Hibernate detects change
transaction.commit(); // UPDATE SQL generated automatically
```

**Database Independence:**
```java
// Same code works with different databases
// Just change database dialect in configuration
hibernate.dialect=org.hibernate.dialect.MySQLDialect
hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
hibernate.dialect=org.hibernate.dialect.OracleDialect
```

## Entity Mapping

### How to use composite keys in Hibernate?

**Answer:**
Composite keys are primary keys composed of multiple columns. Hibernate provides two approaches to handle composite keys:

**1. Using @EmbeddedId with @Embeddable:**
```java
// Composite key class
@Embeddable
public class OrderItemId implements Serializable {
    @Column(name = "order_id")
    private Long orderId;
    
    @Column(name = "product_id")
    private Long productId;
    
    // Default constructor
    public OrderItemId() {}
    
    public OrderItemId(Long orderId, Long productId) {
        this.orderId = orderId;
        this.productId = productId;
    }
    
    // equals() and hashCode() methods are required
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        OrderItemId that = (OrderItemId) o;
        return Objects.equals(orderId, that.orderId) &&
               Objects.equals(productId, that.productId);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(orderId, productId);
    }
    
    // Getters and setters
}

// Entity using composite key
@Entity
@Table(name = "order_items")
public class OrderItem {
    @EmbeddedId
    private OrderItemId id;
    
    @Column(name = "quantity")
    private Integer quantity;
    
    @Column(name = "price")
    private BigDecimal price;
    
    // Constructors, getters, setters
}

// Usage
OrderItemId id = new OrderItemId(1L, 100L);
OrderItem orderItem = new OrderItem();
orderItem.setId(id);
orderItem.setQuantity(2);
orderItem.setPrice(new BigDecimal("29.99"));

session.save(orderItem);
```

**2. Using @IdClass:**
```java
// Composite key class (no @Embeddable annotation)
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
    
    // Default constructor
    public OrderItemId() {}
    
    public OrderItemId(Long orderId, Long productId) {
        this.orderId = orderId;
        this.productId = productId;
    }
    
    // equals() and hashCode() required
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        OrderItemId that = (OrderItemId) o;
        return Objects.equals(orderId, that.orderId) &&
               Objects.equals(productId, that.productId);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(orderId, productId);
    }
    
    // Getters and setters
}

// Entity using @IdClass
@Entity
@Table(name = "order_items")
@IdClass(OrderItemId.class)
public class OrderItem {
    @Id
    @Column(name = "order_id")
    private Long orderId;
    
    @Id
    @Column(name = "product_id")
    private Long productId;
    
    @Column(name = "quantity")
    private Integer quantity;
    
    @Column(name = "price")
    private BigDecimal price;
    
    // Constructors, getters, setters
}

// Usage
OrderItem orderItem = new OrderItem();
orderItem.setOrderId(1L);
orderItem.setProductId(100L);
orderItem.setQuantity(2);
orderItem.setPrice(new BigDecimal("29.99"));

session.save(orderItem);

// Querying with composite key
OrderItemId id = new OrderItemId(1L, 100L);
OrderItem found = session.get(OrderItem.class, id);
```

**3. Composite keys with relationships:**
```java
@Entity
@Table(name = "order_items")
public class OrderItem {
    @EmbeddedId
    private OrderItemId id;
    
    @ManyToOne
    @MapsId("orderId") // Maps orderId from composite key
    @JoinColumn(name = "order_id")
    private Order order;
    
    @ManyToOne
    @MapsId("productId") // Maps productId from composite key
    @JoinColumn(name = "product_id")
    private Product product;
    
    private Integer quantity;
    private BigDecimal price;
    
    // Constructors, getters, setters
}
```

### What is the difference between @Id and @EmbeddedId?

**Answer:**
@Id and @EmbeddedId are used for different types of primary keys in JPA/Hibernate:

**@Id - Simple Primary Key:**
- Used for single-column primary keys
- Applied directly to entity fields
- Can be used with @GeneratedValue for auto-generation
- Simple and straightforward approach

**@EmbeddedId - Composite Primary Key:**
- Used for multi-column primary keys
- Applied to an embedded object that contains multiple key fields
- The embedded class must be @Embeddable
- More complex but handles composite keys elegantly

**Comparison:**

| Aspect | @Id | @EmbeddedId |
|--------|-----|-------------|
| **Columns** | Single column | Multiple columns |
| **Complexity** | Simple | More complex |
| **Class requirement** | No additional class | Requires @Embeddable class |
| **Auto-generation** | Supports @GeneratedValue | No auto-generation |
| **equals/hashCode** | Not required | Required in embedded class |

**Examples:**

**@Id Example:**
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; // Single column primary key
    
    private String username;
    private String email;
    
    // Getters and setters
}

// Usage
User user = new User();
user.setUsername("john");
user.setEmail("john@example.com");
session.save(user); // ID auto-generated
```

**@EmbeddedId Example:**
```java
@Embeddable
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
    
    // Constructor, equals, hashCode, getters, setters
}

@Entity
public class OrderItem {
    @EmbeddedId
    private OrderItemId id; // Composite primary key
    
    private Integer quantity;
    private BigDecimal price;
    
    // Getters and setters
}

// Usage
OrderItemId id = new OrderItemId(1L, 100L);
OrderItem orderItem = new OrderItem();
orderItem.setId(id);
orderItem.setQuantity(5);
session.save(orderItem);
```

**When to use each:**
- **Use @Id:** For simple entities with single-column primary keys
- **Use @EmbeddedId:** For entities that naturally have composite primary keys (like junction tables, order items, etc.)

### What is the generated strategy for ID and what options are available?

**Answer:**
The @GeneratedValue annotation specifies how primary key values are generated. It provides different strategies for automatic ID generation:

**Generation Strategies:**

**1. GenerationType.IDENTITY:**
- Uses database auto-increment/identity columns
- Database-specific (MySQL AUTO_INCREMENT, SQL Server IDENTITY)
- Values generated on INSERT
- Cannot be used with batch inserts efficiently

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; // Database auto-increment
    
    private String username;
}

// Generated SQL (MySQL)
CREATE TABLE user (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255)
);
```

**2. GenerationType.SEQUENCE:**
- Uses database sequences
- Supported by databases like PostgreSQL, Oracle, H2
- More efficient for batch operations
- Can pre-allocate values

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
    @SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 1)
    private Long id;
    
    private String username;
}

// Generated SQL (PostgreSQL)
CREATE SEQUENCE user_sequence START 1;
CREATE TABLE user (
    id BIGINT PRIMARY KEY,
    username VARCHAR(255)
);
```

**3. GenerationType.TABLE:**
- Uses a separate table to generate unique values
- Database-independent approach
- Less efficient than other strategies
- Useful when database doesn't support sequences or identity

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "user_gen")
    @TableGenerator(
        name = "user_gen",
        table = "id_generator",
        pkColumnName = "gen_name",
        valueColumnName = "gen_value",
        pkColumnValue = "user_id",
        allocationSize = 1
    )
    private Long id;
    
    private String username;
}

// Generated table
CREATE TABLE id_generator (
    gen_name VARCHAR(255) PRIMARY KEY,
    gen_value BIGINT
);
```

**4. GenerationType.AUTO:**
- Hibernate chooses the best strategy based on database
- Default strategy if not specified
- Usually selects IDENTITY or SEQUENCE

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id; // Hibernate chooses strategy
    
    private String username;
}
```

**Custom Generators:**
```java
@Entity
public class User {
    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    private String id; // UUID generator
    
    private String username;
}

// Custom sequence with specific configuration
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
    @SequenceGenerator(
        name = "user_seq",
        sequenceName = "user_id_seq",
        initialValue = 1000,
        allocationSize = 50 // Pre-allocate 50 values for performance
    )
    private Long id;
    
    private String username;
}
```

**Strategy Selection Guidelines:**
- **IDENTITY:** Use with MySQL, SQL Server for simple cases
- **SEQUENCE:** Use with PostgreSQL, Oracle for better performance
- **TABLE:** Use when database portability is crucial
- **AUTO:** Use when you want Hibernate to decide

## Operations

### What is the difference between save, saveOrUpdate, and persist?

**Answer:**
These methods are used to save entities to the database, but they have different behaviors and use cases:

**save() method:**
- Hibernate-specific method (not JPA standard)
- Always generates a new identifier
- Returns the generated identifier
- Assigns identifier immediately
- Can be used for both transient and detached objects

```java
// save() example
User user = new User("john", "john@example.com");
Serializable id = session.save(user); // Returns generated ID
System.out.println("Generated ID: " + id);

// If entity already has ID, save() will still generate new one
User existingUser = new User();
existingUser.setId(100L); // Existing ID
existingUser.setUsername("jane");
Serializable newId = session.save(existingUser); // Generates new ID, ignores existing
```

**persist() method:**
- JPA standard method
- Does not return the identifier
- Assigns identifier only when transaction commits or flush occurs
- Only works with transient objects (throws exception for detached objects)
- More strict about entity state

```java
// persist() example
User user = new User("john", "john@example.com");
entityManager.persist(user); // No return value
// ID is assigned when transaction commits or flush occurs

// Will throw exception if entity is detached
User detachedUser = new User();
detachedUser.setId(100L);
detachedUser.setUsername("jane");
entityManager.persist(detachedUser); // PersistenceException
```

**saveOrUpdate() method:**
- Hibernate-specific method
- Decides whether to save or update based on entity state
- Uses identifier and version to determine if entity is transient or detached
- Convenient for scenarios where you don't know entity state

```java
// saveOrUpdate() example
User user = new User("john", "john@example.com");
session.saveOrUpdate(user); // Will save (no ID)

User existingUser = session.get(User.class, 1L);
existingUser.setEmail("newemail@example.com");
session.evict(existingUser); // Make it detached
session.saveOrUpdate(existingUser); // Will update (has ID)
```

**Comparison table:**

| Method | JPA Standard | Returns ID | Works with Transient | Works with Detached | ID Assignment |
|--------|--------------|------------|---------------------|-------------------|---------------|
| **save()** | No | Yes | Yes | Yes | Immediate |
| **persist()** | Yes | No | Yes | No | On flush/commit |
| **saveOrUpdate()** | No | No | Yes | Yes | Depends on state |

**Practical examples:**

**When to use save():**
```java
// When you need the generated ID immediately
User user = new User("john", "john@example.com");
Long userId = (Long) session.save(user);
// Use userId for other operations immediately
```

**When to use persist():**
```java
// Standard JPA approach, better for portability
User user = new User("john", "john@example.com");
entityManager.persist(user);
// ID available after flush or commit
entityManager.flush();
Long userId = user.getId();
```

**When to use saveOrUpdate():**
```java
// When entity state is unknown
public void saveUser(User user) {
    session.saveOrUpdate(user); // Handles both new and existing users
}
```

### What is the difference between load and get methods?

**Answer:**
Both load() and get() methods are used to retrieve entities from the database, but they have different behaviors:

**get() method:**
- Returns the actual object or null if not found
- Hits the database immediately (eager loading)
- Returns null if entity doesn't exist
- Safe to use - no exceptions for missing entities

**load() method:**
- Returns a proxy object
- Uses lazy loading - database hit only when properties are accessed
- Throws ObjectNotFoundException if entity doesn't exist when accessed
- More efficient for scenarios where entity might not be fully used

**Detailed comparison:**

| Aspect | get() | load() |
|--------|-------|--------|
| **Return value** | Actual object or null | Proxy object |
| **Database hit** | Immediate | Lazy (on property access) |
| **Missing entity** | Returns null | Throws ObjectNotFoundException |
| **Performance** | Less efficient | More efficient (lazy) |
| **Use case** | When you need to check existence | When you're sure entity exists |

**Code examples:**

**get() method:**
```java
// get() - immediate database hit
User user = session.get(User.class, 1L);
if (user != null) {
    System.out.println(user.getUsername()); // Object already loaded
} else {
    System.out.println("User not found");
}

// SQL executed immediately:
// SELECT * FROM users WHERE id = 1
```

**load() method:**
```java
// load() - returns proxy, no immediate database hit
User user = session.load(User.class, 1L);
System.out.println(user.getClass()); // Proxy class (User$HibernateProxy$...)

// Database hit occurs here when property is accessed
System.out.println(user.getUsername()); // SQL executed now

// If entity doesn't exist, exception thrown here
// ObjectNotFoundException: No row with the given identifier exists
```

**Practical scenarios:**

**Use get() when:**
```java
// Checking if entity exists
public boolean userExists(Long userId) {
    User user = session.get(User.class, userId);
    return user != null;
}

// When you need the entity data immediately
public String getUserDisplayName(Long userId) {
    User user = session.get(User.class, userId);
    if (user != null) {
        return user.getFirstName() + " " + user.getLastName();
    }
    return "Unknown User";
}
```

**Use load() when:**
```java
// Setting up relationships (you know entity exists)
public void assignUserToProject(Long userId, Long projectId) {
    User user = session.load(User.class, userId); // Proxy is enough
    Project project = session.load(Project.class, projectId); // Proxy is enough
    
    project.addUser(user); // No need to load full objects
    session.save(project);
}

// When you only need the entity for reference
public void createOrder(Long customerId, List<OrderItem> items) {
    Customer customer = session.load(Customer.class, customerId); // Proxy
    
    Order order = new Order();
    order.setCustomer(customer); // Just setting reference
    order.setItems(items);
    
    session.save(order);
    // Customer data never actually loaded unless accessed
}
```

**Lazy loading with load():**
```java
// Demonstrating lazy loading behavior
User userProxy = session.load(User.class, 1L);
System.out.println("Proxy created, no SQL yet");

// First property access triggers SQL
String username = userProxy.getUsername(); // SQL: SELECT * FROM users WHERE id = 1
System.out.println("Username: " + username);

// Subsequent property access uses loaded data
String email = userProxy.getEmail(); // No additional SQL
System.out.println("Email: " + email);
```

**Exception handling with load():**
```java
try {
    User user = session.load(User.class, 999L); // Non-existent ID
    // No exception yet - proxy created
    
    String username = user.getUsername(); // Exception thrown here
} catch (ObjectNotFoundException e) {
    System.out.println("User not found: " + e.getMessage());
}
```

### How to implement pagination in Hibernate?

**Answer:**
Pagination is essential for handling large datasets efficiently. Hibernate provides several approaches to implement pagination:

**1. Using setFirstResult() and setMaxResults():**
```java
// Basic pagination with Hibernate Query
public List<User> getUsersWithPagination(int pageNumber, int pageSize) {
    Session session = sessionFactory.getCurrentSession();
    
    Query<User> query = session.createQuery("FROM User u ORDER BY u.id", User.class);
    query.setFirstResult(pageNumber * pageSize); // Offset
    query.setMaxResults(pageSize); // Limit
    
    return query.getResultList();
}

// Usage
List<User> page1 = getUsersWithPagination(0, 10); // First 10 users
List<User> page2 = getUsersWithPagination(1, 10); // Next 10 users
```

**2. JPA Criteria API pagination:**
```java
public List<User> getUsersWithCriteria(int pageNumber, int pageSize) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<User> query = cb.createQuery(User.class);
    Root<User> root = query.from(User.class);
    
    query.select(root).orderBy(cb.asc(root.get("id")));
    
    TypedQuery<User> typedQuery = entityManager.createQuery(query);
    typedQuery.setFirstResult(pageNumber * pageSize);
    typedQuery.setMaxResults(pageSize);
    
    return typedQuery.getResultList();
}
```

**3. Spring Data JPA Pageable:**
```java
// Repository interface
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByUsernameContaining(String username, Pageable pageable);
    Page<User> findByActiveTrue(Pageable pageable);
}

// Service implementation
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public Page<User> getUsers(int page, int size, String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        return userRepository.findAll(pageable);
    }
    
    public Page<User> searchUsers(String username, int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return userRepository.findByUsernameContaining(username, pageable);
    }
}

// Controller usage
@RestController
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/users")
    public ResponseEntity<Page<User>> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy) {
        
        Page<User> users = userService.getUsers(page, size, sortBy);
        return ResponseEntity.ok(users);
    }
}
```

**4. Custom pagination with count query:**
```java
public class PaginationResult<T> {
    private List<T> data;
    private long totalElements;
    private int totalPages;
    private int currentPage;
    private int pageSize;
    
    // Constructors, getters, setters
}

@Repository
public class UserDao {
    
    @Autowired
    private EntityManager entityManager;
    
    public PaginationResult<User> getUsersWithPagination(int page, int size, String sortBy) {
        // Data query
        String dataQuery = "SELECT u FROM User u ORDER BY u." + sortBy;
        TypedQuery<User> query = entityManager.createQuery(dataQuery, User.class);
        query.setFirstResult(page * size);
        query.setMaxResults(size);
        List<User> users = query.getResultList();
        
        // Count query
        String countQuery = "SELECT COUNT(u) FROM User u";
        TypedQuery<Long> countTypedQuery = entityManager.createQuery(countQuery, Long.class);
        long totalElements = countTypedQuery.getSingleResult();
        
        int totalPages = (int) Math.ceil((double) totalElements / size);
        
        return new PaginationResult<>(users, totalElements, totalPages, page, size);
    }
}
```

**5. Native SQL pagination:**
```java
public List<User> getUsersWithNativeQuery(int page, int size) {
    String sql = "SELECT * FROM users ORDER BY id LIMIT :limit OFFSET :offset";
    
    Query query = entityManager.createNativeQuery(sql, User.class);
    query.setParameter("limit", size);
    query.setParameter("offset", page * size);
    
    return query.getResultList();
}

// For databases that don't support LIMIT/OFFSET
public List<User> getUsersWithNativeQuerySQLServer(int page, int size) {
    String sql = "SELECT * FROM users ORDER BY id " +
                "OFFSET :offset ROWS FETCH NEXT :limit ROWS ONLY";
    
    Query query = entityManager.createNativeQuery(sql, User.class);
    query.setParameter("offset", page * size);
    query.setParameter("limit", size);
    
    return query.getResultList();
}
```

**6. Pagination with filtering and sorting:**
```java
public Page<User> getUsersWithFilters(String username, String email, 
                                     int page, int size, String sortBy, String sortDir) {
    
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<User> query = cb.createQuery(User.class);
    Root<User> root = query.from(User.class);
    
    // Build predicates for filtering
    List<Predicate> predicates = new ArrayList<>();
    
    if (username != null && !username.isEmpty()) {
        predicates.add(cb.like(cb.lower(root.get("username")), 
                              "%" + username.toLowerCase() + "%"));
    }
    
    if (email != null && !email.isEmpty()) {
        predicates.add(cb.like(cb.lower(root.get("email")), 
                              "%" + email.toLowerCase() + "%"));
    }
    
    if (!predicates.isEmpty()) {
        query.where(cb.and(predicates.toArray(new Predicate[0])));
    }
    
    // Add sorting
    if ("desc".equalsIgnoreCase(sortDir)) {
        query.orderBy(cb.desc(root.get(sortBy)));
    } else {
        query.orderBy(cb.asc(root.get(sortBy)));
    }
    
    TypedQuery<User> typedQuery = entityManager.createQuery(query);
    typedQuery.setFirstResult(page * size);
    typedQuery.setMaxResults(size);
    
    List<User> users = typedQuery.getResultList();
    
    // Count query for total elements
    CriteriaQuery<Long> countQuery = cb.createQuery(Long.class);
    Root<User> countRoot = countQuery.from(User.class);
    countQuery.select(cb.count(countRoot));
    
    if (!predicates.isEmpty()) {
        // Rebuild predicates for count query
        List<Predicate> countPredicates = new ArrayList<>();
        if (username != null && !username.isEmpty()) {
            countPredicates.add(cb.like(cb.lower(countRoot.get("username")), 
                                      "%" + username.toLowerCase() + "%"));
        }
        if (email != null && !email.isEmpty()) {
            countPredicates.add(cb.like(cb.lower(countRoot.get("email")), 
                                      "%" + email.toLowerCase() + "%"));
        }
        countQuery.where(cb.and(countPredicates.toArray(new Predicate[0])));
    }
    
    long totalElements = entityManager.createQuery(countQuery).getSingleResult();
    
    return new PageImpl<>(users, PageRequest.of(page, size), totalElements);
}
```

**Performance considerations:**
- Use appropriate indexes on columns used in ORDER BY and WHERE clauses
- Consider using cursor-based pagination for very large datasets
- Avoid using OFFSET with large values (performance degrades)
- Use database-specific optimizations when possible

## Caching

### What is first-level cache (session) in Hibernate?

**Answer:**
First-level cache, also known as session cache, is a built-in caching mechanism in Hibernate that operates at the session level. It's mandatory and cannot be disabled.

**Key characteristics:**
- **Session-scoped:** Cache exists only within a single session
- **Automatic:** Enabled by default, no configuration needed
- **Identity map:** Ensures only one instance of an entity per session
- **Transactional:** Cache is cleared when session is closed
- **Thread-safe:** Each session has its own cache

**How it works:**
```java
Session session = sessionFactory.openSession();

// First database hit - entity loaded and cached
User user1 = session.get(User.class, 1L); 
System.out.println("First get: " + user1.getUsername());

// Second call - returns cached instance (no database hit)
User user2 = session.get(User.class, 1L);
System.out.println("Second get: " + user2.getUsername());

// Both references point to the same object
System.out.println("Same object: " + (user1 == user2)); // true

session.close(); // Cache is cleared
```

**Benefits:**
1. **Performance:** Reduces database round trips
2. **Identity guarantee:** Same entity ID returns same object instance
3. **Automatic dirty checking:** Hibernate tracks changes automatically
4. **Consistency:** Ensures data consistency within session

**Cache operations:**

**Automatic caching:**
```java
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();

// Entity automatically cached after load
User user = session.get(User.class, 1L); // Database hit

// Modify entity - change tracked in cache
user.setEmail("newemail@example.com");

// No explicit save needed - dirty checking handles it
tx.commit(); // UPDATE SQL generated automatically
session.close();
```

**Manual cache management:**
```java
Session session = sessionFactory.openSession();

User user = session.get(User.class, 1L); // Cached

// Check if entity is in cache
boolean inCache = session.contains(user); // true

// Remove entity from cache
session.evict(user);
boolean stillInCache = session.contains(user); // false

// Clear entire session cache
session.clear();

// Refresh entity from database (bypass cache)
session.refresh(user); // Forces database reload

session.close();
```

**Cache behavior with different operations:**
```java
Session session = sessionFactory.openSession();

// Load and cache entity
User user = session.get(User.class, 1L); // Database hit + cache

// Update operation uses cached entity
user.setUsername("newusername");
session.update(user); // Uses cached entity

// Delete operation uses cached entity
session.delete(user); // Uses cached entity

// Query operations may or may not use cache
List<User> users = session.createQuery("FROM User", User.class).list();
// Query results are added to cache, but query itself doesn't use cache

session.close();
```

**First-level cache limitations:**
```java
// Different sessions have separate caches
Session session1 = sessionFactory.openSession();
Session session2 = sessionFactory.openSession();

User user1 = session1.get(User.class, 1L); // Database hit
User user2 = session2.get(User.class, 1L); // Another database hit

// Different objects even though same entity
System.out.println("Same object: " + (user1 == user2)); // false

session1.close();
session2.close();
```

### What is second-level cache (sessionFactory) in Hibernate?

**Answer:**
Second-level cache is an optional, application-level cache that operates across multiple sessions. It's shared among all sessions created by the same SessionFactory.

**Key characteristics:**
- **SessionFactory-scoped:** Shared across all sessions
- **Optional:** Must be explicitly configured and enabled
- **Pluggable:** Supports different cache providers (EHCache, Hazelcast, etc.)
- **Configurable:** Can be configured per entity or collection
- **Persistent:** Survives session closure

**Configuration:**
```xml
<!-- hibernate.cfg.xml -->
<hibernate-configuration>
    <session-factory>
        <!-- Enable second-level cache -->
        <property name="hibernate.cache.use_second_level_cache">true</property>
        
        <!-- Cache provider -->
        <property name="hibernate.cache.region.factory_class">
            org.hibernate.cache.ehcache.EhCacheRegionFactory
        </property>
        
        <!-- Enable query cache -->
        <property name="hibernate.cache.use_query_cache">true</property>
        
        <!-- Cache statistics -->
        <property name="hibernate.generate_statistics">true</property>
    </session-factory>
</hibernate-configuration>
```

**Entity-level caching:**
```java
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String email;
    
    // Getters and setters
}

// Usage
Session session1 = sessionFactory.openSession();
User user1 = session1.get(User.class, 1L); // Database hit + cache
session1.close();

Session session2 = sessionFactory.openSession();
User user2 = session2.get(User.class, 1L); // Cache hit (no database)
session2.close();
```

**Collection caching:**
```java
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Department {
    @Id
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "department")
    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE) // Cache collection
    private List<Employee> employees;
    
    // Getters and setters
}
```

**Cache concurrency strategies:**
```java
// READ_ONLY - for data that never changes
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
public class Country {
    // Immutable reference data
}

// READ_WRITE - for data that can be updated
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    // Regular entity data
}

// NONSTRICT_READ_WRITE - for data updated infrequently
@Entity
@Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
public class Configuration {
    // Configuration data
}

// TRANSACTIONAL - for JTA environments
@Entity
@Cache(usage = CacheConcurrencyStrategy.TRANSACTIONAL)
public class Account {
    // Transactional data
}
```

### How to implement second-level cache?

**Answer:**
Implementing second-level cache involves configuration, entity annotation, and cache provider setup:

**1. Add dependencies:**
```xml
<!-- EHCache provider -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-ehcache</artifactId>
    <version>5.6.15.Final</version>
</dependency>

<!-- Or Hazelcast provider -->
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast-hibernate53</artifactId>
    <version>2.2.1</version>
</dependency>
```

**2. Hibernate configuration:**
```properties
# application.properties
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory
spring.jpa.properties.hibernate.cache.use_query_cache=true
spring.jpa.properties.hibernate.generate_statistics=true
```

**3. EHCache configuration (ehcache.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="ehcache.xsd">
    
    <!-- Default cache configuration -->
    <defaultCache
        maxElementsInMemory="1000"
        eternal="false"
        timeToIdleSeconds="300"
        timeToLiveSeconds="600"
        overflowToDisk="false"
        diskPersistent="false"
        memoryStoreEvictionPolicy="LRU"/>
    
    <!-- Entity-specific cache -->
    <cache name="com.example.User"
           maxElementsInMemory="500"
           eternal="false"
           timeToIdleSeconds="600"
           timeToLiveSeconds="1800"
           overflowToDisk="false"/>
    
    <!-- Collection cache -->
    <cache name="com.example.Department.employees"
           maxElementsInMemory="1000"
           eternal="false"
           timeToIdleSeconds="300"
           timeToLiveSeconds="600"
           overflowToDisk="false"/>
    
    <!-- Query cache -->
    <cache name="org.hibernate.cache.internal.StandardQueryCache"
           maxElementsInMemory="100"
           eternal="false"
           timeToIdleSeconds="120"
           timeToLiveSeconds="300"
           overflowToDisk="false"/>
</ehcache>
```

**4. Entity configuration:**
```java
@Entity
@Table(name = "users")
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE, region = "userCache")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "username")
    private String username;
    
    @Column(name = "email")
    private String email;
    
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    private List<Order> orders = new ArrayList<>();
    
    // Constructors, getters, setters
}

@Entity
@Table(name = "orders")
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;
    
    private BigDecimal amount;
    private LocalDateTime orderDate;
    
    // Constructors, getters, setters
}
```

**5. Query caching:**
```java
@Repository
public class UserRepository {
    
    @Autowired
    private EntityManager entityManager;
    
    public List<User> findActiveUsers() {
        return entityManager.createQuery(
            "SELECT u FROM User u WHERE u.active = true", User.class)
            .setHint("org.hibernate.cacheable", true) // Enable query cache
            .getResultList();
    }
    
    // Using Hibernate Session
    public List<User> findUsersByCity(String city) {
        Session session = entityManager.unwrap(Session.class);
        return session.createQuery("FROM User u WHERE u.city = :city", User.class)
            .setParameter("city", city)
            .setCacheable(true) // Enable query cache
            .list();
    }
}
```

**6. Programmatic cache management:**
```java
@Service
public class CacheService {
    
    @Autowired
    private EntityManager entityManager;
    
    @Autowired
    private SessionFactory sessionFactory;
    
    public void evictEntity(Class<?> entityClass, Serializable id) {
        sessionFactory.getCache().evict(entityClass, id);
    }
    
    public void evictEntityRegion(Class<?> entityClass) {
        sessionFactory.getCache().evict(entityClass);
    }
    
    public void evictCollection(String roleName, Serializable id) {
        sessionFactory.getCache().evictCollection(roleName, id);
    }
    
    public void evictAllRegions() {
        sessionFactory.getCache().evictAllRegions();
    }
    
    public void printCacheStatistics() {
        Statistics stats = sessionFactory.getStatistics();
        System.out.println("Cache Hit Ratio: " + 
            (double) stats.getSecondLevelCacheHitCount() / 
            stats.getSecondLevelCacheRequestCount());
        System.out.println("Query Cache Hit Ratio: " + 
            (double) stats.getQueryCacheHitCount() / 
            stats.getQueryCacheRequestCount());
    }
}
```

### What caching providers are supported by Hibernate?

**Answer:**
Hibernate supports multiple caching providers through its pluggable cache architecture:

**1. EHCache (Most Popular):**
```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-ehcache</artifactId>
    <version>5.6.15.Final</version>
</dependency>
```

```properties
hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory
```

**Features:**
- Mature and stable
- Excellent documentation
- Supports clustering
- JMX monitoring
- Disk overflow support

**2. Hazelcast:**
```xml
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast-hibernate53</artifactId>
    <version>2.2.1</version>
</dependency>
```

```properties
hibernate.cache.region.factory_class=com.hazelcast.hibernate.HazelcastCacheRegionFactory
```

**Features:**
- Distributed caching
- High availability
- Elastic scaling
- Near cache support

**3. Infinispan:**
```xml
<dependency>
    <groupId>org.infinispan</groupId>
    <artifactId>infinispan-hibernate-cache-v53</artifactId>
    <version>12.1.7.Final</version>
</dependency>
```

```properties
hibernate.cache.region.factory_class=org.infinispan.hibernate.cache.v53.InfinispanRegionFactory
```

**Features:**
- Red Hat supported
- Distributed and replicated modes
- Transaction support
- Cloud-native

**4. Redis (via Redisson):**
```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-hibernate-53</artifactId>
    <version>3.17.7</version>
</dependency>
```

```properties
hibernate.cache.region.factory_class=org.redisson.hibernate.RedissonRegionFactory
```

**Features:**
- Redis-based caching
- Distributed by nature
- High performance
- Rich data structures

**5. Caffeine:**
```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.1.1</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-jcache</artifactId>
    <version>5.6.15.Final</version>
</dependency>
```

```properties
hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
hibernate.javax.cache.provider=com.github.benmanes.caffeine.jcache.spi.CaffeineJCachingProvider
```

**Features:**
- High performance
- Modern Java 8+ API
- Excellent hit ratios
- Low memory footprint

**Comparison table:**

| Provider | Type | Clustering | Performance | Complexity | Best For |
|----------|------|------------|-------------|------------|----------|
| **EHCache** | Local/Distributed | Yes | High | Medium | General purpose |
| **Hazelcast** | Distributed | Yes | Very High | Medium | Distributed apps |
| **Infinispan** | Distributed | Yes | High | High | Enterprise apps |
| **Redis** | Distributed | Yes | Very High | Medium | High-scale apps |
| **Caffeine** | Local | No | Excellent | Low | Single-node apps |

**Configuration examples:**

**EHCache with Spring Boot:**
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        return new EhCacheCacheManager(ehCacheManagerFactory().getObject());
    }
    
    @Bean
    public EhCacheManagerFactoryBean ehCacheManagerFactory() {
        EhCacheManagerFactoryBean factory = new EhCacheManagerFactoryBean();
        factory.setConfigLocation(new ClassPathResource("ehcache.xml"));
        factory.setShared(true);
        return factory;
    }
}
```

**Hazelcast configuration:**
```java
@Configuration
public class HazelcastConfig {
    
    @Bean
    public Config hazelcastConfig() {
        Config config = new Config();
        config.setInstanceName("hibernate-cache");
        
        MapConfig mapConfig = new MapConfig();
        mapConfig.setTimeToLiveSeconds(300);
        mapConfig.setMaxIdleSeconds(600);
        config.getMapConfigs().put("default", mapConfig);
        
        return config;
    }
}
```

**Choosing the right provider:**
- **Single-node applications:** Caffeine or EHCache
- **Distributed applications:** Hazelcast or Infinispan
- **High-performance requirements:** Redis or Hazelcast
- **Enterprise environments:** Infinispan or EHCache
- **Cloud-native applications:** Hazelcast or Redis

## Relationships

### What are the different types of relationships in Hibernate?

**Answer:**
Hibernate supports four main types of entity relationships that map to relational database associations:

**1. @OneToOne:**
Represents a one-to-one relationship where each entity instance is related to exactly one instance of another entity.

```java
// Unidirectional One-to-One
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
    
    // Constructors, getters, setters
}

@Entity
public class UserProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String firstName;
    private String lastName;
    private String bio;
    
    // Constructors, getters, setters
}
```

**2. @OneToMany:**
Represents a one-to-many relationship where one entity instance is associated with multiple instances of another entity.

```java
// Bidirectional One-to-Many (Recommended)
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employee> employees = new ArrayList<>();
    
    // Helper methods for bidirectional relationship
    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }
    
    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);
    }
    
    // Constructors, getters, setters
}

@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
    
    // Constructors, getters, setters
}
```

**3. @ManyToOne:**
Represents a many-to-one relationship where multiple entity instances are associated with one instance of another entity.

```java
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private LocalDateTime orderDate;
    private BigDecimal totalAmount;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;
    
    // Constructors, getters, setters
}
```

**4. @ManyToMany:**
Represents a many-to-many relationship where multiple instances of one entity are associated with multiple instances of another entity.

```java
// Bidirectional Many-to-Many
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
    
    // Helper methods for bidirectional relationship
    public void addCourse(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }
    
    public void removeCourse(Course course) {
        courses.remove(course);
        course.getStudents().remove(this);
    }
    
    // Constructors, getters, setters
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    private String description;
    
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
    
    // Constructors, getters, setters
}
```

### What happens if we delete a parent entity with child entities?

**Answer:**
The behavior when deleting a parent entity with child entities depends on the cascade configuration and orphan removal settings:

**1. Without Cascade (Default behavior):**
```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department")
    private List<Employee> employees = new ArrayList<>();
    // No cascade specified
}

// Attempting to delete department with employees
Department dept = session.get(Department.class, 1L);
session.delete(dept); // Will throw ConstraintViolationException
// Foreign key constraint prevents deletion
```

**2. With CascadeType.REMOVE:**
```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department", cascade = CascadeType.REMOVE)
    private List<Employee> employees = new ArrayList<>();
}

// Deleting department will delete all employees
Department dept = session.get(Department.class, 1L);
session.delete(dept); // Deletes department AND all employees
```

**3. With CascadeType.ALL:**
```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Employee> employees = new ArrayList<>();
}

// All cascade operations including REMOVE
Department dept = session.get(Department.class, 1L);
session.delete(dept); // Deletes department AND all employees
```

**4. With orphanRemoval = true:**
```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employee> employees = new ArrayList<>();
}

// Orphan removal handles child entities removed from collection
Department dept = session.get(Department.class, 1L);
dept.getEmployees().clear(); // Removes all employees from collection
session.saveOrUpdate(dept); // Employees are deleted from database
```

### What are the different types of cascade operations?

**Answer:**
Cascade operations in Hibernate/JPA define how operations on parent entities should be propagated to related child entities:

**1. CascadeType.PERSIST:**
```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department", cascade = CascadeType.PERSIST)
    private List<Employee> employees = new ArrayList<>();
}

// Usage
Department dept = new Department("IT");
Employee emp1 = new Employee("John");
Employee emp2 = new Employee("Jane");

dept.addEmployee(emp1);
dept.addEmployee(emp2);

session.persist(dept); // Persists department AND employees
// No need to explicitly persist employees
```

**2. CascadeType.MERGE:**
```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department", cascade = CascadeType.MERGE)
    private List<Employee> employees = new ArrayList<>();
}

// Usage
Department detachedDept = // ... get from somewhere
Employee newEmployee = new Employee("Bob");
detachedDept.addEmployee(newEmployee);

Department mergedDept = session.merge(detachedDept); // Merges department AND employees
```

**3. CascadeType.REMOVE:**
```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department", cascade = CascadeType.REMOVE)
    private List<Employee> employees = new ArrayList<>();
}

// Usage
Department dept = session.get(Department.class, 1L);
session.remove(dept); // Removes department AND all employees
```

**4. CascadeType.ALL:**
```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Employee> employees = new ArrayList<>();
}

// Equivalent to:
@OneToMany(mappedBy = "department", cascade = {
    CascadeType.PERSIST,
    CascadeType.MERGE,
    CascadeType.REMOVE,
    CascadeType.REFRESH,
    CascadeType.DETACH
})
```

## Inheritance Mapping

### How to save parent and child classes in inheritance mapping?

**Answer:**
Hibernate provides three strategies for mapping inheritance hierarchies to database tables. The approach for saving parent and child classes depends on the chosen strategy:

**1. Single Table Strategy (TABLE_PER_CLASS_HIERARCHY):**
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "employee_type", discriminatorType = DiscriminatorType.STRING)
public abstract class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private BigDecimal salary;
    
    // Constructors, getters, setters
}

@Entity
@DiscriminatorValue("FULL_TIME")
public class FullTimeEmployee extends Employee {
    private BigDecimal annualBonus;
    private Integer vacationDays;
    
    // Constructors, getters, setters
}

@Entity
@DiscriminatorValue("PART_TIME")
public class PartTimeEmployee extends Employee {
    private BigDecimal hourlyRate;
    private Integer hoursPerWeek;
    
    // Constructors, getters, setters
}

// Saving entities
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();

// Save different types of employees
FullTimeEmployee fullTime = new FullTimeEmployee();
fullTime.setName("John Doe");
fullTime.setSalary(new BigDecimal("75000"));
fullTime.setAnnualBonus(new BigDecimal("5000"));
fullTime.setVacationDays(25);
session.save(fullTime);

PartTimeEmployee partTime = new PartTimeEmployee();
partTime.setName("Jane Smith");
partTime.setSalary(new BigDecimal("30000"));
partTime.setHourlyRate(new BigDecimal("25"));
partTime.setHoursPerWeek(20);
session.save(partTime);

tx.commit();
session.close();
```

**2. Joined Table Strategy (TABLE_PER_SUBCLASS):**
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Vehicle {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String manufacturer;
    private String model;
    private Integer year;
    
    // Constructors, getters, setters
}

@Entity
@Table(name = "cars")
public class Car extends Vehicle {
    private Integer numberOfDoors;
    private String fuelType;
    
    // Constructors, getters, setters
}

// Saving entities
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();

Car car = new Car();
car.setManufacturer("Toyota");
car.setModel("Camry");
car.setYear(2023);
car.setNumberOfDoors(4);
car.setFuelType("Gasoline");
session.save(car); // Inserts into both vehicle and cars tables

tx.commit();
session.close();
```

### What are the different inheritance mapping strategies?

**Answer:**
Hibernate/JPA provides three main inheritance mapping strategies:

**1. Single Table (TABLE_PER_CLASS_HIERARCHY):**
- All classes in hierarchy stored in single table
- Uses discriminator column to identify entity type
- Best performance (no joins required)
- Subclass columns must be nullable

**2. Joined Table (TABLE_PER_SUBCLASS):**
- Each class has its own table
- Subclass tables have foreign key to parent table
- Normalized database design
- Requires joins for queries

**3. Table Per Class (TABLE_PER_CONCRETE_CLASS):**
- Each concrete class has its own table
- No shared table for common attributes
- Good performance for concrete class queries
- Poor performance for polymorphic queries

## Annotations

### What are the common annotations used in Hibernate?

**Answer:**
Hibernate uses a combination of JPA standard annotations and Hibernate-specific annotations:

**Entity and Table Mapping:**
```java
@Entity                              // Marks class as JPA entity
@Table(name = "users",               // Maps to specific table
       schema = "public",            // Database schema
       uniqueConstraints = {
           @UniqueConstraint(columnNames = {"username", "email"})
       })
public class User {
    // Entity implementation
}
```

**Primary Key Annotations:**
```java
@Id                                  // Primary key
@GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-generation
@Column(name = "user_id")            // Column mapping
private Long id;

// Composite key
@EmbeddedId
private UserProfileId id;
```

**Column Mapping:**
```java
@Column(name = "username",           // Column name
        length = 50,                 // Column length
        nullable = false,            // NOT NULL constraint
        unique = true,               // UNIQUE constraint
        updatable = false,           // Prevent updates
        insertable = true)           // Allow inserts
private String username;

@Temporal(TemporalType.TIMESTAMP)    // Date/time mapping
private Date createdDate;

@Enumerated(EnumType.STRING)         // Enum mapping
private UserStatus status;

@Lob                                 // Large object
private byte[] profilePicture;

@Transient                           // Not persisted
private String temporaryField;
```

### How to map entity relationships using annotations?

**Answer:**
Entity relationships are mapped using JPA annotations that define the association type, fetch strategy, cascade behavior, and join configuration:

**One-to-One Relationships:**
```java
// Unidirectional One-to-One
@Entity
public class User {
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "address_id", referencedColumnName = "id")
    private Address address;
}

// Bidirectional One-to-One
@Entity
public class User {
    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private UserProfile profile;
}

@Entity
public class UserProfile {
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;
}
```

**Many-to-Many Relationships:**
```java
// Bidirectional Many-to-Many
@Entity
public class Student {
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE},
                fetch = FetchType.LAZY)
    @JoinTable(name = "student_course",
               joinColumns = @JoinColumn(name = "student_id"),
               inverseJoinColumns = @JoinColumn(name = "course_id"))
    private Set<Course> courses = new HashSet<>();
    
    // Helper methods
    public void enrollInCourse(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }
}

@Entity
public class Course {
    @ManyToMany(mappedBy = "courses", fetch = FetchType.LAZY)
    private Set<Student> students = new HashSet<>();
}
```