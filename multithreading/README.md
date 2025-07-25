# Multithreading Interview Questions

## Thread Basics

- What are the different ways of creating threads and their usage?
- What is the difference between parallelism and concurrency?
- What is the difference between a thread and a process?
- Explain the life cycle of a Thread

## Thread Coordination

- How would you print odd and even numbers from two threads using wait() and notify()?
- What are the various methods to control thread execution? (wait, join, sleep, notify, notifyAll)
- What is the yield() method and when would you use it?
- What is the difference between wait() and sleep()?
- What is the difference between notify() and notifyAll()?

## Thread Synchronization

- Why is synchronization necessary? Explain with examples
- What is the difference between synchronized block and synchronized method?
- What is the difference between class-level locking and object-level locking?
- What is the difference between synchronized keyword and Lock class?

## Thread Pools

- What is the Executor framework?
- What are the types of thread pools in the Executor framework?
- How would you create a custom ThreadPool?
- What is the difference between Future and CompletableFuture?

## Thread Safety

- What is a daemon thread?
- Do daemon threads wait for the main thread to complete?
- What tools can be used for monitoring threads? (jstack, VisualVM)
- What is the difference between deadlock, livelock, and starvation?
- How would you resolve a deadlock?

## Thread Local Storage

- What is the ThreadLocal class and how is it used?

## Code Examples

```java
// Example: Printing odd and even numbers using two threads
class SharedPrinter {
    private boolean isOdd = true;
    
    synchronized void printEven(int number) {
        while (!isOdd) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        System.out.println("Even: " + number);
        isOdd = false;
        notify();
    }
    
    synchronized void printOdd(int number) {
        while (isOdd) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        System.out.println("Odd: " + number);
        isOdd = true;
        notify();
    }
}
```

## Resources

- [Multithreading Resources](https://drive.google.com/drive/u/1/folders/1uJvX0ntpu3LGJlT6Iuy9LygTZeOJziCA)