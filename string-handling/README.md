# String Handling Interview Questions

## String Immutability

- Apart from security aspects, what are the reasons behind making strings immutable in Java?
- How to create an immutable class?

## String Operations

- How to count the frequency of characters in a string? (e.g., "abca abca")
- How to reverse a string in Java?
- How to remove duplicate characters from a string? (e.g., "abcdbacad")

## String Classes

- How would you differentiate between String, StringBuffer, and StringBuilder?
- How is the creation of a String using new() different from that of a literal?
- Why are character arrays preferred over strings for storing confidential information?

## String Methods

- What is the difference between equals() method and equality operator (==) in Java?
- What is the intern() method and how does it work?

## Code Examples

```java
// Example: Removing duplicate characters from a string
public static String removeDuplicates(String input) {
    StringBuilder result = new StringBuilder();
    Set<Character> seen = new HashSet<>();
    
    for (char c : input.toCharArray()) {
        if (!seen.contains(c)) {
            seen.add(c);
            result.append(c);
        }
    }
    
    return result.toString();
}
```

## Resources

- [String Handling Resources](https://drive.google.com/drive/u/1/folders/1BxF88_V1lyzZIpC1-okqd4dDbKYRpNa8)