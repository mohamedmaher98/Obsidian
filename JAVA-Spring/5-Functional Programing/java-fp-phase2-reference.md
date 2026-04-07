# Java Functional Programming — المرحلة التانية (القوة الحقيقية)

---

## 1. Optional — الصندوق الذكي

### المشكلة

في Java العادية لما method بترجع `null` مفيش حاجة بتحذرك:

```java
Customer customer = customerDao.findById(999); // ممكن يرجع null
System.out.println(customer.getName());         // 💥 NullPointerException
```

فبتضطر تعمل null check في كل مكان:

```java
if (customer != null) {
    System.out.println(customer.getName());
} else {
    System.out.println("Not found");
}
```

### الحل: Optional

الـ `Optional<T>` هو **صندوق** بيجبرك تتعامل مع احتمال إن القيمة مش موجودة:

```java
Optional<Customer> result = customerDao.findById(id);
```

- لو الـ id موجود → الصندوق **فيه الـ Customer**
- لو مش موجود → الصندوق **فاضي**

> **Optional مش بيمنع الـ null — ده بيجبرك تفكر في احتمال إن القيمة مش موجودة.**

### طرق التعامل مع Optional

#### الطريقة 1: `isPresent()` + `get()` — اسأل الصندوق

```java
if (result.isPresent()) {
    Customer customer = result.get();
    System.out.println(customer.getName());
} else {
    System.out.println("Not found");
}
```

> ده شبه الـ null check القديمة — مش أحسن طريقة.

#### الطريقة 2: `ifPresent(Consumer)` — لو فيه حاجة، استهلكها

```java
result.ifPresent(customer -> System.out.println(customer.getName()));
```

> سطر واحد بدل 5! بتاخد **Consumer** — لو الصندوق فيه قيمة، نفّذ السلوك ده عليها.

#### الطريقة 3: `orElse(value)` — لو فاضي هاتلي بديل

```java
Customer customer = result.orElse(new Customer("Guest"));
```

> لو الصندوق فيه customer هاته، لو فاضي هاتلي واحد اسمه Guest.

#### الطريقة 4: `map(Function)` — حوّل القيمة اللي جوا الصندوق

```java
Optional<String> name = result.map(customer -> customer.getName());
```

> لو الصندوق فيه customer، حوّله لاسمه. لو فاضي، سيبه فاضي.

### القوة الحقيقية: Chaining

بدل null checks متداخلة:

```java
// الطريقة القديمة 😫
Customer customer = customerDao.findById(id);
String displayName;
if (customer != null) {
    String name = customer.getName();
    if (name != null) {
        displayName = name.toUpperCase();
    } else {
        displayName = "UNKNOWN";
    }
} else {
    displayName = "UNKNOWN";
}
```

بتكتب chain بسيط:

```java
// مع Optional 😍
String displayName = customerDao.findById(id)
    .map(customer -> customer.getName())
    .map(name -> name.toUpperCase())
    .orElse("UNKNOWN");
```

### ملخص Optional Methods

| Method | بتاخد | بتعمل ايه |
|--------|-------|-----------|
| `isPresent()` | — | بترجع true لو فيه قيمة |
| `get()` | — | بترجع القيمة (خطر لو فاضي!) |
| `ifPresent()` | Consumer | لو فيه قيمة، نفّذ السلوك ده |
| `map()` | Function | لو فيه قيمة، حوّلها |
| `orElse()` | قيمة بديلة | لو فاضي، استخدم البديل |

---

## 2. Streams API — الـ Pipeline

### الفكرة

فاكر الـ `filterList` و `transformList` اللي كتبناهم بنفسنا؟ الـ Stream هو **Java عملتهم جاهزين** — بس محتاج تحوّل الـ List لـ Stream الأول:

```java
// بدل ما تكتب method بنفسك
filterList(names, n -> n.length() > 3);

// Java عملتهالك جاهزة
names.stream().filter(n -> n.length() > 3);
```

### الـ 3 خطوات

أي Stream بيتكون من 3 خطوات:

```java
names.stream()                          // 1. افتح الـ Stream
    .filter(n -> n.length() > 3)        // 2. العمليات (filter, map, ...)
    .collect(Collectors.toList());       // 3. اجمع النتيجة
```

1. **`.stream()`** — حوّل الـ List لـ Stream
2. **العمليات الوسيطة** — `filter`, `map`, `sorted` (دي بتجهز بس مش بتنفذ)
3. **العملية النهائية** — `collect`, `forEach`, `reduce` (دي بتنفذ وبتجيب النتيجة)

> ملاحظة مهمة: العمليات الوسيطة **مش بتنفذ لوحدها** — لازم يبقى فيه عملية نهائية عشان كل حاجة تشتغل.

---

### العمليات الوسيطة (Intermediate Operations)

#### `filter(Predicate)` — فلتر بشرط

```java
// الأسماء اللي طولها أكبر من 3
names.stream()
    .filter(n -> n.length() > 3)
    .toList();
```

> بترجع **عدد أقل** من العناصر، بس **نفس الداتا**.

#### `map(Function)` — حوّل كل عنصر

```java
// حوّل كل اسم لحروف كبيرة
names.stream()
    .map(n -> n.toUpperCase())
    .toList();
// أو بـ method reference
names.stream()
    .map(String::toUpperCase)
    .toList();
```

> بترجع **نفس العدد**، بس **داتا متحولة**.

#### `sorted()` — رتّب

```java
// ترتيب أبجدي (تصاعدي)
names.stream()
    .sorted()
    .toList();

// ترتيب حسب الطول
names.stream()
    .sorted(Comparator.comparing(String::length))
    .toList();
```

---

### العمليات النهائية (Terminal Operations)

#### `forEach(Consumer)` — نفّذ على كل عنصر

```java
names.stream()
    .forEach(System.out::println);
```

> مفيش بعدها `collect` — عشان هي مش بترجع حاجة.

#### `collect(Collector)` — اجمع النتيجة

```java
List<String> result = names.stream()
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());

// أو الاختصار (Java 16+)
List<String> result = names.stream()
    .filter(n -> n.length() > 3)
    .toList();
```

#### `reduce(identity, BinaryOperator)` — لخّص في قيمة واحدة

```java
// جمع كل الأرقام
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);

// ضرب كل الأرقام
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);
```

الـ `reduce` بتشتغل خطوة خطوة:
- بتبدأ بالقيمة الابتدائية (identity)
- بتاخد أول عنصر وتطبق العملية
- بتاخد تاني عنصر وتطبق العملية على النتيجة السابقة
- وهكذا... لحد ما تخلص

---

### مثال شامل — Pipeline كامل

```java
List<String> names = Arrays.asList("Ahmed", "Sara", "", "Ali", null, "Samira");

// الأسماء الصالحة بحروف كبيرة مرتبة
List<String> result = names.stream()
    .filter(Objects::nonNull)           // شيل الـ null
    .filter(n -> !n.isEmpty())          // شيل الفاضي
    .map(String::toUpperCase)           // حوّل لحروف كبيرة
    .sorted()                           // رتّب
    .toList();                          // اجمع النتيجة

// النتيجة: ["AHMED", "ALI", "SAMIRA", "SARA"]
```

### مثال عملي — حساب بعد خصم

```java
List<Double> prices = Arrays.asList(150.0, 30.0, 200.0, 80.0, 15.0, 300.0, 50.0);

// مجموع الأسعار فوق 50 بعد خصم 10%
double total = prices.stream()
    .filter(price -> price > 50.0)                    // فلتر
    .map(price -> price - (price * 10 / 100))         // خصم 10%
    .reduce(0.0, (a, b) -> a + b);                    // اجمع

// النتيجة: 657.0
```

---

## 3. Collectors — طرق تجميع النتيجة

### الـ Collectors الأساسية

#### `toList()` — اجمع في List

```java
List<String> result = names.stream()
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());
```

#### `toSet()` — اجمع في Set (من غير تكرار)

```java
Set<String> result = names.stream()
    .collect(Collectors.toSet());
```

#### `joining()` — الزق في String واحد

```java
String result = names.stream()
    .collect(Collectors.joining(", "));
// "Ahmed, Ali, Sara, Samira, Amr"

// ممكن تحط أي فاصل
.collect(Collectors.joining(" - "))     // Ahmed - Ali - Sara
.collect(Collectors.joining(" و "))     // Ahmed و Ali و Sara
```

### Collectors للأرقام

```java
List<Integer> numbers = Arrays.asList(10, 20, 30, 40, 50);

// المجموع
int sum = numbers.stream()
    .collect(Collectors.summingInt(n -> n));

// المتوسط
double avg = numbers.stream()
    .collect(Collectors.averagingInt(n -> n));

// العدد
long count = numbers.stream()
    .collect(Collectors.counting());
```

### `groupingBy(Function)` — صنّف في مجموعات

```java
List<String> names = Arrays.asList("Ahmed", "Ali", "Sara", "Samira", "Amr");

// صنّف حسب أول حرف
Map<Character, List<String>> byLetter = names.stream()
    .collect(Collectors.groupingBy(name -> name.charAt(0)));
// A → ["Ahmed", "Ali", "Amr"]
// S → ["Sara", "Samira"]

// صنّف حسب الطول
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
// 3 → ["Ali", "Amr"]
// 4 → ["Sara"]
// 5 → ["Ahmed"]
// 6 → ["Samira"]
```

> استخدم `groupingBy` لما عندك **مجموعات كتير**.

### `partitioningBy(Predicate)` — قسّم لمجموعتين

```java
List<Double> prices = Arrays.asList(150.0, 30.0, 200.0, 80.0, 15.0);

Map<Boolean, List<Double>> result = prices.stream()
    .collect(Collectors.partitioningBy(n -> n >= 100));
// true  → [150.0, 200.0]        // غالي
// false → [30.0, 80.0, 15.0]    // رخيص
```

> استخدم `partitioningBy` لما عندك **مجموعتين بس (أيوه أو لأ)**.

### عرض نتيجة الـ Map

```java
// الطريقة البسيطة
System.out.println("غالي: " + result.get(true));
System.out.println("رخيص: " + result.get(false));

// الطريقة الـ FP بـ forEach
result.forEach((isExpensive, priceList) -> {
    String label = isExpensive ? "غالي" : "رخيص";
    System.out.println(label + ": " + priceList);
});
```

> الـ `forEach` على الـ Map بتاخد **2 parameters** (key و value).

---

## ملخص جدول الـ Collectors

| Collector | النتيجة | بيعمل ايه |
|-----------|---------|-----------|
| `toList()` | List | يجمع في list |
| `toSet()` | Set | يجمع من غير تكرار |
| `joining()` | String | يلزق في نص واحد |
| `groupingBy()` | Map | يصنّف في مجموعات كتير |
| `partitioningBy()` | Map | يقسّم لمجموعتين (true/false) |
| `counting()` | long | يعد |
| `summingInt()` | int | يجمع أرقام |
| `averagingInt()` | double | يحسب المتوسط |

---

## ملخص المرحلة التانية

1. **Optional** = صندوق بيجبرك تتعامل مع احتمال إن القيمة مش موجودة
2. **Stream** = الـ List بتاعتك بس بـ methods جاهزة (filter, map, sorted, forEach, reduce)
3. **Collectors** = طرق مختلفة لتجميع نتيجة الـ Stream

والأهم — كل ده مبني على **الـ 4 Functional Interfaces** اللي اتعلمناهم في المرحلة الأولى:

- `filter()` بتاخد **Predicate**
- `map()` بتاخد **Function**
- `forEach()` بتاخد **Consumer**
- `Supplier` بيظهر مع **Optional** و**Stream generation**

---

> **المرحلة الجاية: Composition, Custom Functional Interfaces, Currying, Lazy Evaluation**
