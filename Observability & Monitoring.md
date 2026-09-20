# Observability & Monitoring — Quick Recall Notes

## Phase 1 — Observability Fundamentals

### 1. What is Observability?

**Meaning:** Observability is the ability to understand what is happening inside a running application by analyzing the data it produces.

* **Main sources:**

  * **Logs**
  * **Metrics**
  * **Traces**
* Helps answer: **"What is happening inside my application and why?"**

---

### 2. Why Backend Microservices Need Observability

**Meaning:** Microservices have many independent services and dependencies, so observability helps identify where problems occur.

* Understand service health
* Identify problematic services
* Detect downstream failures
* Identify latency/bottlenecks
* Understand request flow across services

**Example:**

```text
Order Service
     ↓
Payment Service
     ↓
Database
```

If Order API is slow, observability helps determine **where the delay is happening**.

---

### 3. Logs vs Metrics vs Traces

**Meaning:** Logs, metrics, and traces provide different views of application behavior.

| Type        | Quick Recall                         | Example                            |
| ----------- | ------------------------------------ | ---------------------------------- |
| **Logs**    | What happened?                       | `Payment failed for order ORD-101` |
| **Metrics** | How much/how often/how fast?         | API latency = 2 seconds            |
| **Traces**  | Where did the request go/spend time? | Order → Payment → DB               |

**Memory trick:**

```text
Logs    → What happened?
Metrics → How much/how fast?
Traces  → Where/which service?
```

---

### 4. Monitoring vs Observability

**Meaning:** Monitoring detects known problems, while observability helps investigate the reason behind system behavior.

**Monitoring**

* Detect predefined/known problems
* Example: CPU > 90%
* Example: 5xx error rate increased

**Observability**

* Helps investigate **why** something is happening
* Uses logs, metrics, traces together

```text
Monitoring     → Something is wrong
Observability  → Why is it wrong?
```

---

### 5. Basic Production Troubleshooting

**Meaning:** Use observability data step-by-step instead of immediately assuming the root cause.

**Basic flow:**

```text
Problem
   ↓
Check Metrics
   ↓
Find abnormal behavior
   ↓
Check Logs
   ↓
Find supporting evidence
   ↓
Identify likely problematic component
   ↓
Investigate further
```

**Example:**

```text
Order API slow
      ↓
Metrics → High latency
      ↓
Logs → Payment timeout
      ↓
Payment Service becomes a likely area to investigate
```

**Important:** Don't immediately conclude that the first suspicious component is definitely the root cause.

---

# Phase 2 — Spring Boot Logging

## 1. Application Logging

**Meaning:** Application logging records important events, actions, warnings, and failures occurring inside an application.

**Example:**

```java
log.info("Order created successfully: {}", orderId);
```

**Used for:**

* Understanding application behavior
* Debugging
* Troubleshooting
* Investigating failures

---

## 2. Log Levels

**Meaning:** Log levels indicate the importance/detail of a log message.

```text
TRACE → Extremely detailed
DEBUG → Debugging information
INFO  → Normal important events
WARN  → Potential/unusual problem
ERROR → Failure/error
```

### Quick recall

* **TRACE** → very detailed investigation
* **DEBUG** → developer debugging
* **INFO** → normal application flow
* **WARN** → something unusual, application continues
* **ERROR** → operation/failure requiring attention

**Example:**

```java
log.info("Order created: {}", orderId);

log.warn("Payment response is slow: {} ms", responseTime);

log.error("Payment failed for order: {}", orderId);
```

---

## 3. SLF4J

**Meaning:** SLF4J is a **logging abstraction/API** that decouples application code from the actual logging implementation.

**Common SLF4J APIs:**

```text
Logger
LoggerFactory
```

**Architecture:**

```text
Application
    ↓
  SLF4J
    ↓
  Logback
    ↓
Console/File
```

### Key point

> **SLF4J = abstraction/API**
>
> **Logback = implementation**

---

## 4. Logger

**Meaning:** `Logger` is used by application code to write log messages.

**Standard pattern:**

```java
private static final Logger log =
        LoggerFactory.getLogger(OrderService.class);
```

### Components

**`Logger`**

Provides methods:

```java
log.trace()
log.debug()
log.info()
log.warn()
log.error()
```

**`LoggerFactory`**

Creates/returns a logger associated with a class.

**`OrderService.class`**

Identifies the source/class associated with the logs.

**Example:**

```java
LoggerFactory.getLogger(PaymentService.class);
```

This helps identify that the log came from `PaymentService`.

---

## 5. Logback

**Meaning:** Logback is a **logging implementation** that actually processes and outputs logs.

Spring Boot commonly uses Logback as its default logging implementation.

**Flow:**

```text
Your Code
   ↓
SLF4J Logger
   ↓
Logback
   ↓
Console / File
```

Logback can control things such as:

* Log output destination
* Log levels
* Log format
* Basic logging configuration

### Important distinction

```text
SLF4J  → What API the application uses
Logback → How logging is actually implemented
```

---

# Proper Logging Practices

## 6. Good vs Bad Log Messages

**Meaning:** Logs should provide useful information for understanding application behavior and troubleshooting.

### ✅ Good

```java
log.info("Order created successfully: {}", orderId);
```

### ❌ Bad/noisy

```java
log.info("Entered method");
log.info("Going to execute next line");
```

### Rule

> Log meaningful application/business events, not every line of code.

---

## 7. Parameterized Logging `{}`

**Meaning:** SLF4J supports parameterized logging using `{}` placeholders.

### ❌ Avoid

```java
log.info("Creating order: " + orderId);
```

### ✅ Prefer

```java
log.info("Creating order: {}", orderId);
```

### Remember

```text
{}         → placeholder
argument   → actual value
```

---

## 8. What Should / Shouldn't Be Logged

**Meaning:** Logs should contain useful troubleshooting information but must not expose sensitive data.

### Generally useful

* Order ID
* User/request identifiers where appropriate
* Operation/status
* Error information
* Timing information

### Don't log sensitive information

❌ Passwords

❌ OTPs

❌ Authentication tokens

❌ API keys/secrets

❌ Sensitive card/payment information

**Example:**

```java
// ❌ Never do this
log.info("User password: {}", password);
```

---

## 9. Exception Logging

**Meaning:** Exception logging records both useful context and the exception/stack trace.

### ❌ Less useful

```java
log.error("Payment failed");
```

### ✅ Better

```java
log.error("Payment failed for order: {}", orderId, e);
```

This gives:

```text
Context
  ↓
orderId

+

Exception
  ↓
message + stack trace
```

### Why stack trace matters?

It helps developers identify **where the failure occurred**.

---

## 10. Useful Contextual Information

**Meaning:** Context makes a log easier to understand and investigate.

**Example:**

```java
log.error("Payment failed for order: {}", orderId, e);
```

Instead of:

```java
log.error("Payment failed");
```

Useful context can include:

```text
orderId
request information
service/class
operation
exception
timing information
```

The exact information depends on the application and security requirements.

---

# Structured Logging

## 11. Structured Logging Basics

**Meaning:** Structured logging represents log information as consistent, machine-readable fields, commonly JSON.

### Plain-text example

```text
Payment failed for order ORD-101
```

### Structured example

```json
{
  "level": "ERROR",
  "service": "payment-service",
  "orderId": "ORD-101",
  "message": "Payment failed"
}
```

Now tools can understand individual fields:

```text
level   = ERROR
service = payment-service
orderId = ORD-101
```

### Why useful?

Especially with centralized logging tools such as ELK:

* Easier searching
* Easier filtering
* Easier analysis
* Consistent fields across services

### Important distinction

**Parameterized logging ≠ Structured logging**

```java
log.info("Order created: {}", orderId);
```

This is **parameterized logging**.

Structured logging might produce:

```json
{
  "message": "Order created",
  "orderId": "ORD-101"
}
```

### Important practical point

You **don't normally manually write JSON for every log message**.

The logging framework/configuration can be set up to produce structured/JSON logs automatically.

We'll see this hands-on during **ELK**.

---

# Spring Boot Logging Hands-on

## 12. Spring Boot Logging Practice

**Meaning:** We implemented the concepts in a small Spring Boot application.

### Logger setup

```java
private static final Logger log =
        LoggerFactory.getLogger(OrderService.class);
```

### INFO logging

```java
log.info("Creating order: {}", orderId);
```

### DEBUG logging

```java
log.debug("Validating order: {}", orderId);
```

### ERROR + exception

```java
log.error("Payment failed for order: {}", orderId, e);
```

### Package-level DEBUG configuration

In:

```text
application.properties
```

we configured:

```properties
logging.level.com.testing.Observability_Practice=DEBUG
```

This enabled DEBUG logs for that package.

### What we verified

```text
INFO  → visible
DEBUG → initially not visible
DEBUG → visible after configuration
ERROR → visible with exception/stack trace
```

---

# 🧠 Ultra-Quick Recall

If you only have **2 minutes before an interview**, remember this:

```text
OBSERVABILITY
→ Understand what's happening inside a running system.

3 PILLARS
→ Logs    = What happened?
→ Metrics = How much/how fast?
→ Traces  = Where did the request go?

MONITORING
→ Detect known problems.

OBSERVABILITY
→ Investigate why.

SPRING BOOT LOGGING
→ SLF4J = logging abstraction/API
→ Logback = logging implementation
→ Logger = writes logs
→ LoggerFactory = creates/returns Logger

LOG LEVELS
→ TRACE → DEBUG → INFO → WARN → ERROR

GOOD LOGGING
→ Useful business/application events
→ Use {} parameterized logging
→ Add useful context
→ Log exceptions with stack traces
→ Don't log passwords/OTPs/tokens/secrets/card data

STRUCTURED LOGGING
→ Consistent machine-readable fields
→ Commonly JSON
→ Easier search/filter in ELK
→ Not the same as parameterized logging

TROUBLESHOOTING
→ Metrics → identify abnormal behavior
→ Logs → investigate evidence
→ Identify likely component
→ Investigate further
```
