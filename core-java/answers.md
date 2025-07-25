# Core Java Interview Questions and Answers

## Memory Management

### What is the difference between stack and heap memory?

**Answer:**
- **Stack Memory:**
  - Stores local variables and method calls
  - Memory allocation/deallocation is automatic (LIFO order)
  - Faster access than heap
  - Limited size (typically smaller)
  - Thread-specific (each thread has its own stack)

- **Heap Memory:**
  - Stores objects and JRE classes
  - Memory allocation/deallocation managed by Garbage Collector
  - Slower access than stack
  - Larger size than stack
  - Shared across all threads

**Example:**
```java
public void exampleMethod() {
    int localVar = 5;                 // Stored in stack
    String localString = "Hello";     // Reference in stack, actual string in heap
    Person person = new Person();     // Reference in stack, object in heap
}
```

### How to identify memory leakage in Java applications?

**Answer:**
1. **Signs of memory leaks:**
   - OutOfMemoryError exceptions
   - Gradually increasing memory usage
   - Application slowdown over time

2. **Tools for memory leak detection:**
   - Java VisualVM
   - JProfiler
   - Eclipse Memory Analyzer (MAT)
   - YourKit Java Profiler
   - JConsole

3. **Common causes:**
   - Unclosed resources (streams, connections)
   - Improper handling of long-lived objects
   - Incorrect implementation of caching
   - Static collections growing unbounded

**Example of fixing a memory leak:**
```java
// Memory leak - connection never closed
public void badCode() {
    try {
        Connection conn = DriverManager.getConnection(url, user, password);
        // Use connection
        // Missing conn.close()
    } catch (SQLException e) {
        e.printStackTrace();
    }
}

// Fixed version with try-with-resources
public void goodCode() {
    try (Connection conn = DriverManager.getConnection(url, user, password)) {
        // Use connection
        // Automatically closed when block exits
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
```

### How would you optimize a Java application for better memory usage?

**Answer:**
1. **Use proper data structures:**
   - Choose appropriate collections (ArrayList vs LinkedList)
   - Use primitive types instead of wrapper classes when possible

2. **Resource management:**
   - Use try-with-resources for automatic resource closing
   - Close connections, streams, and files explicitly

3. **Object lifecycle management:**
   - Avoid creating unnecessary objects
   - Use object pooling for expensive objects
   - Set object references to null when no longer needed

4. **Collection optimization:**
   - Initialize collections with proper capacity
   - Use weak references for caches
   - Clear collections when no longer needed

5. **JVM tuning:**
   - Configure heap size appropriately (-Xms, -Xmx)
   - Choose appropriate garbage collector
   - Tune GC parameters

**Example:**
```java
// Memory inefficient
List<Integer> list = new ArrayList<>();  // Default capacity (10)
for (int i = 0; i < 10000; i++) {
    list.add(i);  // Multiple resizing operations
}

// Memory optimized
List<Integer> list = new ArrayList<>(10000);  // Pre-sized capacity
for (int i = 0; i < 10000; i++) {
    list.add(i);  // No resizing needed
}
```

## JVM Architecture

### Explain JVM architecture and class loaders

**Answer:**
**JVM Architecture components:**
1. **Class Loader Subsystem:**
   - Loads, links, and initializes classes
   - Three main class loaders: Bootstrap, Extension, Application

2. **Runtime Data Areas:**
   - Method Area (stores class structures)
   - Heap (stores objects)
   - Java Stacks (one per thread)
   - PC Registers (one per thread)
   - Native Method Stacks

3. **Execution Engine:**
   - Interpreter (executes bytecode)
   - JIT Compiler (compiles frequently executed code)
   - Garbage Collector

4. **Native Method Interface (JNI):**
   - Interface to native libraries

**Class Loader Hierarchy:**
- **Bootstrap Class Loader:** Loads core Java API classes (rt.jar)
- **Extension Class Loader:** Loads extension classes (jre/lib/ext)
- **Application Class Loader:** Loads application classes from classpath
- **Custom Class Loaders:** User-defined class loaders

**Class Loading Process:**
1. **Loading:** Find and load class bytecode
2. **Linking:**
   - Verification: Check bytecode correctness
   - Preparation: Allocate memory for static fields
   - Resolution: Replace symbolic references
3. **Initialization:** Execute static initializers

**Example:**
```java
// Custom ClassLoader example
public class CustomClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name); // Load class data from file
        return defineClass(name, classData, 0, classData.length);
    }

    private byte[] loadClassData(String className) {
        // Load class data from file or network
        // Implementation details omitted
        return null;
    }
}
```

### What are the differences between JVM, JRE, and JDK in Java?

**Answer:**
- **JVM (Java Virtual Machine):**
  - Executes Java bytecode
  - Platform-dependent (different for Windows, Linux, Mac)
  - Provides runtime environment for Java applications
  - Core components: Class loader, execution engine, garbage collector

- **JRE (Java Runtime Environment):**
  - Contains JVM
  - Includes core libraries and other components
  - Needed to run Java applications
  - Does not include development tools

- **JDK (Java Development Kit):**
  - Contains JRE
  - Includes development tools (compiler, debugger, documentation)
  - Needed to develop Java applications
  - Contains javac, java, javadoc, etc.

**Relationship:** JDK > JRE > JVM (JDK contains JRE, which contains JVM)

**Example:**
```
// To compile Java code (requires JDK)
javac HelloWorld.java

// To run Java code (requires JRE)
java HelloWorld
```

## OOP Concepts

### Define class, object, inheritance, polymorphism, abstraction, and encapsulation

**Answer:**
- **Class:** Blueprint or template for creating objects with shared attributes and behaviors
  ```java
  public class Car {
      private String model;
      private int year;
      
      public void drive() {
          System.out.println("Driving the car");
      }
  }
  ```

- **Object:** Instance of a class with state and behavior
  ```java
  Car myCar = new Car();
  myCar.drive();
  ```

- **Inheritance:** Mechanism where a class inherits properties and methods from another class
  ```java
  public class ElectricCar extends Car {
      private int batteryCapacity;
      
      public void charge() {
          System.out.println("Charging the car");
      }
  }
  ```

- **Polymorphism:** Ability of an object to take many forms
  - **Compile-time polymorphism:** Method overloading
  - **Runtime polymorphism:** Method overriding
  ```java
  // Method overloading (compile-time)
  public void accelerate() { }
  public void accelerate(int speed) { }
  
  // Method overriding (runtime)
  @Override
  public void drive() {
      System.out.println("Electric car driving silently");
  }
  ```

- **Abstraction:** Hiding implementation details and showing only functionality
  ```java
  public abstract class Vehicle {
      public abstract void start();
      
      public void stop() {
          System.out.println("Vehicle stopped");
      }
  }
  ```

- **Encapsulation:** Binding data and methods together, restricting direct access to data
  ```java
  public class BankAccount {
      private double balance;  // Private data
      
      public double getBalance() {
          return balance;
      }
      
      public void deposit(double amount) {
          if (amount > 0) {
              balance += amount;
          }
      }
  }
  ```

### Why are interfaces used?

**Answer:**
1. **Multiple inheritance:** Java doesn't support multiple class inheritance, but a class can implement multiple interfaces
2. **Contract definition:** Interfaces define what a class can do without specifying how
3. **Loose coupling:** Interfaces reduce dependencies between components
4. **Polymorphism:** Enable runtime polymorphism through interface references
5. **API design:** Define clear boundaries and contracts for APIs
6. **Testing:** Facilitate mocking and testing through interface-based design

**Example:**
```java
// Interface defines a contract
public interface PaymentProcessor {
    boolean processPayment(double amount);
    void refund(double amount);
}

// Multiple implementations
public class CreditCardProcessor implements PaymentProcessor {
    @Override
    public boolean processPayment(double amount) {
        // Credit card specific implementation
        return true;
    }
    
    @Override
    public void refund(double amount) {
        // Credit card refund implementation
    }
}

public class PayPalProcessor implements PaymentProcessor {
    @Override
    public boolean processPayment(double amount) {
        // PayPal specific implementation
        return true;
    }
    
    @Override
    public void refund(double amount) {
        // PayPal refund implementation
    }
}

// Usage with loose coupling
public class PaymentService {
    private PaymentProcessor processor;
    
    public PaymentService(PaymentProcessor processor) {
        this.processor = processor;
    }
    
    public void makePayment(double amount) {
        processor.processPayment(amount);
    }
}
```

### Uses of this, super, final, transient, volatile keywords

**Answer:**
- **this:**
  - References current object
  - Calls another constructor in same class
  - Distinguishes between instance variables and parameters
  ```java
  public class Person {
      private String name;
      
      public Person(String name) {
          this.name = name;  // Distinguish between parameter and instance variable
      }
      
      public Person() {
          this("Unknown");  // Call another constructor
      }
  }
  ```

- **super:**
  - References parent class object
  - Calls parent class constructor
  - Accesses parent class methods and fields
  ```java
  public class Child extends Parent {
      public Child() {
          super();  // Call parent constructor
      }
      
      @Override
      public void display() {
          super.display();  // Call parent method
          System.out.println("Child display");
      }
  }
  ```

- **final:**
  - **final variable:** Cannot be reassigned (constant)
  - **final method:** Cannot be overridden
  - **final class:** Cannot be extended
  ```java
  public final class ImmutableClass {  // Cannot be extended
      private final int value;  // Cannot be changed after initialization
      
      public ImmutableClass(int value) {
          this.value = value;
      }
      
      public final int getValue() {  // Cannot be overridden
          return value;
      }
  }
  ```

- **transient:**
  - Marks field to be excluded from serialization
  ```java
  public class User implements Serializable {
      private String username;
      private transient String password;  // Will not be serialized
  }
  ```

- **volatile:**
  - Ensures variable is read directly from main memory, not from thread cache
  - Provides visibility guarantee in multi-threaded environment
  ```java
  public class SharedResource {
      private volatile boolean flag = false;  // Changes to flag are visible to all threads
      
      public void setFlag() {
          flag = true;
      }
      
      public boolean isFlag() {
          return flag;
      }
  }
  ```

### Use of static method and variable

**Answer:**
- **Static Variables:**
  - Belong to class, not to instances
  - Shared among all instances
  - Accessed using class name
  - Initialized when class is loaded
  - Memory allocated once in method area

- **Static Methods:**
  - Belong to class, not to instances
  - Can access only static data
  - Cannot use this or super keywords
  - Cannot be overridden (can be hidden)
  - Used for utility functions

**Example:**
```java
public class Counter {
    private static int count = 0;  // Shared among all instances
    private int instanceCount = 0;  // Unique to each instance
    
    public Counter() {
        count++;
        instanceCount++;
    }
    
    public static int getCount() {  // Static method
        return count;
        // Cannot access instanceCount here without an instance
    }
    
    public int getInstanceCount() {  // Instance method
        return instanceCount;
    }
    
    public static void resetCount() {  // Utility method
        count = 0;
    }
}

// Usage
Counter c1 = new Counter();
Counter c2 = new Counter();
System.out.println(Counter.getCount());  // Outputs: 2
System.out.println(c1.getInstanceCount());  // Outputs: 1
System.out.println(c2.getInstanceCount());  // Outputs: 1
```

### Different types of access modifiers: public, protected, default, private

**Answer:**
- **public:**
  - Accessible from any class
  - Widest accessibility
  - Used for APIs and interfaces

- **protected:**
  - Accessible within same package
  - Accessible in subclasses (even in different packages)
  - Used for methods/fields that subclasses might need to override

- **default (no modifier):**
  - Accessible only within same package
  - Cannot be accessed from outside package
  - Used for package-level functionality

- **private:**
  - Accessible only within same class
  - Most restrictive access
  - Used for implementation details

**Accessibility Table:**
| Modifier  | Same Class | Same Package | Subclass | Different Package |
|-----------|------------|--------------|----------|-------------------|
| public    | Yes        | Yes          | Yes      | Yes               |
| protected | Yes        | Yes          | Yes      | No                |
| default   | Yes        | Yes          | No       | No                |
| private   | Yes        | No           | No       | No                |

**Example:**
```java
package com.example;

public class AccessExample {
    public int publicVar;       // Accessible everywhere
    protected int protectedVar; // Accessible in package and subclasses
    int defaultVar;             // Accessible only in package
    private int privateVar;     // Accessible only in this class
    
    public void publicMethod() { }
    protected void protectedMethod() { }
    void defaultMethod() { }
    private void privateMethod() { }
}

// Usage in subclass from different package
package com.other;

import com.example.AccessExample;

public class SubClass extends AccessExample {
    public void testAccess() {
        publicVar = 1;       // OK
        protectedVar = 2;    // OK (subclass access)
        // defaultVar = 3;   // ERROR - not accessible
        // privateVar = 4;   // ERROR - not accessible
        
        publicMethod();      // OK
        protectedMethod();   // OK (subclass access)
        // defaultMethod();  // ERROR - not accessible
        // privateMethod();  // ERROR - not accessible
    }
}
```

### Why is composition often preferred over inheritance?

**Answer:**
1. **Flexibility:** Composition allows changing behavior at runtime by swapping components
2. **Less coupling:** Components are loosely coupled, reducing dependencies
3. **No fragile base class problem:** Changes to base class don't unexpectedly affect subclasses
4. **Multiple sources:** Can reuse behavior from multiple sources (vs. single inheritance)
5. **Encapsulation:** Implementation details of composed objects remain hidden
6. **Testing:** Easier to test components in isolation
7. **Design principle:** "Favor composition over inheritance" is a key design principle

**Example:**
```java
// Inheritance approach (less flexible)
public class Bird {
    public void fly() {
        System.out.println("Flying...");
    }
}

public class Penguin extends Bird {
    @Override
    public void fly() {
        // Problem: Penguins can't fly, but inherit fly() method
        throw new UnsupportedOperationException("Penguins can't fly");
    }
}

// Composition approach (more flexible)
public interface Flyer {
    void fly();
}

public class FlyingBehavior implements Flyer {
    @Override
    public void fly() {
        System.out.println("Flying...");
    }
}

public class NoFlyingBehavior implements Flyer {
    @Override
    public void fly() {
        System.out.println("Cannot fly");
    }
}

public class Bird {
    private Flyer flyingBehavior;
    
    public Bird(Flyer flyingBehavior) {
        this.flyingBehavior = flyingBehavior;
    }
    
    public void fly() {
        flyingBehavior.fly();
    }
}

// Usage
Bird sparrow = new Bird(new FlyingBehavior());
Bird penguin = new Bird(new NoFlyingBehavior());
```

### Does Java work as "pass by value" or "pass by reference"?

**Answer:**
Java works strictly as **pass by value**, but this can be confusing because:

- For primitive types (int, boolean, etc.), the actual value is passed
- For objects, the value of the reference (memory address) is passed, not the object itself

This means:
1. Methods cannot change the original primitive variables
2. Methods cannot make a reference variable point to a new object
3. Methods CAN change the internal state of objects passed to them

**Example:**
```java
public class PassByValueExample {
    public static void main(String[] args) {
        // Primitive type example
        int x = 10;
        changeValue(x);
        System.out.println(x);  // Still 10, unchanged
        
        // Object reference example
        StringBuilder sb = new StringBuilder("Hello");
        changeReference(sb);
        System.out.println(sb);  // Still points to same object
        
        // Object state modification
        modifyObject(sb);
        System.out.println(sb);  // "Hello World" - internal state changed
    }
    
    public static void changeValue(int num) {
        num = 20;  // Only changes local copy, not original
    }
    
    public static void changeReference(StringBuilder s) {
        s = new StringBuilder("Goodbye");  // Only changes local reference
    }
    
    public static void modifyObject(StringBuilder s) {
        s.append(" World");  // Modifies the object's state
    }
}
```

Output:
```
10
Hello
Hello World
```