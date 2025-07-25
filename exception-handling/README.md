# Exception Handling Interview Questions

## Exception Hierarchy

- What is the Exception hierarchy in Java?
- What is the difference between Error and Exception?
- What is the difference between checked and unchecked exceptions?

## Common Exceptions

- What is the difference between NoClassDefFoundError, ClassNotFoundException, and ClassCastException?
- When do these exceptions occur?

## Exception Handling Constructs

- What are try, catch, and finally blocks?
- What is try-with-resources and how does it work?
- What is the difference between throw and throws keywords?
- What are checked exceptions and who calls them?
- Can a single try block have multiple catch blocks? Explain.

## Exception Handling in Inheritance

- How does exception handling work in inheritance?
- What are the rules for method overriding with exceptions?

## Special Cases

- If we return from a try block, will the finally block still execute?
- What is the difference between System.exit(0) and System.exit(1)?

## Code Examples

```java
// Example: Try-with-resources
public void readFile(String path) {
    try (FileInputStream fis = new FileInputStream(path);
         BufferedReader reader = new BufferedReader(new InputStreamReader(fis))) {
        String line;
        while ((line = reader.readLine()) != null) {
            System.out.println(line);
        }
    } catch (IOException e) {
        System.err.println("Error reading file: " + e.getMessage());
    }
    // No need to close resources - they are automatically closed
}
```

## Resources

- [Exception Handling Resources](https://drive.google.com/drive/u/1/folders/1ng_puMVQyiirPU-SEj1UIfguj7SWoCt_)