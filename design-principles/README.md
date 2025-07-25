# Design Principles Interview Questions

## SOLID Principles

### Single Responsibility Principle (SRP)
- What is the Single Responsibility Principle?
- How does it improve code maintainability?
- Provide an example of SRP in practice

### Open/Closed Principle (OCP)
- What is the Open/Closed Principle?
- How does it help with code extensibility?
- Provide an example of OCP in practice

### Liskov Substitution Principle (LSP)
- What is the Liskov Substitution Principle?
- How does it ensure proper inheritance hierarchies?
- Provide an example of LSP in practice

### Interface Segregation Principle (ISP)
- What is the Interface Segregation Principle?
- How does it prevent interface pollution?
- Provide an example of ISP in practice

### Dependency Inversion Principle (DIP)
- What is the Dependency Inversion Principle?
- How does it promote loose coupling?
- Provide an example of DIP in practice

## Other Design Principles

### DRY (Don't Repeat Yourself)
- What is the DRY principle?
- How does it reduce code duplication?

### KISS (Keep It Simple, Stupid)
- What is the KISS principle?
- How does it improve code readability?

### YAGNI (You Aren't Gonna Need It)
- What is the YAGNI principle?
- How does it prevent over-engineering?

### Law of Demeter (Principle of Least Knowledge)
- What is the Law of Demeter?
- How does it reduce coupling between objects?

## System Design Questions

### URL Shortener
- How would you design a URL shortener service like TinyURL?
- What are the key components required?
- How would you handle high traffic?

### API Design
- What are the key considerations when designing an API?
- How would you ensure API security?
- How would you version an API?

### Caching Strategies
- How would you design caching for a write-heavy system?
- How would you design caching for a read-heavy system?
- What caching strategies would you use for different scenarios?

### Performance Optimization
- How would you increase the performance of an application?
- What tools would you use to identify performance bottlenecks?
- What are common performance optimization techniques?

## Code Examples

```java
// Example: Single Responsibility Principle
// Before: Class with multiple responsibilities
public class User {
    private String name;
    private String email;
    
    public void saveToDatabase() {
        // Database logic here
    }
    
    public void sendEmail(String message) {
        // Email sending logic here
    }
    
    public void generateReport() {
        // Report generation logic here
    }
}

// After: Separated responsibilities
public class User {
    private String name;
    private String email;
    
    // Only user-related methods
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}

public class UserRepository {
    public void save(User user) {
        // Database logic here
    }
}

public class EmailService {
    public void sendEmail(User user, String message) {
        // Email sending logic here
    }
}

public class ReportGenerator {
    public void generateUserReport(User user) {
        // Report generation logic here
    }
}
```

## Resources

- [Design Principles Resources](https://docs.google.com/document/d/1nghsJyDc9Nsxr5S9j8s_IJ27EPYoGU89v3Bw9X0hPwA/edit)