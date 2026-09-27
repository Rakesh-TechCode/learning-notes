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

# 🔐 Database Locking — Quick Recall

## 1. Locking Fundamentals

**What it is:** Database locking controls concurrent access when multiple transactions may modify the same data.

### Why locking is needed

Without proper concurrency control:

```text
Order 101 = PENDING

Transaction A → CONFIRMED
Transaction B → CANCELLED

Both read PENDING
Both update
Last update wins
        ↓
Lost Update
```

`@Transactional` alone does **not automatically prevent every lost-update scenario**.

### Isolation vs Locking

| Isolation                                     | Locking                     |
| --------------------------------------------- | --------------------------- |
| Controls what concurrent transactions can see | Controls conflicting access |
| READ COMMITTED                                | Optimistic locking          |
| REPEATABLE READ                               | Pessimistic locking         |
| SERIALIZABLE                                  | DB row locks                |

**Remember:** Isolation and locking are related but not the same thing.

---

# 2. Optimistic Locking

**What it is:** Assume conflicts are relatively rare; allow transactions to work and detect a conflict when updating.

### Problem

Two requests read:

```text
Order 101
status = PENDING
version = 1
```

A updates first:

```text
CONFIRMED
version = 2
```

B tries to update using version `1`.

The version no longer matches → **conflict detected**.

### JPA

```java
@Entity
public class Order {

    @Id
    private Long id;

    private String status;

    @Version
    private Long version;
}
```

Hibernate conceptually performs:

```sql
UPDATE orders
SET status = ?, version = 2
WHERE id = 101
AND version = 1;
```

If another transaction already changed the row:

```text
0 rows updated
      ↓
Version mismatch
      ↓
Optimistic locking exception
```

### Important exceptions

* `OptimisticLockException` — JPA
* `ObjectOptimisticLockingFailureException` — commonly surfaced by Spring

`@Version` is managed by JPA/Hibernate; **don't manually increment it**.

### Conflict handling

Application decides what to do:

```text
Conflict
   ↓
  ┌───────────────┬────────────────┬─────────────────┐
  ↓               ↓                ↓
Reject      Refresh + retry   Business resolution
```

Automatic bounded retry can also be used when safe.

**Important:** retry should happen in a **fresh transaction**.

### When useful

Good when:

* Concurrent conflicts are relatively uncommon
* You don't want to block transactions upfront
* Conflicts can be safely detected and handled

Very high contention can cause many conflicts/retries, making optimistic locking less attractive.

### Quick Recall

```text
@Version
   ↓
Version mismatch
   ↓
Optimistic locking exception
   ↓
Transaction rollback
   ↓
Application handles conflict
```

### 🎯 3 YOE Interview Answer

> **“`@Version` maintains a version number for an entity. When JPA updates the entity, it includes the version in the update condition. If another transaction has already modified the entity, the version no longer matches, so the update affects zero rows and Hibernate raises an optimistic locking exception instead of silently overwriting the newer data.”**

---

# 3. Pessimistic Locking

**What it is:** Lock the database row upfront because conflicting access is expected.

### Basic flow

```text
Transaction A
     ↓
Read Order 101
     ↓
🔒 Acquire write lock
     ↓
Update
     ↓
Commit / Rollback
     ↓
🔓 Lock released
```

### JPA

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Order> findById(Long id);
```

Conceptually similar to:

```sql
SELECT *
FROM orders
WHERE id = 101
FOR UPDATE;
```

Usually used inside:

```java
@Transactional
```

The lock is generally held until the transaction completes.

### Example — Inventory

```text
Stock = 1

Transaction A → locks product row
             → reads 1
             → decreases to 0
             → commits

Transaction B → waits
             → sees stock = 0
             → Out of Stock
```

### Downsides

* Blocking / waiting
* Lock contention
* Deadlocks
* Reduced concurrency

Keep transactions **short**.

Avoid holding DB locks while:

* Calling external services
* Performing heavy processing
* Doing unnecessary work

### Deadlock

```text
Transaction A:
Lock 101 → waits for 102

Transaction B:
Lock 102 → waits for 101
```

Database detects the deadlock and generally aborts/rolls back one transaction.

Reduce deadlocks with:

1. Consistent lock ordering
2. Short transactions
3. Bounded retry when appropriate

Retry should happen in a **fresh transaction**.

### 🎯 3 YOE Interview Answer

> **“No. Pessimistic locking can cause deadlocks. The database generally detects the deadlock and aborts one of the transactions, but the application should prevent or reduce deadlocks through consistent lock ordering and short transactions, and may retry the failed transaction when appropriate.”**

---

# 📊 Database Indexing — Quick Recall

## 1. Index Fundamentals

**What it is:** An index is a database data structure that helps locate rows efficiently without scanning the entire table.

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Without a useful index:

```text
Large table
   ↓
Scan many rows
```

With an index:

```text
Query
 ↓
Index
 ↓
Matching rows
```

**Index ≠ full copy of the table.**

---

# 2. MySQL / InnoDB & Index Storage

```text
MySQL
  ↓
InnoDB storage engine
  ↓
Table data + indexes
  ↓
Disk / SSD
```

InnoDB provides important features such as:

* Transactions / ACID
* Row-level locking
* Foreign keys
* Recovery mechanisms

Indexes are **persisted on disk/SSD**.

Frequently accessed index pages may also be cached in memory through the buffer pool.

**Important:** Indexes are not stored only in RAM.

---

# 3. Primary Key Index

When using InnoDB:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    status VARCHAR(20)
);
```

The primary key automatically gets an index.

So:

```text
❌ Don't create another normal index just for id
```

In InnoDB:

```text
Primary Key
    ↓
Clustered Index
    ↓
Actual row data
```

---

# 4. Secondary Index

A secondary index is an index created on a non-primary-key column.

Example:

```sql
CREATE INDEX idx_orders_user
ON orders(user_id);
```

Conceptually:

```text
Secondary Index
(user_id)
     ↓
Primary-key values
     ↓
Clustered / Primary Key Index
     ↓
Complete row
```

### 🎯 Interview wording

> **“In InnoDB, a secondary index contains the primary-key value for the indexed row. The primary key is then used to locate the complete row in the clustered index.”**

---

# 5. Which Columns Should Be Indexed?

Good candidates can include columns frequently used for:

* `WHERE` filtering
* `JOIN` conditions
* Common sorting/query patterns

Consider:

* Query frequency
* Selectivity
* Read/write workload
* Storage cost

Example:

```sql
SELECT *
FROM orders
WHERE user_id = 101;
```

If this query runs frequently, `user_id` is a good index candidate.

### Low selectivity

If a column has very few possible values:

```text
status:
ACTIVE
INACTIVE
```

and most rows are `ACTIVE`, an index may provide limited benefit.

**Don't automatically index every column.**

---

# 6. Composite Index

**What it is:** One index containing multiple columns.

```sql
CREATE INDEX idx_user_status
ON orders(user_id, status);
```

### Important

The **table's column order does not matter**.

The **index's column order matters**.

For:

```text
(user_id, status, created_at)
```

the useful leftmost prefixes are generally:

```text
user_id
user_id + status
user_id + status + created_at
```

But:

```text
status
created_at
status + created_at
```

do not get the same leftmost-prefix benefit.

### Mental model

```text
Index:
(user_id, status)

user_id                  ✅
user_id + status         ✅
status alone             ⚠️
```

### Important distinction

The query:

```sql
WHERE status = 'PAID'
```

can still return the correct data, but an index on:

```text
(user_id, status)
```

is generally not an efficient leftmost-prefix lookup for `status` alone.

Don't change the query just to force an index—the query must represent the actual business requirement.

---

# 7. Index Maintenance

The DB engine **automatically maintains indexes**.

The developer does not manually update them.

### INSERT

```text
Insert row
   ↓
Maintain affected indexes
```

### UPDATE

If an indexed column changes:

```text
Old index entry
      ↓
Modify/remove
      ↓
New index entry
```

Updating a non-indexed column generally doesn't require changing that index's key.

### DELETE

```text
Delete row
   ↓
Remove affected index entries
```

### Key idea

**Automatic maintenance ≠ free maintenance.**

The DB engine performs extra work using:

* CPU
* I/O
* Memory/buffer resources
* Storage
* Processing time

---

# 8. Index Costs

Indexes provide:

```text
Faster reads ✅
```

But introduce:

```text
Extra storage ❌
Write maintenance ❌
```

They can add overhead to:

* `INSERT`
* `UPDATE`
* `DELETE`

Composite indexes generally require more storage than single-column indexes.

### 🎯 Saved 3 YOE Interview Point

> **“Indexes improve read performance, but they consume storage and add maintenance overhead to INSERT, UPDATE, and DELETE operations. Composite indexes can provide efficient multi-column lookups, but they also generally require more storage than single-column indexes.”**

---

# 9. When Indexes May Not Help

An index isn't automatically faster.

It may provide limited benefit when:

### Small table

Scanning a tiny table may already be cheap.

### Low selectivity

If a query returns a huge percentage of the table, an index may not help much.

### Leading wildcard

```sql
WHERE name LIKE '%kumar'
```

A normal index generally can't efficiently use the unknown beginning of the value.

Whereas:

```sql
WHERE name LIKE 'Rakesh%'
```

can generally make better use of a normal index.

### Function on indexed column

```sql
WHERE LOWER(email) = 'rakesh@gmail.com'
```

A normal index on `email` may not be usable in the straightforward way.

### Important

The optimizer ultimately decides whether using an index is worthwhile.

---

# 10. `EXPLAIN`

**What it is:** `EXPLAIN` shows the query execution plan MySQL intends to use.

```sql
EXPLAIN
SELECT *
FROM orders
WHERE user_id = 101;
```

Useful things to inspect:

### `key`

Which index MySQL actually chose.

```text
key = idx_orders_user
```

### `possible_keys`

Indexes MySQL considers potentially useful.

**It does not mean the index was actually used.**

### `rows`

Estimated number of rows MySQL expects to examine.

**It's an estimate, not necessarily the exact number.**

### `type`

Basic access method.

```text
ALL    → generally full table scan
ref    → index-based lookup
const  → very efficient constant/single-row case
```

### Production flow

```text
Slow query
    ↓
EXPLAIN
    ↓
Check chosen index
    ↓
Check access type
    ↓
Check estimated rows
    ↓
Investigate query/index/selectivity
```

### 🎯 3 YOE Interview Answer

> **“I use `EXPLAIN` to inspect the query execution plan. I check which index MySQL selected, the access type, and the estimated rows examined. This helps determine whether the query is using an appropriate index or performing an expensive table scan.”**

---

# 11. Too Many Indexes

Don't index every column just because indexes can improve reads.

Too many indexes mean:

```text
More indexes
    ↓
More storage
    +
More INSERT/UPDATE/DELETE maintenance
    +
Potentially unnecessary indexes
```

An unused index can provide little/no read benefit while still carrying storage and maintenance costs.

**Index based on actual query patterns.**

---

# 12. Production Index Decision

Don't decide:

```text
Big table → automatically add indexes
```

Instead:

```text
Application query patterns
          ↓
Frequently executed queries
          ↓
Filtering / JOIN / sorting
          ↓
Selectivity
          ↓
Read vs write workload
          ↓
Storage cost
          ↓
EXPLAIN
          ↓
Verify actual benefit
```

### 🎯 Saved 3 YOE Interview Answer

> **“I look at the application's query patterns rather than simply the table size. Frequently executed queries involving filtering, joins, or sorting are candidates for indexes. I also consider selectivity, read/write workload, storage, and the execution plan to verify whether the index actually improves the query.”**

---

# 🧠 Final Locking + Indexing Mental Map

```text
CONCURRENCY
    │
    ├── Isolation
    │     └── What can transactions see?
    │
    └── Locking
          ├── Optimistic
          │     └── @Version → detect conflict
          │
          └── Pessimistic
                └── Lock row → prevent conflicting access


QUERY PERFORMANCE
    │
    └── Indexing
          ├── Primary Index
          ├── Secondary Index
          ├── Composite Index
          │     └── Leftmost-prefix
          ├── Index Maintenance
          ├── Selectivity
          ├── Index Costs
          └── EXPLAIN
                └── Verify execution plan
```

## ✅ Status

**LOCKING — COMPLETE**

**INDEXING — COMPLETE**
