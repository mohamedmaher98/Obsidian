# 📦 Java Records - دليل شامل بالمصري

## المحتويات
- [مقدمة](#مقدمة)
- [المشكلة والحل](#المشكلة-والحل)
- [الـ Components](#الـ-components)
- [أنواع الـ Constructors](#أنواع-الـ-constructors)
- [إضافة Methods](#إضافة-methods)
- [Implementing Interfaces](#implementing-interfaces)
- [Records مع Generics](#records-مع-generics)
- [Record Patterns](#record-patterns)
- [أمثلة عملية](#أمثلة-عملية)
- [أسئلة الانترفيو](#أسئلة-الانترفيو)

---

## مقدمة

### يعني إيه Record؟

الـ **Record** هو نوع خاص من الـ classes اتقدم في **Java 14** كـ preview وبقى stable في **Java 16**.

الهدف منه إنه يكون **carrier للـ data** - يعني class بسيط بيشيل data ومش محتاج منه حاجة تانية.

### الفكرة الأساسية

بدل ما تكتب class كامل بـ:
- Fields
- Constructor  
- Getters
- equals()
- hashCode()
- toString()

تكتب سطر واحد بس! 🎉

---

## المشكلة والحل

### ❌ الطريقة القديمة (POJO)

```java
public class Employee {
    private final String name;
    private final int age;
    private final String department;

    public Employee(String name, int age, String department) {
        this.name = name;
        this.age = age;
        this.department = department;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public String getDepartment() {
        return department;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Employee employee = (Employee) o;
        return age == employee.age &&
               Objects.equals(name, employee.name) &&
               Objects.equals(department, employee.department);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age, department);
    }

    @Override
    public String toString() {
        return "Employee{" +
               "name='" + name + '\'' +
               ", age=" + age +
               ", department='" + department + '\'' +
               '}';
    }
}
```

**المشكلة:** 60+ سطر عشان class بسيط! ده اسمه **Boilerplate Code**.

### ✅ الحل: Record

```java
public record Employee(String name, int age, String department) {}
```

**سطر واحد بس!**

Java بتعملك كل حاجة أوتوماتيك:
- ✅ Private final fields
- ✅ Canonical Constructor
- ✅ Accessor methods (زي getters بس من غير `get`)
- ✅ `equals()` - بيقارن كل الـ components
- ✅ `hashCode()` - based على كل الـ components
- ✅ `toString()` - بيطبع كل الـ components

---

## الـ Components

### تعريف

الحاجات اللي بين القوسين في الـ Record اسمها **Components** أو **Record Components**:

```java
public record Person(String name, int age) {}
//                   ↑           ↑
//              component 1  component 2
```

### كل Component بيعمل إيه؟

1. **Private final field** بنفس الاسم والنوع
2. **Accessor method** بنفس الاسم (من غير `get`)

```java
public record Person(String name, int age) {}

// Java بتعمل ده تلقائياً:
// private final String name;
// private final int age;
// 
// public String name() { return this.name; }
// public int age() { return this.age; }
```

### الاستخدام

```java
Person person = new Person("أحمد", 25);

// الوصول للـ data
System.out.println(person.name());  // أحمد
System.out.println(person.age());   // 25

// toString()
System.out.println(person);  // Person[name=أحمد, age=25]

// equals()
Person person2 = new Person("أحمد", 25);
System.out.println(person.equals(person2));  // true

// hashCode()
System.out.println(person.hashCode() == person2.hashCode());  // true
```

---

## أنواع الـ Constructors

### 1. Canonical Constructor (الافتراضي)

ده اللي Java بتعمله لوحدها تلقائياً:

```java
public record Person(String name, int age) {}

// Java بتولد ده:
// public Person(String name, int age) {
//     this.name = name;
//     this.age = age;
// }
```

### 2. Compact Constructor ⭐

ده أهم نوع - بيخليك تضيف **validation** من غير ما تكتب كل الكود:

```java
public record Person(String name, int age) {
    
    // لاحظ: مفيش parameters بين القوسين!
    public Person {
        // Validation
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("الاسم مينفعش يكون فاضي!");
        }
        if (age < 0) {
            throw new IllegalArgumentException("العمر لازم يكون موجب!");
        }
        
        // ممكن تعدل القيم
        name = name.trim().toUpperCase();
        
        // ⚠️ مش محتاج تكتب this.name = name
        // Java بتعمل الـ assignment أوتوماتيك في الآخر!
    }
}
```

**الفرق عن الـ Regular Constructor:**
- مفيش parameters
- مفيش `this.field = param`
- Java بتعمل الـ assignment في الآخر

### 3. Custom Canonical Constructor

لو عايز تكتب الـ constructor بالكامل:

```java
public record Person(String name, int age) {
    
    // لازم تاخد نفس الـ parameters بالظبط
    public Person(String name, int age) {
        // Validation
        if (name == null) {
            throw new IllegalArgumentException("الاسم مش ممكن يكون null");
        }
        
        // لازم تعمل assignment لكل الـ fields
        this.name = name.toUpperCase();
        this.age = Math.max(0, age);
    }
}
```

### 4. Non-Canonical Constructors (إضافية)

```java
public record Person(String name, int age) {
    
    // Constructor بـ parameter واحد
    public Person(String name) {
        this(name, 0);  // ⚠️ لازم يكون أول statement
    }
    
    // Constructor من غير parameters
    public Person() {
        this("Unknown", 0);
    }
    
    // Copy constructor مع تعديل
    public Person withAge(int newAge) {
        return new Person(this.name, newAge);
    }
}

// الاستخدام
Person p1 = new Person("أحمد", 25);
Person p2 = new Person("محمد");      // age = 0
Person p3 = new Person();            // name = "Unknown", age = 0
Person p4 = p1.withAge(30);          // name = "أحمد", age = 30
```

---

## إضافة Methods

### Instance Methods

```java
public record Rectangle(double width, double height) {
    
    public double area() {
        return width * height;
    }
    
    public double perimeter() {
        return 2 * (width + height);
    }
    
    public boolean isSquare() {
        return width == height;
    }
    
    public Rectangle scale(double factor) {
        return new Rectangle(width * factor, height * factor);
    }
}

// الاستخدام
Rectangle rect = new Rectangle(5, 3);
System.out.println(rect.area());       // 15.0
System.out.println(rect.perimeter());  // 16.0
System.out.println(rect.isSquare());   // false

Rectangle scaled = rect.scale(2);
System.out.println(scaled);  // Rectangle[width=10.0, height=6.0]
```

### Static Methods و Fields

```java
public record Temperature(double value, String unit) {
    
    // Static fields
    public static final Temperature ABSOLUTE_ZERO = 
        new Temperature(-273.15, "C");
    
    // Static factory methods
    public static Temperature celsius(double value) {
        return new Temperature(value, "C");
    }
    
    public static Temperature fahrenheit(double value) {
        return new Temperature(value, "F");
    }
    
    // Conversion methods
    public Temperature toCelsius() {
        if (unit.equals("C")) return this;
        return new Temperature((value - 32) * 5/9, "C");
    }
    
    public Temperature toFahrenheit() {
        if (unit.equals("F")) return this;
        return new Temperature(value * 9/5 + 32, "F");
    }
}

// الاستخدام
Temperature t1 = Temperature.celsius(100);
Temperature t2 = t1.toFahrenheit();
System.out.println(t2);  // Temperature[value=212.0, unit=F]
```

### Override Accessor Methods

```java
public record Person(String name, int age) {
    
    // Override accessor method
    @Override
    public String name() {
        return name.toUpperCase();  // دايماً capital
    }
    
    @Override
    public int age() {
        return Math.max(0, age);  // أبداً مش سالب
    }
}
```

### Override equals, hashCode, toString

```java
public record Product(String code, String name, double price) {
    
    @Override
    public String toString() {
        return String.format("📦 %s (%s) - %.2f جنيه", name, code, price);
    }
    
    @Override
    public boolean equals(Object o) {
        // المقارنة بالـ code بس
        if (this == o) return true;
        if (!(o instanceof Product p)) return false;
        return code.equals(p.code);
    }
    
    @Override
    public int hashCode() {
        return code.hashCode();
    }
}
```

---

## Implementing Interfaces

```java
public interface Printable {
    void print();
}

public interface Validatable {
    boolean isValid();
}

public record Email(String address) implements Printable, Validatable {
    
    private static final Pattern EMAIL_PATTERN = 
        Pattern.compile("^[A-Za-z0-9+_.-]+@(.+)$");
    
    @Override
    public void print() {
        System.out.println("📧 " + address);
    }
    
    @Override
    public boolean isValid() {
        return address != null && EMAIL_PATTERN.matcher(address).matches();
    }
}

// مع Comparable
public record Student(String name, double gpa) implements Comparable<Student> {
    
    @Override
    public int compareTo(Student other) {
        return Double.compare(other.gpa, this.gpa);  // ترتيب تنازلي
    }
}

// الاستخدام
List<Student> students = new ArrayList<>(List.of(
    new Student("أحمد", 3.5),
    new Student("محمد", 3.8),
    new Student("علي", 3.2)
));

Collections.sort(students);
students.forEach(s -> System.out.println(s.name() + ": " + s.gpa()));
// محمد: 3.8
// أحمد: 3.5
// علي: 3.2
```

---

## الحاجات الممنوعة في Records ⛔

### 1. Instance Fields إضافية

```java
public record Person(String name, int age) {
    // ❌ Compile Error!
    private String nickname;
    
    // ❌ Compile Error!
    public int birthYear;
}
```

**ليه؟** لأن الـ Record لازم يكون **transparent carrier** للـ data.

### 2. Extend Classes

```java
public class Animal {}

// ❌ Compile Error!
public record Dog(String name) extends Animal {}
```

**ليه؟** لأن كل Records بتـ extend `java.lang.Record` implicitly.

### 3. Abstract Records

```java
// ❌ Compile Error!
public abstract record Shape(String color) {}
```

**ليه؟** لأن Records هي **implicitly final**.

### 4. Non-final Fields

```java
// الـ fields دايماً final - مش ممكن تغيرها
Person p = new Person("أحمد", 25);
// ❌ مفيش setters!
// p.setName("محمد");  // مش موجود
```

---

## Records مع Generics

### Record واحد مع Type Parameter

```java
public record Box<T>(T content) {
    
    public boolean isEmpty() {
        return content == null;
    }
    
    public <R> Box<R> map(Function<T, R> mapper) {
        return new Box<>(mapper.apply(content));
    }
}

// الاستخدام
Box<String> stringBox = new Box<>("Hello");
Box<Integer> intBox = new Box<>(42);

Box<Integer> lengthBox = stringBox.map(String::length);
System.out.println(lengthBox.content());  // 5
```

### Multiple Type Parameters

```java
public record Pair<K, V>(K key, V value) {
    
    public Pair<V, K> swap() {
        return new Pair<>(value, key);
    }
    
    public static <K, V> Pair<K, V> of(K key, V value) {
        return new Pair<>(key, value);
    }
}

public record Triple<A, B, C>(A first, B second, C third) {
    
    public static <A, B, C> Triple<A, B, C> of(A a, B b, C c) {
        return new Triple<>(a, b, c);
    }
}

// الاستخدام
Pair<String, Integer> pair = Pair.of("age", 25);
Triple<String, Integer, Boolean> triple = Triple.of("test", 1, true);
```

### Bounded Type Parameters

```java
public record NumberPair<T extends Number>(T first, T second) {
    
    public double sum() {
        return first.doubleValue() + second.doubleValue();
    }
    
    public double average() {
        return sum() / 2;
    }
}

// الاستخدام
NumberPair<Integer> intPair = new NumberPair<>(10, 20);
NumberPair<Double> doublePair = new NumberPair<>(3.14, 2.71);

System.out.println(intPair.sum());      // 30.0
System.out.println(doublePair.average()); // 2.925
```

---

## Nested و Local Records

### Nested Records

```java
public class Order {
    
    // Nested Record
    public record Item(String name, int quantity, double price) {
        public double total() {
            return quantity * price;
        }
    }
    
    // Nested Record تاني
    public record Summary(int totalItems, double totalAmount) {}
    
    private final List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    public Summary getSummary() {
        int totalItems = items.stream().mapToInt(Item::quantity).sum();
        double totalAmount = items.stream().mapToDouble(Item::total).sum();
        return new Summary(totalItems, totalAmount);
    }
}

// الاستخدام
Order order = new Order();
order.addItem(new Order.Item("لابتوب", 1, 15000));
order.addItem(new Order.Item("ماوس", 2, 500));

Order.Summary summary = order.getSummary();
System.out.println("عدد المنتجات: " + summary.totalItems());
System.out.println("الإجمالي: " + summary.totalAmount());
```

### Local Records (جوه Method)

```java
public class DataProcessor {
    
    public Map<String, Long> processData(List<String> data) {
        // Local Record - متاح جوه الـ method بس
        record WordCount(String word, long count) {}
        
        return data.stream()
            .flatMap(line -> Arrays.stream(line.split("\\s+")))
            .map(String::toLowerCase)
            .collect(Collectors.groupingBy(
                Function.identity(),
                Collectors.counting()
            ))
            .entrySet().stream()
            .map(e -> new WordCount(e.getKey(), e.getValue()))
            .sorted((a, b) -> Long.compare(b.count(), a.count()))
            .collect(Collectors.toMap(
                WordCount::word,
                WordCount::count,
                (a, b) -> a,
                LinkedHashMap::new
            ));
    }
}
```

---

## Record Patterns (Java 21+)

### Pattern Matching مع instanceof

```java
public record Point(int x, int y) {}

public void process(Object obj) {
    // الطريقة القديمة
    if (obj instanceof Point) {
        Point p = (Point) obj;
        System.out.println(p.x() + ", " + p.y());
    }
    
    // الطريقة الجديدة (Java 16+)
    if (obj instanceof Point p) {
        System.out.println(p.x() + ", " + p.y());
    }
    
    // Record Pattern (Java 21+)
    if (obj instanceof Point(int x, int y)) {
        System.out.println(x + ", " + y);
    }
}
```

### Nested Record Patterns

```java
public record Point(int x, int y) {}
public record Line(Point start, Point end) {}
public record Triangle(Point a, Point b, Point c) {}

public void describe(Object shape) {
    switch (shape) {
        case Point(int x, int y) -> 
            System.out.printf("نقطة عند (%d, %d)%n", x, y);
            
        case Line(Point(int x1, int y1), Point(int x2, int y2)) -> 
            System.out.printf("خط من (%d,%d) إلى (%d,%d)%n", x1, y1, x2, y2);
            
        case Triangle(Point a, Point b, Point c) -> 
            System.out.println("مثلث بثلاث نقاط");
            
        default -> 
            System.out.println("شكل غير معروف");
    }
}
```

### Guards في Patterns

```java
public String classify(Object obj) {
    return switch (obj) {
        case Point(int x, int y) when x == 0 && y == 0 -> "نقطة الأصل";
        case Point(int x, int y) when x == 0 -> "على محور Y";
        case Point(int x, int y) when y == 0 -> "على محور X";
        case Point(int x, int y) when x == y -> "على القطر";
        case Point(int x, int y) -> "نقطة عادية";
        default -> "مش نقطة";
    };
}
```

---

## Records مع Sealed Classes

```java
// Sealed interface
public sealed interface Result<T> permits Success, Failure {}

// Record implementations
public record Success<T>(T value) implements Result<T> {}
public record Failure<T>(String error) implements Result<T> {}

// استخدام مع Pattern Matching
public <T> void handleResult(Result<T> result) {
    switch (result) {
        case Success<T>(T value) -> System.out.println("نجح: " + value);
        case Failure<T>(String error) -> System.out.println("فشل: " + error);
    }
}

// مثال عملي
public Result<Integer> divide(int a, int b) {
    if (b == 0) {
        return new Failure<>("القسمة على صفر!");
    }
    return new Success<>(a / b);
}
```

---

## أمثلة عملية

### 1. API Response

```java
public record ApiResponse<T>(
    int statusCode,
    String message,
    T data,
    LocalDateTime timestamp
) {
    public ApiResponse {
        if (timestamp == null) {
            timestamp = LocalDateTime.now();
        }
    }
    
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(200, "Success", data, null);
    }
    
    public static <T> ApiResponse<T> error(int code, String message) {
        return new ApiResponse<>(code, message, null, null);
    }
    
    public boolean isSuccess() {
        return statusCode >= 200 && statusCode < 300;
    }
}
```

### 2. Domain Value Objects

```java
public record Money(BigDecimal amount, String currency) {
    
    public Money {
        Objects.requireNonNull(amount, "Amount cannot be null");
        Objects.requireNonNull(currency, "Currency cannot be null");
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        currency = currency.toUpperCase();
    }
    
    public static Money of(double amount, String currency) {
        return new Money(BigDecimal.valueOf(amount), currency);
    }
    
    public Money add(Money other) {
        validateSameCurrency(other);
        return new Money(amount.add(other.amount), currency);
    }
    
    public Money subtract(Money other) {
        validateSameCurrency(other);
        return new Money(amount.subtract(other.amount), currency);
    }
    
    public Money multiply(int factor) {
        return new Money(amount.multiply(BigDecimal.valueOf(factor)), currency);
    }
    
    private void validateSameCurrency(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("Currencies must match");
        }
    }
    
    @Override
    public String toString() {
        return String.format("%s %.2f", currency, amount);
    }
}
```

### 3. Builder Pattern مع Records

```java
public record HttpRequest(
    String method,
    String url,
    Map<String, String> headers,
    String body
) {
    public HttpRequest {
        headers = headers == null ? Map.of() : Map.copyOf(headers);
    }
    
    // Builder class
    public static class Builder {
        private String method = "GET";
        private String url;
        private Map<String, String> headers = new HashMap<>();
        private String body;
        
        public Builder method(String method) {
            this.method = method;
            return this;
        }
        
        public Builder url(String url) {
            this.url = url;
            return this;
        }
        
        public Builder header(String key, String value) {
            this.headers.put(key, value);
            return this;
        }
        
        public Builder body(String body) {
            this.body = body;
            return this;
        }
        
        public HttpRequest build() {
            Objects.requireNonNull(url, "URL is required");
            return new HttpRequest(method, url, headers, body);
        }
    }
    
    public static Builder builder() {
        return new Builder();
    }
}

// الاستخدام
HttpRequest request = HttpRequest.builder()
    .method("POST")
    .url("https://api.example.com/data")
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer token123")
    .body("{\"name\": \"test\"}")
    .build();
```

### 4. Immutable Collections في Records

```java
public record ShoppingCart(
    String customerId,
    List<CartItem> items
) {
    public record CartItem(String productId, int quantity, BigDecimal price) {
        public BigDecimal total() {
            return price.multiply(BigDecimal.valueOf(quantity));
        }
    }
    
    // Defensive copy في الـ constructor
    public ShoppingCart {
        items = items == null ? List.of() : List.copyOf(items);
    }
    
    // بدل modification، بنرجع instance جديد
    public ShoppingCart addItem(CartItem item) {
        List<CartItem> newItems = new ArrayList<>(items);
        newItems.add(item);
        return new ShoppingCart(customerId, newItems);
    }
    
    public ShoppingCart removeItem(String productId) {
        List<CartItem> newItems = items.stream()
            .filter(i -> !i.productId().equals(productId))
            .toList();
        return new ShoppingCart(customerId, newItems);
    }
    
    public BigDecimal total() {
        return items.stream()
            .map(CartItem::total)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

---

## جدول مقارنة

| الخاصية | Record | Class | Lombok @Data |
|---------|--------|-------|--------------|
| Boilerplate | ✅ أقل | ❌ كتير | ✅ أقل |
| Immutable | ✅ دايماً | ❌ اختياري | ❌ mutable |
| Inheritance | ❌ ممنوع | ✅ عادي | ✅ عادي |
| Instance fields إضافية | ❌ ممنوع | ✅ عادي | ✅ عادي |
| IDE Support | ✅ built-in | ✅ عادي | ⚠️ محتاج plugin |
| Reflection | ✅ أحسن | ✅ عادي | ✅ عادي |
| JPA Entity | ❌ صعب | ✅ عادي | ✅ عادي |
| Pattern Matching | ✅ ممتاز | ❌ محدود | ❌ محدود |

---

## إمتى تستخدم Record؟

### ✅ استخدم Record لما:

1. **DTOs** (Data Transfer Objects)
2. **Value Objects** - زي Money, Email, Address
3. **API Responses/Requests**
4. **Configuration objects**
5. **Multiple return values**
6. **Map keys** (equals و hashCode جاهزين)
7. **Event objects**
8. **Temporary data grouping**

### ❌ ماتستخدمش Record لما:

1. محتاج **mutable state**
2. محتاج **inheritance** من class تاني
3. **JPA Entities** (محتاج no-arg constructor و setters)
4. محتاج **lazy initialization**
5. محتاج **complex state management**
6. الـ class هيكبر ويبقى complex

---

# 🎯 أسئلة الانترفيو

## أسئلة نظرية

### س1: إيه هو الـ Record في Java؟
**الإجابة:**
الRecord هو نوع خاص من الـ classes اتقدم في Java 14 (preview) وبقى stable في Java 16. هو **immutable data carrier** - يعني class بيشيل data ومش محتاج منه حاجة تانية. Java بتولد أوتوماتيك: private final fields، canonical constructor، accessor methods، equals()، hashCode()، و toString().

---

### س2: إيه الفرق بين Record و Class العادي؟
**الإجابة:**

| Record                             | Class                                 |
| ---------------------------------- | ------------------------------------- |
| Implicitly final                   | ممكن يكون final أو لا                 |
| مش ممكن يـ extend class            | ممكن يـ extend أي class               |
| الFields دايماً private final      | الFields ممكن تكون أي access modifier |
| مفيش instance fields إضافية        | ممكن تضيف أي عدد من fields            |
| Canonical constructor أوتوماتيك    | لازم تكتب constructor                 |
| Accessor methods أوتوماتيك         | لازم تكتب getters                     |
| equals/hashCode/toString أوتوماتيك | لازم تـ override manually             |

---

### س3: إيه هو الـ Canonical Constructor؟
**الإجابة:**
الـ Canonical Constructor هو الـ constructor الأساسي اللي بياخد كل الـ components كـ parameters بنفس الترتيب. Java بتولده أوتوماتيك، لكن ممكن نعمله override عشان نضيف validation أو نعدل القيم.

```java
public record Person(String name, int age) {
    // Canonical Constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

---

### س4: إيه الفرق بين Compact Constructor و Canonical Constructor؟
**الإجابة:**

**Compact Constructor:**
- مفيش parameters بين القوسين
- مفيش assignment statements
- الJava بتعمل الـ assignment أوتوماتيك في الآخر
- بيستخدم للـ validation والـ normalization

```java
public record Person(String name, int age) {
    public Person {  // Compact - مفيش parameters
        if (age < 0) throw new IllegalArgumentException();
        name = name.trim();
        // مفيش this.name = name
    }
}
```

**Canonical Constructor:**
- لازم يكون فيه كل الـ parameters
- لازم تعمل assignment لكل الـ fields manually

```java
public record Person(String name, int age) {
    public Person(String name, int age) {  // كل الـ parameters
        this.name = name.trim();  // لازم assignment
        this.age = age;
    }
}
```

---

### س5: ليه الـ Records مش بتستخدم `getX()` زي الـ JavaBeans؟
**الإجابة:**
الـ Records بتستخدم accessor methods بنفس اسم الـ component من غير `get` prefix لأنها:
1. أبسط وأقصر
2. بتتبع convention مختلف عن JavaBeans
3. الـ Records مش mutable فمش محتاجين نفرق بين getter و setter
4. بتتماشى مع modern Java style

```java
record Person(String name) {}
Person p = new Person("أحمد");
p.name();  // مش getName()
```

---

### س6: هل ممكن الـ Record يـ implement interface؟
**الإجابة:**
أيوه، الـ Record ممكن يـ implement أي عدد من الـ interfaces:

```java
public record Employee(String name, double salary) 
    implements Comparable<Employee>, Serializable {
    
    @Override
    public int compareTo(Employee other) {
        return Double.compare(this.salary, other.salary);
    }
}
```

---

### س7: هل ممكن الـ Record يـ extend class؟
**الإجابة:**
لا، مش ممكن. كل Records بتـ extend `java.lang.Record` implicitly، و Java مش بتسمح بـ multiple inheritance. لكن الـ Record ممكن يـ implement interfaces.

```java
// ❌ Compile Error
public record Dog(String name) extends Animal {}

// ✅ صح
public record Dog(String name) implements Pet {}
```

---

### س8: إزاي تضيف validation للـ Record؟
**الإجابة:**
باستخدام Compact Constructor أو Custom Canonical Constructor:

```java
public record Email(String address) {
    // Compact Constructor
    public Email {
        Objects.requireNonNull(address, "Email cannot be null");
        if (!address.contains("@")) {
            throw new IllegalArgumentException("Invalid email format");
        }
        address = address.toLowerCase().trim();
    }
}
```

---

### س9: هل ممكن نضيف instance fields للـ Record؟
**الإجابة:**
لا، مش ممكن نضيف instance fields إضافية. الـ Record بيحتوي بس على الـ fields اللي معرفة كـ components. لكن ممكن نضيف static fields:

```java
public record Config(String key, String value) {
    // ❌ مش مسموح
    // private String extraField;
    
    // ✅ static fields مسموح
    public static final Config DEFAULT = new Config("default", "none");
}
```

---

### س10: إيه هو Record Pattern وإمتى نستخدمه؟
**الإجابة:**
Record Pattern (Java 21+) بيسمحلك تعمل destructuring للـ Record components في pattern matching:

```java
public record Point(int x, int y) {}

// بدل كده:
if (obj instanceof Point p) {
    int x = p.x();
    int y = p.y();
}

// نقدر نكتب:
if (obj instanceof Point(int x, int y)) {
    // x و y متاحين مباشرة
}
```

---

### س11: إزاي الـ equals() بتشتغل في Records؟
**الإجابة:**
الـ equals() المولدة أوتوماتيك بتقارن كل الـ components بالترتيب. Two records are equal إذا:
1. نفس الـ type
2. كل الـ components متساوية

```java
record Person(String name, int age) {}

Person p1 = new Person("أحمد", 25);
Person p2 = new Person("أحمد", 25);
Person p3 = new Person("أحمد", 30);

p1.equals(p2);  // true
p1.equals(p3);  // false
```

---

### س12: هل الـ Records thread-safe؟
**الإجابة:**
الـ Records هي **immutable by design** - الـ fields كلها `final`. ده بيخليها thread-safe للـ reading. لكن لو الـ components نفسها mutable objects (زي List أو Date)، الـ Record مش هيكون fully thread-safe:

```java
// ⚠️ مش thread-safe لأن List mutable
record Team(String name, List<String> members) {}

// ✅ thread-safe
record Team(String name, List<String> members) {
    public Team {
        members = List.copyOf(members);  // immutable copy
    }
}
```

---

### س13: إيه الفرق بين Record و Lombok @Value/@Data؟
**الإجابة:**

| Feature | Record | Lombok @Value | Lombok @Data |
|---------|--------|---------------|--------------|
| Built-in | ✅ | ❌ | ❌ |
| Immutable | ✅ دايماً | ✅ | ❌ |
| IDE support | ✅ native | ⚠️ plugin needed | ⚠️ plugin needed |
| Pattern matching | ✅ | ❌ | ❌ |
| Inheritance | ❌ | ⚠️ محدود | ✅ |
| Extra fields | ❌ | ✅ | ✅ |
| Compile-time | ✅ | ⚠️ annotation processor | ⚠️ annotation processor |

---

### س14: إزاي نستخدم Records مع JPA/Hibernate؟
**الإجابة:**
الـ Records صعب استخدامها كـ JPA Entities لأنها:
1. مفيش no-arg constructor
2. مفيش setters
3. الـ fields final

لكن ممكن نستخدمها كـ:
- **DTOs** للـ projections
- **Value Objects** embedded

```java
// كـ DTO/Projection
public record UserDTO(String name, String email) {}

@Query("SELECT new com.example.UserDTO(u.name, u.email) FROM User u")
List<UserDTO> findAllUserDTOs();
```

---

### س15: إيه هي الـ Local Records؟
**الإجابة:**
Local Records هي Records معرفة جوه method، بتستخدم لـ temporary data grouping:

```java
public void processData(List<String> data) {
    // Local Record
    record Entry(String key, int count) {}
    
    Map<String, Long> counts = data.stream()
        .collect(Collectors.groupingBy(
            Function.identity(),
            Collectors.counting()
        ));
    
    List<Entry> entries = counts.entrySet().stream()
        .map(e -> new Entry(e.getKey(), e.getValue().intValue()))
        .sorted(Comparator.comparingInt(Entry::count).reversed())
        .toList();
}
```

---

## أسئلة عملية (Code)

### س16: اكتب Record للـ Money مع validation
```java
public record Money(BigDecimal amount, String currency) {
    
    public Money {
        Objects.requireNonNull(amount, "Amount cannot be null");
        Objects.requireNonNull(currency, "Currency cannot be null");
        
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        
        if (currency.length() != 3) {
            throw new IllegalArgumentException("Currency must be 3 characters");
        }
        
        currency = currency.toUpperCase();
    }
    
    public Money add(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("Currencies must match");
        }
        return new Money(amount.add(other.amount), currency);
    }
    
    public Money multiply(int factor) {
        return new Money(amount.multiply(BigDecimal.valueOf(factor)), currency);
    }
}
```

---

### س17: اكتب Record للـ Range مع validation
```java
public record Range(int start, int end) {
    
    public Range {
        if (start > end) {
            throw new IllegalArgumentException(
                "Start must be <= end: " + start + " > " + end
            );
        }
    }
    
    public boolean contains(int value) {
        return value >= start && value <= end;
    }
    
    public boolean overlaps(Range other) {
        return this.start <= other.end && other.start <= this.end;
    }
    
    public int length() {
        return end - start + 1;
    }
    
    public static Range of(int start, int end) {
        return new Range(start, end);
    }
    
    public static Range single(int value) {
        return new Range(value, value);
    }
}
```

---

### س18: اكتب sealed interface مع Record implementations
```java
public sealed interface Shape permits Circle, Rectangle, Triangle {
    double area();
    double perimeter();
}

public record Circle(double radius) implements Shape {
    
    public Circle {
        if (radius <= 0) {
            throw new IllegalArgumentException("Radius must be positive");
        }
    }
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public double perimeter() {
        return 2 * Math.PI * radius;
    }
}

public record Rectangle(double width, double height) implements Shape {
    
    public Rectangle {
        if (width <= 0 || height <= 0) {
            throw new IllegalArgumentException("Dimensions must be positive");
        }
    }
    
    @Override
    public double area() {
        return width * height;
    }
    
    @Override
    public double perimeter() {
        return 2 * (width + height);
    }
    
    public boolean isSquare() {
        return width == height;
    }
}

public record Triangle(double a, double b, double c) implements Shape {
    
    public Triangle {
        if (a <= 0 || b <= 0 || c <= 0) {
            throw new IllegalArgumentException("Sides must be positive");
        }
        if (a + b <= c || b + c <= a || a + c <= b) {
            throw new IllegalArgumentException("Invalid triangle sides");
        }
    }
    
    @Override
    public double area() {
        double s = (a + b + c) / 2;
        return Math.sqrt(s * (s - a) * (s - b) * (s - c));
    }
    
    @Override
    public double perimeter() {
        return a + b + c;
    }
}

// استخدام مع Pattern Matching
public static String describe(Shape shape) {
    return switch (shape) {
        case Circle(double r) -> "دائرة بنصف قطر " + r;
        case Rectangle(double w, double h) when w == h -> "مربع بضلع " + w;
        case Rectangle(double w, double h) -> "مستطيل " + w + "x" + h;
        case Triangle(double a, double b, double c) -> "مثلث بأضلاع " + a + "," + b + "," + c;
    };
}
```

---

### س19: اكتب Record Pattern مع nested records
```java
public record Address(String street, String city, String country) {}
public record Person(String name, int age, Address address) {}
public record Company(String name, Person ceo, List<Person> employees) {}

// Pattern matching examples
public static void printInfo(Object obj) {
    switch (obj) {
        case Person(String name, int age, Address(String street, String city, _)) 
            when age >= 18 -> 
                System.out.printf("%s (%d سنة) يسكن في %s، %s%n", 
                    name, age, street, city);
        
        case Person(String name, int age, null) -> 
            System.out.printf("%s (%d سنة) - عنوان غير معروف%n", name, age);
        
        case Company(String name, Person(String ceoName, _, _), var employees) ->
            System.out.printf("شركة %s - المدير: %s - عدد الموظفين: %d%n",
                name, ceoName, employees.size());
        
        default -> 
            System.out.println("Unknown type");
    }
}
```

---

### س20: اكتب Result type باستخدام sealed interface و Records
```java
public sealed interface Result<T> {
    
    record Success<T>(T value) implements Result<T> {
        public Success {
            Objects.requireNonNull(value, "Success value cannot be null");
        }
    }
    
    record Failure<T>(String error, Exception cause) implements Result<T> {
        public Failure(String error) {
            this(error, null);
        }
    }
    
    // Factory methods
    static <T> Result<T> success(T value) {
        return new Success<>(value);
    }
    
    static <T> Result<T> failure(String error) {
        return new Failure<>(error);
    }
    
    static <T> Result<T> failure(Exception e) {
        return new Failure<>(e.getMessage(), e);
    }
    
    // Utility methods
    default boolean isSuccess() {
        return this instanceof Success;
    }
    
    default boolean isFailure() {
        return this instanceof Failure;
    }
    
    default T getOrElse(T defaultValue) {
        return switch (this) {
            case Success<T>(T value) -> value;
            case Failure<T> f -> defaultValue;
        };
    }
    
    default <R> Result<R> map(Function<T, R> mapper) {
        return switch (this) {
            case Success<T>(T value) -> Result.success(mapper.apply(value));
            case Failure<T>(String error, Exception cause) -> new Failure<>(error, cause);
        };
    }
    
    default <R> Result<R> flatMap(Function<T, Result<R>> mapper) {
        return switch (this) {
            case Success<T>(T value) -> mapper.apply(value);
            case Failure<T>(String error, Exception cause) -> new Failure<>(error, cause);
        };
    }
}

// الاستخدام
public Result<User> findUser(String id) {
    try {
        User user = userRepository.findById(id);
        return user != null 
            ? Result.success(user) 
            : Result.failure("User not found");
    } catch (Exception e) {
        return Result.failure(e);
    }
}

// معالجة النتيجة
Result<User> result = findUser("123");

String message = switch (result) {
    case Result.Success<User>(User user) -> "مرحباً " + user.name();
    case Result.Failure<User>(String error, _) -> "خطأ: " + error;
};
```

---

## أسئلة متقدمة

### س21: إزاي تعمل deep copy لـ Record فيه mutable objects؟
```java
public record Team(String name, List<Player> players) {
    
    // Defensive copy في الـ constructor
    public Team {
        Objects.requireNonNull(name);
        Objects.requireNonNull(players);
        // Deep copy
        players = players.stream()
            .map(p -> new Player(p.name(), p.number()))
            .collect(Collectors.toUnmodifiableList());
    }
    
    // Defensive copy في الـ accessor
    @Override
    public List<Player> players() {
        return List.copyOf(players);
    }
}

public record Player(String name, int number) {}
```

---

### س22: إيه المشاكل المحتملة مع Records و reflection؟
**الإجابة:**
1. **Private fields**: الـ fields دايماً private، لكن ممكن الوصول ليها بـ reflection
2. **No setters**: مش هتلاقي setters، فالـ frameworks اللي بتعتمد عليها هتفشل
3. **Final fields**: حتى بـ reflection، تغيير final fields ممكن يسبب مشاكل
4. **Record Components API**: Java 16+ وفرت `getRecordComponents()` للـ reflection

```java
// الحصول على الـ components
RecordComponent[] components = Person.class.getRecordComponents();
for (RecordComponent comp : components) {
    System.out.println(comp.getName() + ": " + comp.getType());
}
```

---

### س23: إزاي تستخدم Records كـ Map keys بطريقة صحيحة؟
```java
// ✅ Records ممتازة كـ Map keys لأن equals و hashCode جاهزين
record CacheKey(String userId, String resourceType) {}

Map<CacheKey, Object> cache = new ConcurrentHashMap<>();

// الاستخدام
CacheKey key = new CacheKey("user123", "profile");
cache.put(key, userProfile);

// هيشتغل صح حتى لو عملت key جديد
CacheKey lookupKey = new CacheKey("user123", "profile");
Object result = cache.get(lookupKey);  // ✅ هيرجع userProfile
```

---

### س24: إزاي تعمل custom serialization لـ Record؟
```java
public record SecureUser(String username, String password) implements Serializable {
    
    @Serial
    private static final long serialVersionUID = 1L;
    
    // Custom serialization - encrypt password
    @Serial
    private Object writeReplace() {
        return new SerializationProxy(username, encrypt(password));
    }
    
    private static String encrypt(String value) {
        // encryption logic
        return Base64.getEncoder().encodeToString(value.getBytes());
    }
    
    private static String decrypt(String value) {
        // decryption logic
        return new String(Base64.getDecoder().decode(value));
    }
    
    private record SerializationProxy(String username, String encryptedPassword) 
        implements Serializable {
        
        @Serial
        private Object readResolve() {
            return new SecureUser(username, decrypt(encryptedPassword));
        }
    }
}
```

---

### س25: إيه الفرق بين Records في Java و data classes في Kotlin؟
**الإجابة:**

| Feature | Java Record | Kotlin Data Class |
|---------|-------------|-------------------|
| Mutability | دايماً immutable | ممكن var (mutable) |
| Inheritance | ممنوع | ممنوع extends، بس ممكن interfaces |
| copy() | مفيش built-in | ✅ موجود |
| componentN() | مفيش | ✅ موجود |
| Default values | ❌ | ✅ |
| Named arguments | ❌ | ✅ |
| No-arg constructor | ❌ | ممكن مع default values |

---

## نصائح للانترفيو 💡

1. **اعرف الفرق** بين Record و Class و Lombok
2. **افهم الـ immutability** وليه مهمة
3. **اعرف الـ limitations** - مفيش inheritance، مفيش extra fields
4. **اتكلم عن use cases** - DTOs, Value Objects, API responses
5. **اعرف الـ Pattern Matching** لو Java 21+
6. **افهم الـ Compact Constructor** كويس
7. **اعرف إمتى ماتستخدمش** Records

---

## موارد إضافية 📚

- [JEP 395: Records](https://openjdk.org/jeps/395)
- [JEP 440: Record Patterns](https://openjdk.org/jeps/440)
- [Oracle Java Records Tutorial](https://docs.oracle.com/en/java/javase/17/language/records.html)

---

*تم إعداد هذا الدليل بالعامية المصرية عشان يكون سهل الفهم والمراجعة* 🇪🇬
