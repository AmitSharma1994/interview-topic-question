# JUnit Testing Interview Questions and Answers

## JUnit Basics

### What is JUnit?

**Answer:**
JUnit is a popular open-source testing framework for Java applications. It provides annotations, assertions, and test runners to write and execute unit tests.

**Key characteristics:**
- **Unit testing framework:** Tests individual components in isolation
- **Annotation-based:** Uses annotations to define test methods and lifecycle
- **Assertion library:** Provides methods to verify expected outcomes
- **Test runners:** Executes tests and reports results
- **Integration support:** Works with IDEs, build tools, and CI/CD pipelines

**Core components:**
- **Test classes:** Contain test methods
- **Test methods:** Individual test cases
- **Assertions:** Verify expected vs actual results
- **Test lifecycle:** Setup and teardown methods
- **Test suites:** Group related tests

**Basic test structure:**
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {
    
    @Test
    void testAddition() {
        // Arrange
        Calculator calculator = new Calculator();
        
        // Act
        int result = calculator.add(2, 3);
        
        // Assert
        assertEquals(5, result);
    }
}
```

### What are the key features of JUnit 5?

**Answer:**
JUnit 5 introduced significant improvements and new features over JUnit 4:

**Architecture:**
- **JUnit Platform:** Foundation for launching testing frameworks on the JVM
- **JUnit Jupiter:** New programming and extension model
- **JUnit Vintage:** Backward compatibility with JUnit 3 and 4

**Key features:**

1. **New Annotations:**
```java
@Test                    // Test method
@DisplayName            // Custom test names
@ParameterizedTest      // Parameterized tests
@RepeatedTest          // Repeated execution
@TestFactory           // Dynamic tests
@Nested                // Nested test classes
@Tag                   // Test categorization
@Timeout               // Test timeout
@EnabledIf/@DisabledIf // Conditional execution
```

2. **Improved Assertions:**
```java
// Multiple assertions
assertAll("person",
    () -> assertEquals("John", person.getFirstName()),
    () -> assertEquals("Doe", person.getLastName())
);

// Exception assertions
assertThrows(IllegalArgumentException.class, () -> {
    calculator.divide(1, 0);
});

// Custom error messages with suppliers
assertEquals(expected, actual, () -> "Expensive message calculation");
```

3. **Dynamic Tests:**
```java
@TestFactory
Stream<DynamicTest> dynamicTests() {
    return Stream.of("apple", "banana", "orange")
        .map(fruit -> DynamicTest.dynamicTest(
            "Test " + fruit,
            () -> assertTrue(fruit.length() > 0)
        ));
}
```

4. **Parameterized Tests:**
```java
@ParameterizedTest
@ValueSource(strings = {"racecar", "radar", "able was I ere I saw elba"})
void palindromes(String candidate) {
    assertTrue(isPalindrome(candidate));
}

@ParameterizedTest
@CsvSource({
    "1, 1, 2",
    "2, 3, 5",
    "3, 5, 8"
})
void testAddition(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}
```

5. **Extension Model:**
```java
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {
    // Test implementation
}
```

### What is the difference between JUnit 4 and JUnit 5?

**Answer:**
JUnit 5 represents a major evolution from JUnit 4 with significant architectural and feature improvements:

**Architecture:**
| Aspect | JUnit 4 | JUnit 5 |
|--------|---------|---------|
| **Structure** | Monolithic JAR | Modular (Platform, Jupiter, Vintage) |
| **Java Version** | Java 5+ | Java 8+ |
| **Annotations Package** | org.junit | org.junit.jupiter.api |

**Annotations:**
| JUnit 4 | JUnit 5 | Purpose |
|---------|---------|---------|
| @Test | @Test | Test method |
| @Before | @BeforeEach | Setup before each test |
| @After | @AfterEach | Cleanup after each test |
| @BeforeClass | @BeforeAll | Setup before all tests |
| @AfterClass | @AfterAll | Cleanup after all tests |
| @Ignore | @Disabled | Skip test |
| @Category | @Tag | Test categorization |
| @RunWith | @ExtendWith | Test extensions |

**Code comparison:**
```java
// JUnit 4
import org.junit.Test;
import org.junit.Before;
import org.junit.After;
import static org.junit.Assert.*;

public class JUnit4Test {
    
    @Before
    public void setUp() {
        // Setup code
    }
    
    @Test
    public void testMethod() {
        assertEquals(expected, actual);
    }
    
    @After
    public void tearDown() {
        // Cleanup code
    }
}

// JUnit 5
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.AfterEach;
import static org.junit.jupiter.api.Assertions.*;

public class JUnit5Test {
    
    @BeforeEach
    void setUp() {
        // Setup code
    }
    
    @Test
    void testMethod() {
        assertEquals(expected, actual);
    }
    
    @AfterEach
    void tearDown() {
        // Cleanup code
    }
}
```

**New features in JUnit 5:**
- Lambda expressions support
- Multiple assertions with assertAll()
- Dynamic tests
- Parameterized tests improvements
- Nested test classes
- Custom display names
- Conditional test execution
- Timeout assertions

## Testing Layers

### How to write JUnit tests for the controller layer?

**Answer:**
Controller layer tests verify HTTP endpoints, request/response handling, and integration with the service layer.

**Using @WebMvcTest:**
```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void testGetUser_WhenUserExists_ReturnsUser() throws Exception {
        // Arrange
        Long userId = 1L;
        User user = new User(userId, "John", "john@example.com");
        when(userService.getUserById(userId)).thenReturn(Optional.of(user));
        
        // Act & Assert
        mockMvc.perform(get("/api/users/{id}", userId))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(userId))
            .andExpect(jsonPath("$.name").value("John"))
            .andExpect(jsonPath("$.email").value("john@example.com"));
        
        verify(userService).getUserById(userId);
    }
    
    @Test
    void testGetUser_WhenUserNotFound_ReturnsNotFound() throws Exception {
        // Arrange
        Long userId = 1L;
        when(userService.getUserById(userId)).thenReturn(Optional.empty());
        
        // Act & Assert
        mockMvc.perform(get("/api/users/{id}", userId))
            .andExpect(status().isNotFound());
        
        verify(userService).getUserById(userId);
    }
    
    @Test
    void testCreateUser_WithValidData_ReturnsCreatedUser() throws Exception {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("John", "john@example.com");
        User createdUser = new User(1L, "John", "john@example.com");
        when(userService.createUser(any(CreateUserRequest.class))).thenReturn(createdUser);
        
        // Act & Assert
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1L))
            .andExpect(jsonPath("$.name").value("John"));
        
        verify(userService).createUser(any(CreateUserRequest.class));
    }
    
    @Test
    void testCreateUser_WithInvalidData_ReturnsBadRequest() throws Exception {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("", "invalid-email");
        
        // Act & Assert
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest());
    }
}
```

**Testing with @SpringBootTest (Integration test):**
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserControllerIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void testGetUser_IntegrationTest() {
        // Arrange
        User savedUser = userRepository.save(new User("John", "john@example.com"));
        
        // Act
        ResponseEntity<User> response = restTemplate.getForEntity(
            "/api/users/" + savedUser.getId(), User.class);
        
        // Assert
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertEquals(savedUser.getName(), response.getBody().getName());
    }
}
```

### How to write JUnit tests for the service layer?

**Answer:**
Service layer tests focus on business logic, typically using mocks for dependencies like repositories.

**Service layer test example:**
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testCreateUser_WithValidData_ReturnsCreatedUser() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("John", "john@example.com");
        User savedUser = new User(1L, "John", "john@example.com");
        
        when(userRepository.existsByEmail("john@example.com")).thenReturn(false);
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        
        // Act
        User result = userService.createUser(request);
        
        // Assert
        assertEquals(savedUser.getId(), result.getId());
        assertEquals(savedUser.getName(), result.getName());
        assertEquals(savedUser.getEmail(), result.getEmail());
        
        verify(userRepository).existsByEmail("john@example.com");
        verify(userRepository).save(any(User.class));
        verify(emailService).sendWelcomeEmail("john@example.com");
    }
    
    @Test
    void testCreateUser_WithExistingEmail_ThrowsException() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("John", "john@example.com");
        when(userRepository.existsByEmail("john@example.com")).thenReturn(true);
        
        // Act & Assert
        assertThrows(UserAlreadyExistsException.class, () -> {
            userService.createUser(request);
        });
        
        verify(userRepository).existsByEmail("john@example.com");
        verify(userRepository, never()).save(any(User.class));
        verify(emailService, never()).sendWelcomeEmail(anyString());
    }
    
    @Test
    void testGetUserById_WhenUserExists_ReturnsUser() {
        // Arrange
        Long userId = 1L;
        User expectedUser = new User(userId, "John", "john@example.com");
        when(userRepository.findById(userId)).thenReturn(Optional.of(expectedUser));
        
        // Act
        Optional<User> result = userService.getUserById(userId);
        
        // Assert
        assertTrue(result.isPresent());
        assertEquals(expectedUser, result.get());
        verify(userRepository).findById(userId);
    }
    
    @Test
    void testDeleteUser_WhenUserExists_DeletesUser() {
        // Arrange
        Long userId = 1L;
        User existingUser = new User(userId, "John", "john@example.com");
        when(userRepository.findById(userId)).thenReturn(Optional.of(existingUser));
        
        // Act
        userService.deleteUser(userId);
        
        // Assert
        verify(userRepository).findById(userId);
        verify(userRepository).delete(existingUser);
    }
    
    @Test
    void testDeleteUser_WhenUserNotFound_ThrowsException() {
        // Arrange
        Long userId = 1L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());
        
        // Act & Assert
        assertThrows(UserNotFoundException.class, () -> {
            userService.deleteUser(userId);
        });
        
        verify(userRepository).findById(userId);
        verify(userRepository, never()).delete(any(User.class));
    }
}
```

### How to write JUnit tests for the repository layer?

**Answer:**
Repository layer tests verify data access operations, typically using @DataJpaTest for JPA repositories.

**Repository test example:**
```java
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void testFindByEmail_WhenUserExists_ReturnsUser() {
        // Arrange
        User user = new User("John", "john@example.com");
        entityManager.persistAndFlush(user);
        
        // Act
        Optional<User> result = userRepository.findByEmail("john@example.com");
        
        // Assert
        assertTrue(result.isPresent());
        assertEquals("John", result.get().getName());
        assertEquals("john@example.com", result.get().getEmail());
    }
    
    @Test
    void testFindByEmail_WhenUserNotExists_ReturnsEmpty() {
        // Act
        Optional<User> result = userRepository.findByEmail("nonexistent@example.com");
        
        // Assert
        assertFalse(result.isPresent());
    }
    
    @Test
    void testExistsByEmail_WhenUserExists_ReturnsTrue() {
        // Arrange
        User user = new User("John", "john@example.com");
        entityManager.persistAndFlush(user);
        
        // Act
        boolean exists = userRepository.existsByEmail("john@example.com");
        
        // Assert
        assertTrue(exists);
    }
    
    @Test
    void testExistsByEmail_WhenUserNotExists_ReturnsFalse() {
        // Act
        boolean exists = userRepository.existsByEmail("nonexistent@example.com");
        
        // Assert
        assertFalse(exists);
    }
    
    @Test
    void testFindByNameContaining_ReturnsMatchingUsers() {
        // Arrange
        User user1 = new User("John Doe", "john@example.com");
        User user2 = new User("Jane Doe", "jane@example.com");
        User user3 = new User("Bob Smith", "bob@example.com");
        
        entityManager.persist(user1);
        entityManager.persist(user2);
        entityManager.persist(user3);
        entityManager.flush();
        
        // Act
        List<User> result = userRepository.findByNameContaining("Doe");
        
        // Assert
        assertEquals(2, result.size());
        assertTrue(result.stream().anyMatch(u -> u.getName().equals("John Doe")));
        assertTrue(result.stream().anyMatch(u -> u.getName().equals("Jane Doe")));
    }
    
    @Test
    void testSave_CreatesNewUser() {
        // Arrange
        User user = new User("John", "john@example.com");
        
        // Act
        User savedUser = userRepository.save(user);
        
        // Assert
        assertNotNull(savedUser.getId());
        assertEquals("John", savedUser.getName());
        assertEquals("john@example.com", savedUser.getEmail());
        
        // Verify in database
        User foundUser = entityManager.find(User.class, savedUser.getId());
        assertEquals(savedUser, foundUser);
    }
}
```

## Database Testing

### How to write JUnit tests for database operations?

**Answer:**
Database testing in JUnit involves testing data access layer operations with proper test data setup and cleanup.

**Using @DataJpaTest:**
```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class DatabaseOperationTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    @Transactional
    @Rollback
    void testComplexDatabaseOperation() {
        // Arrange
        User user = new User("John", "john@example.com");
        entityManager.persistAndFlush(user);
        
        // Act
        userRepository.updateUserStatus(user.getId(), "ACTIVE");
        entityManager.clear(); // Clear persistence context
        
        // Assert
        User updatedUser = entityManager.find(User.class, user.getId());
        assertEquals("ACTIVE", updatedUser.getStatus());
    }
    
    @Test
    void testBatchOperation() {
        // Arrange
        List<User> users = Arrays.asList(
            new User("John", "john@example.com"),
            new User("Jane", "jane@example.com"),
            new User("Bob", "bob@example.com")
        );
        
        // Act
        List<User> savedUsers = userRepository.saveAll(users);
        
        // Assert
        assertEquals(3, savedUsers.size());
        assertEquals(3, userRepository.count());
    }
    
    @Test
    void testCustomQuery() {
        // Arrange
        User activeUser = new User("John", "john@example.com");
        activeUser.setStatus("ACTIVE");
        User inactiveUser = new User("Jane", "jane@example.com");
        inactiveUser.setStatus("INACTIVE");
        
        entityManager.persist(activeUser);
        entityManager.persist(inactiveUser);
        entityManager.flush();
        
        // Act
        List<User> activeUsers = userRepository.findByStatus("ACTIVE");
        
        // Assert
        assertEquals(1, activeUsers.size());
        assertEquals("John", activeUsers.get(0).getName());
    }
}
```

### What is @DataJpaTest and how is it used?

**Answer:**
@DataJpaTest is a Spring Boot test annotation that provides a test slice for JPA repositories, configuring only the components needed for JPA testing.

**Key features:**
- Configures in-memory database (H2 by default)
- Configures Spring Data JPA repositories
- Configures TestEntityManager
- Enables SQL logging
- Applies @Transactional with rollback
- Disables full auto-configuration

**Basic usage:**
```java
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void testRepositoryOperation() {
        // Test implementation
    }
}
```

**Configuration options:**
```java
// Use real database instead of in-memory
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryTest {
    // Test implementation
}

// Include additional components
@DataJpaTest
@Import({UserService.class, CustomConfiguration.class})
class UserRepositoryTest {
    // Test implementation
}

// Exclude auto-configuration
@DataJpaTest(excludeAutoConfiguration = {FlywayAutoConfiguration.class})
class UserRepositoryTest {
    // Test implementation
}
```

**TestEntityManager usage:**
```java
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void testWithTestEntityManager() {
        // Persist test data
        User user = new User("John", "john@example.com");
        User savedUser = entityManager.persistAndFlush(user);
        
        // Test repository method
        Optional<User> found = userRepository.findById(savedUser.getId());
        
        assertTrue(found.isPresent());
        assertEquals("John", found.get().getName());
    }
    
    @Test
    void testEntityManagerOperations() {
        // Create and persist
        User user = new User("John", "john@example.com");
        entityManager.persist(user);
        
        // Flush to database
        entityManager.flush();
        
        // Clear persistence context
        entityManager.clear();
        
        // Find by ID
        User foundUser = entityManager.find(User.class, user.getId());
        assertNotNull(foundUser);
    }
}
```

### How to set up an in-memory database for testing?

**Answer:**
In-memory databases provide fast, isolated testing environments that reset between test runs.

**H2 Database setup (default with @DataJpaTest):**
```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

**Test configuration:**
```properties
# application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true

# Enable H2 console for debugging
spring.h2.console.enabled=true
```

**Test class setup:**
```java
@DataJpaTest
@TestPropertySource(locations = "classpath:application-test.properties")
class DatabaseTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void testInMemoryDatabase() {
        // Test runs against H2 in-memory database
        User user = new User("John", "john@example.com");
        User saved = userRepository.save(user);
        
        assertNotNull(saved.getId());
        assertTrue(userRepository.existsById(saved.getId()));
    }
}
```

**Custom database configuration:**
```java
@TestConfiguration
public class TestDatabaseConfig {
    
    @Bean
    @Primary
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .addScript("classpath:schema.sql")
            .addScript("classpath:test-data.sql")
            .build();
    }
}

@DataJpaTest
@Import(TestDatabaseConfig.class)
class CustomDatabaseTest {
    // Test implementation
}
```

**SQL scripts for test data:**
```sql
-- schema.sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    status VARCHAR(50) DEFAULT 'ACTIVE'
);

-- test-data.sql
INSERT INTO users (name, email, status) VALUES 
('John Doe', 'john@example.com', 'ACTIVE'),
('Jane Smith', 'jane@example.com', 'INACTIVE');
```

**Using @Sql annotation:**
```java
@DataJpaTest
class SqlScriptTest {
    
    @Test
    @Sql("/test-data.sql")
    void testWithSqlScript() {
        // Test runs with data from SQL script
        assertEquals(2, userRepository.count());
    }
    
    @Test
    @Sql(scripts = "/test-data.sql", executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD)
    @Sql(scripts = "/cleanup.sql", executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
    void testWithSetupAndCleanup() {
        // Test with setup and cleanup scripts
    }
}
```

## Mocking

### How to write JUnit tests for static methods?

**Answer:**
Testing static methods requires special handling since they can't be mocked with standard Mockito. Here are different approaches:

**1. Using Mockito with mockStatic (Mockito 3.4+):**
```java
class StaticMethodTest {
    
    @Test
    void testStaticMethod() {
        try (MockedStatic<StringUtils> mockedStatic = mockStatic(StringUtils.class)) {
            // Arrange
            mockedStatic.when(() -> StringUtils.isEmpty("test")).thenReturn(true);
            
            // Act
            boolean result = StringUtils.isEmpty("test");
            
            // Assert
            assertTrue(result);
            mockedStatic.verify(() -> StringUtils.isEmpty("test"));
        }
    }
    
    @Test
    void testServiceWithStaticDependency() {
        try (MockedStatic<FileUtils> mockedFileUtils = mockStatic(FileUtils.class)) {
            // Arrange
            FileService fileService = new FileService();
            mockedFileUtils.when(() -> FileUtils.readFile("test.txt"))
                          .thenReturn("mocked content");
            
            // Act
            String content = fileService.processFile("test.txt");
            
            // Assert
            assertEquals("MOCKED CONTENT", content);
            mockedFileUtils.verify(() -> FileUtils.readFile("test.txt"));
        }
    }
}
```

**2. Wrapper approach (recommended for better testability):**
```java
// Wrapper class for static methods
@Component
public class FileUtilsWrapper {
    public String readFile(String filename) {
        return FileUtils.readFile(filename);
    }
}

// Service using wrapper
@Service
public class FileService {
    private final FileUtilsWrapper fileUtils;
    
    public FileService(FileUtilsWrapper fileUtils) {
        this.fileUtils = fileUtils;
    }
    
    public String processFile(String filename) {
        String content = fileUtils.readFile(filename);
        return content.toUpperCase();
    }
}

// Test with mocked wrapper
@ExtendWith(MockitoExtension.class)
class FileServiceTest {
    
    @Mock
    private FileUtilsWrapper fileUtils;
    
    @InjectMocks
    private FileService fileService;
    
    @Test
    void testProcessFile() {
        // Arrange
        when(fileUtils.readFile("test.txt")).thenReturn("content");
        
        // Act
        String result = fileService.processFile("test.txt");
        
        // Assert
        assertEquals("CONTENT", result);
        verify(fileUtils).readFile("test.txt");
    }
}
```

**3. Testing static methods directly:**
```java
class MathUtilsTest {
    
    @Test
    void testStaticMethodDirectly() {
        // Test static method without mocking
        int result = MathUtils.add(2, 3);
        assertEquals(5, result);
    }
    
    @ParameterizedTest
    @CsvSource({
        "2, 3, 5",
        "0, 0, 0",
        "-1, 1, 0"
    })
    void testStaticMethodWithParameters(int a, int b, int expected) {
        assertEquals(expected, MathUtils.add(a, b));
    }
}
```

### How to use Mockito with JUnit?

**Answer:**
Mockito is a popular mocking framework that integrates seamlessly with JUnit for creating test doubles.

**Setup and basic usage:**
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testCreateUser() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("John", "john@example.com");
        User savedUser = new User(1L, "John", "john@example.com");
        
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        
        // Act
        User result = userService.createUser(request);
        
        // Assert
        assertEquals(savedUser, result);
        verify(userRepository).save(any(User.class));
        verify(emailService).sendWelcomeEmail("john@example.com");
    }
}
```

**Different ways to create mocks:**
```java
class MockCreationTest {
    
    // 1. Using @Mock annotation
    @Mock
    private UserRepository userRepository;
    
    // 2. Using Mockito.mock() in setup
    private UserRepository userRepository;
    
    @BeforeEach
    void setUp() {
        userRepository = Mockito.mock(UserRepository.class);
    }
    
    // 3. Using MockitoAnnotations.openMocks()
    @BeforeEach
    void initMocks() {
        MockitoAnnotations.openMocks(this);
    }
}
```

**Stubbing methods:**
```java
@Test
void testStubbingExamples() {
    UserRepository mockRepository = mock(UserRepository.class);
    
    // Basic stubbing
    when(mockRepository.findById(1L)).thenReturn(Optional.of(new User()));
    
    // Multiple return values
    when(mockRepository.count())
        .thenReturn(1L)
        .thenReturn(2L)
        .thenReturn(3L);
    
    // Stubbing with exceptions
    when(mockRepository.findById(999L))
        .thenThrow(new UserNotFoundException("User not found"));
    
    // Stubbing void methods
    doNothing().when(mockRepository).delete(any(User.class));
    doThrow(new RuntimeException()).when(mockRepository).deleteAll();
    
    // Argument matchers
    when(mockRepository.findByEmail(anyString())).thenReturn(Optional.empty());
    when(mockRepository.findByAge(eq(25))).thenReturn(Arrays.asList(new User()));
    
    // Custom argument matchers
    when(mockRepository.save(argThat(user -> user.getEmail().contains("@"))))
        .thenReturn(new User());
}
```

**Verification:**
```java
@Test
void testVerificationExamples() {
    UserRepository mockRepository = mock(UserRepository.class);
    UserService userService = new UserService(mockRepository);
    
    // Act
    userService.deleteUser(1L);
    
    // Verify method calls
    verify(mockRepository).findById(1L);
    verify(mockRepository).delete(any(User.class));
    
    // Verify number of interactions
    verify(mockRepository, times(1)).findById(1L);
    verify(mockRepository, never()).save(any(User.class));
    verify(mockRepository, atLeast(1)).delete(any(User.class));
    verify(mockRepository, atMost(2)).findById(anyLong());
    
    // Verify no more interactions
    verifyNoMoreInteractions(mockRepository);
    
    // Verify in order
    InOrder inOrder = inOrder(mockRepository);
    inOrder.verify(mockRepository).findById(1L);
    inOrder.verify(mockRepository).delete(any(User.class));
}
```

**Spying on real objects:**
```java
@Test
void testSpying() {
    // Create spy of real object
    List<String> spyList = spy(new ArrayList<>());
    
    // Use real methods
    spyList.add("one");
    spyList.add("two");
    
    // Verify real method calls
    verify(spyList).add("one");
    verify(spyList).add("two");
    
    assertEquals(2, spyList.size());
    
    // Stub specific methods
    when(spyList.size()).thenReturn(100);
    assertEquals(100, spyList.size());
}
```

### What is PowerMockito and when would you use it?

**Answer:**
PowerMockito extends Mockito's capabilities to mock static methods, constructors, final classes, and private methods. However, it's generally recommended to use design patterns that avoid these needs.

**When to use PowerMockito:**
- Legacy code with static dependencies
- Third-party libraries with static methods
- Final classes that can't be mocked with standard Mockito
- Private method testing (though this is often a code smell)

**Setup:**
```xml
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>2.0.9</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito2</artifactId>
    <version>2.0.9</version>
    <scope>test</scope>
</dependency>
```

**Usage examples:**
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({StaticUtils.class, System.class})
public class PowerMockitoTest {
    
    @Test
    public void testStaticMethod() {
        // Mock static method
        PowerMockito.mockStatic(StaticUtils.class);
        PowerMockito.when(StaticUtils.getValue()).thenReturn("mocked");
        
        // Test
        String result = StaticUtils.getValue();
        assertEquals("mocked", result);
        
        // Verify
        PowerMockito.verifyStatic(StaticUtils.class);
        StaticUtils.getValue();
    }
    
    @Test
    public void testConstructor() throws Exception {
        // Mock constructor
        User mockUser = PowerMockito.mock(User.class);
        PowerMockito.whenNew(User.class)
                   .withArguments("John", "john@example.com")
                   .thenReturn(mockUser);
        
        // Test
        User user = new User("John", "john@example.com");
        assertEquals(mockUser, user);
    }
    
    @Test
    public void testPrivateMethod() throws Exception {
        UserService userService = PowerMockito.spy(new UserService());
        
        // Mock private method
        PowerMockito.doReturn("mocked").when(userService, "privateMethod", "input");
        
        // Test method that calls private method
        String result = userService.publicMethodThatCallsPrivate("input");
        assertEquals("mocked", result);
    }
}
```

**Modern alternatives (recommended):**
```java
// Instead of PowerMockito, use dependency injection
@Service
public class UserService {
    private final TimeProvider timeProvider;
    
    public UserService(TimeProvider timeProvider) {
        this.timeProvider = timeProvider;
    }
    
    public User createUser(String name) {
        User user = new User(name);
        user.setCreatedAt(timeProvider.now()); // Instead of System.currentTimeMillis()
        return user;
    }
}

// Test with mockable dependency
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private TimeProvider timeProvider;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testCreateUser() {
        // Arrange
        LocalDateTime fixedTime = LocalDateTime.of(2023, 1, 1, 12, 0);
        when(timeProvider.now()).thenReturn(fixedTime);
        
        // Act
        User user = userService.createUser("John");
        
        // Assert
        assertEquals(fixedTime, user.getCreatedAt());
    }
}
```

## Test Lifecycle

### How to execute code before each test method?

**Answer:**
JUnit provides lifecycle annotations to execute setup and cleanup code at different points in the test execution.

**@BeforeEach annotation:**
```java
class UserServiceTest {
    
    private UserService userService;
    private UserRepository userRepository;
    
    @BeforeEach
    void setUp() {
        // Executed before each test method
        userRepository = mock(UserRepository.class);
        userService = new UserService(userRepository);
        
        System.out.println("Setting up test");
    }
    
    @Test
    void testCreateUser() {
        // Test implementation
        // setUp() is called before this method
    }
    
    @Test
    void testDeleteUser() {
        // Test implementation
        // setUp() is called before this method too
    }
    
    @AfterEach
    void tearDown() {
        // Executed after each test method
        System.out.println("Cleaning up after test");
        // Reset mocks, clean resources, etc.
    }
}
```

**Multiple @BeforeEach methods:**
```java
class MultipleSetupTest {
    
    @BeforeEach
    void setupDatabase() {
        // Database setup
        System.out.println("Setting up database");
    }
    
    @BeforeEach
    void setupMocks() {
        // Mock setup
        System.out.println("Setting up mocks");
    }
    
    @Test
    void testMethod() {
        // Both setup methods are called before this test
        // Order is not guaranteed
    }
}
```

**Inheritance and lifecycle methods:**
```java
class BaseTest {
    
    @BeforeEach
    void baseSetup() {
        System.out.println("Base setup");
    }
    
    @AfterEach
    void baseCleanup() {
        System.out.println("Base cleanup");
    }
}

class UserServiceTest extends BaseTest {
    
    @BeforeEach
    void specificSetup() {
        System.out.println("Specific setup");
    }
    
    @Test
    void testMethod() {
        // Execution order:
        // 1. baseSetup()
        // 2. specificSetup()
        // 3. testMethod()
        // 4. baseCleanup()
    }
}
```

### How to execute code once before all test methods?

**Answer:**
Use @BeforeAll and @AfterAll annotations to execute code once per test class.

**@BeforeAll and @AfterAll:**
```java
class DatabaseIntegrationTest {
    
    private static TestDatabase testDatabase;
    
    @BeforeAll
    static void setUpClass() {
        // Executed once before all test methods
        System.out.println("Starting test database");
        testDatabase = new TestDatabase();
        testDatabase.start();
        
        // Initialize expensive resources
        loadTestData();
    }
    
    @AfterAll
    static void tearDownClass() {
        // Executed once after all test methods
        System.out.println("Stopping test database");
        if (testDatabase != null) {
            testDatabase.stop();
        }
    }
    
    @BeforeEach
    void setUp() {
        // Executed before each test
        testDatabase.clearData();
    }
    
    @Test
    void testDatabaseOperation1() {
        // Test implementation
    }
    
    @Test
    void testDatabaseOperation2() {
        // Test implementation
    }
    
    private static void loadTestData() {
        // Load test data once
    }
}
```

**Non-static approach with @TestInstance:**
```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class NonStaticLifecycleTest {
    
    private TestDatabase testDatabase;
    
    @BeforeAll
    void setUpClass() {
        // Non-static method (due to @TestInstance)
        testDatabase = new TestDatabase();
        testDatabase.start();
    }
    
    @AfterAll
    void tearDownClass() {
        // Non-static method
        testDatabase.stop();
    }
    
    @Test
    void testMethod() {
        // Test implementation
    }
}
```

### What are the JUnit lifecycle annotations?

**Answer:**
JUnit 5 provides several lifecycle annotations to control test execution flow:

**Lifecycle annotations:**

| Annotation | Purpose | Execution | Method Type |
|------------|---------|-----------|-------------|
| @BeforeAll | Setup before all tests | Once per class | Static (or non-static with @TestInstance) |
| @BeforeEach | Setup before each test | Before each test method | Instance method |
| @AfterEach | Cleanup after each test | After each test method | Instance method |
| @AfterAll | Cleanup after all tests | Once per class | Static (or non-static with @TestInstance) |

**Complete lifecycle example:**
```java
class LifecycleTest {
    
    @BeforeAll
    static void initAll() {
        System.out.println("@BeforeAll - executed once before all tests");
    }
    
    @BeforeEach
    void init() {
        System.out.println("@BeforeEach - executed before each test");
    }
    
    @Test
    void testMethod1() {
        System.out.println("Test method 1");
    }
    
    @Test
    void testMethod2() {
        System.out.println("Test method 2");
    }
    
    @AfterEach
    void cleanup() {
        System.out.println("@AfterEach - executed after each test");
    }
    
    @AfterAll
    static void cleanupAll() {
        System.out.println("@AfterAll - executed once after all tests");
    }
}

// Output:
// @BeforeAll - executed once before all tests
// @BeforeEach - executed before each test
// Test method 1
// @AfterEach - executed after each test
// @BeforeEach - executed before each test
// Test method 2
// @AfterEach - executed after each test
// @AfterAll - executed once after all tests
```

**Nested test lifecycle:**
```java
class NestedLifecycleTest {
    
    @BeforeAll
    static void outerBeforeAll() {
        System.out.println("Outer @BeforeAll");
    }
    
    @BeforeEach
    void outerBeforeEach() {
        System.out.println("Outer @BeforeEach");
    }
    
    @Test
    void outerTest() {
        System.out.println("Outer test");
    }
    
    @Nested
    class InnerTests {
        
        @BeforeEach
        void innerBeforeEach() {
            System.out.println("Inner @BeforeEach");
        }
        
        @Test
        void innerTest() {
            System.out.println("Inner test");
            // Execution order for this test:
            // 1. outerBeforeAll()
            // 2. outerBeforeEach()
            // 3. innerBeforeEach()
            // 4. innerTest()
        }
    }
}
```

## Configuration

### How to mock the application.properties file for JUnit test cases?

**Answer:**
There are several ways to override or mock application properties in JUnit tests:

**1. Using @TestPropertySource:**
```java
@SpringBootTest
@TestPropertySource(properties = {
    "app.name=Test Application",
    "app.version=1.0.0-TEST",
    "spring.datasource.url=jdbc:h2:mem:testdb"
})
class PropertyOverrideTest {
    
    @Value("${app.name}")
    private String appName;
    
    @Test
    void testPropertyOverride() {
        assertEquals("Test Application", appName);
    }
}
```

**2. Using test-specific properties file:**
```java
@SpringBootTest
@TestPropertySource(locations = "classpath:application-test.properties")
class TestPropertiesFileTest {
    
    @Autowired
    private Environment environment;
    
    @Test
    void testTestProperties() {
        assertEquals("test-value", environment.getProperty("test.property"));
    }
}
```

**3. Using @ActiveProfiles:**
```java
@SpringBootTest
@ActiveProfiles("test")
class ProfileBasedTest {
    // Uses application-test.properties automatically
    
    @Value("${database.url}")
    private String databaseUrl;
    
    @Test
    void testProfileSpecificProperty() {
        assertEquals("jdbc:h2:mem:testdb", databaseUrl);
    }
}
```

**4. Using @DynamicPropertySource (JUnit 5.2.5+):**
```java
@SpringBootTest
class DynamicPropertyTest {
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("app.external.url", () -> "http://localhost:8080");
        registry.add("app.timeout", () -> "5000");
    }
    
    @Value("${app.external.url}")
    private String externalUrl;
    
    @Test
    void testDynamicProperty() {
        assertEquals("http://localhost:8080", externalUrl);
    }
}
```

**5. Using @MockBean for configuration classes:**
```java
@SpringBootTest
class MockConfigurationTest {
    
    @MockBean
    private AppConfig appConfig;
    
    @Test
    void testWithMockedConfig() {
        // Mock configuration values
        when(appConfig.getMaxRetries()).thenReturn(5);
        when(appConfig.getTimeout()).thenReturn(Duration.ofSeconds(10));
        
        // Test with mocked configuration
    }
}
```

**Test properties files:**
```properties
# application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.hibernate.ddl-auto=create-drop

# Test-specific properties
test.property=test-value
app.feature.enabled=false
external.service.url=http://mock-service:8080
```

### How to configure different properties for testing?

**Answer:**
Spring Boot provides multiple mechanisms to configure different properties for testing environments:

**1. Profile-based configuration:**
```yaml
# application.yml (default)
app:
  name: Production App
  database:
    url: jdbc:mysql://prod-server:3306/proddb
  external:
    service-url: https://api.production.com

---
# application-test.yml
spring:
  config:
    activate:
      on-profile: test

app:
  name: Test App
  database:
    url: jdbc:h2:mem:testdb
  external:
    service-url: http://localhost:8080
```

**Test class using profiles:**
```java
@SpringBootTest
@ActiveProfiles("test")
class ProfileConfigurationTest {
    
    @Value("${app.name}")
    private String appName;
    
    @Value("${app.database.url}")
    private String databaseUrl;
    
    @Test
    void testProfileConfiguration() {
        assertEquals("Test App", appName);
        assertEquals("jdbc:h2:mem:testdb", databaseUrl);
    }
}
```

**2. Test configuration classes:**
```java
@TestConfiguration
public class TestConfig {
    
    @Bean
    @Primary
    public ExternalService externalService() {
        return Mockito.mock(ExternalService.class);
    }
    
    @Bean
    @Primary
    public DataSource testDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}

@SpringBootTest
@Import(TestConfig.class)
class ConfigurationOverrideTest {
    
    @Autowired
    private ExternalService externalService; // This will be the mock
    
    @Test
    void testWithOverriddenConfiguration() {
        // Test with mocked external service
    }
}
```

**3. Environment-specific test properties:**
```java
@SpringBootTest
@TestPropertySource(properties = {
    "logging.level.org.springframework=DEBUG",
    "spring.jpa.show-sql=true",
    "app.feature.new-algorithm=true"
})
class EnvironmentSpecificTest {
    
    @Test
    void testWithSpecificProperties() {
        // Test with debug logging and new algorithm enabled
    }
}
```

**4. Conditional configuration:**
```java
@Configuration
@Profile("!test")
public class ProductionConfig {
    
    @Bean
    public ExternalService externalService() {
        return new RealExternalService();
    }
}

@TestConfiguration
@Profile("test")
public class TestConfig {
    
    @Bean
    public ExternalService externalService() {
        return new MockExternalService();
    }
}
```

**5. Property source hierarchy:**
```java
@SpringBootTest
@TestPropertySource(
    locations = "classpath:test.properties",
    properties = {
        "override.property=test-value"
    }
)
class PropertyHierarchyTest {
    // Properties are loaded in order:
    // 1. application.properties
    // 2. test.properties
    // 3. inline properties (highest priority)
}
```

**6. Container-based testing configuration:**
```java
@SpringBootTest
@Testcontainers
class ContainerConfigurationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Test
    void testWithRealDatabase() {
        // Test with real PostgreSQL database in container
    }
}
```