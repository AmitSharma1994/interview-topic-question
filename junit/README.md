# JUnit Testing Interview Questions

## JUnit Basics

- What is JUnit?
- What are the key features of JUnit 5?
- What is the difference between JUnit 4 and JUnit 5?

## Testing Layers

- How to write JUnit tests for the controller layer?
- How to write JUnit tests for the service layer?
- How to write JUnit tests for the repository layer?

## Database Testing

- How to write JUnit tests for database operations?
- What is @DataJpaTest and how is it used?
- How to set up an in-memory database for testing?

## Mocking

- How to write JUnit tests for static methods?
- How to use Mockito with JUnit?
- What is PowerMockito and when would you use it?

## Test Lifecycle

- How to execute code before each test method?
- How to execute code once before all test methods?
- What are the JUnit lifecycle annotations?

## Configuration

- How to mock the application.properties file for JUnit test cases?
- How to configure different properties for testing?

## Code Examples

```java
// Example: Service layer test with Mockito
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
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
    void testGetUserById_WhenUserDoesNotExist_ReturnsEmpty() {
        // Arrange
        Long userId = 1L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());
        
        // Act
        Optional<User> result = userService.getUserById(userId);
        
        // Assert
        assertFalse(result.isPresent());
        verify(userRepository).findById(userId);
    }
}
```

## Resources

- [JUnit Testing Resources](https://drive.google.com/drive/u/1/folders/1w08LbwRgl0S92aQ7XR3vLkTbWqDmYP7f)