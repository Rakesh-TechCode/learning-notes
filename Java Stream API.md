# Java Stream API — Quick Recall Notes

> **Core idea:** Stream API is used to process data from collections in a clean, readable, functional-style pipeline.

---

## 1. Stream API Basics ✅

**What it is:** A Stream is a way to **process elements from a data source** without directly modifying the original collection.

### Why do we need Streams?

Traditional loop:

```java
List<String> result = new ArrayList<>();

for (String name : names) {
    if (name.startsWith("R")) {
        result.add(name.toUpperCase());
    }
}
```

Using Stream:

```java
List<String> result = names.stream()
        .filter(name -> name.startsWith("R"))
        .map(name -> name.toUpperCase())
        .toList();
```

Streams make **data-processing logic more readable and declarative**.

### Important points

* Stream is used for **processing data**.
* Stream does **not store data**.
* Collection stores data; Stream processes data.
* Stream operations generally don't modify the original collection.
* A Stream is **single-use** after a terminal operation.

### Collection vs Stream

```text
Collection
    ↓
Stores data

Stream
    ↓
Processes data
```

### Quick recall

> **Collection = data holder**
>
> **Stream = data processor**

---

# 2. Creating Streams ✅

**What it is:** Ways to create a Stream from different data sources.

### From Collection

```java
List<Integer> numbers = List.of(10, 20, 30);

numbers.stream();
```

### Using `Stream.of()`

```java
Stream<String> stream = Stream.of("Java", "Spring", "SQL");
```

### From Array

```java
int[] numbers = {10, 20, 30};

Arrays.stream(numbers);
```

### Quick recall

```text
Collection → collection.stream()

Values → Stream.of()

Array → Arrays.stream()
```

---

# 3. Stream Pipeline ✅

**What it is:** A Stream pipeline is the sequence of operations through which data flows.

A pipeline normally contains:

```text
Source
  ↓
Intermediate Operations
  ↓
Intermediate Operations
  ↓
Terminal Operation
```

### 1. Source

Where the data comes from:

```java
numbers.stream();
```

### 2. Intermediate Operation

Processes/transforms data and returns another Stream.

Examples:

```java
filter()
map()
flatMap()
distinct()
```

### 3. Terminal Operation

Finishes the Stream processing and produces a final result.

Examples:

```java
toList()
count()
forEach()
```

### Example

```java
List<Integer> result = numbers.stream()
        .filter(n -> n > 10)
        .map(n -> n * 2)
        .toList();
```

Pipeline:

```text
numbers
   ↓
stream()
   ↓
filter()
   ↓
map()
   ↓
toList()
```

---

## Lazy Evaluation ✅

**What it is:** Intermediate operations don't execute immediately. They execute when a **terminal operation** is called.

Example:

```java
numbers.stream()
        .filter(n -> n > 10)
        .map(n -> n * 2);
```

Nothing actually processes the elements yet.

When we add:

```java
.toList();
```

the pipeline executes.

### Quick recall

> **Intermediate operations prepare the pipeline.**
>
> **Terminal operation triggers execution.**

---

# 4. `filter()` ✅

**What it is:** `filter()` **selects elements** based on a condition.

```java
List<Integer> result = numbers.stream()
        .filter(n -> n > 10)
        .toList();
```

If:

```text
[5, 15, 20, 8]
```

Result:

```text
[15, 20]
```

### Important points

* `filter()` is an **intermediate operation**.
* Uses a condition/predicate.
* Keeps elements that satisfy the condition.
* Removes elements that don't satisfy it.
* Does **not transform** the element.

### Mental model

```text
filter()
    ↓
"Should I keep this element?"
```

### Quick recall

> **`filter()` = select**

---

# 5. `map()` ✅

**What it is:** `map()` **transforms each element into another value/object**.

### Basic example

```java
List<Integer> result = numbers.stream()
        .map(n -> n * 2)
        .toList();
```

```text
10 → 20
20 → 40
30 → 60
```

### One input → One output

```text
Employee → Employee Name
Employee → Salary
String   → Length
Entity   → DTO
```

### It can change the type

```text
List<String>
      ↓ map()
List<Integer>
```

Example:

```java
List<Integer> lengths = names.stream()
        .map(name -> name.length())
        .toList();
```

### Employee example

```java
List<String> names = employees.stream()
        .map(employee -> employee.getName())
        .toList();
```

```text
Employee
   ↓
getName()
   ↓
String
```

### Salary example

```java
List<Double> salaries = employees.stream()
        .map(employee -> employee.getSalary())
        .toList();
```

---

## Entity → DTO with `map()`

`map()` is **not specifically a DTO operation**.

Its job is simply:

> **Take one element and transform it into another value/object.**

### Constructor approach

```java
List<EmployeeDto> result = employees.stream()
        .map(employee -> new EmployeeDto(
                employee.getId(),
                employee.getName()
        ))
        .toList();
```

### Setter approach

```java
List<EmployeeDto> result = employees.stream()
        .map(employee -> {
            EmployeeDto dto = new EmployeeDto();

            dto.setId(employee.getId());
            dto.setName(employee.getName());

            return dto;
        })
        .toList();
```

Here we explicitly map:

```text
employee.getName()
        ↓
dto.setName(...)

employee.getId()
        ↓
dto.setId(...)
```

### ObjectMapper approach

```java
List<EmployeeDto> result = employees.stream()
        .map(employee ->
                objectMapper.convertValue(employee, EmployeeDto.class))
        .toList();
```

Here **ObjectMapper performs the field conversion/mapping**, while `map()` controls the overall transformation.

### Mental model

```text
map()
  ↓
"What should this element become?"
```

### Quick recall

> **`map()` = transform**

---

# 6. `flatMap()` ✅

**What it is:** `flatMap()` is used when one input can produce **multiple elements**, and we want to combine them into one flat stream.

---

## First understand "flatten"

Nested:

```text
[
    [1, 2],
    [3, 4]
]
```

Flattened:

```text
[1, 2, 3, 4]
```

**Flatten = remove one level of nesting.**

---

## `map()` vs `flatMap()`

### `map()`

```text
one → one
```

Example:

```text
Employee → EmployeeDto
```

### `flatMap()`

```text
one → many → combine
```

Example:

```text
Employee → [Java, Spring]
Employee → [SQL, Docker]

             ↓ flatMap()

[Java, Spring, SQL, Docker]
```

---

## Basic example

```java
List<List<Integer>> numbers = List.of(
        List.of(1, 2),
        List.of(3, 4)
);
```

Using `map()`:

```java
numbers.stream()
        .map(list -> list)
        .toList();
```

Result:

```text
[[1, 2], [3, 4]]
```

Still nested.

Using `flatMap()`:

```java
numbers.stream()
        .flatMap(list -> list.stream())
        .toList();
```

Result:

```text
[1, 2, 3, 4]
```

---

## Backend-style example

Each Employee has multiple skills:

```java
class Employee {
    List<String> skills;
}
```

Get all skills:

```java
List<String> skills = employees.stream()
        .flatMap(employee -> employee.getSkills().stream())
        .toList();
```

Flow:

```text
Employee 1 → [Java, Spring]
Employee 2 → [SQL, Docker]
Employee 3 → [Kafka, AWS]

                  ↓ flatMap()

[Java, Spring, SQL, Docker, Kafka, AWS]
```

### Quick recall

> **`flatMap()` = multiple values + flatten/combine**

---

# 7. `distinct()` 🔄

**What it is:** `distinct()` removes **duplicate elements** from a Stream.

Example:

```java
List<Integer> numbers =
        List.of(1, 2, 2, 3, 4, 4, 5);
```

```java
List<Integer> result = numbers.stream()
        .distinct()
        .toList();
```

Result:

```text
[1, 2, 3, 4, 5]
```

### Important points

* `distinct()` is an **intermediate operation**.
* Removes duplicate elements.
* Useful when you need only unique values.
* Duplicate determination relies on `equals()` and `hashCode()`.

### Example with departments

```java
List<String> departments = List.of(
        "IT", "HR", "IT", "Finance", "HR"
);

List<String> uniqueDepartments = departments.stream()
        .distinct()
        .toList();
```

Result:

```text
[IT, HR, Finance]
```

### Quick recall

> **`distinct()` = remove duplicates**

---

# 🔥 Most Important Difference So Far

This is the part I would memorize for interviews:

```text
filter()
   ↓
SELECT
"Should I keep it?"

map()
   ↓
TRANSFORM
"What should it become?"

flatMap()
   ↓
FLATTEN
"Does it produce multiple values that I need to combine?"

distinct()
   ↓
UNIQUE
"Remove duplicates"
```

### Example combining them

```java
List<String> skills = employees.stream()
        .filter(employee -> employee.getSalary() > 50000)
        .flatMap(employee -> employee.getSkills().stream())
        .distinct()
        .toList();
```

Read it like English:

```text
Get employees
     ↓
keep employees whose salary > 50000
     ↓
get all their skills
     ↓
flatten all skills
     ↓
remove duplicate skills
     ↓
make a List
```

---

# 📊 Current Learning Progress

| Topic                                       | Status                          |
| ------------------------------------------- | ------------------------------- |
| Stream API Basics                           | ✅ Done                          |
| Creating Streams                            | ✅ Done                          |
| Stream Pipeline                             | ✅ Done                          |
| Lazy Evaluation                             | ✅ Done                          |
| `filter()`                                  | ✅ Done                          |
| `map()`                                     | ✅ Done                          |
| Entity → DTO with `map()`                   | ✅ Done                          |
| `flatMap()`                                 | ✅ Done                          |
| `distinct()`                                | 🔄 Started — practice remaining |
| `sorted()`                                  | ⏳ Next                          |
| `limit()` / `skip()`                        | ⏳                               |
| `findFirst()` / `findAny()`                 | ⏳                               |
| `anyMatch()` / `allMatch()` / `noneMatch()` | ⏳                               |
| `count()`                                   | ⏳                               |
| `min()` / `max()`                           | ⏳                               |
| `reduce()`                                  | ⏳                               |
| `collect()`                                 | ⏳                               |
| `groupingBy()` / `partitioningBy()` / etc.  | ⏳                               |
| Real interview problems                     | ⏳                               |
| Optional + Streams                          | ⏳                               |
| Primitive Streams                           | ⏳                               |
| Parallel Streams                            | ⏳                               |
| Common mistakes & limitations               | ⏳                               |

---

# 🧠 Your Current Stream Mental Model

```text
                STREAM API
                    │
                    ↓
                 Source
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
       filter()             map()
       SELECT              TRANSFORM
          │                   │
          └─────────┬─────────┘
                    ↓
                flatMap()
                 FLATTEN
                    ↓
                distinct()
                 UNIQUE
                    ↓
              Terminal Operation
                    ↓
                Final Result
```
