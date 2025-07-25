# Exception Handling Interview Questions and Answers

## Exception Hierarchy

### What is the Exception hierarchy in Java?

**Answer:**
The Java exception hierarchy starts with the `Throwable` class at the top, which has two main subclasses:

1. **Error:**
   - Represents serious problems that a reasonable application should not try to catch
   - Examples: OutOfMemoryError, StackOverflowError, VirtualMachineError
   - Typically unrecoverable and should not be caught

2. **Exception:**
   - Represents conditions that a reasonable application might want to catch
   - Two main categories:
     
     a. **Checked Exceptions** (Compile-time exceptions):
     - Direct subclasses of Exception (except RuntimeException)
     - Must be either caught or declared in method signature
     - Examples: IOException, SQLException, ClassNotFoundException
     
     b. **Unchecked Exceptions** (Runtime exceptions):
     - Subclasses of RuntimeException
     - Do not need to be explicitly caught or declared
     - Examples: NullPointerException, ArrayIndexOutOfBoundsException, ArithmeticException

**Visual Hierarchy:**
```
Throwable
├── Error
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
└── Exception
    ├── IOException (checked)
    ├── SQLException (checked)
    ├── ClassNotFoundException (checked)
    ├── RuntimeException (unchecked)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── ArithmeticException
    │   └── ...
    └── ...
```

**Example:**
```java
// Checked exception - must be caught or declared
try {
    FileInputStream file = new FileInputStream("file.txt");
} catch (FileNotFoundException e) {
    System.err.println("File not found: " + e.getMessage());
}

// Unchecked exception - catching is optional
try {
    int result = 10 / 0;  // ArithmeticException
} catch (ArithmeticException e) {
    System.err.println("Cannot divide by zero");
}

// Error - typically not caught
try {
    // Cause OutOfMemoryError
    int[] hugeArray = new int[Integer.MAX_VALUE];
} catch (OutOfMemoryError e) {
    // Not recommended to catch Errors
    System.err.println("Out of memory");
}
```

### What is the difference between Error and Exception?

**Answer:**

| Feature | Error | Exception |
|---------|-------|-----------|
| Purpose | Indicates serious problems | Indicates exceptional conditions |
| Recovery | Usually unrecoverable | Often recoverable |
| Handling | Should not be caught | Should be caught and handled |
| Cause | Usually caused by the environment | Usually caused by the application |
| Examples | OutOfMemoryError, StackOverflowError | IOException, NullPointerException |
| Type | Unchecked | Both checked and unchecked |

**Key differences:**
1. **Severity:** Errors are more severe and typically unrecoverable
2. **Origin:** Errors usually occur at JVM level, exceptions at application level
3. **Handling approach:** Errors should generally not be caught, exceptions should be
4. **Prevention:** Exceptions can often be prevented with proper coding, errors often cannot

**Example:**
```java
// Exception - can be handled and recovered from
try {
    int[] arr = new int[5];
    arr[10] = 50;  // ArrayIndexOutOfBoundsException
} catch (ArrayIndexOutOfBoundsException e) {
    System.err.println("Array index out of bounds: " + e.getMessage());
    // Can recover and continue execution
}

// Error - typically cannot be handled or recovered from
// void causeStackOverflow() {
//     causeStackOverflow();  // StackOverflowError
// }
// causeStackOverflow();  // Program will terminate
```

### What is the difference between checked and unchecked exceptions?

**Answer:**

| Feature | Checked Exceptions | Unchecked Exceptions |
|---------|-------------------|---------------------|
| Inheritance | Subclasses of Exception (excluding RuntimeException) | Subclasses of RuntimeException |
| Compile-time checking | Yes - must be caught or declared | No - catching is optional |
| When to use | For recoverable conditions | For programming errors |
| Examples | IOException, SQLException | NullPointerException, IllegalArgumentException |
| Declaration | Must be declared in method signature with `throws` | No need to declare |
| Handling | Must be explicitly handled | Can be handled optionally |

**Key differences:**
1. **Compile-time enforcement:** Checked exceptions force handling at compile time
2. **Usage philosophy:** 
   - Checked: External factors that code should anticipate
   - Unchecked: Programming errors that should be fixed, not caught
3. **Recovery expectation:** Checked exceptions assume recovery is possible

**Example:**
```java
// Checked exception - must be caught or declared
public void readFile(String path) throws IOException {  // Declared in method signature
    FileInputStream file = new FileInputStream(path);
    // Read file...
    file.close();
}

// Alternative: catch the checked exception
public void readFileWithTryCatch(String path) {
    try {
        FileInputStream file = new FileInputStream(path);
        // Read file...
        file.close();
    } catch (IOException e) {
        System.err.println("Error reading file: " + e.getMessage());
    }
}

// Unchecked exception - no need to catch or declare
public int divide(int a, int b) {
    return a / b;  // May throw ArithmeticException if b is 0
}

// Optional: catch unchecked exception
public int divideWithTryCatch(int a, int b) {
    try {
        return a / b;
    } catch (ArithmeticException e) {
        System.err.println("Cannot divide by zero");
        return 0;  // Default value
    }
}
```

## Common Exceptions

### What is the difference between NoClassDefFoundError, ClassNotFoundException, and ClassCastException?

**Answer:**

**1. NoClassDefFoundError:**
- **Type:** Error (unchecked)
- **Cause:** JVM cannot find a class that was available at compile time but missing at runtime
- **Occurrence:** When a class was present during compilation but not found during execution
- **Common reasons:** 
  - Class was deleted after compilation
  - Classpath issues at runtime
  - Class initialization failed

**2. ClassNotFoundException:**
- **Type:** Exception (checked)
- **Cause:** Application tries to load a class through its string name using methods like `Class.forName()` or `ClassLoader.loadClass()` but no definition found
- **Occurrence:** During explicit class loading by name
- **Common reasons:**
  - Incorrect class name
  - Class not in classpath
  - Missing dependency

**3. ClassCastException:**
- **Type:** RuntimeException (unchecked)
- **Cause:** Attempting to cast an object to a subclass it doesn't belong to
- **Occurrence:** During explicit casting operations
- **Common reasons:**
  - Incorrect assumptions about object types
  - Heterogeneous collections without type checking

**Example:**
```java
// ClassNotFoundException (checked)
try {
    Class.forName("com.example.NonExistentClass");
} catch (ClassNotFoundException e) {
    System.err.println("Class not found: " + e.getMessage());
}

// NoClassDefFoundError (unchecked)
// Assume MyClass was compiled but the .class file was deleted
// MyClass obj = new MyClass();  // Throws NoClassDefFoundError

// ClassCastException (unchecked)
Object obj = new Integer(10);
try {
    String str = (String) obj;  // Throws ClassCastException
} catch (ClassCastException e) {
    System.err.println("Invalid cast: " + e.getMessage());
}
```

## Exception Handling Constructs

### What are try, catch, and finally blocks?

**Answer:**

**1. try block:**
- Contains code that might throw exceptions
- Must be followed by at least one catch block or a finally block
- Multiple statements can be included

**2. catch block:**
- Catches and handles exceptions thrown in the try block
- Specifies the type of exception it can handle
- Multiple catch blocks can be used to handle different exceptions
- More specific exceptions should be caught before more general ones

**3. finally block:**
- Contains code that always executes, whether an exception occurs or not
- Typically used for cleanup operations (closing resources)
- Executes even if the try or catch blocks contain return statements
- Does not execute only if `System.exit()` is called or the JVM crashes

**Example:**
```java
FileInputStream file = null;
try {
    file = new FileInputStream("example.txt");
    int data = file.read();
    while(data != -1) {
        System.out.print((char) data);
        data = file.read();
    }
} catch (FileNotFoundException e) {
    System.err.println("File not found: " + e.getMessage());
} catch (IOException e) {
    System.err.println("Error reading file: " + e.getMessage());
} finally {
    // This will always execute
    try {
        if (file != null) {
            file.close();
        }
    } catch (IOException e) {
        System.err.println("Error closing file: " + e.getMessage());
    }
    System.out.println("File operation completed");
}
```

### What is try-with-resources and how does it work?

**Answer:**
- **Definition:** A try statement with resources that are automatically closed after the statement completes
- **Introduced:** Java 7
- **Purpose:** Simplifies resource management and ensures resources are properly closed
- **Eligible resources:** Any object that implements `AutoCloseable` or `Closeable` interface
- **Advantage:** Eliminates the need for explicit finally blocks to close resources
- **Behavior:** Resources are closed in reverse order of their creation

**Example:**
```java
// Before try-with-resources (Java 6 and earlier)
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("file.txt"));
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    System.err.println("Error: " + e.getMessage());
} finally {
    try {
        if (br != null) {
            br.close();
        }
    } catch (IOException ex) {
        System.err.println("Error closing reader: " + ex.getMessage());
    }
}

// With try-with-resources (Java 7 and later)
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    System.err.println("Error: " + e.getMessage());
}
// BufferedReader is automatically closed

// Multiple resources
try (FileInputStream input = new FileInputStream("input.txt");
     FileOutputStream output = new FileOutputStream("output.txt")) {
    // Read from input and write to output
    byte[] buffer = new byte[1024];
    int bytesRead;
    while ((bytesRead = input.read(buffer)) != -1) {
        output.write(buffer, 0, bytesRead);
    }
} catch (IOException e) {
    System.err.println("Error: " + e.getMessage());
}
// Both resources are automatically closed in reverse order (output first, then input)
```

### What is the difference between throw and throws keywords?

**Answer:**

| Feature | throw | throws |
|---------|-------|--------|
| Purpose | Explicitly throws an exception | Declares exceptions a method might throw |
| Usage | Used inside method body | Used in method signature |
| Syntax | `throw exceptionObject;` | `returnType methodName() throws ExceptionType1, ExceptionType2 {...}` |
| Number of exceptions | Throws a single exception | Can declare multiple exception types |
| When used | To throw an exception based on condition | To inform caller about possible exceptions |
| Object creation | Usually with `new` keyword | No object creation |

**Example:**
```java
// throws - declares that method might throw exceptions
public void readFile(String fileName) throws FileNotFoundException, IOException {
    File file = new File(fileName);
    
    // throw - explicitly throws an exception
    if (!file.exists()) {
        throw new FileNotFoundException("File not found: " + fileName);
    }
    
    if (!file.canRead()) {
        throw new IOException("File cannot be read: " + fileName);
    }
    
    // Read file contents...
}

// Usage
public void processFile(String fileName) {
    try {
        readFile(fileName);
    } catch (FileNotFoundException e) {
        System.err.println("File not found: " + e.getMessage());
    } catch (IOException e) {
        System.err.println("IO error: " + e.getMessage());
    }
}
```

### What are checked exceptions and who calls them?

**Answer:**
- **Definition:** Exceptions that must be either caught or declared in the method signature
- **Who calls them:** 
  - They are "called" (thrown) by methods that encounter exceptional conditions
  - They are "declared" by methods that might throw them but don't handle them
  - They are "handled" by code that catches them

- **Common checked exceptions:**
  - IOException
  - SQLException
  - ClassNotFoundException
  - InterruptedException

- **When to use:** For recoverable conditions that the calling code should be aware of

**Example:**
```java
// Method that throws a checked exception
public void readData(String path) throws IOException {
    // This method calls a method that throws a checked exception
    BufferedReader reader = new BufferedReader(new FileReader(path));
    
    try {
        String line;
        while ((line = reader.readLine()) != null) {
            processLine(line);
        }
    } finally {
        reader.close();
    }
}

// Method that handles the checked exception
public void safeReadData(String path) {
    try {
        readData(path);
    } catch (IOException e) {
        System.err.println("Error reading data: " + e.getMessage());
        // Take recovery action
    }
}

// Method that re-throws the checked exception
public void delegateReadData(String path) throws IOException {
    readData(path);  // Doesn't handle the exception, declares it instead
}
```

### Can a single try block have multiple catch blocks? Explain.

**Answer:**
Yes, a single try block can have multiple catch blocks. This is called "multi-catch" and allows handling different types of exceptions differently.

**Key points:**
1. **Order matters:** More specific exceptions must be caught before more general ones
2. **Execution:** Only the first matching catch block executes
3. **Multi-catch syntax:** Since Java 7, you can catch multiple exception types in a single catch block using the pipe symbol (`|`)

**Example:**
```java
try {
    int[] array = new int[10];
    array[20] = 100 / 0;  // Could throw ArithmeticException or ArrayIndexOutOfBoundsException
    FileInputStream file = new FileInputStream("missing.txt");  // Could throw FileNotFoundException
} catch (ArrayIndexOutOfBoundsException e) {
    System.err.println("Array index problem: " + e.getMessage());
} catch (ArithmeticException e) {
    System.err.println("Arithmetic problem: " + e.getMessage());
} catch (FileNotFoundException e) {
    System.err.println("File problem: " + e.getMessage());
} catch (IOException e) {
    System.err.println("General I/O problem: " + e.getMessage());
} catch (Exception e) {
    System.err.println("General problem: " + e.getMessage());
}

// Multi-catch syntax (Java 7+)
try {
    // Some code that might throw exceptions
} catch (FileNotFoundException | SQLException e) {
    // Handle both exception types the same way
    System.err.println("Data access error: " + e.getMessage());
} catch (Exception e) {
    // Handle all other exceptions
    System.err.println("General error: " + e.getMessage());
}
```

## Exception Handling in Inheritance

### How does exception handling work in inheritance?

**Answer:**
When overriding methods that throw exceptions, there are specific rules to follow:

1. **Checked exceptions:**
   - Overriding method can throw the same exceptions as the parent method
   - Overriding method can throw narrower (subclasses) exceptions
   - Overriding method can throw fewer exceptions
   - Overriding method CANNOT throw broader or new checked exceptions

2. **Unchecked exceptions:**
   - Overriding method can throw any unchecked exceptions, regardless of parent method

3. **No exceptions in parent:**
   - If parent method doesn't throw exceptions, overriding method can only throw unchecked exceptions

**Example:**
```java
class Parent {
    // Parent method declares IOException
    public void method1() throws IOException {
        // Implementation
    }
    
    // Parent method declares multiple exceptions
    public void method2() throws IOException, SQLException {
        // Implementation
    }
    
    // Parent method doesn't declare any exceptions
    public void method3() {
        // Implementation
    }
}

class Child extends Parent {
    // VALID: Same exception as parent
    @Override
    public void method1() throws IOException {
        // Implementation
    }
    
    // VALID: More specific exception than parent
    // @Override
    // public void method1() throws FileNotFoundException {
    //     // FileNotFoundException is a subclass of IOException
    // }
    
    // VALID: Fewer exceptions than parent
    @Override
    public void method2() throws IOException {
        // Only throwing IOException, not SQLException
    }
    
    // VALID: No exceptions
    // @Override
    // public void method2() {
    //     // Not throwing any exceptions
    // }
    
    // INVALID: Cannot throw broader or new checked exceptions
    // @Override
    // public void method1() throws Exception {
    //     // Exception is broader than IOException
    // }
    
    // VALID: Can throw unchecked exceptions regardless of parent
    @Override
    public void method3() throws RuntimeException {
        // Throwing unchecked exception is allowed
    }
    
    // INVALID: Cannot throw checked exceptions if parent doesn't
    // @Override
    // public void method3() throws IOException {
    //     // Parent doesn't throw IOException
    // }
}
```

## Special Cases

### If we return from a try block, will the finally block still execute?

**Answer:**
Yes, the finally block will still execute even if there is a return statement in the try or catch blocks. The finally block executes before the method actually returns.

**Execution order:**
1. Code in try block executes
2. If return statement is reached in try block, the return value is calculated and stored
3. Finally block executes
4. The stored return value is returned

**Example:**
```java
public static int testReturn() {
    try {
        System.out.println("In try block");
        return 1;  // Return value is stored
    } catch (Exception e) {
        System.out.println("In catch block");
        return 2;
    } finally {
        System.out.println("In finally block");
        // Finally executes before the actual return happens
    }
}

// Output:
// In try block
// In finally block
// (returns 1)
```

**Special case - modifying return value:**
If the return value is a primitive, the finally block cannot modify it after it's been stored. However, if it's an object reference, the finally block can modify the object's state.

```java
public static int testPrimitive() {
    int x = 0;
    try {
        x = 1;
        return x;  // x=1 is stored as return value
    } finally {
        x = 2;  // Doesn't affect the stored return value
        System.out.println("x in finally: " + x);
    }
}
// Returns 1

public static List<Integer> testObject() {
    List<Integer> list = new ArrayList<>();
    try {
        list.add(1);
        return list;  // Reference to list is stored
    } finally {
        list.add(2);  // Modifies the object that will be returned
        System.out.println("List in finally: " + list);
    }
}
// Returns [1, 2]
```

### What is the difference between System.exit(0) and System.exit(1)?

**Answer:**
Both `System.exit(0)` and `System.exit(1)` terminate the JVM, but they indicate different exit statuses to the operating system:

- **System.exit(0):**
  - Indicates successful termination
  - Convention for normal/expected termination
  - Exit code 0 typically means "success" in operating systems

- **System.exit(1)** (or any non-zero value):
  - Indicates abnormal/error termination
  - Convention for failure or unexpected termination
  - Exit code 1 (or other non-zero values) typically means "error" in operating systems

**Key points:**
1. Both immediately terminate the JVM
2. Neither will execute finally blocks or shutdown hooks
3. The exit code is useful for scripts or parent processes that launch the Java application

**Example:**
```java
try {
    // Some critical operation
    if (operationSuccessful) {
        System.out.println("Operation completed successfully");
        System.exit(0);  // Successful termination
    } else {
        System.err.println("Operation failed");
        System.exit(1);  // Error termination
    }
} finally {
    System.out.println("This will not be executed if System.exit() is called");
}
```

**Shell script example using exit code:**
```bash
java MyApplication
if [ $? -eq 0 ]; then
    echo "Application completed successfully"
else
    echo "Application failed with error code $?"
fi
```