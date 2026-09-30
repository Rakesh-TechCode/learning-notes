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

# Database — Quick Recall Notes — Continuation

> **Previous notes covered:** Fundamentals → Isolation → Locking → Indexing.

---

# 5. Database Bottlenecks

## What is a Database Bottleneck?

**Definition:** A database bottleneck occurs when the database becomes the limiting factor for application performance.

Common causes:

* Slow queries
* Missing/poor indexes
* Excessive database requests
* N+1 queries
* High read/write traffic
* Connection pool pressure

### Identification Flow

```text
API latency
    ↓
DB query latency
    ↓
Which query is slow?
    ↓
EXPLAIN
    ↓
Index / query optimization
```

**Interview answer:**

> A database bottleneck occurs when the database becomes the limiting factor for application performance. I would identify it using query metrics, logs, execution plans such as EXPLAIN, and database monitoring.

---

## Slow Queries

**Definition:** A slow query takes significant time to execute and can keep database resources busy.

Example:

```sql
SELECT *
FROM orders
WHERE user_id = 101;
```

If `orders` has 20M rows and there is no suitable index:

```text
20M rows
   ↓
No suitable index
   ↓
Possible full table scan
   ↓
High query latency
```

### Investigation Flow

```text
20M rows
   ↓
Query has no suitable index
   ↓
Possible full table scan
   ↓
High DB query latency
   ↓
Check query latency / metrics
   ↓
EXPLAIN
   ↓
Confirm access plan
   ↓
Add index if justified
   ↓
EXPLAIN again
   ↓
Verify improvement
```

---

## Missing / Poor Indexes

**Definition:** A missing or unsuitable index can force the database to examine many unnecessary rows.

Example:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Don't automatically add an index. Check:

* Query pattern
* Selectivity
* Read/write workload
* Existing indexes
* `EXPLAIN`
* Storage and write-maintenance cost

---

## `EXPLAIN`

**Definition:** `EXPLAIN` shows the database query execution plan.

```sql
EXPLAIN
SELECT *
FROM orders
WHERE user_id = 101;
```

Important things to inspect:

* `key` → index actually chosen
* `possible_keys` → possible indexes
* `rows` → estimated rows examined
* `type` → access method

Basic `type` recall:

```text
ALL   → generally full table scan
ref   → index lookup
const → very efficient constant/single-row lookup
```

### Interview answer

> I use `EXPLAIN` to inspect the query execution plan, especially the selected index, access type, and estimated rows examined. This helps identify expensive scans and unsuitable index usage.

---

# Too Many Database Requests

**Definition:** Excessive database calls can increase DB load, connection usage, network overhead, and application latency.

Important:

> More queries are not automatically bad. The problem is unnecessary or excessive queries relative to workload.

Mental model:

```text
Query count
     ×
Request volume
     ×
Query cost
     ↓
Database load
```

---

# N+1 Problem

**Definition:** N+1 occurs when the application executes one query to fetch N records and then executes an additional query for each record.

```text
1 query
   ↓
Fetch N records
   ↓
1 additional query per record
   ↓
N additional queries
   ↓
Total = N + 1
```

## Example

```java
List<Order> orders = orderRepository.findAll();

for (Order order : orders) {
    System.out.println(order.getCustomer().getName());
}
```

Conceptually:

```text
1 query → fetch 100 orders
100 queries → fetch each customer's data

Total = 101 queries
```

---

## N+1 in JPA/Hibernate

**Definition:** JPA/Hibernate can trigger N+1 when accessing associated entities causes an additional query for each parent record.

Example:

```text
Orders query
     ↓
100 orders
     ↓
Customer query for Order 1
Customer query for Order 2
...
Customer query for Order 100
```

---

## Practical N+1 Example

Order Service needs:

* Order
* Order items
* Product details

```java
Order order = orderRepository.findById(orderId);

List<OrderItem> items =
        orderItemRepository.findByOrderId(orderId);

for (OrderItem item : items) {
    Product product =
        productRepository.findById(item.getProductId());
}
```

For 10 items:

```text
1 order query
+ 1 item query
+ 10 product queries
--------------------
= 12 DB queries
```

At 1,000 API requests/sec:

```text
12 × 1,000
= 12,000 DB queries/sec
```

This can put significant pressure on the database.

---

## N+1 Solutions

### `JOIN FETCH`

**Definition:** Fetch related data using a join in one query when appropriate.

```java
@Query("""
    SELECT o
    FROM Order o
    JOIN FETCH o.customer
    WHERE o.id = :id
""")
Optional<Order> findOrderWithCustomer(Long id);
```

---

### DTO Projection

**Definition:** Fetch only the columns/data required by the API instead of loading complete entities.

Useful when the response needs a limited set of fields.

---

### Batch Fetching

**Definition:** Fetch related records in batches instead of executing one query per record.

Example idea:

```text
100 individual queries
        ↓
Fetch related data in batches
        ↓
Far fewer DB requests
```

### Interview answer

> I would first confirm the N+1 problem using query logs and monitoring. Depending on the use case, I could use JOIN FETCH, DTO projections, or batch fetching, and then verify that query count and latency improved.

---

# High Read Traffic

**Definition:** A workload where the database receives a large number of read operations.

Example:

```text
10,000 requests/sec

9,500 reads
  500 writes
```

Even optimized queries can overload one database because DB resources are finite:

* CPU
* Memory
* Disk I/O
* Connections
* Query-processing capacity

For read-heavy workloads:

```text
Primary DB
    ↓
Read Replicas
```

can distribute suitable reads.

---

# High Write Traffic

**Definition:** A workload where database writes become the major source of database load.

Example:

```text
10,000 requests/sec

2,000 reads
8,000 writes
```

Typical approaches:

* Optimize writes
* Batch writes where appropriate
* Async processing for suitable workloads
* Sharding when necessary

Kafka can buffer/smooth suitable workloads, but it does **not magically increase sustained database write capacity**.

### Recall

```text
Read-heavy
    ↓
Read replicas may help


Write-heavy
    ↓
Optimize writes / batching /
async processing / sharding where appropriate
```

---

# Read-Heavy vs Write-Heavy

| Workload    | Typical scaling consideration                                              |
| ----------- | -------------------------------------------------------------------------- |
| Read-heavy  | Read replicas                                                              |
| Write-heavy | Write optimization, batching, async processing, sharding where appropriate |

Always:

> **Find the bottleneck first.**

---

# Connection Pool Pressure

**Definition:** Connection pool pressure occurs when application requests need more DB connections than are currently available in the connection pool.

Example:

```text
Pool size = 20

20 connections → busy
100 requests → waiting
```

The DB itself may not necessarily be overloaded; the application may simply be waiting for available connections.

### Identification Flow

```text
API latency
    ↓
DB-related latency
    ↓
Are requests waiting for connections?
    ↓
Check connection pool metrics
    ↓
Active connections
Idle connections
Pending requests
Pool utilization
    ↓
Investigate underlying cause
```

---

## Connection Pool Metrics

Useful signals:

* Active connections
* Idle connections
* Pending/waiting requests
* Pool utilization
* Connection acquisition time

Example:

```text
Pool size = 20
Active = 20
Idle = 0
Pending = 150
```

Strong signal of connection pool pressure.

---

## Why Not Blindly Increase Pool Size?

Don't immediately change:

```text
20 → 100
```

because:

```text
Bigger pool
    ↓
More concurrent DB work
    ↓
More DB CPU / I/O / lock contention
    ↓
Database may become the bottleneck
```

First investigate:

```text
Connection pool pressure
        ↓
Slow queries?
Long transactions?
High concurrency?
Connection leaks?
        ↓
Fix underlying cause
        ↓
If pool is genuinely undersized
        ↓
Tune based on workload + DB capacity
```

---

# HikariCP

**Definition:** HikariCP is a JDBC database connection pool commonly used by Spring Boot.

It maintains reusable database connections so the application doesn't create a new connection for every request.

### Overall Flow

```text
Spring Boot
    ↓
JPA / Hibernate
    ↓
DataSource
    ↓
HikariCP
    ↓
JDBC Driver
    ↓
MySQL
    ↓
Database
```

### Request Flow

```text
Request
   ↓
HikariCP
   ↓
Borrow connection
   ↓
JDBC
   ↓
MySQL
   ↓
Execute SQL
   ↓
Return connection
   ↓
HikariCP
```

---

## HikariCP Configuration

Practical snapshot:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/order_db
spring.datasource.username=root
spring.datasource.password=password

spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
```

### `maximum-pool-size`

**Definition:** Maximum number of connections that the HikariCP pool can contain.

```properties
spring.datasource.hikari.maximum-pool-size=20
```

Approximately 20 DB connections can be available from that application instance.

---

### `minimum-idle`

**Definition:** Minimum idle connections HikariCP tries to maintain as a pool-management target.

```properties
spring.datasource.hikari.minimum-idle=5
```

---

### `connection-timeout`

**Definition:** Maximum time a request waits to obtain a connection from the pool.

```properties
spring.datasource.hikari.connection-timeout=30000
```

```text
30,000 ms = 30 seconds
```

Important:

```text
Connection timeout
→ Waiting for HikariCP connection

Query timeout
→ Waiting for SQL execution
```

They are different.

---

## Pool Size Per Application Instance

Pool size is **per application instance**.

Example:

```text
3 Order Service instances
        ×
20 connections each
        =
60 potential DB connections
```

```text
Instance 1 → 20
Instance 2 → 20
Instance 3 → 20
              ↓
          MySQL
       ≈ 60 potential
       connections
```

### Interview point

> I wouldn't blindly increase the connection pool size. I would first investigate slow queries, long transactions, high concurrency, connection leaks, and database capacity.

---

# 6. Read Replicas

## Primary DB vs Replica

**Definition:** The primary database handles writes, while read replicas contain replicated data and can serve suitable read traffic.

```text
                    Application
                  /            \
               WRITE            READ
                  ↓              ↓
             ┌─────────┐    ┌──────────┐
             │ Primary │───→│ Replica  │
             │   DB    │    │    DB    │
             └─────────┘    └──────────┘
```

---

## Why Read Replicas?

**Definition:** Read replicas scale read capacity by distributing suitable reads across additional database instances.

Useful for:

```text
Read-heavy workload
        ↓
Primary receives writes
        ↓
Replicas handle suitable reads
```

Multiple replicas:

```text
                       Application
                    /              \
                 WRITE              READ
                    ↓                ↓
              ┌─────────┐      ┌──────────┐
              │ Primary │─────→│ Replica 1│
              └─────────┘      └──────────┘
                    │
                    └─────────→┌──────────┐
                               │ Replica 2│
                               └──────────┘
```

---

## Replication

**Definition:** Replication copies database changes from a primary database to one or more replicas.

```text
Primary
   ↓
Replication
   ↓
Replica
```

Spring Boot does not itself create the database replicas. Database infrastructure/cloud services handle replication; the application later needs appropriate read/write routing.

---

## Replication Lag

**Definition:** Replication lag is the delay between a change being committed on the primary and becoming available on a replica.

Example:

```text
Primary → Product price = ₹900
Replica → Product price = ₹1000
```

The replica is temporarily behind.

---

## Stale Reads

**Definition:** A stale read occurs when a replica returns older data because replication has not caught up.

Example:

```text
User updates order → CONFIRMED
        ↓
Immediately reads replica
        ↓
Replica still says → PENDING
```

---

## When to Read from Primary

Use the primary when the latest committed state is important.

Examples:

* Immediately after an order update
* Payment status confirmation
* Consistency-sensitive operations

Suitable replica reads may include:

* Product browsing
* Catalog
* Search
* Reports
* Analytics-style reads
* Non-critical reads

---

## Replica Failure

**Definition:** A replica failure means that replica should be removed from read traffic and another replica or appropriate fallback can be used.

```text
Replica 1 ❌
    ↓
Route reads to Replica 2
```

---

## Primary Failure / Failover

**Definition:** Failover is the process of promoting or switching to another database instance when the primary fails, depending on the infrastructure/setup.

A replica may potentially be promoted to primary.

---

## Read Replica ≠ Backup

A replica is primarily for:

* Read scaling
* Availability/failover support

It is **not a replacement for backups**.

Replicated deletes can also reach replicas.

---

## When Read Replicas Don't Solve the Problem

Read replicas don't directly fix:

* Slow queries
* Missing indexes
* N+1 queries
* Poor query design
* Write-heavy bottlenecks

Always identify the actual bottleneck first.

---

## Application Read/Write Routing

```text
                 Application
                 /          \
                /            \
           WRITE              READ
              ↓                ↓
          Primary          Replica
              │
              └──── replication ────→ Replica
```

---

## Read Replicas vs Write-Heavy Workloads

```text
Read-heavy
    ↓
Read replicas may help


Write-heavy
    ↓
Optimize writes / batching /
async processing / sharding where appropriate
```

### Strong 3-YOE Interview Answer

> First I would identify whether the bottleneck is actually read traffic. If the workload is read-heavy and queries are already optimized and properly indexed, I would consider read replicas. Writes would continue going to the primary, while suitable reads would be distributed across replicas. I would also consider replication lag because replicas may temporarily return stale data, so consistency-sensitive reads may need to go to the primary. I would also account for replica failures and have appropriate failover or fallback mechanisms.

---

# 7. Sharding

## What is Sharding?

**Definition:** Sharding is horizontally splitting a large dataset across multiple independent database instances called shards.

```text
                    Orders
                      ↓
                 Shard Router
                /      |      \
               ↓       ↓       ↓
          ┌────────┐ ┌────────┐ ┌────────┐
          │Shard 1 │ │Shard 2 │ │Shard 3 │
          │Orders  │ │Orders  │ │Orders  │
          └────────┘ └────────┘ └────────┘
```

Example:

```text
90M orders
    ↓
30M → Shard 1
30M → Shard 2
30M → Shard 3
```

---

## Horizontal Data Partitioning

**Definition:** Horizontal partitioning splits rows of the same logical table across different shards.

```text
Shard 1 → some orders
Shard 2 → different orders
Shard 3 → different orders
```

---

## Shards as Separate DB Instances

Typically:

```text
Shard 1 → DB Instance 1
Shard 2 → DB Instance 2
Shard 3 → DB Instance 3
```

---

## Shard Key

**Definition:** A shard key is the field used to determine which shard stores a record.

Example:

```text
user_id
```

Possible range:

```text
user_id 1 - 1,000,000
        → Shard 1

user_id 1,000,001 - 2,000,000
        → Shard 2

user_id 2,000,001 - 3,000,000
        → Shard 3
```

---

## Shard Routing

**Definition:** Shard routing determines which shard should execute a query.

```text
Request
   ↓
Order Service
   ↓
Read user_id
   ↓
Shard Router
   ↓
Determine shard
   ↓
Execute query
```

Example:

```text
user_id = 1,500,000
        ↓
Shard 2
```

---

## Range-Based Sharding

**Definition:** Range-based sharding assigns ranges of shard-key values to different shards.

Example:

```text
1 - 1,000,000       → Shard 1
1,000,001 - 2,000,000 → Shard 2
2,000,001 - 3,000,000 → Shard 3
```

---

## Querying Using the Shard Key

If:

```sql
WHERE user_id = 1500
```

and `user_id` is the shard key:

```text
Request
   ↓
user_id = 1500
   ↓
Router
   ↓
Correct shard
   ↓
Query only that shard
```

Efficient because the application knows where the data belongs.

---

## Non-Shard-Key Queries

Example:

```sql
WHERE status = 'PAID'
```

If `status` isn't the shard key, the application may not know which shard contains matching records.

This can lead to:

```text
status = PAID
      ↓
 ┌────┼────┐
 ↓    ↓    ↓
S1   S2   S3
 ↓    ↓    ↓
Query each shard
```

---

## Scatter-Gather

**Definition:** Scatter-gather means sending a query to multiple shards and gathering their results.

```text
             status = PAID
                    ↓
             ┌──────┼──────┐
             ↓      ↓      ↓
          Shard 1 Shard 2 Shard 3
             ↓      ↓      ↓
           Query  Query  Query
             \      |      /
                    ↓
                 Gather
                 results
```

---

## Hot Shards

**Definition:** A hot shard receives disproportionately high traffic or data compared with other shards.

Example:

```text
Shard 1 → 90% traffic
Shard 2 → 5%
Shard 3 → 5%
```

The shard key may be causing uneven distribution.

---

## Cross-Shard Queries

**Definition:** A query requiring data from multiple shards is a cross-shard query.

These can be more expensive and complex than querying one shard.

---

## Cross-Shard Joins

**Definition:** A join involving data stored on different shards is a cross-shard join.

These are more complicated and can be expensive.

---

## Cross-Shard Transactions

**Definition:** A transaction involving multiple shards is harder to coordinate because each shard has its own database transaction.

Example:

```text
Update Shard 1 → SUCCESS
Update Shard 2 → FAILURE
```

Now coordinating rollback/commit is more complicated.

Prefer designing operations so that they stay within one shard where possible.

---

## Data Migration / Rebalancing

**Definition:** Rebalancing moves data between shards to distribute storage and workload more evenly.

Needed when:

```text
Shard 1 → overloaded
Shard 2 → lightly loaded
Shard 3 → lightly loaded
```

Migration/rebalancing adds operational complexity.

---

## Sharding Trade-offs

Benefits:

* Distributes large datasets
* Distributes database workload
* Can scale beyond one database instance

Costs:

* Routing complexity
* Cross-shard queries
* Cross-shard joins
* Cross-shard transactions
* Hot shards
* Rebalancing/data migration
* Operational complexity

---

## When NOT to Shard

Avoid unnecessary sharding when:

* Dataset is small
* Traffic is manageable
* One DB can handle workload
* Simpler optimizations haven't been exhausted

### Decision Flow

```text
Database bottleneck
       ↓
Check slow queries
       ↓
Check indexes
       ↓
Check N+1 / excessive DB calls
       ↓
Check caching
       ↓
Check read replicas for read-heavy workload
       ↓
Can one DB still handle the
data/workload?
       ↓
       NO
       ↓
Consider sharding
```

---

## Read Replica vs Sharding

```text
Read Replica
     ↓
Same data
     ↓
Scale READS
```

```text
Sharding
     ↓
Different data
     ↓
Scale DATA + WORKLOAD
```

---

## Sharding + Replication

**Definition:** Sharding splits the dataset; replication creates copies of each shard.

```text
                 Application
                     ↓
                Shard Router
                 /         \
                ↓           ↓
           Shard 1       Shard 2
              │             │
          ┌───┴───┐     ┌───┴───┐
          ↓       ↓     ↓       ↓
       Primary Replica Primary Replica
```

Example:

```text
Shard 1 → users 1–1000
          Primary + Replica

Shard 2 → users 1001–2000
          Primary + Replica
```

---

## Strong 3-YOE Sharding Answer

> Sharding can improve scalability by distributing data and workload across multiple database instances, but it also increases application complexity. We need shard-key-based routing, and cross-shard queries and transactions become more complicated. A poor shard key can also create uneven distribution or hot shards. Therefore, I would consider sharding after optimizing queries, indexes, database access patterns, and read scaling, when a single database can no longer handle the required data or workload.

---

# 8. Spring Boot / JPA Practical Database Concepts

# 8.1 Transaction Propagation

## `Propagation.REQUIRED`

**Definition:** `REQUIRED` joins an existing transaction or creates a new transaction if none exists.

```java
@Transactional(propagation = Propagation.REQUIRED)
public void saveOrderAudit() {
    // DB operation
}
```

### Existing Transaction

```text
createOrder()
     ↓
Transaction A starts
     ↓
saveOrder()
     ↓
saveOrderAudit()
     ↓
REQUIRED checks:
transaction active?
     ↓
YES
     ↓
Join Transaction A
     ↓
A commits
```

### No Existing Transaction

```text
saveOrderAudit()
     ↓
No active transaction
     ↓
REQUIRED
     ↓
Create Transaction A
     ↓
Execute
     ↓
Commit
```

Important:

> Two methods having `@Transactional` does not automatically mean two transactions.

With `REQUIRED`, they can participate in the same transaction.

---

# `REQUIRES_NEW`

**Definition:** `REQUIRES_NEW` always starts an independent transaction and suspends the existing transaction if one exists.

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAudit() {
    // independent transaction
}
```

### Short Practical Flow

```text
Transaction A
     ↓
createOrder()
     ↓
saveOrder()
     ↓
saveAudit() → REQUIRES_NEW
     ↓
SUSPEND A
     ↓
Transaction B
     ↓
saveAudit()
     ↓
COMMIT B
     ↓
RESUME A
     ↓
continue createOrder()
     ↓
COMMIT / ROLLBACK A
```

Important:

> A does not start again. It is suspended and then resumes.

Possible outcome:

```text
A commits, B commits
A rolls back, B commits
A commits, B rolls back
```

Transaction boundaries are independent, but **exception propagation is a separate concern**.

If B throws an exception and A doesn't handle it, the exception can still propagate into A.

---

## REQUIRED vs REQUIRES_NEW

|                      | REQUIRED            | REQUIRES_NEW |
| -------------------- | ------------------- | ------------ |
| Existing transaction | Joins it            | Suspends it  |
| New transaction      | Only if none exists | Always       |
| Same transaction?    | Usually yes         | No           |
| Independent commit?  | No                  | Yes          |

---

# 8.2 Spring Proxy

## What is a Spring Proxy?

**Definition:** A Spring proxy is a wrapper around a Spring-managed bean that can intercept method calls and apply framework behavior such as transaction management.

```text
You
 ↓
Spring Proxy
 ↓
Actual Service Object
```

For `@Transactional`:

```text
External call
      ↓
Spring Proxy
      ↓
Start Transaction
      ↓
Actual method
      ↓
Method finishes
      ↓
Commit / Rollback
```

---

## External Call → Spring Proxy

```java
orderService.createOrder();
```

Conceptually:

```text
External caller
      ↓
OrderService Proxy
      ↓
createOrder()
      ↓
Transaction handling
```

---

## Self-Invocation

**Definition:** Self-invocation occurs when a method calls another method on the same object using `this`.

```java
@Transactional
public void A() {
    this.B();
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void B() {
}
```

Flow:

```text
External call
     ↓
Spring Proxy
     ↓
A()
     ↓
this.B()
     ↓
B()
```

The `this.B()` call does not go back through the Spring proxy.

Therefore the proxy-based transactional interception for B is bypassed.

---

## Why `REQUIRES_NEW` Can Fail with Self-Invocation

Because:

```text
A()
 ↓
this.B()
 ↓
B()
```

doesn't go through the proxy again.

So Spring doesn't get the proxy interception opportunity to apply the `REQUIRES_NEW` behavior to that internal call.

---

## Separate Spring Bean / Service

Common practical approach:

```java
@Service
public class OrderService {

    private final AuditService auditService;

    @Transactional
    public void createOrder() {
        saveOrder();
        auditService.saveAudit();
    }
}
```

```java
@Service
public class AuditService {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void saveAudit() {
        // audit
    }
}
```

Flow:

```text
You
 ↓
OrderService Proxy
 ↓
createOrder()
 ↓
auditService.saveAudit()
 ↓
AuditService Proxy
 ↓
REQUIRES_NEW
 ↓
Suspend A → Start B
```

The important concept is not "REQUIRES_NEW requires another service."

The important point is:

> The call must go through Spring's transactional interception/proxy mechanism for proxy-based `@Transactional` behavior to be applied.

---

# 8.3 Practical Optimistic Locking

## `@Version`

**Definition:** `@Version` enables optimistic locking in JPA by maintaining a version value for an entity.

### Practical Entity

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

Hibernate manages the version.

**Do not manually increment it.**

---

## Version Column

Example:

```text
orders

id     status       version
---------------------------
101    PENDING         1
102    CONFIRMED       3
103    CANCELLED       2
```

---

## Concurrent Update Scenario

A and B both read:

```text
Order 101
version = 5
```

A updates successfully:

```text
version 5 → 6
```

Conceptually:

```sql
UPDATE orders
SET status = ?,
    version = 6
WHERE id = 101
AND version = 5;
```

Then B still has version `5`.

B attempts:

```sql
UPDATE orders
SET status = ?,
    version = 6
WHERE id = 101
AND version = 5;
```

But DB now has version `6`.

Therefore:

```text
0 rows updated
      ↓
Optimistic locking conflict
      ↓
Exception
```

---

## `@Transactional` + `@Version`

```text
@Transactional
      ↓
Load Order
      ↓
Change status
      ↓
Hibernate tracks entity
      ↓
Transaction flush
      ↓
UPDATE ... WHERE id=? AND version=?
      ↓
Version matches?
   ↙       ↘
 YES       NO
  ↓         ↓
Update    Exception
```

---

## Exception

Common Spring exception:

```text
ObjectOptimisticLockingFailureException
```

JPA-level exception:

```text
OptimisticLockException
```

---

## Handling the Conflict

```text
Optimistic Lock Exception
          ↓
   ┌──────┼──────────┐
   ↓      ↓          ↓
 Reject  Refresh    Propagate
         + retry
```

A retry should generally happen in a **fresh transaction**.

For an API, a common choice is:

```text
HTTP 409 Conflict
```

The exact API response is an application design decision.

---

## Short Real Scenario

```text
Order 101
version = 5

A + B read version 5
        ↓
A updates → version 6
        ↓
B updates using version 5
        ↓
0 rows updated
        ↓
Optimistic locking exception
        ↓
Reject / Refresh + retry / Propagate
```

---

# 8.4 Practical Pessimistic Locking

## `@Lock`

**Definition:** JPA's `@Lock` specifies the locking mode used when retrieving an entity.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Order> findById(Long id);
```

---

## `PESSIMISTIC_WRITE`

**Definition:** Requests a database-level write lock so conflicting transactions cannot freely modify the same row simultaneously.

Conceptually:

```sql
SELECT *
FROM orders
WHERE id = 101
FOR UPDATE;
```

---

## `@Transactional` + Database Lock

```java
@Transactional
public void cancelOrder(Long orderId) {

    Order order = orderRepository.findById(orderId)
            .orElseThrow();

    order.setStatus("CANCELLED");
}
```

Repository:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Order> findById(Long id);
```

Short flow:

```text
BEGIN
  ↓
SELECT ... FOR UPDATE
  ↓
🔒 Lock acquired
  ↓
Update order
  ↓
COMMIT
  ↓
🔓 Lock released
```

---

## Request A Gets Lock / Request B Waits

**Definition:** A conflicting transaction generally waits for the lock to be released, or may fail because of timeout/deadlock behavior.

```text
Request A
   ↓
@Transactional
   ↓
🔒 Lock Order 101
   ↓
Update
   ↓
COMMIT
   ↓
🔓 Lock released


Request B
   ↓
Try same row
   ↓
⏳ WAIT
   ↓
A commits
   ↓
🔓 Lock released
   ↓
B gets lock
   ↓
Reads latest state
```

### Key Interview Wording

> With pessimistic locking, a conflicting transaction generally waits for the lock to be released, or it may fail if a lock timeout or deadlock occurs.

---

## Lock Timeout

**Definition:** A lock timeout limits how long a transaction waits for a conflicting lock.

It prevents the request from waiting indefinitely, depending on database/configuration.

---

## Deadlock

**Definition:** A deadlock occurs when transactions wait for locks held by each other.

```text
Transaction A
    ↓
Lock 101
    ↓
Wait for 102


Transaction B
    ↓
Lock 102
    ↓
Wait for 101
```

```text
A waits for B
B waits for A
     ↓
Deadlock
```

The database generally detects the deadlock and aborts one transaction.

### Reduce Deadlocks

* Consistent lock ordering
* Short transactions
* Avoid unnecessary locks
* Retry the failed transaction when appropriate

### Saved Interview Answer

> No. Pessimistic locking can cause deadlocks. The database generally detects the deadlock and aborts one of the transactions, but the application should reduce deadlocks through consistent lock ordering and short transactions, and may retry the failed transaction when appropriate.

---

## Keep Locking Transactions Short

Avoid:

```java
@Transactional
public void cancelOrder(Long id) {

    Order order = repository.findById(id);

    callPaymentService();       // external call
    callAnotherService();       // external call
    heavyProcessing();

    order.setStatus("CANCELLED");
}
```

Because:

```text
DB lock
   ↓
External service waiting
   ↓
Long transaction
   ↓
Other transactions wait
   ↓
Lock / connection contention
```

### Practical Rule

> Keep pessimistic-locking transactions short and avoid unnecessary external calls while holding database locks.

---

# 8.5 Practical JPA Index

## `@Table`

**Definition:** JPA's `@Table` maps an entity to a database table and can also define table-level indexes and constraints.

---

## `@Index`

**Definition:** JPA's `@Index` defines a database index for specified columns.

### Exact Practical Snapshot

```java
@Entity
@Table(
    name = "orders",
    indexes = {
        @Index(
            name = "idx_orders_user_status",
            columnList = "user_id, status"
        )
    }
)
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "user_id")
    private Long userId;

    private String status;

    // getters and setters
}
```

This represents:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

---

## Composite Index

**Definition:** A composite index contains multiple columns in a defined order.

```text
(user_id, status)
```

Column order matters.

---

## Leftmost-Prefix Rule

For:

```text
(user_id, status)
```

Generally:

```text
user_id              → ✅
user_id + status     → ✅
status alone         → generally ❌
```

The index is organized according to its column order.

### Important Interview Wording

> Column order matters because composite indexes are organized according to their column order, and the leftmost-prefix rule determines which query patterns can efficiently use the index.

---

## Choosing Column Order

Choose the order based on:

* Query patterns
* Filtering
* Joins
* Sorting
* Selectivity
* Workload

Example:

```sql
WHERE user_id = 101
AND status = 'PAID'
```

A practical index:

```text
(user_id, status)
```

---

## JPA Index vs Database Migration

JPA:

```java
@Index(
    name = "idx_orders_user_status",
    columnList = "user_id, status"
)
```

Production database migration:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

For production, important schema changes are generally better managed through version-controlled migrations such as Flyway/Liquibase.

---

## Don't Blindly Create Many Indexes

```text
More indexes
     ↓
Potentially faster reads
     ↓
BUT
     ↓
More storage
More maintenance
More write overhead
```

Choose indexes based on actual query patterns and verify with `EXPLAIN`.

---

# 9. Spring Boot DB Configuration / HikariCP

## Datasource Configuration

**Definition:** Datasource configuration tells Spring Boot how to connect to the database.

### Exact Practical Snapshot

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/order_db
spring.datasource.username=root
spring.datasource.password=password

spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
```

---

## MySQL JDBC URL

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/order_db
```

Conceptually:

```text
jdbc:mysql://
localhost:
3306/
order_db
```

```text
JDBC
 ↓
MySQL
 ↓
localhost:3306
 ↓
order_db
```

---

## Username / Password

```properties
spring.datasource.username=root
spring.datasource.password=password
```

Used by the application to authenticate with MySQL.

---

## HikariCP Configuration

```properties
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
```

### Recall

```text
maximum-pool-size
→ Maximum pool connections

minimum-idle
→ Minimum idle connection target

connection-timeout
→ Maximum wait to obtain a pool connection
```

---

## Connection Acquisition Timeout vs Query Timeout

Important distinction:

```text
Connection timeout
        ↓
Waiting for HikariCP connection


Query timeout
        ↓
Waiting for SQL/query execution
```

They solve different problems.

---

## Pool Size Per Application Instance

Example:

```text
3 application instances
×
20 connections
=
60 potential DB connections
```

```text
Instance 1 → 20
Instance 2 → 20
Instance 3 → 20
              ↓
           MySQL
```

---

## Connection Pool Pressure Scenario

```text
maximum-pool-size = 20
active connections = 20
pending requests = 100
```

Don't immediately increase the pool.

Investigate:

```text
Slow queries?
Long transactions?
High concurrency?
Connection leaks?
DB capacity?
```

---

## Why Not Blindly Increase Pool Size?

```text
Bigger pool
    ↓
More concurrent DB work
    ↓
More CPU / I/O / locks
    ↓
Database may become bottleneck
```

### Production Rule

> First investigate why connections are busy. Increase the pool only when evidence shows the pool is genuinely undersized and the database can handle the additional concurrency.

---

# 10. Application Config vs Database Migration

## What Belongs in `application.properties`?

**Definition:** Application configuration controls how the application connects to and interacts with the database at runtime.

Examples:

```properties
spring.datasource.url=...
spring.datasource.username=...
spring.datasource.password=...

spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
```

Think:

```text
Application configuration
        ↓
"How does my application connect/use the DB?"
```

---

## Datasource Configuration

Belongs in application configuration:

* JDBC URL
* Username
* Password
* Datasource settings
* Connection pool configuration

---

## HikariCP Runtime Configuration

Examples:

```text
maximum-pool-size
minimum-idle
connection-timeout
```

These control application-side connection-pool behavior.

---

# What Belongs in Database Migrations?

**Definition:** Database migrations are version-controlled changes that modify the database schema.

Examples:

* Tables
* Columns
* Indexes
* Constraints
* Schema changes

Example:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

---

## Tables

Example migration:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    status VARCHAR(50)
);
```

---

## Columns

Schema changes such as:

```sql
ALTER TABLE orders
ADD COLUMN payment_id BIGINT;
```

belong in a database migration.

---

## Indexes

Example:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

---

## Constraints

Examples:

* Primary key
* Foreign key
* Unique constraint
* Check constraint

These are database schema concerns and should be version-controlled.

---

# Flyway / Liquibase

**Definition:** Flyway and Liquibase are database migration tools used to version and apply database schema changes.

Example Flyway-style history:

```text
V1__create_orders_table.sql
V2__add_user_status_index.sql
V3__add_payment_reference.sql
```

Flow:

```text
Application
     ↓
Flyway / Liquibase
     ↓
Run required migrations
     ↓
MySQL schema updated
     ↓
Application starts
```

---

# Why Controlled Migrations in Production?

**Definition:** Versioned migrations provide a controlled and repeatable history of database schema changes.

Benefits:

* Version control
* Repeatability
* Team coordination
* Controlled deployments
* Clear schema history

---

# `ddl-auto=update` vs Versioned Migrations

Example:

```properties
spring.jpa.hibernate.ddl-auto=update
```

Hibernate can automatically attempt schema changes based on entities.

Useful for local development, but production systems generally prefer controlled migrations.

### Recall

```text
Local development
→ ddl-auto can be convenient


Production
→ Versioned migrations
   (Flyway / Liquibase)
```

### Interview Answer

> I use application configuration for database connectivity and runtime settings such as the datasource and connection pool. For production schema changes such as tables, indexes, and constraints, I prefer version-controlled migrations such as Flyway or Liquibase instead of relying on Hibernate's automatic schema updates.

---

# 🔥 Final Quick Recall — This Entire Section

```text
DATABASE BOTTLENECK
    ↓
Slow query / Missing index / N+1 / Traffic / Pool pressure
    ↓
Identify bottleneck
    ↓
EXPLAIN / metrics / monitoring
    ↓
Optimize
```

```text
READ-HEAVY
    ↓
Optimized queries + indexes
    ↓
Read replicas
```

```text
WRITE-HEAVY / HUGE DATASET
    ↓
Optimize / batch / async where suitable
    ↓
Sharding when necessary
```

```text
REQUIRED
    ↓
Join existing transaction
    ↓
No existing → create one
```

```text
REQUIRES_NEW
    ↓
Suspend A
    ↓
Start B
    ↓
Commit/Rollback B
    ↓
Resume A
```

```text
OPTIMISTIC
    ↓
@Version
    ↓
Detect conflict during update
    ↓
Exception
```

```text
PESSIMISTIC
    ↓
PESSIMISTIC_WRITE
    ↓
🔒 Lock row
    ↓
Other conflicting transaction waits
    ↓
Commit
    ↓
🔓 Release
```

```text
JPA INDEX
    ↓
@Table + @Index
    ↓
Composite index
    ↓
Column order matters
    ↓
Leftmost-prefix rule
```

```text
SPRING BOOT
    ↓
HikariCP
    ↓
JDBC
    ↓
MySQL
```

```text
APPLICATION CONFIG
    ↓
DB URL / credentials / HikariCP


DATABASE MIGRATION
    ↓
Tables / columns / indexes / constraints
    ↓
Flyway / Liquibase
```

## ⭐ Must-Remember Practical Snapshots

### `@Version`

```java
@Version
private Long version;
```

### Pessimistic Lock

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Order> findById(Long id);
```

### JPA Composite Index

```java
@Entity
@Table(
    name = "orders",
    indexes = {
        @Index(
            name = "idx_orders_user_status",
            columnList = "user_id, status"
        )
    }
)
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "user_id")
    private Long userId;

    private String status;

    // getters and setters
}
```

### HikariCP

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/order_db
spring.datasource.username=root
spring.datasource.password=password

spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
```

### Practical Pessimistic Scenario

```text
A → @Transactional
  → PESSIMISTIC_WRITE
  → 🔒 Lock row
  → Update
  → COMMIT
  → 🔓 Release

B → tries same row
  → ⏳ Waits
  → gets lock
  → reads latest state
```

### Practical Optimistic Scenario

```text
A + B read version 5
        ↓
A updates → version 6
        ↓
B updates using version 5
        ↓
0 rows updated
        ↓
Optimistic locking exception
        ↓
Reject / Refresh + retry / Propagate
```

### `REQUIRES_NEW` Scenario

```text
Transaction A
     ↓
createOrder()
     ↓
saveAudit() → REQUIRES_NEW
     ↓
SUSPEND A
     ↓
Transaction B
     ↓
saveAudit()
     ↓
COMMIT B
     ↓
RESUME A
     ↓
COMMIT / ROLLBACK A
```

---

# 🎯 Interview Mental Model

```text
First:
Find the actual DB bottleneck
        ↓
Slow query?
Index?
N+1?
Read traffic?
Write traffic?
Connection pool?
        ↓
Optimize the actual problem
        ↓
Read-heavy → consider replicas
        ↓
Huge data/workload → consider sharding
        ↓
Concurrency → optimistic / pessimistic locking
        ↓
Spring transactions → REQUIRED / REQUIRES_NEW
        ↓
Production schema → migrations
```
