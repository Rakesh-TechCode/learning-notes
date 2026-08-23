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
