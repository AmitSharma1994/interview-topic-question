# Collection Framework Interview Questions and Answers

## Collection Hierarchy

### What is the hierarchy of the Collection Framework?

**Answer:**
The Java Collection Framework hierarchy consists of interfaces, implementations, and algorithms organized as follows:

**Top-level interfaces:**
1. **Collection:** Root interface for most collections
2. **Map:** Interface for key-value pair collections (not a true Collection)

**Main interfaces extending Collection:**
1. **List:** Ordered collection that allows duplicates
2. **Set:** Collection that doesn't allow duplicates
3. **Queue:** Collection designed for holding elements for processing

**Main interfaces extending Map:**
1. **SortedMap:** Map that maintains keys in sorted order

**Main implementations:**
- **List implementations:** ArrayList, LinkedList, Vector, Stack
- **Set implementations:** HashSet, LinkedHashSet, TreeSet
- **Queue implementations:** PriorityQueue, ArrayDeque
- **Map implementations:** HashMap, LinkedHashMap, TreeMap, Hashtable, Properties

**Visual Hierarchy:**
```
Iterable
└── Collection
    ├── List
    │   ├── ArrayList
    │   ├── LinkedList
    │   └── Vector
    │       └── Stack
    ├── Set
    │   ├── HashSet
    │   │   └── LinkedHashSet
    │   └── SortedSet
    │       └── TreeSet
    └── Queue
        ├── PriorityQueue
        └── Deque
            ├── ArrayDeque
            └── LinkedList

Map
├── HashMap
│   └── LinkedHashMap
├── Hashtable
│   └── Properties
└── SortedMap
    └── TreeMap
```

**Example:**
```java
// List example
List<String> arrayList = new ArrayList<>();
List<String> linkedList = new LinkedList<>();

// Set example
Set<String> hashSet = new HashSet<>();
Set<String> linkedHashSet = new LinkedHashSet<>();
Set<String> treeSet = new TreeSet<>();

// Queue example
Queue<String> priorityQueue = new PriorityQueue<>();
Deque<String> arrayDeque = new ArrayDeque<>();

// Map example
Map<String, Integer> hashMap = new HashMap<>();
Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
Map<String, Integer> treeMap = new TreeMap<>();
```

## Lists

### What is the difference between ArrayList and LinkedList?

**Answer:**

| Feature | ArrayList | LinkedList |
|---------|-----------|------------|
| Internal Structure | Dynamic array | Doubly linked list |
| Random Access | O(1) - constant time | O(n) - linear time |
| Insertion/Deletion | O(n) - requires shifting elements | O(1) - only updates references |
| Memory Overhead | Less - only stores elements | More - stores elements and references |
| Performance for add/remove at end | Better | Similar |
| Performance for add/remove at middle | Worse - requires shifting | Better - only updates references |
| Performance for iteration | Better - memory locality | Worse - references are scattered |
| Implementation of | List, RandomAccess | List, Deque, Queue |

**When to use:**
- **ArrayList:** When you need frequent random access, mostly read operations, and infrequent insertions/deletions
- **LinkedList:** When you need frequent insertions/deletions, especially in the middle, and don't need random access

**Example:**
```java
// ArrayList - better for random access and iteration
ArrayList<Integer> arrayList = new ArrayList<>();
arrayList.add(1);
arrayList.add(2);
arrayList.add(3);

// Fast random access
int element = arrayList.get(1);  // O(1) operation

// Slow insertion in the middle
arrayList.add(1, 10);  // O(n) operation - shifts elements

// LinkedList - better for frequent insertions/deletions
LinkedList<Integer> linkedList = new LinkedList<>();
linkedList.add(1);
linkedList.add(2);
linkedList.add(3);

// Slow random access
int element2 = linkedList.get(1);  // O(n) operation - traverses list

// Fast insertion in the middle
linkedList.add(1, 10);  // O(1) operation - updates references

// LinkedList can also be used as a Queue or Deque
linkedList.addFirst(0);  // Adds at the beginning
linkedList.addLast(4);   // Adds at the end
int first = linkedList.removeFirst();  // Removes from beginning
int last = linkedList.removeLast();    // Removes from end
```

### How would you implement ArrayList using an array?

**Answer:**
A simplified implementation of ArrayList using an array would include:

1. An internal array to store elements
2. A size variable to track the number of elements
3. Dynamic resizing when the array is full
4. Methods for adding, removing, and accessing elements

**Example Implementation:**
```java
public class SimpleArrayList<E> {
    private static final int DEFAULT_CAPACITY = 10;
    private Object[] elements;
    private int size;
    
    // Constructor
    public SimpleArrayList() {
        elements = new Object[DEFAULT_CAPACITY];
        size = 0;
    }
    
    // Add element to the end
    public boolean add(E element) {
        ensureCapacity(size + 1);
        elements[size++] = element;
        return true;
    }
    
    // Get element at index
    @SuppressWarnings("unchecked")
    public E get(int index) {
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);
        }
        return (E) elements[index];
    }
    
    // Remove element at index
    @SuppressWarnings("unchecked")
    public E remove(int index) {
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);
        }
        
        E oldValue = (E) elements[index];
        
        // Shift elements to the left
        int numMoved = size - index - 1;
        if (numMoved > 0) {
            System.arraycopy(elements, index + 1, elements, index, numMoved);
        }
        
        // Clear the last element and decrement size
        elements[--size] = null;
        
        return oldValue;
    }
    
    // Get current size
    public int size() {
        return size;
    }
    
    // Ensure capacity for adding elements
    private void ensureCapacity(int minCapacity) {
        if (minCapacity > elements.length) {
            // Grow the array (typically 1.5x or 2x)
            int newCapacity = elements.length * 2;
            if (newCapacity < minCapacity) {
                newCapacity = minCapacity;
            }
            
            // Create new array and copy elements
            Object[] newElements = new Object[newCapacity];
            System.arraycopy(elements, 0, newElements, 0, size);
            elements = newElements;
        }
    }
}
```

**Key aspects of the implementation:**
1. **Dynamic resizing:** When the array is full, create a larger array and copy elements
2. **Bounds checking:** Validate indices to prevent ArrayIndexOutOfBoundsException
3. **Shifting elements:** When removing an element, shift subsequent elements to fill the gap
4. **Null references:** Clear references to removed elements to help garbage collection

### How would you implement LinkedList using a doubly linked list?

**Answer:**
A simplified implementation of LinkedList using a doubly linked list would include:

1. Node class with references to previous and next nodes
2. Head and tail references to the first and last nodes
3. Size variable to track the number of elements
4. Methods for adding, removing, and accessing elements

**Example Implementation:**
```java
public class SimpleLinkedList<E> {
    // Node class for doubly linked list
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;
        
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    
    private Node<E> first;    // Head of the list
    private Node<E> last;     // Tail of the list
    private int size;
    
    // Constructor
    public SimpleLinkedList() {
        first = null;
        last = null;
        size = 0;
    }
    
    // Add element to the end
    public boolean add(E element) {
        linkLast(element);
        return true;
    }
    
    // Add element at the beginning
    public void addFirst(E element) {
        linkFirst(element);
    }
    
    // Add element at the end
    public void addLast(E element) {
        linkLast(element);
    }
    
    // Get element at index
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
    
    // Get first element
    public E getFirst() {
        if (first == null) {
            throw new NoSuchElementException();
        }
        return first.item;
    }
    
    // Get last element
    public E getLast() {
        if (last == null) {
            throw new NoSuchElementException();
        }
        return last.item;
    }
    
    // Remove element at index
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
    
    // Remove first element
    public E removeFirst() {
        if (first == null) {
            throw new NoSuchElementException();
        }
        return unlinkFirst();
    }
    
    // Remove last element
    public E removeLast() {
        if (last == null) {
            throw new NoSuchElementException();
        }
        return unlinkLast();
    }
    
    // Get current size
    public int size() {
        return size;
    }
    
    // Link element at the beginning
    private void linkFirst(E element) {
        Node<E> f = first;
        Node<E> newNode = new Node<>(null, element, f);
        first = newNode;
        if (f == null) {
            last = newNode;
        } else {
            f.prev = newNode;
        }
        size++;
    }
    
    // Link element at the end
    private void linkLast(E element) {
        Node<E> l = last;
        Node<E> newNode = new Node<>(l, element, null);
        last = newNode;
        if (l == null) {
            first = newNode;
        } else {
            l.next = newNode;
        }
        size++;
    }
    
    // Unlink first node
    private E unlinkFirst() {
        Node<E> f = first;
        E element = f.item;
        Node<E> next = f.next;
        
        f.item = null;
        f.next = null;
        first = next;
        
        if (next == null) {
            last = null;
        } else {
            next.prev = null;
        }
        
        size--;
        return element;
    }
    
    // Unlink last node
    private E unlinkLast() {
        Node<E> l = last;
        E element = l.item;
        Node<E> prev = l.prev;
        
        l.item = null;
        l.prev = null;
        last = prev;
        
        if (prev == null) {
            first = null;
        } else {
            prev.next = null;
        }
        
        size--;
        return element;
    }
    
    // Unlink specific node
    private E unlink(Node<E> x) {
        E element = x.item;
        Node<E> next = x.next;
        Node<E> prev = x.prev;
        
        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }
        
        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }
        
        x.item = null;
        size--;
        return element;
    }
    
    // Get node at index
    private Node<E> node(int index) {
        // Optimize by searching from beginning or end based on index
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++) {
                x = x.next;
            }
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--) {
                x = x.prev;
            }
            return x;
        }
    }
    
    // Check if index is valid
    private void checkElementIndex(int index) {
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);
        }
    }
}
```

**Key aspects of the implementation:**
1. **Doubly linked nodes:** Each node has references to both previous and next nodes
2. **First and last references:** Maintain references to both ends for O(1) operations at either end
3. **Optimization:** For get(index), start traversal from the closer end
4. **Null handling:** Clear references in removed nodes to help garbage collection
5. **Index validation:** Check indices to prevent out-of-bounds access

### What is the insertion order in different List implementations?

**Answer:**
Different List implementations maintain different insertion orders:

1. **ArrayList:**
   - Maintains insertion order
   - Elements are stored in the order they are added
   - Index-based access preserves this order

2. **LinkedList:**
   - Maintains insertion order
   - Elements are linked in the order they are added
   - Iteration follows this linked order

3. **Vector:**
   - Maintains insertion order (similar to ArrayList)
   - Legacy class, thread-safe but less efficient

4. **Stack:**
   - Extends Vector, but follows LIFO (Last-In-First-Out) order for access
   - Insertion order is maintained, but typical usage (push/pop) follows stack order

**Example:**
```java
// ArrayList - maintains insertion order
List<String> arrayList = new ArrayList<>();
arrayList.add("A");
arrayList.add("B");
arrayList.add("C");
System.out.println(arrayList);  // [A, B, C]

// LinkedList - maintains insertion order
List<String> linkedList = new LinkedList<>();
linkedList.add("A");
linkedList.add("B");
linkedList.add("C");
System.out.println(linkedList);  // [A, B, C]

// Vector - maintains insertion order
List<String> vector = new Vector<>();
vector.add("A");
vector.add("B");
vector.add("C");
System.out.println(vector);  // [A, B, C]

// Stack - maintains insertion order but typically used as LIFO
Stack<String> stack = new Stack<>();
stack.push("A");
stack.push("B");
stack.push("C");
System.out.println(stack);  // [A, B, C]
System.out.println(stack.pop());  // C (LIFO access)
System.out.println(stack.pop());  // B
System.out.println(stack.pop());  // A
```

## Maps

### What is the internal working of HashMap?

**Answer:**
HashMap internally works based on the principles of hashing. Here's how it works:

1. **Data Structure:**
   - Array of buckets (called table)
   - Each bucket can store multiple entries as a linked list or a balanced tree

2. **Key Components:**
   - **Entry:** Key-value pair stored in the map
   - **Bucket:** Slot in the array that can hold one or more entries
   - **Hash function:** Converts keys to array indices
   - **Load factor:** Ratio of entries to buckets that triggers resizing (default 0.75)
   - **Initial capacity:** Initial size of the bucket array (default 16)

3. **Operations:**
   - **put(K key, V value):**
     1. Calculate hash code of the key
     2. Map hash code to an index in the array
     3. If bucket is empty, create new entry
     4. If bucket has entries, either update existing entry with same key or add new entry
     5. If bucket size exceeds threshold, convert linked list to balanced tree (Java 8+)
     6. If load factor threshold is exceeded, resize and rehash

   - **get(Object key):**
     1. Calculate hash code of the key
     2. Find the bucket using the hash code
     3. Search the bucket for an entry with matching key
     4. Return the value or null if not found

   - **remove(Object key):**
     1. Find the bucket using the hash code
     2. Remove the entry with matching key
     3. Return the value or null if not found

4. **Collision Handling:**
   - **Java 7 and before:** Simple linked list in each bucket
   - **Java 8 and later:** Linked list for small buckets, balanced tree (Red-Black Tree) when bucket size exceeds threshold (typically 8)

5. **Resizing:**
   - When entries > capacity * load factor
   - Creates new array with double the capacity
   - Rehashes all existing entries to new buckets

**Visual Representation:**
```
HashMap Internal Structure:

Array (table):
[0] → null
[1] → Entry(K1,V1) → Entry(K4,V4)  // Collision handled by linked list
[2] → null
[3] → Entry(K2,V2)
[4] → null
[5] → Entry(K3,V3) → Entry(K5,V5) → Entry(K6,V6) → ...  // Converted to tree if > 8 entries
...
```

**Example:**
```java
HashMap<String, Integer> map = new HashMap<>();

// Put operation
map.put("one", 1);  // hash("one") % capacity = bucket index (e.g., 3)
map.put("two", 2);  // hash("two") % capacity = different bucket (e.g., 5)
map.put("three", 3);

// Collision example (hypothetical keys with same bucket)
map.put("oneA", 11);  // Same bucket as "one", creates linked list

// Get operation
int value = map.get("two");  // hash("two") to find bucket, then find entry

// Remove operation
map.remove("one");  // Removes entry, updates linked list if needed
```

### What is the difference between HashMap and Hashtable?

**Answer:**

| Feature | HashMap | Hashtable |
|---------|---------|-----------|
| Synchronization | Not synchronized (not thread-safe) | Synchronized (thread-safe) |
| Null values | Allows one null key and multiple null values | Does not allow null keys or values |
| Performance | Generally better in single-threaded environments | Slower due to synchronization overhead |
| Iterator type | Fail-fast iterator | Enumerator and fail-fast iterator |
| Introduction | Java 1.2 | Java 1.0 (legacy class) |
| Inheritance | Extends AbstractMap | Extends Dictionary |
| Default capacity | 16 | 11 |
| Growth | Doubles in size (power of 2) | Increases by 2n+1 |

**When to use:**
- **HashMap:** Single-threaded applications or when external synchronization is used
- **Hashtable:** Legacy code or when built-in synchronization is required (though ConcurrentHashMap is usually better)

**Example:**
```java
// HashMap - not synchronized, allows null
HashMap<String, Integer> hashMap = new HashMap<>();
hashMap.put("one", 1);
hashMap.put("two", 2);
hashMap.put(null, 0);    // Allowed
hashMap.put("none", null);  // Allowed

// Hashtable - synchronized, doesn't allow null
Hashtable<String, Integer> hashtable = new Hashtable<>();
hashtable.put("one", 1);
hashtable.put("two", 2);
// hashtable.put(null, 0);    // NullPointerException
// hashtable.put("none", null);  // NullPointerException

// Thread safety
// HashMap needs external synchronization
Map<String, Integer> synchronizedMap = Collections.synchronizedMap(hashMap);

// Better alternative to Hashtable
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
```

### Compare HashMap, LinkedHashMap, and TreeMap

**Answer:**

| Feature | HashMap | LinkedHashMap | TreeMap |
|---------|---------|--------------|---------|
| Internal Structure | Hash table | Hash table + Doubly linked list | Red-Black Tree |
| Order | No guaranteed order | Insertion order or access order | Sorted by keys |
| Performance (get/put) | O(1) average | O(1) average | O(log n) |
| Performance (iteration) | O(n) | O(n) but more efficient | O(n) |
| Null keys | One null key allowed | One null key allowed | No null keys allowed |
| Null values | Multiple null values allowed | Multiple null values allowed | Multiple null values allowed |
| Memory usage | Lower | Higher (extra linked list) | Higher (tree nodes) |
| Use case | Fast access, no order needed | When order matters | When sorted order needed |

**When to use:**
- **HashMap:** General purpose, when order doesn't matter
- **LinkedHashMap:** When insertion order or access order is important
- **TreeMap:** When keys need to be sorted or range operations are needed

**Example:**
```java
// HashMap - no guaranteed order
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("banana", 2);
hashMap.put("apple", 1);
hashMap.put("cherry", 3);
System.out.println(hashMap);  // No guaranteed order, e.g., {apple=1, cherry=3, banana=2}

// LinkedHashMap - maintains insertion order
Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
linkedHashMap.put("banana", 2);
linkedHashMap.put("apple", 1);
linkedHashMap.put("cherry", 3);
System.out.println(linkedHashMap);  // {banana=2, apple=1, cherry=3}

// LinkedHashMap with access order
Map<String, Integer> accessOrderMap = new LinkedHashMap<>(16, 0.75f, true);
accessOrderMap.put("banana", 2);
accessOrderMap.put("apple", 1);
accessOrderMap.put("cherry", 3);
accessOrderMap.get("apple");  // Accessing apple
System.out.println(accessOrderMap);  // {banana=2, cherry=3, apple=1} - apple moved to end

// TreeMap - sorted by natural order of keys
Map<String, Integer> treeMap = new TreeMap<>();
treeMap.put("banana", 2);
treeMap.put("apple", 1);
treeMap.put("cherry", 3);
System.out.println(treeMap);  // {apple=1, banana=2, cherry=3} - alphabetical order

// TreeMap with custom comparator
Map<String, Integer> customTreeMap = new TreeMap<>(Comparator.reverseOrder());
customTreeMap.put("banana", 2);
customTreeMap.put("apple", 1);
customTreeMap.put("cherry", 3);
System.out.println(customTreeMap);  // {cherry=3, banana=2, apple=1} - reverse order
```

### What is ConcurrentHashMap and how is it different from Hashtable?

**Answer:**
ConcurrentHashMap is a thread-safe implementation of Map that offers better concurrency than Hashtable by using a more fine-grained locking mechanism.

| Feature | ConcurrentHashMap | Hashtable |
|---------|-------------------|-----------|
| Locking mechanism | Segment/bucket level locking | Object level locking |
| Concurrency | High - multiple threads can access different segments | Low - one thread at a time |
| Null values | Does not allow null keys or values | Does not allow null keys or values |
| Performance | Better in multi-threaded environments | Worse due to coarse-grained locking |
| Iterator behavior | Weakly consistent (doesn't throw ConcurrentModificationException) | Fail-fast |
| Introduction | Java 1.5 | Java 1.0 |

**Key differences:**
1. **Locking granularity:**
   - Hashtable locks the entire map for each operation
   - ConcurrentHashMap locks only a portion of the map (segment or bucket)

2. **Concurrency level:**
   - In Java 7: Uses segments (default 16) for locking
   - In Java 8+: Uses bucket-level locking for even better concurrency

3. **Iterator behavior:**
   - Hashtable iterator throws ConcurrentModificationException if map is modified during iteration
   - ConcurrentHashMap iterator is weakly consistent and doesn't throw exceptions

**Example:**
```java
// Hashtable - entire map locked for each operation
Hashtable<String, Integer> hashtable = new Hashtable<>();
hashtable.put("one", 1);
hashtable.put("two", 2);

// ConcurrentHashMap - finer-grained locking
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("one", 1);
concurrentMap.put("two", 2);

// Thread safety with better performance
// Multiple threads can read/write simultaneously if different keys

// Atomic operations
concurrentMap.putIfAbsent("three", 3);  // Atomic check-then-act
Integer oldValue = concurrentMap.replace("two", 2, 22);  // Atomic replace

// Bulk operations
concurrentMap.forEach(2, (key, value) -> 
    System.out.println(key + "=" + value));  // Parallel processing with parallelism threshold
```

### What is a synchronized HashMap and how to create one?

**Answer:**
A synchronized HashMap is a thread-safe wrapper around a regular HashMap, created using the `Collections.synchronizedMap()` method. It provides thread safety by synchronizing all map operations on a mutex object.

**How to create:**
```java
Map<String, Integer> hashMap = new HashMap<>();
Map<String, Integer> synchronizedMap = Collections.synchronizedMap(hashMap);
```

**Key characteristics:**
1. **Thread safety:** All operations are synchronized on a mutex object
2. **Performance:** Lower than ConcurrentHashMap due to coarse-grained locking
3. **Iteration:** Must be manually synchronized to avoid ConcurrentModificationException
4. **Null handling:** Same as HashMap (allows null keys and values)

**Comparison with alternatives:**

| Feature | synchronized HashMap | Hashtable | ConcurrentHashMap |
|---------|---------------------|-----------|-------------------|
| Locking mechanism | Object level locking | Object level locking | Segment/bucket level locking |
| Concurrency | Low - one thread at a time | Low - one thread at a time | High - multiple threads |
| Null values | Allows null keys and values | No null keys or values | No null keys or values |
| Performance | Low | Low | High |
| Creation | Wrapper around HashMap | Standalone class | Standalone class |

**Example with proper iteration:**
```java
Map<String, Integer> map = new HashMap<>();
map.put("one", 1);
map.put("two", 2);
map.put("three", 3);

// Create synchronized map
Map<String, Integer> synchronizedMap = Collections.synchronizedMap(map);

// Thread-safe operations
synchronizedMap.put("four", 4);
Integer value = synchronizedMap.get("one");
synchronizedMap.remove("two");

// Iteration MUST be synchronized manually
synchronized (synchronizedMap) {
    for (Map.Entry<String, Integer> entry : synchronizedMap.entrySet()) {
        System.out.println(entry.getKey() + ": " + entry.getValue());
    }
}
```

**When to use:**
- When you need thread safety for an existing HashMap
- When you can't use ConcurrentHashMap (e.g., in older Java versions)
- When you need to allow null keys or values in a thread-safe map

## Sets

### What makes a HashSet different from a TreeSet and LinkedHashSet?

**Answer:**

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|--------------|---------|
| Internal Structure | Backed by HashMap | Backed by LinkedHashMap | Backed by TreeMap |
| Order | No guaranteed order | Insertion order | Sorted order |
| Performance (add/remove/contains) | O(1) average | O(1) average | O(log n) |
| Performance (iteration) | O(n) | O(n) but more efficient | O(n) |
| Null elements | One null allowed | One null allowed | No null allowed |
| Memory usage | Lower | Higher (extra linked list) | Higher (tree nodes) |
| Use case | Fast operations, no order needed | When insertion order matters | When sorted order needed |

**When to use:**
- **HashSet:** General purpose, when order doesn't matter and fast operations are needed
- **LinkedHashSet:** When insertion order is important
- **TreeSet:** When elements need to be sorted or range operations are needed

**Example:**
```java
// HashSet - no guaranteed order
Set<String> hashSet = new HashSet<>();
hashSet.add("banana");
hashSet.add("apple");
hashSet.add("cherry");
System.out.println(hashSet);  // No guaranteed order, e.g., [apple, cherry, banana]

// LinkedHashSet - maintains insertion order
Set<String> linkedHashSet = new LinkedHashSet<>();
linkedHashSet.add("banana");
linkedHashSet.add("apple");
linkedHashSet.add("cherry");
System.out.println(linkedHashSet);  // [banana, apple, cherry]

// TreeSet - sorted by natural order
Set<String> treeSet = new TreeSet<>();
treeSet.add("banana");
treeSet.add("apple");
treeSet.add("cherry");
System.out.println(treeSet);  // [apple, banana, cherry] - alphabetical order

// TreeSet with custom comparator
Set<String> customTreeSet = new TreeSet<>(Comparator.reverseOrder());
customTreeSet.add("banana");
customTreeSet.add("apple");
customTreeSet.add("cherry");
System.out.println(customTreeSet);  // [cherry, banana, apple] - reverse order

// Range operations with TreeSet
TreeSet<Integer> numberSet = new TreeSet<>();
numberSet.add(10);
numberSet.add(20);
numberSet.add(30);
numberSet.add(40);
System.out.println(numberSet.headSet(25));  // [10, 20]
System.out.println(numberSet.tailSet(25));  // [30, 40]
System.out.println(numberSet.subSet(15, 35));  // [20, 30]
```

### How are these Set implementations implemented internally?

**Answer:**

**1. HashSet:**
- Internally backed by a HashMap
- Elements are stored as keys in the HashMap
- Values are a dummy object (constant PRESENT)
- Uses hashCode() and equals() for element comparison
- No guaranteed iteration order

**Implementation details:**
```java
// Simplified internal structure
public class HashSet<E> {
    private static final Object PRESENT = new Object();
    private HashMap<E, Object> map;
    
    public HashSet() {
        map = new HashMap<>();
    }
    
    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
    
    public boolean contains(Object o) {
        return map.containsKey(o);
    }
    
    public boolean remove(Object o) {
        return map.remove(o) == PRESENT;
    }
    
    public int size() {
        return map.size();
    }
}
```

**2. LinkedHashSet:**
- Extends HashSet
- Internally backed by a LinkedHashMap
- Maintains a doubly-linked list of entries
- Preserves insertion order during iteration
- Slightly slower than HashSet for add/remove but faster for iteration

**Implementation details:**
```java
// Simplified internal structure
public class LinkedHashSet<E> extends HashSet<E> {
    private LinkedHashMap<E, Object> map;
    
    public LinkedHashSet() {
        map = new LinkedHashMap<>();
    }
    
    // Other methods inherited from HashSet but using LinkedHashMap internally
}
```

**3. TreeSet:**
- Internally backed by a TreeMap
- Elements are stored as keys in the TreeMap
- Uses a Red-Black Tree for storage
- Elements are sorted according to natural order or a comparator
- Provides guaranteed log(n) time for add, remove, contains operations

**Implementation details:**
```java
// Simplified internal structure
public class TreeSet<E> {
    private TreeMap<E, Object> map;
    private static final Object PRESENT = new Object();
    
    public TreeSet() {
        map = new TreeMap<>();
    }
    
    public TreeSet(Comparator<? super E> comparator) {
        map = new TreeMap<>(comparator);
    }
    
    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
    
    public boolean contains(Object o) {
        return map.containsKey(o);
    }
    
    public boolean remove(Object o) {
        return map.remove(o) == PRESENT;
    }
    
    public E first() {
        return map.firstKey();
    }
    
    public E last() {
        return map.lastKey();
    }
}
```

**Example of internal behavior:**
```java
// HashSet internal behavior
HashSet<String> hashSet = new HashSet<>();
hashSet.add("apple");  // Internally: map.put("apple", PRESENT)
hashSet.contains("apple");  // Internally: map.containsKey("apple")

// LinkedHashSet internal behavior
LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>();
linkedHashSet.add("apple");  // Adds to HashMap and linked list
linkedHashSet.add("banana");  // Maintains order in linked list

// TreeSet internal behavior
TreeSet<String> treeSet = new TreeSet<>();
treeSet.add("banana");  // Internally sorted using compareTo
treeSet.add("apple");  // Internally rebalances tree
```

## Sorting and Comparison

### What is the difference between Comparable and Comparator?

**Answer:**

| Feature | Comparable | Comparator |
|---------|------------|------------|
| Interface | java.lang.Comparable | java.util.Comparator |
| Method | compareTo(T o) | compare(T o1, T o2) |
| Purpose | Natural ordering of a class | Custom ordering independent of class |
| Implementation | Class itself implements | Separate class implements |
| Usage | Collections.sort(list) | Collections.sort(list, comparator) |
| Multiple orderings | No - only one natural order | Yes - multiple comparators possible |
| Modifying classes | Requires modifying the class | Works without modifying the class |
| Java 8 syntax | N/A | Lambda expressions supported |

**When to use:**
- **Comparable:** When a class has a natural ordering (e.g., alphabetical for strings)
- **Comparator:** When you need multiple ordering strategies or can't modify the class

**Example:**
```java
// Comparable example
public class Person implements Comparable<Person> {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    @Override
    public int compareTo(Person other) {
        // Natural ordering by name
        return this.name.compareTo(other.name);
    }
    
    @Override
    public String toString() {
        return name + " (" + age + ")";
    }
}

// Usage
List<Person> people = new ArrayList<>();
people.add(new Person("Charlie", 40));
people.add(new Person("Alice", 25));
people.add(new Person("Bob", 30));

// Using natural ordering (Comparable)
Collections.sort(people);
System.out.println(people);  // [Alice (25), Bob (30), Charlie (40)]

// Comparator example - sort by age
Comparator<Person> ageComparator = new Comparator<Person>() {
    @Override
    public int compare(Person p1, Person p2) {
        return Integer.compare(p1.getAge(), p2.getAge());
    }
};

// Using custom ordering (Comparator)
Collections.sort(people, ageComparator);
System.out.println(people);  // [Alice (25), Bob (30), Charlie (40)]

// Java 8 lambda syntax
Collections.sort(people, (p1, p2) -> Integer.compare(p1.getAge(), p2.getAge()));

// Java 8 method reference and comparator methods
Collections.sort(people, Comparator.comparing(Person::getAge));
Collections.sort(people, Comparator.comparing(Person::getName).reversed());
```

### How would you sort objects based on ID?

**Answer:**
There are several ways to sort objects based on ID:

**1. Using Comparable interface:**
```java
public class Employee implements Comparable<Employee> {
    private int id;
    private String name;
    
    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }
    
    public int getId() { return id; }
    public String getName() { return name; }
    
    @Override
    public int compareTo(Employee other) {
        // Sort by ID
        return Integer.compare(this.id, other.id);
    }
    
    @Override
    public String toString() {
        return id + ": " + name;
    }
}

// Usage
List<Employee> employees = new ArrayList<>();
employees.add(new Employee(103, "Charlie"));
employees.add(new Employee(101, "Alice"));
employees.add(new Employee(102, "Bob"));

Collections.sort(employees);  // Uses compareTo method
System.out.println(employees);  // [101: Alice, 102: Bob, 103: Charlie]
```

**2. Using Comparator interface:**
```java
public class Employee {
    private int id;
    private String name;
    
    // Constructor, getters, toString as before
    // No Comparable implementation
}

// Create comparator
Comparator<Employee> idComparator = new Comparator<Employee>() {
    @Override
    public int compare(Employee e1, Employee e2) {
        return Integer.compare(e1.getId(), e2.getId());
    }
};

// Usage
List<Employee> employees = new ArrayList<>();
employees.add(new Employee(103, "Charlie"));
employees.add(new Employee(101, "Alice"));
employees.add(new Employee(102, "Bob"));

Collections.sort(employees, idComparator);
System.out.println(employees);  // [101: Alice, 102: Bob, 103: Charlie]
```

**3. Using Java 8 lambda expressions:**
```java
// Lambda syntax
Collections.sort(employees, (e1, e2) -> Integer.compare(e1.getId(), e2.getId()));

// Method reference
Collections.sort(employees, Comparator.comparingInt(Employee::getId));

// For reverse order
Collections.sort(employees, Comparator.comparingInt(Employee::getId).reversed());
```

**4. Using streams API:**
```java
List<Employee> sortedEmployees = employees.stream()
    .sorted(Comparator.comparingInt(Employee::getId))
    .collect(Collectors.toList());

System.out.println(sortedEmployees);
```

**5. Using TreeSet with custom comparator:**
```java
Set<Employee> employeeSet = new TreeSet<>(Comparator.comparingInt(Employee::getId));
employeeSet.add(new Employee(103, "Charlie"));
employeeSet.add(new Employee(101, "Alice"));
employeeSet.add(new Employee(102, "Bob"));

System.out.println(employeeSet);  // Automatically sorted by ID
```

### How would you sort objects based on name?

**Answer:**
There are several ways to sort objects based on name:

**1. Using Comparable interface:**
```java
public class Employee implements Comparable<Employee> {
    private int id;
    private String name;
    
    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }
    
    public int getId() { return id; }
    public String getName() { return name; }
    
    @Override
    public int compareTo(Employee other) {
        // Sort by name
        return this.name.compareTo(other.name);
    }
    
    @Override
    public String toString() {
        return id + ": " + name;
    }
}

// Usage
List<Employee> employees = new ArrayList<>();
employees.add(new Employee(103, "Charlie"));
employees.add(new Employee(101, "Alice"));
employees.add(new Employee(102, "Bob"));

Collections.sort(employees);  // Uses compareTo method
System.out.println(employees);  // [101: Alice, 102: Bob, 103: Charlie]
```

**2. Using Comparator interface:**
```java
public class Employee {
    private int id;
    private String name;
    
    // Constructor, getters, toString as before
    // No Comparable implementation
}

// Create comparator
Comparator<Employee> nameComparator = new Comparator<Employee>() {
    @Override
    public int compare(Employee e1, Employee e2) {
        return e1.getName().compareTo(e2.getName());
    }
};

// Usage
List<Employee> employees = new ArrayList<>();
employees.add(new Employee(103, "Charlie"));
employees.add(new Employee(101, "Alice"));
employees.add(new Employee(102, "Bob"));

Collections.sort(employees, nameComparator);
System.out.println(employees);  // [101: Alice, 102: Bob, 103: Charlie]
```

**3. Using Java 8 lambda expressions:**
```java
// Lambda syntax
Collections.sort(employees, (e1, e2) -> e1.getName().compareTo(e2.getName()));

// Method reference
Collections.sort(employees, Comparator.comparing(Employee::getName));

// Case-insensitive sorting
Collections.sort(employees, Comparator.comparing(
    Employee::getName, String.CASE_INSENSITIVE_ORDER));

// For reverse order
Collections.sort(employees, Comparator.comparing(Employee::getName).reversed());
```

**4. Using streams API:**
```java
List<Employee> sortedEmployees = employees.stream()
    .sorted(Comparator.comparing(Employee::getName))
    .collect(Collectors.toList());

System.out.println(sortedEmployees);
```

**5. Handling null values:**
```java
// Safely handle null names
Comparator<Employee> nullSafeNameComparator = Comparator.comparing(
    Employee::getName, Comparator.nullsFirst(String::compareTo));

Collections.sort(employees, nullSafeNameComparator);
```

**6. Multiple sort criteria (sort by name, then by ID):**
```java
Comparator<Employee> multiComparator = Comparator
    .comparing(Employee::getName)
    .thenComparingInt(Employee::getId);

Collections.sort(employees, multiComparator);
```

## Hashcode and Equals

### What is the hashCode and equals method implementation and what contract between them?

**Answer:**
The `hashCode()` and `equals()` methods are fundamental to the proper functioning of hash-based collections like HashMap and HashSet. They have a specific contract that must be maintained for correct behavior.

**equals() method:**
- Defines object equality (logical equality, not reference equality)
- Must be:
  - Reflexive: x.equals(x) is true
  - Symmetric: if x.equals(y) is true, then y.equals(x) must be true
  - Transitive: if x.equals(y) and y.equals(z), then x.equals(z)
  - Consistent: multiple calls with unchanged objects return same result
  - For any non-null reference x, x.equals(null) should return false

**hashCode() method:**
- Returns an integer hash value for an object
- Used by hash-based collections to determine bucket location
- Must be consistent with equals (same hash for equal objects)

**Contract between equals() and hashCode():**
1. If two objects are equal according to equals(), they MUST have the same hashCode()
2. If two objects have the same hashCode(), they are NOT necessarily equal
3. If hashCode() is overridden, equals() must be overridden too (and vice versa)

**Consequences of breaking the contract:**
- Objects may not be found in hash-based collections
- Duplicate entries may appear in Sets
- Unexpected behavior in Maps

**Example of proper implementation:**
```java
public class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;  // Same reference
        if (o == null || getClass() != o.getClass()) return false;  // Null or different class
        
        Person person = (Person) o;
        
        if (age != person.age) return false;
        return name != null ? name.equals(person.name) : person.name == null;
    }
    
    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + age;
        return result;
    }
}
```

**Java 7+ implementation using Objects helper:**
```java
import java.util.Objects;

public class Person {
    private String name;
    private int age;
    
    // Constructor
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && 
               Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

**Example demonstrating the contract:**
```java
Person p1 = new Person("John", 30);
Person p2 = new Person("John", 30);
Person p3 = new Person("Jane", 25);

// Equality check
System.out.println(p1.equals(p2));  // true - same name and age
System.out.println(p1.equals(p3));  // false - different name and age

// HashCode check
System.out.println(p1.hashCode() == p2.hashCode());  // true - must be same if equals is true
System.out.println(p1.hashCode() == p3.hashCode());  // likely false, but not guaranteed

// Using in collections
Set<Person> personSet = new HashSet<>();
personSet.add(p1);
System.out.println(personSet.contains(p2));  // true - equals and hashCode work together
```

## Reflection

### What is the importance of reflection in Java?

**Answer:**
Reflection is a powerful feature in Java that allows programs to examine and modify the behavior of classes, interfaces, fields, and methods at runtime. It enables code to inspect itself or other code dynamically.

**Key capabilities of reflection:**
1. **Examine class information:** Get class name, methods, fields, constructors, annotations
2. **Create instances:** Instantiate objects without knowing the class at compile time
3. **Access and modify fields:** Get/set field values, even private ones
4. **Invoke methods:** Call methods dynamically, including private methods
5. **Manipulate arrays:** Create and modify arrays dynamically

**Importance and use cases:**
1. **Frameworks and libraries:**
   - Dependency injection containers (Spring)
   - ORM frameworks (Hibernate)
   - Serialization/deserialization (Jackson, GSON)
   - Testing frameworks (JUnit)

2. **IDE features:**
   - Code completion
   - Debuggers
   - Visual designers

3. **Dynamic code:**
   - Plugin systems
   - Class browsers
   - Code analysis tools

4. **Runtime adaptability:**
   - Loading classes by name
   - Dynamic proxy creation
   - Adapting to different environments

**Example:**
```java
// Examining a class
Class<?> clazz = String.class;
System.out.println("Class name: " + clazz.getName());
System.out.println("Methods: " + Arrays.toString(clazz.getMethods()));

// Creating an instance
Constructor<?> constructor = clazz.getConstructor(String.class);
Object instance = constructor.newInstance("Hello");
System.out.println(instance);  // Hello

// Accessing a private field
Class<?> personClass = Person.class;
Field nameField = personClass.getDeclaredField("name");
nameField.setAccessible(true);  // Override access control
Person person = new Person("John", 30);
String name = (String) nameField.get(person);
System.out.println("Name: " + name);  // John
nameField.set(person, "Jane");
System.out.println("New name: " + person.getName());  // Jane

// Invoking a method
Method method = personClass.getMethod("setAge", int.class);
method.invoke(person, 35);
System.out.println("New age: " + person.getAge());  // 35

// Invoking a private method
Method privateMethod = personClass.getDeclaredMethod("secretMethod");
privateMethod.setAccessible(true);
privateMethod.invoke(person);
```

**Drawbacks of reflection:**
1. **Performance overhead:** Reflection is slower than direct code
2. **Security restrictions:** May be limited by security manager
3. **Loss of compile-time checking:** Runtime errors instead of compile-time errors
4. **Exposure of internals:** Can break encapsulation
5. **Maintenance challenges:** Code using reflection can be harder to understand and maintain

### What methods are available in the Objects class?

**Answer:**
The `java.util.Objects` class was introduced in Java 7 and provides static utility methods for operating on objects. It helps with common operations like null checking, equality testing, and hashing.

**Key methods in the Objects class:**

1. **Null checking:**
   - `isNull(Object obj)`: Returns true if the provided reference is null
   - `nonNull(Object obj)`: Returns true if the provided reference is not null
   - `requireNonNull(T obj)`: Checks if the object is not null, throws NullPointerException if it is
   - `requireNonNull(T obj, String message)`: Same as above but with custom exception message
   - `requireNonNullElse(T obj, T defaultObj)`: Returns obj if non-null, otherwise returns defaultObj
   - `requireNonNullElseGet(T obj, Supplier<? extends T> supplier)`: Returns obj if non-null, otherwise returns result from supplier

2. **Equality testing:**
   - `equals(Object a, Object b)`: Null-safe equality comparison
   - `deepEquals(Object a, Object b)`: Null-safe deep equality comparison for arrays

3. **Hashing:**
   - `hash(Object... values)`: Generates a hash code based on the given values
   - `hashCode(Object o)`: Returns the hash code of a non-null object or 0 if null

4. **String operations:**
   - `toString(Object o)`: Returns the string representation of an object, "null" if null
   - `toString(Object o, String nullDefault)`: Returns the string representation or nullDefault if null

5. **Comparison:**
   - `compare(T a, T b, Comparator<? super T> c)`: Null-safe comparison using the provided comparator

**Example usage:**
```java
// Null checking
String str = null;
if (Objects.isNull(str)) {
    System.out.println("String is null");
}

try {
    String required = Objects.requireNonNull(str, "String cannot be null");
} catch (NullPointerException e) {
    System.out.println(e.getMessage());  // "String cannot be null"
}

String value = Objects.requireNonNullElse(str, "Default");
System.out.println(value);  // "Default"

// Equality testing
String s1 = "hello";
String s2 = "hello";
String s3 = null;
System.out.println(Objects.equals(s1, s2));  // true
System.out.println(Objects.equals(s1, s3));  // false
System.out.println(Objects.equals(s3, s3));  // true

// Array equality
int[] arr1 = {1, 2, 3};
int[] arr2 = {1, 2, 3};
System.out.println(Objects.equals(arr1, arr2));      // false (reference comparison)
System.out.println(Objects.deepEquals(arr1, arr2));  // true (content comparison)

// Hashing
Person p1 = new Person("John", 30);
Person p2 = new Person("Jane", 25);
int hash = Objects.hash(p1, p2);
System.out.println(hash);

// toString
System.out.println(Objects.toString(s1));        // "hello"
System.out.println(Objects.toString(s3));        // "null"
System.out.println(Objects.toString(s3, "N/A")); // "N/A"

// Comparison
Comparator<String> comparator = Comparator.naturalOrder();
System.out.println(Objects.compare("a", "b", comparator));  // negative (a < b)
System.out.println(Objects.compare("b", "a", comparator));  // positive (b > a)
```

**Benefits of using Objects methods:**
1. **Null safety:** Prevents NullPointerExceptions
2. **Code readability:** Makes intentions clearer
3. **Consistency:** Standardized approach to common operations
4. **Convenience:** Reduces boilerplate code

## Fail-Fast vs Fail-Safe

### What are fail-fast and fail-safe iterators?

**Answer:**
Fail-fast and fail-safe iterators differ in how they handle concurrent modifications to the collection during iteration.

**Fail-Fast Iterators:**
- Throw ConcurrentModificationException if the collection is modified during iteration
- Use a modification counter to detect changes
- Do not create a copy of the collection
- More memory efficient but less safe for concurrent access
- Examples: Iterator from ArrayList, HashMap, HashSet, Vector

**Fail-Safe Iterators:**
- Do not throw exceptions if the collection is modified during iteration
- Work on a copy of the collection or use other mechanisms to handle concurrent modifications
- More thread-safe but may not reflect the latest state of the collection
- Examples: Iterator from ConcurrentHashMap, CopyOnWriteArrayList, CopyOnWriteArraySet

**Key differences:**

| Feature | Fail-Fast | Fail-Safe |
|---------|-----------|-----------|
| Exception | Throws ConcurrentModificationException | Does not throw exception |
| Working copy | Works on original collection | Works on copy or uses special mechanisms |
| Memory usage | Lower | Higher (for copy-based implementations) |
| Thread safety | Not thread-safe | Thread-safe |
| Consistency | Always up-to-date but may fail | May not reflect recent modifications |
| Examples | HashMap, ArrayList, HashSet | ConcurrentHashMap, CopyOnWriteArrayList |

**Example of fail-fast iterator:**
```java
List<String> list = new ArrayList<>();
list.add("one");
list.add("two");
list.add("three");

// Fail-fast iterator
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String value = iterator.next();
    System.out.println(value);
    
    if (value.equals("two")) {
        // This will throw ConcurrentModificationException
        list.remove(value);  // Modifying collection during iteration
    }
}

// Safe way to remove during iteration
Iterator<String> safeIterator = list.iterator();
while (safeIterator.hasNext()) {
    String value = safeIterator.next();
    if (value.equals("two")) {
        safeIterator.remove();  // Safe way to remove current element
    }
}
```

**Example of fail-safe iterator:**
```java
List<String> list = new CopyOnWriteArrayList<>();
list.add("one");
list.add("two");
list.add("three");

// Fail-safe iterator
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String value = iterator.next();
    System.out.println(value);
    
    // This will NOT throw exception, but "two" will still be in the iterator
    // because iterator works on a snapshot
    if (value.equals("two")) {
        list.remove(value);  // Safe but iterator won't see this change
    }
}

// ConcurrentHashMap example
Map<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("one", 1);
concurrentMap.put("two", 2);
concurrentMap.put("three", 3);

// Fail-safe iterator
Iterator<Map.Entry<String, Integer>> mapIterator = concurrentMap.entrySet().iterator();
while (mapIterator.hasNext()) {
    Map.Entry<String, Integer> entry = mapIterator.next();
    System.out.println(entry.getKey() + ": " + entry.getValue());
    
    // This will NOT throw exception
    if (entry.getKey().equals("two")) {
        concurrentMap.remove("two");
    }
}
```

### Which collections are fail-fast and which are fail-safe?

**Answer:**

**Fail-Fast Collections:**
1. **ArrayList**
2. **LinkedList**
3. **Vector**
4. **HashMap**
5. **HashSet**
6. **TreeMap**
7. **TreeSet**
8. **LinkedHashMap**
9. **LinkedHashSet**
10. **EnumMap**
11. **EnumSet**

**Fail-Safe Collections:**
1. **ConcurrentHashMap**
2. **CopyOnWriteArrayList**
3. **CopyOnWriteArraySet**
4. **ConcurrentSkipListMap**
5. **ConcurrentSkipListSet**

**Collections with special behavior:**
- **Collections.synchronizedList/Map/Set**: Fail-fast, but require external synchronization during iteration

**Example demonstrating the difference:**
```java
// Fail-fast example
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("one", 1);
hashMap.put("two", 2);
hashMap.put("three", 3);

try {
    for (Map.Entry<String, Integer> entry : hashMap.entrySet()) {
        System.out.println(entry.getKey() + ": " + entry.getValue());
        if (entry.getKey().equals("two")) {
            hashMap.put("four", 4);  // Modifying during iteration
        }
    }
} catch (ConcurrentModificationException e) {
    System.out.println("ConcurrentModificationException caught with HashMap");
}

// Fail-safe example
Map<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("one", 1);
concurrentMap.put("two", 2);
concurrentMap.put("three", 3);

// This will not throw exception
for (Map.Entry<String, Integer> entry : concurrentMap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
    if (entry.getKey().equals("two")) {
        concurrentMap.put("four", 4);  // Safe modification during iteration
    }
}
```

**How fail-fast works internally:**
- Uses a modification counter (modCount)
- Iterator takes a snapshot of modCount when created
- Before each next() call, checks if modCount has changed
- Throws ConcurrentModificationException if changed

**How fail-safe works internally:**
- **Copy-on-write collections:** Create a snapshot of the collection for iteration
- **ConcurrentHashMap:** Uses a segmented/bucket locking approach
- Changes to the collection don't affect ongoing iterations

**Best practices:**
1. **For fail-fast collections:**
   - Use iterator's remove() method for safe removal during iteration
   - Use concurrent collections if modifications during iteration are needed
   - Create a copy of the collection before iteration if needed

2. **For fail-safe collections:**
   - Be aware that iterator may not reflect recent modifications
   - Consider if the memory overhead of copy-based collections is acceptable