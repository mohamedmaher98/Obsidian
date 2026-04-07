# Java Functional Programming — المرحلة الرابعة (الـ FP في الواقع)

---

## 1. امتى تستخدم FP وامتى لأ؟

### استخدم FP لما:

- بتتعامل مع **collections** (filter, map, sort, group)
- بتعمل **تحويلات على data** من شكل لشكل
- بتعمل **حسابات** أو **تجميع** (مجموع، متوسط، عدد)
- العملية **مفيهاش side effects** (مش بتغير حاجة برا الـ function)

### استخدم OOP لما:

- بتعدّل **state** في objects
- بتتعامل مع **database أو IO** (حفظ، قراءة ملفات)
- بتبني **business objects** معقدة بـ fields كتير
- العملية **ليها side effects** (بتأثر على حاجة برا الـ function)

### أمثلة عملية

```java
// ✅ FP مناسب — فلترة وحسابات على data
double total = invoices.stream()
    .filter(inv -> !inv.isPaid())
    .mapToDouble(Invoice::getAmount)
    .sum();

// ✅ FP مناسب — تحويل من شكل لشكل
List<String> names = employees.stream()
    .map(Employee::getName)
    .toList();

// ❌ OOP أنسب — تعديل state وحفظ في database
Employee emp = employeeDao.findById(id);
emp.setName("Ahmed");
emp.setSalary(15000);
employeeDao.save(emp);
```

---

## 2. Refactoring من OOP لـ FP

### مثال: فلترة فواتير متأخرة

#### قبل (OOP):

```java
List<Invoice> getOverdueInvoices(List<Invoice> invoices) {
    List<Invoice> result = new ArrayList<>();
    for (Invoice inv : invoices) {
        if (inv.getDueDate().isBefore(LocalDate.now())) {
            if (!inv.isPaid()) {
                result.add(inv);
            }
        }
    }
    Collections.sort(result, new Comparator<Invoice>() {
        @Override
        public int compare(Invoice a, Invoice b) {
            return a.getDueDate().compareTo(b.getDueDate());
        }
    });
    return result;
}
```

#### بعد (FP):

```java
List<Invoice> getOverdueInvoices(List<Invoice> invoices) {
    return invoices.stream()
        .filter(inv -> inv.getDueDate().isBefore(LocalDate.now()))
        .filter(inv -> !inv.isPaid())
        .sorted(Comparator.comparing(Invoice::getDueDate))
        .toList();
}
```

> 15 سطر بقوا 4 سطور — أوضح وأسهل في القراءة.

### نصائح الـ Refactoring

1. **دور على الـ pattern ده**: loop + if + list جديدة → حوّله لـ `stream().filter().toList()`
2. **دور على Anonymous classes**: `new Comparator<>() { ... }` → حوّلها لـ lambda أو method reference
3. **دور على null checks متداخلة**: `if (x != null) { if (x.y != null) ... }` → حوّلها لـ `Optional.map().orElse()`
4. **متحولش كل حاجة** — لو الكود بيعدّل state أو بيعمل IO، سيبه OOP

---

## 3. Error Handling بأسلوب FP

### المشكلة

```java
String input = "abc";
int number = Integer.parseInt(input); // 💥 NumberFormatException
```

### الحل: Wrap the Danger

اعمل method آمنة بترجع **Optional** بدل ما ترمي exception:

```java
public static Optional<Integer> safeParseInt(String input) {
    try {
        return Optional.of(Integer.parseInt(input));
    } catch (NumberFormatException e) {
        return Optional.empty();
    }
}
```

### الاستخدام

```java
// حالة النجاح
String result = safeParseInt("123")
    .map(n -> "الرقم: " + n)
    .orElse("Invalid number");
// "الرقم: 123"

// حالة الفشل
String result = safeParseInt("abc")
    .map(n -> "الرقم: " + n)
    .orElse("Invalid number");
// "Invalid number"
```

### الـ Pattern: Wrap the Danger

1. **اعمل method آمنة** — الخطر (try-catch) مخبي جواها
2. **ارجع Optional** — بدل ما ترمي exception
3. **استخدمها** بـ `map` و `orElse` — كود نضيف من غير try-catch

```java
// methods آمنة
Optional<Customer> findCustomer(int id) { ... }
Optional<Integer> safeParseInt(String s) { ... }
Optional<Date> safeParseDate(String s) { ... }

// استخدام نضيف — pipeline من غير أي try-catch
String name = findCustomer(id)
    .map(Customer::getName)
    .map(String::toUpperCase)
    .orElse("UNKNOWN");
```

---

## 4. الـ Bi-Variants — نسخة الـ 2 inputs

الأربع Functional Interfaces الأساسية ليهم نسخة بتاخد **2 inputs بدل 1**:

### الجدول الكامل

| Interface | بياخد | بيرجع | الـ Bi version | بياخد | بيرجع |
|-----------|-------|-------|----------------|-------|-------|
| `Function<T,R>` | 1 | قيمة | `BiFunction<T,U,R>` | 2 | قيمة |
| `Consumer<T>` | 1 | void | `BiConsumer<T,U>` | 2 | void |
| `Predicate<T>` | 1 | boolean | `BiPredicate<T,U>` | 2 | boolean |
| `Supplier<T>` | — | قيمة | ❌ مفيش BiSupplier | — | — |

> مفيش BiSupplier — عشان الـ Supplier أصلاً مش بياخد حاجة!

### أمثلة

#### BiFunction — بياخد 2 وبيرجع قيمة

```java
BiFunction<Double, Integer, Double> totalPrice = 
    (price, quantity) -> price * quantity;

totalPrice.apply(150.0, 3);  // 450.0
```

#### BiConsumer — بياخد 2 ومش بيرجع

```java
BiConsumer<String, Integer> printInfo = 
    (name, age) -> System.out.println(name + " عنده " + age + " سنة");

printInfo.accept("Ahmed", 30);
// "Ahmed عنده 30 سنة"
```

#### BiPredicate — بياخد 2 وبيرجع true/false

```java
BiPredicate<String, Integer> isNameLongerThan = 
    (name, length) -> name.length() > length;

isNameLongerThan.test("Ahmed", 3);   // true
isNameLongerThan.test("Ali", 3);     // false
```

### وين بنشوفهم في الواقع؟

```java
// Map.forEach بتاخد BiConsumer
map.forEach((key, value) -> System.out.println(key + ": " + value));

// Map.merge بتاخد BiFunction
map.merge("key", 1, (oldVal, newVal) -> oldVal + newVal);

// Map.replaceAll بتاخد BiFunction
map.replaceAll((key, value) -> value.toUpperCase());
```

### القاعدة

- محتاج **1 input**؟ → `Function`, `Consumer`, `Predicate`
- محتاج **2 inputs**؟ → `BiFunction`, `BiConsumer`, `BiPredicate`
- محتاج **3 inputs أو أكتر**؟ → اعمل **Custom Functional Interface**

---

## ملخص المرحلة الرابعة

1. **FP مناسب** لـ collections, تحويلات, حسابات — **OOP أنسب** لـ state, database, IO
2. **Refactoring** — دور على loops + ifs + anonymous classes وحولهم لـ streams + lambdas
3. **Error Handling** — اعمل methods آمنة بترجع Optional بدل ما ترمي exceptions
4. **Bi-Variants** — نفس الـ interfaces الأساسية بس بـ 2 inputs

---

## ملخص الـ Roadmap الكامل

### المرحلة 1: الأساسات
- FP = بتبعت **سلوك** كـ parameter مش بس data
- Lambda = **method من غير اسم** (`input -> body`)
- Functional Interfaces = **نوع بيانات للسلوك** (Consumer, Predicate, Function, Supplier)
- Method References = **اختصار** للـ lambda (`Class::method`)

### المرحلة 2: القوة الحقيقية
- Optional = **صندوق** بيجبرك تتعامل مع احتمال إن القيمة مش موجودة
- Streams = **pipeline** جاهز لـ filter, map, sorted, forEach, reduce
- Collectors = **طرق تجميع** النتيجة (toList, groupingBy, joining, counting...)

### المرحلة 3: التكتيكات المتقدمة
- Composition = **ربط** Predicates بـ and/or/negate و Functions بـ andThen/compose
- Custom Interfaces = اعمل **interface خاص** لما الجاهزين مش كفاية
- Currying = **ثبّت input** واعمل function متخصصة
- Lazy Evaluation = الـ Stream **مش بينفذ غير لما يحتاج**

### المرحلة 4: الـ FP في الواقع
- امتى FP وامتى OOP
- Refactoring من loops + ifs لـ streams + lambdas
- Error Handling بـ Optional بدل exceptions
- Bi-Variants للـ 2 inputs
