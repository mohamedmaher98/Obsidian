# Stack – Quick Summary (Java)

## What is Stack?
- Data structure based on **LIFO**
- Last In First Out
- آخر عنصر يدخل هو أول عنصر يطلع

---

## Stack in Java
- `Stack` is a class
- Extends `Vector`
- Thread-safe
- Considered **Legacy**

Example:
```java
Stack<Integer> stack = new Stack<>();

```
---

## Main Operations
- push() → add element
- pop() → remove & return top
- peek() → return top without removing
- isEmpty() → check if empty

---

## Time Complexity (Big O)

| Operation | Complexity |
| --------- | ---------- |
| push      | O(1)       |
| pop       | O(1)       |
| peek      | O(1)       |

---

## Common Use Cases

- Call Stack
- Undo / Redo
- Expression evaluation
- DFS

---

## Important Note
- `Stack` class is **not recommended** for new code

### Preferred Alternative
```java
Deque<Integer> stack = new ArrayDeque<>();
```

---

## Key Sentence
Stack = LIFO structure, legacy in Java, use Deque instead
