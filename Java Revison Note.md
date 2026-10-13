# Java Core — Topic 1: Classes, Objects & OOP Interview Fundamentals

## 1. Classes and access modifiers

### Concept

A class defines the structure and behavior of objects. Interviewers usually test the restrictions around declaring classes rather than asking what a class is.

### Rules to remember

| Declaration | Top-level class | Nested class |
|---|---|---|
| `public` | Yes | Yes |
| Package-private (no modifier) | Yes | Yes |
| `private` | No | Yes |
| `protected` | No | Yes |
| `static` | No | Yes, for a static nested class |
| `final` | Yes | Yes |
| `abstract` | Yes | Yes |

A top-level class is declared directly inside a package. A nested class is declared inside another class.

```java
public class PaymentService {

    private class PaymentHelper {
    }

    static class PaymentConfig {
    }
}
```

- `PaymentHelper` is a non-static inner class.
- `PaymentConfig` is a static nested class.

### Why can't a top-level class be `private` or `protected`?

Because a top-level class has no enclosing class that owns it. `private` and `protected` are used to control access in contexts where an enclosing type or inheritance relationship applies.

**Memory hook:** Top-level class → only `public` or package-private.

### Static nested class vs inner class

| Static nested class | Non-static inner class |
|---|---|
| Does not require an enclosing object | Requires an enclosing object |
| Cannot directly access the enclosing instance's non-static members | Can access the enclosing instance's members |
| Created using `new Outer.Nested()` | Created using `outer.new Inner()` |

```java
class Outer {
    int value = 10;

    static class Nested {
        void show() {
            // Cannot directly access Outer.value
        }
    }

    class Inner {
        void show() {
            System.out.println(value);
        }
    }
}
```

### Tricky interview rules

- A class can be `final` — it cannot be subclassed.
- A class can be `abstract` — it cannot be instantiated directly.
- A class cannot be both `abstract` and `final`: abstract requires subclassing to become concrete, while final prohibits subclassing.
- `static` applies to member nested classes, not top-level classes.
- A local class declared inside a method has different restrictions from a member nested class.

**One-line answer:** A top-level class can be public or package-private; private and protected class declarations are allowed for member nested classes, not top-level classes.

---

## 2. Objects, references, `==` and `equals()`

### Concept

An object is an instance created from a class. A reference is a variable that can refer to an object.

```java
Payment p1 = new Payment();
Payment p2 = p1;
```

There is one object and two references pointing to it.

```text
p1 and p2

Two references

One Payment object

Both references point to the same object
```

### Important rules

- `new` creates an object.
- Assigning one reference to another does not create another object.
- Changing an object's state through one reference is visible through the other reference to that same object.
- A reference can be `null`, meaning it does not refer to an object.
- An object becomes eligible for garbage collection when it is no longer reachable through live references or other GC roots. Eligibility does not guarantee immediate collection.

### `==` vs `equals()`

| `==` | `equals()` |
|---|---|
| For object references, checks whether they refer to the same object | Checks equality according to the class's implementation |
| Cannot be customized | Can be overridden |
| For primitives, compares primitive values | Not applicable to primitives |

```java
String a = new String("Java");
String b = new String("Java");

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
```

The two objects are different, but their string contents are equal.

**Important:** The default implementation of `Object.equals()` behaves like reference equality. A class can override it to define logical equality.

### Common interviewer trap

**Q: Is `hashCode()` calculated using the key, and does `equals()` compare the value?**

Not as a universal rule. For hash-based collections, a key's `hashCode()` helps determine a bucket, and `equals()` helps distinguish keys that land in the same bucket. Whether equality compares fields, values, or identities depends on the class's implementation. This is an important connection to Collections, which we'll handle separately.

**One-line answer:** A reference points to an object; `==` checks reference identity for objects, while `equals()` checks the equality defined by the class.

---

## 3. The four OOP pillars

### A. Encapsulation

**Concept:** Keep data and the operations that control it together, while restricting direct access to internal state.

```java
class Account {
    private double balance;

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
}
```

**Why useful?** The class can protect its state and enforce validation rules.

**Interview answer:** Encapsulation hides internal state and provides controlled access through methods.

### B. Inheritance — IS-A

A child class inherits accessible behavior and state from a parent class.

```java
class Payment {
    void process() {
        System.out.println("Processing payment");
    }
}

class UpiPayment extends Payment {
}
```

`UpiPayment IS-A Payment`, so it can inherit and use the accessible `process()` method.

**Rules:**

- Java classes support single class inheritance: a class cannot extend two classes.
- A class can implement multiple interfaces.
- Constructors are not inherited.
- Private members are not directly accessible in a subclass, although private superclass state still exists as part of the superclass portion of the object.
- A subclass may override eligible inherited instance methods.

### C. Composition — HAS-A

**Concept:** One class contains or uses an object of another class.

```java
class PaymentService {
    private final PaymentGateway gateway;

    PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

`PaymentService HAS-A PaymentGateway`.

Composition is useful when a class needs another component to do its work without becoming a specialized version of that component.

| Inheritance | Composition |
|---|---|
| IS-A relationship | HAS-A relationship |
| `UpiPayment extends Payment` | `PaymentService` uses `PaymentGateway` |
| Reuses inherited behavior | Reuses behavior through a contained/dependent object |
| Can create tighter coupling to a parent design | Often allows dependencies to be replaced more easily |

**Backend connection:** A service using a repository or gateway is typically composition/dependency usage, not inheritance.

### D. Abstraction

**Concept:** Expose what a component can do while hiding the implementation details.

For example, callers use `paymentService.pay()` without needing to know every internal step used to process the payment.

Abstraction can be achieved using abstract classes and interfaces.

### Abstraction vs encapsulation

- **Abstraction:** Hides implementation complexity; focuses on what an operation does.
- **Encapsulation:** Controls access to internal state and implementation; focuses on protecting how the state is managed.

**One-line OOP recall:** Encapsulation protects state, inheritance reuses and specializes behavior, polymorphism allows different implementations through a common type, and abstraction hides implementation details.

---

## 4. Abstract class constructor — critical distinction

This is the clarification you specifically asked me to preserve.

### Rule 1: Can an abstract class have a constructor?

Yes. An abstract class cannot be instantiated directly, but its constructor executes when a concrete subclass object is created.

```java
abstract class Payment {
    Payment() {
        System.out.println("Payment constructor");
    }

    void pay() {
        System.out.println("Payment pay");
    }
}

class UpiPayment extends Payment {
    UpiPayment() {
        System.out.println("UPI constructor");
    }

    @Override
    void pay() {
        System.out.println("UPI pay");
    }
}
```

Execution:

```java
Payment p = new UpiPayment();
p.pay();
```

Output:

```text
Payment constructor
UPI constructor
UPI pay
```

### What exactly happens?

1. `new UpiPayment()` starts creating a `UpiPayment` object.
2. The superclass constructor, `Payment()`, runs first to initialize the superclass portion.
3. The subclass constructor, `UpiPayment()`, runs next.
4. `p.pay()` is then called. Runtime dispatch selects `UpiPayment.pay()` because the actual object is a `UpiPayment`.

### The distinction you must never mix up

| Constructor execution | Method overriding |
|---|---|
| Initializes the object | Selects an eligible overridden instance method when called |
| Superclass constructor runs before subclass constructor | Actual object type determines runtime dispatch |
| Constructors are not inherited or overridden | Eligible instance methods can be inherited and overridden |
| Happens as part of object construction | Happens when the method is invoked |

**Very important:** Inheritance does not mean that only overridden methods execute. Inherited methods can also execute normally. Only an invocation of an eligible overridden instance method uses runtime dispatch to select the overriding implementation.

**One-line interview answer:**

> An abstract class cannot be instantiated directly, but its constructor runs when a concrete subclass object is created; separately, overridden instance methods execute according to the actual object's type at runtime.

---

## 5. Polymorphism: compile-time vs runtime

### Concept

Polymorphism means the same method name or common reference can work with different inputs or implementations.

### Compile-time polymorphism — overloading

The compiler selects an overloaded method based on the method arguments and applicable overload-resolution rules.

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
}
```

### Runtime polymorphism — overriding

The actual object's type determines which overridden instance method implementation runs.

```java
Payment p = new UpiPayment();
p.pay(); // UpiPayment.pay()
```

Here:

- Reference type: `Payment`
- Actual object type: `UpiPayment`
- Selected overridden method: `UpiPayment.pay()`

The reference type controls which members are accessible at compile time. The actual object type controls runtime dispatch for overridden instance methods.

### Memory hook

- **Overloading** → compiler chooses among signatures.
- **Overriding** → runtime dispatch chooses the implementation.

---

## 6. Method overloading — tricky rules

### Concept

Methods share a name but have different parameter lists.

```java
void pay(int amount) { }

void pay(double amount) { }

void pay(int amount, String mode) { }
```

### Rules

- Changing the number of parameters can create an overload.
- Changing parameter types can create an overload.
- Changing parameter order can create an overload if the resulting parameter types differ by position.
- Changing only parameter names does not create an overload.
- Changing only the return type does not create an overload.
- Overloading is resolved at compile time.

### Tricky variations

**1. Can return type alone overload a method?**

No.

```java
int getAmount() {
    return 10;
}

// Invalid overload: same parameter list
// double getAmount() { return 10.0; }
```

**2. Widening vs boxing**

When choosing between applicable overloads, Java generally prefers widening primitive conversion over boxing.

```java
void show(long x) { }
void show(Integer x) { }

show(10); // show(long)
```

The `int` argument widens to `long`; this is preferred over boxing to `Integer` in this case.

**3. What if `null` is passed?**

```java
void show(Object x) { }
void show(String x) { }

show(null); // show(String)
```

`String` is more specific than `Object`, so the `String` overload is selected.

But:

```java
void show(String x) { }
void show(Integer x) { }

// show(null); // Ambiguous
```

Neither `String` nor `Integer` is more specific than the other, so the call is ambiguous.

**One-line interview answer:**

> Overloading means the same method name with different parameter lists and compile-time selection; overriding means redefining an inherited instance method in a subclass, with runtime dispatch based on the actual object.

---

## 7. Method overriding — tricky rules

### Concept

A subclass provides a new implementation of an eligible inherited instance method with a compatible signature.

```java
class Payment {
    void pay() {
        System.out.println("Generic payment");
    }
}

class UpiPayment extends Payment {
    @Override
    void pay() {
        System.out.println("UPI payment");
    }
}
```

### Rules to remember

- The overriding method cannot reduce visibility.
- The return type must be compatible; a covariant reference return type is allowed.
- A `final` method cannot be overridden.
- A `private` method is not overridden because it is not inherited as an overridable method.
- Static methods are hidden, not overridden.
- Constructors cannot be overridden.
- Constructors are not inherited.
- `@Override` helps the compiler verify that the method actually overrides an inherited method.

### Common traps

**1. Can a `public` method be overridden as `protected`?**

No. That reduces visibility. The overriding method must remain `public`.

**2. Can static methods be overridden?**

No. A matching static method in the subclass hides the parent's static method.

**3. Can a private method be overridden?**

No. A same-signature method in the subclass is a separate method, not an override of the parent's private method.

**4. Can a final method be overridden?**

No.

**5. Can a constructor be overridden or inherited?**

Neither. A subclass constructor can invoke a superclass constructor, but that is not inheritance or overriding of the constructor.

**6. Can the return type change?**

A compatible covariant reference return type is allowed when overriding. An unrelated return type is not allowed.

### Constructor trap: parent method called inside a constructor

During construction, an overridden instance method can be dispatched to the subclass implementation even before the subclass constructor has finished. Calling overridable methods from constructors is therefore risky: the subclass's fields might not yet be initialized.

---

## 8. Quick comparison table

| Question | Correct recall |
|---|---|
| Top-level class can be `private`? | No |
| Nested class can be `private`? | Yes, for a member nested class |
| Top-level class can be `static`? | No |
| Class can be `abstract` and `final` together? | No |
| Abstract class can have a constructor? | Yes |
| Abstract class can be instantiated directly? | No |
| Constructor inherited? | No |
| Constructor overridden? | No |
| Static method overridden? | No — hidden |
| Private method overridden? | No |
| Final method overridden? | No |
| Return type alone overloads a method? | No |
| `==` vs `equals()`? | Reference identity vs class-defined equality for objects |
| Can a class extend two classes? | No |
| Can a class implement multiple interfaces? | Yes |
| Can an overridden method reduce visibility? | No |
| Can a superclass constructor run when creating a subclass object? | Yes, before the subclass constructor |

---

## 9. Final interview recall

### Five answers to practise saying naturally

1. **What is polymorphism?** The same method name or common type can represent different behavior; overloading is resolved at compile time, while overriding uses runtime dispatch.
2. **What is the difference between inheritance and composition?** Inheritance represents an IS-A relationship; composition represents a HAS-A relationship.
3. **Why can an abstract class have a constructor?** Its constructor initializes the superclass state when a concrete subclass object is created, even though the abstract class itself cannot be instantiated directly.
4. **Why does `Payment p = new UpiPayment()` call `UpiPayment.pay()`?** The reference type is `Payment`, but the actual object is `UpiPayment`, so runtime dispatch selects the overridden implementation.
5. **What is the difference between overloading and overriding?** Overloading means the same method name with different parameter lists and compile-time selection; overriding means redefining an inherited instance method in a subclass, with runtime dispatch based on the actual object.

---

## 10. Topic 1 tracker

- [ ] Classes and access modifiers
- [ ] Objects, references, `==` and `equals()`
- [ ] Four OOP pillars
- [ ] Abstract class constructor
- [ ] Compile-time vs runtime polymorphism
- [ ] Method overloading
- [ ] Method overriding
- [ ] Quick comparison questions
- [ ] Final interview recall practice


# Java Core — Topic 2 Quick Recall

# Abstract Class vs Interface

*Interview revision · Core rules · Tricky variations · Code examples*

## 1. Abstract Class

An abstract class is a base class that can provide shared state and implementation while leaving some methods for subclasses to implement.

```java
abstract class Payment {
    protected double amount;

    Payment(double amount) {
        this.amount = amount;
    }

    void printAmount() {
        System.out.println(amount);
    }

    abstract void pay();
}
```

**Remember:**

- Cannot instantiate an abstract class directly.
- Can have constructors, instance fields, static fields/methods, and concrete methods.
- Can have `final` concrete methods, but they cannot be overridden.
- An abstract method cannot be `private`, `static`, or `final`.
- Its constructor executes when a concrete subclass object is created.

## 2. Interface

An interface defines a contract or capability that implementing classes agree to provide.

```java
interface PaymentMethod {
    int MAX_RETRIES = 3;  // public static final
    void pay();           // public abstract

    default void printStatus() {
        System.out.println("Payment initiated");
    }

    static void showRules() {
        System.out.println("Payment rules");
    }

    private void log() {  // Java 9+
        System.out.println("Internal helper");
    }
}
```

**Remember:**

- No constructor or per-object instance fields.
- Fields are implicitly `public static final`.
- Ordinary abstract methods are implicitly `public abstract`.
- `default` methods provide overridable instance behavior.
- `static` methods belong to the interface.
- `private` helper methods are allowed since Java 9.
- A class can implement multiple interfaces but extend only one class.
- An interface can extend multiple interfaces, but cannot extend a class.

## 3. Key Differences

| Abstract Class | Interface |
|---|---|
| Can hold instance state | No per-object instance fields |
| Can have constructors | Cannot have constructors |
| Can provide concrete and abstract methods | Supports abstract, `default`, `static`, and private helper methods |
| A class extends one class | A class can implement multiple interfaces |
| Useful for shared base behavior | Useful for contracts and capabilities |

**Interview answer:** An abstract class provides a common base with shared state and implementation; an interface defines a contract that different classes can implement.

<div style="border: 2px solid #D97706; border-radius: 10px; padding: 16px; margin: 16px 0;">

## 4. Abstract Class — Tricky Interview Box

### Must-Know Variations

- Can an abstract class have a static variable or method? **Yes.**
- Can it have a `final` concrete method? **Yes.**
- Can a method be both `abstract` and `final`? **No.**
- Can a subclass override a final method? **No.** It can inherit and call it.
- Can an abstract class have a constructor? **Yes.** It runs as part of subclass construction.

```java
abstract class Payment {
    static int count = 0;

    final void process() {
        System.out.println("Processing");
    }

    abstract void pay();
}

class UpiPayment extends Payment {
    @Override
    void pay() {
        System.out.println("UPI");
    }

    // void process() {} // Compilation error
}
```

**One-line answer:** An abstract class can have constructors, fields, static members, concrete and abstract methods, and final concrete methods; incompatible combinations such as an abstract `final` method are prohibited.

</div>

<div style="border: 2px solid #2563EB; border-radius: 10px; padding: 16px; margin: 16px 0;">

## 5. Interface — Tricky Interview Box

### Must-Know Variations

- Can an interface have variables? **Yes**, implicitly `public static final`.
- Can it have a constructor? **No.**
- Can it have `static` methods? **Yes**, called using the interface name.
- Can it have concrete instance methods? **Yes**, using `default`.
- Can it have private or private static methods? **Yes**, since Java 9.
- Can an interface method be `final`? **No.**
- Can an implementing class change an interface constant? **No.**
- Can an interface extend multiple interfaces? **Yes.**
- Can an abstract class implement an interface without implementing every method? **Yes**, if it remains abstract.
- Must an implementation of an interface's abstract method be `public`? **Yes.**
- Can a class override an interface's static method? **No.** It is called using the interface name.
- Is `default` an access modifier? **No.** It marks an interface instance method with a default implementation.

**One-line answer:** An interface defines a contract and supports constants, abstract methods, default methods, static methods, and private helper methods, but has no constructor or per-object instance fields.

</div>

## 6. Special Scenario — Abstract Class vs Interface

### When Should You Choose Each?

**Abstract class:** Related classes need shared state, constructor initialization, or common implementation.

```java
abstract class Payment {
    protected double amount;

    Payment(double amount) {
        this.amount = amount;
    }

    void printAmount() {
        System.out.println(amount);
    }

    abstract void pay();
}
```

**Interface:** Different classes need to share a capability, even if they have different parent classes.

```java
interface Refundable {
    void refund();
}

class CardPayment implements Refundable {
    public void refund() {
        System.out.println("Card refunded");
    }
}
```

**Interview answer:** An abstract class provides a common base with shared state and implementation; an interface defines a contract that different classes can implement.

## 7. Special Scenario — Conflicting `default` Methods

**What happens when two interfaces provide a `default` method with the same signature?**

```java
interface A {
    default void show() {
        System.out.println("A");
    }
}

interface B {
    default void show() {
        System.out.println("B");
    }
}

class Demo implements A, B {
    // Compilation error until show() is resolved
}
```

**Why?** Java cannot decide which implementation to use.

**Fix:**

```java
class Demo implements A, B {
    @Override
    public void show() {
        A.super.show();
    }
}
```

You can instead choose `B.super.show()` or provide your own implementation.

**Interview answer:** When two interfaces provide conflicting default methods with the same signature, the implementing class must override the method to resolve the conflict.

<div style="border: 2px solid #16A34A; border-radius: 10px; padding: 16px; margin: 16px 0;">

## 8. Special Scenario — Postponing Interface Implementation

### Compilation Trap: Missing `refund()`

```java
interface PaymentMethod {
    void pay();
    void refund();
}

class CardPayment implements PaymentMethod {
    public void pay() {
        System.out.println("Paying by card");
    }

    // refund() is missing
}
```

**Why does compilation fail?**

`CardPayment` is concrete, but `refund()` remains unimplemented.

**Two fixes:**

1. Implement `refund()` in `CardPayment`.
2. Declare `CardPayment` as `abstract`.

**Memory trick:** Abstract class = implementation can be postponed. Concrete class = implementation must be completed.

The same rule applies when an abstract class implements an interface:

```java
interface PaymentMethod {
    void pay();
    void refund();
}

abstract class Payment implements PaymentMethod {
    public void pay() {
        System.out.println("Payment initiated");
    }

    // refund() is postponed
}

class CardPayment extends Payment {
    public void refund() {
        System.out.println("Refund completed");
    }
}
```

`Payment` postpones `refund()`, while concrete `CardPayment` completes it.

</div>

## 9. Constructor Execution — Quick Recall

An abstract class constructor does not create an abstract-class object independently. It initializes the superclass portion of a subclass object.

```java
abstract class Payment {
    Payment() {
        System.out.println("A");
    }
}

class UpiPayment extends Payment {
    UpiPayment() {
        System.out.println("B");
    }
}

class FastUpiPayment extends UpiPayment {
    FastUpiPayment() {
        System.out.println("C");
    }
}

// new FastUpiPayment(); prints A, B, C
```

**Memory trick:** Parent → Child → Grandchild.

## 10. Final Rapid-Recall Checklist

Before an interview, make sure you can answer these without looking:

- [ ] Why can an abstract class have a constructor but an interface cannot?
- [ ] Why can't an abstract method be `final`?
- [ ] Why must interface methods be implemented as `public`?
- [ ] Why does a concrete class fail to compile if an abstract method is missing?
- [ ] How do you resolve conflicting interface `default` methods?
- [ ] When would you choose an abstract class over an interface?
- [ ] Can a class extend multiple classes or implement multiple interfaces?
