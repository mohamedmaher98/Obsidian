# Vector – Java Collection

## 1. What is Vector?
- Implementation من `List`
- مبني داخليًا على `Array`
- Thread-safe (كل الميثودز synchronized)
- Legacy class (قديم)

Example:
```java
Vector<String> vector = new Vector<>();

```

---

## 2. Main Characteristics
- Ordered
- Allows duplicates
- Allows null
- Dynamic size
- Thread-safe by default

---

## 3. Why Vector Exists?
- اتعمل في Java 1.0
- كان الحل الوحيد للـ multi-threading قبل Concurrent Collections

---

## 4. Thread Safety
- كل methods synchronized
- Thread واحد بس يشتغل في نفس الوقت
- أمان عالي ❌ أداء أقل

---

## 5. Common Operations

### Add
vector.add("A");

### Get
vector.get(0);

### Remove
vector.remove(0);

---

## 6. Big O (Time Complexity)

| Operation | Complexity |
|----------|------------|
| get | O(1) |
| add (end) | O(1) |
| add (middle) | O(n) |
| remove | O(n) |
| search | O(n) |

> ملاحظة: أبطأ من ArrayList بسبب synchronization

---

## 7. Vector vs ArrayList

| Point | Vector | ArrayList |
|------|--------|-----------|
| Thread-safe | Yes | No |
| Performance | Slower | Faster |
| Modern usage | Rare | Common |
| Legacy | Yes | No |

---

## 8. Modern Alternatives
- CopyOnWriteArrayList
- Collections.synchronizedList(new ArrayList<>())

---

## 9. When to Use Vector?
- كود قديم (Legacy systems)
- Interview questions
- نادرًا في شغل جديد

---

## 10. Key Sentence
Vector = ArrayList + synchronized + legacy
