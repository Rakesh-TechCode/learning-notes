# Database — Quick Recall Notes

## 1. Database Fundamentals

### 1.1 Database Transaction

**Definition:**
A **transaction** is a group of database operations treated as **one logical business operation**.

```text
BEGIN
  ↓
DB Operations
  ↓
COMMIT / ROLLBACK
```

**Why do we need transactions?**

To ensure that related database operations are completed **together**, instead of leaving the database in a partially updated state.

**Example — Create Order:**

```text
Create Order
Update Stock
Create Payment Record
```

If everything succeeds:

```text
COMMIT
```

If something fails:

```text
ROLLBACK
```

---

### 1.2 COMMIT

**Definition:**
`COMMIT` permanently saves the changes made by the transaction.

```sql
COMMIT;
```

```text
Transaction
    ↓
COMMIT
    ↓
Changes become committed
```

---

### 1.3 ROLLBACK

**Definition:**
`ROLLBACK` cancels the changes made by the current transaction since its start/savepoint.

```sql
ROLLBACK;
```

Example:

```text
INSERT Order
UPDATE Stock
Payment fails
    ↓
ROLLBACK
    ↓
Previous changes are undone
```

**Important:** You normally don't manually `DELETE` the inserted data to undo it. The database transaction mechanism handles the rollback.

---

### 1.4 Spring Boot `@Transactional`

**Definition:**
`@Transactional` tells Spring to execute a method inside a database transaction.

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);

    paymentRepository.save(payment);

}
```

Conceptually:

```text
BEGIN
 ↓
save order
 ↓
save payment
 ↓
COMMIT
```

If an appropriate exception causes rollback:

```text
BEGIN
 ↓
save order
 ↓
exception
 ↓
ROLLBACK
```

---

### 1.5 How Rollback Actually Works

Example:

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);

    throw new RuntimeException("Something went wrong!");
}
```

Conceptually:

```text
BEGIN
 ↓
INSERT Order
 ↓
RuntimeException
 ↓
ROLLBACK
```

The database discards the uncommitted transaction changes.

---

### 1.6 RuntimeException vs Checked Exception

By default, Spring's `@Transactional` rollback behavior includes **unchecked exceptions such as `RuntimeException`**.

For a checked exception, explicitly configure rollback when required:

```java
@Transactional(rollbackFor = Exception.class)
public void createOrder() {

    // DB operations

}
```

**Interview recall:**

```text
RuntimeException → rollback by default
Checked Exception → configure rollback when required
```

---

### 1.7 Local Transaction vs Distributed Transaction

**Local Transaction:**

Transaction operates within one transactional resource/database.

```text
Order Service
     ↓
Order DB
     ↓
One transaction
```

**Distributed Transaction:**

One business operation spans **multiple services/databases**.

```text
Order Service → Order DB
      ↓
Payment Service → Payment DB
```

**Basic awareness only for now:**

Normal `@Transactional` does not automatically make multiple independent service databases one atomic transaction.

---

# 2. ACID Properties

ACID describes the key properties expected from reliable database transactions.

```text
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```

---

## 2.1 Atomicity

**Definition:**
A transaction is **all-or-nothing**.

If one operation fails, the transaction's changes are rolled back.

Example:

```text
Create Order       ✅
Update Stock       ✅
Create Payment     ❌
        ↓
ROLLBACK
        ↓
Previous operations undone
```

**Remember:**

> All operations succeed → COMMIT
> Any failure → ROLLBACK

---

## 2.2 Consistency

**Definition:**
A successful transaction moves the database from **one valid state to another valid state**, according to database constraints and business rules.

Example:

```text
Stock = 10
Buy 3
Stock = 7 ✅
```

But:

```text
Stock = 10
Buy 15
Stock = -5 ❌
```

if the business rule requires:

```text
stock >= 0
```

**Important:** Consistency comes from a combination of:

* Database constraints
* Application/business rules

**Don't confuse it with:**

> "All databases always contain the same data."

---

## 2.3 Isolation

**Definition:**
Isolation controls **how concurrent transactions interact with each other**.

Multiple transactions can run concurrently:

```text
Transaction A
      ↕
Transaction B
      ↕
Database
```

Isolation determines **what one transaction can see from another transaction**.

Without appropriate isolation, problems such as:

* Dirty Read
* Non-Repeatable Read
* Phantom Read

can occur.

**Important:**

> Isolation does NOT mean only one transaction runs at a time.

---

## 2.4 Durability

**Definition:**
Once a transaction is committed, its changes should survive a database/system crash.

```text
Transaction
    ↓
COMMIT
    ↓
Data persisted
    ↓
Database crashes
    ↓
Database restarts
    ↓
Committed data remains
```

### How?

At a high level, MySQL/InnoDB uses **persistent storage and recovery mechanisms such as redo logging**.

After a crash, database crash recovery uses the persisted recovery information to preserve committed changes.

### Durability ≠ Backup

**Durability:**

```text
Protect committed data from transaction/system crash
```

**Backup:**

```text
Protect against things like accidental deletion,
corruption, or larger disasters
```

---

# 3. Isolation Levels

## 3.1 Isolation Level

**Definition:**
An isolation level defines **how isolated one transaction is from other concurrent transactions**.

It controls what a transaction can see while other transactions are running.

Main levels covered:

```text
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

---

# 4. Concurrent Transaction Problems

## 4.1 Dirty Read

**Definition:**
A **Dirty Read** occurs when Transaction B reads data changed by Transaction A **before A commits**.

Example:

```text
Initial:
balance = 1000

A → UPDATE balance = 5000
A → NOT COMMITTED

B → READ balance
B → sees 5000 ❌

A → ROLLBACK

Actual value → 1000
```

B read data that was **never committed**.

### Key recall

```text
Dirty Read
→ Read UNCOMMITTED data
```

---

# 5. READ COMMITTED

**Definition:**
`READ COMMITTED` allows a transaction to read **only committed data** from other transactions.

Spring Boot:

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void processOrder() {
    // DB operations
}
```

### Dirty Read

```text
Transaction A
    ↓
UPDATE
    ↓
NOT COMMITTED
       X
       ↓
Transaction B
    ↓
Cannot see A's uncommitted change
```

Therefore:

```text
READ COMMITTED
→ Dirty Read ❌
```

### Practical use

Useful when:

> You need to avoid reading uncommitted data but don't necessarily require the same data snapshot throughout the entire transaction.

### Important limitation

`READ COMMITTED` does **not** prevent Non-Repeatable Reads.

Another transaction can commit a change between your two reads.

```text
A → READ → PENDING

B → UPDATE → SHIPPED
B → COMMIT

A → READ again → SHIPPED
```

---

# 6. Non-Repeatable Read

**Definition:**
A **Non-Repeatable Read** occurs when the same transaction reads the **same row twice** but gets different values because another transaction committed a change between the reads.

Example:

```text
Initial:
Order 101 → PENDING

A → READ → PENDING

B → UPDATE → SHIPPED
B → COMMIT

A → READ again → SHIPPED
```

Same transaction.
Same row.
Different value.

### Key recall

> **Non-Repeatable Read = Same row, different value.**

---

# 7. REPEATABLE READ

**Definition:**
`REPEATABLE READ` provides a **consistent view of previously read data within a transaction**, so another transaction's committed update doesn't normally change the result of that repeated read.

Spring Boot:

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void checkOrder(Long orderId) {

    Order order = orderRepository.findById(orderId)
            .orElseThrow();

    // Business logic

    Order orderAgain = orderRepository.findById(orderId)
            .orElseThrow();
}
```

Conceptually:

```text
A → READ → PENDING

B → UPDATE → SHIPPED
B → COMMIT

A → READ again → PENDING
```

### Important distinction

A seeing `PENDING` here is **not a Dirty Read**.

Why?

Because:

```text
PENDING → previously committed value
```

It is not:

```text
SHIPPED → uncommitted value
```

So:

```text
Dirty Read
→ Uncommitted data ❌

Repeatable Read
→ Consistent view of committed data ✅
```

### Practical trade-off

REPEATABLE READ can intentionally give your transaction a consistent/older view rather than always showing the latest committed state.

That's useful when a transaction needs **consistent reads throughout its work**.

### Key recall

> **Non-Repeatable Read = Same row changes between reads.**
> **REPEATABLE READ = Keep the read consistent within the transaction.**

---

# 8. Phantom Read

**Definition:**
A **Phantom Read** occurs when the same query is executed twice inside a transaction and the **set/number of matching rows changes** because another transaction commits changes that match the query.

Example:

Initial:

```text
101 → PENDING
102 → PENDING
```

Transaction A:

```sql
SELECT *
FROM orders
WHERE status = 'PENDING';
```

Result:

```text
2 rows
```

Transaction B:

```text
INSERT 103 → PENDING
COMMIT
```

Transaction A runs the same query again:

```text
3 rows
```

A new matching row appeared.

### Key recall

```text
Non-Repeatable Read
→ Same ROW
→ Value changes

Phantom Read
→ Same QUERY
→ Matching ROWS change
```

---

# 9. SERIALIZABLE

**Definition:**
`SERIALIZABLE` is the **strictest standard isolation level**, making conflicting concurrent transactions behave as though they were executed serially.

Spring Boot:

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void processOrders() {

    List<Order> orders =
            orderRepository.findByStatus("PENDING");

}
```

Conceptually:

```text
Transaction A
      ↓
Works with data
      ↓
Transaction B has conflicting operation
      ↓
B may WAIT / conflict
      ↓
A finishes
      ↓
B proceeds
```

### What it protects against?

```text
Dirty Read              ❌
Non-Repeatable Read     ❌
Phantom Read            ❌
```

### Conflict handling

Depending on the database and situation, a conflicting transaction may:

```text
WAIT
```

or may fail with a concurrency-related error.

If the failure is transient, the application can retry the **whole transaction**:

```text
Transaction fails
      ↓
ROLLBACK
      ↓
Start NEW transaction
      ↓
Re-read current data
      ↓
Retry
```

### Trade-off

```text
More isolation
      ↓
More protection
      ↓
Potentially more blocking/contention
      ↓
Potentially lower concurrency
```

Therefore:

> **SERIALIZABLE is not automatically "better"; it provides stronger isolation at a higher concurrency cost.**

---

# 10. Isolation Level Comparison

| Isolation Level     | Dirty Read | Non-Repeatable Read | Phantom Read       |
| ------------------- | ---------- | ------------------- | ------------------ |
| **READ COMMITTED**  | ❌          | Possible            | Possible           |
| **REPEATABLE READ** | ❌          | ❌                   | Database-dependent |
| **SERIALIZABLE**    | ❌          | ❌                   | ❌                  |

### Quick mental model

```text
READ COMMITTED
→ "Don't show me uncommitted data."

REPEATABLE READ
→ "Keep my transaction's reads consistent."

SERIALIZABLE
→ "Give me the strongest isolation between conflicting transactions."
```

### Important

> Higher isolation ≠ automatically better.

The correct isolation level depends on the **consistency requirement and concurrency needs** of the operation.

---

# 11. Spring Boot / JPA Isolation Implementation

### READ COMMITTED

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
```

### REPEATABLE READ

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
```

### SERIALIZABLE

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
```

### Default behavior

If you simply use:

```java
@Transactional
```

Spring uses the configured/default transaction isolation behavior rather than automatically meaning `REPEATABLE_READ`.

---

# 12. MySQL / InnoDB Default

**Definition:**
For MySQL's **InnoDB** storage engine, the default transaction isolation level is **REPEATABLE READ**.

Therefore:

```java
@Transactional
public void processOrder() {
    // DB operations
}
```

does not mean:

```java
Isolation.REPEATABLE_READ
```

because Spring itself isn't defining that as its universal default.

Instead, the database's configured/default isolation applies when no explicit isolation is requested.

### Interview recall

❌ Don't say:

> "Spring Boot's default isolation level is REPEATABLE READ."

✅ Say:

> "MySQL/InnoDB's default isolation level is REPEATABLE READ."

If you want to explicitly request it:

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
```

---

# 🎯 3 YOE Interview Recall — Isolation

### Dirty Read

> Reading **uncommitted data** from another transaction.

```text
Solution → READ COMMITTED
```

### Non-Repeatable Read

> Reading the **same row twice** and getting different values because another transaction committed an update.

```text
Solution → REPEATABLE READ
```

### Phantom Read

> Running the **same query twice** and getting different matching rows because another transaction inserted/deleted/changed matching data.

```text
Strong standard solution → SERIALIZABLE
```

### Isolation Levels

```text
READ COMMITTED
→ Only committed data
→ Dirty Read prevented

REPEATABLE READ
→ Consistent view within transaction
→ Non-Repeatable Read prevented

SERIALIZABLE
→ Strongest standard isolation
→ Conflicting operations behave serially
```

### The one-line interview answer

> **"Isolation levels control how concurrent transactions interact. READ COMMITTED prevents dirty reads, REPEATABLE READ provides consistent reads within a transaction, and SERIALIZABLE provides the strongest standard isolation but can reduce concurrency because of increased contention."**

---

# Overall Database Progress

```text
1. Database Fundamentals
   └── ACID
       ├── Atomicity       ✅
       ├── Consistency     ✅
       ├── Isolation       ✅
       └── Durability      ✅

2. Isolation Levels
   ├── Dirty Read          ✅
   ├── READ COMMITTED      ✅
   ├── Non-Repeatable Read ✅
   ├── REPEATABLE READ     ✅
   ├── Phantom Read        ✅
   ├── SERIALIZABLE        ✅
   ├── Spring Boot/JPA     ✅
   └── MySQL/InnoDB        ✅

3. Database Locking
   └── 🔜 NEXT
       ├── Optimistic Locking
       └── Pessimistic Locking
```

## 🔑 Ultra-Short Memory Trick

```text
DIRTY
→ Uncommitted data
→ READ COMMITTED

NON-REPEATABLE
→ Same row, different value
→ REPEATABLE READ

PHANTOM
→ Same query, different rows
→ SERIALIZABLE
```
