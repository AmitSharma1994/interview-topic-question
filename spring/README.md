# Spring Framework Interview Questions

## Dependency Injection

- What is IoC container, DI, and inversion of control?
- What is the difference between @Inject and @Autowired?
- What are the types of autowiring in Spring?
- Compare setter injection and constructor injection. Which one is better and why?

## Spring Configuration

- How does Spring configuration work?
- What are the different ways to configure a Spring application?
- What is the difference between BeanFactory and ApplicationContext?

## Spring Beans

- What is a component and a bean in Spring?
- What is the lifecycle of a Spring bean?
- What are the scopes of Spring beans?
- What happens if a prototype-scoped bean is defined inside a singleton-scoped bean?

## Dependency Resolution

- What is circular dependency in Spring Framework and how to resolve it?
- What is the use of @Qualifier annotation and @Primary annotation?

## Stereotypes

- What are stereotype annotations in Spring?
- Can we exchange stereotype annotations with each other? (e.g., replace @Service with @Configuration)

## Exception Handling

- How is exception handling done in Spring using @ControllerAdvice and @ExceptionHandler?

## Code Examples

```java
// Example: Constructor injection (preferred approach)
@Service
public class UserService {
    private final UserRepository userRepository;
    
    @Autowired // Optional in newer Spring versions
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    // Service methods
}
```

## Resources

- [Spring Framework Resources](https://docs.google.com/document/d/1qCnAAsyLDKE3NqJ0w9ZP4obVWI6MHaQIjkuXo2agJk4/edit)