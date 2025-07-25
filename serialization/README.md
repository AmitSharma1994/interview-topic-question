# Serialization Interview Questions

## Marker Interfaces

- What do you understand about marker interfaces in Java?
- What are some examples of marker interfaces?

## Serial Version UID

- What is the serial version ID?
- Can we make a serializable class without serialVersionUID?
- What happens if we don't define serialVersionUID?

## Externalization

- What is the Externalization Interface in Java?
- How is Externalization different from Serialization?
- When would you use Externalization over Serialization?

## Code Examples

```java
// Example: Basic Serializable class with serialVersionUID
import java.io.Serializable;

public class Employee implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private String name;
    private int id;
    private transient String password; // transient fields are not serialized
    
    // Constructors, getters, setters
}
```

## Resources

- [Serialization Resources](https://drive.google.com/drive/u/1/folders/1eXPUK770Og4w0kqpvpLLAwfy8MmsowCj)