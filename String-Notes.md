# Java String — Notes

## 1. What is String?

1. `String` is a **class** in Java used to represent a sequence of characters/text.
2. It is a **non-primitive/reference type** because `String` is a class.
3. String literals are written inside double quotes: `"Java"`.
4. Every String object has characters arranged with indexes starting from `0`.
5. `String` is a **predefined class** provided by Java.

---

## 2. Why String Gets Special Treatment

1. Strings are used very frequently in Java applications.
2. Common uses: usernames, emails, URLs, JSON, logs, passwords, etc.
3. Java provides special support for String literals and the **String Constant Pool**.
4. This allows identical String literals to be reused instead of creating unnecessary objects.

---

## 3. String Literal

```java
String s = "Java";
```

1. `"Java"` is a **String literal**.
2. Java checks the **String Constant Pool** for `"Java"`.
3. If it already exists, the existing object is reused.
4. If not, the String is added to the pool.
5. `s` stores the reference to that pooled String.

---

## 4. String Constant Pool

1. The String Constant Pool stores/reuses String literals.
2. It is a special area **within the JVM heap** in modern Java.
3. Identical literals generally refer to the same pooled String object.
4. Example: `String a = "Java"; String b = "Java";` → `a == b` is `true`.
5. Pooling reduces unnecessary duplicate String objects.

---

## 5. `new String()`

```java
String s = new String("Java");
```

1. `new` explicitly creates a **new String object**.
2. The `"Java"` literal is associated with the String Pool.
3. The newly created String object is separate from the pooled object.
4. Therefore, `"Java" == new String("Java")` is `false`.
5. `new String()` is generally unnecessary when a normal String literal is sufficient.

---

## 6. String Immutability

1. **Immutable** means a String object cannot be changed after creation.
2. String operations return a **new String** instead of modifying the existing one.
3. Example: `s.toUpperCase()` does not change `s`.
4. To use the result, assign it: `s = s.toUpperCase();`.
5. Immutability helps with security, thread safety, pooling, and reliable hashing.

---

## 7. Why String is Immutable

1. String Pool sharing would be unsafe if String objects could be modified.
2. Immutability makes shared String objects safe for multiple references/threads.
3. Strings are frequently used for security-sensitive values such as paths and URLs.
4. String immutability also makes Strings reliable as `HashMap` keys.
5. Once a String's content is created, its state remains unchanged.

---

## 8. Why String is `final`

1. `String` is declared as a **final class**.
2. Therefore, it cannot be inherited/subclassed.
3. This prevents subclasses from altering String's intended behavior.
4. It helps preserve String's predictable and secure behavior.
5. **`final class String`** and **`final String s`** have different meanings.

---

## 9. String Methods and Immutability

Methods such as:

* `concat()`
* `toUpperCase()`
* `toLowerCase()`
* `replace()`
* `substring()`

return a new String when a changed result is required.

```java
String s = "Java";
s.toUpperCase();
```

`s` is still `"Java"` because the returned `"JAVA"` was not stored.

---

## 10. `==` vs `equals()`

### `==`

1. For objects, `==` compares **references**.
2. It checks whether both references point to the same object.

### `equals()`

1. `equals()` compares the **logical content** of Strings.
2. Example: `"Java".equals(new String("Java"))` → `true`.
3. For comparing String text, normally use `equals()`.

---

## 11. `intern()`

1. `intern()` returns the **canonical String from the String Pool**.
2. If the same content already exists in the pool, its reference is returned.
3. Example: `String s3 = s2.intern();`.
4. `intern()` does not change `s2` unless you reassign it.
5. After `s2 = s2.intern()`, `s2` refers to the pooled String.

---

## 12. Compile-Time String Concatenation

```java
String s1 = "Ja" + "va";
String s2 = "Java";
```

1. `"Ja"` and `"va"` are compile-time constants.
2. The compiler can evaluate `"Ja" + "va"` as `"Java"`.
3. Therefore both can refer to the same pooled String.
4. `s1 == s2` → `true`.

---

## 13. Runtime String Concatenation

```java
String a = "Ja";
String b = "va";
String s1 = "Java";
String s2 = a + b;
```

1. `a + b` involves runtime variables.
2. The concatenation is therefore performed at runtime.
3. The resulting String is generally a separate object.
4. `s1 == s2` → `false`.
5. `s1.equals(s2)` → `true`.

---

## 14. `final String` and Compile-Time Constants

```java
final String a = "Ja";
final String b = "va";
String s = a + b;
```

1. `final` means the reference cannot be reassigned.
2. Because `a` and `b` are compile-time constant Strings, their values are known at compile time.
3. The compiler can evaluate `a + b` as `"Java"`.
4. Therefore it can refer to the pooled `"Java"` String.
5. `final` does **not** mean the variable is automatically a compile-time constant in every situation.

---

## 15. `final` Class vs `final` Variable

```java
final class String
```

→ `String` cannot be inherited.

```java
final String s = "Java";
```

→ `s` cannot be reassigned.

**Remember:** `final` has different effects depending on what it is applied to.

---

## 16. Heap vs String Pool

1. String objects are objects stored in the **JVM heap**.
2. The String Constant Pool is a special area within the heap.
3. A String literal uses/reuses the pooled object.
4. `new String("Java")` creates a separate String object.
5. Local variables hold references to these objects.

---

## 17. Number of String Objects

```java
String s1 = "Java";
String s2 = "Java";
String s3 = new String("Java");
String s4 = new String("Java");
```

1. `s1` and `s2` refer to one pooled `"Java"` object.
2. `s3` and `s4` refer to two separate objects.
3. Therefore, under the usual model, **3 distinct String objects** are involved.
4. `s1 == s2` → `true`.
5. `s1 == s3` → `false`, `s3 == s4` → `false`, `s3.equals(s4)` → `true`.

---

## 18. String and Garbage Collection

1. An object becomes **eligible for GC when it is no longer reachable**.
2. `s2 = null` can make a `new String()` object eligible if no other reference exists.
3. A pooled String can also become eligible if it becomes unreachable.
4. `eligible for GC` does **not** mean it is immediately destroyed.
5. GC depends on object reachability, not simply whether the object is a String.

---

# ⭐ Most Important Rules to Remember

```text
String → class + reference type
String → immutable
String → final class
String Pool → special area within JVM Heap
"Java" → String literal → pooled/reused
new String("Java") → separate String object
== → reference comparison
equals() → String content comparison
intern() → returns pooled String reference
```

# Java String — Notes — Continuation

## 19. `compareTo()`

**What it is:** `compareTo()` compares two Strings based on their lexicographical order.

```java
"Apple".compareTo("Banana");  // negative
"Banana".compareTo("Apple");  // positive
"Java".compareTo("Java");     // 0
```

1. `0` → both Strings are equal in ordering.
2. Negative → first String comes before the second.
3. Positive → first String comes after the second.
4. Lexicographical comparison is similar to dictionary ordering.
5. `equals()` checks content equality, while `compareTo()` is mainly used for ordering.

---

## 20. String ↔ Other Data Types

**What it is:** Java provides methods to convert Strings to other data types and other values to Strings.

### String → `int`

```java
String s = "25";
int n = Integer.parseInt(s);
```

### `int` → String

```java
int n = 25;
String s = String.valueOf(n);
```

### String → `char[]`

```java
String s = "Java";
char[] chars = s.toCharArray();
```

### `char[]` → String

```java
char[] chars = {'J', 'a', 'v', 'a'};
String s = new String(chars);
```

> Parsing/conversion is different from **boxing/unboxing**.

---

## 21. String, `int`, `char` and `char[]`

**What it is:** These are different Java data types even when they contain similar-looking values.

```java
"25"   // String
25     // int
'2'    // char
```

```java
char[] chars = {'J', 'a', 'v', 'a'};
```

1. `"2"` is a String.
2. `'2'` is a single `char`.
3. `25` is an `int`.
4. Multiple characters can be stored in a `char[]`.
5. `toCharArray()` converts a String into `char[]`.

---

## 22. Basic String Utility Methods

**What it is:** String provides methods for common text-processing operations.

We covered operations related to:

* Character access
* Finding String length
* Searching/checking content
* String modification operations
* Splitting
* Joining
* String comparison

> Remember: because String is immutable, operations that produce changed text return a new String.

---

# StringBuilder

## 23. What is `StringBuilder`?

**What it is:** `StringBuilder` is a mutable character sequence used when String content needs to be modified repeatedly.

```java
StringBuilder sb = new StringBuilder("Java");

sb.append(" Developer");

System.out.println(sb);
```

Output:

```text
Java Developer
```

1. `String` → immutable.
2. `StringBuilder` → mutable.
3. Builder modifies its existing character sequence.
4. Useful for repeated String modifications.
5. Especially useful inside loops.

---

## 24. StringBuilder and String Pool

**What it is:** A `StringBuilder` object is separate from the String Pool.

```java
String s = "Java";
StringBuilder sb = new StringBuilder(s);
```

```text
"Java" → String Pool
   ↓
StringBuilder → separate object
```

1. `"Java"` is a String literal and participates in the String Pool.
2. `StringBuilder` itself is **not** stored in the String Pool.
3. Builder creates its own mutable character sequence from the String.
4. Builder does not share the same object reference as `"Java"`.
5. Same initial content does not mean same object.

---

## 25. `StringBuilder.append()`

**What it is:** `append()` adds content at the end of the Builder.

```java
StringBuilder sb = new StringBuilder("Java");

sb.append(" Backend");
```

Result:

```text
Java Backend
```

1. Content is added at the end.
2. It can append different data types.
3. The same Builder object is modified.
4. `append()` returns the Builder itself.

---

## 26. `StringBuilder.insert()`

**What it is:** `insert()` adds content at a specific index.

```java
StringBuilder sb = new StringBuilder("Java");

sb.insert(2, "XX");
```

Result:

```text
JaXXva
```

1. The new content is inserted before the character currently at that index.
2. Existing characters are shifted to the right.
3. Index starts from `0`.

---

## 27. `delete()` and `deleteCharAt()`

**What it is:** These methods remove characters from a Builder.

### `delete()`

```java
sb.delete(1, 4);
```

1. Start index is **inclusive**.
2. End index is **exclusive**.

### `deleteCharAt()`

```java
sb.deleteCharAt(2);
```

1. Removes exactly one character.
2. The specified index must exist.

---

## 28. `replace()`

**What it is:** `replace()` replaces characters within a specified range.

```java
sb.replace(1, 4, "XYZ");
```

1. Start index is inclusive.
2. End index is exclusive.
3. The selected range is replaced with the new String.

---

## 29. `reverse()`

**What it is:** `reverse()` reverses the Builder's character sequence.

```java
StringBuilder sb = new StringBuilder("Java");

sb.reverse();
```

Result:

```text
avaJ
```

1. The Builder itself is modified.
2. No new String is required for the operation.

---

## 30. `setCharAt()` and `charAt()`

**What it is:** These methods modify or read an individual character.

### `setCharAt()`

```java
sb.setCharAt(0, 'X');
```

→ Replaces the character at the specified index.

### `charAt()`

```java
char ch = sb.charAt(0);
```

→ Reads the character at the specified index.

---

## 31. `length()` vs `capacity()`

**What it is:** `length()` tells how many characters currently exist; `capacity()` tells how much internal storage the Builder currently has.

```java
StringBuilder sb = new StringBuilder();
```

```text
length   = 0
capacity = 16
```

For:

```java
StringBuilder sb = new StringBuilder("Java");
```

```text
length   = 4
capacity = 20
```

Because:

```text
16 + 4 = 20
```

> **Remember:**
> `length` → current characters
> `capacity` → internal storage

---

## 32. StringBuilder Capacity Expansion

**What it is:** When the current capacity is not enough, StringBuilder automatically expands its internal storage.

Typical growth formula:

```text
new capacity = old capacity × 2 + 2
```

Example:

```text
old capacity = 10

10 × 2 + 2
= 22
```

1. Capacity grows automatically.
2. The final capacity must also be large enough for the required content.
3. Therefore, the actual new capacity can be larger than the basic growth formula when necessary.

---

## 33. `ensureCapacity()`

**What it is:** `ensureCapacity()` ensures that the Builder has at least the requested capacity.

```java
sb.ensureCapacity(50);
```

1. If current capacity is already `50` or more → no expansion is needed.
2. If current capacity is smaller → Builder expands.
3. Useful when a large amount of text is expected.

---

## 34. `toString()`

**What it is:** `toString()` converts the mutable Builder content into an immutable String.

```java
StringBuilder sb = new StringBuilder("Java");

String s = sb.toString();
```

```text
StringBuilder → String
```

Useful when a normal `String` is required.

---

## 35. Why StringBuilder Is Useful

**What it is:** StringBuilder avoids repeatedly creating new immutable String objects during repeated modifications.

With String:

```java
String s = "";

s = s + "Java";
s = s + " Backend";
s = s + " Developer";
```

With Builder:

```java
StringBuilder sb = new StringBuilder();

sb.append("Java");
sb.append(" Backend");
sb.append(" Developer");
```

1. String is immutable, so modifications produce new Strings.
2. StringBuilder is mutable, so the same Builder can be modified.
3. This is especially useful for repeated operations and loops.
4. We should not say StringBuilder is **always** faster; simple fixed concatenation can be optimized by Java.

---

## 36. StringBuilder Thread Safety

**What it is:** `StringBuilder` is not thread-safe.

1. Its methods are generally not synchronized.
2. Multiple threads modifying the same Builder can cause concurrency problems.
3. It is normally preferred when the Builder is used by one thread.
4. Thread safety is different from mutability.

```text
StringBuilder
→ Mutable
→ Not synchronized
→ Not thread-safe
```

---

# StringBuffer

## 37. What is `StringBuffer`?

**What it is:** `StringBuffer` is a mutable character sequence similar to StringBuilder.

```java
StringBuffer sb = new StringBuffer("Java");

sb.append(" Backend");
```

1. It supports repeated String modifications.
2. It provides methods similar to StringBuilder.
3. Its important difference is built-in synchronization.

---

## 38. StringBuffer Synchronization

**What it is:** StringBuffer's relevant methods are synchronized to provide thread-safe access.

Conceptually:

```java
public synchronized StringBuffer append(String str) {
    ...
}
```

1. Methods use synchronization internally.
2. Multiple threads can safely access the same StringBuffer through those synchronized operations.
3. Synchronized operations execute one at a time for the same object lock.
4. Synchronization introduces additional overhead.

> Thread-safe does **not** mean other threads cannot use the object. It means access is synchronized.

---

## 39. StringBuilder vs StringBuffer

**What it is:** Both are mutable, but their synchronization behavior is different.

| Feature                  | StringBuilder    | StringBuffer                                  |
| ------------------------ | ---------------- | --------------------------------------------- |
| Mutable                  | Yes              | Yes                                           |
| Thread-safe              | No               | Yes                                           |
| Methods synchronized     | No               | Yes                                           |
| Synchronization overhead | No               | Yes                                           |
| Typical performance      | Generally faster | Generally slower                              |
| Common use               | Single-threaded  | Shared mutable text requiring synchronization |

### Quick Rule

```text
Single-threaded → StringBuilder

Built-in synchronization required → StringBuffer
```

---

## 40. Explicit Synchronization with StringBuilder

**What it is:** StringBuilder itself is not synchronized, but we can synchronize access externally.

```java
synchronized (sb) {
    sb.append("Java");
}
```

We can also synchronize our own method:

```java
public synchronized void addText(String text) {
    sb.append(text);
}
```

1. External synchronization can protect shared Builder access.
2. Synchronizing one method does not automatically protect every direct access.
3. All shared access must follow a consistent locking strategy.

---

## 41. String vs StringBuilder vs StringBuffer

**What it is:** These classes mainly differ in mutability and synchronization.

```text
String
→ Immutable
→ Safe to share because state cannot change

StringBuilder
→ Mutable
→ Not thread-safe
→ Generally preferred for single-threaded modifications

StringBuffer
→ Mutable
→ Synchronized
→ Useful when built-in synchronization is required
```

### String Pool

```text
String literal
→ String Pool

StringBuilder
→ Normal object, not String Pool

StringBuffer
→ Normal object, not String Pool
```

---

## 42. StringBuilder Reference Behavior

**What it is:** Two references can point to the same mutable StringBuilder object.

```java
StringBuilder sb1 = new StringBuilder("Java");
StringBuilder sb2 = sb1;

sb2.append(" Developer");
```

Now:

```java
System.out.println(sb1);
System.out.println(sb2);
```

Both print:

```text
Java Developer
```

Because:

```text
sb1 ──┐
      ├──> same StringBuilder object
sb2 ──┘
```

> Modifying through `sb2` changes the same object seen through `sb1`.

---

## 43. String vs StringBuilder Reference Behavior

**What it is:** This demonstrates the difference between an immutable object and a mutable object.

### String

```java
String s1 = "Java";
String s2 = s1;

s2 = s2 + " Developer";
```

Result:

```text
s1 → "Java"
s2 → "Java Developer"
```

A new String is created and `s2` is reassigned.

### StringBuilder

```java
StringBuilder sb1 = new StringBuilder("Java");
StringBuilder sb2 = sb1;

sb2.append(" Developer");
```

Result:

```text
sb1 → "Java Developer"
sb2 → "Java Developer"
```

Both references point to the **same mutable object**.

> **Core rule:**
> Two references to the same **mutable object** see its modifications.
> Modifying an **immutable String** creates a new String instead.

