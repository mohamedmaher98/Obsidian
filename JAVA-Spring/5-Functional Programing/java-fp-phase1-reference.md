# Java Functional Programming — المرحلة الأولى (الأساسات)

---

## 1. الفكرة الأساسية — ليه FP؟

في Java العادية، لما بتنادي method بتبعتلها **بيانات (data)**:

```java
calculator.add(5, 3);    // بعتنا أرقام
printer.print("hello");  // بعتنا نص
```

بس أحياناً مش محتاج تبعت **بيانات** — محتاج تبعت **سلوك** (يعني "هتعمل ايه بالبيانات").

### مثال بسيط

تخيل عندك list فيها أرقام، ومرة عايز **تطبعها**، ومرة عايز **تضربها في 2**، ومرة عايز **تفلترها**.

الأرقام (الـ data) هي هي — اللي بيتغير هو **"عايز تعمل ايه بيها"** — ده اسمه **سلوك**.

> **Functional Programming = القدرة إنك تبعت "سلوك" كـ parameter لـ method، زي ما بتبعت data.**

---

## 2. المشكلة — Java محتاجة Type لكل حاجة

Java لغة كل حاجة فيها لازم يبقى ليها **نوع (type)**:

- عايز تبعت رقم؟ → `int`
- عايز تبعت نص؟ → `String`
- عايز تبعت **سلوك**؟ → ؟؟؟ 🤔

**الحل: Functional Interface = نوع بيانات للسلوك.**

زي ما `int` نوع للأرقام، الـ Functional Interface نوع للسلوك.

---

## 3. Lambda Expressions — طريقة كتابة السلوك

الـ Lambda هي **method من غير اسم**. بنكتبها في خطوات:

```java
// 1. method عادية
void printNumber(int n) {
    System.out.println(n);
}

// 2. شيلنا الاسم والـ return type وحطينا سهم
(int n) -> { System.out.println(n); }

// 3. شيلنا الـ type (Java بتستنتجه لوحدها)
(n) -> { System.out.println(n); }

// 4. لو input واحد بنشيل الأقواس، ولو سطر واحد بنشيل الـ {}
n -> System.out.println(n)
```

### القاعدة

الـ Lambda بتتكون من 3 أجزاء:

```
input → سهم → body
  n      ->    System.out.println(n)
```

- **input**: الحاجة اللي هتيجيلك
- **`->`**: "هعمل بيها كده"
- **body**: الشغل اللي هيتعمل

### أمثلة

```java
// بياخد رقم وبيضربه في 2
n -> n * 2

// بياخد رقم وبيشوف لو أكبر من صفر
n -> n > 0

// بياخد اسم وبيحوله لحروف كبيرة
name -> name.toUpperCase()

// مش بياخد حاجة وبيرجع رقم عشوائي
() -> new Random().nextInt()

// بياخد 2 inputs
(a, b) -> a + b
```

### ملاحظات مهمة

- لو **input واحد** → الأقواس اختيارية: `n -> ...`
- لو **أكتر من input** → الأقواس إجبارية: `(a, b) -> ...`
- لو **مفيش input** → لازم أقواس فاضية: `() -> ...`
- لو الـ **body أكتر من سطر** → لازم `{}` و `return`:

```java
name -> {
    String upper = name.toUpperCase();
    return "Hello " + upper;
}
```

---

## 4. الـ Functional Interfaces الأربعة الأساسية

أي سلوك في الدنيا لازم يكون واحد من 4 أشكال:

### الجدول

| Interface | Method | بياخد | بيرجع | معناه |
|-----------|--------|-------|-------|-------|
| `Consumer<T>` | `accept(T)` | T | void | **مستهلك** — بياخد حاجة وبيستهلكها (بيعمل شغل بس) |
| `Predicate<T>` | `test(T)` | T | boolean | **شرط** — بياخد حاجة وبيقول أيوه أو لأ |
| `Function<T,R>` | `apply(T)` | T | R | **دالة** — بياخد حاجة وبيرجع حاجة تانية |
| `Supplier<T>` | `get()` | — | T | **مورّد** — مش بياخد حاجة بس بيرجع حاجة |

### ازاي تعرف أنهي واحد؟

اسأل نفسك سؤالين:
1. **بياخد input ولا لأ؟** → لو لأ = `Supplier`
2. **بيرجع ايه؟**
   - مش بيرجع حاجة = `Consumer`
   - بيرجع true/false = `Predicate`
   - بيرجع حاجة تانية = `Function`

### أمثلة مع الـ Lambda المناسبة

```java
// Consumer — بياخد وبيعمل شغل بس (طباعة، حفظ، إرسال...)
Consumer<String> printer = name -> System.out.println(name);
printer.accept("Ahmed");  // بيطبع Ahmed

// Predicate — بياخد وبيرجع true أو false
Predicate<String> isLong = name -> name.length() > 3;
isLong.test("Ahmed");  // بيرجع true

// Function — بياخد حاجة وبيرجع حاجة تانية
Function<String, String> shout = name -> name.toUpperCase();
shout.apply("ahmed");  // بيرجع "AHMED"

// Supplier — مش بياخد حاجة وبيرجع حاجة
Supplier<String> greeter = () -> "Hello World";
greeter.get();  // بيرجع "Hello World"
```

---

## 5. الاستخدام العملي — Methods بتاخد سلوك كـ parameter

### مثال 1: Consumer — نفذ سلوك على كل عنصر

```java
void processList(List<String> names, Consumer<String> action) {
    for (String name : names) {
        action.accept(name);
    }
}

// استدعاء
List<String> names = Arrays.asList("Ahmed", "Sara", "Mohamed");

processList(names, name -> System.out.println(name));       // اطبع
processList(names, name -> System.out.println(name.toUpperCase())); // اطبع بحروف كبيرة
processList(names, name -> database.save(name));             // احفظ
```

> **نفس الـ method + سلوك مختلف = نتيجة مختلفة.**

### مثال 2: Predicate — فلتر بشرط

```java
List<String> filterList(List<String> names, Predicate<String> condition) {
    List<String> result = new ArrayList<>();
    for (String name : names) {
        if (condition.test(name))
            result.add(name);
    }
    return result;
}

// استدعاء
filterList(names, n -> n.startsWith("S"));  // الأسماء اللي بتبدأ بـ S
filterList(names, n -> n.length() > 3);     // الأسماء اللي طولها أكبر من 3
filterList(names, n -> n.contains("m"));    // الأسماء اللي فيها حرف m
```

> **الـ filter بترجع list جديدة فيها عدد أقل بس نفس الداتا.**

### مثال 3: Function — حوّل كل عنصر

```java
List<String> transformList(List<String> names, Function<String, String> operation) {
    List<String> result = new ArrayList<>();
    for (String name : names) {
        result.add(operation.apply(name));
    }
    return result;
}

// استدعاء
transformList(names, n -> n.toUpperCase());    // حوّل لحروف كبيرة
transformList(names, n -> "Mr. " + n);         // ضيف لقب
transformList(names, n -> n.substring(0, 1));  // خد أول حرف بس
```

> **الـ transform بترجع list جديدة فيها نفس العدد بس داتا متحولة.**

### مثال 4: Supplier — ولّد object

```java
static List<String> createAndFill(Supplier<List<String>> listMaker) {
    List<String> list = listMaker.get();
    list.add("default item");
    return list;
}

// استدعاء
createAndFill(() -> new ArrayList<>());   // اعمل ArrayList
createAndFill(() -> new LinkedList<>());  // اعمل LinkedList
```

---

## 6. الفرق بين Filter و Transform

| | filterList | transformList |
|---|---|---|
| بتعمل ايه | بتشيل عناصر | بتحوّل عناصر |
| العدد | أقل أو يساوي الأصل | نفس الأصل |
| الداتا | نفسها زي ما هي | متغيرة |
| بتستخدم | `Predicate` | `Function` |

```java
List<String> names = Arrays.asList("Ahmed", "Sara", "Ali");

filterList(names, n -> n.length() > 3);
// النتيجة: ["Ahmed", "Sara"] — عدد أقل، نفس الأسماء

transformList(names, n -> n.toUpperCase());
// النتيجة: ["AHMED", "SARA", "ALI"] — نفس العدد، أسماء متحولة
```

### مبدأ مهم: Immutability

في الحالتين **الـ list الأصلية ما اتغيرتش**. ده مبدأ أساسي في FP:

> **مبنغيرش الداتا الأصلية — بنعمل نسخة جديدة.**

---

## 7. Method References — الاختصار الذكي

لو الـ lambda مش بتعمل حاجة غير إنها **بتنادي method موجودة** — استبدلها بـ **method reference** باستخدام `::`:

```java
// lambda                          →  method reference
n -> System.out.println(n)         →  System.out::println
n -> n.toUpperCase()               →  String::toUpperCase
n -> n.length()                    →  String::length
() -> new ArrayList<>()            →  ArrayList::new
```

### ازاي Java بتعرف الـ parameter؟

من الـ **type**. لما تكتب:

```java
transformList(names, String::toUpperCase);
```

Java بتبص على الـ parameter وتلاقيه `Function<String, String>` — يعني "هييجيلي String وهرجع String". فبتروح تدور على method اسمها `toUpperCase` في String وبتربطها تلقائياً.

### القاعدة

> **لو الـ lambda بتمرر الـ input مباشرة لـ method موجودة من غير أي شغل إضافي → استخدم method reference.**

### امتى متستخدمش method reference؟

لو بتعمل أي شغل إضافي:

```java
// ده فيه عملية إضافية (ضرب في 2) — مينفعش method reference
n -> n * 2

// ده فيه شرط — مينفعش method reference
n -> n.length() > 3

// ده فيه concatenation — مينفعش method reference
n -> "Mr. " + n
```

---

## ملخص المرحلة الأولى

1. **FP** = القدرة إنك تبعت سلوك كـ parameter
2. **Functional Interface** = نوع بيانات للسلوك (Consumer, Predicate, Function, Supplier)
3. **Lambda** = طريقة كتابة السلوك (`input -> body`)
4. **Method Reference** = اختصار للـ lambda لما بتنادي method موجودة (`Class::method`)

---

> **المرحلة الجاية: Optional, Streams API, Collectors**
