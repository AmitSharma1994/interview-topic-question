# Java Concurrency Interview Questions

## Executor Framework

- What is the Executor Framework?
- How does it decouple task submission from task execution?
- What are the interfaces in the Executor Framework? (Executor, ExecutorService, ScheduledExecutorService)

## Thread Pools

- What are the different types of thread pools provided by the Executors class?
- When would you use each type of thread pool?
- How would you create a custom thread pool?

## Concurrent Collections

- What are concurrent collections?
- How are they different from their counterparts in java.util?
- Examples: ConcurrentHashMap, ConcurrentLinkedQueue, CopyOnWriteArrayList

## Synchronizers

- What are synchronizers in Java concurrency?
- Explain Semaphore, CountDownLatch, CyclicBarrier, and Phaser
- When would you use each of these synchronizers?

## Locks

- What are the lock interfaces and classes in java.util.concurrent?
- How are they more flexible than intrinsic locks?
- Explain ReentrantLock and ReentrantReadWriteLock

## Atomic Variables

- What are atomic variables?
- How do they support atomic operations without requiring synchronization?
- Examples: AtomicInteger, AtomicLong, AtomicReference

## Fork/Join Framework

- What is the Fork/Join Framework?
- How does it help with parallelizing recursive divide-and-conquer tasks?
- What are ForkJoinPool and RecursiveTask?

## Code Examples

```java
// Example: Using CountDownLatch
public void executeParallelTasks() {
    int numberOfThreads = 3;
    CountDownLatch latch = new CountDownLatch(numberOfThreads);
    ExecutorService executor = Executors.newFixedThreadPool(numberOfThreads);
    
    for (int i = 0; i < numberOfThreads; i++) {
        executor.submit(() -> {
            try {
                // Task execution
                System.out.println("Task executed by " + Thread.currentThread().getName());
            } finally {
                latch.countDown();
            }
        });
    }
    
    try {
        latch.await(); // Wait for all tasks to complete
        System.out.println("All tasks completed");
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        executor.shutdown();
    }
}
```

## Resources

- [Java Concurrency Resources](https://drive.google.com/drive/u/1/folders/1uJvX0ntpu3LGJlT6Iuy9LygTZeOJziCA)