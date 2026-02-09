# ArrayList – Complete Guide (Java)

## 1. What is ArrayList?
- هي  Implementation من `List`
- مبنية داخليًا على `Array`
- حجمها Dynamic (بيكبر ويصغر تلقائيًا)

Example:
```java
List<String> list = new ArrayList<>();
```


------

## 2. Main Characteristics
- Ordered (مرتبة)
- Allows duplicates
- Allows null
- Fast access by index
- Not thread-safe

---

## 3. When to Use ArrayList

### Use it when:
- الترتيب مهم
- القراءة أكتر من الإضافة والحذف
- محتاج وصول سريع بالـ index

### Avoid it when:
- في حذف أو إضافة كتير في النص
- الأداء في التعديل مهم جدًا

---

## 4. Common Operations

### Add
- list.add("A")
- list.add(1, "B")

### Get
- list.get(0)

### Update
- list.set(0, "C")

### Remove
- list.remove(0)
- list.remove("A")

---

## 5. Big O (Time Complexity)

### get(index) → O(1)
- وصول مباشر بالـ index
- بدون looping

### remove(index) → O(n)
- تحريك العناصر بعد الحذف
- أسوأ حالة: الحذف من البداية

### remove(object) → O(n)
- Search + remove + shift

---

## 6. Big O Summary


| Operation | Complexity |
|----------|------------|
| get | O(1) |
| add (end) | O(1) |
| add (middle) | O(n) |
| remove | O(n) |
| search | O(n) |

---

## 7. ArrayList vs Array

| Array | ArrayList |
|------|-----------|
| Fixed size | Dynamic |
| Faster | Easier |
| Primitives allowed | Objects only |

---

## 8. ArrayList vs LinkedList

| Point | ArrayList | LinkedList |
|------|-----------|------------|
| get | O(1) | O(n) |
| add/remove middle | O(n) | O(1) |
| Memory | Less | More |

---

## 9. Thread Safety
- ArrayList مش thread-safe

Solutions:
- Collections.synchronizedList
- CopyOnWriteArrayList

---

## 10. Best Practices

- Set initial capacity if size known
- Don’t remove in for-each loop

---

## 11. Key Sentence
ArrayList = fast read, slow remove, default choice
