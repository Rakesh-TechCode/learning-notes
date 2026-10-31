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


# Java Core — Topic 3: Constructors & Object Initialization

## 1. Constructor vs Normal Method

- Constructor name must match the class name.
- Constructor has **no return type**, not even `void`.
- A normal method can have a return type.
- Java provides a default no-argument constructor only when no constructor is declared.

```java
class Payment {
    Payment() {}       // Constructor
    void Payment() {}  // Normal method
}
```

## 2. Types of Constructors

- **No-argument constructor:** Takes zero parameters.
- **Parameterized constructor:** Takes parameters.
- **Default constructor:** Compiler-generated no-argument constructor when no constructor is declared.
- If any constructor is declared, Java does not automatically generate a no-argument constructor.

```java
class Payment {
    Payment(String method) {}
    // new Payment() causes a compilation error
}
```

## 3. Constructor Overloading

- Same class, multiple constructors with different parameter lists.
- Parameter names alone cannot distinguish constructors.
- Constructors cannot have return types.

```java
class Payment {
    Payment() {}
    Payment(String method) {}
    Payment(int amount) {}
}
```

## 4. Constructor Chaining

- `this()` → invokes another constructor in the same class.
- `super()` → invokes a superclass constructor.
- Constructor calls must be the first statement.
- You cannot use both `this()` and `super()` as separate calls in the same constructor.
- If neither is explicitly called, Java inserts `super()` when applicable.

**One-line answer:** `super()` invokes a superclass constructor, initializing the parent portion before the subclass constructor body executes.

## 5. Parent and Subclass Constructors — Important Examples

### Case 1: Creating a parent object

```java
class Payment {
    Payment(String method) {
        System.out.println(method);
    }

    void processPayment() {
        System.out.println("Payment processed");
    }
}

public class Main {
    public static void main(String[] args) {
        Payment p = new Payment("UPI");
        p.processPayment();
    }
}
```

Output:

```text
UPI
Payment processed
```

`new Payment("UPI")` works because the required argument is supplied.

`Payment p = new Payment();` fails because no no-argument constructor exists.

### Case 2: Creating a subclass object

```java
class Payment {
    Payment(String method) {
        System.out.println(method);
    }

    void processPayment() {
        System.out.println("Payment processed");
    }
}

class UpiPayment extends Payment {
    UpiPayment() {
        super("UPI");
    }
}
```

Then:

```java
UpiPayment upi = new UpiPayment();
upi.processPayment();
```

Output:

```text
UPI
Payment processed
```

**Tricky variation:** If `super("UPI")` is removed, compilation fails because Java attempts to insert `super()`, but `Payment` has no no-argument constructor.

The method `processPayment()` does not affect constructor rules. The object must be constructed successfully before its method can be called.

## 6. Private Constructor

- Directly callable only from within the declaring class.
- Commonly used in utility classes and Singleton implementations.
- A static factory method can create an object without an existing instance.

```java
class PaymentUtil {
    private PaymentUtil() {
        System.out.println("Object created");
    }

    static PaymentUtil create() {
        return new PaymentUtil(); // Allowed inside same class
    }
}

// PaymentUtil p = new PaymentUtil(); // Compilation error
PaymentUtil p = PaymentUtil.create(); // Allowed
```

**Remember:** `private` controls access; `static` allows a method to be called without an existing object.

## 7. Constructor Access Modifiers

| Modifier | Access |
|---|---|
| `public` | Anywhere |
| `protected` | Same package and qualifying subclass access |
| Package-private | Same package |
| `private` | Within the declaring class |

**Tricky variation:** A `public` class can have a `protected` constructor. An unrelated class in another package cannot directly invoke that constructor.

## 8. Constructors and Inheritance

**Tricky interview variations**

- Are constructors inherited? **No.**
- Can a child invoke a parent constructor? **Yes, using `super(...)`.**
- Are constructors overridden? **No.**
- Can a child declare a constructor with the same parameter list as its parent? **Yes.** It is a separate constructor, not overriding.
- Does a child automatically receive the parent's parameterized constructors? **No.**

**Memory trick:** Constructors are not inherited, not overridden, and not polymorphic. A child constructor can call a parent constructor using `super(...)`.

## 9. Constructors and Abstract Classes

- An abstract class cannot be instantiated directly.
- It can have constructors.
- Its constructor executes when a concrete subclass object is created.

**One-line answer:** An abstract class can have a constructor because it initializes the superclass portion of a concrete subclass object.

## 10. Constructors and Interfaces

- An interface cannot declare constructors.
- An implementing class can declare constructors.
- `Payment p = new UpiPayment()` invokes the `UpiPayment` constructor.

**One-line answer:** Interfaces cannot have constructors because they cannot be instantiated; the implementing class's constructor initializes its objects.

## 11. Object Creation and Initialization Order

For a subclass object:

1. Memory is allocated; instance fields initially receive default values.
2. Superclass initialization and constructor execution occur first.
3. Subclass instance field initializers and initializer blocks execute.
4. Subclass constructor body executes.

Static initialization occurs during class initialization, with the superclass initialized before the subclass.

Example:

```java
class Payment {
    int amount = 100;

    Payment() {
        amount = 200;
    }
}

class UpiPayment extends Payment {
    int fee = 10;

    UpiPayment() {
        System.out.println(amount);
        System.out.println(fee);
    }
}
```

Output:

```text
200
10
```

## 12. Static and Final Constructor Traps

- `static` constructor → **Not allowed**.
- `final` constructor → **Not allowed**.
- Constructors cannot be overridden because they are not inherited.
- A class itself can be `final` and still have a constructor.

```java
final class Payment {
    Payment() {}
}
```

## Final Interview Memory Sheet

- `this()` = same-class constructor.
- `super()` = superclass constructor.
- Parent constructor executes before child constructor body.
- Default constructor = compiler-generated only when no constructor is declared.
- No-argument constructor = any constructor with zero parameters.
- Constructors can be overloaded, but not overridden.
- Constructors are not inherited.
- Private constructors restrict direct construction from outside the declaring class.
- Abstract classes can have constructors; interfaces cannot.
- `static` and `final` cannot be applied to constructors.


# Java Core Topic 4 — Methods, Overloading & Overriding

## 4.1 Method Overloading

**Definition:** Same method name with different parameter lists. The compiler selects the method at compile time.

### Rules
- Parameters can differ by **number, type, or order**.
- Changing only the return type is not enough.
- Overloading is resolved at compile time.
- Overloading can happen in the same class or through inheritance.
- `main()` can be overloaded, but the JVM starts with `public static void main(String[] args)`.

### Tricky interview variations

```java
void pay(int amount) {}
void pay(double amount) {}
void pay(int amount, String mode) {}
```

All three can coexist because their parameter lists differ.

**1. Return type alone**

```java
int calculate(int a) { return a; }
// double calculate(int a) { return a; } // Compilation error
```

Same parameters, different return type → invalid overload.

**2. Exact match vs widening**

```java
void pay(int a) {}
void pay(long a) {}
void pay(double a) {}

pay(100);  // int
pay(100L); // long
```

Exact match wins. If only `pay(int)` and `pay(double)` exist, `pay(100L)` selects `pay(double)` because `long` can widen to `double`, but cannot implicitly narrow to `int`.

**3. Overloading with inheritance**

```java
class Payment {
    void pay(int amount) {}
}

class UpiPayment extends Payment {
    void pay(double amount) {}
}

Payment p = new UpiPayment();
p.pay(100); // Payment.pay(int)
```

The compiler uses the reference type and available overloads to select a signature; runtime dispatch can then choose an overriding implementation of that signature.

**4. Overloaded `main()`**

```java
public static void main(String[] args) {
    main(10); // Explicitly calls the overloaded method
}

static void main(int n) {}
```

The JVM does not automatically start with `main(int)`.

---

## 4.2 Method Overriding

**Definition:** A subclass provides its own implementation of an inherited instance method with the same signature.

**Interview one-liner:** Overloading means the same method name with different parameter lists and compile-time selection; overriding means redefining an inherited instance method in a subclass, with runtime dispatch based on the actual object.

### Runtime polymorphism

```java
class Payment {
    void pay() {
        System.out.println("General payment");
    }
}

class UpiPayment extends Payment {
    @Override
    void pay() {
        System.out.println("UPI payment");
    }
}

Payment p = new UpiPayment();
p.pay(); // UPI payment
```

Reference type: `Payment`; actual object: `UpiPayment`. The overridden instance method in `UpiPayment` executes.

### Overriding rules

| Rule | Interview answer |
|---|---|
| Signature | Same method name and parameter list |
| Visibility | Cannot reduce access visibility |
| Return type | Same or a compatible covariant reference type |
| `final` method | Cannot be overridden |
| `private` method | Not inherited; same-named child method is not an override |
| `static` method | Hidden, not overridden |
| Checked exceptions | Cannot declare a broader or unrelated checked exception than permitted by the parent declaration |

### Important tricky variations

**1. Visibility cannot be reduced**

If the parent method is `protected`, the child cannot override it as `private`. The child may use `protected` or `public`.

**2. Covariant return type**

```java
class Payment {
    Payment process() { return this; }
}

class UpiPayment extends Payment {
    @Override
    UpiPayment process() { return this; }
}
```

Valid because `UpiPayment` is a subtype of `Payment`.

**3. Checked exceptions — preserve this interview rule**

An overriding method may declare the same checked exception, a narrower checked exception, or no checked exception. It cannot declare a broader or unrelated checked exception.

```java
class Payment {
    void process() throws IOException {}
}

class UpiPayment extends Payment {
    @Override
    void process() throws FileNotFoundException {}
}
```

Allowed because `FileNotFoundException` is a subclass of `IOException`.

But declaring `throws Exception` in the child would be invalid here because `Exception` is broader than `IOException`.

**4. Static method hiding**

```java
class Payment {
    static void process() {
        System.out.println("Payment");
    }
}

class UpiPayment extends Payment {
    static void process() {
        System.out.println("UPI");
    }
}

Payment p = new UpiPayment();
p.process(); // Payment
```

Static method selection depends on the reference/class type, not the actual object. Prefer calling static methods using the class name.

**5. Private methods**

A parent `private` method is not inherited. A same-named child method is a separate method, not an override. Calling the private method through a parent reference from unrelated code causes an access error.

**6. Final methods**

A `final` instance method cannot be overridden. Attempting to override it with the same signature causes a compilation error.

**7. Constructors**

Constructors are neither inherited nor overridden. A child constructor implicitly calls `super()` when no explicit `this(...)` or `super(...)` invocation is supplied and the implicit call is applicable. The parent constructor runs before the child's constructor body.

---

## Final 30-second revision

- **Overloading:** Different parameter lists; compile-time selection.
- **Overriding:** Same signature; runtime dispatch for instance methods.
- **Return type alone:** Cannot overload.
- **Primitive widening:** `long` can widen to `double`, not implicitly narrow to `int`.
- **Static:** Hiding, not overriding.
- **Private:** Not inherited, so not overridden.
- **Final:** Cannot be overridden.
- **Covariant return:** Child can return a subtype.
- **Checked exceptions:** Same, narrower, or none; not broader or unrelated.
- **Constructors:** Not inherited or overridden; parent constructor runs first.

## Progress

- Topic 4.1 — Method Overloading: Completed (quiz 3/5).
- Topic 4.2 — Method Overriding: Completed (quiz 7/8).
- Revision priorities: return-type-only overloading, primitive widening, static method hiding, and covariant return types.



# Java Core — Topic 5: `main()` and Program Execution

## 1. Why is `main()` static?
The traditional launcher can invoke it without creating an object first.

## 2. Why is `main()` public?
It allows the traditional launcher to access the entry-point method.

## 3. Why is the return type void?
The traditional entry point doesn't return a value to the launcher.

## 4. What is `String[] args`?
It receives command-line arguments as strings.

Example: `java PaymentApp order123 500`

- `args[0]` → `"order123"`
- `args[1]` → `"500"`
- `args.length` → `2`

With no arguments, the launcher normally supplies an empty array.

## 5. Can `main()` be overloaded?
**Yes.** The traditional launcher starts with the recognized entry-point signature; it doesn't automatically choose `main(int)`.

## 6. Can `main()` be overridden?
Static methods are **hidden**, not overridden. A subclass can declare a static method with the same signature.

## 7. What if `main()` isn't static?
With the traditional launch rules, an instance method such as `public void main(String[] args)` isn't the traditional entry point.

## 8. What if the signature changes?
A valid method such as `public static void main()` can compile, but the traditional launcher won't recognize it as the expected entry point.

## 9. Can Java execute code without a traditional `main()`?
Yes. Examples include JShell, test frameworks, and newer Java launch mechanisms. The exact rules depend on the execution mechanism and Java version.

## 10. Modern Java launch mechanisms
Java 25 supports flexible main methods, including eligible instance main methods, and compact source files. These features aren't available under every older JDK or launch mode.

## 11. What does the launcher look for?
For a traditional Java application:

`public static void main(String[] args)`

- `public` → accessible
- `static` → callable without an instance first
- `void` → no return value
- `main` → expected method name
- `String[] args` → command-line arguments

**Technical distinction:** The Java launcher locates and invokes the entry point; the JVM executes the bytecode.

## Interview traps to remember

- Overloading `main()` is allowed.
- Static methods are hidden, not overridden.
- Compilation success doesn't guarantee successful launching.
- A method named `start()` doesn't become the entry point automatically.
- Distinguish traditional entry-point rules from Java 25's flexible main-method rules.

# Java 8+ Core Features — Quick Recall

Completed topics: Functional Interfaces, Lambdas, Built-in Functional Interfaces, Default/Static Methods, Optional

## 1. Functional Interface

Definition: An interface with exactly one abstract method (SAM — Single Abstract Method).

```java
@FunctionalInterface
interface PaymentProcessor {
    void processPayment();

    default void log() {}
    static void validate() {}
}
```

Remember:

- `@FunctionalInterface` is optional; it tells the compiler to verify the rule.
- Adding a second abstract method causes a compile-time error when the annotation is present.
- Multiple `default` and `static` methods are allowed because they have implementations.
- `Runnable` is a functional interface because it has one abstract method: `void run()`.

### Future-demand note: remember this mental model

> Interface: defines the behavior required.  
> Lambda: supplies the behavior.  
> Functional interface: connects the lambda to a method contract.

## 2. Lambda Expressions

A lambda provides the implementation of a functional interface's abstract method with less boilerplate than an anonymous class.

General pattern:

```java
(parameters) -> { method body }
```

| Part | Meaning |
|---|---|
| `(parameters)` | Inputs the behavior needs |
| `->` | Separates inputs from behavior |
| `{ method body }` | Work to perform |

Java allows shorter forms for simple code; parentheses and braces are not always needed.

### How to decide what to write

**Case 1 — No input, no return value**

```java
Runnable task =
    () -> System.out.println("Payment processed");
```

- `()` means no input.
- Printing is the work.
- No value is returned.

**Case 2 — One input, returns `true` or `false`**

```java
Predicate<Integer> expensive =
    amount -> amount > 1000;
```

- `amount` is the input.
- `amount > 1000` is the condition.
- Result is a `boolean`.

**Case 3 — One input, transforms it into another value**

```java
Function<Integer, Integer> addFee =
    amount -> amount + 100;
```

- `amount` is the input.
- `amount + 100` calculates the result.
- An expression body returns its result automatically.

**Case 4 — One input, performs an action without returning a result**

```java
Consumer<String> logger =
    orderId -> System.out.println(orderId);
```

- `orderId` is the input.
- Printing is the action.
- Nothing is returned.

### Expression body vs block body

```java
Function<Integer, Integer> doubleIt =
    n -> n * 2;
```

The expression result is returned automatically.

```java
Function<Integer, Integer> doubleIt =
    n -> {
        return n * 2;
    };
```

With a block body, use `return` when a value must be returned.

### When should you use a lambda?

Use one when a method expects a functional interface and you need to pass a small piece of behavior.

```java
amounts.stream()
       .filter(amount -> amount > 1000)
       .forEach(System.out::println);
```

- `filter()` expects a `Predicate`.
- `forEach()` expects a `Consumer`.
- A method reference such as `System.out::println` is a shorter way to refer to existing behavior.

For larger, reusable business logic, a named method or separate class may be clearer.

## 3. Built-in Functional Interfaces

The most important thing is to recognize the job each interface performs.

| Interface | Mental model | Main method |
|---|---|---|
| `Predicate<T>` | Input → check → boolean | `test()` |
| `Function<T, R>` | Input → output | `apply()` |
| `Consumer<T>` | Input → action → nothing returned | `accept()` |
| `Supplier<T>` | Nothing → output | `get()` |
| `Runnable` | Nothing → task → nothing returned | `run()` |

### Predicate — check something

```java
Predicate<Integer> isBig =
    amount -> amount > 5000;
```

Input `6000` → `true`; input `3000` → `false`.

### Function — transform something

```java
Function<Integer, Integer> addFee =
    amount -> amount + 100;
```

Input `1000` → output `1100`.

**Future-demand note — do not miss this:**

`Function<InputType, OutputType>`

- First `Integer` → the type of value you provide.
- Second `Integer` → the type of value you get back.

The two types can differ:

```java
Function<String, Integer> getLength =
    text -> text.length();
```

Input `"Payment"` (`String`) → output `7` (`Integer`).

**Memory trick:** First type goes in; second type comes out.

`Function<T, R>` is not a `Map`. A `Map` stores key-value pairs; a `Function` represents an operation that takes an input and produces an output.

### Consumer — perform an action

```java
Consumer<String> log =
    id -> System.out.println(id);
```

Input `"ORD101"` → prints the ID; returns nothing.

### Supplier — supply a value

```java
Supplier<String> getStatus =
    () -> "PENDING";

String status = getStatus.get();
```

- Takes no input.
- `.get()` returns `"PENDING"`.
- It does not print anything unless you explicitly print the result.

### Runnable — perform a task

```java
Runnable task =
    () -> System.out.println("Payment processed");

task.run();
```

- Takes no input and returns nothing.
- `task.run()` executes like a normal method call on the current thread.
- `new Thread(task).start()` starts a new thread to execute the task.

**Anonymous class vs lambda:**

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Payment processing");
    }
};
```

The anonymous class implements `Runnable`; `run()` supplies the task's behavior. Defining the task does not start a thread.

**Lambda equivalent:**

```java
Runnable task =
    () -> System.out.println("Payment processing");

Thread thread = new Thread(task);
thread.start();
```

**Interview one-liner:**

> `Runnable` defines a task through its `run()` method. We can implement it using an anonymous class or lambda, pass it to a `Thread`, and call `start()` to execute it on a new thread.

## 4. Default and Static Methods in Interfaces

Both methods have implementations, so neither counts as an abstract method.

### Default method

Called through an implementing object's reference.

```java
interface PaymentService {
    default void printStatus() {
        System.out.println("Ready");
    }
}

class CardPayment implements PaymentService {}

new CardPayment().printStatus();
```

### Static method

Called using the interface name.

```java
interface PaymentService {
    static void validate() {
        System.out.println("Validating");
    }
}

PaymentService.validate();
```

| Default | Static |
|---|---|
| Called through an implementing object | Called through the interface name |
| Can be inherited by an implementing class | Belongs to the interface itself |
| Has an implementation | Has an implementation |

**Interview trap:** A functional interface may have one abstract method plus multiple default and static methods.

## 5. Optional

`Optional` represents a value that may or may not be present. It's useful when a method may not find a result.

For example, an Order Service may not find an order for a given ID.

### Creating Optional values

| Method | Meaning |
|---|---|
| `Optional.of(value)` | Value must not be `null` |
| `Optional.ofNullable(value)` | Value may be `null` |
| `Optional.empty()` | No value is present |

```java
String status = null;

Optional<String> result =
    Optional.ofNullable(status);
```

This creates an empty `Optional` rather than throwing an exception because `status` is `null`.

### Handling a missing value

```java
Optional<String> order = Optional.empty();

String result = order.orElse("ORDER NOT FOUND");
```

`result` becomes `"ORDER NOT FOUND"`.

| Method | What it does |
|---|---|
| `orElse(value)` | Returns the existing value or the fallback |
| `orElseGet(supplier)` | Returns the existing value or calls the supplier if empty |
| `orElseThrow()` | Returns the value or throws `NoSuchElementException` if empty |
| `orElseThrow(() -> new MyException())` | Throws your specified exception if empty |

**Important difference:**

```java
order.orElse("DEFAULT");
```

The fallback expression is evaluated even when the `Optional` contains a value.

```java
order.orElseGet(() -> "DEFAULT");
```

The supplier is called only when the `Optional` is empty.

### Where should you use Optional?

- Commonly as a method return type when a result may be absent.
- It helps callers handle missing values explicitly.
- It does not guarantee that `NullPointerException` can never occur.
- Don't use it automatically for every variable, field, or parameter.

**Example interview answer:**

> I use `Optional` as a return type when a method may not find a result. It makes the absence of a value explicit and provides methods such as `orElse`, `orElseGet`, and `orElseThrow` to handle that case.

## Final 60-second revision

**Functional interface**

Exactly one abstract method; multiple default and static methods are allowed.

**Lambda**

Supplies behavior to a functional interface; use `->` between inputs and behavior.

**Five interfaces**

Predicate checks, Function transforms, Consumer acts, Supplier supplies, Runnable runs.

**Default vs static**

Default is called through an object; static is called through the interface name.

**Optional**

Represents a possibly absent value; use `ofNullable()` for a potentially null value.

## Topic 6 completion tracker

- [x] Functional Interface
- [x] Lambda Expressions
- [x] Built-in Functional Interfaces
- [x] Default and Static Methods
- [x] Optional


# Java Core — Topic 7: Exception Handling & Null Handling

## 1. Exception fundamentals

### Throwable hierarchy

```text
Object
  └── Throwable
       ├── Error
       └── Exception
            ├── Checked Exceptions
            └── RuntimeException (Unchecked)
```

| Type | Meaning | Example |
|---|---|---|
| Error | Serious JVM/system-level problem | `OutOfMemoryError` |
| Checked exception | Compiler requires handling or declaration | `IOException` |
| Unchecked exception | No mandatory catch or declaration | `NullPointerException` |

**Interview answer:** Exception represents conditions an application may handle; Error generally represents serious problems that applications are not expected to recover from.

## 2. `throw` vs `throws`

| Point | `throw` | `throws` |
|---|---|---|
| Purpose | Actually throws an exception | Declares possible exceptions |
| Location | Inside method body | Method signature |
| Followed by | Exception object | Exception class names |
| Quantity | Throws one exception at a time | Can declare multiple exceptions |

**Interview one-liner:**

"`throw` is used to explicitly throw an exception, whereas `throws` is used in a method signature to declare exceptions that the method may pass to its caller."

## 3. Common exception-handling keywords

| Keyword | Purpose |
|---|---|
| `try` | Contains code that may throw an exception |
| `catch` | Handles an exception |
| `finally` | Executes cleanup code in normal execution paths |
| `throw` | Explicitly throws an exception |
| `throws` | Declares exceptions in a method signature |

**Exception propagation:** If a method doesn't handle an exception, it propagates to its caller until handled.

### Common built-in exceptions

| Exception | When it occurs |
|---|---|
| `IllegalArgumentException` | Invalid method argument |
| `IllegalStateException` | Operation attempted in an invalid state |
| `NullPointerException` | Unexpected null reference usage |
| `UnsupportedOperationException` | Requested operation isn't supported |
| `IOException` | Input/output failure |

**Remember:** Don't throw `NullPointerException` just to report invalid input. Prefer validation and a meaningful exception such as `IllegalArgumentException`.

## 4. Custom exceptions in Spring Boot

Custom exceptions represent application-specific or business errors.

**Step 1 — Create custom exception**

```java
public class PaymentNotFoundException extends RuntimeException {
    public PaymentNotFoundException(String message) {
        super(message);
    }
}
```

- `extends RuntimeException` makes it unchecked.
- `super(message)` passes the message to the parent exception.

**Step 2 — Throw it from service**

```java
public Payment getPayment(Long paymentId) {
    return repository.findById(paymentId)
        .orElseThrow(() ->
            new PaymentNotFoundException("Payment not found: " + paymentId)
        );
}
```

**Step 3 — Handle it globally**

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(PaymentNotFoundException.class)
    public ResponseEntity<String> handlePaymentNotFound(
            PaymentNotFoundException ex) {

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(ex.getMessage());
    }
}
```

This returns HTTP 404 Not Found with a meaningful message.

**Interview one-liner:**

"Custom exceptions are user-defined exceptions created to represent application-specific errors. In Spring Boot, I commonly extend `RuntimeException` for business exceptions and handle them centrally using `@RestControllerAdvice` and `@ExceptionHandler`."

### Checked vs unchecked custom exceptions

| Point | Checked | Unchecked |
|---|---|---|
| Extends | `Exception` | `RuntimeException` |
| Can use `throw`? | Yes | Yes |
| Must catch or declare? | Yes | No |
| Must use `throws`? | No, if caught | No |
| Can use try-catch? | Yes | Yes |

## 5. Exception overriding rules

An overriding method cannot throw a broader checked exception than the parent method.

**Example:**

```java
class Payment {
    public void process() throws IOException {
    }
}

class UpiPayment extends Payment {
    @Override
    public void process() throws FileNotFoundException {
    }
}
```

This is valid because `FileNotFoundException` is a subclass of `IOException`.

| Parent method | Child method | Valid? |
|---|---|---|
| `throws IOException` | `throws IOException` | Yes |
| `throws IOException` | `throws FileNotFoundException` | Yes |
| `throws IOException` | No throws declaration | Yes |
| `throws IOException` | `throws Exception` | No |
| No checked exception | `throws IOException` | No |
| No checked exception | `throws RuntimeException` | Yes |

**Interview one-liner:**

"An overriding method cannot throw broader checked exceptions than the overridden method, but it can throw narrower checked exceptions or unchecked exceptions."

## 6. Tricky `finally` scenarios

### When does finally execute?

| Situation | Finally executes? |
|---|---|
| No exception in try | Yes |
| Exception handled by catch | Yes |
| Exception not handled | Yes, before propagation |
| Return inside try | Yes |
| Return inside catch | Yes |

**When might it not execute?**

- `System.exit(0)` terminates the JVM.
- JVM crashes.
- Process is forcibly terminated.
- Machine shuts down unexpectedly.

**Interview answer:**

"Finally generally executes regardless of whether an exception occurs or is handled. It is mainly used for cleanup operations, but it is not guaranteed during JVM termination or abnormal process shutdown."

### Return inside try and finally

```java
public int test() {
    try {
        return 10;
    } finally {
        return 20;
    }
}
```

<escape>**Tricky:**</escape> The result is `20`.

- Finally executes before the pending return.
- If finally itself returns, it overrides the return from try or catch.
- Returning from finally is discouraged because it can suppress exceptions and override results.

### Exceptions thrown by try and finally

| Situation | What propagates? |
|---|---|
| Only try throws | Try exception |
| Only finally throws | Finally exception |
| Both throw | Finally exception takes precedence |
| Try throws, finally completes normally | Try exception |

If both try and finally throw, the finally exception takes precedence. The try exception does not automatically become a suppressed exception.

**Important distinction:** Try-with-resources handles closing exceptions differently. Closing exceptions are recorded as suppressed exceptions on the primary exception and can be accessed using `getSuppressed()`.

## 7. NullPointerException (NPE)

### Why does NPE happen?

NPE occurs when we try to access a field, method, or perform an operation through a null reference.

```java
Payment payment = null;
payment.getAmount(); // NullPointerException
```

### Three ways to prevent NPE

**1. Null validation**

```java
if (payment == null) {
    throw new IllegalArgumentException("Payment cannot be null");
}
System.out.println(payment.getAmount());
```

**2. `Objects.requireNonNull()`**

```java
Objects.requireNonNull(payment, "Payment cannot be null");
System.out.println(payment.getAmount());
```

Throws NPE with the supplied message if the reference is null.

**3. Optional**

```java
Optional<Payment> payment = repository.findById(paymentId);
payment.ifPresent(p -> System.out.println(p.getAmount()));
```

Optional is useful for potentially absent return values. It is not a replacement for every null check.

### Null-safe design

Null-safe design means preventing unexpected null values from spreading through the application.

**Principles:**

- Validate required inputs.
- Avoid returning null unnecessarily.
- Return empty collections when there are no results.
- Use Optional when a return value may be absent.

**Interview-ready answer:**

"Null-safe design means preventing unexpected null values from spreading through the application. I validate required inputs, avoid returning null unnecessarily, use empty collections where appropriate, and use Optional when a return value may be absent."

### Why is catching NPE everywhere bad?

- Hides the actual problem.
- Makes debugging difficult.
- Can hide programming mistakes.
- Creates poor error-handling practices.

> **Remember:**
>
> - Validate required data.
> - Use Optional when absence is expected.
> - Return empty collections instead of null.
> - Catch NPE only when you have a specific recovery strategy.

## 8. Optional — important interview methods

Optional is a container that may or may not contain a value. It helps handle missing return values explicitly instead of returning null.

### `Optional.of()`, `ofNullable()` and `empty()`

#### 1. `Optional.of(value)`

Use when you are sure the value is not null.

```java
Payment payment = new Payment();

Optional<Payment> result = Optional.of(payment);
```

**Value is present**

If `payment` is `null`, it throws a `NullPointerException`.

#### 2. `Optional.ofNullable(value)`

Use when the value might be null.

```java
Payment payment = null;

Optional<Payment> result = Optional.ofNullable(payment);
```

**No exception**

If `payment` is `null`, it returns `Optional.empty()`.

#### 3. `Optional.empty()`

Use when you want to explicitly return an Optional with no value.

```java
public Optional<Payment> getPayment() {
    return Optional.empty();
}
```

This tells the caller that no payment is available.

### Quick comparison

| Method | When to use | If value is null |
|---|---|---|
| `of()` | Value is guaranteed non-null | Throws NPE |
| `ofNullable()` | Value might be null | Returns empty |
| `empty()` | No value is available | Creates empty Optional |

**Interview memory trick:**

- `of()` → Definitely have a value.
- `ofNullable()` → Maybe have a value.
- `empty()` → Definitely have no value.

### `orElse()` vs `orElseGet()`

| Method | Fallback execution |
|---|---|
| `orElse()` | Always evaluated |
| `orElseGet()` | Evaluated only when Optional is empty |

```java
Payment payment = result.orElse(new Payment());
```

```java
Payment payment = result.orElseGet(() -> new Payment());
```

**Interview one-liner:**

"`orElse()` evaluates the fallback eagerly, whereas `orElseGet()` evaluates it lazily using a Supplier only when the Optional is empty."

### `orElseThrow()`

Returns the value if present; otherwise throws the specified exception.

```java
Payment payment = repository.findById(paymentId)
    .orElseThrow(() ->
        new PaymentNotFoundException("Payment not found")
    );
```

### `isPresent()` vs `ifPresent()`

| Method | Purpose | Return |
|---|---|---|
| `isPresent()` | Checks whether value exists | `boolean` |
| `ifPresent()` | Executes action if value exists | `void` |

```java
if (result.isPresent()) {
    System.out.println("Payment found");
}
```

```java
result.ifPresent(payment ->
    System.out.println(payment.getAmount())
);
```

If Optional is empty, `ifPresent()` does nothing.

### `map()` — transform a value

```java
Optional<Payment> result = paymentRepository.findById(101L);
Optional<Double> amount = result.map(Payment::getAmount);
```

If empty, the result remains `Optional.empty()`.

### `filter()` — retain a value if condition matches

```java
Optional<Payment> result = Optional.of(payment);

Optional<Payment> filtered =
    result.filter(p -> p.getAmount() > 1000);
```

If the condition fails, returns `Optional.empty()`.

### Chaining `filter()` and `map()`

```java
Optional<Payment> result = Optional.of(payment);

Optional<Double> amount = result
    .filter(p -> p.getAmount() > 1000)
    .map(Payment::getAmount);
```

If amount is `1500.0`, result is `Optional[1500.0]`.

If the filter condition fails, result is `Optional.empty()`.

### `get()` vs `orElseThrow()`

| Method | If Optional is empty |
|---|---|
| `get()` | Throws `NoSuchElementException` |
| `orElseThrow()` | Throws the specified exception |

Prefer `orElseThrow()` over unsafe `get()` when handling missing database records.

## 9. How do you handle NPE in your Spring Boot project?

**Interview-ready answer**

"In my Spring Boot project, I prevent NullPointerException by validating required request objects before accessing their fields. If any required data is missing, I throw a meaningful custom exception. I handle these exceptions centrally using `@RestControllerAdvice` and `@ExceptionHandler`. For database records that may not exist, I use Optional with `orElseThrow()` instead of returning null."

### Practical request validation example

```java
public class PaymentRequest {

    @NotNull(message = "Customer is required")
    @Valid
    private Customer customer;

    @NotNull(message = "Amount is required")
    private BigDecimal amount;
}
```

**Controller:**

```java
@PostMapping("/payments")
public ResponseEntity<?> createPayment(
        @Valid @RequestBody PaymentRequest request) {

    return ResponseEntity.ok(paymentService.process(request));
}
```

- `@Valid` triggers request validation.
- `@NotNull` checks that the customer object itself is not null.
- `@Valid` on the nested customer enables validation of its fields too.

## Final Topic 7 status

**Completed**

| Section | Status |
|---|---|
| Exception fundamentals | Completed |
| `throw` vs `throws` | Completed |
| try/catch/finally | Completed |
| Exception propagation | Completed |
| Custom exceptions | Completed |
| Tricky finally scenarios | Completed |
| Exception overriding rules | Completed |
| NPE prevention | Completed |
| Null-safe design | Completed |
| Optional | Completed |
| Spring Boot NPE interview scenario | Completed |
