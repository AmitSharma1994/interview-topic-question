# Spring Boot Interview Questions

## Core Concepts

- What is Spring Boot and how is it different from Spring Framework?
- What are the key features of Spring Boot?
- List common annotations used in Spring Boot

## Auto-Configuration

- What is Auto-Configuration in Spring Boot?
- What is the similarity and difference between IoC container and Auto-Configuration?
- How to exclude beans from Auto-Configuration?

## Configuration

- What configuration can be kept in application.properties?
- How to override the embedded Tomcat server settings?
- How to change server port in Spring Boot?

## Annotations

- What is the difference between @Component, @Service, and @Bean?
- How to get values from query parameters, path parameters, headers, and request body?

## Developer Tools

- What are Spring Boot DevTools?
- How do they help in development?

## Actuator

- What is Spring Boot Actuator?
- How are endpoints exposed in Actuator?
- How to change log level using Actuator?

## Exception Handling

- How is exception handling done in Spring Boot?
- How would you handle error responses in a RESTful API to ensure they are informative and user-friendly?

## Transactions

- How to manage transactions in Spring Boot?
- How do transactions work internally in Spring Boot?
- What is Spring Boot transaction propagation?
- What are isolation levels in Spring Boot transactions?

## Caching

- How to implement cache in Spring Boot?
- What cache providers can be used with Spring Boot?
- How to increase the timeout of cache data?
- What are cache policies in Spring Boot?

## Security

- How would you secure a Spring Boot application against common vulnerabilities?
- How would you implement authentication and authorization in a Spring Boot application?

## Code Examples

```java
// Example: RESTful Controller with error handling
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return userService.getUserById(id)
            .map(ResponseEntity::ok)
            .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
    }
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
}
```

## Resources

- [Spring Boot Resources](https://drive.google.com/drive/u/1/folders/1DzAmcESOlL1xh7HWoz-PnD9bM543gUHw)
- [Additional Spring Boot Resources](https://docs.google.com/document/d/151rvjUYE6eyx27gA2HfHkrylUTxu0FYiYBcRB4Khz3U/edit)