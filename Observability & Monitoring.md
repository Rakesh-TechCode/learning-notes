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

# Phase 3 — ELK Stack: Quick-Recall + End-to-End Local Setup

**Goal:** Collect Spring Boot JSON logs, store them in Elasticsearch, and search them in Kibana.

**Scope:** ELK fundamentals, Spring Boot integration, local Docker setup, searching, and troubleshooting. Production architecture and deployment are excluded because Topic 6 is still in progress.

---

# 1. ELK Fundamentals

**ELK = Elasticsearch + Logstash + Kibana**

- **Elasticsearch:** Indexes, stores, and searches logs.
- **Logstash:** Reads, parses, transforms, and forwards logs.
- **Kibana:** Searches, filters, and visualizes logs.
- **Filebeat:** Lightweight log shipper that can send logs to Logstash or Elasticsearch. It is not one of the three ELK components.

### Required architecture — remember this flow

**Spring Boot → Logback → JSON log file → Logstash → Elasticsearch → Kibana**

- Spring Boot generates application logs.
- Logback writes logs to a file in JSON format.
- Logstash reads and parses the file, then forwards events.
- Elasticsearch indexes and stores the events.
- Kibana lets us search the stored logs.

**Important:** Logback is NOT a component of Logstash. Logback runs inside the Spring Boot application; Logstash is a separate service.

### Why centralized logging?

In microservices, logs are spread across different services. Centralized logging lets us search multiple services from one place using fields such as `orderId`, `serviceName`, `level`, and `@timestamp`.

### ELK vs Elastic Stack

- **ELK:** Elasticsearch, Logstash, Kibana.
- **Elastic Stack:** The broader ecosystem, including tools such as Beats and Elastic Agent.

### Filebeat vs Logstash

- Filebeat collects and ships logs.
- Logstash processes and transforms events.
- Filebeat can send directly to Elasticsearch; Logstash is optional when extra processing isn't needed.

---

# 2. Spring Boot Logging Prerequisites

Complete these before connecting ELK.

### A. File logging

In `application.properties`:

```properties
logging.file.name=logs/application.log
```

Spring Boot writes logs to `logs/application.log`, relative to the application's working directory.

### B. Automatic JSON logging

Add this Maven dependency:

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>8.1</version>
</dependency>
```

Create:

`src/main/resources/logback-spring.xml`

Configure a Logback `FileAppender` with `LogstashEncoder` to write JSON logs.

**Remember:** JSON formatting is automatic. Do not manually construct JSON for every log statement.

### C. Custom fields using MDC

```java
MDC.put("orderId", orderId);
MDC.put("serviceName", "payment-service");

try {
    log.info("Processing payment");
} finally {
    MDC.remove("orderId");
    MDC.remove("serviceName");
}
```

LogstashEncoder includes MDC fields by default. The `finally` block prevents stale context from leaking into later requests when threads are reused.

### Verify the log file

```bash
tail -n 5 /YOUR/APP/PATH/logs/application.log
```

Replace `/YOUR/APP/PATH` with your project's actual location.

---

# 3. End-to-End ELK Local Setup — Step by Step

**We used:** MacBook, Docker Desktop, Docker Compose, Spring Boot, Elasticsearch, Logstash, and Kibana.

## Step 1: Identify your application and JSON log path

Our original example:

- Application: `Observability_Practice`
- Project folder: `/Users/rakeshbehera/Downloads/STS-Projects/Observability_Practice`
- Log directory: `/Users/rakeshbehera/Downloads/STS-Projects/Observability_Practice/logs`
- Log file: `application.log`

For another application, identify:

- **YOUR_APP_NAME:** `________________`
- **YOUR_PROJECT_PATH:** `________________`
- **YOUR_LOG_DIRECTORY:** `________________`
- **YOUR_LOG_FILE:** `________________`

The log directory must contain the JSON log file generated by your application.

## Step 2: Create a separate ELK practice folder

Open Terminal:

```bash
mkdir -p ~/elk-practice/logstash
cd ~/elk-practice
```

Create this structure:

```text
elk-practice/
├── compose.yaml
└── logstash/
    └── pipeline.conf
```

This folder is separate from your Spring Boot project.

## Step 3: Create `pipeline.conf`

Create the file:

`~/elk-practice/logstash/pipeline.conf`

Paste this configuration:

```ruby
input {
  file {
    path => "/app-logs/application.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
    codec => json
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "spring-boot-logs"
  }

  stdout {
    codec => rubydebug
  }
}
```

### What each part does

- `path`: File path as seen **inside the Logstash container**.
- `start_position`: Reads from the beginning when Logstash first encounters the file.
- `sincedb_path`: Tracks file-reading position; `/dev/null` discards that tracking state for our practice.
- `codec => json`: Parses each JSON log line.
- `hosts`: Elasticsearch's address on the Docker network.
- `index`: Name of the Elasticsearch index.
- `stdout`: Prints processed events to the Logstash console for debugging.

**Important:** `/dev/null` is convenient for this demo but can cause old logs to be reread after a restart. For repeated use, configure persistent `sincedb` state and mount Logstash's data directory persistently. File-position tracking does not guarantee exactly-once delivery.

## Step 4: Create `compose.yaml`

Create:

`~/elk-practice/compose.yaml`

Paste:

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.17.4
    container_name: elk-elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:8.17.4
    container_name: elk-logstash
    depends_on:
      - elasticsearch
    volumes:
      - ./logstash/pipeline.conf:/usr/share/logstash/pipeline/logstash.conf:ro
      - /YOUR/ABSOLUTE/LOG/DIRECTORY:/app-logs:ro
    networks:
      - elk

  kibana:
    image: docker.elastic.co/kibana/kibana:8.17.4
    container_name: elk-kibana
    depends_on:
      - elasticsearch
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    networks:
      - elk

volumes:
  elasticsearch_data:

networks:
  elk:
    driver: bridge
```

### The most important customization

Replace:

`/YOUR/ABSOLUTE/LOG/DIRECTORY`

with the **absolute host path** to your application's log directory.

Our original example:

```yaml
- /Users/rakeshbehera/Downloads/STS-Projects/Observability_Practice/logs:/app-logs:ro
```

If your new application writes to `/Users/you/projects/PaymentService/logs`, use:

```yaml
- /Users/you/projects/PaymentService/logs:/app-logs:ro
```

Keep the container-side directory `/app-logs` unchanged unless you also update `pipeline.conf`.

The host directory is mounted read-only (`:ro`) so Logstash can read the logs without modifying the application's files.

### Why do the versions match?

All three Elastic images use version `8.17.4` in our practice setup to keep them compatible.

**Local-only warning:** `xpack.security.enabled=false` disables Elasticsearch security. Never expose this practice configuration as a production deployment.

## Step 5: Validate and start ELK

From `~/elk-practice`:

```bash
docker compose config
```

If the configuration is valid, start the containers:

```bash
docker compose up -d
```

Check their status:

```bash
docker compose ps
```

View Logstash output:

```bash
docker compose logs --tail=100 logstash
```

You should see processed events if Logstash can read and parse your file.

## Step 6: Verify Elasticsearch

Run:

```bash
curl "http://localhost:9200"
```

You should receive Elasticsearch cluster information.

Check the index:

```bash
curl "http://localhost:9200/_cat/indices?v"
```

Look for:

`spring-boot-logs`

Count stored documents:

```bash
curl "http://localhost:9200/spring-boot-logs/_count?pretty"
```

Inspect actual log records:

```bash
curl "http://localhost:9200/spring-boot-logs/_search?pretty&size=3"
```

**Remember:** Logstash printing an event to its console proves it processed the event, but querying Elasticsearch verifies that documents are indexed.

## Step 7: Open Kibana and search logs

Open:

`http://localhost:5601`

In Kibana:

1. Open **Discover**.
2. Create a data view named `Spring Boot Logs`.
3. Set its index pattern to `spring-boot-logs`.
4. Select `@timestamp` as the time field.
5. Set the time range to include your logs.
6. Search and filter the documents.

Examples:

```text
orderId : "ORD-103"
```

```text
level : "ERROR"
```

```text
serviceName : "payment-service"
```

Combined filter:

```text
orderId : "ORD-103" AND level : "ERROR"
```

The field names must match the fields in your actual JSON logs.

<div style="border: 2px solid #16A34A; border-radius: 10px; padding: 16px; margin: 16px 0;">

## Step 8: Repeat the setup for another application

For each new Spring Boot application:

1. Configure JSON file logging.
2. Generate a few log entries.
3. Identify the application's absolute log directory and file name.
4. Update `path` in `pipeline.conf` if the file name or container-side path changes.
5. Update the host-side log directory in `compose.yaml`.
6. Choose an appropriate Elasticsearch index name.
7. Start or restart Logstash after changing its configuration.
8. Verify indexing directly in Elasticsearch.
9. Search the records in Kibana.

**Note:** Our sample Compose file uses one Logstash pipeline and one application log mount. For multiple applications, configure additional file inputs or pipelines and appropriate index routing. Don't assume the single-file configuration automatically collects every application.

</div>

### Quick command reference

```bash
# Start
docker compose up -d

# Check containers
docker compose ps

# Inspect Logstash
docker compose logs --tail=100 logstash

# Stop
docker compose down

# Stop and remove Elasticsearch's stored data too
docker compose down -v
```

Use `docker compose down -v` carefully: it removes the named Elasticsearch data volume and its stored data. Normal stopping does not require it.

---

<div style="border: 2px solid #D97706; border-radius: 10px; padding: 16px; margin: 16px 0;">

# 4. ELK Troubleshooting — 10 Questions and Answers

### Q1. Spring Boot logs aren't appearing in Kibana. What do you check?

**Answer:**

1. Confirm Spring Boot wrote new entries to the log file.
2. Check whether Logstash can read the mounted file.
3. Inspect Logstash logs.
4. Check the Elasticsearch index and document count.
5. Check Kibana's data view, time range and filters.

### Q2. Logstash reads logs but cannot connect to Elasticsearch. What do you check?

**Answer:**

- Check the Elasticsearch URL in `pipeline.conf`.
- Confirm Elasticsearch is running.
- Verify Docker networking and connectivity.
- Check connection errors in Logstash logs.

**Remember:** Inside Logstash, `localhost` refers to the Logstash container itself. Use `http://elasticsearch:9200` for our Compose network.

### Q3. Old logs appear twice after restarting Logstash. Why?

**Answer:** With `sincedb_path => "/dev/null"`, file-position state isn't retained. Logstash may reread previously processed lines. Use persistent `sincedb` state and persistent storage for normal repeated practice.

### Q4. Elasticsearch contains documents, but Kibana shows nothing. What do you check?

**Answer:**

- Correct data view and index pattern.
- Correct time range.
- Search query and filters.
- Refresh Discover and verify the timestamps in Elasticsearch.

### Q5. Elasticsearch health is yellow. Does that mean logging has stopped?

**Answer:** Not necessarily. Yellow means all primary shards are allocated but one or more replica shards are unassigned. In our single-node setup, the replica cannot be allocated to a second node. Check indexing separately.

- Green: All primary and replica shards allocated.
- Yellow: Some replicas unassigned.
- Red: At least one primary shard unassigned.

### Q6. The Logstash container has stopped. What do you do?

**Answer:**

```bash
docker compose ps -a
docker compose logs --tail=100 logstash
```

Identify the cause, fix it, and then restart Logstash:

```bash
docker compose up -d logstash
```

### Q7. Some logs written while Logstash was down are missing after restart. Why?

**Answer:** Check the saved file position, file rotation, file replacement, permissions and whether the missing entries are still available. A saved position alone does not guarantee recovery of every event.

### Q8. Some JSON logs fail to parse. What do you investigate?

**Answer:** Inspect the malformed log line and Logstash parsing errors. Verify the JSON encoder and configure suitable error handling so malformed events can be investigated instead of silently lost.

### Q9. Elasticsearch is running out of disk space and rejecting new logs. What do you do?

**Answer:** Confirm disk usage and indexing errors. Address capacity, indexed-data retention and lifecycle policies safely. Deleting the original application log file does not automatically delete indexed Elasticsearch documents.

### Q10. How do you confirm Elasticsearch received logs from Logstash?

**Answer:**

```bash
curl "http://localhost:9200/_cat/indices?v"
curl "http://localhost:9200/spring-boot-logs/_count?pretty"
curl "http://localhost:9200/spring-boot-logs/_search?pretty&size=3"
```

Check that the index exists, its document count increases, and actual documents contain the expected fields.

</div>

<div style="border: 2px solid #2563EB; border-radius: 10px; padding: 16px; margin: 16px 0;">

# 5. Final Quick Recall

- **Elasticsearch:** Stores, indexes and searches logs.
- **Logstash:** Reads, processes and forwards events.
- **Kibana:** Searches and visualizes indexed logs.
- **Filebeat:** Collects and ships logs.
- **Logback:** Generates and writes application logs from Spring Boot.
- **MDC:** Adds contextual fields such as `orderId` and `serviceName`.
- **JSON logging:** Makes log fields easier to search and filter.
- **`sincedb`:** Tracks file-reading progress; it is not an exactly-once guarantee.
- **Docker service name:** Use `elasticsearch:9200` from Logstash in our Compose network.
- **Kibana data view:** Must match the Elasticsearch index pattern.
- **Yellow health:** Replica allocation issue; not automatic proof of ingestion failure.
- **Best debugging habit:** Check each stage independently—source file → Logstash → Elasticsearch → Kibana.

**Phase 3 status:** ELK fundamentals, Spring Boot integration, local hands-on, searches and 10 troubleshooting scenarios are complete. Production ELK architecture and deployment remain in progress and are intentionally not covered in these notes.

</div>
