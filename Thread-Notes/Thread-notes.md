# Java Threads — Notes So Far

## 1. What is a Thread

* A **thread** is a path of execution inside a program.
* It performs a particular task.
* Every Java application starts with the **main thread**.
* Multiple threads allow multiple tasks to progress concurrently.

## 2. Process vs Thread

* **Process** = running program/application.
* **Thread** = smaller execution unit inside a process.
* One process can contain multiple threads.
* Threads within the same process can share resources.

## 3. Why Do We Need Threads?

* To perform multiple tasks concurrently.
* Example: one thread downloads data while another processes something else.
* Without multiple threads, tasks may need to execute sequentially.
* Threads can improve responsiveness and resource utilization.

## 4. Main Thread

* Java starts execution through the **main thread**.
* It begins from:

```java
public static void main(String[] args)
```

* The main thread can create and start other threads.
* The main thread itself is also a thread.

## 5. Creating a Thread — `extends Thread`

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Task");
    }
}
```

* The class itself becomes a Thread.
* Create object and call `start()`.
* Java allows extending only **one class**.

## 6. Creating a Thread — `implements Runnable`

```java
class MyTask implements Runnable {
    public void run() {
        System.out.println("Task");
    }
}
```

* `Runnable` represents the **task**, not the thread.
* A Thread is needed to execute that task:

```java
Thread t = new Thread(new MyTask());
t.start();
```

* Allows your class to extend another class.
* Separates **task** from **thread**.

## 7. Thread vs Runnable

* `Thread` = thread + task together.
* `Runnable` = task only.
* `Runnable` is generally preferred for separating responsibilities.
* `Runnable` also avoids Java's single-class-inheritance limitation.
* Same Runnable task can be given to multiple Thread objects.

## 8. `start()`

* `start()` starts a **new thread**.
* The new thread eventually executes `run()`.
* Calling `start()` does **not directly execute `run()` on the current thread**.

```java
t.start();
```

## 9. `run()`

* `run()` contains the task that the thread should perform.
* Calling `run()` directly does **not create a new thread**.

```java
t.run(); // normal method call
```

* The current thread executes `run()` directly.

## 10. Multiple Threads

* Multiple threads can execute within the same program.
* Example:

```java
t1.start();
t2.start();
```

* Their execution order isn't guaranteed.
* The OS/JVM scheduling system determines when threads get CPU time.

## 11. Thread Scheduling

* The CPU cannot necessarily execute every thread simultaneously on one core.
* The OS scheduler decides which runnable thread gets CPU time.
* Threads can be switched between rapidly.
* Therefore, output order is often unpredictable.
* **Priority doesn't guarantee execution order.**

## 12. Thread Lifecycle

Main states we learned:

* **NEW** — thread object created, not started.
* **RUNNABLE** — ready to run / may be running when scheduled.
* **TERMINATED** — execution completed.
* A terminated thread **cannot be started again**.

## 13. `sleep()`

* Makes the current thread pause for a specified time.

```java
Thread.sleep(5000);
```

* Thread enters **TIMED_WAITING**.
* After the time expires, it becomes eligible to run again.
* `sleep()` does **not release an object's monitor/lock**.

## 14. `join()`

* Makes the current thread wait for another thread to finish.

```java
t1.start();
t1.join();
```

* If `main` calls `t1.join()`, **main waits for `t1`**.
* After `t1` finishes, main continues.
* Useful when one task must finish before continuing.

## 15. `interrupt()`

* Used to request that a thread stop waiting/sleeping or respond to interruption.
* If a thread is sleeping/waiting, interruption can cause `InterruptedException`.

```java
t1.interrupt();
```

* It doesn't forcibly kill the thread.
* The interrupted thread should respond appropriately.

## 16. Thread Priority

* Threads can have different priority values.
* Higher priority may influence scheduling.
* **Higher priority does NOT guarantee that thread executes first.**
* Scheduling ultimately depends on the JVM/OS.

## 17. Race Condition

* Happens when multiple threads access **shared mutable data** concurrently.
* At least one thread modifies the data.
* Timing/order of execution can affect the result.
* Example:

```java
count++;
```

* Two threads can interfere with each other's updates.
* This can produce an incorrect/unexpected result.

## 18. Critical Section

* A **critical section** is the part of code that accesses shared data and needs protection.

```java
synchronized (this) {
    count++;
}
```

* Here, `count++` is the critical section.

## 19. `synchronized` Method

* Allows only one thread at a time to execute that synchronized method **for the same object's lock**.

```java
synchronized void increment() {
    count++;
}
```

* Protects the whole method.
* Helps prevent race conditions.

## 20. `synchronized` Block

* Protects only a specific section instead of the entire method.

```java
void increment() {
    System.out.println("Start");

    synchronized (this) {
        count++;
    }

    System.out.println("End");
}
```

* Useful when only a small part needs protection.
* The object inside `synchronized(...)` determines the lock used.

## 21. `synchronized(this)`

* `this` refers to the **current object**.
* `synchronized(this)` uses that object's intrinsic lock.
* A synchronized instance method also uses the current object's lock.
* Therefore, they can compete for the **same lock**.

## 22. `Lock` / `ReentrantLock`

* `ReentrantLock` provides explicit locking.

```java
lock.lock();

try {
    count++;
} finally {
    lock.unlock();
}
```

* Unlike `synchronized`, you explicitly acquire/release the lock.
* `finally` helps ensure the lock is released.
* Provides more control than basic `synchronized`.

## 23. Inter-Thread Communication

* Threads sometimes need to **coordinate with each other**.
* Java provides:

  * `wait()`
  * `notify()`
  * `notifyAll()`
* We have currently covered **`wait()`**.
* `notify()` and `notifyAll()` are next.

## 24. `wait()`

* `wait()` puts the current thread into **WAITING**.
* It **releases the object's monitor** while waiting.
* It must be called while the thread owns that object's monitor.

```java
synchronized (lock) {
    lock.wait();
}
```

* Another thread can later signal it using `notify()` / `notifyAll()`.

## 25. `wait()` vs `sleep()`

| `sleep()`                              | `wait()`                                         |
| -------------------------------------- | ------------------------------------------------ |
| Waits for a **specific time**          | Waits for **communication/signalling**           |
| Does **not release** the monitor       | **Releases** the object's monitor                |
| Used for timed pause                   | Used for thread coordination                     |
| Can be called without owning a monitor | Must be called while owning the object's monitor |

Absolutely. I cleaned up the formatting, fixed the broken comparison table, removed unnecessary empty code fences, and made it **GitHub-ready Markdown** while keeping your notes simple and powerful.

# Java Threads — Topics 26 to 46

## 26. `notify()`

* `notify()` wakes **one thread** that is waiting on the same object's monitor.
* The notified thread does **not immediately run**; it must first reacquire the monitor.
* `notify()` must be called while owning that object's monitor.
* Used for **inter-thread communication**.

```java
synchronized (lock) {
    dataReady = true;
    lock.notify();
}
```

---

## 27. `notifyAll()`

* `notifyAll()` wakes **all threads** waiting on the same object's monitor.
* They compete to reacquire the monitor.
* Only **one thread at a time** can reacquire that monitor.
* Useful when multiple waiting threads may need to re-check a condition.

```java
synchronized (lock) {
    dataReady = true;
    lock.notifyAll();
}
```

### `notify()` vs `notifyAll()`

| `notify()`                    | `notifyAll()`                               |
| ----------------------------- | ------------------------------------------- |
| Wakes **one** waiting thread  | Wakes **all** waiting threads               |
| Less overhead                 | More overhead                               |
| Use when one waiter is enough | Use when multiple waiters may need to react |

---

## 28. Complete `wait()` / `notify()` Flow

* Thread 1 acquires the monitor and checks a condition.
* If the condition isn't ready → `wait()`.
* `wait()` releases the monitor.
* Thread 2 acquires the monitor and performs the required work.
* Thread 2 calls `notify()`.
* Thread 1 wakes and **reacquires the monitor** before continuing.

```text
Thread 1 → wait() → releases monitor
Thread 2 → work → notify()
Thread 1 → reacquires monitor → continues
```

---

## 29. Why `wait()` Is Usually Used with `while`

* Always re-check the condition after waking.
* `notify()` means **"something changed"**, not necessarily **"your condition is true"**.
* A thread can also wake without the expected condition being ready.
* Therefore, use `while` instead of `if`.

```java
synchronized (lock) {
    while (!dataReady) {
        lock.wait();
    }

    processData();
}
```

---

## 30. Thread Safety

* **Thread-safe** means shared data behaves correctly when accessed by multiple threads.
* A thread-safe class prevents incorrect results caused by concurrent access.
* Common techniques:

  * `synchronized`
  * `Lock`
  * Atomic classes
  * Concurrent collections
  * Proper immutability

---

## 31. `volatile`

* `volatile` provides **visibility** of a variable's latest value between threads.
* It does **not make compound operations atomic**.
* `count++` is still **not thread-safe** with only `volatile`.

```java
volatile boolean running = true;
```

* Useful for simple shared flags/state.

---

## 32. Atomicity

* **Atomicity** means an operation happens as **one indivisible operation**.
* Another thread cannot interfere **in the middle of that atomic operation**.
* Atomicity and visibility are different concepts.

```text
Atomicity
→ Operation cannot be partially observed/interfered with

Visibility
→ Threads see the latest value
```

---

## 33. Atomic Classes

* Java provides atomic classes in `java.util.concurrent.atomic`.
* Common classes:

  * `AtomicInteger`
  * `AtomicLong`
  * `AtomicBoolean`
  * `AtomicReference`
* They provide **atomic operations on shared variables**.
* Multiple threads can access them concurrently.

```java
AtomicInteger count = new AtomicInteger(0);

count.incrementAndGet();
```

---

## 34. `volatile` vs `AtomicInteger`

| `volatile`                                | `AtomicInteger`                        |
| ----------------------------------------- | -------------------------------------- |
| Provides visibility                       | Provides atomic operations             |
| `count++` is not atomic                   | `incrementAndGet()` is atomic          |
| Good for simple flags/state               | Good for counters/simple shared values |
| Doesn't provide compound-operation safety | Provides specific atomic operations    |

---

## 35. Lock vs Atomicity

* **Lock** is a mechanism for controlling thread access.
* **Atomicity** is a property of an operation.
* `synchronized` allows only one thread at a time inside the protected critical section.
* `AtomicInteger` allows concurrent access while making supported operations atomic.

```text
Lock
→ Controls access to critical section

Atomicity
→ Operation happens indivisibly
```

---

# Executor Framework

## 36. Why Executor Framework?

* Creating a new thread for every task doesn't scale well.
* Too many threads can consume memory and CPU.
* Executor Framework separates **task submission from thread management**.
* It allows reusable worker threads through thread pools.

```text
Tasks
   ↓
Executor
   ↓
Worker Threads
```

---

## 37. Thread Pool

* A thread pool is a group of **reusable worker threads**.
* Tasks are submitted to the pool.
* If all workers are busy, additional tasks wait in a queue.
* When a worker finishes, it takes another task.

```text
3 Workers

T1 → Worker 1
T2 → Worker 2
T3 → Worker 3
T4 → Queue
```

---

## 38. `Executor`

* `Executor` is an **interface** in the Executor Framework.
* Its main method is:

```java
execute(Runnable task);
```

* It accepts a task and arranges for its execution.
* It does **not itself mean a thread pool**.
* Its implementation decides how the task is executed.

---

## 39. `ExecutorService`

* `ExecutorService` is an interface that **extends `Executor`**.
* Provides additional task and executor lifecycle management.
* Important methods:

  * `submit()`
  * `shutdown()`
  * `shutdownNow()`
* It is commonly used with thread-pool implementations.

```java
ExecutorService service =
    Executors.newFixedThreadPool(3);
```

---

## 40. `execute()`

* `execute()` comes from `Executor`.
* Submits a `Runnable` task for execution.
* It does **not return a `Future`**.

```java
service.execute(() -> {
    System.out.println("Task");
});
```

---

## 41. `submit()`

* `submit()` is provided by `ExecutorService`.
* Submits a task for execution.
* Returns a **`Future`** representing the submitted task.
* `submit()` does **not wait for the task to finish**.

```java
Future<?> future =
    service.submit(() -> {
        System.out.println("Task");
    });
```

---

## 42. `Future`

* `Future` is a **handle representing an asynchronous task**.
* It allows us to check status, get results, or cancel the task.
* Important methods:

  * `get()`
  * `isDone()`
  * `cancel()`
  * `isCancelled()`

```text
submit()
   ↓
 Future
   ↓
Represents the task
```

---

## 43. `Future.get()`

* `get()` obtains the task's result.
* If the task isn't finished, `get()` **blocks** the calling thread until completion.
* For a task with a result, it returns that result.

```java
Integer result = future.get();
```

```text
Task running
     ↓
   get()
     ↓
  waits
     ↓
Task finished
     ↓
  result
```

---

## 44. `Future.isDone()`

* Checks whether the task has completed.
* Returns `true` if completed.
* Returns `false` if still running/not completed.
* It **does not block**.

```java
if (future.isDone()) {
    System.out.println("Completed");
}
```

---

## 45. `Future.cancel()`

* Requests cancellation of the task.
* `cancel(true)` → attempts cancellation and requests interruption if running.
* `cancel(false)` → attempts cancellation without interrupting a running task.
* It does **not forcefully kill a thread**.

```java
future.cancel(true);
```

---

## 46. `Future.isCancelled()`

* Checks whether the task was successfully cancelled.
* Returns `true` if the task was cancelled.
* Returns `false` otherwise.

```java
future.isCancelled();
```

---

# Topics Covered So Far

1. What is a Thread
2. Process vs Thread
3. Why Do We Need Threads?
4. Main Thread
5. Creating Thread using `extends Thread`
6. Creating Thread using `implements Runnable`
7. Thread vs Runnable
8. `start()`
9. `run()`
10. Multiple Threads
11. Thread Scheduling
12. Thread Lifecycle
13. `sleep()`
14. `join()`
15. `interrupt()`
16. Thread Priority
17. Race Condition
18. Critical Section
19. `synchronized` Method
20. `synchronized` Block
21. `synchronized(this)`
22. `Lock` / `ReentrantLock`
23. Inter-Thread Communication
24. `wait()`
25. `wait()` vs `sleep()`
26. `notify()`
27. `notifyAll()`
28. Complete `wait()` / `notify()` Flow
29. `while` with `wait()`
30. Thread Safety
31. `volatile`
32. Atomicity
33. Atomic Classes
34. `volatile` vs `AtomicInteger`
35. Lock vs Atomicity
36. Executor Framework
37. Thread Pool
38. `Executor`
39. `ExecutorService`
40. `execute()`
41. `submit()`
42. `Future`
43. `Future.get()`
44. `Future.isDone()`
45. `Future.cancel()`
46. `Future.isCancelled()`

---

# Next Topics

* `Callable`
* `Future<T>` with return values
* `ExecutorService` practical examples
* Thread Pool types
* `shutdown()` vs `shutdownNow()`
* **Multiprocessing**
* **Multithreading vs Multiprocessing**
* `CompletableFuture`
* Concurrent Collections
* Deadlock
* More real-world threading patterns
