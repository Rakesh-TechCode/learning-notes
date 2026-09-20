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

# 11. Availability ⭐

## What is Availability?

Availability = How often the system is up and usable when users need it.

Example:

```text
3 Order Service instances

Order-1 ✅
Order-2 ❌
Order-3 ✅

        ↓

System is still available
because Order-1 and Order-3 can serve requests.
```

## Measurement

Availability is expressed as a **percentage of expected time the system is available**.

Example:

```text
Expected uptime = 100 hours
Downtime       = 1 hour

Availability = 99%
```

> More **9s** → less allowed downtime.

## High Availability

To improve availability:

* Multiple service instances
* Load Balancer
* Health checks
* Avoid Single Points of Failure
* Database redundancy/failover

## SPOF — Single Point of Failure

A component whose failure can make the system unavailable.

Example:

```text
App-1 ─┐
App-2 ─┼──→ Single DB ❌
App-3 ─┘
```

If the only DB goes down, the application may become unavailable.

## Interview shortcut

> **Availability = Can users access/use the system?**

---

# 12. Reliability ⭐

## What is Reliability?

Reliability = Ability of the system to perform its intended job correctly and consistently over time.

## Availability vs Reliability

```text
Availability → Can I use it?
Reliability  → Can I trust it to work correctly?
```

Example:

Payment Service is running and accepting requests:

```text
Availability ✅
```

But it occasionally charges the customer twice:

```text
Reliability ❌
```

## Basic ways to improve reliability

### Retry

> Try again when a temporary failure occurs.

```text
Request → Failed
             ↓
           Retry
             ↓
           Success
```

### Timeout

> Stop waiting after a defined amount of time.

Useful when another service is slow/unresponsive.

### Failure Isolation

> One component's failure should have limited impact on other components.

Example:

```text
Notification Service ❌
        ↓
Order Service should ideally continue
```

### Idempotency

> Repeating the same operation should not create an unwanted duplicate effect.

Example:

```text
Create Order request
        ↓
Network problem
        ↓
Client retries
        ↓
Same order should NOT be created twice
```

## Interview shortcut

Reliability = Can I trust the system to do the correct thing consistently?

---

# 13. Performance ⭐

## What is Performance?

Performance = How efficiently and quickly the system handles a given workload.

Two important performance metrics:

```text
Performance
   ├── Latency
   └── Throughput
```

## Latency

Time taken for one request to receive a response.

Example:

```text
Order API → 5 seconds response
```

High latency → performance concern.

## Throughput

Amount of work the system processes per unit of time.

Example:

```text
1000 requests / second
```

## Performance vs Throughput

Performance is broader; throughput is one metric used to evaluate performance.

## Finding a Performance Bottleneck

Basic approach:

```text
Metrics
  ↓
Trace request
  ↓
Find slow/saturated component
  ↓
Confirm bottleneck
  ↓
Optimize / Scale
```

Possible bottlenecks:

* Application CPU
* Memory
* Database
* Slow DB query
* Network
* External API
* Connection pool
* Kafka/queue

### Example

```text
Order API CPU  → 30%
MySQL CPU      → 98%
DB query       → 3 sec
API latency    → 3.5 sec
```

MySQL is a strong **initial bottleneck candidate**, but you should confirm it rather than assuming from one metric.

## Important

High CPU is a signal, not automatically proof of a bottleneck.

---

# 14. Load Balancer ⭐⭐⭐

## What is a Load Balancer?

> A component that distributes incoming traffic across multiple service instances.

```text
               Load Balancer
              /      |      \
             ↓       ↓       ↓
         Order-1  Order-2  Order-3
```

## Why do we need it?

* Distribute traffic
* Use multiple instances
* Support horizontal scaling
* Avoid overloading one instance
* Route traffic away from unhealthy instances

---

## Client → LB → Multiple Instances

```text
Client
   ↓
Load Balancer
   ↓
┌────────┬────────┬────────┐
Order-1  Order-2  Order-3
```

The client generally doesn't need to know which particular instance handles the request.

---

## Horizontal Scaling + LB

**Horizontal scaling:**

> Add more instances.

```text
Before:

LB → Order-1
```

```text
After:

               ┌→ Order-1
Client → LB ──┼→ Order-2
               └→ Order-3
```

**Horizontal scaling adds capacity; LB distributes traffic.**

---

# 15. Health Checks

The Load Balancer periodically checks whether an instance is healthy.

Example:

```text
Order-1 → Healthy ✅
Order-2 → Unhealthy ❌
Order-3 → Healthy ✅
```

The LB stops sending **new requests** to Order-2 and uses the healthy instances.

### Typical Spring Boot setup we discussed

```text
Spring Boot Actuator
       ↓
/actuator/health
       ↑
Load Balancer checks it
```

---

# 16. Server-Side vs Client-Side Load Balancing

## Server-Side LB

A separate infrastructure component chooses the instance.

```text
Client
  ↓
AWS ALB
  ↓
Order-1 / Order-2 / Order-3
```

The external client doesn't choose the backend instance.

## Client-Side LB

The **caller/application** chooses the service instance.

```text
Order Service
      ↓
Spring Cloud LoadBalancer
      ↓
Payment-1 / Payment-2 / Payment-3
```

Here, the "client" can be another microservice.

## Important

> **Client-side doesn't necessarily mean the user's browser.**

It means **the caller that is making the service request**.

---

# 17. Load-Balancing Algorithms

## Round Robin

Requests are distributed sequentially.

```text
1 → Order-1
2 → Order-2
3 → Order-3
4 → Order-1
5 → Order-2
```

**Remember:**

> **Round Robin = Take turns**

---

## Least Connections

Send the new request to the instance with the **fewest active connections**.

```text
Order-1 → 8 connections
Order-2 → 3 connections ✅
Order-3 → 5 connections
```

New request → **Order-2**

**Remember:**

> **Least Connections = Choose the least busy by active connections**

---

## IP Hash

The client's IP is used to determine the target instance.

Conceptually:

```text
Client IP
    ↓
  Hash(IP)
    ↓
Selected Instance
```

Example:

```text
Client A → Order-2
Client A → Order-2
Client A → Order-2
```

The same client IP will **typically** map to the same instance while the mapping/available servers permit it.

**Remember:**

> **IP Hash = Client IP determines the target**

⚠️ IP Hash does not guarantee perfectly even distribution.

---

# 18. What Happens When an Instance Goes Down?

Suppose:

```text
Order-1 ✅
Order-2 ❌
Order-3 ✅
```

Health checks detect the failure.

The LB removes Order-2 from the available pool:

```text
               Load Balancer
                  /      \
                 ↓        ↓
             Order-1    Order-3
```

New requests go to the remaining healthy instances.

## Example

Before:

```text
Order-1 → ~33 req/s
Order-2 → ~33 req/s
Order-3 → ~33 req/s
```

After Order-2 fails:

```text
Order-1 → ~50 req/s
Order-3 → ~50 req/s
Order-2 → 0
```

Exact distribution depends on the LB algorithm/configuration.

---

# 19. Stateless Service + Load Balancer

A stateless service should **not depend on important state stored only inside one particular instance**.

Bad design:

```text
Order-1
   ↓
JVM HashMap
   ↓
Important session/cart state
```

If the next request goes to Order-2:

```text
Request → Order-2

Order-2 ❌ doesn't have
Order-1's local HashMap
```

Better:

```text
Order-1 ──┐
Order-2 ──┼──→ MySQL / Redis
Order-3 ──┘
```

Now any healthy instance can access the required shared state.

## Important clarification

> **Stateless ≠ No state anywhere.**

It means:

> **Don't depend on important state stored only inside one backend instance.**

Also, **not using Redis does not automatically make a service stateful**. MySQL can be the shared external storage.

---

# 20. E-Commerce Load Balancer Scenario

Our Order Service:

```text
                    Load Balancer
                   /      |      \
                  ↓       ↓       ↓
              Order-1  Order-2  Order-3
```

Normal traffic:

```text
Request → Order-1
Request → Order-2
Request → Order-3
```

Order-2 crashes:

```text
Order-1 ✅
Order-2 ❌
Order-3 ✅
```

Health check detects the failure.

```text
                    Load Balancer
                   /             \
                  ↓               ↓
              Order-1          Order-3
```

New requests continue through healthy instances.

If important state is stored externally:

```text
Order-1 ──┐
Order-3 ──┼──→ MySQL / Redis
          └──→ Shared state
```

Either instance can handle the request.

---

# 21. Spring Cloud Gateway + LoadBalancer

We discussed this practical flow:

```text
Client
  ↓
Spring Cloud Gateway
  ↓
lb://ORDER-SERVICE
  ↓
Spring Cloud LoadBalancer
  ↓
Eureka
  ↓
Order-1 / Order-2 / Order-3
```

## Remember the roles

**Eureka**

> **Where are the service instances?**

**Spring Cloud LoadBalancer**

> **Which instance should I call?**

**Spring Cloud Gateway**

> **Entry point/routing layer for incoming requests.**

Also:

```text
Gateway ≠ LoadBalancer
```

They are separate responsibilities/components, though Gateway can integrate with Spring Cloud LoadBalancer.

---

# 🧠 30-Second Revision

```text
Availability
→ Can users access the system?

Reliability
→ Can users trust it to work correctly?

Performance
→ How efficiently/quickly does it handle workload?

Latency
→ Time for one request to get a response

Throughput
→ Amount of work processed per unit time

Load Balancer
→ Distributes traffic across instances

Health Check
→ Detect unhealthy instances

Round Robin
→ Take turns

Least Connections
→ Choose fewest active connections

IP Hash
→ Client IP determines target

Server-side LB
→ Infrastructure chooses instance

Client-side LB
→ Caller/application chooses instance

Instance failure
→ Remove unhealthy instance + route to healthy ones

Stateless + LB
→ Any healthy instance can handle request
   without depending on important local state

Gateway
→ Routing/entry point

Eureka
→ Finds service instances

Spring Cloud LoadBalancer
→ Chooses service instance
```

