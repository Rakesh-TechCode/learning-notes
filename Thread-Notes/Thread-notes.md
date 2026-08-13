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

---

## Topics Covered So Far

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

### Next Topics

* `notify()`
* `notifyAll()`
* Complete `wait()` / `notify()` example
* `synchronized` + inter-thread communication
* **Multiprocessing**
* **Multithreading vs Multiprocessing**
* More practical thread examples
* Thread pools / `ExecutorService`
