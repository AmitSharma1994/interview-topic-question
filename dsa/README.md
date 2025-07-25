# Data Structures & Algorithms Interview Questions

## String Manipulation

- How to find all permutations of a string?
- How to count the occurrences of each character in a string?
- How to check if a string is a palindrome?
- How to reverse a string without using built-in functions?

## Arrays

- How to remove duplicate elements from an array?
- How to sort an array using different algorithms?
- How to find the maximum and minimum elements in an array?
- How to solve the Two Sum problem?

## Linked Lists

- How to create a linked list and print all nodes?
- How to delete nodes from a linked list?
- How to detect a cycle in a linked list?
- How to reverse a linked list?

## Stacks and Queues

- How to implement a stack using arrays and linked lists?
- How to implement a queue using arrays and linked lists?
- How to solve problems using stacks and queues?

## Trees

- How to implement a binary tree?
- How to traverse a binary tree (in-order, pre-order, post-order)?
- How to check if a binary tree is balanced?
- How to find the height of a binary tree?

## Graphs

- How to represent graphs (adjacency matrix, adjacency list)?
- How to implement graph traversal algorithms (BFS, DFS)?
- How to detect cycles in a graph?
- How to find the shortest path in a graph?

## Sorting and Searching

- How to implement different sorting algorithms?
- What is the time and space complexity of different sorting algorithms?
- How to implement binary search?
- When to use different sorting algorithms?

## Dynamic Programming

- What is dynamic programming?
- How to solve problems using dynamic programming?
- What is memoization and how is it used?

## Code Examples

```java
// Example: Remove duplicates from an array
public static int[] removeDuplicates(int[] arr) {
    if (arr == null || arr.length == 0) {
        return new int[0];
    }
    
    Set<Integer> set = new LinkedHashSet<>();
    for (int num : arr) {
        set.add(num);
    }
    
    int[] result = new int[set.size()];
    int i = 0;
    for (int num : set) {
        result[i++] = num;
    }
    
    return result;
}

// Example: Two Sum problem
public static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[] { map.get(complement), i };
        }
        map.put(nums[i], i);
    }
    
    throw new IllegalArgumentException("No two sum solution");
}

// Example: Create and print a linked list
class Node {
    int data;
    Node next;
    
    public Node(int data) {
        this.data = data;
        this.next = null;
    }
}

class LinkedList {
    Node head;
    
    public void append(int data) {
        Node newNode = new Node(data);
        
        if (head == null) {
            head = newNode;
            return;
        }
        
        Node current = head;
        while (current.next != null) {
            current = current.next;
        }
        
        current.next = newNode;
    }
    
    public void printList() {
        Node current = head;
        while (current != null) {
            System.out.print(current.data + " ");
            current = current.next;
        }
        System.out.println();
    }
    
    public void deleteOddElements() {
        if (head == null) {
            return;
        }
        
        // Remove odd elements from the beginning
        while (head != null && head.data % 2 != 0) {
            head = head.next;
        }
        
        if (head == null) {
            return;
        }
        
        Node current = head;
        while (current.next != null) {
            if (current.next.data % 2 != 0) {
                current.next = current.next.next;
            } else {
                current = current.next;
            }
        }
    }
}
```

## Resources

- [GeeksforGeeks - Data Structures](https://www.geeksforgeeks.org/data-structures/)
- [LeetCode](https://leetcode.com/)
- [HackerRank](https://www.hackerrank.com/)