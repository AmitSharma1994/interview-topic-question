# Microservices Interview Questions

## Core Concepts

- What is a microservice?
- What are the advantages and disadvantages of microservices?
- What are the challenges faced while migrating monolithic applications to microservices?
- What are the design principles of microservices?

## Design Patterns

- What are the common design patterns used in microservices?
- What is the API Gateway pattern?
- What is the Circuit Breaker pattern?
- What is the Saga pattern?
- What is Event-Driven Architecture and Event Sourcing?

## Service Discovery

- What is Service Discovery and Service Registry?
- What is Eureka server and Eureka client?
- What happens if the Eureka server goes down?

## Load Balancing

- What is Load Balancing in microservices?
- How is Load Balancing implemented in Spring Cloud?
- What is client-side load balancing?

## Circuit Breaker

- What is a Circuit Breaker?
- How is Circuit Breaker implemented?
- What are the states of a Circuit Breaker?
- What is sliding window based on time and count?

## Communication

- How do microservices communicate with each other?
- What is OpenFeign and how is it used?
- What is the difference between synchronous and asynchronous communication?
- How do RabbitMQ and ActiveMQ work in microservices?

## Deployment

- How to deploy microservices without downtime?
- What is the 12-factor app methodology?
- What containerization technologies are used for microservices?

## Code Examples

```java
// Example: Circuit Breaker with Resilience4j
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping
    @CircuitBreaker(name = "productService", fallbackMethod = "fallbackGetAllProducts")
    public ResponseEntity<List<Product>> getAllProducts() {
        return ResponseEntity.ok(productService.findAll());
    }
    
    public ResponseEntity<List<Product>> fallbackGetAllProducts(Exception e) {
        // Return cached or default products
        return ResponseEntity.ok(Arrays.asList(
            new Product(1L, "Fallback Product 1", 9.99),
            new Product(2L, "Fallback Product 2", 19.99)
        ));
    }
}
```

## Resources

- [Microservices Resources](https://docs.google.com/document/d/1XFfRbq3x89fd6lng8UAA-EtTuDUgDQMpfUR4N5_JkiU/edit)