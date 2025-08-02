# Data Structures & Algorithms Interview Questions and Answers

## String Manipulation

### How to find all permutations of a string?

**Answer:**
Finding all permutations involves generating all possible arrangements of characters in a string.

**Recursive approach:**
```java
public static List<String> findPermutations(String str) {
    List<String> result = new ArrayList<>();
    if (str == null || str.length() == 0) {
        result.add("");
        return result;
    }
    
    char[] chars = str.toCharArray();
    Arrays.sort(chars); // For lexicographic order
    boolean[] used = new boolean[chars.length];
    StringBuilder current = new StringBuilder();
    
    backtrack(chars, used, current, result);
    return result;
}

private static void backtrack(char[] chars, boolean[] used, 
                             StringBuilder current, List<String> result) {
    if (current.length() == chars.length) {
        result.add(current.toString());
        return;
    }
    
    for (int i = 0; i < chars.length; i++) {
        if (used[i] || (i > 0 && chars[i] == chars[i-1] && !used[i-1])) {
            continue;
        }
        
        used[i] = true;
        current.append(chars[i]);
        backtrack(chars, used, current, result);
        current.deleteCharAt(current.length() - 1);
        used[i] = false;
    }
}
```

**Iterative approach using Heap's algorithm:**
```java
public static void heapPermutation(char[] a, int size, List<String> result) {
    if (size == 1) {
        result.add(new String(a));
        return;
    }
    
    for (int i = 0; i < size; i++) {
        heapPermutation(a, size - 1, result);
        
        if (size % 2 == 1) {
            swap(a, 0, size - 1);
        } else {
            swap(a, i, size - 1);
        }
    }
}

private static void swap(char[] a, int i, int j) {
    char temp = a[i];
    a[i] = a[j];
    a[j] = temp;
}
```

**Time Complexity:** O(n! × n)  
**Space Complexity:** O(n)

### How to count the occurrences of each character in a string?

**Answer:**
Count character frequencies using different approaches:

**Using HashMap:**
```java
public static Map<Character, Integer> countCharacters(String str) {
    Map<Character, Integer> charCount = new HashMap<>();
    
    for (char c : str.toCharArray()) {
        charCount.put(c, charCount.getOrDefault(c, 0) + 1);
    }
    
    return charCount;
}
```

**Using array for ASCII characters:**
```java
public static void countCharactersArray(String str) {
    int[] count = new int[256]; // ASCII characters
    
    for (char c : str.toCharArray()) {
        count[c]++;
    }
    
    for (int i = 0; i < count.length; i++) {
        if (count[i] > 0) {
            System.out.println((char) i + ": " + count[i]);
        }
    }
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1) for array approach, O(k) for HashMap where k is unique characters

### How to check if a string is a palindrome?

**Answer:**
A palindrome reads the same forwards and backwards.

**Two-pointer approach:**
```java
public static boolean isPalindrome(String str) {
    if (str == null) return false;
    
    int left = 0;
    int right = str.length() - 1;
    
    while (left < right) {
        if (str.charAt(left) != str.charAt(right)) {
            return false;
        }
        left++;
        right--;
    }
    
    return true;
}
```

**Ignoring case and non-alphanumeric characters:**
```java
public static boolean isPalindromeAlphanumeric(String str) {
    if (str == null) return false;
    
    int left = 0;
    int right = str.length() - 1;
    
    while (left < right) {
        while (left < right && !Character.isLetterOrDigit(str.charAt(left))) {
            left++;
        }
        while (left < right && !Character.isLetterOrDigit(str.charAt(right))) {
            right--;
        }
        
        if (Character.toLowerCase(str.charAt(left)) != 
            Character.toLowerCase(str.charAt(right))) {
            return false;
        }
        
        left++;
        right--;
    }
    
    return true;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

### How to reverse a string without using built-in functions?

**Answer:**
Reverse a string using different approaches:

**Using character array:**
```java
public static String reverseString(String str) {
    if (str == null || str.length() <= 1) {
        return str;
    }
    
    char[] chars = str.toCharArray();
    int left = 0;
    int right = chars.length - 1;
    
    while (left < right) {
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;
        left++;
        right--;
    }
    
    return new String(chars);
}
```

**Using StringBuilder:**
```java
public static String reverseStringBuilder(String str) {
    if (str == null) return null;
    
    StringBuilder reversed = new StringBuilder();
    for (int i = str.length() - 1; i >= 0; i--) {
        reversed.append(str.charAt(i));
    }
    
    return reversed.toString();
}
```

**Recursive approach:**
```java
public static String reverseRecursive(String str) {
    if (str == null || str.length() <= 1) {
        return str;
    }
    
    return reverseRecursive(str.substring(1)) + str.charAt(0);
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1) for iterative, O(n) for recursive

## Arrays

### How to remove duplicate elements from an array?

**Answer:**
Remove duplicates using different approaches:

**Using HashSet (preserves first occurrence):**
```java
public static int[] removeDuplicates(int[] arr) {
    if (arr == null || arr.length == 0) {
        return new int[0];
    }
    
    Set<Integer> seen = new LinkedHashSet<>();
    for (int num : arr) {
        seen.add(num);
    }
    
    return seen.stream().mapToInt(Integer::intValue).toArray();
}
```

**In-place for sorted array:**
```java
public static int removeDuplicatesSorted(int[] arr) {
    if (arr == null || arr.length == 0) {
        return 0;
    }
    
    int writeIndex = 1;
    for (int i = 1; i < arr.length; i++) {
        if (arr[i] != arr[i - 1]) {
            arr[writeIndex++] = arr[i];
        }
    }
    
    return writeIndex; // New length
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n) for HashSet, O(1) for sorted array

### How to sort an array using different algorithms?

**Answer:**
Implementation of common sorting algorithms:

**Bubble Sort:**
```java
public static void bubbleSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr, j, j + 1);
                swapped = true;
            }
        }
        if (!swapped) break; // Optimization
    }
}
```

**Quick Sort:**
```java
public static void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr, i, j);
        }
    }
    
    swap(arr, i + 1, high);
    return i + 1;
}
```

**Merge Sort:**
```java
public static void mergeSort(int[] arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}

private static void merge(int[] arr, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;
    
    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) {
            temp[k++] = arr[i++];
        } else {
            temp[k++] = arr[j++];
        }
    }
    
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    
    System.arraycopy(temp, 0, arr, left, temp.length);
}
```

### How to find the maximum and minimum elements in an array?

**Answer:**
Find max and min elements efficiently:

**Single pass approach:**
```java
public static int[] findMinMax(int[] arr) {
    if (arr == null || arr.length == 0) {
        throw new IllegalArgumentException("Array is empty");
    }
    
    int min = arr[0];
    int max = arr[0];
    
    for (int i = 1; i < arr.length; i++) {
        if (arr[i] < min) {
            min = arr[i];
        } else if (arr[i] > max) {
            max = arr[i];
        }
    }
    
    return new int[]{min, max};
}
```

**Divide and conquer approach:**
```java
public static int[] findMinMaxDivideConquer(int[] arr, int low, int high) {
    if (low == high) {
        return new int[]{arr[low], arr[high]};
    }
    
    if (high == low + 1) {
        return arr[low] < arr[high] ? 
               new int[]{arr[low], arr[high]} : 
               new int[]{arr[high], arr[low]};
    }
    
    int mid = (low + high) / 2;
    int[] left = findMinMaxDivideConquer(arr, low, mid);
    int[] right = findMinMaxDivideConquer(arr, mid + 1, high);
    
    return new int[]{
        Math.min(left[0], right[0]),
        Math.max(left[1], right[1])
    };
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1) for iterative, O(log n) for divide and conquer

### How to solve the Two Sum problem?

**Answer:**
Find two numbers in an array that add up to a target sum:

**HashMap approach (optimal):**
```java
public static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    
    throw new IllegalArgumentException("No two sum solution");
}
```

**Two-pointer approach (for sorted array):**
```java
public static int[] twoSumSorted(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) {
            return new int[]{left, right};
        } else if (sum < target) {
            left++;
        } else {
            right--;
        }
    }
    
    throw new IllegalArgumentException("No two sum solution");
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n) for HashMap, O(1) for two-pointer

## Linked Lists

### How to create a linked list and print all nodes?

**Answer:**
Basic linked list implementation:

```java
class ListNode {
    int val;
    ListNode next;
    
    ListNode(int val) {
        this.val = val;
        this.next = null;
    }
}

class LinkedList {
    private ListNode head;
    
    public void append(int val) {
        ListNode newNode = new ListNode(val);
        
        if (head == null) {
            head = newNode;
            return;
        }
        
        ListNode current = head;
        while (current.next != null) {
            current = current.next;
        }
        current.next = newNode;
    }
    
    public void prepend(int val) {
        ListNode newNode = new ListNode(val);
        newNode.next = head;
        head = newNode;
    }
    
    public void printList() {
        ListNode current = head;
        while (current != null) {
            System.out.print(current.val + " -> ");
            current = current.next;
        }
        System.out.println("null");
    }
    
    public int size() {
        int count = 0;
        ListNode current = head;
        while (current != null) {
            count++;
            current = current.next;
        }
        return count;
    }
}
```

### How to delete nodes from a linked list?

**Answer:**
Delete nodes by value or position:

```java
public void deleteByValue(int val) {
    if (head == null) return;
    
    if (head.val == val) {
        head = head.next;
        return;
    }
    
    ListNode current = head;
    while (current.next != null && current.next.val != val) {
        current = current.next;
    }
    
    if (current.next != null) {
        current.next = current.next.next;
    }
}

public void deleteAtPosition(int position) {
    if (head == null || position < 0) return;
    
    if (position == 0) {
        head = head.next;
        return;
    }
    
    ListNode current = head;
    for (int i = 0; i < position - 1 && current.next != null; i++) {
        current = current.next;
    }
    
    if (current.next != null) {
        current.next = current.next.next;
    }
}
```

### How to detect a cycle in a linked list?

**Answer:**
Use Floyd's Cycle Detection Algorithm (Tortoise and Hare):

```java
public boolean hasCycle(ListNode head) {
    if (head == null || head.next == null) {
        return false;
    }
    
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        
        if (slow == fast) {
            return true;
        }
    }
    
    return false;
}

public ListNode detectCycleStart(ListNode head) {
    if (head == null || head.next == null) {
        return null;
    }
    
    ListNode slow = head;
    ListNode fast = head;
    
    // Detect cycle
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) break;
    }
    
    if (fast == null || fast.next == null) {
        return null; // No cycle
    }
    
    // Find cycle start
    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next;
    }
    
    return slow;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

### How to reverse a linked list?

**Answer:**
Reverse a linked list iteratively and recursively:

**Iterative approach:**
```java
public ListNode reverseIterative(ListNode head) {
    ListNode prev = null;
    ListNode current = head;
    
    while (current != null) {
        ListNode next = current.next;
        current.next = prev;
        prev = current;
        current = next;
    }
    
    return prev;
}
```

**Recursive approach:**
```java
public ListNode reverseRecursive(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    
    ListNode newHead = reverseRecursive(head.next);
    head.next.next = head;
    head.next = null;
    
    return newHead;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1) for iterative, O(n) for recursive

## Stacks and Queues

### How to implement a stack using arrays and linked lists?

**Answer:**
Stack implementation using different data structures:

**Array-based stack:**
```java
class ArrayStack {
    private int[] stack;
    private int top;
    private int capacity;
    
    public ArrayStack(int capacity) {
        this.capacity = capacity;
        this.stack = new int[capacity];
        this.top = -1;
    }
    
    public void push(int item) {
        if (isFull()) {
            throw new RuntimeException("Stack overflow");
        }
        stack[++top] = item;
    }
    
    public int pop() {
        if (isEmpty()) {
            throw new RuntimeException("Stack underflow");
        }
        return stack[top--];
    }
    
    public int peek() {
        if (isEmpty()) {
            throw new RuntimeException("Stack is empty");
        }
        return stack[top];
    }
    
    public boolean isEmpty() {
        return top == -1;
    }
    
    public boolean isFull() {
        return top == capacity - 1;
    }
}
```

**Linked list-based stack:**
```java
class LinkedStack {
    private ListNode top;
    
    public void push(int item) {
        ListNode newNode = new ListNode(item);
        newNode.next = top;
        top = newNode;
    }
    
    public int pop() {
        if (isEmpty()) {
            throw new RuntimeException("Stack underflow");
        }
        int item = top.val;
        top = top.next;
        return item;
    }
    
    public int peek() {
        if (isEmpty()) {
            throw new RuntimeException("Stack is empty");
        }
        return top.val;
    }
    
    public boolean isEmpty() {
        return top == null;
    }
}
```

### How to implement a queue using arrays and linked lists?

**Answer:**
Queue implementation using different approaches:

**Array-based circular queue:**
```java
class ArrayQueue {
    private int[] queue;
    private int front;
    private int rear;
    private int size;
    private int capacity;
    
    public ArrayQueue(int capacity) {
        this.capacity = capacity;
        this.queue = new int[capacity];
        this.front = 0;
        this.rear = -1;
        this.size = 0;
    }
    
    public void enqueue(int item) {
        if (isFull()) {
            throw new RuntimeException("Queue overflow");
        }
        rear = (rear + 1) % capacity;
        queue[rear] = item;
        size++;
    }
    
    public int dequeue() {
        if (isEmpty()) {
            throw new RuntimeException("Queue underflow");
        }
        int item = queue[front];
        front = (front + 1) % capacity;
        size--;
        return item;
    }
    
    public boolean isEmpty() {
        return size == 0;
    }
    
    public boolean isFull() {
        return size == capacity;
    }
}
```

**Linked list-based queue:**
```java
class LinkedQueue {
    private ListNode front;
    private ListNode rear;
    
    public void enqueue(int item) {
        ListNode newNode = new ListNode(item);
        
        if (rear == null) {
            front = rear = newNode;
            return;
        }
        
        rear.next = newNode;
        rear = newNode;
    }
    
    public int dequeue() {
        if (isEmpty()) {
            throw new RuntimeException("Queue underflow");
        }
        
        int item = front.val;
        front = front.next;
        
        if (front == null) {
            rear = null;
        }
        
        return item;
    }
    
    public boolean isEmpty() {
        return front == null;
    }
}
```

### How to solve problems using stacks and queues?

**Answer:**
Common problems solved using stacks and queues:

**Valid Parentheses (Stack):**
```java
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    Map<Character, Character> mapping = new HashMap<>();
    mapping.put(')', '(');
    mapping.put('}', '{');
    mapping.put(']', '[');
    
    for (char c : s.toCharArray()) {
        if (mapping.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != mapping.get(c)) {
                return false;
            }
        } else {
            stack.push(c);
        }
    }
    
    return stack.isEmpty();
}
```

**Level Order Traversal (Queue):**
```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        List<Integer> currentLevel = new ArrayList<>();
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            currentLevel.add(node.val);
            
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        
        result.add(currentLevel);
    }
    
    return result;
}
```

## Trees

### How to implement a binary tree?

**Answer:**
Basic binary tree implementation:

```java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    
    TreeNode(int val) {
        this.val = val;
        this.left = null;
        this.right = null;
    }
}

class BinaryTree {
    private TreeNode root;
    
    public void insert(int val) {
        root = insertRec(root, val);
    }
    
    private TreeNode insertRec(TreeNode root, int val) {
        if (root == null) {
            return new TreeNode(val);
        }
        
        if (val < root.val) {
            root.left = insertRec(root.left, val);
        } else if (val > root.val) {
            root.right = insertRec(root.right, val);
        }
        
        return root;
    }
    
    public boolean search(int val) {
        return searchRec(root, val);
    }
    
    private boolean searchRec(TreeNode root, int val) {
        if (root == null) {
            return false;
        }
        
        if (val == root.val) {
            return true;
        }
        
        return val < root.val ? 
               searchRec(root.left, val) : 
               searchRec(root.right, val);
    }
}
```

### How to traverse a binary tree (in-order, pre-order, post-order)?

**Answer:**
Tree traversal algorithms:

**Recursive traversals:**
```java
public void inorderTraversal(TreeNode root) {
    if (root != null) {
        inorderTraversal(root.left);
        System.out.print(root.val + " ");
        inorderTraversal(root.right);
    }
}

public void preorderTraversal(TreeNode root) {
    if (root != null) {
        System.out.print(root.val + " ");
        preorderTraversal(root.left);
        preorderTraversal(root.right);
    }
}

public void postorderTraversal(TreeNode root) {
    if (root != null) {
        postorderTraversal(root.left);
        postorderTraversal(root.right);
        System.out.print(root.val + " ");
    }
}
```

**Iterative inorder traversal:**
```java
public List<Integer> inorderIterative(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    Stack<TreeNode> stack = new Stack<>();
    TreeNode current = root;
    
    while (current != null || !stack.isEmpty()) {
        while (current != null) {
            stack.push(current);
            current = current.left;
        }
        
        current = stack.pop();
        result.add(current.val);
        current = current.right;
    }
    
    return result;
}
```

### How to check if a binary tree is balanced?

**Answer:**
Check if tree height difference between subtrees is at most 1:

```java
public boolean isBalanced(TreeNode root) {
    return checkBalance(root) != -1;
}

private int checkBalance(TreeNode root) {
    if (root == null) {
        return 0;
    }
    
    int leftHeight = checkBalance(root.left);
    if (leftHeight == -1) return -1;
    
    int rightHeight = checkBalance(root.right);
    if (rightHeight == -1) return -1;
    
    if (Math.abs(leftHeight - rightHeight) > 1) {
        return -1;
    }
    
    return Math.max(leftHeight, rightHeight) + 1;
}
```

### How to find the height of a binary tree?

**Answer:**
Calculate tree height recursively:

```java
public int maxDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    
    int leftDepth = maxDepth(root.left);
    int rightDepth = maxDepth(root.right);
    
    return Math.max(leftDepth, rightDepth) + 1;
}

// Iterative approach using level order traversal
public int maxDepthIterative(TreeNode root) {
    if (root == null) return 0;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    int depth = 0;
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        depth++;
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
    }
    
    return depth;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(h) where h is height

## Graphs

### How to represent graphs (adjacency matrix, adjacency list)?

**Answer:**
Graph representation using different data structures:

**Adjacency Matrix:**
```java
class GraphMatrix {
    private int[][] adjMatrix;
    private int numVertices;
    
    public GraphMatrix(int numVertices) {
        this.numVertices = numVertices;
        this.adjMatrix = new int[numVertices][numVertices];
    }
    
    public void addEdge(int src, int dest) {
        adjMatrix[src][dest] = 1;
        adjMatrix[dest][src] = 1; // For undirected graph
    }
    
    public void removeEdge(int src, int dest) {
        adjMatrix[src][dest] = 0;
        adjMatrix[dest][src] = 0;
    }
    
    public boolean hasEdge(int src, int dest) {
        return adjMatrix[src][dest] == 1;
    }
    
    public void printGraph() {
        for (int i = 0; i < numVertices; i++) {
            for (int j = 0; j < numVertices; j++) {
                System.out.print(adjMatrix[i][j] + " ");
            }
            System.out.println();
        }
    }
}
```

**Adjacency List:**
```java
class GraphList {
    private List<List<Integer>> adjList;
    private int numVertices;
    
    public GraphList(int numVertices) {
        this.numVertices = numVertices;
        this.adjList = new ArrayList<>();
        
        for (int i = 0; i < numVertices; i++) {
            adjList.add(new ArrayList<>());
        }
    }
    
    public void addEdge(int src, int dest) {
        adjList.get(src).add(dest);
        adjList.get(dest).add(src); // For undirected graph
    }
    
    public void removeEdge(int src, int dest) {
        adjList.get(src).remove(Integer.valueOf(dest));
        adjList.get(dest).remove(Integer.valueOf(src));
    }
    
    public List<Integer> getNeighbors(int vertex) {
        return adjList.get(vertex);
    }
    
    public void printGraph() {
        for (int i = 0; i < numVertices; i++) {
            System.out.print("Vertex " + i + ": ");
            for (int neighbor : adjList.get(i)) {
                System.out.print(neighbor + " ");
            }
            System.out.println();
        }
    }
}
```

### How to implement graph traversal algorithms (BFS, DFS)?

**Answer:**
Graph traversal implementations:

**Breadth-First Search (BFS):**
```java
public void bfs(int startVertex) {
    boolean[] visited = new boolean[numVertices];
    Queue<Integer> queue = new LinkedList<>();
    
    visited[startVertex] = true;
    queue.offer(startVertex);
    
    while (!queue.isEmpty()) {
        int vertex = queue.poll();
        System.out.print(vertex + " ");
        
        for (int neighbor : adjList.get(vertex)) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                queue.offer(neighbor);
            }
        }
    }
}
```

**Depth-First Search (DFS) - Recursive:**
```java
public void dfs(int startVertex) {
    boolean[] visited = new boolean[numVertices];
    dfsUtil(startVertex, visited);
}

private void dfsUtil(int vertex, boolean[] visited) {
    visited[vertex] = true;
    System.out.print(vertex + " ");
    
    for (int neighbor : adjList.get(vertex)) {
        if (!visited[neighbor]) {
            dfsUtil(neighbor, visited);
        }
    }
}
```

**DFS - Iterative:**
```java
public void dfsIterative(int startVertex) {
    boolean[] visited = new boolean[numVertices];
    Stack<Integer> stack = new Stack<>();
    
    stack.push(startVertex);
    
    while (!stack.isEmpty()) {
        int vertex = stack.pop();
        
        if (!visited[vertex]) {
            visited[vertex] = true;
            System.out.print(vertex + " ");
            
            for (int neighbor : adjList.get(vertex)) {
                if (!visited[neighbor]) {
                    stack.push(neighbor);
                }
            }
        }
    }
}
```

### How to detect cycles in a graph?

**Answer:**
Cycle detection for different graph types:

**Undirected Graph (using DFS):**
```java
public boolean hasCycleUndirected() {
    boolean[] visited = new boolean[numVertices];
    
    for (int i = 0; i < numVertices; i++) {
        if (!visited[i]) {
            if (hasCycleUtil(i, visited, -1)) {
                return true;
            }
        }
    }
    
    return false;
}

private boolean hasCycleUtil(int vertex, boolean[] visited, int parent) {
    visited[vertex] = true;
    
    for (int neighbor : adjList.get(vertex)) {
        if (!visited[neighbor]) {
            if (hasCycleUtil(neighbor, visited, vertex)) {
                return true;
            }
        } else if (neighbor != parent) {
            return true;
        }
    }
    
    return false;
}
```

**Directed Graph (using DFS with colors):**
```java
public boolean hasCycleDirected() {
    int[] color = new int[numVertices]; // 0: white, 1: gray, 2: black
    
    for (int i = 0; i < numVertices; i++) {
        if (color[i] == 0) {
            if (hasCycleDirectedUtil(i, color)) {
                return true;
            }
        }
    }
    
    return false;
}

private boolean hasCycleDirectedUtil(int vertex, int[] color) {
    color[vertex] = 1; // Gray
    
    for (int neighbor : adjList.get(vertex)) {
        if (color[neighbor] == 1) {
            return true; // Back edge found
        }
        if (color[neighbor] == 0 && hasCycleDirectedUtil(neighbor, color)) {
            return true;
        }
    }
    
    color[vertex] = 2; // Black
    return false;
}
```

### How to find the shortest path in a graph?

**Answer:**
Shortest path algorithms:

**Dijkstra's Algorithm:**
```java
public int[] dijkstra(int src) {
    int[] dist = new int[numVertices];
    boolean[] visited = new boolean[numVertices];
    
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
    pq.offer(new int[]{src, 0});
    
    while (!pq.isEmpty()) {
        int[] current = pq.poll();
        int u = current[0];
        
        if (visited[u]) continue;
        visited[u] = true;
        
        for (int v : adjList.get(u)) {
            if (!visited[v] && dist[u] + 1 < dist[v]) {
                dist[v] = dist[u] + 1;
                pq.offer(new int[]{v, dist[v]});
            }
        }
    }
    
    return dist;
}
```

**BFS for unweighted graphs:**
```java
public int[] shortestPathBFS(int src) {
    int[] dist = new int[numVertices];
    boolean[] visited = new boolean[numVertices];
    Queue<Integer> queue = new LinkedList<>();
    
    Arrays.fill(dist, -1);
    dist[src] = 0;
    visited[src] = true;
    queue.offer(src);
    
    while (!queue.isEmpty()) {
        int u = queue.poll();
        
        for (int v : adjList.get(u)) {
            if (!visited[v]) {
                visited[v] = true;
                dist[v] = dist[u] + 1;
                queue.offer(v);
            }
        }
    }
    
    return dist;
}
```

## Sorting and Searching

### How to implement different sorting algorithms?

**Answer:**
Implementation of various sorting algorithms:

**Selection Sort:**
```java
public static void selectionSort(int[] arr) {
    int n = arr.length;
    
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) {
                minIdx = j;
            }
        }
        swap(arr, minIdx, i);
    }
}
```

**Insertion Sort:**
```java
public static void insertionSort(int[] arr) {
    int n = arr.length;
    
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

**Heap Sort:**
```java
public static void heapSort(int[] arr) {
    int n = arr.length;
    
    // Build max heap
    for (int i = n / 2 - 1; i >= 0; i--) {
        heapify(arr, n, i);
    }
    
    // Extract elements from heap
    for (int i = n - 1; i > 0; i--) {
        swap(arr, 0, i);
        heapify(arr, i, 0);
    }
}

private static void heapify(int[] arr, int n, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    
    if (left < n && arr[left] > arr[largest]) {
        largest = left;
    }
    
    if (right < n && arr[right] > arr[largest]) {
        largest = right;
    }
    
    if (largest != i) {
        swap(arr, i, largest);
        heapify(arr, n, largest);
    }
}
```

### What is the time and space complexity of different sorting algorithms?

**Answer:**
Complexity analysis of sorting algorithms:

| Algorithm | Best Case | Average Case | Worst Case | Space Complexity | Stable |
|-----------|-----------|--------------|------------|------------------|--------|
| **Bubble Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes |
| **Selection Sort** | O(n²) | O(n²) | O(n²) | O(1) | No |
| **Insertion Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes |
| **Merge Sort** | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| **Quick Sort** | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| **Heap Sort** | O(n log n) | O(n log n) | O(n log n) | O(1) | No |

### How to implement binary search?

**Answer:**
Binary search implementation:

**Iterative binary search:**
```java
public static int binarySearch(int[] arr, int target) {
    int left = 0;
    int right = arr.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return -1; // Not found
}
```

**Recursive binary search:**
```java
public static int binarySearchRecursive(int[] arr, int target, int left, int right) {
    if (left > right) {
        return -1;
    }
    
    int mid = left + (right - left) / 2;
    
    if (arr[mid] == target) {
        return mid;
    } else if (arr[mid] < target) {
        return binarySearchRecursive(arr, target, mid + 1, right);
    } else {
        return binarySearchRecursive(arr, target, left, mid - 1);
    }
}
```

**Binary search variations:**
```java
// Find first occurrence
public static int findFirst(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            result = mid;
            right = mid - 1; // Continue searching left
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}

// Find last occurrence
public static int findLast(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            result = mid;
            left = mid + 1; // Continue searching right
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}
```

**Time Complexity:** O(log n)  
**Space Complexity:** O(1) for iterative, O(log n) for recursive

### When to use different sorting algorithms?

**Answer:**
Choosing the right sorting algorithm:

**Use Cases:**

**Bubble Sort:**
- Educational purposes only
- Very small datasets (< 10 elements)
- When simplicity is more important than efficiency

**Selection Sort:**
- Small datasets
- When memory writes are expensive
- When simplicity is preferred

**Insertion Sort:**
- Small datasets (< 50 elements)
- Nearly sorted arrays
- Online sorting (sorting data as it arrives)
- As a subroutine in hybrid algorithms

**Merge Sort:**
- When stable sorting is required
- Large datasets
- When worst-case O(n log n) is needed
- External sorting (data doesn't fit in memory)
- Linked lists

**Quick Sort:**
- General-purpose sorting
- When average-case performance is important
- In-place sorting is required
- Good cache performance needed

**Heap Sort:**
- When consistent O(n log n) performance is needed
- Limited memory available
- When you need to find top-k elements

**Practical considerations:**
```java
public static void hybridSort(int[] arr) {
    if (arr.length < 10) {
        insertionSort(arr); // Small arrays
    } else if (isNearlySorted(arr)) {
        insertionSort(arr); // Nearly sorted
    } else {
        quickSort(arr, 0, arr.length - 1); // General case
    }
}

private static boolean isNearlySorted(int[] arr) {
    int inversions = 0;
    for (int i = 0; i < arr.length - 1; i++) {
        if (arr[i] > arr[i + 1]) {
            inversions++;
            if (inversions > arr.length / 4) {
                return false;
            }
        }
    }
    return true;
}
```

## Dynamic Programming

### What is dynamic programming?

**Answer:**
Dynamic Programming (DP) is an algorithmic technique for solving optimization problems by breaking them down into simpler subproblems and storing the results to avoid redundant calculations.

**Key characteristics:**
- **Optimal Substructure:** Optimal solution contains optimal solutions to subproblems
- **Overlapping Subproblems:** Same subproblems are solved multiple times
- **Memoization:** Store results to avoid recomputation

**DP approaches:**
1. **Top-down (Memoization):** Recursive with caching
2. **Bottom-up (Tabulation):** Iterative with table

### How to solve problems using dynamic programming?

**Answer:**
Common DP problems and solutions:

**Fibonacci Sequence:**
```java
// Memoization (Top-down)
public int fibMemo(int n, int[] memo) {
    if (n <= 1) return n;
    if (memo[n] != 0) return memo[n];
    
    memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
    return memo[n];
}

// Tabulation (Bottom-up)
public int fibTab(int n) {
    if (n <= 1) return n;
    
    int[] dp = new int[n + 1];
    dp[0] = 0;
    dp[1] = 1;
    
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    
    return dp[n];
}

// Space optimized
public int fibOptimized(int n) {
    if (n <= 1) return n;
    
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int current = prev1 + prev2;
        prev2 = prev1;
        prev1 = current;
    }
    
    return prev1;
}
```

**Longest Common Subsequence (LCS):**
```java
public int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length();
    int n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    
    return dp[m][n];
}
```

**0/1 Knapsack Problem:**
```java
public int knapsack(int[] weights, int[] values, int capacity) {
    int n = weights.length;
    int[][] dp = new int[n + 1][capacity + 1];
    
    for (int i = 1; i <= n; i++) {
        for (int w = 1; w <= capacity; w++) {
            if (weights[i - 1] <= w) {
                dp[i][w] = Math.max(
                    values[i - 1] + dp[i - 1][w - weights[i - 1]],
                    dp[i - 1][w]
                );
            } else {
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    
    return dp[n][capacity];
}
```

**Coin Change Problem:**
```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;
    
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    
    return dp[amount] > amount ? -1 : dp[amount];
}
```

### What is memoization and how is it used?

**Answer:**
Memoization is a technique to store the results of expensive function calls and return cached results when the same inputs occur again.

**Memoization implementation:**
```java
class Memoization {
    private Map<String, Integer> memo = new HashMap<>();
    
    public int longestCommonSubsequenceMemo(String text1, String text2, int i, int j) {
        if (i == text1.length() || j == text2.length()) {
            return 0;
        }
        
        String key = i + "," + j;
        if (memo.containsKey(key)) {
            return memo.get(key);
        }
        
        int result;
        if (text1.charAt(i) == text2.charAt(j)) {
            result = 1 + longestCommonSubsequenceMemo(text1, text2, i + 1, j + 1);
        } else {
            result = Math.max(
                longestCommonSubsequenceMemo(text1, text2, i + 1, j),
                longestCommonSubsequenceMemo(text1, text2, i, j + 1)
            );
        }
        
        memo.put(key, result);
        return result;
    }
}
```

**Generic memoization decorator:**
```java
public class MemoizedFunction<T, R> {
    private final Function<T, R> function;
    private final Map<T, R> cache = new ConcurrentHashMap<>();
    
    public MemoizedFunction(Function<T, R> function) {
        this.function = function;
    }
    
    public R apply(T input) {
        return cache.computeIfAbsent(input, function);
    }
    
    public static <T, R> MemoizedFunction<T, R> memoize(Function<T, R> function) {
        return new MemoizedFunction<>(function);
    }
}

// Usage
MemoizedFunction<Integer, Integer> memoizedFib = 
    MemoizedFunction.memoize(n -> n <= 1 ? n : 
        memoizedFib.apply(n - 1) + memoizedFib.apply(n - 2));
```

**Benefits of memoization:**
- Reduces time complexity from exponential to polynomial
- Maintains recursive structure (easier to understand)
- Automatic caching of subproblem results

**When to use memoization:**
- Recursive problems with overlapping subproblems
- When the recursive solution is intuitive
- When you want to optimize existing recursive code
- Problems with optimal substructure