# Collection Framework Interview Questions

## Collection Hierarchy

- What is the hierarchy of the Collection Framework?
- What are the main interfaces in the Collection Framework?

## Lists

- What is the difference between ArrayList and LinkedList?
- How would you implement ArrayList using an array?
- How would you implement LinkedList using a doubly linked list?
- What is the insertion order in different List implementations?

## Maps

- What is the internal working of HashMap?
- What is the difference between HashMap and Hashtable?
- Compare HashMap, LinkedHashMap, and TreeMap
- What is ConcurrentHashMap and how is it different from Hashtable?
- What is a synchronized HashMap and how to create one?
- What is WeakHashMap?
- What data structures use HashMap internally?

## Sets

- What makes a HashSet different from a TreeSet and LinkedHashSet?
- How are these Set implementations implemented internally?

## Sorting and Comparison

- What is the difference between Comparable and Comparator?
- How would you sort objects based on ID?
- How would you sort objects based on name?

## Hashcode and Equals

- What is the hashCode and equals method implementation?
- What is the contract between hashCode and equals?

## Reflection

- What is the importance of reflection in Java?
- What methods are available in the Objects class?

## Fail-Fast vs Fail-Safe

- What are fail-fast and fail-safe iterators?
- Which collections are fail-fast and which are fail-safe?

## Code Examples

```java
// Example: Custom sorting with Comparator
List<Employee> employees = getEmployees();

// Sort by ID
Collections.sort(employees, Comparator.comparingInt(Employee::getId));

// Sort by name
Collections.sort(employees, Comparator.comparing(Employee::getName));
```

## Resources

- [Collection Framework Resources](https://drive.google.com/drive/u/1/folders/1eXPUK770Og4w0kqpvpLLAwfy8MmsowCj)