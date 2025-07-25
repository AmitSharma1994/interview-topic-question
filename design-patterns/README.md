# Design Patterns Interview Questions

## Creational Design Patterns

### Singleton Pattern
- What is the Singleton Design Pattern?
- How to implement a thread-safe Singleton?
- How can the Singleton pattern be broken? (Reflection API, Serialization, Cloning)

### Prototype Pattern
- What is the Prototype Design Pattern?
- When would you use the Prototype pattern?

### Builder Pattern
- What is the Builder Design Pattern?
- How does it help with creating complex objects?

### Factory Pattern
- What is the Factory Design Pattern?
- What is the difference between Simple Factory and Factory Method?

### Abstract Factory Pattern
- What is the Abstract Factory Design Pattern?
- How is it different from the Factory Method pattern?

## Structural Design Patterns

### Adapter Pattern
- What is the Adapter Pattern?
- When would you use the Adapter pattern?

### Bridge Pattern
- What is the Bridge Pattern?
- How does it separate abstraction from implementation?

### Composite Pattern
- What is the Composite Pattern?
- When would you use the Composite pattern?

### Decorator Pattern
- What is the Decorator Pattern?
- How is it different from inheritance?

### Facade Pattern
- What is the Facade Pattern?
- How does it simplify complex systems?

### Flyweight Pattern
- What is the Flyweight Pattern?
- When would you use the Flyweight pattern?

### Proxy Pattern
- What is the Proxy Pattern?
- What are the different types of proxies?

## Behavioral Design Patterns

### Observer Pattern
- What is the Observer Pattern?
- How is it used in event-driven programming?

### Strategy Pattern
- What is the Strategy Pattern?
- When would you use the Strategy pattern?

### Command Pattern
- What is the Command Pattern?
- What are the benefits of using the Command pattern?

### Template Method Pattern
- What is the Template Method Pattern?
- How does it promote code reuse?

### Iterator Pattern
- What is the Iterator Pattern?
- How is it implemented in Java Collections?

### State Pattern
- What is the State Pattern?
- How does it differ from the Strategy pattern?

### Chain of Responsibility Pattern
- What is the Chain of Responsibility Pattern?
- When would you use this pattern?

## Code Examples

```java
// Example: Singleton Pattern
public class Singleton {
    // Private static instance
    private static volatile Singleton instance;
    
    // Private constructor
    private Singleton() {
        // Prevent reflection
        if (instance != null) {
            throw new IllegalStateException("Singleton already initialized");
        }
    }
    
    // Public static method to get instance
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
    
    // Prevent cloning
    @Override
    protected Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException("Cloning of Singleton not supported");
    }
    
    // Prevent deserialization creating a new instance
    protected Object readResolve() {
        return getInstance();
    }
}
```

## Resources

- [Design Patterns Resources](https://refactoring.guru/design-patterns/)