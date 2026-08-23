Yep dude 👍 I cleaned it up into **plain GitHub-compatible Markdown** — no unnecessary `id` attributes or empty code fences. You can directly paste this into your `.md` file. The content is based on your uploaded Kafka notes. 

# 📘 Kafka Notes — Part 1

### Practical Beginner → Notification Service Foundation

> **Goal:** These notes contain **only what we have covered so far**.
>
> We will add new sections later as we learn them.

---

# 1. What is Apache Kafka?

Kafka is a **distributed event streaming platform** used to send, store, and process events/messages.

Instead of services communicating directly:

```text
Order Service ──────────→ Inventory Service
```

Kafka allows asynchronous communication:

```text
Order Service
     │
     │ Event
     ▼
   Kafka
     │
     ▼
Inventory Service
```

This creates **loose coupling** between services.

---

# 2. Producer

A **Producer** is an application that sends records/events to Kafka.

Example:

```text
Order Service
     │
     │ OrderCreated
     ▼
   Kafka
```

Example record:

```text
Key   : customer-101
Value : Order-101
```

The producer doesn't need to directly call the consumer.

---

# 3. Consumer

A **Consumer** reads records from Kafka.

```text
Kafka
  │
  │ OrderCreated
  ▼
Notification Service
```

In Spring Kafka, we commonly consume using:

```java
@KafkaListener(
    topics = "demand",
    groupId = "inventory-group"
)
```

---

# 4. Topic

A **Topic** is a logical category/name where Kafka stores records.

Example:

```text
order-events
notification-events
payment-events
```

Think of a topic as:

> **A stream/category of related events.**

Example:

```text
Topic: demand

Order-1
Order-2
Order-3
Order-4
```

---

# 5. Topic Architecture

A topic can contain multiple partitions:

```text
                 Topic: demand
              ┌───────────────┐
              │   Partition 0 │
              │ Order-1       │
              │ Order-2       │
              └───────────────┘

              ┌───────────────┐
              │   Partition 1 │
              │ Order-6       │
              │ Order-7       │
              └───────────────┘

              ┌───────────────┐
              │   Partition 2 │
              │               │
              └───────────────┘
```

Partitions allow Kafka to **scale** message processing.

---

# 6. Why Partitions?

Suppose millions of records arrive.

Putting everything into one partition limits scalability.

Instead:

```text
              Topic
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
       P0      P1      P2
```

Multiple consumers can process different partitions in parallel.

### Important:

> **Partitions are the main mechanism Kafka uses for scalability and parallelism.**

---

# 7. Kafka Record

A Kafka record can contain:

```text
Key
Value
Partition
Offset
Timestamp
```

Example:

```text
Key       : customer-101
Value     : Order-1
Partition : 0
Offset    : 33
```

The **key is optional**.

The **value contains the actual event/data**.

---

# 8. Kafka Key

A producer can send a record with a key:

```text
customer-101 → Order-1
customer-101 → Order-2
customer-101 → Order-3
```

Kafka uses the key to determine the partition.

Therefore, records with the same key normally go to the same partition.

```text
customer-101
      ↓
Partition 0

customer-101
      ↓
Partition 0
```

---

# 9. Key and Ordering

Kafka guarantees ordering **within a partition**.

Example:

```text
Partition 0

Order-1
Order-2
Order-3
Order-4
```

They are processed in that partition's order.

If related events need ordering, using the same key helps keep them in the same partition.

### Important:

> Kafka does **not** guarantee one global order across all partitions.

---

# 10. Different Keys Can Share a Partition

Suppose there are only 3 partitions:

```text
customer-101 → P0
customer-102 → P0
customer-103 → P1
```

Multiple keys can naturally end up in the same partition.

This is **not a collision**.

It happens because many keys are distributed across a limited number of partitions.

---

# 11. Offset

An **offset is the position/sequence number of a record inside a partition.**

Example:

```text
Partition 1

Offset 0 → Order-6
Offset 1 → Order-7
Offset 2 → Order-8
Offset 3 → Order-9
```

Offsets are maintained **separately for each partition**.

So:

```text
P0 → 0, 1, 2, 3...
P1 → 0, 1, 2, 3...
P2 → 0, 1, 2, 3...
```

---

# 12. Offset Is Not Global

Suppose:

```text
P0 → Offset 0, 1, 2
P1 → Offset 0, 1, 2
P2 → Offset 0, 1, 2
```

The offset `2` in P0 and offset `2` in P1 are different records.

Therefore:

> **Offset belongs to a specific partition.**

---

# 13. Consumer Group

A **Consumer Group** is a group of consumers working together to consume a topic.

Example:

```text
             inventory-group
                    │
             ┌──────┴──────┐
             ▼             ▼
         Consumer-1    Consumer-2
```

Consumers in the **same group share the partitions**.

This allows parallel processing.

---

# 14. Partition + Consumer Rule

Within one consumer group:

> **One partition is assigned to only one consumer at a time.**

Example:

```text
3 Partitions + 2 Consumers

Consumer-1 → P0, P1
Consumer-2 → P2
```

With:

```text
3 Partitions + 3 Consumers
```

each consumer can get one partition.

With:

```text
3 Partitions + 4 Consumers
```

one consumer will have no partition assigned.

---

# 15. Consumer Group Architecture

```text
                    Topic
               ┌──────┼──────┐
               ▼      ▼      ▼
              P0     P1     P2
               │      │      │
               └──┬───┴──┬───┘
                  │      │
              Consumer Consumer
                  1      2

               Consumer Group
               inventory-group
```

The group allows Kafka to distribute work among consumers.

---

# 16. `auto.offset.reset`

This setting determines **where a consumer starts when no valid committed offset exists**.

```properties
spring.kafka.consumer.auto-offset-reset=earliest
```

means:

> Start from the earliest available record.

```properties
spring.kafka.consumer.auto-offset-reset=latest
```

means:

> Start from the latest/end position and wait for new records.

---

# 17. `earliest` vs `latest`

Suppose the topic already contains:

```text
Offset 0 → Order-1
Offset 1 → Order-2
Offset 2 → Order-3
```

### `earliest`

```text
Order-1
Order-2
Order-3
```

### `latest`

Existing records are skipped and the consumer waits for new records.

### Important:

> `earliest/latest` matters when there is **no valid committed offset**.

---

# 18. Committed Offset

A committed offset tells Kafka:

> **"This consumer group has progressed up to this position."**

Example:

```text
Order-25 → Offset 19
              ↓
     Committed position → 20
```

The next time the consumer starts, it can continue from position `20`.

This is why your consumer restarted with **Order-26**, rather than replaying Order-1.

---

# 19. Offset vs Committed Offset

Don't confuse these:

### Record Offset

```text
Order-26 → Offset 20
```

Means:

> Order-26 is stored at position 20.

### Committed Offset

```text
20
```

means:

> Consumer group's saved progress/position.

Usually, after successfully processing offset 20, the committed position can become **21**, meaning the next record to consume is 21.

---

# 20. Auto Commit

Kafka consumers can automatically commit offsets.

```properties
spring.kafka.consumer.enable-auto-commit=true
```

Kafka periodically commits the consumer's progress.

You don't manually call:

```java
acknowledgment.acknowledge();
```

for basic auto-commit behavior.

---

# 21. Manual Offset Management

For our practice, we configured:

```properties
spring.kafka.consumer.enable-auto-commit=false
spring.kafka.listener.ack-mode=MANUAL
```

This gives the application more control over when processing is acknowledged.

Listener:

```java
public void consume(
    ConsumerRecord<String, String> record,
    Acknowledgment acknowledgment) {

    // process record

    acknowledgment.acknowledge();
}
```

---

# 22. What is Acknowledgment?

```java
acknowledgment.acknowledge();
```

means:

> **"I have successfully processed this record; acknowledge my progress."**

Flow:

```text
Kafka
  ↓
Receive record
  ↓
Process record
  ↓
SUCCESS
  ↓
acknowledge()
  ↓
Offset progress can be committed
```

Without acknowledgment in our **MANUAL** configuration, the record wasn't being acknowledged for commit.

---

# 23. Why Acknowledgment Is Important

Imagine:

```text
Kafka
  ↓
OrderCreated
  ↓
Update database
  ↓
SUCCESS
  ↓
acknowledge()
```

But if processing fails:

```text
Kafka
  ↓
OrderCreated
  ↓
Database failure 💥
```

You don't want to falsely mark the record as successfully processed.

This concept becomes important for:

* Retries
* Duplicate processing
* DLT
* Idempotency

---

# 24. Your Current Consumer Configuration

Our practice configuration is:

```properties
spring.kafka.bootstrap-servers=localhost:9092

spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.enable-auto-commit=false

spring.kafka.listener.ack-mode=MANUAL

spring.kafka.consumer.key-deserializer=\
org.apache.kafka.common.serialization.StringDeserializer

spring.kafka.consumer.value-deserializer=\
org.apache.kafka.common.serialization.StringDeserializer
```

And:

```java
@KafkaListener(
    topics = "demand",
    groupId = "inventory-group"
)
```

---

# 25. What We Practically Proved

We created:

```text
demand
├── P0
├── P1
└── P2
```

We produced records using keys:

```text
customer-101 → P0
customer-102 → P1
```

We observed offsets increasing:

```text
P0 → 0,1,2,3,4
P1 → 0,1,2,3...
```

Then after manual acknowledgment, the consumer restarted from its **committed position**, not from the beginning.

---

# 26. Consumer Lag

**Lag** tells us how far behind a consumer group is from the latest available records.

The basic idea is:

```text
Lag = Log End Offset - Current/Committed Offset
```

Example:

```text
Current Offset = 25
Log End Offset = 30

Lag = 30 - 25
    = 5
```

Meaning the consumer has **5 records of work remaining**.

---

# 27. Your `demand` Example

You observed:

```text
Partition 0
CURRENT-OFFSET = 5
LOG-END-OFFSET = 5
LAG = 0
```

Therefore:

```text
5 - 5 = 0
```

Consumer is caught up.

For Partition 1:

```text
CURRENT-OFFSET = 25
LOG-END-OFFSET = 25
LAG = 0
```

Again, the consumer is caught up.

---

# 28. Empty Partition

You also observed:

```text
Partition 2

CURRENT-OFFSET = -
LOG-END-OFFSET = 0
LAG = -
```

This happened because there were **no records in Partition 2**.

So there was no committed consumer position for that partition.

This is normal.

---

# 29. Collision — What We Learned

Kafka doesn't have a major concept called "collision" like `HashMap`.

These are different concepts:

```text
Same key → same partition        ≠ collision
Different keys → same partition  ≠ collision
Duplicate processing             ≠ collision
Partition reassignment           ≠ collision
```

The important real-world problem for us is **duplicate processing**, which leads to **idempotency**.

---

# 🛠️ Kafka Commands We Have Used

## 30. Show Consumer Group Offset & Lag

### Purpose:

Shows committed/current offset, latest available offset and lag.

```bash
./kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group inventory-group \
  --describe
```

Look at:

```text
CURRENT-OFFSET
LOG-END-OFFSET
LAG
```

---

## 31. Read Topic From Beginning

### Purpose:

Read available records starting from the earliest available record.

```bash
./kafka-console-consumer.sh \
  --topic demand \
  --from-beginning \
  --bootstrap-server localhost:9092
```

This is useful for **observing/testing the topic**.

---

## 32. Important Difference

Don't confuse:

```text
--from-beginning
```

with:

```properties
auto-offset-reset=earliest
```

`--from-beginning` is a **console consumer option**.

`auto-offset-reset=earliest` is a **consumer behavior when no valid committed offset exists**.

---

# 🧠 Kafka Mental Model — Current Level

Keep this picture in your mind:

```text
                         KAFKA
                           │
                     ┌─────▼─────┐
                     │   Topic   │
                     │   demand  │
                     └─────┬─────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Partition 0  Partition 1  Partition 2
              │            │            │
           Offset 0      Offset 0      Offset 0
           Offset 1      Offset 1      Offset 1
           Offset 2      Offset 2      ...
              │            │
              └──────┬─────┘
                     ▼
              Consumer Group
              inventory-group
                     │
                ┌────┴────┐
                ▼         ▼
           Consumer-1  Consumer-2
```

### ⭐ The most important concepts so far:

```text
Topic
  ↓
Partitions
  ↓
Keys
  ↓
Offsets
  ↓
Consumer Groups
  ↓
Committed Offsets
  ↓
Acknowledgment
  ↓
Consumer Lag
```

---

# 🚀 What Comes Next

We are **not** jumping to the full advanced Kafka list yet.

Our next practical section is:

```text
Consumer Groups
      ↓
Multiple Consumers
      ↓
Partition Assignment
      ↓
Rebalancing
      ↓
Error Handling
      ↓
Retries + Backoff
      ↓
Dead Letter Topic
      ↓
Idempotency
      ↓
Delivery Semantics
      ↓
Consumer Lag / Monitoring
      ↓
Notification Service
```
