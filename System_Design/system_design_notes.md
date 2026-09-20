# System Design — Quick Revision Notes

> **Goal:** Short notes for quick recall. You should be able to revise the concepts below without relearning the theory.

---

## 1. What is System Design?

* Designing **components, communication, data storage, and architecture** of a software system.
* Main goal: satisfy **functional + non-functional requirements**.
* We also consider **scale, performance, availability, reliability, and failures**.

**Think:**

`Requirements → Architecture → Communication → Data → Scale → Failure Handling`

---

## 2. HLD vs LLD

### HLD — High-Level Design

* Focus: **overall system architecture**
* Defines major components and how they communicate.
* Example: `API Gateway → Order Service → MySQL`

### LLD — Low-Level Design

* Focus: **internal implementation of a component/service**
* Classes, interfaces, methods, relationships, etc.
* Example: `Controller → Service → Repository`

**Remember:**

`HLD = What components + how they communicate`

`LLD = How a component is implemented`

---

## 3. Functional Requirements

* **What should the system do?**
* Represents features/behaviour.
* Example:

  * User can add product to cart.
  * User can place order.
  * User can cancel order.

**Shortcut:** `WHAT? → Functional`

---

## 4. Non-Functional Requirements

* **How well should the system perform?**
* Examples:

  * Scalability
  * Availability
  * Reliability
  * Performance
  * Low latency
  * Security

**Shortcut:** `HOW WELL? → Non-Functional`

---

## 5. Trade-off

* **Getting one benefit while accepting a disadvantage somewhere else.**
* There is usually no perfect solution.
* Example:

  * Kafka → asynchronous + decoupled
  * Trade-off → more complexity + eventual consistency

**Remember:**

> **"If I choose this, what am I giving up?"**

---

## 6. Scale / Scaling

### Scale

* **How much load the system needs to handle.**
* Load can mean users, requests/sec, transactions, data, messages, etc.

### Scaling

* **Making the system capable of handling increased load.**

---

## 7. Vertical Scaling

* Increase resources of an **existing machine**.
* Example:

`4 CPU + 8 GB RAM → 16 CPU + 64 GB RAM`

**Remember:**

> **Make one machine bigger.**

---

## 8. Horizontal Scaling

* Add **more machines/instances**.

```text
          ┌→ Instance 1
Users → LB├→ Instance 2
          └→ Instance 3
```

* Useful for handling increased application traffic.

**Remember:**

> **Add more machines.**

---

## 9. Bottleneck ⭐

* **The component that limits the overall system's capacity/performance.**
* Example:

```text
App → MySQL
       ↑
   Bottleneck
```

If App servers can handle `50K req/sec` but DB can handle only `5K req/sec`, the **DB is the bottleneck**.

**Important logic:**

> Don't blindly scale every component. **Find the bottleneck first.**

---

## 10. Stateless Service ⭐

* A service that **doesn't depend on important state stored only inside one particular instance**.
* Any instance can generally handle the request.

```text
             ┌→ App 1
User → LB ───┼→ App 2
             └→ App 3
```

* Shared state can be kept in external systems like **DB/Redis**.
* Stateless services make **horizontal scaling easier**.

**Remember:**

> `Stateless ≠ No data/state anywhere`
>
> It means **don't depend on one instance's local state**.

---

# 🧠 30-Second Revision

```text
System Design
→ Design the overall system

HLD
→ Components + communication

LLD
→ Internal implementation

Functional
→ WHAT system does

Non-Functional
→ HOW WELL it does it

Scale
→ Amount of load

Scaling
→ Handle increasing load

Vertical
→ Bigger machine

Horizontal
→ More machines

Bottleneck
→ Limiting component

Stateless
→ Any instance can handle requests without depending
   on important local state
```
