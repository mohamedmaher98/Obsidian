# Comparable vs Comparator in Java

## Comparable - "أنا بعرف أقارن نفسي"

لما class بيعمل implement لـ `Comparable`، يعني كل object بيعرف يقارن **نفسه** بواحد تاني من نفس النوع.

الـ interface فيها method واحدة بس:

```java
public interface Comparable<T> {
    int compareTo(T other);
}
```

### مثال

```java
public class Employee implements Comparable<Employee> {
    String name;
    int salary;

    @Override
    public int compareTo(Employee other) {
        return this.salary - other.salary; // ترتيب حسب المرتب
    }
}
```

الاستخدام:

```java
Collections.sort(employees); // خلاص هو عارف يرتب نفسه
```

### القيمة الراجعة من `compareTo`

- **سالب** → أنا أصغر (أنا الأول)
- **صفر** → متساويين
- **موجب** → أنا أكبر (التاني الأول)

### مثال عملي على القيم

```java
Employee ali = new Employee("Ali", 5000);
Employee omar = new Employee("Omar", 8000);

ali.compareTo(omar);  // 5000 - 8000 = -3000 (سالب) → ali الأول
omar.compareTo(ali);  // 8000 - 5000 = 3000 (موجب) → ali الأول برضو
```

---

## Comparator - "حد تاني بيقارن بين اتنين"

الobject **خارجي** مسؤول عن المقارنة. مفيد لما عايز ترتب بأكتر من طريقة.

الـ interface فيها method واحدة أساسية:

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

### مثال

```java
Comparator<Employee> byName = (e1, e2) -> e1.name.compareTo(e2.name);
Comparator<Employee> bySalary = (e1, e2) -> e1.salary - e2.salary;
Comparator<Employee> bySalaryDesc = (e1, e2) -> e2.salary - e1.salary;
```

الاستخدام:

```java
Collections.sort(employees, byName);       // رتب بالاسم
Collections.sort(employees, bySalary);     // رتب بالمرتب تصاعدي
Collections.sort(employees, bySalaryDesc); // رتب بالمرتب تنازلي
```

نفس الـ list، تلات ترتيبات مختلفة، من غير ما نغير حاجة في الـ Employee class.

---

## الفرق بينهم

| | Comparable | Comparator |
|---|---|---|
| **الـ method** | `compareTo(T other)` | `compare(T o1, T o2)` |
| **مين بيقارن** | الـ object نفسه | object خارجي |
| **عدد طرق الترتيب** | طريقة **واحدة** بس | **أي عدد** من الطرق |
| **بيتعدل فين** | جوه الـ class نفسه | بره الـ class |
| **الـ package** | `java.lang` | `java.util` |

---

## امتى تستخدم إيه؟

### استخدم Comparable لما

يكون فيه ترتيب **طبيعي واحد واضح** ومنطقي للـ class:

- الأرقام → من الأصغر للأكبر
- النصوص → ترتيب أبجدي
- التواريخ → من الأقدم للأحدث
- الموظفين → بالرقم الوظيفي مثلا

```java
public class Invoice implements Comparable<Invoice> {
    int invoiceNumber;

    @Override
    public int compareTo(Invoice other) {
        return this.invoiceNumber - other.invoiceNumber;
    }
}
```

### استخدم Comparator لما

- عايز **أكتر من طريقة** ترتيب لنفس الـ class
- الـ class **مش بتاعك** (مثلا من library) ومش قادر تعدله
- عايز ترتيب **مؤقت** في مكان معين بس

```java
// نفس الـ Invoice ممكن أرتبها بطرق مختلفة
Comparator<Invoice> byDate = (a, b) -> a.date.compareTo(b.date);
Comparator<Invoice> byAmount = (a, b) -> a.amount.compareTo(b.amount);
Comparator<Invoice> byCustomer = (a, b) -> a.customer.compareTo(b.customer);
```

---

## Comparator Chaining - تركيب أكتر من ترتيب

من أقوى المميزات في Comparator إنك تقدر تركب أكتر من معيار ترتيب مع بعض:

```java
employees.sort(
    Comparator.comparing(Employee::getDepartment)
              .thenComparing(Employee::getSalary)
              .reversed()
              .thenComparing(Employee::getName)
);
```

ده معناه: رتب بالقسم الأول، ولو نفس القسم رتب بالمرتب، ولو نفس المرتب رتب بالاسم.

### Method References مع Comparator.comparing

```java
// بدل ما تكتب lambda كامل
Comparator<Employee> bySalary = Comparator.comparing(Employee::getSalary);

// ترتيب عكسي
Comparator<Employee> bySalaryDesc = Comparator.comparing(Employee::getSalary).reversed();

// ترتيب مع التعامل مع null
Comparator<Employee> byName = Comparator.comparing(
    Employee::getName,
    Comparator.nullsLast(String::compareTo)
);
```

---

## أمثلة عملية مع Streams

```java
// أعلى 5 مرتبات
employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .limit(5)
    .forEach(System.out::println);

// أقل موظف مرتب
Employee lowest = employees.stream()
    .min(Comparator.comparing(Employee::getSalary))
    .orElse(null);

// أعلى موظف مرتب
Employee highest = employees.stream()
    .max(Comparator.comparing(Employee::getSalary))
    .orElse(null);
```

---

## ملاحظات مهمة

### خلي بالك من Integer Overflow

لما تطرح أرقام كبيرة ممكن يحصل overflow:

```java
// خطر - ممكن يحصل overflow
public int compareTo(Employee other) {
    return this.salary - other.salary;
}

// آمن
public int compareTo(Employee other) {
    return Integer.compare(this.salary, other.salary);
}
```

### الComparable و equals لازم يكونوا متسقين

لو `a.compareTo(b) == 0` يبقى المفروض `a.equals(b) == true` عشان ما يحصلش مشاكل مع `TreeSet` و `TreeMap`.

### الـ Collections اللي بتعتمد على الترتيب

- ال`TreeSet` و `TreeMap` بيستخدموا `Comparable` أو `Comparator` للترتيب
- ال`Collections.sort()` و `Arrays.sort()` بيستخدموا `Comparable` لو ما اديتوش `Comparator`
- ال`PriorityQueue` بتستخدم `Comparable` أو `Comparator` لتحديد الأولوية


---
