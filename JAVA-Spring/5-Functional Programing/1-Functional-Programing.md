# Functional Programming in Java 🚀

## ١. المشكلة اللي حلّها الـ Functional Programming

### الطريقة القديمة ❌

```java
public void doubleNumbers(List<Integer> numbers) {
    for (int n : numbers) { System.out.println(n * 2); }
}

public void tripleNumbers(List<Integer> numbers) {
    for (int n : numbers) { System.out.println(n * 3); }
}
// نفس الكود متكرر 10 مرات، لو غيرت شرط هتغيره في كل مكان!
```

### الطريقة الـ Functional ✅

```java
// method واحدة بتاخد السلوك من بره
processNumbers(numbers, n -> n * 2);   // ضاعف
processNumbers(numbers, n -> n * 3);   // تلت
processNumbers(numbers, n -> n * 10);  // ضرب في 10
```

> **الفكرة الكبيرة:** بدل ما تبعت بيانات بس، بتبعت **السلوك** مع البيانات.

---

## ٢. الـ Lambda Expression

Method صغيرة من غير اسم:

```java
// Method عادية
int doubleIt(int n) {
    return n * 2;
}

// نفس الكلام كـ Lambda
n -> n * 2
```

### تركيب الـ Lambda

```
(parameter) -> { body }

// أمثلة:
n -> n * 2                          // سطر واحد، بترجع قيمة
n -> System.out.println(n)          // سطر واحد، void
(n, m) -> n + m                     // parameter أكتر من واحد
n -> {                              // أكتر من سطر
    int result = n * 2;
    return result;
}
```

---

## ٣. الـ Functional Interfaces الأساسية

| Interface       | Method     | بتاخد | بترجع   | متى تستخدمها                             |
| --------------- | ---------- | ----- | ------- | ---------------------------------------- |
| `Consumer<T>`   | `accept()` | T     | void    | لما عايز تعمل حاجة بالقيمة (طباعة مثلاً) |
| `Predicate<T>`  | `test()`   | T     | boolean | لما عايز تفلتر (شرط)                     |
| `Function<T,R>` | `apply()`  | T     | R       | لما عايز تحوّل من نوع لنوع               |
| `Supplier<T>`   | `get()`    | —     | T       | لما عايز تولّد قيمة                      |

---

## ٤. أمثلة عملية من ERP

### Consumer — طباعة فواتير

```java
public void processInvoices(List<Invoice> invoices, Consumer<Invoice> operation) {
    for (Invoice i : invoices) {
        operation.accept(i);
    }
}

// استخدام
processInvoices(invoices, inv -> System.out.println(inv.getAmount()));
processInvoices(invoices, inv -> System.out.println("Tax: " + inv.getAmount() * 0.14));
```

### Predicate — فلترة فواتير

```java
public List<Invoice> filterInvoices(List<Invoice> invoices, Predicate<Invoice> operation) {
    List<Invoice> result = new ArrayList<>();
    for (Invoice i : invoices) {
        if (operation.test(i))
            result.add(i);
    }
    return result;
}

// استخدام
List<Invoice> overdue    = filterInvoices(invoices, inv -> inv.isOverDue());
List<Invoice> bigInvoices = filterInvoices(invoices, inv -> inv.getAmount() > 10_000);
List<Invoice> forClient  = filterInvoices(invoices, inv -> inv.getClientId() == clientId);
```

### Function — تحويل بيانات

```java
public List<String> transformInvoices(List<Invoice> invoices, Function<Invoice, String> operation) {
    List<String> result = new ArrayList<>();
    for (Invoice i : invoices) {
        result.add(operation.apply(i));
    }
    return result;
}

// استخدام
List<String> summaries = transformInvoices(invoices, inv -> "Invoice #" + inv.getId());
List<String> emails    = transformInvoices(invoices, inv -> inv.getClient().getEmail());
```

### Supplier — توليد قيم

```java
public Invoice createInvoice(Supplier<Invoice> supplier) {
    return supplier.get();
}

// استخدام
Invoice newInvoice = createInvoice(() -> new Invoice(LocalDate.now()));
```

---

## ٥. الـ Streams API (الطريقة الأحدث والأجمل)

```java
List<Invoice> invoices = getInvoices();

// فلترة + طباعة في سطرين بس!
invoices.stream()
        .filter(inv -> inv.isOverDue())
        .forEach(inv -> System.out.println(inv.getAmount()));

// فلترة + تحويل + تجميع في list
List<String> overdueEmails = invoices.stream()
        .filter(inv -> inv.isOverDue())
        .map(inv -> inv.getClient().getEmail())
        .collect(Collectors.toList());
```

---

## ٦. أسئلة الـ Interview 🎯

### الأسئلة الأساسية (لازم تعرفها)

**Q: إيه هو الـ Functional Programming؟**

> A: أسلوب برمجة بيتعامل مع الـ functions كـ first-class citizens، يعني تقدر تبعتها كـ parameter، ترجعها من method، أو تحطها في متغير.

**Q: إيه الفرق بين `Consumer` و `Function`؟**

> A: الـ `Consumer` بتاخد قيمة وبترجع `void` (زي طباعة). الـ `Function` بتاخد قيمة وبترجع قيمة تانية (زي تحويل `Invoice` لـ `String`).

**Q: إيه هو الـ Lambda Expression؟**

> A: طريقة مختصرة لكتابة anonymous method. بدل ما تعرّف class جديدة أو method، بتكتب السلوك مباشرة في سطر.

**Q: إيه هو الـ Functional Interface؟**

> A: Interface عنده **method واحدة بس abstract**. ده اللي بيخلي الـ Lambda تشتغل معاه. مثال: `Predicate`، `Consumer`، `Function`، `Supplier`.

**Q: إيه الفرق بين `Predicate` و `Function`؟**

> A: الـ `Predicate` دايماً بترجع `boolean` وبتستخدمها للفلترة. الـ `Function` بترجع أي نوع تحدده.

---

### أسئلة متوسطة

**Q: إيه هو الـ `@FunctionalInterface` annotation؟**

> A: بيقول للـ compiler إن الـ interface ده لازم يكون فيه method واحدة بس abstract. لو حاولت تضيف تانية، هيديك compile error.

```java
@FunctionalInterface
interface MyOperation {
    int execute(int a, int b);
}
```

**Q: إيه الفرق بين `map()` و `filter()` في الـ Streams؟**

> A: `filter()` بتاخد `Predicate` وبتشيل العناصر اللي مش بتحقق الشرط. `map()` بتاخد `Function` وبتحوّل كل عنصر لحاجة تانية.

**Q: إيه هو الـ Method Reference؟**

> A: طريقة أقصر من الـ Lambda لما بتعمل إلا استدعاء method موجودة:

```java
// Lambda
invoices.forEach(inv -> System.out.println(inv));

// Method Reference — نفس الكلام بس أقصر
invoices.forEach(System.out::println);
```

---

### أسئلة متقدمة

**Q: إيه الفرق بين `stream()` و `parallelStream()`؟**

> A: الـ `stream()` بتشتغل بـ thread واحدة. الـ `parallelStream()` بتقسّم الشغل على أكتر من thread تلقائياً — أسرع في الـ collections الكبيرة، بس محتاج تتأكد إن الكود thread-safe.

**Q: إيه هو الـ Optional وعلاقته بالـ Functional Programming؟**

> A: بيستخدم بدل `null` عشان يتجنب الـ `NullPointerException`، وبيدعم الـ functional style:

```java
Optional<Invoice> invoice = findInvoice(id);
invoice.ifPresent(inv -> System.out.println(inv.getAmount()));
String amount = invoice.map(inv -> inv.getAmount().toString()).orElse("Not found");
```

**Q: إيه هو الـ Closure في Java؟**

> A: الـ Lambda قادرة تستخدم variables من الـ scope اللي فيها، بس بشرط إنها `effectively final` (مش بتتغير):

```java
double taxRate = 0.14; // effectively final
invoices.forEach(inv -> System.out.println(inv.getAmount() * taxRate)); // ✅
taxRate = 0.15; // لو عملت ده، الـ Lambda هتديك error ❌
```

---

## ٧. ملخص سريع

```
Lambda          = method من غير اسم
Consumer        = بتاخد → void         (accept)
Predicate       = بتاخد → boolean      (test)
Function        = بتاخد → قيمة تانية   (apply)
Supplier        = مش بتاخد → بترجع     (get)
Streams         = pipeline لمعالجة collections بأسلوب functional
```