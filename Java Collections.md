# Java Collections — Quick Recall Notes

---

# 1. Collection Framework Basics

### What is the Collection Framework?

The **Java Collection Framework** is a set of interfaces and classes used to **store, organize, and manipulate groups of objects**.

```text
Iterable
   ↓
Collection
   ├── List
   ├── Set
   └── Queue

Map   ← separate from Collection
```

---

## Why do we need Collections?

Arrays have a **fixed size** and provide limited built-in operations.

Collections provide:

* Dynamic size
* Ready-made methods
* Different data structures
* Easier searching, insertion, removal, etc.

**Example:**

```java
List<String> names = new ArrayList<>();
names.add("Rakesh");
names.add("Amit");
names.remove("Amit");
```

---

## Array vs Collection

| Array                         | Collection              |
| ----------------------------- | ----------------------- |
| Usually fixed size            | Dynamic size            |
| Limited operations            | Many ready-made methods |
| Can store primitives directly | Stores objects          |
| Less flexible                 | More flexible           |

---

## `Collection` vs `Collections`

### `Collection`

`Collection` is an **interface** that represents a group of objects.

```java
Collection<String> names;
```

### `Collections`

`Collections` is a **utility class** containing helper methods.

```java
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
```

### Easy memory trick

```text
Collection  → interface
Collections → utility class
```

---

## `Iterable`

`Iterable` is the top-level interface that allows an object to be **iterated/traversed**.

Important method:

```java
iterator()
```

It also provides:

```java
forEach()
```

---

## Main Collection Interfaces

### `List`

> Ordered collection that allows duplicates and index-based access.

### `Set`

> Collection that does not allow duplicate elements.

### `Queue`

> Collection generally used for processing elements in a particular order, commonly FIFO.

### `Map`

> Stores data as **key-value pairs**.

---

## Why is `Map` separate?

`Collection` represents individual elements:

```text
A
B
C
```

`Map` represents relationships:

```text
1 → Rakesh
2 → Amit
3 → Rahul
```

Therefore:

```text
Collection → elements
Map        → key + value
```

`Map` is part of the **Java Collections Framework**, but it does **not** extend `Collection`.

---

## Interface Reference vs Implementation

Preferred:

```java
List<String> list = new ArrayList<>();
```

Here:

```text
List       → reference/interface type
ArrayList  → actual object/implementation
```

This is called **programming to an interface**.

It allows easier implementation changes:

```java
List<String> list = new ArrayList<>();
```

can later become:

```java
List<String> list = new LinkedList<>();
```

without changing code that only depends on `List` operations.

---

# 2. List

### What is List?

`List` is an ordered collection that:

* Maintains order
* Allows duplicates
* Supports index-based access

```java
List<String> names = new ArrayList<>();

names.add("Rakesh");
names.add("Amit");
names.add("Amit");
```

Output:

```text
[Rakesh, Amit, Amit]
```

---

## Common List Operations

```java
list.add("A");       // add
list.get(0);         // read
list.set(0, "B");    // replace
list.remove(0);      // remove
```

---

# 3. ArrayList

### What is ArrayList?

`ArrayList` is a **dynamic-array-based implementation of List**.

Internally it uses a **resizable backing array**.

```text
ArrayList
    ↓
Backing Array

[A][B][C][ ][ ]
```

When the backing array becomes full, a larger array is created and elements are copied.

---

## ArrayList Properties

```text
✓ Maintains insertion order
✓ Allows duplicates
✓ Index-based
✓ Dynamic size
✓ Fast random access
```

---

## Size vs Capacity

### Size

Number of elements currently stored.

```text
[A][B][C]
```

Size = `3`

### Capacity

Number of elements the current backing array can hold before resizing.

```text
[A][B][C][ ][ ]
```

Size = `3`

Capacity = `5` in this simplified example.

---

## ArrayList Operations

### `get(index)`

```java
list.get(2);
```

Because it is array-backed, index access is direct.

```text
get(index) → O(1)
```

---

### `set(index, value)`

Replaces an existing element.

```text
[A][B][C]

set(1, "X")

[A][X][C]
```

```text
O(1)
```

---

### Add at end

```java
list.add("X");
```

Usually:

```text
O(1) amortized
```

Sometimes resizing requires copying elements, making that particular operation O(n).

---

### Insert at index

```text
[A][B][C][D]

add(1, "X")

[A][X][B][C][D]
```

Elements after the position must shift.

```text
O(n)
```

---

### Remove from index

```text
[A][B][C][D]

remove(1)

[A][C][D]
```

Elements after the removed element shift left.

```text
O(n)
```

---

## ArrayList Complexity

```text
get(index)       → O(1)
set(index,value) → O(1)
add(end)         → O(1) amortized
add(index)       → O(n)
remove(index)    → O(n)
```

### When to use ArrayList?

Use it as the **default List choice** when:

* You need frequent reading/index access
* You mostly add at the end
* You don't frequently insert/remove from the middle

---

# 4. LinkedList

### What is LinkedList?

`LinkedList` is a **doubly linked list**.

Each node conceptually contains:

```text
[previous | data | next]
```

Example:

```text
[A] ⇄ [B] ⇄ [C] ⇄ [D]
```

---

## Why is `get(index)` O(n)?

To reach an index, LinkedList must traverse nodes.

```text
get(2)

Head
 ↓
[A] → [B] → [C]
              ↑
            target
```

So:

```text
get(index) → O(n)
```

It can traverse from either end, but it still involves traversal.

---

## First/Last Operations

```java
list.addFirst("A");
list.addLast("B");

list.removeFirst();
list.removeLast();
```

These operations are efficient because the list maintains links to the ends.

---

## Insertion

```text
[A] ⇄ [B] ⇄ [C]

add(1, X)

[A] ⇄ [X] ⇄ [B] ⇄ [C]
```

Unlike ArrayList, LinkedList doesn't physically shift all elements.

However, finding the indexed position can take O(n).

---

## ArrayList vs LinkedList

|                 | ArrayList      | LinkedList          |
| --------------- | -------------- | ------------------- |
| Structure       | Dynamic array  | Doubly linked nodes |
| `get(index)`    | O(1)           | O(n)                |
| `set(index)`    | O(1)           | O(n)                |
| Add at end      | O(1) amortized | O(1)                |
| Index insertion | O(n)           | O(n) overall        |
| Default choice  | Usually yes    | Usually no          |

### Key interview point

> Don't say "LinkedList insertion is always O(1)." Finding the position can itself be O(n). Insertion/link adjustment is O(1) once the node/position is known.

---

# 5. Vector

### What is Vector?

`Vector` is an older **dynamic-array-based List implementation**.

It:

* Maintains insertion order
* Allows duplicates
* Supports index access
* Has synchronized methods

---

## Vector vs ArrayList

```text
ArrayList → not synchronized
Vector    → synchronized
```

Synchronization means operations use locking so that concurrent access to the same Vector is controlled at the method level.

### Why Vector is called legacy?

**Legacy** means an older API that is still supported but modern Java usually has better alternatives.

Legacy ≠ deprecated.

---

# 6. Stack

### What is Stack?

Stack follows:

> **LIFO — Last In, First Out**

Think of plates:

```text
Push A
Push B
Push C

Top
 ↓
 C
 B
 A
```

---

## Stack Operations

```java
stack.push("A");  // add
stack.pop();      // remove top
stack.peek();     // see top
```

Example:

```text
push A
push B
push C

pop → C
pop → B
```

---

## Modern Stack Approach

`Stack` is a legacy class.

Modern Java generally prefers:

```java
Deque<Integer> stack = new ArrayDeque<>();
```

We'll study `Deque` properly later.

---

# 7. Set

### What is Set?

`Set` is a collection that **does not allow duplicate elements**.

```text
Set
 ├── HashSet
 ├── LinkedHashSet
 └── TreeSet
```

---

# 8. HashSet

### What is HashSet?

`HashSet` stores **unique elements** and does not guarantee insertion order.

```java
Set<Integer> numbers = new HashSet<>();

numbers.add(10);
numbers.add(20);
numbers.add(10);
```

Only:

```text
10
20
```

are stored.

---

## Important HashSet Operations

```java
add()
contains()
remove()
size()
isEmpty()
```

### "Fast lookup" means

Checking whether an element exists:

```java
numbers.contains(20);
```

HashSet provides **average O(1)** lookup.

---

## HashSet vs ArrayList

Use:

```text
HashSet → uniqueness + fast existence checking
ArrayList → duplicates + order + index access
```

Neither is universally "better".

---

# 9. HashSet Internals

### How does HashSet store elements?

Conceptually:

```text
HashSet
   ↓
HashMap
   ↓
key → PRESENT
```

The HashSet element becomes the **HashMap key**.

The value is a common dummy object, often represented conceptually as:

```text
PRESENT
```

So:

```java
set.add("A");
```

is conceptually similar to:

```text
HashMap:

"A" → PRESENT
```

---

# 10. HashSet + `hashCode()` + `equals()`

### Why are they important?

HashSet uses hashing to find where an object belongs and `equals()` to determine whether another object is actually equal.

Simplified flow:

```text
Object
   ↓
hashCode()
   ↓
bucket
   ↓
equals()
   ↓
duplicate or new?
```

---

## Important Cases

```text
Different hashCode
→ different hash/bucket area
→ can coexist

Same hashCode
+
equals() == true
→ duplicate

Same hashCode
+
equals() == false
→ collision
→ both can exist
```

### Golden rule

```text
equals() == true
        ↓
hashCode() MUST be same
```

But:

```text
same hashCode
        ↓
does NOT mean equals() == true
```

---

# 11. Custom Objects in HashSet

Suppose:

```java
Employee e1 = new Employee(1, "Rakesh");
Employee e2 = new Employee(1, "Rakesh");
```

Even though their fields are the same, Java doesn't automatically consider them equal based on those fields.

Without appropriate `equals()`/`hashCode()` overrides:

```java
Set<Employee> employees = new HashSet<>();
employees.add(e1);
employees.add(e2);
```

can contain both objects.

---

## Correct approach

Override both:

```text
equals()
hashCode()
```

For example:

```java
@Override
public boolean equals(Object obj) {
    Employee other = (Employee) obj;

    return this.id == other.id &&
           this.name.equals(other.name);
}

@Override
public int hashCode() {
    return Objects.hash(id, name);
}
```

### Why both?

Because HashSet uses:

```text
hashCode() → find bucket
equals()   → confirm equality
```

Overriding only `equals()` can break the expected hashing contract.

---

# 12. LinkedHashSet

### What is LinkedHashSet?

`LinkedHashSet` is a Set that:

* Does not allow duplicates
* Maintains insertion order

```java
Set<String> names = new LinkedHashSet<>();

names.add("Rakesh");
names.add("Amit");
names.add("Rakesh");
```

Result:

```text
[Rakesh, Amit]
```

### Remember

```text
HashSet
→ unique + no guaranteed order

LinkedHashSet
→ unique + insertion order
```

---

# 13. TreeSet

### What is TreeSet?

`TreeSet` stores **unique elements in sorted order**.

```java
Set<Integer> numbers = new TreeSet<>();

numbers.add(30);
numbers.add(10);
numbers.add(20);
numbers.add(10);
```

Result:

```text
[10, 20, 30]
```

By default it uses **natural ordering**.

```text
Integer → numeric order
String  → natural lexicographical order
```

Basic operations are generally:

```text
O(log n)
```

Custom sorting will later be covered with `Comparable` and `Comparator`.

---

# 14. Set Quick Comparison

| Set             | Duplicate | Order               | Typical complexity |
| --------------- | --------- | ------------------- | ------------------ |
| `HashSet`       | ❌         | No guaranteed order | O(1) average       |
| `LinkedHashSet` | ❌         | Insertion order     | O(1) average       |
| `TreeSet`       | ❌         | Sorted              | O(log n)           |

### Memory trick

```text
HashSet       → Unique
LinkedHashSet → Unique + insertion order
TreeSet       → Unique + sorted
```

---

# 15. Map

### What is Map?

A `Map` stores data as:

```text
key → value
```

Example:

```text
101 → Rakesh
102 → Amit
103 → Rahul
```

Unlike List:

```text
List → index → value
Map  → key   → value
```

---

## Map Rules

```text
Keys   → unique
Values → duplicates allowed
```

Example:

```java
map.put(1, "Java");
map.put(2, "Java");
```

Allowed because the values can repeat.

But:

```java
map.put(1, "Java");
map.put(1, "Spring");
```

The second value replaces the first:

```text
1 → Spring
```

---

# 16. HashMap

### What is HashMap?

`HashMap` is the most commonly used general-purpose `Map` implementation.

It stores:

```text
key → value
```

and does **not guarantee insertion order**.

---

## Important Methods

```java
put()
get()
remove()
containsKey()
containsValue()
size()
isEmpty()
```

**Example:**

```java
Map<Integer, String> map = new HashMap<>();

map.put(1, "A");
map.put(2, "B");

map.get(1);         // A
map.containsKey(2); // true
map.remove(2);
```

---

# 17. HashMap Null Handling

`HashMap` allows:

```text
1 null key
multiple null values
```

**Example:**

```java
map.put(null, "Rakesh");
map.put(1, null);
map.put(2, null);
```

Valid.

If:

```java
map.put(null, "A");
map.put(null, "B");
```

there is still only **one null key**:

```text
null → B
```

because keys are unique.

---

## `get()` vs `containsKey()`

This is an important interview trap.

```java
map.get(10);
```

returning `null` could mean:

```text
key doesn't exist
```

OR:

```text
key exists and its value is null
```

So use:

```java
map.containsKey(10);
```

when you need to distinguish the two cases.

---

# 18. HashMap Complexity

Average case:

```text
put()        → O(1)
get()        → O(1)
remove()     → O(1)
containsKey  → O(1)
```

We say **average O(1)** because collisions and resizing can affect individual operations.

---

# 19. HashMap Internal Working 🔥

This is where the deeper understanding starts.

The basic mental model is:

```text
Key
 ↓
hashCode()
 ↓
hash
 ↓
bucket
 ↓
equals()
 ↓
entry
```

---

# 20. `hashCode()`

### What is `hashCode()`?

`hashCode()` returns an integer hash value for an object.

```java
String key = "Rakesh";

int hash = key.hashCode();
```

Important:

```text
hashCode ≠ bucket number
```

Instead:

```text
hashCode
   ↓
used to calculate bucket/index
```

---

# 21. Hashing

### What is hashing?

Hashing is the process of using a key's hash information to determine **where that key should be located** in a hash-based data structure.

Mental model:

```text
Key
 ↓
hashCode()
 ↓
hash
 ↓
bucket/index
```

---

# 22. Bucket

### What is a bucket?

A bucket is a position/slot in the HashMap's internal table where entries can be stored.

Conceptually:

```text
Bucket
0
1
2
3
4
5
6
7
```

Important:

> A bucket can contain more than one entry because collisions can occur.

---

# 23. Key → Hash → Bucket

Suppose, just for learning, we have 8 buckets.

```text
"A"
 ↓
hashCode()
 ↓
65
 ↓
65 % 8
 ↓
1
```

So:

```text
"A" → Bucket 1
```

⚠️ The `% 8` calculation was a **simplified illustration**, not the exact modern HashMap implementation formula.

The important concept is:

```text
hashCode()
     ↓
bucket/index calculation
     ↓
bucket
```

---

# 24. `HashMap.put()` Internal Flow

When we do:

```java
map.put("A", "Apple");
```

Conceptually:

```text
"A"
 ↓
hashCode()
 ↓
hash
 ↓
find bucket
 ↓
is bucket empty?
```

### Empty bucket

```text
Bucket 3
    ↓
"A" → "Apple"
```

Store the entry.

### Non-empty bucket

HashMap checks the existing keys.

```text
hash
 ↓
bucket
 ↓
equals()
```

Then:

```text
Same key
→ update value

Different key
→ collision
→ keep both
```

---

# 25. `HashMap.get()` Internal Flow

When:

```java
map.get("C");
```

Conceptually:

```text
"C"
 ↓
hashCode()
 ↓
hash
 ↓
find bucket
 ↓
check keys in that bucket
 ↓
equals()
 ↓
return value
```

HashMap does **not normally scan the entire map**.

It first finds the relevant bucket.

---

# 26. Collision

### What is collision?

A collision happens when **different keys end up in the same bucket**.

Example:

```text
"A" → Bucket 3
"C" → Bucket 3
```

Bucket:

```text
Bucket 3

"A" → "Apple"
"C" → "Cat"
```

But:

```java
"A".equals("C") // false
```

Therefore both can exist.

### Remember

> Collision does NOT mean duplicate key.

---

# 27. `equals()` in HashMap

### Why does HashMap need `equals()`?

`hashCode()` helps HashMap find the **bucket**.

But several keys may be inside that bucket.

So `equals()` identifies the **exact key**.

```text
hashCode()
   ↓
find bucket

equals()
   ↓
find exact key
```

### Example

```text
Bucket 3

"A" → Apple
"C" → Cat
```

Searching:

```java
map.get("C");
```

Conceptually:

```text
"C".equals("A") → false
"C".equals("C") → true
```

Return:

```text
Cat
```

---

# 28. Java 8+ HashMap Treeification

### What is treeification?

When a bucket gets heavily populated with colliding entries, Java 8+ `HashMap` can convert that bucket's linked-list structure into a **Red-Black Tree** under certain conditions.

Initially:

```text
Bucket
   ↓
A → B → C → D
```

Heavily populated:

```text
Bucket
   ↓
Red-Black Tree
```

---

## Important Thresholds

### `TREEIFY_THRESHOLD = 8`

At around **8 nodes in a bucket**, HashMap considers treeification.

### `MIN_TREEIFY_CAPACITY = 64`

The table must have sufficient capacity.

Simplified interview rule:

```text
bucket reaches 8
        +
capacity >= 64
        ↓
can treeify
```

---

## Why capacity matters?

Suppose:

```text
capacity = 32
bucket = 8 nodes
```

Instead of immediately treeifying:

```text
32 < 64
 ↓
resize
```

For example:

```text
32 → 64
```

The larger table gives the keys more possible bucket positions and may reduce collisions.

---

# 29. What is a Threshold?

### Threshold

A **threshold is a limit/value at which some action is considered or triggered**.

Examples:

```text
TREEIFY_THRESHOLD = 8
```

means 8 is the relevant limit for considering treeification.

```text
UNTREEIFY_THRESHOLD = 6
```

is the relevant threshold used for converting a tree back to a linked-list structure during certain operations.

Don't think:

> "Exactly 6 nodes always immediately become a list."

It's an implementation threshold used during particular operations.

---

# 30. Tree → Linked List

Suppose a bucket has already become a tree:

```text
       D
      / \
     B   F
    / \ / \
   A  C E  G
```

After removals/resizing, the tree may become small.

When the relevant conditions for untreeification are met:

```text
Tree
 ↓
small enough
 ↓
untreeification
 ↓
Linked-list structure
```

Conceptually:

```text
B → D → F → G → H → I
```

Remember:

```text
TREEIFY_THRESHOLD    → 8
MIN_TREEIFY_CAPACITY → 64
UNTREEIFY_THRESHOLD  → 6
```

---

# 31. HashMap Resize

### Why does HashMap resize?

When the table becomes sufficiently full according to its capacity/load-factor threshold, HashMap increases its table capacity and redistributes entries.

Simplified:

```text
32 buckets
     ↓
resize
     ↓
64 buckets
```

### Important misconception

More buckets does **NOT** mean:

```text
1 entry = 1 bucket
```

Instead, each key still goes to the bucket determined by its hash.

So after resizing you can still have:

```text
Bucket 5  → A → C → F
Bucket 37 → B → D
```

while many other buckets remain empty.

---

## Why doesn't HashMap randomly use empty buckets?

Because the bucket is determined by the key's hash.

If `"C"` belongs to Bucket 5 according to the bucket calculation, HashMap cannot simply say:

> "Bucket 6 is empty, I'll put C there."

Otherwise, when:

```java
map.get("C");
```

is called, HashMap would calculate Bucket 5 and fail to find it.

So:

> **Resizing gives keys more possible bucket positions; it doesn't allow arbitrary placement.**

---

# 🧠 Final Mental Model — HashMap

If you remember only one diagram, remember this:

```text
                 HashMap
                    |
                  Key
                    ↓
               hashCode()
                    ↓
                  hash
                    ↓
             bucket/index
                    ↓
             ┌──────┴──────┐
             ↓             ↓
          empty         occupied
             ↓             ↓
           store        compare key
                           ↓
                       equals()
                           ↓
                  ┌────────┴────────┐
                  ↓                 ↓
                true              false
                  ↓                 ↓
             same key            collision
             update value        keep both
```

And for lookup:

```text
get(key)
   ↓
hashCode()
   ↓
find bucket
   ↓
equals()
   ↓
find exact key
   ↓
return value
```

---

# 🎯 Quick Interview Recall

### ArrayList

> Dynamic-array-based List. Maintains insertion order, allows duplicates, and provides O(1) index access.

### LinkedList

> Doubly linked List where elements are stored in nodes. Index access is O(n) because traversal is required.

### HashSet

> Set that stores unique elements with no guaranteed order and provides average O(1) add/search/remove.

### LinkedHashSet

> HashSet-like Set that additionally maintains insertion order.

### TreeSet

> Set that stores unique elements in sorted order, generally with O(log n) basic operations.

### HashMap

> Map that stores key-value pairs, allows unique keys and duplicate values, and provides average O(1) lookup.

### HashMap internal working

> HashMap uses the key's hash information to locate a bucket and `equals()` to identify the exact key within that bucket.

### Collision

> When different keys end up in the same bucket.

### Java 8+ treeification

> Heavily collided buckets can be converted from a linked-list structure to a Red-Black Tree when the relevant thresholds and capacity conditions are met.

---

# 📍 Current Progress

```text
Java Collections
│
├── Collection Framework              ✅
│
├── List                              ✅
│   ├── List basics                   ✅
│   ├── ArrayList                     ✅
│   ├── LinkedList                    ✅
│   ├── Vector                        ✅
│   └── Stack                         ✅
│
├── Set                               ✅
│   ├── Set basics                    ✅
│   ├── HashSet                       ✅
│   ├── HashSet internals             ✅
│   ├── equals/hashCode               ✅
│   ├── LinkedHashSet                 ✅
│   └── TreeSet                       ✅
│
└── Map
    ├── Map basics                    ✅
    ├── HashMap basics                ✅
    └── HashMap internals             🔥
        ├── hashCode()                ✅
        ├── Hashing                   ✅
        ├── Bucket                    ✅
        ├── hash → bucket             ✅
        ├── put()                     ✅
        ├── get()                     ✅
        ├── Collision                 ✅
        ├── equals()                  ✅
        ├── Treeification             ✅
        ├── Resize concept            ✅
        │
        ├── Mutable Keys              ⬅️ NEXT
        ├── Resize/Rehashing deeper
        └── Final HashMap explanation
```
