# ✍️ Writing Simple Lambdas في Java - دليل شامل بالمصري

## المحتويات
- [مقدمة عن Lambda](#مقدمة-عن-lambda)
- [Lambda Syntax](#lambda-syntax)
- [Functional Interfaces](#functional-interfaces)
- [أنواع Lambda Expressions](#أنواع-lambda-expressions)
- [Variable Capture](#variable-capture)
- [Method References](#method-references)
- [Built-in Functional Interfaces](#built-in-functional-interfaces)
- [Lambda مع Collections](#lambda-مع-collections)
- [أمثلة عملية](#أمثلة-عملية)
- [أسئلة الانترفيو](#أسئلة-الانترفيو)

---

# مقدمة عن Lambda

## يعني إيه Lambda Expression؟

الـ **Lambda Expression** هي طريقة مختصرة عشان تكتب **Anonymous Function** - يعني function من غير اسم!

اتقدمت في **Java 8** وغيرت طريقة كتابة الكود تماماً.

## المشكلة قبل Lambda

```java
// ❌ الطريقة القديمة - Anonymous Inner Class
// عايزين نعمل Runnable بسيط

Runnable oldWay = new Runnable() {
    @Override
    public void run() {
        System.out.println("مرحباً من الـ Thread!");
    }
};

// 6 سطور عشان حاجة بسيطة! 😫
```

## الحل مع Lambda

```java
// ✅ الطريقة الجديدة - Lambda Expression
Runnable newWay = () -> System.out.println("مرحباً من الـ Thread!");

// سطر واحد بس! 🎉
```

## ليه Lambda مهمة؟

| الميزة | الشرح |
|--------|-------|
| **كود أقصر** | بدل 6 سطور تكتب سطر واحد |
| **Readability** | الكود أسهل في القراءة والفهم |
| **Functional Programming** | بتفتح الباب لـ FP في Java |
| **Streams API** | أساس شغل الـ Streams |
| **Parallel Processing** | أسهل في كتابة parallel code |

---

# Lambda Syntax

## الشكل العام

```java
(parameters) -> expression
// أو
(parameters) -> { statements; }
```

## تفصيل الـ Syntax

```
    (parameters)      ->        expression/body
         ↓            ↓              ↓
   Input parameters  Arrow     ده اللي هيتنفذ
                    Operator
```

## قواعد كتابة Lambda

### 1. الأقواس حول الـ Parameters

```java
// ✅ Parameter واحد - الأقواس اختيارية
x -> x * 2
(x) -> x * 2  // نفس الحاجة

// ✅ مفيش parameters - لازم أقواس فاضية
() -> System.out.println("Hello")

// ✅ أكتر من parameter - لازم أقواس
(x, y) -> x + y
(a, b, c) -> a + b + c
```

### 2. نوع الـ Parameters

```java
// ✅ من غير نوع - Type inference
(x, y) -> x + y

// ✅ مع النوع - Explicit types
(int x, int y) -> x + y
(String s, int n) -> s.repeat(n)

// ❌ مش ممكن تخلط!
// (int x, y) -> x + y  // Compile Error!
```

### 3. الـ Body

```java
// ✅ Expression واحد - من غير curly braces و return
x -> x * 2
(a, b) -> a + b
s -> s.toUpperCase()

// ✅ Multiple statements - لازم curly braces
(x, y) -> {
    int sum = x + y;
    return sum * 2;
}

// ✅ مفيش return لو void
s -> {
    System.out.println(s);
    // مفيش return
}

// ⚠️ لو فيه curly braces ومحتاج ترجع قيمة - لازم return
x -> { return x * 2; }  // لازم return هنا
```

## أمثلة على كل الأشكال

```java
// 1. مفيش parameters، expression واحد
() -> 42
() -> "Hello"
() -> Math.random()

// 2. مفيش parameters، multiple statements
() -> {
    System.out.println("Starting...");
    return Math.random();
}

// 3. Parameter واحد، expression واحد
x -> x * 2
s -> s.length()
n -> n > 0

// 4. Parameter واحد، multiple statements
x -> {
    System.out.println("Processing: " + x);
    return x * 2;
}

// 5. Multiple parameters، expression واحد
(x, y) -> x + y
(a, b) -> a.compareTo(b)
(s, n) -> s.substring(0, n)

// 6. Multiple parameters، multiple statements
(x, y) -> {
    int max = x > y ? x : y;
    System.out.println("Max is: " + max);
    return max;
}

// 7. مع explicit types
(int x, int y) -> x + y
(String s) -> s.toUpperCase()
(List<String> list) -> list.size()
```

---

# Functional Interfaces

## يعني إيه Functional Interface؟

الـ **Functional Interface** هو interface فيه **method واحدة abstract بس**.

ده الشرط الأساسي عشان نقدر نستخدم Lambda!

```java
// ✅ Functional Interface - method واحدة abstract
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

// استخدام مع Lambda
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;
Calculator subtract = (a, b) -> a - b;

System.out.println(add.calculate(5, 3));       // 8
System.out.println(multiply.calculate(5, 3)); // 15
System.out.println(subtract.calculate(5, 3)); // 2
```

## @FunctionalInterface Annotation

```java
@FunctionalInterface  // اختياري بس مفيد جداً!
interface MyFunction {
    // ✅ Abstract method واحدة - ده الأساس
    void execute();
    
    // ❌ لو ضفت abstract method تاني - Compile Error!
    // void anotherMethod();
    
    // ✅ Default methods - مش بتتحسب
    default void helper() {
        System.out.println("Helper method");
    }
    
    // ✅ Static methods - مش بتتحسب
    static void utility() {
        System.out.println("Utility method");
    }
    
    // ✅ Methods من Object class - مش بتتحسب
    boolean equals(Object obj);
    String toString();
    int hashCode();
}
```

### ليه نستخدم @FunctionalInterface؟

```java
// من غير الـ annotation
interface Calculator {
    int calculate(int a, int b);
}

// حد تاني ممكن يضيف method تانية بالغلط!
interface Calculator {
    int calculate(int a, int b);
    int anotherMethod();  // مفيش error!
}

// ✅ مع الـ annotation - الـ compiler هيحميك
@FunctionalInterface
interface SafeCalculator {
    int calculate(int a, int b);
    // int anotherMethod();  // ❌ Compile Error!
}
```

## أمثلة على Functional Interfaces

```java
// 1. بدون parameters، بدون return
@FunctionalInterface
interface Task {
    void execute();
}

Task sayHello = () -> System.out.println("مرحباً!");
sayHello.execute();  // مرحباً!

// 2. مع parameter، بدون return
@FunctionalInterface
interface Printer {
    void print(String message);
}

Printer printer = msg -> System.out.println("📝 " + msg);
printer.print("Hello");  // 📝 Hello

// 3. بدون parameters، مع return
@FunctionalInterface
interface NumberGenerator {
    int generate();
}

NumberGenerator random = () -> (int)(Math.random() * 100);
System.out.println(random.generate());  // رقم عشوائي

// 4. مع parameter، مع return
@FunctionalInterface
interface Transformer {
    String transform(String input);
}

Transformer upper = s -> s.toUpperCase();
Transformer reverse = s -> new StringBuilder(s).reverse().toString();

System.out.println(upper.transform("hello"));    // HELLO
System.out.println(reverse.transform("hello"));  // olleh

// 5. Multiple parameters
@FunctionalInterface
interface MathOperation {
    double apply(double x, double y);
}

MathOperation add = (x, y) -> x + y;
MathOperation power = (x, y) -> Math.pow(x, y);

System.out.println(add.apply(5, 3));    // 8.0
System.out.println(power.apply(2, 3));  // 8.0

// 6. Generic Functional Interface
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}

Converter<String, Integer> stringToInt = s -> Integer.parseInt(s);
Converter<Integer, String> intToString = n -> String.valueOf(n);

System.out.println(stringToInt.convert("123"));  // 123
System.out.println(intToString.convert(456));    // "456"
```

---

# أنواع Lambda Expressions

## 1. Lambda بدون Parameters

```java
// Runnable - الأشهر
Runnable task = () -> System.out.println("Task running!");
new Thread(task).start();

// أو مباشرة
new Thread(() -> System.out.println("Direct lambda!")).start();

// Supplier - بترجع قيمة
Supplier<Double> randomSupplier = () -> Math.random();
Supplier<LocalDateTime> nowSupplier = () -> LocalDateTime.now();
Supplier<UUID> uuidSupplier = () -> UUID.randomUUID();

System.out.println(randomSupplier.get());  // 0.7234...
System.out.println(nowSupplier.get());     // 2024-01-15T10:30:00
System.out.println(uuidSupplier.get());    // a1b2c3d4-...

// Callable - زي Supplier بس بيرمي Exception
Callable<String> callable = () -> {
    Thread.sleep(1000);
    return "Done!";
};
```

## 2. Lambda بـ Parameter واحد

```java
// Consumer - بياخد input ومش بيرجع حاجة
Consumer<String> printer = s -> System.out.println(s);
Consumer<Integer> doubler = n -> System.out.println(n * 2);

printer.accept("مرحباً!");  // مرحباً!
doubler.accept(5);          // 10

// Predicate - بياخد input وبيرجع boolean
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<String> isEmpty = s -> s == null || s.isEmpty();
Predicate<String> startsWithA = s -> s.startsWith("A");

System.out.println(isPositive.test(5));   // true
System.out.println(isPositive.test(-3));  // false
System.out.println(isEven.test(4));       // true
System.out.println(isEmpty.test(""));     // true

// Function - بياخد input وبيرجع output (ممكن نوع مختلف)
Function<String, Integer> length = s -> s.length();
Function<Integer, String> toString = n -> "Number: " + n;
Function<String, String> toUpper = s -> s.toUpperCase();

System.out.println(length.apply("Hello"));    // 5
System.out.println(toString.apply(42));       // "Number: 42"
System.out.println(toUpper.apply("hello"));   // "HELLO"

// UnaryOperator - Function بس نفس النوع input و output
UnaryOperator<Integer> square = n -> n * n;
UnaryOperator<String> addPrefix = s -> "Mr. " + s;
UnaryOperator<Integer> increment = n -> n + 1;

System.out.println(square.apply(5));        // 25
System.out.println(addPrefix.apply("أحمد")); // Mr. أحمد
System.out.println(increment.apply(10));    // 11
```

## 3. Lambda بـ Multiple Parameters

```java
// BiFunction - 2 inputs، output واحد
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);
BiFunction<String, String, String> concat = (a, b) -> a + " " + b;

System.out.println(add.apply(5, 3));           // 8
System.out.println(repeat.apply("Ha", 3));     // HaHaHa
System.out.println(concat.apply("Hello", "World")); // Hello World

// BinaryOperator - BiFunction بس نفس النوع للكل
BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
BinaryOperator<Integer> min = (a, b) -> a < b ? a : b;
BinaryOperator<String> longer = (a, b) -> a.length() > b.length() ? a : b;

System.out.println(max.apply(5, 3));  // 5
System.out.println(min.apply(5, 3));  // 3
System.out.println(longer.apply("Hello", "Hi"));  // Hello

// BiConsumer - 2 inputs، مفيش output
BiConsumer<String, Integer> printWithAge = 
    (name, age) -> System.out.println(name + " عمره " + age + " سنة");
BiConsumer<String, String> printKeyValue = 
    (k, v) -> System.out.println(k + " = " + v);

printWithAge.accept("أحمد", 25);     // أحمد عمره 25 سنة
printKeyValue.accept("name", "Ali"); // name = Ali

// BiPredicate - 2 inputs، boolean output
BiPredicate<String, Integer> hasLength = (s, len) -> s.length() == len;
BiPredicate<Integer, Integer> isGreater = (a, b) -> a > b;

System.out.println(hasLength.test("Hello", 5));  // true
System.out.println(isGreater.test(10, 5));       // true

// Comparator - للـ sorting
Comparator<String> byLength = (s1, s2) -> s1.length() - s2.length();
Comparator<Integer> descending = (a, b) -> b - a;
Comparator<Person> byAge = (p1, p2) -> p1.getAge() - p2.getAge();

List<String> words = Arrays.asList("banana", "apple", "kiwi");
words.sort(byLength);
System.out.println(words);  // [kiwi, apple, banana]
```

## 4. Lambda مع Blocks

```java
// Single expression - مفيش curly braces
Function<Integer, Integer> simple = n -> n * 2;

// Multiple statements - لازم curly braces و return
Function<Integer, Integer> complex = n -> {
    System.out.println("Processing: " + n);
    int result = n * 2;
    System.out.println("Result: " + result);
    return result;
};

// Void lambda مع block
Consumer<String> processor = s -> {
    String upper = s.toUpperCase();
    String trimmed = upper.trim();
    System.out.println("Processed: " + trimmed);
};

// Complex logic
BiFunction<List<Integer>, Integer, List<Integer>> filterGreaterThan = 
    (list, threshold) -> {
        List<Integer> result = new ArrayList<>();
        for (Integer n : list) {
            if (n > threshold) {
                result.add(n);
            }
        }
        return result;
    };

List<Integer> numbers = Arrays.asList(1, 5, 3, 8, 2, 9);
List<Integer> filtered = filterGreaterThan.apply(numbers, 4);
System.out.println(filtered);  // [5, 8, 9]
```

---

# Variable Capture

## يعني إيه Variable Capture؟

الـ Lambda ممكن تستخدم variables من الـ scope اللي حواليها - ده اسمه **Variable Capture** أو **Closure**.

## القواعد المهمة

```java
public class VariableCaptureExample {
    
    // Instance variable
    private int instanceVar = 10;
    
    // Static variable
    private static int staticVar = 20;
    
    public void demonstrate() {
        // Local variable - لازم يكون effectively final
        int localVar = 30;
        final int finalVar = 40;
        
        // Lambda can capture:
        Consumer<Integer> lambda = (x) -> {
            // ✅ Instance variable - عادي
            System.out.println("Instance: " + instanceVar);
            instanceVar = 100;  // ✅ ممكن تغيره!
            
            // ✅ Static variable - عادي
            System.out.println("Static: " + staticVar);
            staticVar = 200;  // ✅ ممكن تغيره!
            
            // ✅ Local variable (effectively final)
            System.out.println("Local: " + localVar);
            // localVar = 50;  // ❌ Compile Error!
            
            // ✅ Final local variable
            System.out.println("Final: " + finalVar);
            // finalVar = 50;  // ❌ طبعاً مش ممكن
            
            // ✅ Parameter
            System.out.println("Param: " + x);
        };
        
        lambda.accept(50);
        
        // ❌ ده هيخلي localVar مش effectively final
        // localVar = 60;  // لو عملت كده، الـ lambda فوق هتعمل compile error
    }
}
```

## يعني إيه Effectively Final؟

الـ variable يكون **effectively final** لما مش بيتغير بعد ما يتعرف، حتى لو مش مكتوب `final`.

```java
public void effectivelyFinalExample() {
    // ✅ Effectively final - مش بيتغير
    int count = 10;
    
    Runnable r = () -> System.out.println(count);  // ✅ شغال
    
    // count = 20;  // ❌ لو عملت كده، الـ lambda فوق هتفشل
}

public void notEffectivelyFinal() {
    // ❌ مش effectively final - بيتغير
    int count = 10;
    count = 20;  // بيتغير!
    
    int finalCount = count;  // workaround
    
    // Runnable r = () -> System.out.println(count);  // ❌ Compile Error
    Runnable r = () -> System.out.println(finalCount);  // ✅ شغال
}
```

## ليه لازم Effectively Final؟

```java
public void whyEffectivelyFinal() {
    int value = 10;
    
    // الـ Lambda ممكن تتنفذ في thread تاني وفي وقت تاني
    Runnable r = () -> System.out.println(value);
    
    // لو سمحنا بتغيير value:
    // value = 20;  // ده ممكن يحصل قبل الـ lambda تتنفذ
    
    new Thread(r).start();  // الـ lambda هتتنفذ في thread تاني
    
    // إيه القيمة اللي الـ lambda هتشوفها؟ 10 ولا 20؟
    // عشان نتجنب الـ confusion ده، Java بتفرض effectively final
}
```

## Workarounds للـ Effectively Final

```java
// ❌ ده مش هيشتغل
public void problem() {
    int counter = 0;
    
    List<String> names = Arrays.asList("أحمد", "محمد", "علي");
    names.forEach(name -> {
        // counter++;  // ❌ Compile Error - counter مش effectively final
        System.out.println(name);
    });
}

// ✅ Solution 1: Array wrapper
public void solution1() {
    int[] counter = {0};  // Array هو reference، والـ reference ثابت
    
    List<String> names = Arrays.asList("أحمد", "محمد", "علي");
    names.forEach(name -> {
        counter[0]++;  // ✅ شغال! بنغير content مش reference
        System.out.println(counter[0] + ". " + name);
    });
    
    System.out.println("Total: " + counter[0]);  // 3
}

// ✅ Solution 2: AtomicInteger (Thread-safe)
public void solution2() {
    AtomicInteger counter = new AtomicInteger(0);
    
    List<String> names = Arrays.asList("أحمد", "محمد", "علي");
    names.forEach(name -> {
        int index = counter.incrementAndGet();
        System.out.println(index + ". " + name);
    });
    
    System.out.println("Total: " + counter.get());  // 3
}

// ✅ Solution 3: Use instance variable
public class Counter {
    private int count = 0;
    
    public void process() {
        List<String> names = Arrays.asList("أحمد", "محمد", "علي");
        names.forEach(name -> {
            count++;  // ✅ instance variable - مفيش مشكلة
            System.out.println(count + ". " + name);
        });
    }
}
```

## الفرق بين Lambda و Anonymous Class في `this`

```java
public class ThisExample {
    private String name = "Outer";
    
    public void testThis() {
        // Anonymous Inner Class
        // this = الـ Anonymous class نفسه
        Runnable anon = new Runnable() {
            private String name = "Anonymous";
            
            @Override
            public void run() {
                System.out.println(this.name);  // "Anonymous"
                System.out.println(ThisExample.this.name);  // "Outer"
            }
        };
        
        // Lambda Expression
        // this = الـ Enclosing class (ThisExample)
        Runnable lambda = () -> {
            System.out.println(this.name);  // "Outer"
            // مفيش حاجة اسمها Lambda.this
        };
        
        anon.run();    // Anonymous
        lambda.run();  // Outer
    }
}
```

---

# Method References

## يعني إيه Method Reference؟

الـ **Method Reference** هي طريقة أقصر لكتابة Lambda لما بتـ call method موجود.

```java
// Lambda
Function<String, Integer> lambda = s -> s.length();

// Method Reference - أقصر وأوضح
Function<String, Integer> methodRef = String::length;
```

## أنواع Method References

### 1. Static Method Reference

```java
// Syntax: ClassName::staticMethodName

// Lambda
Function<String, Integer> parse1 = s -> Integer.parseInt(s);
// Method Reference
Function<String, Integer> parse2 = Integer::parseInt;

// Lambda
BiFunction<Integer, Integer, Integer> max1 = (a, b) -> Math.max(a, b);
// Method Reference
BiFunction<Integer, Integer, Integer> max2 = Math::max;

// Lambda
Supplier<Double> random1 = () -> Math.random();
// Method Reference
Supplier<Double> random2 = Math::random;

// أمثلة عملية
List<String> numbers = Arrays.asList("1", "2", "3");
List<Integer> parsed = numbers.stream()
    .map(Integer::parseInt)  // بدل s -> Integer.parseInt(s)
    .collect(Collectors.toList());
```

### 2. Instance Method Reference (Bound)

```java
// Syntax: instance::methodName
// الـ instance معروف ومحدد

String prefix = "Hello, ";

// Lambda
Function<String, String> greet1 = s -> prefix.concat(s);
// Method Reference
Function<String, String> greet2 = prefix::concat;

System.out.println(greet2.apply("World"));  // Hello, World

// مثال تاني
PrintStream out = System.out;

// Lambda
Consumer<String> print1 = s -> out.println(s);
// Method Reference
Consumer<String> print2 = out::println;

// الأشهر
Consumer<String> print3 = System.out::println;

List<String> names = Arrays.asList("أحمد", "محمد", "علي");
names.forEach(System.out::println);  // بدل name -> System.out.println(name)
```

### 3. Instance Method Reference (Unbound)

```java
// Syntax: ClassName::instanceMethodName
// الـ instance هو أول parameter في الـ Lambda

// Lambda
Function<String, String> upper1 = s -> s.toUpperCase();
// Method Reference
Function<String, String> upper2 = String::toUpperCase;

// Lambda
Function<String, Integer> length1 = s -> s.length();
// Method Reference
Function<String, Integer> length2 = String::length;

// Lambda
Predicate<String> empty1 = s -> s.isEmpty();
// Method Reference
Predicate<String> empty2 = String::isEmpty;

// مع 2 parameters
// Lambda
BiPredicate<String, String> equals1 = (s1, s2) -> s1.equals(s2);
// Method Reference
BiPredicate<String, String> equals2 = String::equals;

// Lambda
Comparator<String> comp1 = (s1, s2) -> s1.compareTo(s2);
// Method Reference
Comparator<String> comp2 = String::compareTo;

// أمثلة عملية
List<String> names = Arrays.asList("ahmed", "mohamed", "ali");

// باستخدام Lambda
List<String> upper3 = names.stream()
    .map(s -> s.toUpperCase())
    .collect(Collectors.toList());

// باستخدام Method Reference
List<String> upper4 = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 4. Constructor Reference

```java
// Syntax: ClassName::new

// Lambda
Supplier<ArrayList<String>> list1 = () -> new ArrayList<>();
// Constructor Reference
Supplier<ArrayList<String>> list2 = ArrayList::new;

// Lambda
Function<String, Integer> int1 = s -> new Integer(s);
// Constructor Reference
Function<String, Integer> int2 = Integer::new;

// مع custom class
class Person {
    private String name;
    
    public Person() { }
    public Person(String name) { this.name = name; }
    
    public String getName() { return name; }
}

// No-arg constructor
Supplier<Person> personSupplier = Person::new;
Person p1 = personSupplier.get();

// One-arg constructor
Function<String, Person> personFactory = Person::new;
Person p2 = personFactory.apply("أحمد");

// أمثلة عملية
List<String> names = Arrays.asList("أحمد", "محمد", "علي");

// Lambda
List<Person> people1 = names.stream()
    .map(name -> new Person(name))
    .collect(Collectors.toList());

// Constructor Reference
List<Person> people2 = names.stream()
    .map(Person::new)
    .collect(Collectors.toList());

// Array Constructor Reference
// Lambda
String[] arr1 = names.stream().toArray(size -> new String[size]);
// Constructor Reference
String[] arr2 = names.stream().toArray(String[]::new);
```

## جدول مقارنة

| النوع | Lambda | Method Reference |
|-------|--------|------------------|
| Static | `s -> Integer.parseInt(s)` | `Integer::parseInt` |
| Bound Instance | `s -> str.concat(s)` | `str::concat` |
| Unbound Instance | `s -> s.toUpperCase()` | `String::toUpperCase` |
| Constructor | `() -> new ArrayList<>()` | `ArrayList::new` |

## إمتى تستخدم Method Reference؟

```java
// ✅ استخدم Method Reference لما:
// 1. الـ Lambda بتعمل call لـ method واحد بس
names.forEach(System.out::println);  // ✅
names.stream().map(String::length);  // ✅

// ❌ ما تستخدمش Method Reference لما:
// 1. محتاج تعمل حاجة إضافية
names.forEach(name -> System.out.println("Name: " + name));  // ❌ مش ممكن method reference

// 2. الـ arguments مش بنفس الترتيب
BiFunction<String, String, Boolean> check = (a, b) -> b.startsWith(a);
// مش ممكن method reference لأن الترتيب مختلف

// 3. محتاج تعمل transformation على الـ argument
Function<String, Integer> parse = s -> Integer.parseInt(s.trim());
// مش ممكن method reference لأن فيه .trim()
```

---

# Built-in Functional Interfaces

Java وفرت مجموعة جاهزة من الـ Functional Interfaces في `java.util.function`.

## الجدول الشامل

| Interface | Input | Output | Method | الاستخدام |
|-----------|-------|--------|--------|-----------|
| `Predicate<T>` | T | boolean | `test()` | Filtering، Validation |
| `Function<T,R>` | T | R | `apply()` | Transformation |
| `Consumer<T>` | T | void | `accept()` | Side effects، Printing |
| `Supplier<T>` | - | T | `get()` | Generation، Factory |
| `BiPredicate<T,U>` | T, U | boolean | `test()` | 2-input testing |
| `BiFunction<T,U,R>` | T, U | R | `apply()` | 2-input transform |
| `BiConsumer<T,U>` | T, U | void | `accept()` | 2-input action |
| `UnaryOperator<T>` | T | T | `apply()` | Same type transform |
| `BinaryOperator<T>` | T, T | T | `apply()` | Combine same type |

## 1. Predicate<T>

```java
import java.util.function.Predicate;

// Basic Predicates
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<String> isEmpty = s -> s == null || s.isEmpty();
Predicate<String> longerThan5 = s -> s.length() > 5;

// استخدام
System.out.println(isPositive.test(5));   // true
System.out.println(isEven.test(4));       // true
System.out.println(isEmpty.test(""));     // true
System.out.println(longerThan5.test("Hello World"));  // true

// Combining Predicates
Predicate<Integer> isPositiveAndEven = isPositive.and(isEven);
Predicate<Integer> isPositiveOrEven = isPositive.or(isEven);
Predicate<Integer> isNotPositive = isPositive.negate();

System.out.println(isPositiveAndEven.test(4));   // true (positive AND even)
System.out.println(isPositiveAndEven.test(3));   // false (positive but odd)
System.out.println(isPositiveAndEven.test(-4));  // false (even but negative)
System.out.println(isPositiveOrEven.test(-4));   // true (even)
System.out.println(isNotPositive.test(-5));      // true

// Predicate.isEqual()
Predicate<String> isHello = Predicate.isEqual("Hello");
System.out.println(isHello.test("Hello"));  // true
System.out.println(isHello.test("World"));  // false

// مع Streams
List<Integer> numbers = Arrays.asList(1, -2, 3, -4, 5, 6, -7, 8);

List<Integer> positiveNumbers = numbers.stream()
    .filter(isPositive)
    .collect(Collectors.toList());
System.out.println(positiveNumbers);  // [1, 3, 5, 6, 8]

List<Integer> evenNumbers = numbers.stream()
    .filter(isEven)
    .collect(Collectors.toList());
System.out.println(evenNumbers);  // [-2, -4, 6, 8]

List<Integer> positiveEvenNumbers = numbers.stream()
    .filter(isPositive.and(isEven))
    .collect(Collectors.toList());
System.out.println(positiveEvenNumbers);  // [6, 8]
```

## 2. Function<T, R>

```java
import java.util.function.Function;

// Basic Functions
Function<String, Integer> stringLength = s -> s.length();
Function<Integer, Integer> square = n -> n * n;
Function<Integer, String> intToString = n -> "Number: " + n;
Function<String, String> toUpperCase = s -> s.toUpperCase();

// استخدام
System.out.println(stringLength.apply("Hello"));  // 5
System.out.println(square.apply(4));              // 16
System.out.println(intToString.apply(42));        // "Number: 42"
System.out.println(toUpperCase.apply("hello"));   // "HELLO"

// Chaining Functions - andThen
Function<Integer, Integer> addOne = n -> n + 1;
Function<Integer, Integer> addOneThenSquare = addOne.andThen(square);
// أول addOne بعدين square
// 3 -> 4 -> 16
System.out.println(addOneThenSquare.apply(3));  // 16

// Chaining Functions - compose
Function<Integer, Integer> squareThenAddOne = addOne.compose(square);
// أول square بعدين addOne
// 3 -> 9 -> 10
System.out.println(squareThenAddOne.apply(3));  // 10

// Function.identity()
Function<String, String> identity = Function.identity();
System.out.println(identity.apply("Hello"));  // "Hello"

// مع Streams
List<String> names = Arrays.asList("ahmed", "mohamed", "ali");

List<Integer> lengths = names.stream()
    .map(stringLength)
    .collect(Collectors.toList());
System.out.println(lengths);  // [5, 7, 3]

List<String> upperNames = names.stream()
    .map(toUpperCase)
    .collect(Collectors.toList());
System.out.println(upperNames);  // [AHMED, MOHAMED, ALI]

// Complex transformation
Function<String, String> processName = 
    ((Function<String, String>) String::trim)
    .andThen(String::toLowerCase)
    .andThen(s -> s.substring(0, 1).toUpperCase() + s.substring(1));

System.out.println(processName.apply("  AHMED  "));  // "Ahmed"
```

## 3. Consumer<T>

```java
import java.util.function.Consumer;

// Basic Consumers
Consumer<String> printer = s -> System.out.println(s);
Consumer<String> logger = s -> System.out.println("[LOG] " + s);
Consumer<List<Integer>> clearList = list -> list.clear();

// استخدام
printer.accept("مرحباً!");  // مرحباً!
logger.accept("Started");   // [LOG] Started

// Chaining Consumers - andThen
Consumer<String> printAndLog = printer.andThen(logger);
printAndLog.accept("Test");
// Test
// [LOG] Test

// مع Streams - forEach
List<String> names = Arrays.asList("أحمد", "محمد", "علي");

names.forEach(printer);
// أحمد
// محمد
// علي

names.forEach(name -> System.out.println("مرحباً " + name));
// مرحباً أحمد
// مرحباً محمد
// مرحباً علي

// Consumer for modification
Consumer<List<String>> addGreeting = list -> list.add("مرحباً");
Consumer<List<String>> addFarewell = list -> list.add("مع السلامة");

List<String> messages = new ArrayList<>();
addGreeting.andThen(addFarewell).accept(messages);
System.out.println(messages);  // [مرحباً, مع السلامة]
```

## 4. Supplier<T>

```java
import java.util.function.Supplier;

// Basic Suppliers
Supplier<Double> randomSupplier = () -> Math.random();
Supplier<String> uuidSupplier = () -> UUID.randomUUID().toString();
Supplier<LocalDateTime> nowSupplier = () -> LocalDateTime.now();
Supplier<List<String>> listFactory = () -> new ArrayList<>();

// استخدام
System.out.println(randomSupplier.get());  // 0.7234...
System.out.println(uuidSupplier.get());    // "a1b2c3d4-..."
System.out.println(nowSupplier.get());     // 2024-01-15T10:30:00

List<String> newList = listFactory.get();
newList.add("item");

// Lazy Evaluation - الـ Supplier مش بيتنفذ إلا لما نحتاجه
Supplier<ExpensiveObject> lazyObject = () -> {
    System.out.println("Creating expensive object...");
    return new ExpensiveObject();
};

System.out.println("Before get()");
// مفيش حاجة اتنفذت لسه
ExpensiveObject obj = lazyObject.get();  // هنا بس بيتعمل
System.out.println("After get()");

// مع Optional
Optional<String> optional = Optional.empty();

// orElse - دايماً بيتنفذ
String result1 = optional.orElse(getDefaultValue());  // getDefaultValue() هتتنفذ

// orElseGet - بيتنفذ لو فاضي بس
String result2 = optional.orElseGet(() -> getDefaultValue());  // Lazy!

// orElseThrow
String result3 = optional.orElseThrow(() -> 
    new IllegalArgumentException("Value not present"));
```

## 5. BiFunction و BiConsumer و BiPredicate

```java
// BiFunction<T, U, R>
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);

System.out.println(add.apply(5, 3));        // 8
System.out.println(repeat.apply("Ha", 3));  // HaHaHa

// BiConsumer<T, U>
BiConsumer<String, Integer> printPerson = 
    (name, age) -> System.out.println(name + ": " + age);

printPerson.accept("أحمد", 25);  // أحمد: 25

// مع Map.forEach
Map<String, Integer> ages = new HashMap<>();
ages.put("أحمد", 25);
ages.put("محمد", 30);

ages.forEach((name, age) -> System.out.println(name + " عمره " + age));

// BiPredicate<T, U>
BiPredicate<String, Integer> hasLength = (s, len) -> s.length() == len;
BiPredicate<Integer, Integer> isDivisible = (a, b) -> a % b == 0;

System.out.println(hasLength.test("Hello", 5));  // true
System.out.println(isDivisible.test(10, 2));     // true
```

## 6. UnaryOperator و BinaryOperator

```java
// UnaryOperator<T> = Function<T, T>
UnaryOperator<Integer> square = n -> n * n;
UnaryOperator<String> toUpper = s -> s.toUpperCase();
UnaryOperator<String> addBrackets = s -> "[" + s + "]";

System.out.println(square.apply(5));          // 25
System.out.println(toUpper.apply("hello"));   // HELLO
System.out.println(addBrackets.apply("test")); // [test]

// مع List.replaceAll
List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
numbers.replaceAll(square);
System.out.println(numbers);  // [1, 4, 9, 16, 25]

// BinaryOperator<T> = BiFunction<T, T, T>
BinaryOperator<Integer> add = (a, b) -> a + b;
BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
BinaryOperator<String> concat = (a, b) -> a + b;

System.out.println(add.apply(5, 3));       // 8
System.out.println(max.apply(5, 3));       // 5
System.out.println(concat.apply("a", "b")); // ab

// مع Stream.reduce
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);
int sum = nums.stream().reduce(0, add);
System.out.println(sum);  // 15

// BinaryOperator static methods
BinaryOperator<Integer> minOp = BinaryOperator.minBy(Integer::compareTo);
BinaryOperator<Integer> maxOp = BinaryOperator.maxBy(Integer::compareTo);

System.out.println(minOp.apply(5, 3));  // 3
System.out.println(maxOp.apply(5, 3));  // 5
```

## Primitive Specializations

عشان نتجنب Boxing/Unboxing:

```java
// IntPredicate, LongPredicate, DoublePredicate
IntPredicate isEven = n -> n % 2 == 0;
isEven.test(4);  // true - مفيش boxing!

// IntFunction<R>, LongFunction<R>, DoubleFunction<R>
IntFunction<String> intToString = n -> "Value: " + n;
intToString.apply(42);  // "Value: 42"

// ToIntFunction<T>, ToLongFunction<T>, ToDoubleFunction<T>
ToIntFunction<String> stringLength = s -> s.length();
stringLength.applyAsInt("Hello");  // 5

// IntConsumer, LongConsumer, DoubleConsumer
IntConsumer printInt = n -> System.out.println(n);
printInt.accept(42);

// IntSupplier, LongSupplier, DoubleSupplier
IntSupplier randomInt = () -> (int)(Math.random() * 100);
randomInt.getAsInt();

// IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator
IntUnaryOperator square = n -> n * n;
square.applyAsInt(5);  // 25

// IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator
IntBinaryOperator add = (a, b) -> a + b;
add.applyAsInt(5, 3);  // 8
```

---

# Lambda مع Collections

## forEach

```java
List<String> names = Arrays.asList("أحمد", "محمد", "علي");

// Lambda
names.forEach(name -> System.out.println(name));

// Method Reference
names.forEach(System.out::println);

// مع Map
Map<String, Integer> ages = Map.of("أحمد", 25, "محمد", 30);
ages.forEach((name, age) -> System.out.println(name + ": " + age));
```

## removeIf

```java
List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6));

// إزالة الأرقام الزوجية
numbers.removeIf(n -> n % 2 == 0);
System.out.println(numbers);  // [1, 3, 5]

// مع Predicate
Predicate<Integer> isNegative = n -> n < 0;
List<Integer> mixed = new ArrayList<>(Arrays.asList(-2, 3, -5, 7, -1));
mixed.removeIf(isNegative);
System.out.println(mixed);  // [3, 7]
```

## replaceAll

```java
List<String> names = new ArrayList<>(Arrays.asList("ahmed", "mohamed", "ali"));

// تحويل لـ uppercase
names.replaceAll(s -> s.toUpperCase());
System.out.println(names);  // [AHMED, MOHAMED, ALI]

// Method Reference
names.replaceAll(String::toLowerCase);
System.out.println(names);  // [ahmed, mohamed, ali]
```

## sort with Comparator

```java
List<String> names = new ArrayList<>(Arrays.asList("محمد", "أحمد", "علي", "عمر"));

// Lambda Comparator
names.sort((a, b) -> a.length() - b.length());
System.out.println(names);  // ترتيب بالطول

// Comparator.comparing
names.sort(Comparator.comparing(String::length));
names.sort(Comparator.comparing(s -> s.length()));

// Reversed
names.sort(Comparator.comparing(String::length).reversed());

// Multiple criteria
List<Person> people = new ArrayList<>();
people.add(new Person("أحمد", 25));
people.add(new Person("محمد", 30));
people.add(new Person("علي", 25));

people.sort(Comparator
    .comparing(Person::getAge)
    .thenComparing(Person::getName));

// Null-safe
people.sort(Comparator.comparing(
    Person::getName,
    Comparator.nullsLast(String::compareTo)
));
```

## Map operations

```java
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("b", 2);

// computeIfAbsent - لو مش موجود، احسبه وضيفه
map.computeIfAbsent("c", key -> key.length());
// c -> 1 (length of "c")

// computeIfPresent - لو موجود، احسبه من جديد
map.computeIfPresent("a", (key, value) -> value * 10);
// a -> 10

// compute - دايماً احسبه
map.compute("d", (key, value) -> value == null ? 1 : value + 1);

// merge
map.merge("a", 5, (oldVal, newVal) -> oldVal + newVal);
// a = 10 + 5 = 15

// replaceAll
map.replaceAll((key, value) -> value * 2);

// getOrDefault
int value = map.getOrDefault("z", 0);
```

---

# أمثلة عملية

## 1. Validation Framework

```java
@FunctionalInterface
interface Validator<T> {
    ValidationResult validate(T value);
    
    default Validator<T> and(Validator<T> other) {
        return value -> {
            ValidationResult result = this.validate(value);
            return result.isValid() ? other.validate(value) : result;
        };
    }
}

class ValidationResult {
    private final boolean valid;
    private final String message;
    
    private ValidationResult(boolean valid, String message) {
        this.valid = valid;
        this.message = message;
    }
    
    public static ValidationResult ok() {
        return new ValidationResult(true, "");
    }
    
    public static ValidationResult fail(String message) {
        return new ValidationResult(false, message);
    }
    
    public boolean isValid() { return valid; }
    public String getMessage() { return message; }
}

// استخدام
Validator<String> notNull = s -> 
    s != null ? ValidationResult.ok() : ValidationResult.fail("القيمة null");

Validator<String> notEmpty = s -> 
    !s.isEmpty() ? ValidationResult.ok() : ValidationResult.fail("القيمة فاضية");

Validator<String> maxLength = s -> 
    s.length() <= 100 ? ValidationResult.ok() : ValidationResult.fail("طويلة جداً");

Validator<String> emailValidator = notNull.and(notEmpty).and(maxLength);

ValidationResult result = emailValidator.validate("test@example.com");
System.out.println(result.isValid());  // true
```

## 2. Event System

```java
class EventEmitter<T> {
    private final List<Consumer<T>> listeners = new ArrayList<>();
    
    public void on(Consumer<T> listener) {
        listeners.add(listener);
    }
    
    public void emit(T event) {
        listeners.forEach(listener -> listener.accept(event));
    }
    
    public void off(Consumer<T> listener) {
        listeners.remove(listener);
    }
}

// استخدام
EventEmitter<String> emitter = new EventEmitter<>();

emitter.on(msg -> System.out.println("Listener 1: " + msg));
emitter.on(msg -> System.out.println("Listener 2: " + msg));
emitter.on(System.out::println);

emitter.emit("مرحباً!");
// Listener 1: مرحباً!
// Listener 2: مرحباً!
// مرحباً!
```

## 3. Retry Logic

```java
public static <T> T retry(Supplier<T> operation, int maxRetries, 
                          Predicate<Exception> retryOn) {
    int attempts = 0;
    while (true) {
        try {
            return operation.get();
        } catch (Exception e) {
            attempts++;
            if (attempts >= maxRetries || !retryOn.test(e)) {
                throw new RuntimeException("Failed after " + attempts + " attempts", e);
            }
            System.out.println("Retry " + attempts + "...");
        }
    }
}

// استخدام
String result = retry(
    () -> fetchDataFromApi(),  // Supplier
    3,                          // max retries
    e -> e instanceof IOException  // retry on IOException
);
```

## 4. Pipeline Pattern

```java
class Pipeline<T> {
    private final List<Function<T, T>> steps = new ArrayList<>();
    
    public Pipeline<T> addStep(Function<T, T> step) {
        steps.add(step);
        return this;
    }
    
    public T execute(T input) {
        T result = input;
        for (Function<T, T> step : steps) {
            result = step.apply(result);
        }
        return result;
    }
}

// استخدام
Pipeline<String> textPipeline = new Pipeline<String>()
    .addStep(String::trim)
    .addStep(String::toLowerCase)
    .addStep(s -> s.replaceAll("\\s+", " "))
    .addStep(s -> s.substring(0, 1).toUpperCase() + s.substring(1));

String result = textPipeline.execute("  HELLO   WORLD  ");
System.out.println(result);  // "Hello world"
```

## 5. Lazy Evaluation

```java
class Lazy<T> {
    private final Supplier<T> supplier;
    private T value;
    private boolean computed = false;
    
    private Lazy(Supplier<T> supplier) {
        this.supplier = supplier;
    }
    
    public static <T> Lazy<T> of(Supplier<T> supplier) {
        return new Lazy<>(supplier);
    }
    
    public T get() {
        if (!computed) {
            value = supplier.get();
            computed = true;
        }
        return value;
    }
    
    public <R> Lazy<R> map(Function<T, R> mapper) {
        return Lazy.of(() -> mapper.apply(get()));
    }
}

// استخدام
Lazy<Double> lazyValue = Lazy.of(() -> {
    System.out.println("Computing...");
    return Math.random();
});

System.out.println("Before get");
System.out.println(lazyValue.get());  // Computing... + value
System.out.println(lazyValue.get());  // same value, no "Computing..."
```

---

# أسئلة الانترفيو

## أسئلة نظرية

### س1: إيه هو الـ Lambda Expression؟

**الإجابة:**
Lambda Expression هي طريقة مختصرة لكتابة Anonymous Function في Java. اتقدمت في Java 8 وبتسمح لنا نكتب code أقصر وأوضح، خصوصاً لما بنتعامل مع Functional Interfaces.

```java
// بدل كده
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

// نكتب كده
Runnable r = () -> System.out.println("Hello");
```

---

### س2: إيه هو الـ Functional Interface؟

**الإجابة:**
Functional Interface هو interface فيه **abstract method واحدة بس** (Single Abstract Method - SAM). ده الشرط الأساسي عشان نستخدم Lambda.

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);  // SAM
    
    // الحاجات دي مش بتتحسب:
    default void helper() { }     // default method
    static void util() { }        // static method
    boolean equals(Object o);     // من Object
}
```

---

### س3: إيه الفرق بين Lambda و Anonymous Inner Class؟

**الإجابة:**

| Lambda | Anonymous Inner Class |
|--------|----------------------|
| Functional Interface بس | أي interface أو class |
| `this` = enclosing class | `this` = anonymous class |
| مفيش state (fields) | ممكن يكون فيه fields |
| أقصر وأبسط | أطول |
| No .class file | بيتعمله .class file |
| Lazy instantiation | Eager instantiation |

```java
public class Example {
    String name = "Outer";
    
    void test() {
        // Anonymous - this = anonymous
        Runnable anon = new Runnable() {
            String name = "Anon";
            public void run() {
                System.out.println(this.name);  // "Anon"
            }
        };
        
        // Lambda - this = Example (enclosing)
        Runnable lambda = () -> {
            System.out.println(this.name);  // "Outer"
        };
    }
}
```

---

### س4: يعني إيه Effectively Final؟

**الإجابة:**
Effectively Final هو variable مش معرف كـ `final` بس مش بيتغير بعد ما يتعرف. الـ Lambda لازم توصل لـ effectively final variables بس من الـ local scope.

```java
void example() {
    int x = 10;  // effectively final
    int y = 20;
    y = 30;      // مش effectively final!
    
    Runnable r = () -> {
        System.out.println(x);  // ✅ OK
        // System.out.println(y);  // ❌ Error
    };
}
```

**السبب:** الـ Lambda ممكن تتنفذ في وقت تاني أو thread تاني، فلازم القيمة تكون ثابتة.

---

### س5: إيه أنواع Method References؟

**الإجابة:**

| النوع | Syntax | مثال |
|-------|--------|------|
| Static | `Class::staticMethod` | `Integer::parseInt` |
| Bound Instance | `instance::method` | `str::length` |
| Unbound Instance | `Class::instanceMethod` | `String::toUpperCase` |
| Constructor | `Class::new` | `ArrayList::new` |

```java
// Static
Function<String, Integer> f1 = Integer::parseInt;

// Bound Instance
String str = "Hello";
Supplier<Integer> f2 = str::length;

// Unbound Instance
Function<String, String> f3 = String::toUpperCase;

// Constructor
Supplier<List<String>> f4 = ArrayList::new;
```

---

### س6: إيه الفرق بين Predicate و Function؟

**الإجابة:**

| Predicate<T> | Function<T, R> |
|--------------|----------------|
| بيرجع `boolean` دايماً | بيرجع أي نوع R |
| للـ testing/filtering | للـ transformation |
| method: `test()` | method: `apply()` |
| عنده `and()`, `or()`, `negate()` | عنده `andThen()`, `compose()` |

```java
Predicate<Integer> isPositive = n -> n > 0;
boolean result = isPositive.test(5);  // true

Function<Integer, String> toString = n -> "Number: " + n;
String result2 = toString.apply(5);  // "Number: 5"
```

---

### س7: إيه الفرق بين `map()` و `flatMap()` في Optional؟

**الإجابة:**

```java
Optional<String> opt = Optional.of("Hello");

// map - لو الـ function بترجع قيمة عادية
Optional<Integer> length = opt.map(s -> s.length());
// Optional<Integer>

// flatMap - لو الـ function بترجع Optional
Optional<String> upper = opt.flatMap(s -> Optional.of(s.toUpperCase()));
// Optional<String>

// المشكلة لو استخدمت map مع function بترجع Optional
Optional<Optional<String>> nested = opt.map(s -> Optional.of(s.toUpperCase()));
// Optional<Optional<String>> - nested!

// flatMap بيعمل flatten
Optional<String> flat = opt.flatMap(s -> Optional.of(s.toUpperCase()));
// Optional<String> - flat!
```

---

### س8: إيه الفرق بين `forEach` و `forEachOrdered` في Streams؟

**الإجابة:**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// forEach - مش guaranteed order في parallel streams
numbers.parallelStream().forEach(System.out::println);
// ممكن: 3, 1, 4, 2, 5

// forEachOrdered - guaranteed order حتى في parallel
numbers.parallelStream().forEachOrdered(System.out::println);
// دايماً: 1, 2, 3, 4, 5
```

---

### س9: إزاي تعمل Functional Interface مع Checked Exception؟

**الإجابة:**

```java
// المشكلة: built-in interfaces مش بتدعم checked exceptions
// list.forEach(item -> Files.delete(path));  // ❌ Error

// الحل 1: Custom Functional Interface
@FunctionalInterface
interface ThrowingConsumer<T, E extends Exception> {
    void accept(T t) throws E;
    
    static <T, E extends Exception> Consumer<T> unchecked(
            ThrowingConsumer<T, E> consumer) {
        return t -> {
            try {
                consumer.accept(t);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        };
    }
}

// الاستخدام
list.forEach(ThrowingConsumer.unchecked(item -> Files.delete(path)));

// الحل 2: Wrapper method
private void safeDelete(Path path) {
    try {
        Files.delete(path);
    } catch (IOException e) {
        throw new UncheckedIOException(e);
    }
}

list.forEach(this::safeDelete);
```

---

### س10: إيه الفرق بين `Supplier.get()` و `Optional.orElse()` و `Optional.orElseGet()`؟

**الإجابة:**

```java
// orElse - دايماً بيتنفذ argument حتى لو Optional مش فاضي
Optional<String> opt = Optional.of("Hello");
String result1 = opt.orElse(expensiveOperation());  // expensiveOperation() هتتنفذ!

// orElseGet - بياخد Supplier، بيتنفذ لو Optional فاضي بس
String result2 = opt.orElseGet(() -> expensiveOperation());  // مش هتتنفذ!

// الفرق في الـ performance
Optional<String> present = Optional.of("value");
Optional<String> empty = Optional.empty();

// مع orElse - getDefault() هتتنفذ في الحالتين!
present.orElse(getDefault());  // getDefault() called ❌
empty.orElse(getDefault());    // getDefault() called

// مع orElseGet - getDefault() هتتنفذ لو فاضي بس
present.orElseGet(() -> getDefault());  // NOT called ✅
empty.orElseGet(() -> getDefault());    // called
```

---

## أسئلة كود

### س11: اكتب Lambda لـ sort list بترتيب معين

```java
List<Person> people = Arrays.asList(
    new Person("أحمد", 25),
    new Person("محمد", 30),
    new Person("علي", 25),
    new Person("عمر", 20)
);

// بالعمر
people.sort((p1, p2) -> p1.getAge() - p2.getAge());
// أو
people.sort(Comparator.comparingInt(Person::getAge));

// بالعمر ثم الاسم
people.sort(Comparator
    .comparingInt(Person::getAge)
    .thenComparing(Person::getName));

// عكسي
people.sort(Comparator.comparingInt(Person::getAge).reversed());

// Null-safe
people.sort(Comparator.comparing(
    Person::getName,
    Comparator.nullsLast(Comparator.naturalOrder())
));
```

---

### س12: اكتب Lambda لـ filter و transform list

```java
List<String> names = Arrays.asList("Ahmed", "Mohamed", "Ali", "Omar", "");

// Filter empty strings و transform to uppercase
List<String> result = names.stream()
    .filter(s -> s != null && !s.isEmpty())  // Predicate
    .map(String::toUpperCase)                 // Function
    .collect(Collectors.toList());

// باستخدام method reference
Predicate<String> notEmpty = s -> s != null && !s.isEmpty();
Function<String, String> toUpper = String::toUpperCase;

List<String> result2 = names.stream()
    .filter(notEmpty)
    .map(toUpper)
    .collect(Collectors.toList());
```

---

### س13: اكتب Custom Functional Interface للـ Validation

```java
@FunctionalInterface
interface Validator<T> {
    boolean validate(T value);
    
    default Validator<T> and(Validator<T> other) {
        return value -> this.validate(value) && other.validate(value);
    }
    
    default Validator<T> or(Validator<T> other) {
        return value -> this.validate(value) || other.validate(value);
    }
    
    default Validator<T> negate() {
        return value -> !this.validate(value);
    }
    
    static <T> Validator<T> notNull() {
        return value -> value != null;
    }
}

// استخدام
Validator<String> notEmpty = s -> !s.isEmpty();
Validator<String> maxLength10 = s -> s.length() <= 10;

Validator<String> combined = Validator.<String>notNull()
    .and(notEmpty)
    .and(maxLength10);

System.out.println(combined.validate("Hello"));  // true
System.out.println(combined.validate(""));       // false
System.out.println(combined.validate(null));     // false
```

---

### س14: اكتب higher-order function بتاخد و بترجع functions

```java
// Function بتاخد function وبترجع function
public static <T, R> Function<T, R> memoize(Function<T, R> fn) {
    Map<T, R> cache = new ConcurrentHashMap<>();
    return input -> cache.computeIfAbsent(input, fn);
}

// استخدام
Function<Integer, Integer> factorial = memoize(n -> {
    System.out.println("Computing factorial of " + n);
    int result = 1;
    for (int i = 2; i <= n; i++) result *= i;
    return result;
});

System.out.println(factorial.apply(5));  // Computing... 120
System.out.println(factorial.apply(5));  // 120 (from cache, no "Computing...")

// Function composition
public static <A, B, C> Function<A, C> compose(
        Function<A, B> f1, 
        Function<B, C> f2) {
    return a -> f2.apply(f1.apply(a));
}

Function<String, Integer> length = String::length;
Function<Integer, Boolean> isEven = n -> n % 2 == 0;
Function<String, Boolean> hasEvenLength = compose(length, isEven);

System.out.println(hasEvenLength.apply("Hi"));    // true (2)
System.out.println(hasEvenLength.apply("Hello")); // false (5)
```

---

### س15: اكتب Lambda-based Builder Pattern

```java
public class PersonBuilder {
    private String name;
    private int age;
    private String email;
    
    public PersonBuilder with(Consumer<PersonBuilder> builderConsumer) {
        builderConsumer.accept(this);
        return this;
    }
    
    public PersonBuilder name(String name) {
        this.name = name;
        return this;
    }
    
    public PersonBuilder age(int age) {
        this.age = age;
        return this;
    }
    
    public PersonBuilder email(String email) {
        this.email = email;
        return this;
    }
    
    public Person build() {
        return new Person(name, age, email);
    }
}

// استخدام
Person person = new PersonBuilder()
    .with(b -> b.name("أحمد"))
    .with(b -> b.age(25))
    .with(b -> b.email("ahmed@example.com"))
    .build();

// أو بشكل أبسط
Person person2 = new PersonBuilder()
    .with(b -> {
        b.name("محمد");
        b.age(30);
        b.email("mohamed@example.com");
    })
    .build();
```

---

## نصائح للانترفيو 💡

1. **افهم الـ Syntax كويس** - اعرف إمتى الأقواس اختيارية وإمتى لازم curly braces
2. **اعرف الفرق بين الـ Built-in Interfaces** - Predicate vs Function vs Consumer vs Supplier
3. **افهم Effectively Final** - ده سؤال شائع جداً
4. **اعرف أنواع Method References الأربعة** - واعرف تحول من Lambda لـ Method Reference والعكس
5. **افهم الفرق بين Lambda و Anonymous Class** - خصوصاً موضوع `this`
6. **اتدرب على كتابة Custom Functional Interfaces** - مع default methods
7. **اعرف إزاي تتعامل مع Checked Exceptions** في Lambda

---

## موارد إضافية 📚

- [Oracle Lambda Expressions Tutorial](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
- [Java 8 Functional Interfaces](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)
- [Method References](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)

---

*تم إعداد هذا الدليل بالعامية المصرية عشان يكون سهل الفهم والمراجعة* 🇪🇬
