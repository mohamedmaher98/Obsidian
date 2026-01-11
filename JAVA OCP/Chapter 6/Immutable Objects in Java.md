
## 🔹 يعني إيه Immutable Object؟
الImmutable Object هو كائن **لا يمكن تغيير حالته الداخلية (state)** بعد إنشائه.  
أي تغيير في القيم ⇒ **إنشاء Object جديد**.

> ❗ Immutable ≠ final  
> final يمنع تغيير الريفرنس  
> Immutable يمنع تغيير الحالة

---

## 🔹 إمتى أستخدم Immutable Objects؟
- Value Objects (Money, Address, DateRange)
- الObjects بتتحط في Cache
- Multi-threaded environments
- Read-heavy systems
- لما التغيير يعمل Bugs أو Side Effects

---

## 🔹 Value Object (مفهوم مهم)
Value Object:
- بيمثل **قيمة** مش هوية
- ملوش ID
-ال equals/hashCode على القيم
- غالبًا Immutable

مثال:
- Money
- Address snapshot
- Config

لو القيمة اتغيرت ⇒ **Object جديد**

---

## 🔹 قواعد إنشاء Immutable Object
1. القيم تتحدد في الـ constructor
2. كل الـ fields تكون `private final`
3. مفيش setters
4. الكلاس يكون `final` (مفضل)
5. لو فيه Mutable fields ⇒ Defensive Copy

---

## 🔹 مثال Immutable بسيط
```java
public final class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
}

```

المشكلة مع Mutable Fields
```java
class Order {
    private final List<String> items;

    public Order(List<String> items) {
        this.items = items; // ❌ خطر
    }
}

```

المشكلة:

- الOrder ماسك نفس الـ List اللي برّه
    
- أي تغيير برّه ⇒ يغيّر Order

---
## 🔹 الحل: Defensive Copy

### Constructor

`this.items = new ArrayList<>(items);`

### أو الأفضل (Java 9+)

`this.items = List.copyOf(items);`

- نسخة جديدة
    
- Unmodifiable
    
- Safe
    

---

## 🔹 Getter صح

`public List<String> getItems() {     return items; // آمن لو Unmodifiable }`

---

## 🔹 final vs Immutable

- final variable → يمنع إعادة الإسناد
    
- Immutable object → يمنع تغيير الحالة
    

`final List<String> list = new ArrayList<>(); list.add("Java"); // مسموح ❌ مش Immutable`

---

## 🔹 ليه String Immutable؟

- String Pooling
    
- Caching
    
- Thread Safety
    
- Security
    
- Performance
    

> Immutability makes String safe to share.

---

## 🔹 Immutability & Cache

- Cached objects بتتشارك
    
- Mutable object ⇒ cache corruption
    
- Immutable ⇒ safe + predictable
    

---

## 🔹 جملة Interview جاهزة

> Immutability ensures that an object’s state cannot be changed after creation, making code safer, easier to reason about, and concurrency-friendly.

---

## 🔹 خلاصة Senior

- Immutable = Design decision
    
- مش كل Object ينفع يبقى Immutable
    
- أي تغيير ⇒ Object جديد
    
- لو الكلاس شكله Immutable بس exposes mutable state ⇒ خطر

---
