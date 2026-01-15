# 🎯 Method References & Built-in Functional Interfaces في Java - دليل شامل بالمصري

## المحتويات
- [الجزء الأول: Method References](#الجزء-الأول-method-references)
  - [مقدمة عن Method References](#مقدمة-عن-method-references)
  - [أنواع Method References](#أنواع-method-references)
  - [Method Reference مع Constructors](#method-reference-مع-constructors)
  - [Method Reference مع Arrays](#method-reference-مع-arrays)
  - [إمتى نستخدم Method Reference](#إمتى-نستخدم-method-reference)
- [الجزء الثاني: Built-in Functional Interfaces](#الجزء-الثاني-built-in-functional-interfaces)
  - [الـ Core Interfaces](#الـ-core-interfaces)
  - [الـ Bi-Variants](#الـ-bi-variants)
  - [الـ Operators](#الـ-operators)
  - [Primitive Specializations](#primitive-specializations)
- [الجزء الثالث: أمثلة عملية متكاملة](#الجزء-الثالث-أمثلة-عملية-متكاملة)
- [أسئلة الانترفيو](#أسئلة-الانترفيو)

---

# الجزء الأول: Method References

## مقدمة عن Method References

### يعني إيه Method Reference؟

الـ **Method Reference** هي طريقة مختصرة لكتابة Lambda Expression لما الـ Lambda بتعمل call لـ method موجود.

بدل ما تكتب Lambda كاملة، بتستخدم الـ `::` operator عشان تشير للـ method مباشرة.

```java
// Lambda Expression
Function<String, Integer> lambda = s -> s.length();

// Method Reference - نفس الحاجة بس أقصر!
Function<String, Integer> methodRef = String::length;
```

### الـ Syntax الأساسي

```java
ClassName::methodName
// أو
instance::methodName
// أو
ClassName::new
```

### ليه نستخدم Method References؟

| الميزة | الشرح |
|--------|-------|
| **Readability** | الكود أوضح وأسهل في القراءة |
| **Conciseness** | كود أقصر |
| **Reusability** | بتشير لـ method موجود بالفعل |
| **Intent** | بتوضح إنك بتستخدم method معينة |

---

## أنواع Method References

### النظرة العامة

```
Method References
├── 1. Static Method Reference      → ClassName::staticMethod
├── 2. Instance Method (Bound)      → instance::instanceMethod
├── 3. Instance Method (Unbound)    → ClassName::instanceMethod
└── 4. Constructor Reference        → ClassName::new
```

---

## 1. Static Method Reference

### التعريف

بتستخدم للـ static methods. الـ syntax هو `ClassName::staticMethodName`.

```java
// Lambda
Function<String, Integer> parse1 = s -> Integer.parseInt(s);

// Method Reference
Function<String, Integer> parse2 = Integer::parseInt;

// الاتنين بيعملوا نفس الحاجة
System.out.println(parse1.apply("42"));  // 42
System.out.println(parse2.apply("42"));  // 42
```

### أمثلة متنوعة

```java
// ===== Math class =====
// Lambda
Function<Double, Double> sqrt1 = d -> Math.sqrt(d);
BiFunction<Double, Double, Double> pow1 = (a, b) -> Math.pow(a, b);
Function<Double, Double> abs1 = d -> Math.abs(d);
Supplier<Double> random1 = () -> Math.random();

// Method Reference
Function<Double, Double> sqrt2 = Math::sqrt;
BiFunction<Double, Double, Double> pow2 = Math::pow;
Function<Double, Double> abs2 = Math::abs;
Supplier<Double> random2 = Math::random;

System.out.println(sqrt2.apply(16.0));      // 4.0
System.out.println(pow2.apply(2.0, 3.0));   // 8.0
System.out.println(abs2.apply(-5.0));       // 5.0
System.out.println(random2.get());          // 0.xxxxx

// ===== Integer class =====
Function<String, Integer> parseInt = Integer::parseInt;
BiFunction<String, Integer, Integer> parseIntRadix = Integer::parseInt;
Function<Integer, String> toBinary = Integer::toBinaryString;
Function<Integer, String> toHex = Integer::toHexString;
Comparator<Integer> compare = Integer::compare;

System.out.println(parseInt.apply("100"));              // 100
System.out.println(parseIntRadix.apply("1010", 2));    // 10
System.out.println(toBinary.apply(10));                 // 1010
System.out.println(toHex.apply(255));                   // ff
System.out.println(compare.compare(5, 3));              // 1

// ===== String class =====
Function<Object, String> valueOf = String::valueOf;
BiFunction<String, String, String> join = String::join;

System.out.println(valueOf.apply(123));                  // "123"
System.out.println(join.apply("-", "a", "b", "c"));     // "a-b-c" - ده مش هيشتغل كده

// ===== Collections class =====
Consumer<List<?>> shuffle = Collections::shuffle;
Consumer<List<? extends Comparable>> sort = Collections::sort;

List<Integer> numbers = new ArrayList<>(Arrays.asList(3, 1, 4, 1, 5));
shuffle.accept(numbers);
System.out.println(numbers);  // Random order

// ===== System class =====
Consumer<Object> printer = System.out::println;  // ده actually instance method
LongSupplier currentTime = System::currentTimeMillis;
LongSupplier nanoTime = System::nanoTime;

System.out.println(currentTime.getAsLong());  // 1705312345678
```

### استخدام مع Streams

```java
List<String> numberStrings = Arrays.asList("1", "2", "3", "4", "5");

// تحويل Strings لـ Integers
List<Integer> numbers = numberStrings.stream()
    .map(Integer::parseInt)          // Static method reference
    .collect(Collectors.toList());
System.out.println(numbers);  // [1, 2, 3, 4, 5]

// حساب الـ square root
List<Double> values = Arrays.asList(4.0, 9.0, 16.0, 25.0);
List<Double> sqrtValues = values.stream()
    .map(Math::sqrt)
    .collect(Collectors.toList());
System.out.println(sqrtValues);  // [2.0, 3.0, 4.0, 5.0]

// Sorting with comparator
List<Integer> nums = Arrays.asList(5, 2, 8, 1, 9);
nums.sort(Integer::compare);
System.out.println(nums);  // [1, 2, 5, 8, 9]
```

### Custom Static Methods

```java
public class StringUtils {
    
    public static String capitalize(String s) {
        if (s == null || s.isEmpty()) return s;
        return s.substring(0, 1).toUpperCase() + s.substring(1).toLowerCase();
    }
    
    public static boolean isValidEmail(String s) {
        return s != null && s.contains("@") && s.contains(".");
    }
    
    public static String reverse(String s) {
        return new StringBuilder(s).reverse().toString();
    }
    
    public static int countWords(String s) {
        return s.trim().split("\\s+").length;
    }
}

// استخدام Method References
Function<String, String> capitalize = StringUtils::capitalize;
Predicate<String> isValidEmail = StringUtils::isValidEmail;
Function<String, String> reverse = StringUtils::reverse;
ToIntFunction<String> countWords = StringUtils::countWords;

List<String> names = Arrays.asList("ahmed", "MOHAMED", "ALI");
List<String> capitalized = names.stream()
    .map(StringUtils::capitalize)
    .collect(Collectors.toList());
System.out.println(capitalized);  // [Ahmed, Mohamed, Ali]

List<String> emails = Arrays.asList("valid@email.com", "invalid", "test@test.org");
List<String> validEmails = emails.stream()
    .filter(StringUtils::isValidEmail)
    .collect(Collectors.toList());
System.out.println(validEmails);  // [valid@email.com, test@test.org]
```

---

## 2. Instance Method Reference (Bound)

### التعريف

بتستخدم لما عندك **instance محدد** وعايز تـ call method عليه. الـ instance معروف ومحدد وقت كتابة الكود.

```java
String prefix = "Hello, ";

// Lambda
Function<String, String> greet1 = s -> prefix.concat(s);

// Method Reference (Bound to 'prefix' instance)
Function<String, String> greet2 = prefix::concat;

System.out.println(greet1.apply("World"));  // Hello, World
System.out.println(greet2.apply("World"));  // Hello, World
```

### أمثلة متنوعة

```java
// ===== String instance methods =====
String str = "Hello World";

Supplier<Integer> length = str::length;
Supplier<String> toUpper = str::toUpperCase;
Supplier<String> toLower = str::toLowerCase;
Function<String, Boolean> startsWith = str::startsWith;
Function<String, Boolean> endsWith = str::endsWith;
Function<String, Boolean> contains = str::contains;
Function<String, Integer> indexOf = str::indexOf;
Function<Integer, Character> charAt = str::charAt;

System.out.println(length.get());              // 11
System.out.println(toUpper.get());             // HELLO WORLD
System.out.println(startsWith.apply("Hello")); // true
System.out.println(charAt.apply(0));           // H

// ===== StringBuilder instance =====
StringBuilder sb = new StringBuilder();

Consumer<String> append = sb::append;
Supplier<String> getResult = sb::toString;
Supplier<Integer> getLength = sb::length;

append.accept("Hello");
append.accept(" ");
append.accept("World");
System.out.println(getResult.get());  // Hello World

// ===== List instance =====
List<String> names = new ArrayList<>();

Consumer<String> addName = names::add;
Predicate<String> containsName = names::contains;
Consumer<String> removeName = names::remove;
Supplier<Integer> size = names::size;

addName.accept("أحمد");
addName.accept("محمد");
System.out.println(containsName.test("أحمد"));  // true
System.out.println(size.get());                  // 2

// ===== PrintStream instance (System.out) =====
PrintStream out = System.out;

Consumer<Object> print = out::print;
Consumer<Object> println = out::println;

// أو مباشرة
Consumer<String> printer = System.out::println;

List<String> items = Arrays.asList("A", "B", "C");
items.forEach(System.out::println);  // A, B, C

// ===== Random instance =====
Random random = new Random();

IntSupplier nextInt = random::nextInt;
DoubleSupplier nextDouble = random::nextDouble;
Supplier<Boolean> nextBoolean = random::nextBoolean;
IntUnaryOperator boundedInt = random::nextInt;

System.out.println(nextInt.getAsInt());         // Random int
System.out.println(nextDouble.getAsDouble());   // Random double 0.0-1.0
System.out.println(boundedInt.applyAsInt(100)); // Random 0-99

// ===== Object comparison =====
String target = "Hello";
Predicate<String> equalsTarget = target::equals;
Predicate<String> equalsIgnoreCase = target::equalsIgnoreCase;

System.out.println(equalsTarget.test("Hello"));       // true
System.out.println(equalsTarget.test("hello"));       // false
System.out.println(equalsIgnoreCase.test("hello"));   // true
```

### استخدام في Real-World Scenarios

```java
// Scenario 1: Logger
Logger logger = Logger.getLogger("MyApp");

Consumer<String> logInfo = logger::info;
Consumer<String> logWarning = logger::warning;
Consumer<String> logSevere = logger::severe;

List<String> messages = Arrays.asList("Started", "Processing", "Completed");
messages.forEach(logger::info);

// Scenario 2: StringBuilder for building output
StringBuilder output = new StringBuilder();
Consumer<String> appendLine = s -> output.append(s).append("\n");
// أو
Consumer<Object> appendObj = output::append;

List<Integer> numbers = Arrays.asList(1, 2, 3);
numbers.stream()
    .map(n -> "Number: " + n)
    .forEach(appendLine);
System.out.println(output);

// Scenario 3: Configuration object
class Config {
    private String prefix = "APP_";
    
    public String addPrefix(String key) {
        return prefix + key;
    }
}

Config config = new Config();
Function<String, String> addPrefix = config::addPrefix;

List<String> keys = Arrays.asList("NAME", "VERSION", "ENV");
List<String> prefixedKeys = keys.stream()
    .map(config::addPrefix)
    .collect(Collectors.toList());
System.out.println(prefixedKeys);  // [APP_NAME, APP_VERSION, APP_ENV]
```

---

## 3. Instance Method Reference (Unbound)

### التعريف

بتستخدم لما الـ instance مش معروف وقت كتابة الكود - الـ instance هو **أول parameter** في الـ Lambda.

```java
// Lambda - 's' هو الـ instance اللي هنـ call عليه length()
Function<String, Integer> length1 = s -> s.length();

// Method Reference (Unbound) - الـ String اللي هيتمرر هو اللي هيـ call length()
Function<String, Integer> length2 = String::length;

System.out.println(length1.apply("Hello"));  // 5
System.out.println(length2.apply("Hello"));  // 5
```

### الفرق بين Bound و Unbound

```java
// ===== BOUND - الـ instance معروف =====
String fixed = "Hello";
Supplier<Integer> boundLength = fixed::length;  // مفيش parameter
// equivalent to: () -> fixed.length()

// ===== UNBOUND - الـ instance هو الـ parameter =====
Function<String, Integer> unboundLength = String::length;  // فيه parameter
// equivalent to: (s) -> s.length()

// الفرق في الاستخدام
System.out.println(boundLength.get());           // 5 (من fixed)
System.out.println(unboundLength.apply("Hi"));   // 2 (من الـ parameter)
```

### أمثلة متنوعة

```java
// ===== String methods (no parameters) =====
Function<String, Integer> length = String::length;
Function<String, String> toUpperCase = String::toUpperCase;
Function<String, String> toLowerCase = String::toLowerCase;
Function<String, String> trim = String::trim;
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isBlank = String::isBlank;  // Java 11+

System.out.println(length.apply("Hello"));        // 5
System.out.println(toUpperCase.apply("hello"));   // HELLO
System.out.println(isEmpty.test(""));             // true

// ===== String methods (with parameters) =====
BiFunction<String, String, Boolean> startsWith = String::startsWith;
BiFunction<String, String, Boolean> endsWith = String::endsWith;
BiFunction<String, String, Boolean> contains = String::contains;
BiFunction<String, String, String> concat = String::concat;
BiFunction<String, String, Integer> indexOf = String::indexOf;
BiFunction<String, CharSequence, Boolean> containsCS = String::contains;

System.out.println(startsWith.apply("Hello", "He"));   // true
System.out.println(concat.apply("Hello", " World"));   // Hello World
System.out.println(indexOf.apply("Hello", "l"));       // 2

// ===== Object methods =====
Function<Object, String> toString = Object::toString;
Function<Object, Integer> hashCode = Object::hashCode;
BiPredicate<Object, Object> equals = Object::equals;

// ===== Comparable =====
// compareTo on the first argument compared to the second
BiFunction<String, String, Integer> compareStrings = String::compareTo;
Comparator<String> stringComparator = String::compareTo;

System.out.println(compareStrings.apply("a", "b"));  // -1
System.out.println(compareStrings.apply("b", "a"));  // 1

List<String> names = Arrays.asList("Ziad", "Ahmed", "Mohamed");
names.sort(String::compareTo);
System.out.println(names);  // [Ahmed, Mohamed, Ziad]

// ===== Number subclasses =====
Function<Integer, Integer> intValue = Integer::intValue;
Function<Integer, Double> doubleValue = Integer::doubleValue;
Function<Integer, String> intToBinary = Integer::toBinaryString;

// ===== Custom class methods =====
class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    public boolean isAdult() { return age >= 18; }
}

Function<Person, String> getName = Person::getName;
Function<Person, Integer> getAge = Person::getAge;
ToIntFunction<Person> getAgeInt = Person::getAge;
Predicate<Person> isAdult = Person::isAdult;

List<Person> people = Arrays.asList(
    new Person("أحمد", 25),
    new Person("محمد", 15),
    new Person("علي", 30)
);

// Get all names
List<String> allNames = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());
System.out.println(allNames);  // [أحمد, محمد, علي]

// Filter adults
List<Person> adults = people.stream()
    .filter(Person::isAdult)
    .collect(Collectors.toList());

// Sort by age
people.sort(Comparator.comparingInt(Person::getAge));

// Get average age
double avgAge = people.stream()
    .mapToInt(Person::getAge)
    .average()
    .orElse(0);
```

### Two-Argument Instance Methods

```java
// لما الـ instance method بتاخد parameter
// الـ first argument هو الـ instance
// الـ second argument هو الـ method parameter

// String::startsWith
// equivalent to: (str, prefix) -> str.startsWith(prefix)
BiPredicate<String, String> startsWith = String::startsWith;

// String::concat
// equivalent to: (str1, str2) -> str1.concat(str2)
BiFunction<String, String, String> concat = String::concat;

// String::compareTo
// equivalent to: (str1, str2) -> str1.compareTo(str2)
Comparator<String> comparator = String::compareTo;

// استخدام
System.out.println(startsWith.test("Hello", "He"));  // true
System.out.println(concat.apply("Hello", " World")); // Hello World

// مع Streams
List<String> words = Arrays.asList("apple", "banana", "apricot", "cherry");
List<String> aWords = words.stream()
    .filter(word -> startsWith.test(word, "a"))  // أو تستخدم Lambda مباشرة
    .collect(Collectors.toList());
System.out.println(aWords);  // [apple, apricot]
```

---

## 4. Method Reference مع Constructors

### التعريف

Constructor Reference بتستخدم `ClassName::new` عشان تعمل instances جديدة.

```java
// Lambda
Supplier<ArrayList<String>> list1 = () -> new ArrayList<>();

// Constructor Reference
Supplier<ArrayList<String>> list2 = ArrayList::new;

// الاتنين بيعملوا نفس الحاجة
List<String> newList = list2.get();
```

### No-Argument Constructor

```java
// Supplier - مفيش parameters
Supplier<StringBuilder> sbSupplier = StringBuilder::new;
Supplier<ArrayList<String>> listSupplier = ArrayList::new;
Supplier<HashMap<String, Integer>> mapSupplier = HashMap::new;
Supplier<Random> randomSupplier = Random::new;

StringBuilder sb = sbSupplier.get();
List<String> list = listSupplier.get();
Map<String, Integer> map = mapSupplier.get();

// Custom class
class Person {
    public Person() { }
}

Supplier<Person> personSupplier = Person::new;
Person p = personSupplier.get();
```

### Single-Argument Constructor

```java
// Function<T, R> - parameter واحد
Function<String, Integer> integerFactory = Integer::new;
Function<String, StringBuilder> sbFactory = StringBuilder::new;
Function<Integer, ArrayList<?>> sizedListFactory = ArrayList::new;

Integer num = integerFactory.apply("42");
StringBuilder sb = sbFactory.apply("Initial");
List<?> sized = sizedListFactory.apply(100);  // Initial capacity 100

// Custom class
class Person {
    private String name;
    public Person(String name) { this.name = name; }
    public String getName() { return name; }
}

Function<String, Person> personFactory = Person::new;

List<String> names = Arrays.asList("أحمد", "محمد", "علي");
List<Person> people = names.stream()
    .map(Person::new)  // Constructor reference
    .collect(Collectors.toList());

people.forEach(p -> System.out.println(p.getName()));
```

### Two-Argument Constructor

```java
// BiFunction<T, U, R> - 2 parameters
BiFunction<String, Integer, Person> personFactory = Person::new;

class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

Person p = personFactory.apply("أحمد", 25);

// مع data
List<String[]> data = Arrays.asList(
    new String[]{"أحمد", "25"},
    new String[]{"محمد", "30"},
    new String[]{"علي", "28"}
);

// محتاجين نعمل custom functional interface أو نستخدم Lambda
List<Person> people = data.stream()
    .map(arr -> new Person(arr[0], Integer.parseInt(arr[1])))
    .collect(Collectors.toList());
```

### Constructor Reference مع Generics

```java
// Generic class
class Box<T> {
    private T content;
    
    public Box() { }
    
    public Box(T content) {
        this.content = content;
    }
    
    public T getContent() { return content; }
    public void setContent(T content) { this.content = content; }
}

// Factory suppliers
Supplier<Box<String>> stringBoxFactory = Box::new;
Supplier<Box<Integer>> intBoxFactory = Box::new;

Box<String> stringBox = stringBoxFactory.get();
Box<Integer> intBox = intBoxFactory.get();

// With parameter
Function<String, Box<String>> stringBoxWithContent = Box::new;
Box<String> filledBox = stringBoxWithContent.apply("Hello");
```

---

## 5. Method Reference مع Arrays

### Array Constructor Reference

```java
// Lambda
Function<Integer, String[]> arrayFactory1 = size -> new String[size];
IntFunction<String[]> arrayFactory2 = size -> new String[size];

// Array Constructor Reference
Function<Integer, String[]> arrayFactory3 = String[]::new;
IntFunction<String[]> arrayFactory4 = String[]::new;

String[] arr = arrayFactory4.apply(5);  // String[5]

// استخدام مع Stream.toArray()
List<String> list = Arrays.asList("A", "B", "C");

// Lambda
String[] arr1 = list.stream().toArray(size -> new String[size]);

// Array Constructor Reference
String[] arr2 = list.stream().toArray(String[]::new);

System.out.println(Arrays.toString(arr2));  // [A, B, C]

// Different types
Integer[] intArr = Stream.of(1, 2, 3).toArray(Integer[]::new);
Double[] doubleArr = Stream.of(1.0, 2.0, 3.0).toArray(Double[]::new);
Person[] personArr = people.stream().toArray(Person[]::new);
```

### Multi-dimensional Arrays

```java
// 2D Array
Function<Integer, String[][]> array2DFactory = String[][]::new;
String[][] arr2D = array2DFactory.apply(3);  // String[3][]

// 3D Array
Function<Integer, int[][][]> array3DFactory = int[][][]::new;
int[][][] arr3D = array3DFactory.apply(2);  // int[2][][]
```

### Array Methods

```java
// Arrays class static methods
Consumer<int[]> sortIntArray = Arrays::sort;
Consumer<Object[]> sortObjectArray = Arrays::sort;
Function<int[], String> intArrayToString = Arrays::toString;
Function<Object[], String> arrayToString = Arrays::toString;

int[] numbers = {5, 2, 8, 1, 9};
sortIntArray.accept(numbers);
System.out.println(intArrayToString.apply(numbers));  // [1, 2, 5, 8, 9]

// Arrays.copyOf
BiFunction<int[], Integer, int[]> copyIntArray = Arrays::copyOf;
int[] original = {1, 2, 3};
int[] copy = copyIntArray.apply(original, 5);
System.out.println(Arrays.toString(copy));  // [1, 2, 3, 0, 0]
```

---

## إمتى نستخدم Method Reference

### ✅ استخدم Method Reference لما:

```java
// 1. الـ Lambda بتعمل call لـ method واحد بس
list.forEach(System.out::println);           // ✅
list.stream().map(String::toUpperCase);      // ✅
list.sort(String::compareTo);                // ✅

// 2. الـ method signature بتطابق الـ functional interface
Function<String, Integer> f = String::length; // ✅

// 3. الكود بيكون أوضح
people.stream()
    .map(Person::getName)      // ✅ واضح إننا بناخد الـ name
    .collect(Collectors.toList());

// 4. Constructor call بسيط
Supplier<List<String>> factory = ArrayList::new;  // ✅
```

### ❌ ما تستخدمش Method Reference لما:

```java
// 1. محتاج تعمل حاجة إضافية
list.forEach(s -> System.out.println("Item: " + s));  // ✅ Lambda
// list.forEach(System.out::println);  // ❌ مش هيطبع "Item: "

// 2. الـ arguments مش بنفس الترتيب
BiFunction<Integer, Integer, Integer> subtract = (a, b) -> b - a;
// مفيش method reference لـ (a, b) -> b - a

// 3. محتاج تعمل transformation على الـ argument
Function<String, Integer> f = s -> Integer.parseInt(s.trim());
// مش ممكن method reference لأن فيه .trim()

// 4. الـ Lambda معقدة
list.forEach(s -> {
    String processed = s.trim().toUpperCase();
    if (!processed.isEmpty()) {
        System.out.println(processed);
    }
});
// Lambda أوضح هنا

// 5. الـ method overloaded ومش واضح أنهي version
// System.out.println overloaded - ممكن يكون confusing
```

### جدول المقارنة

| Lambda | Method Reference | ملاحظات |
|--------|------------------|---------|
| `s -> s.length()` | `String::length` | ✅ Method Ref أوضح |
| `s -> System.out.println(s)` | `System.out::println` | ✅ Method Ref أوضح |
| `s -> s.toUpperCase()` | `String::toUpperCase` | ✅ Method Ref أوضح |
| `() -> new ArrayList<>()` | `ArrayList::new` | ✅ Method Ref أوضح |
| `s -> "Hello " + s` | ❌ | Lambda بس |
| `(a, b) -> a + b` | ❌ | Lambda بس (إلا لو فيه static method) |
| `s -> s.substring(0, 5)` | ❌ | Lambda بس (فيه parameters) |

---

# الجزء الثاني: Built-in Functional Interfaces

## نظرة عامة على java.util.function

```
java.util.function
├── Core Interfaces
│   ├── Predicate<T>        → T → boolean
│   ├── Function<T, R>      → T → R
│   ├── Consumer<T>         → T → void
│   └── Supplier<T>         → () → T
│
├── Bi-Variants (Two inputs)
│   ├── BiPredicate<T, U>   → (T, U) → boolean
│   ├── BiFunction<T, U, R> → (T, U) → R
│   └── BiConsumer<T, U>    → (T, U) → void
│
├── Operators (Same type in/out)
│   ├── UnaryOperator<T>    → T → T (extends Function<T, T>)
│   └── BinaryOperator<T>   → (T, T) → T (extends BiFunction<T, T, T>)
│
└── Primitive Specializations
    ├── IntPredicate, LongPredicate, DoublePredicate
    ├── IntFunction<R>, LongFunction<R>, DoubleFunction<R>
    ├── IntConsumer, LongConsumer, DoubleConsumer
    ├── IntSupplier, LongSupplier, DoubleSupplier
    ├── IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator
    ├── IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator
    ├── ToIntFunction<T>, ToLongFunction<T>, ToDoubleFunction<T>
    └── ...and more
```

---

## الـ Core Interfaces

### 1. Predicate<T> - Testing/Filtering

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    
    // Default methods
    default Predicate<T> and(Predicate<? super T> other);
    default Predicate<T> or(Predicate<? super T> other);
    default Predicate<T> negate();
    
    // Static methods
    static <T> Predicate<T> isEqual(Object targetRef);
    static <T> Predicate<T> not(Predicate<? super T> target);  // Java 11+
}
```

#### أمثلة شاملة

```java
// ===== Basic Predicates =====
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isGreaterThan10 = n -> n > 10;
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNull = Objects::isNull;
Predicate<String> isNotNull = Objects::nonNull;

// Testing
System.out.println(isPositive.test(5));   // true
System.out.println(isEven.test(4));       // true
System.out.println(isEmpty.test(""));     // true

// ===== Combining with AND =====
Predicate<Integer> isPositiveAndEven = isPositive.and(isEven);
Predicate<Integer> isPositiveEvenAndBig = isPositive.and(isEven).and(isGreaterThan10);

System.out.println(isPositiveAndEven.test(4));        // true
System.out.println(isPositiveAndEven.test(-4));       // false
System.out.println(isPositiveEvenAndBig.test(12));    // true
System.out.println(isPositiveEvenAndBig.test(8));     // false (not > 10)

// ===== Combining with OR =====
Predicate<Integer> isPositiveOrEven = isPositive.or(isEven);

System.out.println(isPositiveOrEven.test(3));    // true (positive)
System.out.println(isPositiveOrEven.test(-4));   // true (even)
System.out.println(isPositiveOrEven.test(-3));   // false

// ===== Negate =====
Predicate<Integer> isNotPositive = isPositive.negate();
Predicate<String> isNotEmpty = isEmpty.negate();

System.out.println(isNotPositive.test(-5));  // true
System.out.println(isNotEmpty.test("Hi"));   // true

// ===== Predicate.not() - Java 11+ =====
Predicate<String> notEmpty = Predicate.not(String::isEmpty);
Predicate<String> notBlank = Predicate.not(String::isBlank);

// ===== Predicate.isEqual() =====
Predicate<String> isHello = Predicate.isEqual("Hello");
Predicate<Integer> isTen = Predicate.isEqual(10);

System.out.println(isHello.test("Hello"));  // true
System.out.println(isHello.test("World"));  // false

// ===== استخدام مع Streams =====
List<Integer> numbers = Arrays.asList(-5, -2, 0, 3, 6, 9, 12, 15);

// Filter positive
List<Integer> positives = numbers.stream()
    .filter(isPositive)
    .collect(Collectors.toList());
System.out.println(positives);  // [3, 6, 9, 12, 15]

// Filter positive and even
List<Integer> positiveEvens = numbers.stream()
    .filter(isPositive.and(isEven))
    .collect(Collectors.toList());
System.out.println(positiveEvens);  // [6, 12]

// Count matching
long count = numbers.stream()
    .filter(isPositiveAndEven)
    .count();
System.out.println(count);  // 2

// anyMatch, allMatch, noneMatch
boolean anyPositive = numbers.stream().anyMatch(isPositive);
boolean allPositive = numbers.stream().allMatch(isPositive);
boolean noneNegative = numbers.stream().noneMatch(n -> n < 0);

System.out.println(anyPositive);   // true
System.out.println(allPositive);   // false
System.out.println(noneNegative);  // false

// ===== Complex Predicates =====
Predicate<String> isValidEmail = s -> s != null 
    && !s.isEmpty()
    && s.contains("@")
    && s.indexOf("@") > 0
    && s.indexOf("@") < s.length() - 1
    && s.contains(".");

Predicate<String> isValidPassword = s -> s != null
    && s.length() >= 8
    && s.matches(".*[A-Z].*")
    && s.matches(".*[a-z].*")
    && s.matches(".*[0-9].*");

System.out.println(isValidEmail.test("test@example.com"));  // true
System.out.println(isValidPassword.test("Pass123!"));       // true
```

### 2. Function<T, R> - Transformation

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    
    // Default methods
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before);
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after);
    
    // Static method
    static <T> Function<T, T> identity();
}
```

#### أمثلة شاملة

```java
// ===== Basic Functions =====
Function<String, Integer> stringLength = String::length;
Function<String, String> toUpperCase = String::toUpperCase;
Function<String, String> toLowerCase = String::toLowerCase;
Function<String, String> trim = String::trim;
Function<Integer, Integer> square = n -> n * n;
Function<Integer, String> intToString = String::valueOf;
Function<String, Integer> stringToInt = Integer::parseInt;
Function<Integer, Double> intToDouble = Integer::doubleValue;

// Applying
System.out.println(stringLength.apply("Hello"));  // 5
System.out.println(toUpperCase.apply("hello"));   // HELLO
System.out.println(square.apply(5));              // 25

// ===== Function Composition with andThen =====
// andThen: execute this function, THEN execute the next
Function<String, Integer> lengthThenSquare = stringLength.andThen(square);
// "Hello" → 5 → 25

System.out.println(lengthThenSquare.apply("Hello"));  // 25

Function<String, String> processString = trim
    .andThen(toLowerCase)
    .andThen(s -> s.replace(" ", "_"));
// "  Hello World  " → "Hello World" → "hello world" → "hello_world"

System.out.println(processString.apply("  Hello World  "));  // hello_world

// ===== Function Composition with compose =====
// compose: execute the other function FIRST, then this one
Function<Integer, Integer> squareThenDouble = square.compose(n -> n * 2);
// 3 → 6 → 36

// مقارنة:
Function<Integer, Integer> f1 = square.andThen(n -> n * 2);  // square first
Function<Integer, Integer> f2 = square.compose(n -> n * 2); // double first

System.out.println(f1.apply(3));  // 9 * 2 = 18
System.out.println(f2.apply(3));  // (3*2)² = 36

// ===== Function.identity() =====
Function<String, String> identity = Function.identity();
System.out.println(identity.apply("Hello"));  // Hello

// Use case: Collectors.toMap
List<String> names = Arrays.asList("Ahmed", "Mohamed", "Ali");
Map<String, Integer> nameLengths = names.stream()
    .collect(Collectors.toMap(
        Function.identity(),  // key = name itself
        String::length        // value = length
    ));
System.out.println(nameLengths);  // {Ahmed=5, Mohamed=7, Ali=3}

// ===== استخدام مع Streams =====
List<String> words = Arrays.asList("  hello  ", "  WORLD  ", "  Java  ");

List<String> processed = words.stream()
    .map(trim)
    .map(toLowerCase)
    .collect(Collectors.toList());
System.out.println(processed);  // [hello, world, java]

// أو باستخدام composed function
List<String> processed2 = words.stream()
    .map(trim.andThen(toLowerCase))
    .collect(Collectors.toList());

// ===== Complex Transformations =====
class Person {
    String firstName;
    String lastName;
    int age;
    
    Person(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }
}

Function<Person, String> getFullName = p -> p.firstName + " " + p.lastName;
Function<Person, String> getInitials = p -> 
    "" + p.firstName.charAt(0) + p.lastName.charAt(0);
Function<Person, Boolean> isAdult = p -> p.age >= 18;

Person person = new Person("أحمد", "محمد", 25);
System.out.println(getFullName.apply(person));   // أحمد محمد
System.out.println(getInitials.apply(person));   // أم
System.out.println(isAdult.apply(person));       // true

// ===== Data Pipeline =====
Function<String, String> sanitize = s -> s.trim().toLowerCase();
Function<String, String> normalize = s -> s.replaceAll("[^a-z0-9]", "");
Function<String, String> truncate = s -> s.length() > 10 ? s.substring(0, 10) : s;

Function<String, String> cleanInput = sanitize
    .andThen(normalize)
    .andThen(truncate);

System.out.println(cleanInput.apply("  Hello World! 123  "));  // helloworld
```

### 3. Consumer<T> - Side Effects

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    
    default Consumer<T> andThen(Consumer<? super T> after);
}
```

#### أمثلة شاملة

```java
// ===== Basic Consumers =====
Consumer<String> print = System.out::println;
Consumer<String> printUpperCase = s -> System.out.println(s.toUpperCase());
Consumer<String> printWithPrefix = s -> System.out.println(">>> " + s);
Consumer<Object> logger = obj -> System.out.println("[LOG] " + obj);

// Using
print.accept("Hello");           // Hello
printUpperCase.accept("hello");  // HELLO
printWithPrefix.accept("Hi");    // >>> Hi

// ===== Chaining Consumers =====
Consumer<String> printAll = print
    .andThen(printUpperCase)
    .andThen(printWithPrefix);

printAll.accept("test");
// test
// TEST
// >>> test

// ===== Modifying Objects =====
Consumer<List<String>> addGreeting = list -> list.add("Hello");
Consumer<List<String>> addFarewell = list -> list.add("Goodbye");
Consumer<List<String>> printList = list -> System.out.println(list);

Consumer<List<String>> initializeList = addGreeting
    .andThen(addFarewell)
    .andThen(printList);

List<String> messages = new ArrayList<>();
initializeList.accept(messages);
// [Hello, Goodbye]

// ===== Consumer for Configuration =====
class Config {
    String host = "localhost";
    int port = 8080;
    boolean debug = false;
    
    @Override
    public String toString() {
        return "Config{host='" + host + "', port=" + port + ", debug=" + debug + "}";
    }
}

Consumer<Config> setProductionHost = c -> c.host = "prod.server.com";
Consumer<Config> setProductionPort = c -> c.port = 443;
Consumer<Config> disableDebug = c -> c.debug = false;

Consumer<Config> configureForProduction = setProductionHost
    .andThen(setProductionPort)
    .andThen(disableDebug);

Config config = new Config();
configureForProduction.accept(config);
System.out.println(config);  // Config{host='prod.server.com', port=443, debug=false}

// ===== استخدام مع Collections =====
List<String> names = Arrays.asList("أحمد", "محمد", "علي");

// forEach
names.forEach(print);
// أحمد
// محمد
// علي

names.forEach(name -> System.out.println("مرحباً " + name));
// مرحباً أحمد
// مرحباً محمد
// مرحباً علي

// ===== Consumer with Streams =====
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

Consumer<Integer> printSquare = n -> System.out.println(n + "² = " + (n * n));

numbers.stream()
    .filter(n -> n % 2 == 0)
    .forEach(printSquare);
// 2² = 4
// 4² = 16

// ===== Peek for debugging =====
List<String> result = names.stream()
    .peek(name -> System.out.println("Before: " + name))
    .map(String::toUpperCase)
    .peek(name -> System.out.println("After: " + name))
    .collect(Collectors.toList());
```

### 4. Supplier<T> - Generation/Factory

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

#### أمثلة شاملة

```java
// ===== Basic Suppliers =====
Supplier<String> helloSupplier = () -> "Hello";
Supplier<Double> randomSupplier = Math::random;
Supplier<LocalDateTime> nowSupplier = LocalDateTime::now;
Supplier<UUID> uuidSupplier = UUID::randomUUID;

System.out.println(helloSupplier.get());   // Hello
System.out.println(randomSupplier.get());  // 0.xxxxx
System.out.println(nowSupplier.get());     // 2024-01-15T10:30:00
System.out.println(uuidSupplier.get());    // a1b2c3d4-...

// ===== Factory Suppliers =====
Supplier<List<String>> listFactory = ArrayList::new;
Supplier<Map<String, Integer>> mapFactory = HashMap::new;
Supplier<StringBuilder> sbFactory = StringBuilder::new;
Supplier<int[]> arrayFactory = () -> new int[10];

List<String> newList = listFactory.get();
Map<String, Integer> newMap = mapFactory.get();

// ===== Lazy Evaluation =====
Supplier<ExpensiveObject> lazyExpensive = () -> {
    System.out.println("Creating expensive object...");
    return new ExpensiveObject();
};

System.out.println("Before get()");
// Nothing printed yet - lazy!
ExpensiveObject obj = lazyExpensive.get();  // Now prints and creates
System.out.println("After get()");

// ===== With Optional =====
Optional<String> optional = Optional.empty();

// orElse - ALWAYS evaluates (even if optional has value)
// String result1 = optional.orElse(expensiveOperation());

// orElseGet - Only evaluates when needed (lazy)
String result2 = optional.orElseGet(() -> "Default");
System.out.println(result2);  // Default

// orElseThrow with Supplier
// String result3 = optional.orElseThrow(() -> new RuntimeException("Not found"));

// ===== Memoized Supplier =====
class MemoizedSupplier<T> implements Supplier<T> {
    private Supplier<T> delegate;
    private T value;
    private boolean initialized = false;
    
    public MemoizedSupplier(Supplier<T> delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public synchronized T get() {
        if (!initialized) {
            value = delegate.get();
            initialized = true;
        }
        return value;
    }
}

Supplier<Double> memoizedRandom = new MemoizedSupplier<>(Math::random);
System.out.println(memoizedRandom.get());  // 0.12345
System.out.println(memoizedRandom.get());  // 0.12345 (same value!)
System.out.println(memoizedRandom.get());  // 0.12345 (same value!)

// ===== Supplier in Stream.generate =====
Supplier<Integer> incrementing = new Supplier<>() {
    private int counter = 0;
    @Override
    public Integer get() {
        return counter++;
    }
};

List<Integer> sequence = Stream.generate(incrementing)
    .limit(5)
    .collect(Collectors.toList());
System.out.println(sequence);  // [0, 1, 2, 3, 4]

// Random numbers
List<Double> randoms = Stream.generate(Math::random)
    .limit(5)
    .collect(Collectors.toList());
System.out.println(randoms);  // [0.xxx, 0.xxx, 0.xxx, 0.xxx, 0.xxx]

// ===== Configuration Supplier =====
Supplier<Connection> connectionSupplier = () -> {
    try {
        return DriverManager.getConnection("jdbc:mysql://localhost/db");
    } catch (SQLException e) {
        throw new RuntimeException(e);
    }
};

// Use when needed
// Connection conn = connectionSupplier.get();
```

---

## الـ Bi-Variants

### BiPredicate<T, U>

```java
@FunctionalInterface
public interface BiPredicate<T, U> {
    boolean test(T t, U u);
    
    default BiPredicate<T, U> and(BiPredicate<? super T, ? super U> other);
    default BiPredicate<T, U> or(BiPredicate<? super T, ? super U> other);
    default BiPredicate<T, U> negate();
}

// أمثلة
BiPredicate<String, Integer> hasLength = (s, len) -> s.length() == len;
BiPredicate<String, String> startsWith = String::startsWith;
BiPredicate<Integer, Integer> isGreater = (a, b) -> a > b;
BiPredicate<String, String> equalsIgnoreCase = String::equalsIgnoreCase;

System.out.println(hasLength.test("Hello", 5));       // true
System.out.println(startsWith.test("Hello", "He"));   // true
System.out.println(isGreater.test(10, 5));            // true
System.out.println(equalsIgnoreCase.test("Hi", "HI")); // true

// Combining
BiPredicate<String, Integer> hasMinLength = (s, min) -> s.length() >= min;
BiPredicate<String, Integer> hasMaxLength = (s, max) -> s.length() <= max;

// Can't directly combine these two because they have same signature
// but different semantics

// Complex BiPredicate
BiPredicate<Person, Integer> isOlderThan = (p, age) -> p.getAge() > age;
BiPredicate<Person, String> livesIn = (p, city) -> p.getCity().equals(city);

List<Person> people = getPeople();
people.stream()
    .filter(p -> isOlderThan.test(p, 25) && livesIn.test(p, "Cairo"))
    .forEach(System.out::println);
```

### BiFunction<T, U, R>

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
    
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after);
}

// أمثلة
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
BiFunction<Integer, Integer, Integer> multiply = (a, b) -> a * b;
BiFunction<String, String, String> concat = String::concat;
BiFunction<String, Integer, String> repeat = String::repeat;
BiFunction<String, Integer, Character> charAt = String::charAt;

System.out.println(add.apply(5, 3));              // 8
System.out.println(multiply.apply(5, 3));         // 15
System.out.println(concat.apply("Hello", " World")); // Hello World
System.out.println(repeat.apply("Ha", 3));        // HaHaHa
System.out.println(charAt.apply("Hello", 1));     // e

// andThen
BiFunction<Integer, Integer, Integer> addThenDouble = add.andThen(n -> n * 2);
System.out.println(addThenDouble.apply(5, 3));  // (5 + 3) * 2 = 16

// With Map
Map<String, Integer> scores = new HashMap<>();
scores.put("أحمد", 85);
scores.put("محمد", 90);

// compute uses BiFunction<Key, OldValue, NewValue>
scores.compute("أحمد", (name, score) -> score + 10);
System.out.println(scores.get("أحمد"));  // 95

// merge uses BiFunction<OldValue, NewValue, MergedValue>
scores.merge("علي", 80, (old, new_) -> old + new_);
System.out.println(scores.get("علي"));  // 80 (new entry)

scores.merge("أحمد", 5, Integer::sum);
System.out.println(scores.get("أحمد"));  // 100

// replaceAll uses BiFunction
Map<String, String> map = new HashMap<>();
map.put("a", "apple");
map.put("b", "banana");
map.replaceAll((key, value) -> value.toUpperCase());
System.out.println(map);  // {a=APPLE, b=BANANA}
```

### BiConsumer<T, U>

```java
@FunctionalInterface
public interface BiConsumer<T, U> {
    void accept(T t, U u);
    
    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after);
}

// أمثلة
BiConsumer<String, Integer> printNameAge = (name, age) -> 
    System.out.println(name + " is " + age + " years old");
BiConsumer<Map<String, Integer>, String> addToMap = (map, key) -> 
    map.put(key, 0);
BiConsumer<List<String>, String> addToList = List::add;

printNameAge.accept("أحمد", 25);  // أحمد is 25 years old

// Chaining
BiConsumer<String, Integer> printAndLog = printNameAge
    .andThen((name, age) -> System.out.println("[LOG] Printed: " + name));

printAndLog.accept("محمد", 30);
// محمد is 30 years old
// [LOG] Printed: محمد

// With Map.forEach
Map<String, Integer> ages = Map.of("أحمد", 25, "محمد", 30, "علي", 28);
ages.forEach((name, age) -> System.out.println(name + ": " + age));
// أحمد: 25
// محمد: 30
// علي: 28

// Building objects
BiConsumer<Person, String> setName = (p, name) -> p.setName(name);
BiConsumer<Person, Integer> setAge = (p, age) -> p.setAge(age);

BiConsumer<Person, String> setNameAndPrint = setName
    .andThen((p, name) -> System.out.println("Set name to: " + name));
```

---

## الـ Operators

### UnaryOperator<T>

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    static <T> UnaryOperator<T> identity();
}

// هو نفسه Function<T, T>

// أمثلة
UnaryOperator<Integer> square = n -> n * n;
UnaryOperator<Integer> negate = n -> -n;
UnaryOperator<Integer> increment = n -> n + 1;
UnaryOperator<String> toUpper = String::toUpperCase;
UnaryOperator<String> trim = String::trim;
UnaryOperator<String> reverse = s -> new StringBuilder(s).reverse().toString();

System.out.println(square.apply(5));       // 25
System.out.println(negate.apply(5));       // -5
System.out.println(toUpper.apply("hello")); // HELLO
System.out.println(reverse.apply("hello")); // olleh

// Chaining (inherited from Function)
UnaryOperator<String> process = trim
    .andThen(toUpper)
    .andThen(reverse);
System.out.println(process.apply("  hello  "));  // OLLEH

// identity
UnaryOperator<String> identity = UnaryOperator.identity();
System.out.println(identity.apply("Hello"));  // Hello

// With List.replaceAll
List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
numbers.replaceAll(square);
System.out.println(numbers);  // [1, 4, 9, 16, 25]

List<String> words = new ArrayList<>(Arrays.asList("hello", "world"));
words.replaceAll(toUpper);
System.out.println(words);  // [HELLO, WORLD]

// With Stream.iterate
List<Integer> powers = Stream.iterate(1, n -> n * 2)
    .limit(10)
    .collect(Collectors.toList());
System.out.println(powers);  // [1, 2, 4, 8, 16, 32, 64, 128, 256, 512]
```

### BinaryOperator<T>

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator);
    static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator);
}

// هو نفسه BiFunction<T, T, T>

// أمثلة
BinaryOperator<Integer> add = (a, b) -> a + b;
BinaryOperator<Integer> multiply = (a, b) -> a * b;
BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
BinaryOperator<Integer> min = (a, b) -> a < b ? a : b;
BinaryOperator<String> concat = String::concat;
BinaryOperator<String> longer = (a, b) -> a.length() >= b.length() ? a : b;

System.out.println(add.apply(5, 3));       // 8
System.out.println(multiply.apply(5, 3));  // 15
System.out.println(max.apply(5, 3));       // 5
System.out.println(concat.apply("Hello", " World"));  // Hello World
System.out.println(longer.apply("Hi", "Hello"));      // Hello

// minBy and maxBy
BinaryOperator<String> shortestString = BinaryOperator.minBy(
    Comparator.comparingInt(String::length)
);
BinaryOperator<String> longestString = BinaryOperator.maxBy(
    Comparator.comparingInt(String::length)
);

System.out.println(shortestString.apply("Hi", "Hello"));   // Hi
System.out.println(longestString.apply("Hi", "Hello"));    // Hello

// With Stream.reduce
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

int sum = numbers.stream().reduce(0, add);
System.out.println(sum);  // 15

int product = numbers.stream().reduce(1, multiply);
System.out.println(product);  // 120

int maximum = numbers.stream().reduce(Integer.MIN_VALUE, max);
System.out.println(maximum);  // 5

int minimum = numbers.stream().reduce(Integer.MAX_VALUE, min);
System.out.println(minimum);  // 1

// Or use built-in
int max2 = numbers.stream().reduce(Integer::max).orElse(0);
int min2 = numbers.stream().reduce(Integer::min).orElse(0);

// String concatenation
List<String> words = Arrays.asList("Hello", " ", "World", "!");
String sentence = words.stream().reduce("", concat);
System.out.println(sentence);  // Hello World!

// Find longest word
List<String> wordList = Arrays.asList("apple", "pie", "banana", "kiwi");
String longest = wordList.stream().reduce(longestString).orElse("");
System.out.println(longest);  // banana

// Using minBy/maxBy with reduce
String shortest = wordList.stream()
    .reduce(BinaryOperator.minBy(Comparator.comparingInt(String::length)))
    .orElse("");
System.out.println(shortest);  // pie
```

---

## Primitive Specializations

### ليه Primitive Specializations؟

```java
// ❌ Generic version - Boxing overhead
Function<Integer, Integer> square1 = n -> n * n;
int result1 = square1.apply(5);  // int → Integer → Integer → int

// ✅ Primitive version - No boxing
IntUnaryOperator square2 = n -> n * n;
int result2 = square2.applyAsInt(5);  // int → int (direct)
```

### Int Specializations

```java
// IntPredicate
IntPredicate isEven = n -> n % 2 == 0;
IntPredicate isPositive = n -> n > 0;
System.out.println(isEven.test(4));      // true
System.out.println(isPositive.test(-1)); // false

// IntFunction<R> - int → R
IntFunction<String> intToString = n -> "Number: " + n;
IntFunction<int[]> createArray = int[]::new;
System.out.println(intToString.apply(42));  // Number: 42

// ToIntFunction<T> - T → int
ToIntFunction<String> stringLength = String::length;
ToIntFunction<String> parseInt = Integer::parseInt;
System.out.println(stringLength.applyAsInt("Hello"));  // 5

// IntConsumer
IntConsumer printInt = n -> System.out.println("Value: " + n);
printInt.accept(42);  // Value: 42

// IntSupplier
IntSupplier randomInt = () -> (int)(Math.random() * 100);
System.out.println(randomInt.getAsInt());  // Random 0-99

// IntUnaryOperator - int → int
IntUnaryOperator square = n -> n * n;
IntUnaryOperator increment = n -> n + 1;
System.out.println(square.applyAsInt(5));      // 25
System.out.println(increment.applyAsInt(5));   // 6

// IntBinaryOperator - (int, int) → int
IntBinaryOperator add = (a, b) -> a + b;
IntBinaryOperator max = Integer::max;
System.out.println(add.applyAsInt(5, 3));  // 8
System.out.println(max.applyAsInt(5, 3));  // 5

// ObjIntConsumer<T> - (T, int) → void
ObjIntConsumer<String> printWithIndex = (s, i) -> 
    System.out.println(i + ": " + s);
printWithIndex.accept("Hello", 0);  // 0: Hello

// IntStream usage
int sum = IntStream.of(1, 2, 3, 4, 5)
    .filter(isEven)
    .map(square)
    .sum();
System.out.println(sum);  // 4 + 16 = 20
```

### Long Specializations

```java
// LongPredicate
LongPredicate isLarge = n -> n > 1_000_000L;

// LongFunction<R>
LongFunction<String> longToString = n -> "Long: " + n;

// ToLongFunction<T>
ToLongFunction<String> parseLong = Long::parseLong;

// LongConsumer
LongConsumer printLong = System.out::println;

// LongSupplier
LongSupplier currentTime = System::currentTimeMillis;

// LongUnaryOperator
LongUnaryOperator doubleLong = n -> n * 2;

// LongBinaryOperator
LongBinaryOperator addLongs = Long::sum;
```

### Double Specializations

```java
// DoublePredicate
DoublePredicate isPositive = d -> d > 0;

// DoubleFunction<R>
DoubleFunction<String> format = d -> String.format("%.2f", d);

// ToDoubleFunction<T>
ToDoubleFunction<String> parseDouble = Double::parseDouble;

// DoubleConsumer
DoubleConsumer printDouble = System.out::println;

// DoubleSupplier
DoubleSupplier random = Math::random;

// DoubleUnaryOperator
DoubleUnaryOperator sqrt = Math::sqrt;
DoubleUnaryOperator square = d -> d * d;

// DoubleBinaryOperator
DoubleBinaryOperator power = Math::pow;
DoubleBinaryOperator addDoubles = Double::sum;

// Usage
System.out.println(sqrt.applyAsDouble(16));       // 4.0
System.out.println(power.applyAsDouble(2, 10));   // 1024.0

DoubleStream.of(1, 4, 9, 16)
    .map(sqrt)
    .forEach(printDouble);
// 1.0, 2.0, 3.0, 4.0
```

### Conversion Functions

```java
// int → long
IntToLongFunction intToLong = n -> (long) n;

// int → double
IntToDoubleFunction intToDouble = n -> (double) n;

// long → int
LongToIntFunction longToInt = n -> (int) n;

// long → double
LongToDoubleFunction longToDouble = n -> (double) n;

// double → int
DoubleToIntFunction doubleToInt = d -> (int) d;

// double → long
DoubleToLongFunction doubleToLong = d -> (long) d;

// Usage in streams
IntStream.of(1, 2, 3)
    .mapToDouble(intToDouble)
    .forEach(System.out::println);
```

### جدول شامل للـ Primitive Specializations

| Base Interface | int | long | double |
|----------------|-----|------|--------|
| `Predicate<T>` | `IntPredicate` | `LongPredicate` | `DoublePredicate` |
| `Function<T,R>` | `IntFunction<R>` | `LongFunction<R>` | `DoubleFunction<R>` |
| `Consumer<T>` | `IntConsumer` | `LongConsumer` | `DoubleConsumer` |
| `Supplier<T>` | `IntSupplier` | `LongSupplier` | `DoubleSupplier` |
| `UnaryOperator<T>` | `IntUnaryOperator` | `LongUnaryOperator` | `DoubleUnaryOperator` |
| `BinaryOperator<T>` | `IntBinaryOperator` | `LongBinaryOperator` | `DoubleBinaryOperator` |
| `To_Function<T>` | `ToIntFunction<T>` | `ToLongFunction<T>` | `ToDoubleFunction<T>` |

---

# الجزء الثالث: أمثلة عملية متكاملة

## 1. Data Processing Pipeline

```java
class Employee {
    String name;
    String department;
    double salary;
    int yearsOfService;
    
    // Constructor, getters...
}

// Define reusable functions
Predicate<Employee> isInIT = e -> "IT".equals(e.getDepartment());
Predicate<Employee> earnsMoreThan50k = e -> e.getSalary() > 50000;
Predicate<Employee> seniorEmployee = e -> e.getYearsOfService() > 5;

Function<Employee, String> getName = Employee::getName;
Function<Employee, Double> getSalary = Employee::getSalary;
ToDoubleFunction<Employee> getSalaryPrimitive = Employee::getSalary;

Consumer<Employee> printEmployee = e -> 
    System.out.println(e.getName() + ": $" + e.getSalary());

// Use in pipeline
List<Employee> employees = getEmployees();

// Get names of senior IT employees earning > 50k
List<String> seniorITHighEarners = employees.stream()
    .filter(isInIT)
    .filter(earnsMoreThan50k)
    .filter(seniorEmployee)
    .map(getName)
    .collect(Collectors.toList());

// Calculate average salary of IT department
double avgITSalary = employees.stream()
    .filter(isInIT)
    .mapToDouble(getSalaryPrimitive)
    .average()
    .orElse(0);

// Print all high earners
employees.stream()
    .filter(earnsMoreThan50k)
    .forEach(printEmployee);

// Group by department and calculate stats
Map<String, DoubleSummaryStatistics> salaryStatsByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.summarizingDouble(Employee::getSalary)
    ));
```

## 2. Validation Framework

```java
@FunctionalInterface
interface Validator<T> {
    ValidationResult validate(T value);
    
    default Validator<T> and(Validator<T> other) {
        return value -> {
            ValidationResult result = this.validate(value);
            if (!result.isValid()) return result;
            return other.validate(value);
        };
    }
    
    static <T> Validator<T> from(Predicate<T> predicate, String errorMessage) {
        return value -> predicate.test(value)
            ? ValidationResult.valid()
            : ValidationResult.invalid(errorMessage);
    }
}

record ValidationResult(boolean isValid, String errorMessage) {
    static ValidationResult valid() { return new ValidationResult(true, ""); }
    static ValidationResult invalid(String msg) { return new ValidationResult(false, msg); }
}

// Define validators using Predicates
Predicate<String> notNull = Objects::nonNull;
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> minLength8 = s -> s.length() >= 8;
Predicate<String> hasUppercase = s -> s.matches(".*[A-Z].*");
Predicate<String> hasLowercase = s -> s.matches(".*[a-z].*");
Predicate<String> hasDigit = s -> s.matches(".*[0-9].*");

// Build password validator
Validator<String> passwordValidator = Validator.from(notNull, "Password required")
    .and(Validator.from(notEmpty, "Password cannot be empty"))
    .and(Validator.from(minLength8, "Password must be at least 8 characters"))
    .and(Validator.from(hasUppercase, "Password must contain uppercase"))
    .and(Validator.from(hasLowercase, "Password must contain lowercase"))
    .and(Validator.from(hasDigit, "Password must contain digit"));

// Usage
ValidationResult result = passwordValidator.validate("Pass123!");
System.out.println(result.isValid());  // true

result = passwordValidator.validate("weak");
System.out.println(result.errorMessage());  // Password must be at least 8 characters
```

## 3. Event-Driven System

```java
class EventBus {
    private final Map<Class<?>, List<Consumer<Object>>> handlers = new HashMap<>();
    
    public <T> void subscribe(Class<T> eventType, Consumer<T> handler) {
        handlers.computeIfAbsent(eventType, k -> new ArrayList<>())
            .add(event -> handler.accept((T) event));
    }
    
    public <T> void subscribe(Class<T> eventType, Predicate<T> filter, Consumer<T> handler) {
        subscribe(eventType, event -> {
            if (filter.test(event)) {
                handler.accept(event);
            }
        });
    }
    
    public void publish(Object event) {
        List<Consumer<Object>> eventHandlers = handlers.get(event.getClass());
        if (eventHandlers != null) {
            eventHandlers.forEach(handler -> handler.accept(event));
        }
    }
}

// Events
record UserCreated(String username, String email) {}
record OrderPlaced(String orderId, double amount) {}
record PaymentReceived(String orderId, double amount) {}

// Usage
EventBus bus = new EventBus();

// Subscribe with Consumer
bus.subscribe(UserCreated.class, event -> 
    System.out.println("Welcome email sent to: " + event.email()));

// Subscribe with filter (Predicate) and handler (Consumer)
bus.subscribe(OrderPlaced.class,
    order -> order.amount() > 100,  // Only high-value orders
    order -> System.out.println("High-value order notification: " + order.orderId()));

// Publish events
bus.publish(new UserCreated("ahmed", "ahmed@email.com"));
bus.publish(new OrderPlaced("ORD-001", 150));  // Will trigger
bus.publish(new OrderPlaced("ORD-002", 50));   // Won't trigger
```

## 4. Retry Mechanism

```java
class RetryUtil {
    
    public static <T> T retry(
            Supplier<T> operation,
            int maxRetries,
            Predicate<Exception> retryOn,
            Consumer<Integer> onRetry) {
        
        Exception lastException = null;
        
        for (int attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                return operation.get();
            } catch (Exception e) {
                lastException = e;
                if (attempt < maxRetries && retryOn.test(e)) {
                    onRetry.accept(attempt);
                } else if (!retryOn.test(e)) {
                    break;
                }
            }
        }
        
        throw new RuntimeException("Failed after " + maxRetries + " attempts", lastException);
    }
}

// Usage
Supplier<String> unreliableOperation = () -> {
    if (Math.random() < 0.7) {
        throw new IOException("Network error");
    }
    return "Success!";
};

Predicate<Exception> retryOnIO = e -> e instanceof IOException;
Consumer<Integer> logRetry = attempt -> 
    System.out.println("Retry attempt " + attempt);

String result = RetryUtil.retry(unreliableOperation, 5, retryOnIO, logRetry);
```

---

# أسئلة الانترفيو

## أسئلة نظرية

### س1: إيه أنواع Method References الأربعة؟

**الإجابة:**

| النوع | Syntax | Lambda Equivalent | مثال |
|-------|--------|-------------------|------|
| Static | `Class::staticMethod` | `x -> Class.staticMethod(x)` | `Integer::parseInt` |
| Bound Instance | `instance::method` | `x -> instance.method(x)` | `str::length` |
| Unbound Instance | `Class::instanceMethod` | `x -> x.instanceMethod()` | `String::toUpperCase` |
| Constructor | `Class::new` | `x -> new Class(x)` | `ArrayList::new` |

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

### س2: إيه الفرق بين Bound و Unbound Instance Method Reference؟

**الإجابة:**

```java
// BOUND - الـ instance معروف ومحدد
String fixed = "Hello";
Supplier<Integer> bound = fixed::length;  // No parameter needed
// equivalent to: () -> fixed.length()
System.out.println(bound.get());  // 5

// UNBOUND - الـ instance هو أول parameter
Function<String, Integer> unbound = String::length;  // Takes String parameter
// equivalent to: (s) -> s.length()
System.out.println(unbound.apply("Hi"));  // 2
```

**الفرق الرئيسي:**
- **Bound**: الـ instance ثابت، الـ functional interface مش بياخد parameter للـ instance
- **Unbound**: الـ instance هو أول parameter في الـ functional interface

---

### س3: إيه الفرق بين Function و UnaryOperator؟

**الإجابة:**

| `Function<T, R>` | `UnaryOperator<T>` |
|------------------|---------------------|
| Input: T, Output: R | Input: T, Output: T |
| ممكن أنواع مختلفة | لازم نفس النوع |
| `Function<String, Integer>` | `UnaryOperator<String>` |

```java
// Function - أنواع مختلفة ممكنة
Function<String, Integer> length = String::length;  // String → Integer

// UnaryOperator - نفس النوع
UnaryOperator<String> toUpper = String::toUpperCase;  // String → String

// UnaryOperator extends Function<T, T>
// فتقدر تستخدمه في أي مكان يتوقع Function<T, T>
```

---

### س4: إيه الفرق بين `andThen` و `compose` في Function؟

**الإجابة:**

```java
Function<Integer, Integer> addOne = x -> x + 1;
Function<Integer, Integer> multiplyTwo = x -> x * 2;

// andThen: this FIRST, then other
// 5 → (5+1) → (6*2) → 12
Function<Integer, Integer> f1 = addOne.andThen(multiplyTwo);
System.out.println(f1.apply(5));  // 12

// compose: other FIRST, then this
// 5 → (5*2) → (10+1) → 11
Function<Integer, Integer> f2 = addOne.compose(multiplyTwo);
System.out.println(f2.apply(5));  // 11
```

**ملخص:**
- `f1.andThen(f2)` = `f2(f1(x))` - تنفيذ f1 الأول
- `f1.compose(f2)` = `f1(f2(x))` - تنفيذ f2 الأول

---

### س5: إمتى تستخدم Predicate.not() بدل negate()؟

**الإجابة:**

```java
// negate() - instance method
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty1 = isEmpty.negate();

// Predicate.not() - static method (Java 11+)
// أفضل مع Method References
Predicate<String> isNotEmpty2 = Predicate.not(String::isEmpty);

// الفرق في الـ readability
list.stream().filter(isEmpty.negate());           // أقل وضوحاً
list.stream().filter(Predicate.not(String::isEmpty)); // أوضح

// Predicate.not() مفيد مباشرة في filter
list.stream()
    .filter(Predicate.not(String::isEmpty))
    .collect(Collectors.toList());
```

---

### س6: ليه نستخدم IntPredicate بدل Predicate<Integer>؟

**الإجابة:**
عشان نتجنب **Boxing/Unboxing overhead**:

```java
// ❌ Predicate<Integer> - فيه Boxing
Predicate<Integer> isEven1 = n -> n % 2 == 0;
boolean result1 = isEven1.test(4);  // 4 → Integer → test → unbox

// ✅ IntPredicate - مفيش Boxing
IntPredicate isEven2 = n -> n % 2 == 0;
boolean result2 = isEven2.test(4);  // direct int operation

// Performance difference في loops
IntStream.range(0, 1_000_000)
    .filter(isEven2)  // Fast - no boxing
    .sum();
```

---

### س7: إيه استخدامات Supplier الشائعة؟

**الإجابة:**

```java
// 1. Lazy evaluation
Optional<String> opt = Optional.empty();
String result = opt.orElseGet(() -> expensiveOperation());  // Lazy!

// 2. Factory pattern
Supplier<List<String>> listFactory = ArrayList::new;
List<String> newList = listFactory.get();

// 3. Deferred execution / Configuration
Supplier<Connection> connSupplier = () -> createConnection();
// Connection created only when get() is called

// 4. Stream generation
Stream.generate(Math::random)
    .limit(10)
    .forEach(System.out::println);

// 5. Default values
public String getValue(Supplier<String> defaultSupplier) {
    return value != null ? value : defaultSupplier.get();
}
```

---

### س8: إيه الفرق بين forEach و forEachOrdered في Streams؟

**الإجابة:**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// forEach - مش guaranteed order في parallel streams
numbers.parallelStream().forEach(System.out::println);
// Output: 3, 1, 5, 2, 4 (random order)

// forEachOrdered - guaranteed order
numbers.parallelStream().forEachOrdered(System.out::println);
// Output: 1, 2, 3, 4, 5 (always in order)

// في sequential streams الاتنين نفس الحاجة
numbers.stream().forEach(System.out::println);  // 1, 2, 3, 4, 5
```

---

## أسئلة كود

### س9: حول الـ Lambdas دي لـ Method References

```java
// 1. Lambda
Function<String, Integer> f1 = s -> Integer.parseInt(s);
// Method Reference
Function<String, Integer> f1_ref = Integer::parseInt;

// 2. Lambda
Consumer<String> c1 = s -> System.out.println(s);
// Method Reference
Consumer<String> c1_ref = System.out::println;

// 3. Lambda
Function<String, String> f2 = s -> s.toUpperCase();
// Method Reference
Function<String, String> f2_ref = String::toUpperCase;

// 4. Lambda
Supplier<List<String>> s1 = () -> new ArrayList<>();
// Method Reference
Supplier<List<String>> s1_ref = ArrayList::new;

// 5. Lambda
BiFunction<String, String, Boolean> bf1 = (s1, s2) -> s1.equals(s2);
// Method Reference
BiFunction<String, String, Boolean> bf1_ref = String::equals;

// 6. Lambda
Comparator<String> comp1 = (s1, s2) -> s1.compareTo(s2);
// Method Reference
Comparator<String> comp1_ref = String::compareTo;

// 7. Lambda (with specific instance)
String prefix = "Hello ";
Function<String, String> f3 = s -> prefix.concat(s);
// Method Reference (Bound)
Function<String, String> f3_ref = prefix::concat;

// 8. Lambda
ToIntFunction<String> tif1 = s -> s.length();
// Method Reference
ToIntFunction<String> tif1_ref = String::length;
```

---

### س10: اكتب Pipeline لمعالجة بيانات Employees

```java
class Employee {
    String name;
    String department;
    double salary;
    
    // constructor, getters...
}

// Define functional components
Predicate<Employee> isInEngineering = e -> "Engineering".equals(e.getDepartment());
Predicate<Employee> earnsAbove60k = e -> e.getSalary() > 60000;
Function<Employee, String> toName = Employee::getName;
ToDoubleFunction<Employee> toSalary = Employee::getSalary;
Consumer<String> printName = name -> System.out.println("- " + name);
Comparator<Employee> bySalaryDesc = Comparator.comparingDouble(Employee::getSalary).reversed();

List<Employee> employees = getEmployees();

// 1. Get names of high-earning engineers
System.out.println("High-earning Engineers:");
employees.stream()
    .filter(isInEngineering.and(earnsAbove60k))
    .map(toName)
    .forEach(printName);

// 2. Calculate average salary in Engineering
double avgSalary = employees.stream()
    .filter(isInEngineering)
    .mapToDouble(toSalary)
    .average()
    .orElse(0);
System.out.println("Avg Engineering Salary: $" + avgSalary);

// 3. Top 3 highest paid employees
System.out.println("Top 3 Earners:");
employees.stream()
    .sorted(bySalaryDesc)
    .limit(3)
    .map(toName)
    .forEach(printName);

// 4. Group by department and count
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));
System.out.println("Count by Dept: " + countByDept);

// 5. Department with highest average salary
Optional<Map.Entry<String, Double>> topDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ))
    .entrySet()
    .stream()
    .max(Map.Entry.comparingByValue());

topDept.ifPresent(entry -> 
    System.out.println("Top Dept: " + entry.getKey() + " ($" + entry.getValue() + ")"));
```

---

### س11: اكتب Comparator chain باستخدام Method References

```java
class Person {
    String firstName;
    String lastName;
    int age;
    String city;
    
    // constructor, getters...
}

List<Person> people = getPeople();

// 1. Sort by lastName
people.sort(Comparator.comparing(Person::getLastName));

// 2. Sort by lastName, then firstName
people.sort(Comparator
    .comparing(Person::getLastName)
    .thenComparing(Person::getFirstName));

// 3. Sort by age (youngest first)
people.sort(Comparator.comparingInt(Person::getAge));

// 4. Sort by age (oldest first)
people.sort(Comparator.comparingInt(Person::getAge).reversed());

// 5. Complex: city, then age desc, then name
people.sort(Comparator
    .comparing(Person::getCity)
    .thenComparing(Person::getAge, Comparator.reverseOrder())
    .thenComparing(Person::getLastName)
    .thenComparing(Person::getFirstName));

// 6. Null-safe sorting
people.sort(Comparator.comparing(
    Person::getCity,
    Comparator.nullsLast(Comparator.naturalOrder())
));

// 7. Case-insensitive
people.sort(Comparator.comparing(
    Person::getLastName,
    String.CASE_INSENSITIVE_ORDER
));
```

---

### س12: اكتب Stream operations باستخدام Built-in Functional Interfaces

```java
List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");

// 1. Filter using Predicate
Predicate<String> longerThan5 = s -> s.length() > 5;
List<String> longWords = words.stream()
    .filter(longerThan5)
    .collect(Collectors.toList());
// [banana, cherry, elderberry]

// 2. Transform using Function
Function<String, String> capitalize = s -> 
    s.substring(0, 1).toUpperCase() + s.substring(1);
List<String> capitalized = words.stream()
    .map(capitalize)
    .collect(Collectors.toList());
// [Apple, Banana, Cherry, Date, Elderberry]

// 3. Reduce using BinaryOperator
BinaryOperator<String> longest = (a, b) -> a.length() >= b.length() ? a : b;
String longestWord = words.stream()
    .reduce(longest)
    .orElse("");
// elderberry

// 4. Collect to Map using Function
Function<String, Integer> wordLength = String::length;
Map<String, Integer> wordLengths = words.stream()
    .collect(Collectors.toMap(
        Function.identity(),
        wordLength
    ));
// {apple=5, banana=6, cherry=6, date=4, elderberry=10}

// 5. Group using Function
Function<String, Integer> firstCharCode = s -> (int) s.charAt(0);
Map<Integer, List<String>> groupedByFirstChar = words.stream()
    .collect(Collectors.groupingBy(firstCharCode));

// 6. Statistics using ToIntFunction
ToIntFunction<String> toLength = String::length;
IntSummaryStatistics stats = words.stream()
    .mapToInt(toLength)
    .summaryStatistics();
System.out.println("Avg length: " + stats.getAverage());
System.out.println("Max length: " + stats.getMax());

// 7. Generate using Supplier
Supplier<Double> randomSupplier = Math::random;
List<Double> randoms = Stream.generate(randomSupplier)
    .limit(5)
    .collect(Collectors.toList());

// 8. Iterate using UnaryOperator
UnaryOperator<Integer> doubler = n -> n * 2;
List<Integer> powers = Stream.iterate(1, doubler)
    .limit(10)
    .collect(Collectors.toList());
// [1, 2, 4, 8, 16, 32, 64, 128, 256, 512]
```

---

### س13: اكتب utility methods باستخدام Functional Interfaces

```java
public class FunctionalUtils {
    
    // Compose multiple predicates with AND
    @SafeVarargs
    public static <T> Predicate<T> allOf(Predicate<T>... predicates) {
        return Arrays.stream(predicates)
            .reduce(t -> true, Predicate::and);
    }
    
    // Compose multiple predicates with OR
    @SafeVarargs
    public static <T> Predicate<T> anyOf(Predicate<T>... predicates) {
        return Arrays.stream(predicates)
            .reduce(t -> false, Predicate::or);
    }
    
    // Chain multiple functions
    @SafeVarargs
    public static <T> UnaryOperator<T> chain(UnaryOperator<T>... operators) {
        return Arrays.stream(operators)
            .reduce(UnaryOperator.identity(), (f1, f2) -> f1.andThen(f2)::apply);
    }
    
    // Memoize a function
    public static <T, R> Function<T, R> memoize(Function<T, R> function) {
        Map<T, R> cache = new ConcurrentHashMap<>();
        return input -> cache.computeIfAbsent(input, function);
    }
    
    // Retry operation
    public static <T> T retry(Supplier<T> operation, int maxRetries) {
        Exception lastException = null;
        for (int i = 0; i < maxRetries; i++) {
            try {
                return operation.get();
            } catch (Exception e) {
                lastException = e;
            }
        }
        throw new RuntimeException("Failed after " + maxRetries + " retries", lastException);
    }
    
    // Execute with timeout (simplified)
    public static <T> Optional<T> executeWithDefault(
            Supplier<T> operation, 
            Supplier<T> defaultSupplier) {
        try {
            return Optional.ofNullable(operation.get());
        } catch (Exception e) {
            return Optional.ofNullable(defaultSupplier.get());
        }
    }
}

// Usage
Predicate<String> notNull = Objects::nonNull;
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> validLength = s -> s.length() >= 3;

Predicate<String> isValid = FunctionalUtils.allOf(notNull, notEmpty, validLength);
System.out.println(isValid.test("Hi"));    // false
System.out.println(isValid.test("Hello")); // true

UnaryOperator<String> trim = String::trim;
UnaryOperator<String> lower = String::toLowerCase;
UnaryOperator<String> process = FunctionalUtils.chain(trim, lower);
System.out.println(process.apply("  HELLO  "));  // hello
```

---

## نصائح للانترفيو 💡

1. **اعرف الأربع أنواع من Method References** واتدرب على التحويل من Lambda
2. **افهم الفرق بين Bound و Unbound** Instance Method References
3. **اعرف الـ Core Functional Interfaces** (Predicate, Function, Consumer, Supplier)
4. **افهم composition** (`and`, `or`, `andThen`, `compose`)
5. **اعرف ليه نستخدم Primitive Specializations** ومتى
6. **اتدرب على كتابة Stream pipelines** مع Method References
7. **افهم Comparator composition** كويس

---

## موارد إضافية 📚

- [Oracle Method References Tutorial](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)
- [java.util.function Package](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)
- [Java 8 in Action](https://www.manning.com/books/java-8-in-action)

---

*تم إعداد هذا الدليل بالعامية المصرية عشان يكون سهل الفهم والمراجعة* 🇪🇬
