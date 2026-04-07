# Java Functional Programming — المرحلة التالتة (التكتيكات المتقدمة)

---

## 1. Composition — ربط السلوكيات مع بعض

### Predicate Composition — ربط الشروط

لو عندك أكتر من شرط وعايز تربطهم مع بعض:

```java
Predicate<String> isLong = n -> n.length() > 3;
Predicate<String> startsWithA = n -> n.startsWith("A");
```

#### `.and()` — الشرطين مع بعض

```java
Predicate<String> combined = isLong.and(startsWithA);
// لازم الاسم يكون طويل وبيبدأ بـ A

names.stream().filter(combined).toList();
// ["Ahmed"] ✅ — طويل وبيبدأ بـ A
// ["Ali"] ❌ — بيبدأ بـ A بس مش طويل
// ["Sara"] ❌ — طويلة بس مش بتبدأ بـ A
```

#### `.or()` — أي واحد من الشرطين

```java
Predicate<String> either = isLong.or(startsWithA);
// يكفي إن الاسم يكون طويل أو بيبدأ بـ A

// ["Ahmed"] ✅ — الاتنين
// ["Ali"] ✅ — بيبدأ بـ A
// ["Sara"] ✅ — طويلة
// ["Mo"] ❌ — لا ده ولا ده
```

#### `.negate()` — عكس الشرط

```java
Predicate<String> isShort = isLong.negate();
// عكس "طويل" = "قصير"

names.stream().filter(isShort).toList();
// ["Ali", "Mo"] — الأسماء القصيرة
```

### ملخص Predicate Composition

| Method | معناه | مثال |
|--------|-------|------|
| `.and()` | الشرطين **مع بعض** | `isLong.and(startsWithA)` |
| `.or()` | **أي واحد** من الشرطين | `isLong.or(startsWithA)` |
| `.negate()` | **عكس** الشرط | `isLong.negate()` |

---

### Function Composition — ربط التحويلات

لو عندك أكتر من تحويل وعايز تربطهم:

```java
Function<String, String> toUpper = s -> s.toUpperCase();
Function<String, String> addGreeting = s -> "Hello " + s;
```

#### `.andThen()` — نفّذ الأولى وبعدين التانية

```java
Function<String, String> greetLoud = toUpper.andThen(addGreeting);
greetLoud.apply("ahmed");
// الخطوة 1 (toUpper): "ahmed" → "AHMED"
// الخطوة 2 (addGreeting): "AHMED" → "Hello AHMED"
```

#### `.compose()` — نفّذ التانية وبعدين الأولى (عكس andThen)

```java
Function<String, String> result = toUpper.compose(addGreeting);
result.apply("ahmed");
// الخطوة 1 (addGreeting): "ahmed" → "Hello ahmed"
// الخطوة 2 (toUpper): "Hello ahmed" → "HELLO AHMED"
```

#### ربط أكتر من Function

```java
Function<Integer, Integer> doubleIt = n -> n * 2;
Function<Integer, Integer> addTen = n -> n + 10;
Function<Integer, Integer> square = n -> n * n;

// ابني الـ pipeline الأول، ونفّذه بعدين
int result = doubleIt.andThen(addTen).andThen(square).apply(3);
// doubleIt: 3 * 2 = 6
// addTen: 6 + 10 = 16
// square: 16 * 16 = 256
```

> **مبدأ مهم: ابني الـ pipeline الأول (بـ andThen)، ونفّذه بعدين (بـ apply).**

### ملخص Function Composition

| Method | معناه | الترتيب |
|--------|-------|---------|
| `.andThen()` | نفّذ الأولى **وبعدين** التانية | A → B |
| `.compose()` | نفّذ التانية **وبعدين** الأولى | B → A |

> **نصيحة: استخدم `andThen` بس — أسهل في القراءة.**

---

## 2. Custom Functional Interfaces — اعمل واحد بنفسك

### امتى تحتاجه؟

لما الأربعة الجاهزين (`Consumer`, `Predicate`, `Function`, `Supplier`) مش بيوصفوا اللي أنت عايزه — سواء في **عدد الـ inputs** أو في **المعنى**.

### ازاي تعمله؟

```java
@FunctionalInterface
interface TriFunction<A, B, C, R> {
    R apply(A a, B b, C c);
}
```

- **`@FunctionalInterface`** — مش إجبارية بس بتخلي Java تتأكد إن فيه **method واحدة بس**
- **الـ generics** — بتحدد أنواع الـ inputs والـ output

### مثال: TriFunction بـ 3 inputs

```java
@FunctionalInterface
interface TriFunction<A, B, C, R> {
    R apply(A a, B b, C c);
}

// استخدام
TriFunction<String, Integer, String, String> format = 
    (name, age, city) -> name + " - " + age + " - " + city;

format.apply("Ahmed", 30, "Cairo");
// "Ahmed - 30 - Cairo"
```

### مثال: Validator بـ return type ثابت

لو الـ return type دايماً نفسه، مش لازم يبقى generic:

```java
@FunctionalInterface
interface Validator<T> {
    String validate(T t);  // دايماً بيرجع String
}

// استخدام
Validator<String> emailValidator = 
    email -> email.contains("@") ? null : "Invalid email";

Validator<Integer> ageValidator = 
    age -> age >= 18 ? null : "Must be 18+";

String error = emailValidator.validate("ahmed.com");
// "Invalid email"

String error2 = ageValidator.validate(25);
// null (مفيش مشكلة)
```

### مثال: Transformer بـ 2 inputs

```java
@FunctionalInterface
interface Transformer<A, B, C> {
    A apply(B b, C c);
}

// حساب الإجمالي
Transformer<Double, Double, Integer> totalCalc = 
    (price, quantity) -> price * quantity;

totalCalc.apply(150.0, 3);  // 450.0

// تنسيق بيانات موظف
Transformer<String, Integer, String> empFormat = 
    (num, name) -> "Employee #" + num + " " + name;

empFormat.apply(1121, "Ali");  // "Employee #1121 Ali"
```

### القاعدة

> اعمل Custom Interface لما:
> - محتاج **أكتر من 2 inputs** (الجاهز بيوصل لـ 2 بس مع BiFunction)
> - عايز **اسم واضح** يوصف الغرض (Validator أوضح من Function)
> - عايز **تثبّت الـ return type** (Validator دايماً بيرجع String)

---

## 3. Currying — ثبّت input واعمل function مخصصة

### الفكرة

تخيل عندك **ماكينة عصير** محتاجة حاجتين:
- **نوع الفاكهة** — برتقال، مانجا
- **كمية السكر** — 1، 2، 3

كل يوم بتقولها "برتقال + 2 سكر" — بتكرر نفس الكلام.

ما لو **ثبّت الفاكهة** مرة واحدة وبعد كده تقولها **السكر بس**؟

### في Java

```java
// مصنع ماكينات — بياخد الفاكهة وبيرجعلك ماكينة فاكراها
Function<String, Function<String, String>> machineFactory = 
    fruit -> sugar -> fruit + " juice with " + sugar + " sugar";
```

ده معناه: **"اديني الفاكهة، وأنا هرجعلك function تانية مستنية السكر"**.

### خطوات الاستخدام

```java
// الخطوة 1: ثبّت الفاكهة
Function<String, String> orangeMachine = machineFactory.apply("Orange");
Function<String, String> mangoMachine = machineFactory.apply("Mango");

// الخطوة 2: ابعت السكر بس
orangeMachine.apply("2");  // "Orange juice with 2 sugar"
orangeMachine.apply("1");  // "Orange juice with 1 sugar"
mangoMachine.apply("3");   // "Mango juice with 3 sugar"
```

### مثال عملي: حساب الضريبة

```java
Function<Double, Function<Double, Double>> addTax = 
    taxRate -> price -> price + (price * taxRate);

// ثبّت ضريبة كل بلد مرة واحدة
Function<Double, Double> egyptTax = addTax.apply(0.14);  // مصر 14%
Function<Double, Double> uaeTax = addTax.apply(0.05);    // الإمارات 5%

// استخدمهم بالسعر بس
egyptTax.apply(100.0);  // 114.0
egyptTax.apply(200.0);  // 228.0

uaeTax.apply(100.0);    // 105.0
uaeTax.apply(200.0);    // 210.0
```

### القاعدة

> **Currying = عندك function محتاجة أكتر من input، بتديها واحد وتثبّته، وهي بترجعلك function جديدة محتاجة الباقي بس.**

### امتى تستخدمه؟

- لما عندك **parameter ثابت** بيتكرر كتير (ضريبة، عملة، إعدادات)
- لما عايز تعمل **functions متخصصة** من function عامة

---

## 4. Lazy Evaluation — متنفذش غير لما تحتاج

### الفكرة

الـ Stream **مش بينفذ العمليات الوسيطة لحد ما تحط عملية نهائية**:

```java
// ده لسه ما عملش حاجة!
names.stream()
    .filter(n -> n.length() > 3)
    .map(n -> n.toUpperCase());
// مفيش عملية نهائية → مفيش تنفيذ
```

### الفايدة: توفير الأداء

لو عندك **مليون** عنصر وعايز أول 3 بس:

```java
List<String> result = names.stream()
    .filter(n -> n.length() > 5)
    .limit(3)
    .toList();
```

الـ Stream **مش هيلف على المليون** — هيقف أول ما يلاقي **3 عناصر** بتحقق الشرط.

### `limit(n)` — خد أول n عنصر بس

```java
// أول 5 أرقام زوجية من 1 لـ 1000
List<Integer> result = numbers.stream()
    .filter(n -> n % 2 == 0)
    .limit(5)
    .toList();
// [2, 4, 6, 8, 10]
// الـ Stream وقف بعد ما لقى 10 — مش لف على الـ 1000
```

### `findFirst()` — خد أول عنصر بس

```java
Optional<String> first = names.stream()
    .filter(n -> n.startsWith("A"))
    .findFirst();
// بيرجع Optional — عشان ممكن مفيش عنصر بيحقق الشرط
```

### القاعدة

> **الـ Stream lazy — بيعمل أقل شغل ممكن عشان يجيبلك اللي أنت طلبته.**

---

## ملخص المرحلة التالتة

1. **Composition** — ربط Predicates بـ `and`/`or`/`negate` وربط Functions بـ `andThen`/`compose`
2. **Custom Functional Interfaces** — اعمل interface خاص لما الجاهزين مش كفاية
3. **Currying** — ثبّت input واعمل function متخصصة
4. **Lazy Evaluation** — الـ Stream مش بينفذ غير لما يحتاج، وبيقف أول ما يلاقي اللي أنت طلبته

---

> **المرحلة الجاية: FP في الواقع — Patterns, Refactoring, Error Handling**
