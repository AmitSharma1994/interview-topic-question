# String Handling Interview Questions and Answers

## String Immutability

### Apart from security aspects, what are the reasons behind making strings immutable in Java?

**Answer:**
1. **String Pool Optimization:** Immutability allows Java to maintain a string pool for reusing string literals, saving memory
2. **Thread Safety:** Immutable objects are inherently thread-safe, no synchronization needed
3. **Hashcode Caching:** String's hashcode can be cached since it won't change, improving performance in collections
4. **Security:** Used in sensitive operations like network connections, database URLs, file paths
5. **Consistent State:** Guarantees that string value won't change during operations
6. **Safe for Parameters:** Can be safely passed to methods without worry of modification

**Example:**
```java
String s1 = "Hello";  // Creates string in string pool
String s2 = "Hello";  // Reuses same string from pool
System.out.println(s1 == s2);  // true - same object reference

String s3 = s1.concat(" World");  // Creates new string, s1 remains unchanged
System.out.println(s1);  // "Hello" - original string is unmodified
System.out.println(s3);  // "Hello World" - new string
```

### How to create an immutable class?

**Answer:**
Steps to create an immutable class:
1. Declare the class as `final` (prevents inheritance)
2. Make all fields `private` and `final`
3. Don't provide setter methods
4. Initialize all fields via constructor
5. Make defensive copies of mutable objects in constructor and getter methods

**Example:**
```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Date;
import java.util.List;

public final class ImmutableStudent {
    private final int id;
    private final String name;
    private final Date birthDate;
    private final List<String> courses;
    
    public ImmutableStudent(int id, String name, Date birthDate, List<String> courses) {
        this.id = id;
        this.name = name;
        // Defensive copy for mutable objects
        this.birthDate = birthDate != null ? new Date(birthDate.getTime()) : null;
        // Create defensive copy of list
        this.courses = courses != null ? 
            Collections.unmodifiableList(new ArrayList<>(courses)) : 
            Collections.emptyList();
    }
    
    public int getId() {
        return id;
    }
    
    public String getName() {
        return name;
    }
    
    public Date getBirthDate() {
        // Return defensive copy
        return birthDate != null ? new Date(birthDate.getTime()) : null;
    }
    
    public List<String> getCourses() {
        // Already unmodifiable, no need for defensive copy
        return courses;
    }
}
```

## String Operations

### How to count the frequency of characters in a string?

**Answer:**
There are several approaches:

**1. Using HashMap:**
```java
public static Map<Character, Integer> countCharacters(String str) {
    Map<Character, Integer> charCountMap = new HashMap<>();
    
    // Convert string to char array
    char[] charArray = str.toCharArray();
    
    // Count each character
    for (char c : charArray) {
        if (charCountMap.containsKey(c)) {
            charCountMap.put(c, charCountMap.get(c) + 1);
        } else {
            charCountMap.put(c, 1);
        }
    }
    
    return charCountMap;
}

// Usage
String input = "abca abca";
Map<Character, Integer> frequency = countCharacters(input);
for (Map.Entry<Character, Integer> entry : frequency.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

**2. Using Java 8 Streams:**
```java
public static Map<Character, Long> countCharactersWithStream(String str) {
    return str.chars()
            .mapToObj(c -> (char) c)
            .collect(Collectors.groupingBy(c -> c, Collectors.counting()));
}
```

**3. Using Array (for ASCII characters):**
```java
public static int[] countCharactersArray(String str) {
    int[] count = new int[256]; // For ASCII characters
    
    for (int i = 0; i < str.length(); i++) {
        count[str.charAt(i)]++;
    }
    
    return count;
}
```

### How to reverse a string in Java?

**Answer:**
Several ways to reverse a string:

**1. Using StringBuilder/StringBuffer:**
```java
public static String reverseString(String str) {
    if (str == null || str.isEmpty()) {
        return str;
    }
    return new StringBuilder(str).reverse().toString();
}
```

**2. Using character array:**
```java
public static String reverseManually(String str) {
    if (str == null || str.isEmpty()) {
        return str;
    }
    
    char[] chars = str.toCharArray();
    int left = 0;
    int right = chars.length - 1;
    
    while (left < right) {
        // Swap characters
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;
        
        // Move towards middle
        left++;
        right--;
    }
    
    return new String(chars);
}
```

**3. Using recursion:**
```java
public static String reverseRecursively(String str) {
    if (str == null || str.length() <= 1) {
        return str;
    }
    return reverseRecursively(str.substring(1)) + str.charAt(0);
}
```

### How to remove duplicate characters from a string?

**Answer:**
Several approaches:

**1. Using LinkedHashSet (preserves order):**
```java
public static String removeDuplicates(String str) {
    if (str == null || str.isEmpty()) {
        return str;
    }
    
    Set<Character> charSet = new LinkedHashSet<>();
    for (char c : str.toCharArray()) {
        charSet.add(c);
    }
    
    StringBuilder sb = new StringBuilder();
    for (Character c : charSet) {
        sb.append(c);
    }
    
    return sb.toString();
}
```

**2. Using StringBuilder and indexOf:**
```java
public static String removeDuplicatesWithStringBuilder(String str) {
    if (str == null || str.isEmpty()) {
        return str;
    }
    
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < str.length(); i++) {
        char c = str.charAt(i);
        if (sb.indexOf(String.valueOf(c)) == -1) {
            sb.append(c);
        }
    }
    
    return sb.toString();
}
```

**3. Using Java 8 Streams:**
```java
public static String removeDuplicatesWithStream(String str) {
    if (str == null || str.isEmpty()) {
        return str;
    }
    
    return str.chars()
            .distinct()
            .mapToObj(c -> String.valueOf((char) c))
            .collect(Collectors.joining());
}
```

## String Classes

### How would you differentiate between String, StringBuffer, and StringBuilder?

**Answer:**
| Feature | String | StringBuffer | StringBuilder |
|---------|--------|--------------|---------------|
| Mutability | Immutable | Mutable | Mutable |
| Thread Safety | Yes | Yes (synchronized) | No |
| Performance | Slower for concatenation | Moderate | Fastest |
| Memory Usage | Creates new objects | Updates in-place | Updates in-place |
| Introduction | JDK 1.0 | JDK 1.0 | JDK 1.5 |

**When to use each:**
- **String:** When the value won't change
- **StringBuffer:** When value will change and thread safety is required
- **StringBuilder:** When value will change and thread safety is not required

**Example:**
```java
// String - creates multiple objects in memory
String s = "Hello";
s += " World";  // Creates a new string object

// StringBuffer - thread-safe, synchronized
StringBuffer sb = new StringBuffer("Hello");
sb.append(" World");  // Modifies the same object

// StringBuilder - not thread-safe, faster
StringBuilder sbd = new StringBuilder("Hello");
sbd.append(" World");  // Modifies the same object

// Performance comparison
long startTime = System.nanoTime();

// String concatenation
String result = "";
for (int i = 0; i < 10000; i++) {
    result += "a";
}

long stringTime = System.nanoTime() - startTime;
startTime = System.nanoTime();

// StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append("a");
}

long stringBuilderTime = System.nanoTime() - startTime;
System.out.println("String time: " + stringTime);
System.out.println("StringBuilder time: " + stringBuilderTime);
// StringBuilder is significantly faster
```

### How is the creation of a String using new() different from that of a literal?

**Answer:**
1. **Memory Location:**
   - String literals are stored in String Pool (part of heap memory)
   - Strings created with `new` are stored in the heap outside the String Pool

2. **Object Creation:**
   - String literals may reuse existing strings from the pool
   - `new String()` always creates a new object, even if identical content exists

3. **Performance:**
   - String literals are more memory-efficient due to reuse
   - `new String()` consumes more memory due to duplicate objects

4. **Equality Comparison:**
   - `==` may return true for identical string literals
   - `==` always returns false for different `new String()` objects with same content

**Example:**
```java
String s1 = "Hello";  // Creates in string pool
String s2 = "Hello";  // Reuses from string pool
String s3 = new String("Hello");  // Creates in heap
String s4 = new String("Hello");  // Creates another object in heap

System.out.println(s1 == s2);  // true - same object reference
System.out.println(s1 == s3);  // false - different objects
System.out.println(s3 == s4);  // false - different objects
System.out.println(s1.equals(s3));  // true - same content
```

### Why are character arrays preferred over strings for storing confidential information?

**Answer:**
1. **Mutability:** 
   - Character arrays can be explicitly cleared after use
   - Strings are immutable and remain in memory until garbage collected

2. **Memory Persistence:**
   - Strings may be present in memory dumps, string pools, or heap dumps
   - Character arrays can be overwritten with dummy data immediately after use

3. **String Internalization:**
   - String literals are stored in the string pool and may persist longer
   - Character arrays are not subject to string internalization

4. **Visibility in Debuggers:**
   - Strings are easily visible in debuggers and memory analysis tools
   - Character arrays can be more easily protected from inspection

**Example:**
```java
// Insecure - using String
String password = "secretPassword";
// Password remains in memory until garbage collection
// Cannot explicitly clear the password from memory
password = null;  // Only removes reference, original string may still be in memory

// Secure - using char array
char[] passwordArray = {'s', 'e', 'c', 'r', 'e', 't', 'P', 'a', 's', 's', 'w', 'o', 'r', 'd'};
// Use the password
// ...
// Explicitly clear the password from memory
Arrays.fill(passwordArray, '*');  // Overwrite with dummy character
```

## String Methods

### What is the difference between equals() method and equality operator (==) in Java?

**Answer:**
- **equals() method:**
  - Compares the **content** of objects
  - For strings, checks if the character sequences are identical
  - Defined in Object class and overridden in String class
  - Used for logical equality comparison

- **== operator:**
  - Compares **object references** (memory addresses)
  - Checks if both references point to the same object in memory
  - For primitives, compares the actual values
  - Used for reference equality comparison

**Example:**
```java
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");
String s4 = new String("Hello");

// Using == operator (reference comparison)
System.out.println(s1 == s2);  // true - both point to same string pool object
System.out.println(s1 == s3);  // false - different objects in memory
System.out.println(s3 == s4);  // false - different objects in memory

// Using equals() method (content comparison)
System.out.println(s1.equals(s2));  // true - same content
System.out.println(s1.equals(s3));  // true - same content
System.out.println(s3.equals(s4));  // true - same content
```

### What is the intern() method and how does it work?

**Answer:**
- **Purpose:** Forces a string to be placed in the string pool
- **Behavior:** 
  - If a string with identical content exists in the pool, returns that reference
  - If not, adds the string to the pool and returns the reference
- **Usage:** Helps save memory by ensuring string reuse
- **Performance:** Can improve memory usage but has overhead for the lookup

**Example:**
```java
String s1 = new String("Hello");  // Creates object in heap
String s2 = new String("Hello");  // Creates another object in heap
System.out.println(s1 == s2);  // false - different objects

String s3 = s1.intern();  // Gets reference from string pool (or adds if not present)
String s4 = s2.intern();  // Gets reference from string pool
System.out.println(s3 == s4);  // true - same reference from pool

// String literals automatically use intern
String s5 = "Hello";  // Uses string pool
System.out.println(s3 == s5);  // true - same reference from pool
```

**How intern() works internally:**
1. Checks if the string exists in the string pool
2. If found, returns the reference to the pooled string
3. If not found, adds the string to the pool and returns its reference

**Use cases:**
- Memory optimization when dealing with many duplicate strings
- Improving performance of string comparison with `==` instead of `equals()`