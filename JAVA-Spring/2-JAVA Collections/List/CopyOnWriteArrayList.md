# CopyOnWriteArrayList – Quick Summary

## What is it?
- Thread-safe implementation of `List`
- Part of `java.util.concurrent`
- Based on Array

---

## How it works
- **Read**: without locks (very fast)
- **Write (add/remove)**:
  - Creates a new copy of the array
  - Applies the change
  - Replaces the old array

---

## Key Characteristics
- Thread-safe
- Fast reads
- Slow writes
- No ConcurrentModificationException
- Uses more memory (due to copying)

---

## Time Complexity (Big O)

| Operation | Complexity |
|----------|------------|
| get | O(1) |
| add | O(n) |
| remove | O(n) |
| iteration | O(n) |

---

## When to Use
- Many reads, few writes
- Multi-threaded environment
- Small to medium list size

Examples:
- Configurations
- Event listeners
- Observer lists

---

## When NOT to Use
- Frequent modifications
- Very large lists
- Memory-sensitive cases

---

## Key Sentence
CopyOnWriteArrayList = thread-safe list with fast reads and slow writes
