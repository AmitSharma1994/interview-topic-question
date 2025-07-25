# Multithreading Interview Questions and Answers

## Thread Basics

### What are the different ways of creating threads and their usage?

**Answer:**
In Java, there are three main ways to create threads:

1. **Extending the Thread class:**
   - Create a subclass of Thread and override the `run()` method
   - Call `start()` method to execute the thread

   ```java
   class MyThread extends Thread {
       @Override
       public void run() {
           System.out.println("Thread running: " + Thread.currentThread().getName());
       }
   }
   
   // Usage
   MyThread thread = new MyThread();
   thread.start();
   ```

2. **Implementing the Runnable interface:**
   - Implement the Runnable interface and provide implementation for `run()` method
   - Pass the Runnable object to Thread constructor
   - Call `start()` method to execute the thread

   ```java
   class MyRunnable implements Runnable {
       @Override
       public void run() {
           System.out.println("Thread running: " + Thread.currentThread().getName());
       }
   }
   
   // Usage
   Thread thread = new Thread(new MyRunnable());
   thread.start();
   ```

3. **Using Lambda expressions (Java 8+):**
   - Create a Runnable using lambda expression
   - Pass the lambda to Thread constructor
   - Call `start()` method to execute the thread

   ```java
   // Usage
   Thread thread = new Thread(() -> {
       System.out.println("Thread running: " + Thread.currentThread().getName());
   });
   thread.start();
   ```

**Comparison and Usage:**

- **Extending Thread:**
  - Advantages: Direct access to Thread methods
  - Disadvantages: Cannot extend other classes (Java single inheritance)
  - Use when: You need to override or customize Thread behavior

- **Implementing Runnable:**
  - Advantages: Can extend other classes, better separation of task from execution
  - Disadvantages: No direct access to Thread methods without using Thread.currentThread()
  - Use when: You want to separate task logic from thread management

- **Lambda expressions:**
  - Advantages: Concise syntax, good for simple tasks
  - Disadvantages: Limited to simple implementations
  - Use when: Task logic is simple and short

- **Executor Framework (modern approach):**
  - Advantages: Thread pool management, task scheduling, future results
  - Use when: Working with multiple threads, need thread lifecycle management

  ```java
  ExecutorService executor = Executors.newFixedThreadPool(5);
  executor.submit(() -> {
      System.out.println("Task executed by " + Thread.currentThread().getName());
  });
  executor.shutdown();
  ```

### What is the difference between parallelism and concurrency?

**Answer:**
Parallelism and concurrency are related concepts but have distinct meanings:

**Concurrency:**
- Definition: Multiple tasks making progress during overlapping time periods
- Execution: Tasks may start, run, and complete in overlapping time periods
- Structure: Single-core or multi-core systems
- Goal: Improve responsiveness and resource utilization
- Example: A single-core CPU switching between multiple tasks

**Parallelism:**
- Definition: Multiple tasks executing simultaneously at the exact same instant
- Execution: Tasks run at literally the same time
- Structure: Requires multi-core or distributed systems
- Goal: Improve throughput and processing speed
- Example: Multiple cores each executing a separate task

**Key differences:**

1. **Execution:**
   - Concurrency: Tasks take turns (time-slicing)
   - Parallelism: Tasks execute simultaneously

2. **Hardware requirement:**
   - Concurrency: Can occur on single-core systems
   - Parallelism: Requires multiple cores/processors

3. **Focus:**
   - Concurrency: Dealing with multiple tasks at once
   - Parallelism: Doing multiple tasks at once

4. **Relationship:**
   - Parallelism is a subset of concurrency
   - A parallel system is concurrent, but a concurrent system is not necessarily parallel

**Visual representation:**
```
Concurrency (Single-core):
Thread A: ----====----====----
Thread B: ==----====----====--
(Tasks interleave but don't run simultaneously)

Parallelism (Multi-core):
Thread A: ================
Thread B: ================
(Tasks run simultaneously)
```

**In Java:**
- Java threads can be both concurrent and parallel
- On multi-core systems, threads can run in parallel
- The JVM and OS determine the actual execution strategy

### What is the difference between a thread and a process?

**Answer:**
Threads and processes are both units of execution, but they differ in several important ways:

**Process:**
- Definition: An independent program in execution with its own memory space
- Memory: Has its own memory address space not shared with other processes
- Resources: Has its own file handles, system resources, and program counter
- Communication: Inter-process communication (IPC) is more complex and expensive
- Creation: Heavier weight, more time and resources to create
- Isolation: Strongly isolated from other processes
- Failure: One process crashing doesn't affect others
- Example: A running application like a web browser or text editor

**Thread:**
- Definition: A lightweight unit of execution within a process
- Memory: Shares memory space with other threads in the same process
- Resources: Shares file handles and system resources with other threads
- Communication: Direct communication through shared memory
- Creation: Lightweight, less time and resources to create
- Isolation: Less isolated, can affect other threads in the same process
- Failure: One thread crashing can affect the entire process
- Example: Multiple tasks within a web browser (rendering, downloading, etc.)

**Key differences:**

1. **Memory:**
   - Process: Independent memory space
   - Thread: Shared memory space within a process

2. **Resource usage:**
   - Process: More resource-intensive
   - Thread: Less resource-intensive

3. **Communication:**
   - Process: IPC mechanisms (pipes, sockets, etc.)
   - Thread: Direct memory access, shared variables

4. **Context switching:**
   - Process: More expensive context switching
   - Thread: Less expensive context switching

5. **Creation time:**
   - Process: Slower to create
   - Thread: Faster to create

**In Java:**
- JVM runs as a single process
- Java applications create threads within the JVM process
- All Java threads within an application share the same heap memory
- Each thread has its own stack memory

### Explain the life cycle of a Thread

**Answer:**
A Java thread goes through various states during its lifecycle, which are defined in the `Thread.State` enum:

1. **NEW:**
   - Thread object is created but `start()` method not yet called
   - Thread is not yet scheduled for execution
   - Example: `Thread t = new Thread()`

2. **RUNNABLE:**
   - After `start()` method is called
   - Thread is ready to run and waiting for CPU allocation
   - Includes both "ready" and "running" states
   - Example: `t.start()`

3. **BLOCKED:**
   - Thread is waiting to acquire a monitor lock
   - Occurs when trying to enter a synchronized block/method
   - Another thread is currently holding the lock
   - Example: Waiting to enter `synchronized(object) { ... }`

4. **WAITING:**
   - Thread is waiting indefinitely for another thread to perform a specific action
   - Entered by calling `Object.wait()`, `Thread.join()`, or `LockSupport.park()`
   - No timeout specified
   - Example: `object.wait()`

5. **TIMED_WAITING:**
   - Thread is waiting for another thread for a specified time period
   - Entered by calling `Thread.sleep(time)`, `Object.wait(time)`, `Thread.join(time)`, etc.
   - Example: `Thread.sleep(1000)`

6. **TERMINATED:**
   - Thread has completed execution or was terminated
   - `run()` method has exited or an uncaught exception occurred
   - Cannot be restarted
   - Example: After `run()` method completes

**State transitions:**
```
NEW → RUNNABLE → TERMINATED
     ↑  ↓
     ↑  ↓
     BLOCKED
     ↑  ↓
     ↑  ↓
     WAITING/TIMED_WAITING
```

**Code example to demonstrate thread states:**
```java
public class ThreadLifecycleDemo {
    public static void main(String[] args) throws InterruptedException {
        Object lock = new Object();
        
        Thread thread = new Thread(() -> {
            try {
                // RUNNABLE state
                System.out.println("Thread is running");
                
                // TIMED_WAITING state
                Thread.sleep(1000);
                
                synchronized (lock) {
                    System.out.println("Thread acquired lock");
                    
                    // WAITING state
                    lock.wait();
                    
                    System.out.println("Thread resumed after notification");
                }
            } catch (InterruptedException e) {
                System.out.println("Thread was interrupted");
            }
        });
        
        // NEW state
        System.out.println("Thread state: " + thread.getState());
        
        thread.start();
        System.out.println("Thread state after start: " + thread.getState());
        
        Thread.sleep(100); // Give thread time to run
        
        // Should be TIMED_WAITING due to sleep
        System.out.println("Thread state during sleep: " + thread.getState());
        
        Thread.sleep(1000); // Wait for sleep to complete
        
        synchronized (lock) {
            // Should be WAITING due to wait()
            System.out.println("Thread state during wait: " + thread.getState());
            
            // Wake up the waiting thread
            lock.notify();
        }
        
        thread.join(); // Wait for thread to terminate
        
        // TERMINATED state
        System.out.println("Thread state after completion: " + thread.getState());
    }
}

## Thread Coordination

### How would you print odd and even numbers from two threads using wait() and notify()?

**Answer:**
To print odd and even numbers from two threads using wait() and notify(), we need to create a shared object that both threads can use for coordination. Here's how to implement it:

```java
public class OddEvenPrinter {
    private static final int MAX = 10;
    private boolean isOdd = true; // Flag to control which thread should print
    
    // Method for printing even numbers
    public synchronized void printEven(int number) {
        // Wait while it's odd's turn
        while (!isOdd) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        // Print even number
        System.out.println("Even: " + number);
        
        // Toggle flag and notify waiting thread
        isOdd = false;
        notify();
    }
    
    // Method for printing odd numbers
    public synchronized void printOdd(int number) {
        // Wait while it's even's turn
        while (isOdd) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        // Print odd number
        System.out.println("Odd: " + number);
        
        // Toggle flag and notify waiting thread
        isOdd = true;
        notify();
    }
    
    public static void main(String[] args) {
        OddEvenPrinter printer = new OddEvenPrinter();
        
        // Thread for printing even numbers
        Thread evenThread = new Thread(() -> {
            for (int i = 2; i <= MAX; i += 2) {
                printer.printEven(i);
            }
        });
        
        // Thread for printing odd numbers
        Thread oddThread = new Thread(() -> {
            for (int i = 1; i <= MAX; i += 2) {
                printer.printOdd(i);
            }
        });
        
        // Start both threads
        evenThread.start();
        oddThread.start();
    }
}
```

**Explanation:**
1. We create a shared `OddEvenPrinter` object with a boolean flag `isOdd` to control which thread should print
2. The `printEven` method waits if it's not its turn (when `isOdd` is true)
3. The `printOdd` method waits if it's not its turn (when `isOdd` is false)
4. After printing, each method toggles the flag and notifies the waiting thread
5. The `wait()` method releases the lock and pauses the thread until notified
6. The `notify()` method wakes up one waiting thread
7. The synchronized keyword ensures only one thread can execute these methods at a time

### What are the various methods to control thread execution? (wait, join, sleep, notify, notifyAll)

**Answer:**
Java provides several methods to control thread execution:

1. **wait():**
   - Makes the current thread wait until another thread calls notify() or notifyAll()
   - Releases the lock on the object
   - Must be called from a synchronized context
   - Can cause InterruptedException
   
   ```java
   synchronized (lock) {
       while (!condition) {
           lock.wait(); // Wait until condition becomes true
       }
       // Continue execution
   }
   ```

2. **notify():**
   - Wakes up a single thread waiting on the object
   - Does not release the lock on the object
   - Must be called from a synchronized context
   - If multiple threads are waiting, the choice of which to wake is arbitrary
   
   ```java
   synchronized (lock) {
       // Change condition
       condition = true;
       lock.notify(); // Wake up one waiting thread
   }
   ```

3. **notifyAll():**
   - Wakes up all threads waiting on the object
   - Does not release the lock on the object
   - Must be called from a synchronized context
   - More reliable than notify() when multiple threads might be waiting
   
   ```java
   synchronized (lock) {
       // Change condition
       condition = true;
       lock.notifyAll(); // Wake up all waiting threads
   }
   ```

4. **sleep(long millis):**
   - Causes the current thread to pause execution for a specified time
   - Does not release any locks
   - Can cause InterruptedException
   - Static method of Thread class
   
   ```java
   try {
       Thread.sleep(1000); // Sleep for 1 second
   } catch (InterruptedException e) {
       Thread.currentThread().interrupt();
   }
   ```

5. **join():**
   - Waits for the thread to die
   - Current thread waits until the thread on which join() is called completes
   - Can cause InterruptedException
   - Useful for waiting for thread completion
   
   ```java
   Thread thread = new Thread(() -> {
       // Task
   });
   thread.start();
   
   try {
       thread.join(); // Wait for thread to complete
   } catch (InterruptedException e) {
       Thread.currentThread().interrupt();
   }
   // Continue after thread completes
   ```

6. **yield():**
   - Suggests that the current thread is willing to yield its current use of CPU
   - Gives other threads a chance to execute
   - No guarantee that it will actually cause a context switch
   - Static method of Thread class
   
   ```java
   Thread.yield(); // Hint to scheduler to let other threads run
   ```

7. **interrupt():**
   - Interrupts a thread, setting its interrupted status
   - If the thread is blocked in wait(), sleep(), or join(), it will throw InterruptedException
   - Used for cancellation and shutdown
   
   ```java
   Thread thread = new Thread(() -> {
       while (!Thread.currentThread().isInterrupted()) {
           // Do work
       }
   });
   thread.start();
   
   // Later
   thread.interrupt(); // Request thread to stop
   ```

### What is the yield() method and when would you use it?

**Answer:**
The `yield()` method is a static method of the Thread class that provides a hint to the scheduler that the current thread is willing to yield its current use of the processor. The scheduler is free to ignore this hint.

**Syntax:**
```java
public static native void yield()
```

**Key characteristics:**
1. It's a static method, so it affects the currently executing thread
2. It temporarily pauses the current thread to allow other threads to execute
3. The yielding thread will return to the runnable state (not waiting or sleeping)
4. There is no guarantee that yield() will actually cause a context switch
5. The exact behavior is platform and JVM implementation dependent

**When to use yield():**

1. **CPU-intensive tasks:**
   - When a thread is performing a CPU-intensive operation and wants to be "nice" to other threads
   - Allows other threads of the same or higher priority to get CPU time

2. **Testing concurrency:**
   - To test how an application behaves with different thread scheduling patterns
   - Can help expose race conditions during testing

3. **Cooperative multitasking:**
   - In scenarios where you want to implement cooperative multitasking
   - Threads voluntarily give up CPU time

4. **Busy-waiting optimization:**
   - To reduce CPU usage in busy-waiting loops
   - Gives other threads a chance to make progress

**Example:**
```java
public class YieldExample implements Runnable {
    @Override
    public void run() {
        String threadName = Thread.currentThread().getName();
        for (int i = 0; i < 5; i++) {
            System.out.println(threadName + " running, count: " + i);
            
            // Yield after each iteration
            if (threadName.equals("YieldingThread")) {
                System.out.println(threadName + " yielding control...");
                Thread.yield();
            }
        }
    }
    
    public static void main(String[] args) {
        YieldExample task = new YieldExample();
        
        Thread t1 = new Thread(task, "YieldingThread");
        Thread t2 = new Thread(task, "NormalThread");
        
        t1.start();
        t2.start();
    }
}
```

**Important notes:**
- `yield()` is rarely used in production code
- Modern JVMs have sophisticated scheduling algorithms that often make yield() unnecessary
- For most concurrency control needs, use higher-level constructs like locks, semaphores, or the java.util.concurrent package
- The behavior of yield() is not guaranteed and may vary across JVM implementations

### What is the difference between wait() and sleep()?

**Answer:**
Both `wait()` and `sleep()` methods cause a thread to pause execution, but they have several important differences:

| Aspect | wait() | sleep() |
|--------|--------|--------|
| **Package** | java.lang.Object | java.lang.Thread |
| **Method type** | Instance method | Static method |
| **Purpose** | Inter-thread communication | Thread execution control |
| **Lock release** | Releases the lock on the object | Does not release any locks |
| **Waking up** | Requires notify()/notifyAll() or timeout | Automatically after time expires or interrupt |
| **Context** | Must be called from synchronized context | Can be called from anywhere |
| **Ownership** | Must own the object's monitor | No ownership requirement |
| **Interrupted state** | Clears interrupted status and throws InterruptedException | Clears interrupted status and throws InterruptedException |

**wait() example:**
```java
public class WaitExample {
    private static final Object lock = new Object();
    private static boolean condition = false;
    
    public static void main(String[] args) throws InterruptedException {
        Thread waiterThread = new Thread(() -> {
            synchronized (lock) {
                System.out.println("Waiter thread acquired lock");
                
                while (!condition) {
                    try {
                        System.out.println("Waiter thread waiting...");
                        lock.wait(); // Releases lock and waits
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                }
                
                System.out.println("Waiter thread resumed execution");
            }
        });
        
        Thread notifierThread = new Thread(() -> {
            try {
                Thread.sleep(2000); // Sleep for 2 seconds
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            
            synchronized (lock) {
                System.out.println("Notifier thread acquired lock");
                condition = true;
                System.out.println("Notifier thread notifying...");
                lock.notify(); // Wakes up waiting thread
            }
        });
        
        waiterThread.start();
        notifierThread.start();
    }
}
```

**sleep() example:**
```java
public class SleepExample {
    private static final Object lock = new Object();
    
    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            synchronized (lock) {
                System.out.println("Thread 1 acquired lock");
                
                try {
                    System.out.println("Thread 1 sleeping...");
                    Thread.sleep(5000); // Sleeps but keeps the lock
                    System.out.println("Thread 1 woke up");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });
        
        Thread thread2 = new Thread(() -> {
            try {
                Thread.sleep(1000); // Give thread1 time to acquire lock
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            
            System.out.println("Thread 2 trying to acquire lock");
            synchronized (lock) { // Will block until thread1 releases lock
                System.out.println("Thread 2 acquired lock");
            }
        });
        
        thread1.start();
        thread2.start();
    }
}
```

**Key takeaways:**
1. Use `wait()` for inter-thread communication and coordination
2. Use `sleep()` for introducing delays without releasing locks
3. `wait()` must always be called in a loop checking the condition
4. `sleep()` can potentially cause deadlocks if called while holding locks

### What is the difference between notify() and notifyAll()?

**Answer:**
Both `notify()` and `notifyAll()` are methods of the Object class used to wake up threads that are waiting on an object's monitor, but they differ in how many threads they wake up:

| Aspect | notify() | notifyAll() |
|--------|----------|-------------|
| **Threads awakened** | Wakes up a single thread | Wakes up all waiting threads |
| **Selection** | Arbitrary selection if multiple threads are waiting | All waiting threads are notified |
| **Efficiency** | More efficient when only one thread needs to be awakened | Less efficient but safer when multiple threads might be waiting |
| **Use case** | When exactly one thread should proceed | When any/all waiting threads should check their conditions |
| **Risk** | Can cause "lost wakeup" problems | No lost wakeups, but may cause "thundering herd" |

**notify() example:**
```java
public class NotifyExample {
    private static final Object lock = new Object();
    private static boolean resourceReady = false;
    
    public static void main(String[] args) throws InterruptedException {
        // Create multiple waiting threads
        for (int i = 1; i <= 3; i++) {
            final int threadId = i;
            Thread waiter = new Thread(() -> {
                synchronized (lock) {
                    while (!resourceReady) {
                        try {
                            System.out.println("Thread " + threadId + " waiting");
                            lock.wait();
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                        }
                    }
                    System.out.println("Thread " + threadId + " processed the resource");
                }
            });
            waiter.start();
        }
        
        // Give time for all threads to start waiting
        Thread.sleep(1000);
        
        // Notify only one thread
        synchronized (lock) {
            resourceReady = true;
            System.out.println("Main thread calling notify()");
            lock.notify(); // Only one thread will wake up
        }
        
        // Wait a bit and notify remaining threads
        Thread.sleep(1000);
        synchronized (lock) {
            System.out.println("Main thread calling notify() again");
            lock.notify(); // Wake up another thread
        }
    }
}
```

**notifyAll() example:**
```java
public class NotifyAllExample {
    private static final Object lock = new Object();
    private static boolean resourceReady = false;
    
    public static void main(String[] args) throws InterruptedException {
        // Create multiple waiting threads
        for (int i = 1; i <= 3; i++) {
            final int threadId = i;
            Thread waiter = new Thread(() -> {
                synchronized (lock) {
                    while (!resourceReady) {
                        try {
                            System.out.println("Thread " + threadId + " waiting");
                            lock.wait();
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                        }
                    }
                    System.out.println("Thread " + threadId + " processed the resource");
                }
            });
            waiter.start();
        }
        
        // Give time for all threads to start waiting
        Thread.sleep(1000);
        
        // Notify all threads
        synchronized (lock) {
            resourceReady = true;
            System.out.println("Main thread calling notifyAll()");
            lock.notifyAll(); // All threads will wake up
        }
    }
}
```

**When to use notify():**
- When you know exactly one thread should proceed
- In producer-consumer scenarios where each item should be processed by exactly one consumer
- When waking up multiple threads would cause unnecessary contention

**When to use notifyAll():**
- When multiple threads might be waiting for different conditions
- When you're not sure which thread should proceed
- To avoid "lost wakeup" problems
- When the condition might be relevant to multiple waiting threads

**Best practices:**
1. Use `notifyAll()` by default unless you have a specific reason to use `notify()`
2. Always check wait conditions in a loop (to handle spurious wakeups)
3. Be aware that after notification, threads still need to reacquire the lock to proceed

## Thread Synchronization

### Why is synchronization necessary? Explain with examples

**Answer:**
Synchronization in Java is necessary to ensure thread safety when multiple threads access shared resources. Without proper synchronization, concurrent access can lead to data corruption, race conditions, and unpredictable behavior.

**Key reasons for synchronization:**

1. **Preventing race conditions:**
   - Race conditions occur when multiple threads access and modify shared data simultaneously
   - The outcome depends on the timing of thread execution
   - Synchronization ensures atomic operations on shared data

2. **Ensuring visibility:**
   - Changes made by one thread may not be immediately visible to other threads due to CPU caching
   - Synchronization establishes a happens-before relationship, ensuring changes are visible

3. **Maintaining invariants:**
   - Some operations require multiple steps to maintain data consistency
   - Synchronization prevents other threads from seeing intermediate states

4. **Preventing thread interference:**
   - Without synchronization, threads can interleave operations on shared data
   - Synchronization provides mutual exclusion

**Example 1: Counter without synchronization (unsafe):**
```java
public class UnsafeCounter {
    private int count = 0;
    
    public void increment() {
        count++; // Not atomic: read, increment, write
    }
    
    public int getCount() {
        return count;
    }
    
    public static void main(String[] args) throws InterruptedException {
        UnsafeCounter counter = new UnsafeCounter();
        
        // Create multiple threads that increment the counter
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    counter.increment();
                }
            });
            threads[i].start();
        }
        
        // Wait for all threads to complete
        for (Thread thread : threads) {
            thread.join();
        }
        
        // Expected: 1,000,000, but likely to be less due to race conditions
        System.out.println("Final count: " + counter.getCount());
    }
}
```

**Example 2: Counter with synchronization (safe):**
```java
public class SafeCounter {
    private int count = 0;
    
    public synchronized void increment() {
        count++; // Atomic due to synchronization
    }
    
    public synchronized int getCount() {
        return count;
    }
    
    public static void main(String[] args) throws InterruptedException {
        SafeCounter counter = new SafeCounter();
        
        // Create multiple threads that increment the counter
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    counter.increment();
                }
            });
            threads[i].start();
        }
        
        // Wait for all threads to complete
        for (Thread thread : threads) {
            thread.join();
        }
        
        // Expected and actual: 1,000,000
        System.out.println("Final count: " + counter.getCount());
    }
}
```

**Example 3: Bank transfer without synchronization (unsafe):**
```java
public class UnsafeBank {
    private static class Account {
        private int balance;
        
        public Account(int initialBalance) {
            this.balance = initialBalance;
        }
        
        public void withdraw(int amount) {
            balance -= amount;
        }
        
        public void deposit(int amount) {
            balance += amount;
        }
        
        public int getBalance() {
            return balance;
        }
    }
    
    public static void transfer(Account from, Account to, int amount) {
        from.withdraw(amount);
        to.deposit(amount);
    }
    
    public static void main(String[] args) throws InterruptedException {
        final Account account1 = new Account(1000);
        final Account account2 = new Account(1000);
        
        // Create threads that transfer money between accounts
        Thread[] threads = new Thread[100];
        for (int i = 0; i < threads.length; i++) {
            final int threadNum = i;
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 10; j++) {
                    if (threadNum % 2 == 0) {
                        transfer(account1, account2, 10);
                    } else {
                        transfer(account2, account1, 10);
                    }
                }
            });
            threads[i].start();
        }
        
        // Wait for all threads to complete
        for (Thread thread : threads) {
            thread.join();
        }
        
        // Expected total: 2000, but may be different due to race conditions
        int totalBalance = account1.getBalance() + account2.getBalance();
        System.out.println("Account 1 balance: " + account1.getBalance());
        System.out.println("Account 2 balance: " + account2.getBalance());
        System.out.println("Total balance: " + totalBalance);
    }
}
```

**Example 4: Bank transfer with synchronization (safe):**
```java
public class SafeBank {
    private static class Account {
        private int balance;
        
        public Account(int initialBalance) {
            this.balance = initialBalance;
        }
        
        public synchronized void withdraw(int amount) {
            balance -= amount;
        }
        
        public synchronized void deposit(int amount) {
            balance += amount;
        }
        
        public synchronized int getBalance() {
            return balance;
        }
    }
    
    // Using a lock object to synchronize the entire transfer
    private static final Object transferLock = new Object();
    
    public static void transfer(Account from, Account to, int amount) {
        synchronized (transferLock) {
            from.withdraw(amount);
            to.deposit(amount);
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        final Account account1 = new Account(1000);
        final Account account2 = new Account(1000);
        
        // Create threads that transfer money between accounts
        Thread[] threads = new Thread[100];
        for (int i = 0; i < threads.length; i++) {
            final int threadNum = i;
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 10; j++) {
                    if (threadNum % 2 == 0) {
                        transfer(account1, account2, 10);
                    } else {
                        transfer(account2, account1, 10);
                    }
                }
            });
            threads[i].start();
        }
        
        // Wait for all threads to complete
        for (Thread thread : threads) {
            thread.join();
        }
        
        // Expected and actual total: 2000
        int totalBalance = account1.getBalance() + account2.getBalance();
        System.out.println("Account 1 balance: " + account1.getBalance());
        System.out.println("Account 2 balance: " + account2.getBalance());
        System.out.println("Total balance: " + totalBalance);
    }
}
```

**Synchronization mechanisms in Java:**
1. Synchronized methods
2. Synchronized blocks
3. Locks from java.util.concurrent.locks package
4. Atomic classes from java.util.concurrent.atomic package
5. Volatile keyword (for visibility, not atomicity)
6. Thread-safe collections

### What is the difference between synchronized block and synchronized method?

**Answer:**
Both synchronized blocks and synchronized methods are mechanisms in Java to achieve thread synchronization, but they differ in scope, flexibility, and performance implications:

| Aspect | Synchronized Method | Synchronized Block |
|--------|---------------------|--------------------|
| **Syntax** | `public synchronized void method() { ... }` | `synchronized(lockObject) { ... }` |
| **Lock object** | Implicit (this for instance methods, Class object for static methods) | Explicit (any object reference) |
| **Scope** | Entire method | Only the code within the block |
| **Granularity** | Coarse-grained | Fine-grained |
| **Flexibility** | Less flexible | More flexible |
| **Performance** | May lock more code than necessary | Can minimize locked code |
| **Nested calls** | Reentrant (same thread can enter other synchronized methods of same object) | Reentrant (same thread can enter other synchronized blocks on same lock) |

**Synchronized Method Example:**
```java
public class SynchronizedMethodExample {
    private int count = 0;
    
    // Synchronized instance method (locks on 'this')
    public synchronized void increment() {
        count++;
    }
    
    // Synchronized static method (locks on SynchronizedMethodExample.class)
    public static synchronized void staticMethod() {
        System.out.println("Static synchronized method");
    }
    
    public int getCount() {
        return count;
    }
}
```

**Synchronized Block Example:**
```java
public class SynchronizedBlockExample {
    private int count = 0;
    private final Object lockObject = new Object(); // Dedicated lock object
    
    public void increment() {
        // Only synchronize the critical section
        synchronized (lockObject) {
            count++;
        }
        
        // This code is not synchronized
        System.out.println("Current count: " + count);
    }
    
    public void complexOperation() {
        // Non-synchronized code
        prepareData(); // No need for synchronization
        
        // Synchronized block for critical section only
        synchronized (lockObject) {
            count++;
            processData();
        }
        
        // More non-synchronized code
        finalizeOperation(); // No need for synchronization
    }
    
    // Using 'this' as lock object
    public void anotherOperation() {
        synchronized (this) {
            // Critical section
        }
    }
    
    // Using class object for static context
    public static void staticOperation() {
        synchronized (SynchronizedBlockExample.class) {
            // Critical section in static context
        }
    }
    
    private void prepareData() { /* ... */ }
    private void processData() { /* ... */ }
    private void finalizeOperation() { /* ... */ }
}
```

**Key differences and considerations:**

1. **Lock object:**
   - Synchronized method: Implicitly locks on `this` (instance methods) or the Class object (static methods)
   - Synchronized block: Explicitly specifies the lock object, allowing for more targeted locking

2. **Scope of synchronization:**
   - Synchronized method: Entire method is synchronized, which may be more than necessary
   - Synchronized block: Only the critical section is synchronized, potentially improving performance

3. **Multiple locks:**
   - Synchronized method: Limited to one lock per method (the implicit lock)
   - Synchronized block: Can use different lock objects for different parts of the code

4. **Performance:**
   - Synchronized block typically offers better performance by minimizing the synchronized code
   - Less contention when synchronization is limited to only what's necessary

5. **Readability:**
   - Synchronized method: Cleaner syntax, intention is clear at method level
   - Synchronized block: More verbose but makes the critical section explicit

**When to use which:**

- **Use synchronized methods when:**
  - The entire method body needs synchronization
  - Simplicity and readability are priorities
  - The method is small and focused on a single operation

- **Use synchronized blocks when:**
  - Only part of the method needs synchronization
  - You need to specify a different lock object than `this`
  - You want to minimize the duration of locking
  - You need to use multiple locks within a single method

### What is the difference between class-level locking and object-level locking?

**Answer:**
Class-level locking and object-level locking are two different synchronization approaches in Java, each with distinct characteristics and use cases:

| Aspect | Class-Level Locking | Object-Level Locking |
|--------|---------------------|----------------------|
| **Lock object** | Class object (*.class) | Instance object (this or other object) |
| **Scope** | Shared across all instances | Specific to each instance |
| **Method type** | Static methods | Instance methods |
| **Syntax (method)** | `public static synchronized void method()` | `public synchronized void method()` |
| **Syntax (block)** | `synchronized(ClassName.class) { ... }` | `synchronized(this) { ... }` or `synchronized(objectRef) { ... }` |
| **Concurrency** | One thread per class | One thread per instance |
| **Use case** | Protecting static resources | Protecting instance-specific resources |

**Class-Level Locking Example:**
```java
public class ClassLevelLockingExample {
    private static int sharedCounter = 0;
    
    // Class-level locking using synchronized static method
    public static synchronized void incrementCounter() {
        sharedCounter++;
        System.out.println("Counter: " + sharedCounter + " - Thread: " + Thread.currentThread().getName());
    }
    
    // Equivalent using synchronized block
    public static void decrementCounter() {
        synchronized (ClassLevelLockingExample.class) {
            sharedCounter--;
            System.out.println("Counter: " + sharedCounter + " - Thread: " + Thread.currentThread().getName());
        }
    }
    
    public static void main(String[] args) {
        // Create multiple threads accessing the same class lock
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                incrementCounter();
            }
        }, "Thread-1");
        
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                decrementCounter();
            }
        }, "Thread-2");
        
        t1.start();
        t2.start();
        
        // These threads will execute sequentially due to class-level locking
    }
}
```

**Object-Level Locking Example:**
```java
public class ObjectLevelLockingExample {
    private int instanceCounter = 0;
    
    // Object-level locking using synchronized instance method
    public synchronized void incrementCounter() {
        instanceCounter++;
        System.out.println("Counter: " + instanceCounter + " - Thread: " + Thread.currentThread().getName());
    }
    
    // Equivalent using synchronized block
    public void decrementCounter() {
        synchronized (this) {
            instanceCounter--;
            System.out.println("Counter: " + instanceCounter + " - Thread: " + Thread.currentThread().getName());
        }
    }
    
    // Using a different object as lock
    private final Object lockObject = new Object();
    public void anotherOperation() {
        synchronized (lockObject) {
            // Critical section
            System.out.println("Another operation - Thread: " + Thread.currentThread().getName());
        }
    }
    
    public static void main(String[] args) {
        // Create two instances
        ObjectLevelLockingExample instance1 = new ObjectLevelLockingExample();
        ObjectLevelLockingExample instance2 = new ObjectLevelLockingExample();
        
        // Threads operating on different instances
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                instance1.incrementCounter();
            }
        }, "Thread-1");
        
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                instance2.incrementCounter();
            }
        }, "Thread-2");
        
        t1.start();
        t2.start();
        
        // These threads can execute concurrently as they lock different objects
    }
}
```

**Combined Example (both types of locking):**
```java
public class CombinedLockingExample {
    // Static (shared) resources
    private static int classCounter = 0;
    
    // Instance resources
    private int instanceCounter = 0;
    
    // Class-level lock
    public static synchronized void incrementClassCounter() {
        classCounter++;
        System.out.println("Class counter: " + classCounter);
    }
    
    // Object-level lock
    public synchronized void incrementInstanceCounter() {
        instanceCounter++;
        System.out.println("Instance counter for " + this + ": " + instanceCounter);
    }
    
    // Method using both locks
    public void complexOperation() {
        // First acquire object lock
        synchronized (this) {
            instanceCounter++;
            System.out.println("Updated instance counter: " + instanceCounter);
            
            // Then acquire class lock
            synchronized (CombinedLockingExample.class) {
                classCounter++;
                System.out.println("Updated class counter: " + classCounter);
            }
        }
    }
}
```

**Key considerations:**

1. **Independence:**
   - Class locks and object locks are independent of each other
   - A thread holding an object lock doesn't block another thread from acquiring a class lock

2. **Concurrency implications:**
   - Class-level locking creates a bottleneck across all instances
   - Object-level locking allows for higher concurrency across different instances

3. **Static vs. Instance context:**
   - Static methods can only use class-level locking
   - Instance methods can use either object-level or class-level locking

4. **When to use class-level locking:**
   - When protecting static variables
   - When ensuring class-wide operations are thread-safe
   - When implementing Singleton pattern

5. **When to use object-level locking:**
   - When protecting instance variables
   - When operations are specific to an instance
   - When maximizing concurrency across instances

### What is the difference between synchronized keyword and Lock class?

**Answer:**
The synchronized keyword and the Lock interface (from java.util.concurrent.locks) are both mechanisms for thread synchronization in Java, but they differ in flexibility, functionality, and usage patterns:

| Aspect | synchronized keyword | Lock interface |
|--------|---------------------|---------------|
| **Introduction** | Java 1.0 | Java 5 |
| **Package** | Language feature | java.util.concurrent.locks |
| **Acquisition** | Implicit | Explicit (lock() method) |
| **Release** | Implicit (end of block/method) | Explicit (unlock() method) |
| **Fairness** | Not fair (no guaranteed order) | Can be fair or unfair |
| **Interruptibility** | Not interruptible while waiting | Supports interruptible waiting |
| **Timeout** | No timeout support | Supports acquisition with timeout |
| **Try-lock** | Not available | Supports non-blocking attempts (tryLock()) |
| **Multiple conditions** | One condition per object | Multiple conditions per lock |
| **Error handling** | Automatic release on exceptions | Must use try-finally to ensure release |
| **Flexibility** | Limited | Highly flexible |

**Synchronized Keyword Example:**
```java
public class SynchronizedExample {
    private final Object lockObject = new Object();
    private int counter = 0;
    
    // Using synchronized method
    public synchronized void increment() {
        counter++;
    }
    
    // Using synchronized block
    public void decrement() {
        synchronized (lockObject) {
            counter--;
        }
    }
    
    // No way to try-lock or timeout
    public void incrementWithTimeout() {
        // Not possible with synchronized
        synchronized (lockObject) {
            counter++;
        }
    }
}
```

**Lock Interface Example:**
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.TimeUnit;

public class LockExample {
    private final Lock lock = new ReentrantLock();
    private int counter = 0;
    
    // Basic locking
    public void increment() {
        lock.lock();
        try {
            counter++;
        } finally {
            lock.unlock(); // Explicit release is required
        }
    }
    
    // Try-lock (non-blocking)
    public boolean tryIncrement() {
        if (lock.tryLock()) {
            try {
                counter++;
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false; // Could not acquire lock
    }
    
    // Lock with timeout
    public boolean incrementWithTimeout() throws InterruptedException {
        if (lock.tryLock(1, TimeUnit.SECONDS)) {
            try {
                counter++;
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false; // Timeout occurred
    }
    
    // Interruptible lock acquisition
    public void incrementInterruptibly() throws InterruptedException {
        lock.lockInterruptibly();
        try {
            counter++;
        } finally {
            lock.unlock();
        }
    }
}
```

**Condition Variables Example:**
```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionExample {
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    
    private final Object[] items = new Object[100];
    private int putIndex = 0;
    private int takeIndex = 0;
    private int count = 0;
    
    public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) {
                // Wait on the notFull condition
                notFull.await();
            }
            
            items[putIndex] = x;
            putIndex = (putIndex + 1) % items.length;
            count++;
            
            // Signal consumers that buffer is not empty
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }
    
    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                // Wait on the notEmpty condition
                notEmpty.await();
            }
            
            Object x = items[takeIndex];
            items[takeIndex] = null;
            takeIndex = (takeIndex + 1) % items.length;
            count--;
            
            // Signal producers that buffer is not full
            notFull.signal();
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```

**Fair Lock Example:**
```java
import java.util.concurrent.locks.ReentrantLock;

public class FairLockExample {
    // Create a fair lock - threads acquire in FIFO order
    private final ReentrantLock fairLock = new ReentrantLock(true);
    
    public void fairMethod() {
        fairLock.lock();
        try {
            // Critical section
            System.out.println(Thread.currentThread().getName() + " acquired the fair lock");
        } finally {
            fairLock.unlock();
        }
    }
}
```

**When to use synchronized:**
- For simple locking requirements
- When code readability is a priority
- When automatic lock release is desired
- For most basic synchronization needs

**When to use Lock:**
- When you need timeout capabilities
- When you need interruptible lock acquisition
- When you need fairness guarantees
- When you need to try acquiring a lock without blocking
- When you need multiple condition variables
- For more complex locking scenarios
- When you need to know if a lock was acquired

**Best practices:**
1. Use synchronized for simple cases
2. Use Lock for advanced requirements
3. Always release Lock in a finally block
4. Consider using try-with-resources with Lock implementations that implement AutoCloseable
5. Be aware of potential deadlocks with both approaches

## Thread Pools

### What is the Executor framework?

**Answer:**
The Executor framework is a high-level API for managing threads and executing asynchronous tasks in Java. Introduced in Java 5 as part of java.util.concurrent package, it provides a standardized way to decouple task submission from task execution.

**Key components of the Executor framework:**

1. **Executor Interface:**
   - Base interface with a single method: `execute(Runnable)`
   - Executes tasks without returning results
   - Simplest abstraction for task execution

2. **ExecutorService Interface:**
   - Extends Executor with additional lifecycle and task management methods
   - Supports task submission that returns Future objects
   - Provides methods for controlled shutdown
   - Key methods: `submit()`, `invokeAll()`, `invokeAny()`, `shutdown()`, `shutdownNow()`

3. **ScheduledExecutorService Interface:**
   - Extends ExecutorService for scheduled task execution
   - Supports delayed and periodic task execution
   - Key methods: `schedule()`, `scheduleAtFixedRate()`, `scheduleWithFixedDelay()`

4. **ThreadPoolExecutor Class:**
   - Concrete implementation of ExecutorService
   - Maintains a pool of worker threads
   - Manages thread lifecycle and task queuing
   - Highly configurable (core size, max size, keep-alive time, etc.)

5. **Executors Class:**
   - Factory methods for creating different types of executor services
   - Provides pre-configured thread pools for common use cases
   - Examples: `newFixedThreadPool()`, `newCachedThreadPool()`, `newSingleThreadExecutor()`

6. **Future Interface:**
   - Represents the result of an asynchronous computation
   - Provides methods to check completion, get results, and cancel tasks
   - Key methods: `get()`, `isDone()`, `cancel()`, `isCancelled()`

7. **CompletableFuture Class:**
   - Enhanced Future with support for composition, chaining, and exception handling
   - Introduced in Java 8
   - Supports functional programming style for asynchronous operations

**Basic usage example:**
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;

public class ExecutorExample {
    public static void main(String[] args) {
        // Create a fixed thread pool with 3 threads
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        try {
            // Submit Runnable tasks (no result)
            executor.execute(() -> {
                System.out.println("Running task 1 in " + Thread.currentThread().getName());
            });
            
            // Submit Callable tasks (with result)
            Future<String> future = executor.submit(() -> {
                System.out.println("Running task 2 in " + Thread.currentThread().getName());
                Thread.sleep(1000);
                return "Task result";
            });
            
            // Get result from future (blocks until result is available)
            System.out.println("Task result: " + future.get());
            
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // Shutdown the executor
            executor.shutdown();
            try {
                // Wait for tasks to complete
                if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
                    executor.shutdownNow();
                }
            } catch (InterruptedException e) {
                executor.shutdownNow();
            }
        }
    }
}
```

**Benefits of the Executor framework:**

1. **Thread management:**
   - Reuses threads to reduce thread creation overhead
   - Manages thread lifecycle automatically
   - Controls the number of active threads

2. **Task management:**
   - Decouples task submission from execution
   - Queues tasks when all threads are busy
   - Provides task prioritization options

3. **Performance:**
   - Reduces overhead of thread creation/destruction
   - Improves application responsiveness
   - Enables better resource utilization

4. **Scalability:**
   - Easily scales from single-threaded to multi-threaded execution
   - Configurable to match application requirements

5. **Advanced features:**
   - Task scheduling
   - Task cancellation
   - Task completion notification
   - Exception handling

**Best practices:**
1. Always shut down executors properly
2. Use appropriate thread pool size for your application
3. Handle exceptions in tasks
4. Consider using higher-level abstractions like CompletableFuture for complex async operations
5. Avoid using the deprecated `java.lang.Thread` directly for new code

### What are the types of thread pools in the Executor framework?

**Answer:**
The Executor framework provides several types of thread pools through the `Executors` factory class, each designed for specific use cases:

1. **Fixed Thread Pool:**
   - Fixed number of threads
   - If all threads are busy, new tasks wait in an unbounded queue
   - Good for limiting resource usage and when you know the optimal thread count
   
   ```java
   ExecutorService fixedPool = Executors.newFixedThreadPool(5);
   ```

2. **Cached Thread Pool:**
   - Flexible number of threads
   - Creates new threads as needed, reuses idle threads
   - Terminates unused threads after 60 seconds
   - Good for many short-lived tasks
   
   ```java
   ExecutorService cachedPool = Executors.newCachedThreadPool();
   ```

3. **Single Thread Executor:**
   - Uses a single worker thread
   - Tasks execute sequentially
   - Provides task queuing
   - Good for tasks that must execute sequentially
   
   ```java
   ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
   ```

4. **Scheduled Thread Pool:**
   - Fixed number of threads for scheduled tasks
   - Supports delayed and periodic task execution
   - Good for recurring tasks or tasks that need to run after a delay
   
   ```java
   ScheduledExecutorService scheduledPool = Executors.newScheduledThreadPool(3);
   ```

5. **Single-Thread Scheduled Executor:**
   - Single thread for scheduled tasks
   - Tasks execute sequentially at specified times
   - Good for time-based sequential tasks
   
   ```java
   ScheduledExecutorService singleThreadScheduledExecutor = Executors.newSingleThreadScheduledExecutor();
   ```

6. **Work-Stealing Pool (Java 8+):**
   - Uses ForkJoinPool implementation
   - Each thread has its own queue
   - Idle threads can "steal" tasks from busy threads' queues
   - Good for recursive divide-and-conquer tasks
   - Defaults to number of available processors
   
   ```java
   ExecutorService workStealingPool = Executors.newWorkStealingPool();
   ```

7. **Custom ThreadPoolExecutor:**
   - Fully customizable thread pool
   - Control over core size, max size, keep-alive time, queue type, thread factory, and rejection policy
   - Good when you need fine-grained control
   
   ```java
   ThreadPoolExecutor customPool = new ThreadPoolExecutor(
       2,                          // Core pool size
       10,                         // Max pool size
       60, TimeUnit.SECONDS,       // Keep-alive time
       new LinkedBlockingQueue<>(), // Work queue
       new CustomThreadFactory(),   // Thread factory
       new ThreadPoolExecutor.CallerRunsPolicy() // Rejection policy
   );
   ```

**Comparison of thread pool types:**

| Thread Pool Type | Thread Creation | Queue Type | Use Case |
|------------------|----------------|------------|----------|
| Fixed | Fixed number | Unbounded | Stable, predictable workloads |
| Cached | Unlimited | SynchronousQueue | Many short-lived tasks |
| Single Thread | One thread | Unbounded | Sequential execution |
| Scheduled | Fixed number | DelayedWorkQueue | Delayed/periodic tasks |
| Work-Stealing | Parallelism level | Work-stealing deque | Recursive, divide-and-conquer tasks |

**Choosing the right thread pool:**

1. **Fixed Thread Pool:**
   - When you want to limit resource usage
   - When you know the optimal number of threads
   - For CPU-bound tasks: typically number of CPU cores
   - For I/O-bound tasks: typically higher than CPU cores

2. **Cached Thread Pool:**
   - When tasks are short-lived
   - When the number of tasks varies significantly
   - When you want to minimize thread creation overhead
   - Caution: Can create too many threads if tasks are long-running

3. **Single Thread Executor:**
   - When tasks must execute sequentially
   - When you need to ensure thread safety without synchronization
   - When you want to replace the "thread-per-task" model

4. **Scheduled Thread Pool:**
   - For delayed execution
   - For periodic tasks
   - When you need cron-like functionality

5. **Work-Stealing Pool:**
   - For recursive, divide-and-conquer algorithms
   - When tasks create subtasks
   - When you want to maximize CPU utilization

### How would you create a custom ThreadPool?

**Answer:**
Creating a custom thread pool in Java involves using the `ThreadPoolExecutor` class directly, which gives you fine-grained control over all aspects of the thread pool's behavior. Here's how to create and configure a custom thread pool:

**Basic Custom ThreadPool:**
```java
import java.util.concurrent.*;

public class CustomThreadPoolExample {
    public static void main(String[] args) {
        // Create a custom thread pool
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2,                          // corePoolSize: minimum number of threads to keep alive
            5,                          // maximumPoolSize: maximum allowed threads
            60, TimeUnit.SECONDS,       // keepAliveTime: time to keep excess idle threads alive
            new LinkedBlockingQueue<>(10), // workQueue: queue for holding tasks before execution
            Executors.defaultThreadFactory(), // threadFactory: factory for creating new threads
            new ThreadPoolExecutor.CallerRunsPolicy() // rejectionHandler: policy when queue is full
        );
        
        // Submit tasks
        for (int i = 0; i < 15; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " executed by " + 
                                 Thread.currentThread().getName());
                try {
                    Thread.sleep(1000); // Simulate work
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                return "Result of task " + taskId;
            });
        }
        
        // Print pool statistics
        System.out.println("Pool size: " + executor.getPoolSize());
        System.out.println("Active threads: " + executor.getActiveCount());
        System.out.println("Task count: " + executor.getTaskCount());
        System.out.println("Completed tasks: " + executor.getCompletedTaskCount());
        
        // Shutdown the executor
        executor.shutdown();
    }
}
```

**Key components for creating a custom thread pool:**

1. **Core parameters:**
   - **corePoolSize:** Minimum number of threads to keep alive
   - **maximumPoolSize:** Maximum allowed threads
   - **keepAliveTime:** Time to keep excess idle threads alive
   - **workQueue:** Queue for holding tasks before execution
   - **threadFactory:** Factory for creating new threads
   - **rejectionHandler:** Policy when queue is full

2. **Work queue options:**
   - **LinkedBlockingQueue:** Unbounded queue (default for fixed thread pool)
   - **ArrayBlockingQueue:** Bounded queue with fixed capacity
   - **SynchronousQueue:** No queue, tasks must be accepted by a thread or rejected (default for cached thread pool)
   - **PriorityBlockingQueue:** Priority-based queue
   - **DelayQueue:** Queue for delayed tasks

3. **Rejection policies:**
   - **AbortPolicy:** Throws RejectedExecutionException (default)
   - **CallerRunsPolicy:** Executes task in the caller's thread
   - **DiscardPolicy:** Silently discards the task
   - **DiscardOldestPolicy:** Discards the oldest task and tries again
   - **Custom policy:** Implement RejectedExecutionHandler interface

4. **Thread factory options:**
   - **Default:** Creates basic threads
   - **Custom:** Control thread naming, priority, daemon status, etc.

**Custom Thread Factory Example:**
```java
public class CustomThreadFactory implements ThreadFactory {
    private final String namePrefix;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    
    public CustomThreadFactory(String namePrefix) {
        this.namePrefix = namePrefix;
    }
    
    @Override
    public Thread newThread(Runnable r) {
        Thread thread = new Thread(r, namePrefix + "-" + threadNumber.getAndIncrement());
        
        // Configure thread properties
        thread.setDaemon(false);
        thread.setPriority(Thread.NORM_PRIORITY);
        
        // Add uncaught exception handler
        thread.setUncaughtExceptionHandler((t, e) -> {
            System.err.println("Thread " + t.getName() + " threw exception: " + e.getMessage());
        });
        
        return thread;
    }
}
```

**Custom Rejection Handler Example:**
```java
public class LoggingRejectionHandler implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.err.println("Task rejected: " + r.toString());
        System.err.println("Executor state: " + executor.toString());
        
        // Options:
        // 1. Run in the current thread
        // r.run();
        
        // 2. Throw exception
        // throw new RejectedExecutionException("Task rejected");
        
        // 3. Log and discard
        // Just log (already done above)
        
        // 4. Save for later execution
        // saveForLaterExecution(r);
    }
}
```

**Best practices for custom thread pools:**

1. **Sizing:**
   - For CPU-bound tasks: cores + 1
   - For I/O-bound tasks: higher, based on expected blocking time
   - Monitor and adjust based on performance metrics

2. **Queue sizing:**
   - Too small: Increased rejections
   - Too large: Potential memory issues and delayed feedback
   - Consider application requirements and resource constraints

3. **Shutdown properly:**
   - Call shutdown() followed by awaitTermination()
   - Use shutdownNow() as a last resort
   - Add shutdown hooks for unexpected JVM termination

4. **Exception handling:**
   - Set UncaughtExceptionHandler for threads
   - Override afterExecute() to handle task exceptions
   - Consider using Future.get() to propagate exceptions

5. **Resource management:**
   - Consider using try-with-resources for auto-closing
   - Implement AutoCloseable interface if appropriate
   - Clean up resources in terminated() method

### What is the difference between Future and CompletableFuture?

**Answer:**
Future and CompletableFuture are both interfaces/classes in Java for handling asynchronous computations, but CompletableFuture (introduced in Java 8) provides significant enhancements over the basic Future interface:

| Aspect | Future | CompletableFuture |
|--------|--------|-------------------|
| **Package** | java.util.concurrent | java.util.concurrent |
| **Introduction** | Java 5 | Java 8 |
| **Type** | Interface | Class (implements Future) |
| **Completion** | Only by the computing thread | Can be completed by any thread |
| **Composition** | Not supported | Supports chaining and combining multiple futures |
| **Exception handling** | Limited (must catch when calling get()) | Rich (exceptionally, handle, whenComplete) |
| **Callback support** | None (blocking only) | Rich (thenApply, thenAccept, thenRun) |
| **Manual completion** | Not supported | Supported (complete, completeExceptionally) |
| **Timeout control** | Only in get() method | Multiple stages with timeouts |
| **Cancellation** | Basic (cancel method) | Enhanced (with handling) |
| **Functional style** | Not supported | Fully supports functional programming |

**Future Example:**
```java
import java.util.concurrent.*;

public class FutureExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(1);
        
        // Submit a task and get a Future
        Future<String> future = executor.submit(() -> {
            Thread.sleep(1000);
            return "Result from Future";
        });
        
        try {
            // Blocking call - waits until the task completes
            String result = future.get();
            System.out.println(result);
            
            // Limitations of Future:
            // 1. Cannot perform further operations on the result without blocking
            // 2. Cannot chain multiple operations
            // 3. No built-in exception handling
            // 4. No way to combine multiple futures
            
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            executor.shutdown();
        }
    }
}
```

**CompletableFuture Example:**
```java
import java.util.concurrent.*;

public class CompletableFutureExample {
    public static void main(String[] args) {
        // Create and run a CompletableFuture
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
                return "Result from CompletableFuture";
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return "Interrupted";
            }
        });
        
        // Chain operations (non-blocking)
        future
            .thenApply(result -> {
                System.out.println("Processing: " + result);
                return result.toUpperCase();
            })
            .thenAccept(result -> {
                System.out.println("Final result: " + result);
            })
            .exceptionally(ex -> {
                System.err.println("Error: " + ex.getMessage());
                return null;
            });
        
        // Do other work while the future is processing
        System.out.println("Doing other work...");
        
        // Wait for completion if needed
        future.join(); // Similar to get() but unchecked exceptions
        
        System.out.println("Main thread completed");
    }
}
```

**Key differences in detail:**

1. **Completion mechanisms:**
   - Future: Can only be completed by the task itself
   - CompletableFuture: Can be explicitly completed by any thread using complete(), completeExceptionally()

2. **Composition and chaining:**
   - Future: No built-in composition
   - CompletableFuture: Rich composition API

   ```java
   CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
   CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");
   
   // Combine two futures
   CompletableFuture<String> combined = future1.thenCombine(future2, 
       (result1, result2) -> result1 + " " + result2);
   ```

3. **Exception handling:**
   - Future: Must use try-catch around get()
   - CompletableFuture: Multiple ways to handle exceptions

   ```java
   CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
       if (Math.random() > 0.5) {
           throw new RuntimeException("Something went wrong");
       }
       return "Success";
   });
   
   // Handle exceptions
   future
       .exceptionally(ex -> "Error: " + ex.getMessage())
       .thenAccept(System.out::println);
   ```

4. **Callback support:**
   - Future: No callbacks, must block on get()
   - CompletableFuture: Multiple callback methods

   ```java
   CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Result");
   
   // Different types of callbacks
   future.thenApply(s -> s + " processed");     // Transform result, returns CompletableFuture
   future.thenAccept(s -> System.out.println(s)); // Consumer, no return value
   future.thenRun(() -> System.out.println("Done")); // Runnable, ignores result
   ```

5. **Combining multiple futures:**
   - Future: No built-in support
   - CompletableFuture: Multiple combination methods

   ```java
   CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Result 1");
   CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "Result 2");
   CompletableFuture<String> future3 = CompletableFuture.supplyAsync(() -> "Result 3");
   
   // Wait for all to complete
   CompletableFuture<Void> allOf = CompletableFuture.allOf(future1, future2, future3);
   
   // Wait for any to complete
   CompletableFuture<Object> anyOf = CompletableFuture.anyOf(future1, future2, future3);
   ```

**When to use Future vs CompletableFuture:**

- **Use Future when:**
  - You're working with Java 5-7
  - You have simple async tasks with no dependencies
  - You don't need composition or callbacks
  - You're using ExecutorService directly

- **Use CompletableFuture when:**
  - You need to chain multiple async operations
  - You want non-blocking code with callbacks
  - You need to combine results from multiple async tasks
  - You want better exception handling
  - You need to manually complete futures
  - You're working with modern Java (8+)

## Thread Safety

### What is a daemon thread?

**Answer:**
A daemon thread in Java is a low-priority thread that runs in the background to perform tasks such as garbage collection and housekeeping operations. The key characteristic of daemon threads is that they don't prevent the JVM from exiting when all non-daemon (user) threads have completed.

**Key characteristics of daemon threads:**

1. **Background execution:**
   - Run in the background to provide services to user threads
   - Typically perform supporting tasks rather than core application logic

2. **JVM exit behavior:**
   - JVM exits when all non-daemon threads finish, even if daemon threads are still running
   - Daemon threads are abruptly terminated when JVM exits

3. **Creation:**
   - Any thread can be marked as a daemon thread before it starts
   - Use `thread.setDaemon(true)` before calling `thread.start()`
   - Cannot change daemon status after thread has started

4. **Default status:**
   - New threads inherit daemon status from the creating thread
   - Threads created by the main thread are non-daemon by default

5. **Use cases:**
   - Garbage collection
   - Monitoring and maintenance tasks
   - Background processing
   - Automatic resource cleanup

**Example:**
```java
public class DaemonThreadExample {
    public static void main(String[] args) {
        Thread daemonThread = new Thread(() -> {
            try {
                System.out.println("Daemon thread starting");
                while (true) {
                    System.out.println("Daemon thread running");
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                System.out.println("Daemon thread interrupted");
            } finally {
                System.out.println("This will not be printed because JVM exits");
            }
        });
        
        // Set as daemon thread
        daemonThread.setDaemon(true);
        daemonThread.start();
        
        // Main thread (non-daemon)
        try {
            System.out.println("Main thread sleeping");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        System.out.println("Main thread exiting");
        // JVM will exit here, terminating the daemon thread
    }
}
```

**Output:**
```
Daemon thread starting
Main thread sleeping
Daemon thread running
Daemon thread running
Daemon thread running
Main thread exiting
```

**Important notes:**
1. Daemon threads should not be used for tasks that require proper cleanup
2. Finally blocks may not execute when JVM exits
3. Resources held by daemon threads might not be properly released
4. Daemon threads are automatically terminated when all non-daemon threads finish

### Do daemon threads wait for the main thread to complete?

**Answer:**
No, daemon threads do not wait for the main thread to complete. Instead, the relationship works in the opposite direction:

1. **Main thread and JVM exit:**
   - The main thread is a non-daemon thread
   - JVM continues running as long as any non-daemon thread is active
   - When the main thread and all other non-daemon threads complete, the JVM exits

2. **Daemon thread termination:**
   - Daemon threads are terminated by the JVM when there are no more non-daemon threads
   - They do not get a chance to complete their work or cleanup resources
   - Finally blocks in daemon threads may not execute

3. **Relationship clarification:**
   - The main thread does not wait for daemon threads
   - Daemon threads do not wait for the main thread
   - The JVM waits for all non-daemon threads (including main) to finish before exiting

**Example demonstrating the relationship:**
```java
public class DaemonMainRelationship {
    public static void main(String[] args) {
        // Create a daemon thread
        Thread daemonThread = new Thread(() -> {
            try {
                System.out.println("Daemon thread starting");
                while (true) {
                    System.out.println("Daemon thread is still running");
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        daemonThread.setDaemon(true);
        
        // Create a non-daemon thread
        Thread nonDaemonThread = new Thread(() -> {
            try {
                System.out.println("Non-daemon thread starting");
                for (int i = 0; i < 5; i++) {
                    System.out.println("Non-daemon thread is running");
                    Thread.sleep(1000);
                }
                System.out.println("Non-daemon thread completed");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        // Start both threads
        daemonThread.start();
        nonDaemonThread.start();
        
        System.out.println("Main thread is completing");
        // Even though main thread completes here, JVM continues running
        // because the non-daemon thread is still active
    }
}
```

**Output:**
```
Main thread is completing
Daemon thread starting
Non-daemon thread starting
Daemon thread is still running
Non-daemon thread is running
Daemon thread is still running
Non-daemon thread is running
Daemon thread is still running
Non-daemon thread is running
Daemon thread is still running
Non-daemon thread is running
Daemon thread is still running
Non-daemon thread is running
Non-daemon thread completed
```

**Key points:**
1. The main thread completes first but the JVM continues running
2. The JVM continues because the non-daemon thread is still active
3. The daemon thread runs alongside the non-daemon thread
4. When the non-daemon thread completes, the JVM exits and terminates the daemon thread
5. The main thread does not wait for either the daemon or non-daemon thread

### What tools can be used for monitoring threads? (jstack, VisualVM)

**Answer:**
Several tools are available for monitoring and analyzing threads in Java applications:

1. **jstack (JDK Tool):**
   - Command-line utility included with JDK
   - Prints Java thread stack traces for a given Java process
   - Helps diagnose deadlocks, high CPU usage, and thread states
   - Usage: `jstack [options] <pid>`
   
   ```bash
   # Example usage
   jstack -l 12345 > thread_dump.txt
   ```

2. **VisualVM (JDK Tool):**
   - Visual tool for monitoring and profiling Java applications
   - Provides thread visualization, CPU sampling, memory analysis
   - Shows thread states, locks, and stack traces graphically
   - Can monitor local and remote applications
   - Available as standalone download or bundled with JDK (until JDK 8)

3. **Java Mission Control (JMC):**
   - Advanced monitoring and management tool
   - Includes Flight Recorder for detailed thread analysis
   - Low-overhead profiling suitable for production
   - Visualizes thread allocation, contention, and CPU usage
   - Available as standalone download or bundled with some JDK distributions

4. **JConsole (JDK Tool):**
   - JMX-based monitoring tool
   - Provides thread statistics and stack traces
   - Shows thread counts, states, and CPU usage
   - Usage: `jconsole [options] [pid | host:port]`

5. **IntelliJ IDEA Profiler:**
   - Built into IntelliJ IDEA Ultimate Edition
   - Provides thread dump analysis and visualization
   - CPU and memory profiling with thread context

6. **Eclipse Memory Analyzer (MAT):**
   - Primarily for heap analysis but also shows thread information
   - Can analyze thread stacks and detect deadlocks
   - Works with heap dumps that include thread information

7. **Arthas:**
   - Open-source Java diagnostic tool by Alibaba
   - Provides thread analysis, method tracing, and more
   - Can be attached to running JVMs without restart
   - Includes thread pool metrics and thread stack analysis

8. **Async-profiler:**
   - Low-overhead profiler for Java
   - Captures CPU, allocation, and lock contention profiles
   - Shows thread activity with flame graphs

**jstack example output:**
```
"DestroyJavaVM" #36 prio=5 os_prio=0 tid=0x00007f9ab006e000 nid=0x1234 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"pool-1-thread-1" #35 prio=5 os_prio=0 tid=0x00007f9a94129000 nid=0x5678 waiting on condition [0x00007f9a8f7fe000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at com.example.ThreadExample.lambda$main$0(ThreadExample.java:15)
        at com.example.ThreadExample$$Lambda$1/1175962212.run(Unknown Source)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
```

**Thread monitoring best practices:**

1. **Regular thread dumps:**
   - Take multiple thread dumps a few seconds apart
   - Compare dumps to identify stuck or high-CPU threads
   - Look for patterns in thread states and stack traces

2. **Deadlock detection:**
   - Use jstack's `-l` option to detect deadlocks
   - Look for "Found one Java-level deadlock" in the output
   - Analyze the threads and locks involved

3. **Thread naming:**
   - Use meaningful thread names in your application
   - Makes thread dumps easier to analyze
   - Example: `Thread.currentThread().setName("OrderProcessor-" + orderId)`

4. **Thread pool monitoring:**
   - Monitor active count, queue size, and completed tasks
   - Expose metrics via JMX for real-time monitoring
   - Adjust pool sizes based on monitoring data

5. **Production monitoring:**
   - Use low-overhead tools like JMC Flight Recorder
   - Set up automated thread dump collection on high CPU usage
   - Correlate thread activity with application metrics

### What is the difference between deadlock, livelock, and starvation?

**Answer:**
Deadlock, livelock, and starvation are three different concurrency problems that can occur in multithreaded applications:

| Aspect | Deadlock | Livelock | Starvation |
|--------|----------|----------|------------|
| **Definition** | Two or more threads blocked forever, each waiting for resources held by the other | Threads actively respond to each other but make no progress | A thread is perpetually denied access to resources it needs |
| **Thread state** | BLOCKED | RUNNABLE (active) | RUNNABLE or BLOCKED |
| **System state** | Frozen, no progress | Active, no progress | Partial progress (some threads make progress) |
| **Resource holding** | Each thread holds a resource and waits for another | Threads keep changing state in response to others | Resources monopolized by other threads |
| **Detection** | Relatively easy (cycle detection) | Harder (threads appear active) | Very difficult (looks like normal execution) |
| **Recovery** | Requires restart or thread termination | May resolve with randomization | Requires policy changes |

**1. Deadlock:**
A situation where two or more threads are blocked forever, each waiting for resources held by the other threads.

**Deadlock example:**
```java
public class DeadlockExample {
    private static final Object LOCK_1 = new Object();
    private static final Object LOCK_2 = new Object();
    
    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            synchronized (LOCK_1) {
                System.out.println("Thread 1: Holding lock 1...");
                try { Thread.sleep(100); } catch (InterruptedException e) {}
                
                System.out.println("Thread 1: Waiting for lock 2...");
                synchronized (LOCK_2) {
                    System.out.println("Thread 1: Holding lock 1 and lock 2");
                }
            }
        });
        
        Thread thread2 = new Thread(() -> {
            synchronized (LOCK_2) {
                System.out.println("Thread 2: Holding lock 2...");
                try { Thread.sleep(100); } catch (InterruptedException e) {}
                
                System.out.println("Thread 2: Waiting for lock 1...");
                synchronized (LOCK_1) {
                    System.out.println("Thread 2: Holding lock 2 and lock 1");
                }
            }
        });
        
        thread1.start();
        thread2.start();
    }
}
```

**2. Livelock:**
A situation where threads keep changing their state in response to each other without making any real progress.

**Livelock example:**
```java
public class LivelockExample {
    static class Spoon {
        private Diner owner;
        
        public Spoon(Diner owner) {
            this.owner = owner;
        }
        
        public Diner getOwner() {
            return owner;
        }
        
        public void setOwner(Diner owner) {
            this.owner = owner;
        }
        
        public synchronized void use() {
            System.out.println(owner.name + " is eating!");
        }
    }
    
    static class Diner {
        private String name;
        private boolean isHungry;
        
        public Diner(String name) {
            this.name = name;
            this.isHungry = true;
        }
        
        public void eatWith(Spoon spoon, Diner spouse) {
            while (isHungry) {
                // Don't have the spoon, so wait patiently for spouse
                if (spoon.getOwner() != this) {
                    try { Thread.sleep(10); } catch (InterruptedException e) {}
                    continue;
                }
                
                // If spouse is hungry, insist upon passing the spoon
                if (spouse.isHungry) {
                    System.out.println(name + ": You eat first, my dear " + spouse.name);
                    spoon.setOwner(spouse);
                    continue;
                }
                
                // Spouse wasn't hungry, so finally eat
                spoon.use();
                isHungry = false;
                System.out.println(name + ": I am no longer hungry.");
                spoon.setOwner(spouse);
            }
        }
    }
    
    public static void main(String[] args) {
        final Diner husband = new Diner("Bob");
        final Diner wife = new Diner("Alice");
        final Spoon spoon = new Spoon(husband);
        
        new Thread(() -> husband.eatWith(spoon, wife)).start();
        new Thread(() -> wife.eatWith(spoon, husband)).start();
    }
}
```

**3. Starvation:**
A situation where a thread is perpetually denied access to resources it needs to progress.

**Starvation example:**
```java
public class StarvationExample {
    private static final Object LOCK = new Object();
    
    public static void main(String[] args) {
        // Create a thread with low priority that tries to acquire the lock
        Thread lowPriorityThread = new Thread(() -> {
            System.out.println("Low priority thread waiting for lock");
            synchronized (LOCK) {
                System.out.println("Low priority thread acquired lock");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {}
            }
        });
        lowPriorityThread.setPriority(Thread.MIN_PRIORITY);
        
        // Create 10 high priority threads that continuously acquire and release the lock
        for (int i = 0; i < 10; i++) {
            Thread highPriorityThread = new Thread(() -> {
                while (true) {
                    synchronized (LOCK) {
                        // Hold the lock briefly
                        try {
                            Thread.sleep(10);
                        } catch (InterruptedException e) {
                            break;
                        }
                    }
                    // Release the lock briefly, but immediately try to reacquire
                    Thread.yield();
                }
            });
            highPriorityThread.setPriority(Thread.MAX_PRIORITY);
            highPriorityThread.setDaemon(true); // Make daemon so JVM can exit
            highPriorityThread.start();
        }
        
        // Start the low priority thread
        lowPriorityThread.start();
    }
}
```

**Prevention strategies:**

1. **Deadlock prevention:**
   - Lock ordering: Always acquire locks in the same order
   - Lock timeouts: Use tryLock with timeout instead of blocking indefinitely
   - Deadlock detection: Use tools to detect and recover from deadlocks
   - Resource hierarchy: Establish a hierarchy for resource acquisition

2. **Livelock prevention:**
   - Add randomness: Introduce random delays before retrying
   - Backoff strategies: Increase delay between retries
   - Priority-based resolution: Establish priorities for conflict resolution
   - Third-party arbitration: Use a separate component to resolve conflicts

3. **Starvation prevention:**
   - Fair locks: Use fair lock implementations (ReentrantLock(true))
   - Thread aging: Gradually increase priority of waiting threads
   - Time slicing: Ensure all threads get CPU time
   - Resource limits: Limit resource usage by any single thread

### How would you resolve a deadlock?

**Answer:**
Resolving deadlocks involves both detection and recovery strategies, as well as preventive measures to avoid them in the future:

**1. Deadlock Detection:**

a) **Manual detection:**
   - Take thread dumps using jstack or similar tools
   - Look for threads in BLOCKED state waiting for locks
   - Identify circular dependencies between threads

   ```bash
   jstack -l <pid> > thread_dump.txt
   ```

b) **Automated detection:**
   - Use tools like VisualVM, JConsole, or Java Mission Control
   - Set up deadlock detection in your application
   - Implement watchdog threads that monitor for potential deadlocks

   ```java
   // Example of programmatic deadlock detection
   ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
   long[] deadlockedThreads = threadMXBean.findDeadlockedThreads();
   if (deadlockedThreads != null) {
       System.out.println("Deadlock detected!");
       // Take recovery action
   }
   ```

**2. Immediate Recovery Strategies:**

a) **Thread interruption:**
   - Interrupt one of the deadlocked threads
   - May work if threads properly handle interruption

   ```java
   deadlockedThread.interrupt();
   ```

b) **Thread termination:**
   - Force termination of one or more deadlocked threads
   - May leave resources in inconsistent state

   ```java
   // Deprecated but sometimes necessary
   deadlockedThread.stop();
   ```

c) **Application restart:**
   - In many cases, restarting the application is the safest recovery method
   - Implement proper shutdown hooks to clean up resources

   ```java
   System.exit(1); // Force JVM to exit
   ```

d) **Timeout-based recovery:**
   - Use lock acquisition with timeouts to break deadlocks
   - Release all acquired resources if timeout occurs

   ```java
   Lock lock1 = new ReentrantLock();
   Lock lock2 = new ReentrantLock();
   
   if (lock1.tryLock(1, TimeUnit.SECONDS)) {
       try {
           if (lock2.tryLock(1, TimeUnit.SECONDS)) {
               try {
                   // Work with both locks
               } finally {
                   lock2.unlock();
               }
           } else {
               // Failed to get lock2, handle accordingly
           }
       } finally {
           lock1.unlock();
       }
   }
   ```

**3. Preventive Measures for Future Code:**

a) **Lock ordering:**
   - Always acquire locks in a consistent, predefined order
   - Prevents circular wait condition

   ```java
   // Good practice: consistent lock ordering
   public void transferMoney(Account from, Account to, double amount) {
       // Sort accounts to ensure consistent lock ordering
       Account firstLock = from.getId() < to.getId() ? from : to;
       Account secondLock = from.getId() < to.getId() ? to : from;
       
       synchronized (firstLock) {
           synchronized (secondLock) {
               // Transfer logic
           }
       }
   }
   ```

b) **Lock timeouts:**
   - Use tryLock with timeout instead of indefinite waiting
   - Release all acquired resources if any lock acquisition fails

c) **Avoid nested locks:**
   - Minimize the number of locks held simultaneously
   - Reduce lock granularity and duration

d) **Use higher-level concurrency utilities:**
   - java.util.concurrent collections
   - Semaphores, CountDownLatch, CyclicBarrier
   - ThreadPoolExecutor for thread management

e) **Lock-free algorithms:**
   - Use atomic variables and CAS operations
   - Implement non-blocking algorithms where possible

   ```java
   AtomicInteger counter = new AtomicInteger(0);
   counter.incrementAndGet(); // Thread-safe without locks
   ```

f) **Resource allocation graph:**
   - Model resource allocation and detect cycles
   - Implement deadlock avoidance algorithms

**4. Monitoring and Management:**

a) **Implement deadlock detection:**
   - Periodic checking for potential deadlocks
   - Automated alerts when deadlocks are detected

b) **Resource health checks:**
   - Monitor lock acquisition times
   - Alert on unusually long lock holding

c) **Circuit breakers:**
   - Implement circuit breakers to prevent cascading failures
   - Automatically release resources when system is under stress

**Example of a deadlock resolution system:**
```java
public class DeadlockDetector {
    private final ThreadMXBean threadMXBean;
    private final ScheduledExecutorService scheduler;
    
    public DeadlockDetector() {
        this.threadMXBean = ManagementFactory.getThreadMXBean();
        this.scheduler = Executors.newSingleThreadScheduledExecutor();
    }
    
    public void start() {
        scheduler.scheduleAtFixedRate(this::checkForDeadlocks, 10, 10, TimeUnit.SECONDS);
    }
    
    private void checkForDeadlocks() {
        long[] deadlockedThreadIds = threadMXBean.findDeadlockedThreads();
        if (deadlockedThreadIds != null) {
            ThreadInfo[] threadInfos = threadMXBean.getThreadInfo(deadlockedThreadIds, true, true);
            System.err.println("Deadlock detected!");
            
            // Log detailed information about deadlocked threads
            for (ThreadInfo threadInfo : threadInfos) {
                System.err.println(threadInfo.toString());
            }
            
            // Take recovery action
            recoverFromDeadlock(threadInfos);
        }
    }
    
    private void recoverFromDeadlock(ThreadInfo[] threadInfos) {
        // Option 1: Log and alert
        alertOperations(threadInfos);
        
        // Option 2: Interrupt deadlocked threads
        for (ThreadInfo threadInfo : threadInfos) {
            Thread thread = findThreadById(threadInfo.getThreadId());
            if (thread != null) {
                thread.interrupt();
            }
        }
        
        // Option 3: Restart application component
        // restartComponent();
    }
    
    private Thread findThreadById(long id) {
        for (Thread thread : Thread.getAllStackTraces().keySet()) {
            if (thread.getId() == id) {
                return thread;
            }
        }
        return null;
    }
    
    private void alertOperations(ThreadInfo[] threadInfos) {
        // Send alert to operations team
    }
    
    public void stop() {
        scheduler.shutdown();
    }
}
```

**Best practices summary:**
1. Design to prevent deadlocks rather than recover from them
2. Implement detection mechanisms for early warning
3. Have a clear recovery strategy when deadlocks occur
4. Use timeouts for all resource acquisitions
5. Minimize lock scope and duration
6. Consider using higher-level concurrency utilities

### What is the ThreadLocal class and how is it used?

**Answer:**
ThreadLocal is a class in Java that provides thread-local variables. These variables differ from normal variables in that each thread that accesses a ThreadLocal variable has its own, independently initialized copy of the variable. ThreadLocal instances are typically private static fields in classes that wish to associate state with a thread.

**Key characteristics of ThreadLocal:**

1. **Thread isolation:**
   - Each thread has its own copy of the variable
   - Changes made by one thread are not visible to other threads
   - No synchronization needed for thread-local access

2. **Use cases:**
   - Storing user identity or transaction context in web applications
   - Maintaining per-thread caches
   - Storing thread-specific configuration
   - Implementing context carriers in frameworks
   - Reducing contention in high-concurrency scenarios

3. **Memory considerations:**
   - ThreadLocal values are stored in the Thread object
   - Can cause memory leaks if not properly managed
   - Should be removed when no longer needed

**Basic usage example:**
```java
public class ThreadLocalExample {
    // Create a ThreadLocal variable
    private static final ThreadLocal<String> userContext = new ThreadLocal<>();
    
    public static void main(String[] args) {
        // Thread 1: Sets and uses its own copy of the ThreadLocal
        Thread thread1 = new Thread(() -> {
            userContext.set("User-A");
            System.out.println("Thread 1: " + userContext.get());
            
            // Process with user context
            processRequest();
            
            // Clean up ThreadLocal
            userContext.remove();
        });
        
        // Thread 2: Sets and uses its own copy of the ThreadLocal
        Thread thread2 = new Thread(() -> {
            userContext.set("User-B");
            System.out.println("Thread 2: " + userContext.get());
            
            // Process with user context
            processRequest();
            
            // Clean up ThreadLocal
            userContext.remove();
        });
        
        thread1.start();
        thread2.start();
    }
    
    private static void processRequest() {
        // Access the ThreadLocal variable from anywhere in the call stack
        System.out.println("Processing request for: " + userContext.get());
    }
}
```

**Output:**
```
Thread 1: User-A
Processing request for: User-A
Thread 2: User-B
Processing request for: User-B
```

**ThreadLocal with initial value:**
```java
private static final ThreadLocal<SimpleDateFormat> dateFormatter = 
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

// Usage
String formattedDate = dateFormatter.get().format(new Date());
```

**InheritableThreadLocal:**
A subclass of ThreadLocal that allows a child thread to inherit the thread-local values from its parent thread.

```java
private static final InheritableThreadLocal<String> contextHolder = 
    new InheritableThreadLocal<>();

public static void main(String[] args) {
    contextHolder.set("Parent Value");
    
    Thread childThread = new Thread(() -> {
        // Child thread inherits the value from parent thread
        System.out.println("Child thread: " + contextHolder.get());
        
        // Child can override its inherited value
        contextHolder.set("Child Value");
        System.out.println("Child thread after change: " + contextHolder.get());
    });
    
    childThread.start();
    System.out.println("Parent thread: " + contextHolder.get());
}
```

**Real-world example: Transaction context management**
```java
public class TransactionContext {
    private static final ThreadLocal<String> currentTransactionId = new ThreadLocal<>();
    private static final ThreadLocal<Boolean> transactionActive = ThreadLocal.withInitial(() -> Boolean.FALSE);
    
    public static void beginTransaction() {
        if (transactionActive.get()) {
            throw new IllegalStateException("Transaction already active");
        }
        currentTransactionId.set(generateTransactionId());
        transactionActive.set(true);
        System.out.println("Transaction started: " + currentTransactionId.get());
    }
    
    public static String getCurrentTransactionId() {
        if (!transactionActive.get()) {
            throw new IllegalStateException("No active transaction");
        }
        return currentTransactionId.get();
    }
    
    public static void commitTransaction() {
        if (!transactionActive.get()) {
            throw new IllegalStateException("No active transaction");
        }
        System.out.println("Transaction committed: " + currentTransactionId.get());
        cleanup();
    }
    
    public static void rollbackTransaction() {
        if (!transactionActive.get()) {
            throw new IllegalStateException("No active transaction");
        }
        System.out.println("Transaction rolled back: " + currentTransactionId.get());
        cleanup();
    }
    
    private static void cleanup() {
        currentTransactionId.remove();
        transactionActive.set(false);
    }
    
    private static String generateTransactionId() {
        return "TXN-" + UUID.randomUUID().toString().substring(0, 8);
    }
}
```

**Memory leak concerns and prevention:**
ThreadLocal variables can cause memory leaks if not properly managed, especially in thread pool environments where threads are reused.

1. **The problem:**
   - ThreadLocal values are stored in the Thread object
   - In thread pools, threads are reused but ThreadLocal values persist
   - If not cleared, values accumulate and cause memory leaks

2. **Prevention:**
   - Always call `remove()` when the ThreadLocal is no longer needed
   - Use try-finally blocks to ensure cleanup
   - Consider using framework support (e.g., Spring's TransactionSynchronizationManager)

   ```java
   public void processRequest() {
       threadLocal.set(value);
       try {
           // Process with thread-local value
       } finally {
           threadLocal.remove(); // Always clean up
       }
   }
   ```

**Best practices:**

1. **Always clean up:**
   - Call remove() when done with ThreadLocal
   - Use try-finally blocks to ensure cleanup

2. **Limit scope:**
   - Keep ThreadLocal usage within a defined scope
   - Don't spread ThreadLocal access throughout the codebase

3. **Document usage:**
   - Clearly document ThreadLocal variables
   - Specify who is responsible for setting and clearing

4. **Consider alternatives:**
   - Explicit parameter passing
   - Context objects
   - Dependency injection

5. **Use ThreadLocal judiciously:**
   - Only when thread isolation is truly needed
   - Not as a replacement for proper design
```