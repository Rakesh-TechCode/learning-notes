# Java Backend Testing — Quick Recall Notes

> **Goal:** Quickly remember what each testing concept is, why we use it, and the important JUnit/Mockito syntax.

---

# 1. Testing Basics ✅

What is Testing? Testing verifies that application code behaves correctly for expected, invalid, and failure scenarios.

### Unit Testing

What is it? Testing a small unit of code independently, usually one method/class.

```text
Example:
OrderService → tested independently
Repository → mocked
```

**Focus:** business logic.

---

### Integration Testing

What is it? Testing how multiple real application components work together.

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

**Focus:** interaction between components.

---

### Positive Testing

What is it? Testing valid and expected inputs.

```text
Valid order ID
Valid amount
Valid request
```

Expected behavior should occur.

---

### Negative Testing

What is it? Testing invalid inputs and failure scenarios.

```text
Invalid ID
Negative amount
Missing value
Dependency failure
```

Expected error/exception behavior should occur.

---

### Test Pyramid

What is it? A testing approach where fast, focused tests form the foundation and higher-level tests are used less frequently.

```text
        Higher-level tests
       Integration tests
      Unit tests
```

**Key idea:** More unit tests, fewer expensive higher-level tests.

---

### Arrange → Act → Assert (AAA)

What is it? A simple structure for organizing a test.

```java
// Arrange
when(repository.findOrderById(101))
        .thenReturn("Order-101");

// Act
String result = service.getOrder(101);

// Assert
assertEquals("Order-101", result);
```

```text
Arrange → prepare test
Act     → execute code
Assert  → verify result
```

---

# 2. JUnit 5 ✅

What is JUnit 5? A Java testing framework used to write and execute automated tests.

## JUnit Basics

### `@Test`

What is it? Marks a method as a test method.

```java
@Test
void shouldReturnOrder() {
    ...
}
```

---

### Basic Test Execution

What is it? Running test methods through the IDE/build tool and checking whether they pass or fail.

```text
✅ Test passed
❌ Test failed
```

---

## Assertions

What are assertions? They compare the actual result with the expected result.

### `assertEquals()`

> Checks that two values are equal.

```java
assertEquals("Order-101", result);
```

### `assertNotEquals()`

> Checks that two values are different.

```java
assertNotEquals("Order-999", result);
```

### `assertTrue()`

> Checks that a condition is true.

```java
assertTrue(number > 0);
```

### `assertFalse()`

> Checks that a condition is false.

```java
assertFalse(number < 0);
```

### `assertNull()`

> Checks that a value is `null`.

```java
assertNull(result);
```

### `assertNotNull()`

> Checks that a value is not `null`.

```java
assertNotNull(result);
```

### `assertThrows()`

> Checks that executing code throws the expected exception.

```java
assertThrows(
    RuntimeException.class,
    () -> orderService.getOrder(999)
);
```

---

## Test Lifecycle

What is the test lifecycle?** It defines code that runs before/after test methods.

### `@BeforeEach`

What is it? Runs before every test.

```java
@BeforeEach
void setup() {
    ...
}
```

### `@AfterEach`

What is it? Runs after every test.

```java
@AfterEach
void cleanup() {
    ...
}
```

### `@BeforeAll`

What is it? Runs once before all tests in the class.

### `@AfterAll`

What is it?** Runs once after all tests in the class.

Note: We discussed these, but skipped the hands-on because you already knew them.

---

## Test Readability

What is this?** Practices that make tests easy to understand from their names and reports.

### `@DisplayName`

What is it?** Gives a human-readable name to a test.

```java
@Test
@DisplayName("Should return order when order exists")
void shouldReturnOrderWhenOrderExists() {
    ...
}
```

---

### Test Naming Best Practices

What is it?** Naming tests according to the behavior/scenario being tested.

✅ Good:

```text
shouldReturnOrderWhenOrderExists()
shouldThrowExceptionWhenOrderDoesNotExist()
```

❌ Poor:

```text
test1()
testOrder()
methodTest()
```

**Recall rule:**

```text
What should happen + Under what condition?
```

---

## Parameterized Testing

What is it?** Running the same test multiple times with different inputs.

### `@ParameterizedTest`

What is it?** Marks a test that receives multiple input values.

```java
@ParameterizedTest
```

### `@ValueSource`

What is it?** Supplies a set of simple values to a parameterized test.

```java
@ValueSource(ints = {1, 2, 5, 10})
```

**Example:**

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 5, 10})
void shouldCheckPositiveNumbers(int number) {
    assertTrue(number > 0);
}
```

---

### `@CsvSource`

What is it?** Supplies multiple values/arguments for each test execution.

```java
@ParameterizedTest
@CsvSource({
    "2, 3, 5",
    "5, 5, 10",
    "10, 20, 30"
})
void shouldAddNumbers(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}
```

**Recall:**

```text
@ValueSource → single parameter
@CsvSource   → multiple parameters
```

---

## Scenario Testing

What is it?** Testing the important normal, edge, and failure paths of the application.

### Normal/Valid Case

> Test the expected successful behavior.

```java
assertEquals(5, calculator.divide(10, 2));
```

### Edge Case

> Test boundary or unusual situations.

```text
divide by zero
null
empty value
negative value
```

### Failure/Exception Case

> Test how the application behaves when something goes wrong.

```java
assertThrows(
    ArithmeticException.class,
    () -> calculator.divide(10, 0)
);
```

---

## Plain Java Service Hands-on

What is it?** Practicing JUnit on simple Java classes without bringing up the Spring context.

We practiced with:

```text
Calculator
OrderService
```

and tested:

```text
normal result
edge case
exception
AAA structure
```

---

## JUnit Practical Understanding

What is it?** Applying JUnit concepts in actual executable tests rather than only learning syntax.

Covered:

```text
Calculator testing
OrderService testing
AAA
Assertions
Parameterized tests
Exception testing
Running tests
Reading test results
```

### 🎯 JUnit Status

**✅ JUnit 5 Fundamentals COMPLETE**

---

# 3. Mockito — Core Concepts ✅

What is Mockito?** A mocking framework used to create controlled fake dependencies for unit testing.

## Why Mockito?

What is it for?** Mockito lets us test a class without depending on real external components such as databases or external services.

```text
OrderService
     ↓
Mock OrderRepository
```

No real database call.

---

## Mock vs Real Dependency

> **What is the difference?** A real dependency performs its actual work; a mock is a controlled fake used during testing.

```text
Real Repository → actual database
Mock Repository → fake dependency
```

---

## `@Mock`

> **What is it?** Creates a Mockito mock of a dependency.

```java
@Mock
OrderRepository orderRepository;
```

---

## `@InjectMocks`

> **What is it?** Creates the real class under test and injects the available mocks into it.

```java
@InjectMocks
OrderService orderService;
```

---

## `MockitoExtension`

> **What is it?** Connects Mockito with JUnit 5 and initializes Mockito annotations.

```java
@ExtendWith(MockitoExtension.class)
```

---

# 4. Mockito — Stubbing & Basic Service Testing ✅

> **What is stubbing?** Defining how a mock should behave when a method is called.

## `when().thenReturn()`

> **What is it?** Tells the mock what value to return for a specific call.

```java
when(orderRepository.findOrderById(101))
        .thenReturn("Order-101");
```

**Meaning:**

```text
When repository gets 101
        ↓
Return "Order-101"
```

---

## Basic Service → Repository

> **What is this pattern?** The service is the real class being tested while its repository dependency is mocked.

```text
OrderService     → REAL
      ↓
OrderRepository  → MOCK
```

---

## Mockito Hands-on

> **What is it?** Applying the Mockito setup in a real JUnit test.

We created:

```text
OrderRepository
OrderService
OrderServiceTest
```

and successfully executed Mockito tests.

---

# 5. Mockito — Exceptions & Verification ✅

> **What is this section?** Testing dependency failures and verifying how the service interacts with its mocks.

## `when().thenThrow()`

> **What is it?** Makes a mock throw an exception instead of returning a value.

```java
when(orderRepository.findOrderById(999))
        .thenThrow(new RuntimeException("Order not found"));
```

Typically combined with:

```java
assertThrows(...)
```

---

## `verify()`

> **What is it?** Checks whether a mock method was called.

```java
verify(orderRepository)
        .findOrderById(101);
```

---

## `verify(..., times())`

> **What is it?** Verifies how many times a mock method was called.

```java
verify(orderRepository, times(2))
        .findOrderById(101);
```

---

## `verify(..., never())`

> **What is it?** Verifies that a mock method was never called.

```java
verify(orderRepository, never())
        .findOrderById(-1);
```

---

## Service → Repository Verification Hands-on

> **What is it?** Combining stubbing, exception testing, and interaction verification in one realistic flow.

We practiced:

```text
Success
→ thenReturn()
→ verify()

Failure
→ thenThrow()
→ assertThrows()
→ verify()

Invalid input
→ assertThrows()
→ never()
```

### 🎯 Section Status

**✅ Mockito Exceptions & Verification COMPLETE**

---

# 6. Mockito — Argument Matchers ⏳

> **What are Argument Matchers?** They tell Mockito what kinds of argument values should match during stubbing or verification.

## `any()`

> **What is it?** Matches any value of the specified type.

For integers:

```java
anyInt()
```

**Example:**

```java
when(orderRepository.findOrderById(anyInt()))
        .thenReturn("Order");
```

---

## `anyInt()`

> **What is it?** Matches any `int` value.

```java
anyInt()
```

So:

```text
101 ✅
200 ✅
999 ✅
```

---

## `eq()`

> **What is it?** Matches a specific exact value.

```java
eq(101)
```

**Meaning:**

```text
101 ✅
200 ❌
999 ❌
```

---

## `anyInt()` vs `eq()`

```text
anyInt() → any integer
eq(101)  → exactly 101
```

---

## Why Argument Matchers Are Needed

> **What are they useful for?** They allow flexible matching when the exact argument value is not important.

**Example:**

```java
when(orderRepository.saveOrder(
        eq(101),
        anyString(),
        anyDouble()
)).thenReturn("Order-101");
```

**Meaning:**

```text
orderId      → exactly 101
customerName → any String
amount       → any double
```

---

## Combining Matchers Correctly

> **What is the rule?** When using matchers in a method call, use matchers for all arguments in that call.

### ✅ Correct

```java
verify(orderRepository).saveOrder(
        eq(101),
        eq("Rakesh"),
        anyDouble()
);
```

### ❌ Avoid mixing matchers with raw values

```java
verify(orderRepository).saveOrder(
        eq(101),
        "Rakesh",
        anyDouble()
);
```
```text
once you use a matcher like eq() or anyDouble(), all arguments in that method call must use matchers.
Mockito does not allow mixing argument matchers with raw values in the same method call. I should use eq("Rakesh") for the second argument.
```
**Recall rule:**

```text
One matcher in the call
        ↓
Use matchers for all arguments
```

---

## `anyString()`

> **What is it?** Matches any `String` value.

```java
anyString()
```

---

## `anyDouble()`

> **What is it?** Matches any `double` value.

```java
anyDouble()
```

---

## Service → Repository Matcher Hands-on

> **What is it?** Applying matchers to a service method that passes multiple arguments to a repository.

We created:

```text
createOrder(orderId, customerName, amount)
```

and practiced:

```text
eq(101)
anyString()
anyDouble()
```

---

## Verify Calls With Specific Arguments

> **What is it?** Checks that the repository received the exact values we expected.

```java
verify(orderRepository).saveOrder(
        eq(101),
        eq("Rakesh"),
        eq(2500.0)
);
```

---

### 🎯 Current Status

**⏳ Argument Matchers — final hands-on confirmation pending**

The final test has been prepared, but we haven't marked this section complete until you confirm that it runs successfully.

---



# 🧠 30-Second Recall

```text
JUnit
→ writes/runs tests
→ assertions verify results

Mockito
→ mocks dependencies
→ controls dependency behavior
→ verifies interactions
```

```text
thenReturn() → return controlled value
thenThrow()  → throw controlled exception

verify()     → was it called?
times()      → how many times?
never()      → was it NOT called?

anyInt()     → any integer
eq(101)      → exact value
```


# Mockito — Quick Recall Notes — Continuation

## Mockito — Argument Matchers ✅

> **What are Argument Matchers?** They tell Mockito what kind of argument should match during stubbing or verification.

### `any()`

> **What is it?** Matches any value of the required type.

```java
when(orderRepository.findOrderById(anyInt()))
        .thenReturn("Order");
```

```text
anyInt() → any integer
```

---

### `eq()`

> **What is it?** Matches one exact value.

```java
when(orderRepository.findOrderById(eq(101)))
        .thenReturn("Order-101");
```

```text
eq(101) → exactly 101
```

#### `anyInt()` vs `eq()`

```text
anyInt() → any integer
eq(101)  → only 101
```

---

### Why Argument Matchers Are Needed

> **What are they useful for?** They allow flexible matching when we don't care about the exact value of every argument.

```text
eq(101)      → exact ID
anyString()  → any name
anyDouble()  → any amount
```

Example:

```java
when(orderRepository.saveOrder(
        eq(101),
        anyString(),
        anyDouble()
)).thenReturn("Order-101");
```

---

### Combining Matchers Correctly

> **What is the rule?** If you use matchers for a method call, use matchers for all arguments in that call.

**Correct:**

```java
verify(orderRepository).saveOrder(
        eq(101),
        eq("Rakesh"),
        anyDouble()
);
```

**Avoid mixing matcher + raw value:**

```java
verify(orderRepository).saveOrder(
        eq(101),
        "Rakesh",
        anyDouble()
);
```

**Recall rule:**

```text
One matcher in a call
        ↓
Use matchers for all arguments
```

---

### `anyString()`

> **What is it?** Matches any `String`.

```java
anyString()
```

---

### `anyDouble()`

> **What is it?** Matches any `double`.

```java
anyDouble()
```

---

### Service → Repository Hands-on

> **What is it?** Applying matchers when a service passes multiple arguments to a repository.

Example:

```java
orderService.createOrder(
        101,
        "Rakesh",
        2500.0
);
```

Repository:

```java
saveOrder(
        eq(101),
        anyString(),
        anyDouble()
);
```

---

### Verify Calls With Specific Arguments

> **What is it?** Checks that the dependency was called with the exact values we expect.

```java
verify(orderRepository).saveOrder(
        eq(101),
        eq("Rakesh"),
        eq(2500.0)
);
```

### 🎯 Status

**✅ Argument Matchers section covered**

---

## 7. Mockito — `ArgumentCaptor` ✅

> **What is `ArgumentCaptor`?** It captures the actual argument passed to a mocked dependency so we can inspect and assert it.

### Why use it?

```text
verify()
→ Was the method called?

ArgumentCaptor
→ What exact value was passed?
```

### Example

```java
ArgumentCaptor<Integer> orderIdCaptor =
        ArgumentCaptor.forClass(Integer.class);

verify(orderRepository).saveOrder(
        orderIdCaptor.capture(),
        nameCaptor.capture(),
        amountCaptor.capture()
);
```

Then retrieve the values:

```java
assertEquals(101, orderIdCaptor.getValue());
assertEquals("Rakesh", nameCaptor.getValue());
assertEquals(2500.0, amountCaptor.getValue());
```

### Mental model

```text
OrderService
     ↓
saveOrder(101, "Rakesh", 2500.0)
     ↓
ArgumentCaptor
     ↓
captures actual arguments
```

---

### Capturing Repository/Dependency Arguments

> **What is it?** Verifying not only that the dependency was called, but also what the service actually sent to it.

```java
verify(orderRepository).saveOrder(
        orderIdCaptor.capture(),
        nameCaptor.capture(),
        amountCaptor.capture()
);
```

---

## 8. Mockito — `doReturn()` ✅

> **What is `doReturn()`?** An alternative way to stub a mock method with a controlled return value.

**Standard:**

```java
when(orderRepository.findOrderById(101))
        .thenReturn("Order-101");
```

**Alternative:**

```java
doReturn("Order-101")
        .when(orderRepository)
        .findOrderById(101);
```

### Recall

```text
when().thenReturn()
→ standard stubbing

doReturn().when()
→ alternative stubbing
```

`doReturn()` is particularly useful in situations such as working with spies where calling the real method during stubbing can be undesirable.

---

## 9. Mockito — `doThrow()` ✅

> **What is `doThrow()`?** Makes a mocked method throw an exception when called.

```java
doThrow(new RuntimeException("Order not found"))
        .when(orderRepository)
        .findOrderById(999);
```

### Recall

```text
thenThrow()
→ when().thenThrow()

doThrow()
→ doThrow().when()
```

`doThrow()` becomes especially useful for **void methods**.

---

## 10. Mockito — Mocking `void` Methods ✅

> **What is a void method?** A method that performs an action but returns no value.

Example:

```java
void updateOrderStatus(int orderId, String status);
```

Because there is no return value, we don't use:

```java
when(...).thenReturn(...)
```

Instead, Mockito provides the `do...` style.

### `doNothing()`

```java
doNothing()
        .when(orderRepository)
        .updateOrderStatus(101, "SHIPPED");
```

> Means: when this void method is called, do nothing.

**Note:** For a normal Mockito mock, `doNothing()` is generally the default behavior for void methods. It is mainly useful to understand the `do...` syntax and for explicit stubbing.

### Verify the call

```java
verify(orderRepository)
        .updateOrderStatus(101, "SHIPPED");
```

---

## 11. Mockito — Void Method Exception Testing ✅

> **What is it?** Testing what happens when a void dependency method throws an exception.

For void methods:

```java
doThrow(new RuntimeException("Database update failed"))
        .when(orderRepository)
        .updateOrderStatus(101, "SHIPPED");
```

Then verify the exception:

```java
assertThrows(
        RuntimeException.class,
        () -> orderService.updateOrderStatus(101, "SHIPPED")
);
```

### Flow

```text
OrderService
     ↓
updateOrderStatus()
     ↓
Mock Repository
     ↓
RuntimeException
     ↓
assertThrows()
```

---

## 12. Mockito — Combined Service → Repository Practice ⏳

> **What is it?** A final exercise that combines the Mockito concepts instead of testing each concept separately.

Planned scenarios:

```text
1. Create Order
   → doReturn()
   → ArgumentCaptor
   → verify()

2. Find Order
   → doReturn()
   → assertEquals()

3. Repository failure
   → doThrow()
   → assertThrows()
   → verify()

4. Update Order Status
   → void method
   → verify()

5. Void method failure
   → doThrow()
   → assertThrows()
```

### Status

**⏳ Combined exercise completion confirmation pending**

The individual concepts above are already covered.

---

# Mockito — Quick Recall

```text
anyInt()
→ any integer

eq(101)
→ exact value
```

```text
when()
→ defines mock behavior
```

```text
doReturn()
→ alternative return stubbing
```

```text
thenThrow()
→ throw exception with when()
```

```text
doThrow()
→ throw exception using do... style
→ especially useful for void methods
```

```text
verify()
→ was method called?
```

```text
times()
→ how many times?
```

```text
never()
→ was it not called?
```

```text
ArgumentCaptor
→ what exact arguments were passed?
```

```text
doNothing()
→ explicitly do nothing for a void method
```

# 📌 Mockito — Current Progress

```text
Argument Matchers
✅ Complete

ArgumentCaptor
✅ Complete

Capturing dependency arguments
✅ Complete

doReturn()
✅ Complete

doThrow()
✅ Complete

Mocking void methods
✅ Complete

Void + exception testing
✅ Complete
```
