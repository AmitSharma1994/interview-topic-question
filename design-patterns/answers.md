# Design Patterns Interview Questions and Answers

## Creational Design Patterns

### Singleton Pattern

#### What is the Singleton Design Pattern?

**Answer:**
The Singleton pattern ensures that a class has only one instance and provides a global point of access to that instance. It restricts the instantiation of a class to a single object.

**Key characteristics:**
- Only one instance of the class can exist
- Global access point to the instance
- Lazy or eager initialization
- Thread-safe implementation considerations

**Use cases:**
- Database connections
- Logging services
- Configuration settings
- Cache implementations
- Thread pools

#### How to implement a thread-safe Singleton?

**Answer:**
There are several ways to implement a thread-safe Singleton:

**1. Double-Checked Locking:**
```java
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {}
    
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
}
```

**2. Bill Pugh Singleton (Initialization-on-demand holder):**
```java
public class Singleton {
    private Singleton() {}
    
    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

**3. Enum Singleton:**
```java
public enum Singleton {
    INSTANCE;
    
    public void doSomething() {
        // Business logic
    }
}
```

#### How can the Singleton pattern be broken?

**Answer:**
The Singleton pattern can be broken in several ways:

**1. Reflection API:**
```java
// Breaking singleton using reflection
Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
constructor.setAccessible(true);
Singleton instance1 = constructor.newInstance();
Singleton instance2 = constructor.newInstance();
// instance1 != instance2

// Prevention:
private Singleton() {
    if (instance != null) {
        throw new IllegalStateException("Singleton already initialized");
    }
}
```

**2. Serialization:**
```java
// Breaking singleton through serialization
Singleton instance1 = Singleton.getInstance();
// Serialize and deserialize
Singleton instance2 = deserialize(serialize(instance1));
// instance1 != instance2

// Prevention:
protected Object readResolve() {
    return getInstance();
}
```

**3. Cloning:**
```java
// Breaking singleton through cloning
Singleton instance1 = Singleton.getInstance();
Singleton instance2 = (Singleton) instance1.clone();
// instance1 != instance2

// Prevention:
@Override
protected Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException("Cloning not supported");
}
```

### Prototype Pattern

#### What is the Prototype Design Pattern?

**Answer:**
The Prototype pattern creates objects by cloning an existing instance rather than creating new instances from scratch. It's useful when object creation is expensive or complex.

**Key characteristics:**
- Creates objects by copying existing instances
- Avoids expensive object creation
- Supports deep and shallow copying
- Implements Cloneable interface

**Implementation:**
```java
public abstract class Shape implements Cloneable {
    private String id;
    protected String type;
    
    public abstract void draw();
    
    public String getType() { return type; }
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    
    @Override
    public Object clone() {
        Object clone = null;
        try {
            clone = super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return clone;
    }
}

public class Circle extends Shape {
    public Circle() {
        type = "Circle";
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a Circle");
    }
}

public class Rectangle extends Shape {
    public Rectangle() {
        type = "Rectangle";
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a Rectangle");
    }
}
```

#### When would you use the Prototype pattern?

**Answer:**
Use the Prototype pattern when:

1. **Object creation is expensive:** Database queries, network calls, complex calculations
2. **Similar objects with slight variations:** Game characters, UI components
3. **Runtime object creation:** When the exact type is determined at runtime
4. **Avoiding subclassing:** When you want to avoid creating many subclasses

### Builder Pattern

#### What is the Builder Design Pattern?

**Answer:**
The Builder pattern constructs complex objects step by step. It separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

**Key characteristics:**
- Step-by-step object construction
- Fluent interface (method chaining)
- Immutable objects
- Optional parameters handling

**Implementation:**
```java
public class Computer {
    // Required parameters
    private String CPU;
    private String RAM;
    
    // Optional parameters
    private boolean isGraphicsCardEnabled;
    private boolean isBluetoothEnabled;
    
    private Computer(ComputerBuilder builder) {
        this.CPU = builder.CPU;
        this.RAM = builder.RAM;
        this.isGraphicsCardEnabled = builder.isGraphicsCardEnabled;
        this.isBluetoothEnabled = builder.isBluetoothEnabled;
    }
    
    // Getters
    public String getCPU() { return CPU; }
    public String getRAM() { return RAM; }
    public boolean isGraphicsCardEnabled() { return isGraphicsCardEnabled; }
    public boolean isBluetoothEnabled() { return isBluetoothEnabled; }
    
    public static class ComputerBuilder {
        // Required parameters
        private String CPU;
        private String RAM;
        
        // Optional parameters
        private boolean isGraphicsCardEnabled;
        private boolean isBluetoothEnabled;
        
        public ComputerBuilder(String CPU, String RAM) {
            this.CPU = CPU;
            this.RAM = RAM;
        }
        
        public ComputerBuilder setGraphicsCardEnabled(boolean isGraphicsCardEnabled) {
            this.isGraphicsCardEnabled = isGraphicsCardEnabled;
            return this;
        }
        
        public ComputerBuilder setBluetoothEnabled(boolean isBluetoothEnabled) {
            this.isBluetoothEnabled = isBluetoothEnabled;
            return this;
        }
        
        public Computer build() {
            return new Computer(this);
        }
    }
}

// Usage
Computer computer = new Computer.ComputerBuilder("Intel i7", "16GB")
    .setGraphicsCardEnabled(true)
    .setBluetoothEnabled(true)
    .build();
```

#### How does it help with creating complex objects?

**Answer:**
The Builder pattern helps with complex object creation by:

1. **Handling optional parameters:** Avoids telescoping constructor problem
2. **Ensuring object consistency:** Validates parameters before object creation
3. **Immutability:** Creates immutable objects safely
4. **Readability:** Fluent interface makes code more readable
5. **Flexibility:** Different builders can create different representations

### Factory Pattern

#### What is the Factory Design Pattern?

**Answer:**
The Factory pattern creates objects without specifying their exact classes. It provides an interface for creating objects, but lets subclasses decide which class to instantiate.

**Simple Factory:**
```java
public class ShapeFactory {
    public Shape getShape(String shapeType) {
        if (shapeType == null) {
            return null;
        }
        if (shapeType.equalsIgnoreCase("CIRCLE")) {
            return new Circle();
        } else if (shapeType.equalsIgnoreCase("RECTANGLE")) {
            return new Rectangle();
        } else if (shapeType.equalsIgnoreCase("SQUARE")) {
            return new Square();
        }
        return null;
    }
}
```

**Factory Method:**
```java
public abstract class ShapeFactory {
    public abstract Shape createShape();
    
    public void drawShape() {
        Shape shape = createShape();
        shape.draw();
    }
}

public class CircleFactory extends ShapeFactory {
    @Override
    public Shape createShape() {
        return new Circle();
    }
}

public class RectangleFactory extends ShapeFactory {
    @Override
    public Shape createShape() {
        return new Rectangle();
    }
}
```

#### What is the difference between Simple Factory and Factory Method?

**Answer:**
**Simple Factory:**
- Not a true design pattern (more of a programming idiom)
- Single factory class creates all objects
- Violates Open/Closed Principle
- Easy to implement but hard to extend

**Factory Method:**
- True design pattern
- Each concrete factory creates specific objects
- Follows Open/Closed Principle
- More flexible and extensible
- Uses inheritance to delegate object creation

### Abstract Factory Pattern

#### What is the Abstract Factory Design Pattern?

**Answer:**
The Abstract Factory pattern provides an interface for creating families of related objects without specifying their concrete classes.

**Implementation:**
```java
// Abstract factory
public interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete factories
public class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

public class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Abstract products
public interface Button {
    void paint();
}

public interface Checkbox {
    void paint();
}

// Concrete products
public class WindowsButton implements Button {
    @Override
    public void paint() {
        System.out.println("Windows Button");
    }
}

public class MacButton implements Button {
    @Override
    public void paint() {
        System.out.println("Mac Button");
    }
}
```

#### How is it different from the Factory Method pattern?

**Answer:**
**Factory Method:**
- Creates one type of object
- Uses inheritance
- Single method creates objects
- Focuses on one product hierarchy

**Abstract Factory:**
- Creates families of related objects
- Uses composition
- Multiple methods create different objects
- Focuses on multiple product hierarchies
- Ensures products are compatible

## Structural Design Patterns

### Adapter Pattern

#### What is the Adapter Pattern?

**Answer:**
The Adapter pattern allows incompatible interfaces to work together. It acts as a bridge between two incompatible interfaces by wrapping an existing class with a new interface.

**Implementation:**
```java
// Target interface
public interface MediaPlayer {
    void play(String audioType, String fileName);
}

// Adaptee (existing class with incompatible interface)
public class AdvancedMediaPlayer {
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file: " + fileName);
    }
    
    public void playMp4(String fileName) {
        System.out.println("Playing mp4 file: " + fileName);
    }
}

// Adapter
public class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer;
    
    public MediaAdapter(String audioType) {
        if (audioType.equalsIgnoreCase("vlc") || audioType.equalsIgnoreCase("mp4")) {
            advancedPlayer = new AdvancedMediaPlayer();
        }
    }
    
    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedPlayer.playVlc(fileName);
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedPlayer.playMp4(fileName);
        }
    }
}

// Client
public class AudioPlayer implements MediaPlayer {
    private MediaAdapter mediaAdapter;
    
    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing mp3 file: " + fileName);
        } else if (audioType.equalsIgnoreCase("vlc") || audioType.equalsIgnoreCase("mp4")) {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);
        } else {
            System.out.println("Invalid media. " + audioType + " format not supported");
        }
    }
}
```

#### When would you use the Adapter pattern?

**Answer:**
Use the Adapter pattern when:

1. **Legacy system integration:** Integrating old systems with new interfaces
2. **Third-party library integration:** Using libraries with incompatible interfaces
3. **Interface mismatch:** When you can't modify existing classes
4. **Wrapper creation:** Creating a wrapper around existing functionality

### Decorator Pattern

#### What is the Decorator Pattern?

**Answer:**
The Decorator pattern adds new functionality to objects dynamically without altering their structure. It provides a flexible alternative to subclassing for extending functionality.

**Implementation:**
```java
// Component interface
public interface Coffee {
    double getCost();
    String getDescription();
}

// Concrete component
public class SimpleCoffee implements Coffee {
    @Override
    public double getCost() {
        return 2.0;
    }
    
    @Override
    public String getDescription() {
        return "Simple coffee";
    }
}

// Base decorator
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    @Override
    public double getCost() {
        return coffee.getCost();
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription();
    }
}

// Concrete decorators
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return super.getCost() + 0.5;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", milk";
    }
}

public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return super.getCost() + 0.2;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", sugar";
    }
}

// Usage
Coffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
System.out.println(coffee.getDescription() + " costs $" + coffee.getCost());
```

#### How is it different from inheritance?

**Answer:**
**Decorator Pattern:**
- Composition-based
- Runtime behavior modification
- Multiple decorators can be combined
- Flexible and dynamic
- Follows Open/Closed Principle

**Inheritance:**
- Class-based
- Compile-time behavior definition
- Single inheritance limitation
- Static and rigid
- Can lead to class explosion

### Facade Pattern

#### What is the Facade Pattern?

**Answer:**
The Facade pattern provides a simplified interface to a complex subsystem. It hides the complexity of the system and provides a unified interface to a set of interfaces in a subsystem.

**Implementation:**
```java
// Complex subsystem classes
class CPU {
    public void freeze() { System.out.println("CPU freeze"); }
    public void jump(long position) { System.out.println("CPU jump to " + position); }
    public void execute() { System.out.println("CPU execute"); }
}

class Memory {
    public void load(long position, byte[] data) {
        System.out.println("Memory load at " + position);
    }
}

class HardDrive {
    public byte[] read(long lba, int size) {
        System.out.println("HardDrive read");
        return new byte[size];
    }
}

// Facade
public class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;
    
    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    public void start() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}

// Client
public class Client {
    public static void main(String[] args) {
        ComputerFacade computer = new ComputerFacade();
        computer.start(); // Simple interface to complex operations
    }
}
```

#### How does it simplify complex systems?

**Answer:**
The Facade pattern simplifies complex systems by:

1. **Unified interface:** Single entry point for multiple subsystems
2. **Hiding complexity:** Clients don't need to know internal details
3. **Loose coupling:** Reduces dependencies between client and subsystem
4. **Easier maintenance:** Changes in subsystem don't affect clients
5. **Improved usability:** Provides convenient methods for common operations

### Proxy Pattern

#### What is the Proxy Pattern?

**Answer:**
The Proxy pattern provides a placeholder or surrogate for another object to control access to it. The proxy acts as an intermediary between the client and the real object.

**Implementation:**
```java
// Subject interface
public interface Image {
    void display();
}

// Real subject
public class RealImage implements Image {
    private String fileName;
    
    public RealImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading " + fileName);
    }
    
    @Override
    public void display() {
        System.out.println("Displaying " + fileName);
    }
}

// Proxy
public class ProxyImage implements Image {
    private RealImage realImage;
    private String fileName;
    
    public ProxyImage(String fileName) {
        this.fileName = fileName;
    }
    
    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}

// Usage
Image image = new ProxyImage("test.jpg");
image.display(); // Loading happens here
image.display(); // No loading, just display
```

#### What are the different types of proxies?

**Answer:**
**Types of proxies:**

1. **Virtual Proxy:** Lazy initialization of expensive objects
2. **Protection Proxy:** Controls access based on permissions
3. **Remote Proxy:** Represents objects in different address spaces
4. **Smart Proxy:** Adds additional behavior (reference counting, caching)
5. **Cache Proxy:** Caches results of expensive operations
6. **Synchronization Proxy:** Controls concurrent access to objects

## Behavioral Design Patterns

### Observer Pattern

#### What is the Observer Pattern?

**Answer:**
The Observer pattern defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

**Key characteristics:**
- Subject maintains a list of observers
- Observers are notified of state changes
- Loose coupling between subject and observers
- Supports broadcast communication

**Implementation:**
```java
// Observer interface
public interface Observer {
    void update(String message);
}

// Subject interface
public interface Subject {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}

// Concrete subject
public class NewsAgency implements Subject {
    private List<Observer> observers;
    private String news;
    
    public NewsAgency() {
        this.observers = new ArrayList<>();
    }
    
    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }
    
    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }
    
    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
}

// Concrete observer
public class NewsChannel implements Observer {
    private String name;
    private String news;
    
    public NewsChannel(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String news) {
        this.news = news;
        System.out.println(name + " received news: " + news);
    }
}
```

#### How is it used in event-driven programming?

**Answer:**
The Observer pattern is fundamental to event-driven programming:

1. **Event handling:** GUI components notify listeners of user actions
2. **Model-View architectures:** Views observe model changes
3. **Publish-Subscribe systems:** Publishers notify subscribers of events
4. **Real-time updates:** Live data feeds notify interested parties
5. **Reactive programming:** Streams of events trigger reactions

### Strategy Pattern

#### What is the Strategy Pattern?

**Answer:**
The Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable. It lets the algorithm vary independently from clients that use it.

**Implementation:**
```java
// Strategy interface
public interface PaymentStrategy {
    void pay(double amount);
}

// Concrete strategies
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using Credit Card: " + cardNumber);
    }
}

public class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using PayPal: " + email);
    }
}

// Context
public class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }
    
    public void checkout(double amount) {
        paymentStrategy.pay(amount);
    }
}
```

#### When would you use the Strategy pattern?

**Answer:**
Use the Strategy pattern when:

1. **Multiple algorithms:** Different ways to perform the same task
2. **Runtime selection:** Algorithm choice depends on runtime conditions
3. **Avoiding conditionals:** Eliminating large if-else or switch statements
4. **Algorithm families:** Related algorithms that can be grouped together
5. **Extensibility:** Easy addition of new algorithms without modifying existing code

### Command Pattern

#### What is the Command Pattern?

**Answer:**
The Command pattern encapsulates a request as an object, allowing you to parameterize clients with different requests, queue operations, and support undo operations.

**Implementation:**
```java
// Command interface
public interface Command {
    void execute();
    void undo();
}

// Receiver
public class Light {
    private boolean isOn = false;
    
    public void turnOn() {
        isOn = true;
        System.out.println("Light is ON");
    }
    
    public void turnOff() {
        isOn = false;
        System.out.println("Light is OFF");
    }
}

// Concrete commands
public class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOn();
    }
    
    @Override
    public void undo() {
        light.turnOff();
    }
}

// Invoker
public class RemoteControl {
    private Command command;
    private Command lastCommand;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        command.execute();
        lastCommand = command;
    }
    
    public void pressUndo() {
        if (lastCommand != null) {
            lastCommand.undo();
        }
    }
}
```

#### What are the benefits of using the Command pattern?

**Answer:**
Benefits of the Command pattern:

1. **Decoupling:** Separates invoker from receiver
2. **Undo/Redo:** Easy implementation of undo functionality
3. **Queuing:** Commands can be queued and executed later
4. **Logging:** Commands can be logged for audit trails
5. **Macro commands:** Combine multiple commands into one
6. **Remote execution:** Commands can be sent over network

### Template Method Pattern

#### What is the Template Method Pattern?

**Answer:**
The Template Method pattern defines the skeleton of an algorithm in a base class, letting subclasses override specific steps without changing the algorithm's structure.

**Implementation:**
```java
// Abstract class with template method
public abstract class DataProcessor {
    
    // Template method
    public final void process() {
        readData();
        processData();
        writeData();
    }
    
    // Abstract methods to be implemented by subclasses
    protected abstract void readData();
    protected abstract void processData();
    
    // Concrete method with default implementation
    protected void writeData() {
        System.out.println("Writing processed data to output");
    }
}

// Concrete implementations
public class CSVDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading data from CSV file");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing CSV data");
    }
}
```

#### How does it promote code reuse?

**Answer:**
The Template Method pattern promotes code reuse by:

1. **Common algorithm structure:** Shared workflow in base class
2. **Selective customization:** Override only necessary steps
3. **Default implementations:** Provide sensible defaults for common operations
4. **Inversion of control:** Framework calls application code
5. **Consistent behavior:** Ensures all implementations follow same pattern

### Iterator Pattern

#### What is the Iterator Pattern?

**Answer:**
The Iterator pattern provides a way to access elements of a collection sequentially without exposing its underlying representation.

**Implementation:**
```java
// Iterator interface
public interface Iterator<T> {
    boolean hasNext();
    T next();
}

// Aggregate interface
public interface Aggregate<T> {
    Iterator<T> createIterator();
}

// Concrete aggregate
public class BookCollection implements Aggregate<String> {
    private String[] books;
    private int count = 0;
    
    public BookCollection(int size) {
        books = new String[size];
    }
    
    public void addBook(String book) {
        if (count < books.length) {
            books[count++] = book;
        }
    }
    
    @Override
    public Iterator<String> createIterator() {
        return new BookIterator();
    }
    
    private class BookIterator implements Iterator<String> {
        private int index = 0;
        
        @Override
        public boolean hasNext() {
            return index < count;
        }
        
        @Override
        public String next() {
            if (hasNext()) {
                return books[index++];
            }
            return null;
        }
    }
}
```

#### How is it implemented in Java Collections?

**Answer:**
Java Collections Framework extensively uses the Iterator pattern:

1. **Iterable interface:** All collections implement Iterable
2. **Iterator interface:** Provides hasNext() and next() methods
3. **Enhanced for loop:** Syntactic sugar for iterator usage
4. **ListIterator:** Bidirectional iteration for lists
5. **Fail-fast behavior:** Detects concurrent modifications

### State Pattern

#### What is the State Pattern?

**Answer:**
The State pattern allows an object to alter its behavior when its internal state changes. The object appears to change its class.

**Implementation:**
```java
// State interface
public interface State {
    void insertCoin(VendingMachine machine);
    void selectProduct(VendingMachine machine);
    void dispenseProduct(VendingMachine machine);
}

// Context
public class VendingMachine {
    private State noCoinState;
    private State hasCoinState;
    private State soldState;
    private State currentState;
    
    public VendingMachine() {
        noCoinState = new NoCoinState();
        hasCoinState = new HasCoinState();
        soldState = new SoldState();
        currentState = noCoinState;
    }
    
    public void insertCoin() {
        currentState.insertCoin(this);
    }
    
    public void selectProduct() {
        currentState.selectProduct(this);
    }
    
    public void dispenseProduct() {
        currentState.dispenseProduct(this);
    }
    
    public void setState(State state) {
        this.currentState = state;
    }
    
    public State getNoCoinState() { return noCoinState; }
    public State getHasCoinState() { return hasCoinState; }
    public State getSoldState() { return soldState; }
}
```

#### How does it differ from the Strategy pattern?

**Answer:**
**State Pattern:**
- Object behavior changes based on internal state
- States often know about each other and trigger transitions
- Context delegates to current state
- State transitions are automatic

**Strategy Pattern:**
- Algorithm selection based on client choice
- Strategies are independent of each other
- Context uses selected strategy
- Strategy selection is explicit

### Chain of Responsibility Pattern

#### What is the Chain of Responsibility Pattern?

**Answer:**
The Chain of Responsibility pattern passes requests along a chain of handlers. Each handler decides either to process the request or pass it to the next handler in the chain.

**Implementation:**
```java
// Handler interface
public abstract class SupportHandler {
    protected SupportHandler nextHandler;
    
    public void setNextHandler(SupportHandler nextHandler) {
        this.nextHandler = nextHandler;
    }
    
    public abstract void handleRequest(SupportRequest request);
}

// Request class
public class SupportRequest {
    private String type;
    private String description;
    
    public SupportRequest(String type, String description) {
        this.type = type;
        this.description = description;
    }
    
    public String getType() { return type; }
    public String getDescription() { return description; }
}

// Concrete handlers
public class Level1Support extends SupportHandler {
    @Override
    public void handleRequest(SupportRequest request) {
        if (request.getType().equals("BASIC")) {
            System.out.println("Level 1 Support handled: " + request.getDescription());
        } else if (nextHandler != null) {
            nextHandler.handleRequest(request);
        }
    }
}
```

#### When would you use this pattern?

**Answer:**
Use the Chain of Responsibility pattern when:

1. **Multiple handlers:** Several objects can handle a request
2. **Dynamic handler selection:** Handler is determined at runtime
3. **Decoupling:** Sender shouldn't know which handler processes the request
4. **Flexible chains:** Chain structure can be modified dynamically
5. **Hierarchical processing:** Requests need to be processed in a specific order
6. **Approval workflows:** Multi-level approval processes
7. **Event handling:** GUI event propagation systems