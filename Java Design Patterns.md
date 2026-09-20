# Java Design Patterns — Quick Recall Notes

## 1. Design Patterns Basics ✅

**What it is:** Reusable approaches for solving common software-design problems.

### What is a Design Pattern?

A **proven design approach** that helps us solve a commonly occurring software-design problem.

### Why do we need Design Patterns?

They help us:

* Avoid repeatedly solving the same design problems.
* Keep code more organized and maintainable.
* Make common design solutions easier to understand.

### Basic Categories

| Category       | Main Purpose                     |
| -------------- | -------------------------------- |
| **Creational** | Object creation                  |
| **Structural** | How classes/objects are combined |
| **Behavioral** | How objects communicate          |

### Our Learning Approach

* One pattern at a time
* Practical implementation
* Backend examples
* 3-YOE interview focus
* Understand → Implement → Interview → Hands-on

---

# 2. Singleton Design Pattern ✅

**What it is:** Ensures that only **one instance** of a class is created and provides access to that same instance.

### Core Concepts

**Why Singleton?**

When we want only **one shared object** of a class throughout the application.

**Private Constructor**

Prevents other classes from creating objects using `new`.

```java
private Logger() {}
```

**Static Instance**

Stores the single instance of the class.

```java
private static Logger instance;
```

**Static `getInstance()`**

Provides access to the same instance.

```java
public static Logger getInstance() {
    return instance;
}
```

**Same Object Every Time**

```java
Logger a = Logger.getInstance();
Logger b = Logger.getInstance();

System.out.println(a == b); // true
```

### Basic Singleton

```java
class Logger {

    private static Logger instance;

    private Logger() {}

    public static Logger getInstance() {

        if (instance == null) {
            instance = new Logger();
        }

        return instance;
    }
}
```

---

## Singleton Implementations

### 1. Eager Initialization

**What it is:** Object is created when the class is loaded.

```java
private static Logger instance = new Logger();
```

**Advantage:** Simple and thread-safe.

**Disadvantage:** Object is created even if it is never used.

---

### 2. Lazy Initialization

**What it is:** Object is created only when `getInstance()` is called for the first time.

```java
if (instance == null) {
    instance = new Logger();
}
```

**Problem:** Basic lazy Singleton is not thread-safe.

Two threads could both see:

```text
instance == null
```

and both create an object.

---

## `synchronized`

**What it is:** Ensures only one thread can execute the critical section at a time.

```java
public static synchronized Logger getInstance() {

    if (instance == null) {
        instance = new Logger();
    }

    return instance;
}
```

This makes the Singleton thread-safe.

**Problem:** Every call to `getInstance()` requires synchronization, even after the object already exists.

---

## Synchronizing Only the Critical Section

Instead of synchronizing the complete method:

```java
synchronized (Logger.class) {
    // object creation
}
```

Only the object-creation part is synchronized.

This avoids unnecessary synchronization after the Singleton has already been created.

---

# Double-Checked Locking

**What it is:** A Singleton technique that checks `instance == null` twice to achieve thread-safe lazy initialization while reducing synchronization overhead.

```java
private static volatile Logger instance;

public static Logger getInstance() {

    if (instance == null) {

        synchronized (Logger.class) {

            if (instance == null) {
                instance = new Logger();
            }
        }
    }

    return instance;
}
```

### Why two `null` checks?

**First check:**

```java
if (instance == null)
```

Avoids synchronization when the object already exists.

**Second check:**

```java
if (instance == null)
```

Ensures another thread didn't create the object while the current thread was waiting for the lock.

### Flow

```text
Thread checks instance
        ↓
   Already exists?
    /          \
  YES           NO
   ↓             ↓
return       synchronized
               ↓
         check again
               ↓
          create object
```

---

# `volatile`

**What it is:** Ensures that changes to the Singleton reference are visible across threads and prevents unsafe instruction reordering during initialization.

```java
private static volatile Logger instance;
```

### Important

`volatile` **does not create thread safety by itself**.

```text
volatile     → visibility + safe initialization ordering
synchronized → mutual exclusion
```

Both solve different problems in Double-Checked Locking.

---

# Bill Pugh Singleton

**What it is:** Uses a static inner `Holder` class to achieve lazy and thread-safe Singleton creation without explicit synchronization.

```java
class Logger {

    private Logger() {}

    private static class Holder {
        private static final Logger INSTANCE = new Logger();
    }

    public static Logger getInstance() {
        return Holder.INSTANCE;
    }
}
```

### Key idea

The `Holder` class is loaded only when `getInstance()` accesses it.

```text
Logger loaded
     ↓
Holder not loaded yet
     ↓
getInstance()
     ↓
Holder loaded
     ↓
INSTANCE created
```

---

# Singleton + Spring

**What it is:** Spring manages beans through its IoC container, and beans are Singleton-scoped by default.

Normally in Java:

```java
Logger logger = new Logger();
```

With Spring:

```java
@Service
class NotificationService {
}
```

Spring creates and manages the bean.

### Important distinction

```text
Singleton Pattern
→ We explicitly design a class to control its instance.

Spring Singleton Scope
→ Spring container manages one bean instance per application context.
```

---

# Singleton — Quick Interview Recall

**Q: Why private constructor?**

To prevent other classes from directly creating objects using `new`.

**Q: Why static instance?**

Because the instance needs to be accessed without creating the class object first.

**Q: Why `synchronized`?**

To prevent multiple threads from creating multiple instances simultaneously.

**Q: Why synchronize only the critical section?**

To avoid unnecessary synchronization after the Singleton has already been created.

**Q: Why `volatile`?**

To provide visibility between threads and prevent unsafe instruction reordering during initialization.

**Q: Singleton vs normal class?**

```text
Normal class → multiple objects can be created.

Singleton    → only one controlled instance.
```

**Q: Singleton Pattern vs Spring Singleton?**

```text
Singleton Pattern → design pattern implemented by us.

Spring Singleton → bean scope managed by Spring.
```

---

# 3. Factory Design Pattern ✅

**What it is:** Centralizes object creation and returns the required implementation based on a type or condition.

## Basic Factory Concept

### Why Factory?

Suppose we have:

```text
Notification
   ├── EmailNotification
   ├── SmsNotification
   └── PushNotification
```

Without a Factory, the client may need to write:

```java
if (type.equals("EMAIL")) {
    notification = new EmailNotification();
}
```

and similar creation logic in multiple places.

Factory moves this creation logic into one place.

---

## Common Interface

```java
interface Notification {
    void send();
}
```

Concrete implementations:

```java
class EmailNotification implements Notification {

    public void send() {
        System.out.println("Sending Email");
    }
}

class SmsNotification implements Notification {

    public void send() {
        System.out.println("Sending SMS");
    }
}
```

---

## Factory Class

```java
class NotificationFactory {

    public static Notification create(String type) {

        if (type.equals("EMAIL")) {
            return new EmailNotification();
        }

        if (type.equals("SMS")) {
            return new SmsNotification();
        }

        throw new IllegalArgumentException("Invalid notification type");
    }
}
```

### Client

```java
Notification notification =
        NotificationFactory.create("EMAIL");

notification.send();
```

The client doesn't directly do:

```java
new EmailNotification();
```

Instead, it asks the Factory for the required object.

---

# Factory Object Creation Flow

```text
Client
  ↓
NotificationFactory
  ↓
checks type
  ↓
"EMAIL"
  ↓
new EmailNotification()
  ↓
Notification object
```

---

# Factory + Polymorphism

**What it is:** The Factory returns a common interface reference while the actual object is a concrete implementation.

```java
Payment payment =
        PaymentFactory.create("UPI");
```

Here:

```text
Reference type → Payment

Actual object  → UpiPayment
```

If the Factory internally does:

```java
return new UpiPayment();
```

then:

```java
payment.pay();
```

executes the `UpiPayment` implementation.

This is where **polymorphism** is used.

---

# Factory vs `if/else`

**Important:** Factory does NOT mean we must completely remove `if/else`.

We can still use:

```java
if (type.equals("EMAIL")) {
    return new EmailNotification();
}
```

inside the Factory.

The main benefit is:

> **Object-creation logic is centralized instead of being spread across the client code.**

### Without Factory

```text
Client 1 → new EmailNotification()
Client 2 → new EmailNotification()
Client 3 → new SmsNotification()
```

Creation logic can be spread everywhere.

### With Factory

```text
Client
   ↓
Factory
   ↓
Creates required object
```

---

# Practical Backend Examples

### Notification Factory

```text
EMAIL    → EmailNotification
SMS      → SmsNotification
WHATSAPP → WhatsAppNotification
```

### Payment Factory

```text
UPI  → UpiPayment
CARD → CardPayment
CASH → CashPayment
```

The common idea is:

```text
Multiple implementations
        ↓
     Factory
        ↓
Required implementation
```

---

# Cleaner Factory Implementation

For many types, instead of many `if/else` statements, we can use a `Map` with `Supplier`.

Example:

```java
Map<String, Supplier<Notification>> notifications =
        Map.of(
            "EMAIL", EmailNotification::new,
            "SMS", SmsNotification::new
        );
```

Here:

```java
EmailNotification::new
```

is a method reference to the constructor.

### Important for our level

You **don't need to memorize this version**.

First understand the basic Factory using `if/else`.

---

# Factory + Spring

**What it is:** Spring's IoC container manages object creation and dependency management for Spring beans.

Example:

```java
@Service
class EmailNotification {
}
```

Spring creates and manages this object as a bean.

### Difference

```text
Our Factory
→ We create a Factory
→ Factory decides which object to create

Spring IoC
→ Spring container creates and manages beans
→ Spring handles dependency management
```

Don't simply say:

> "Spring is a Factory Pattern."

Instead, say:

> **"Spring's IoC container manages object creation and dependencies. It uses various internal mechanisms to do this."**

---

# Factory — Quick Interview Recall

**Q: What is Factory Pattern?**

> Factory Pattern is a creational design pattern that centralizes object creation and returns the required implementation based on some type or condition.

**Q: Why use Factory?**

> When we have multiple implementations and want to centralize their object creation instead of creating concrete objects directly throughout the application.

**Q: Factory vs Singleton?**

```text
Singleton → Controls the number of instances.

Factory   → Controls/centralizes which object gets created.
```

**Q: How does Factory work with interfaces?**

```text
Interface
   ↓
Multiple implementations
   ↓
Factory selects implementation
   ↓
Returns interface reference
```

**Q: What is the actual object here?**

```java
Payment payment = PaymentFactory.create("UPI");
```

If Factory returns:

```java
new UpiPayment();
```

then:

```text
Reference → Payment
Object    → UpiPayment
```

---

# Factory — Hands-on Recall

```java
interface Notification {
    void send();
}

class EmailNotification implements Notification {

    public void send() {
        System.out.println("Email");
    }
}

class SmsNotification implements Notification {

    public void send() {
        System.out.println("SMS");
    }
}
```

Factory:

```java
class NotificationFactory {

    public static Notification create(String type) {

        if (type.equals("EMAIL")) {
            return new EmailNotification();
        }

        if (type.equals("SMS")) {
            return new SmsNotification();
        }

        throw new IllegalArgumentException(
                "Invalid notification type"
        );
    }
}
```

Usage:

```java
Notification notification =
        NotificationFactory.create("EMAIL");

notification.send();
```

Output:

```text
Email
```

---

# 🧠 30-Second Quick Recall

```text
DESIGN PATTERNS
│
├── Singleton
│   └── One object
│       ├── private constructor
│       ├── static instance
│       ├── getInstance()
│       ├── lazy/eager
│       ├── synchronized
│       ├── double-checked locking
│       ├── volatile
│       ├── Bill Pugh
│       └── Spring Singleton scope
│
└── Factory
    └── Which object?
        ├── common interface
        ├── multiple implementations
        ├── centralize creation
        ├── Factory class
        ├── create(type)
        ├── polymorphism
        ├── Factory vs if/else
        ├── backend examples
        └── Spring IoC connection
```

---

# 📌 Current Progress

```text
Java Design Patterns
│
├── 1. Basics                  ✅
├── 2. Singleton               ✅ COMPLETE
├── 3. Factory                 ✅ COMPLETE
└── 4. Builder                 ⏳ NEXT
```

### Patterns completely covered so far

**Singleton ✅ → Factory ✅**

### Next

**Builder Pattern**

**What it is:** A creational pattern used to build complex objects step-by-step, especially when an object has many optional fields.
