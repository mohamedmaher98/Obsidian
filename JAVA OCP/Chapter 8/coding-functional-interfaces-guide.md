# 🔧 Coding Functional Interfaces في Java - دليل شامل بالمصري

## المحتويات
- [مقدمة عن Functional Interfaces](#مقدمة-عن-functional-interfaces)
- [إنشاء Custom Functional Interfaces](#إنشاء-custom-functional-interfaces)
- [Built-in Functional Interfaces بالتفصيل](#built-in-functional-interfaces-بالتفصيل)
- [Composing Functional Interfaces](#composing-functional-interfaces)
- [Primitive Specializations](#primitive-specializations)
- [Functional Interfaces مع Generics](#functional-interfaces-مع-generics)
- [Functional Interfaces مع Exceptions](#functional-interfaces-مع-exceptions)
- [Design Patterns مع Functional Interfaces](#design-patterns-مع-functional-interfaces)
- [Best Practices](#best-practices)
- [أسئلة الانترفيو](#أسئلة-الانترفيو)

---

# مقدمة عن Functional Interfaces

## يعني إيه Functional Interface؟

الـ **Functional Interface** هو interface فيه **Abstract Method واحدة بس** (Single Abstract Method - SAM).

ده الأساس اللي بيخلي Lambda Expressions تشتغل في Java!

```java
// ✅ Functional Interface - method واحدة abstract
@FunctionalInterface
interface Greeting {
    void sayHello(String name);
}

// استخدام مع Lambda
Greeting arabic = name -> System.out.println("أهلاً " + name);
Greeting english = name -> System.out.println("Hello " + name);

arabic.sayHello("أحمد");   // أهلاً أحمد
english.sayHello("Ahmed"); // Hello Ahmed
```

## القواعد الأساسية

```java
@FunctionalInterface
interface ValidFunctionalInterface {
    // ✅ Abstract method واحدة - ده الأساس
    void doSomething();
    
    // ✅ Default methods - مش بتتحسب
    default void helper() {
        System.out.println("Helper");
    }
    
    default void anotherHelper() {
        System.out.println("Another Helper");
    }
    
    // ✅ Static methods - مش بتتحسب
    static void utility() {
        System.out.println("Utility");
    }
    
    // ✅ Methods من Object class - مش بتتحسب
    boolean equals(Object obj);
    int hashCode();
    String toString();
}
```

## ليه @FunctionalInterface مهم؟

```java
// ❌ من غير الـ annotation - خطر!
interface Calculator {
    int calculate(int a, int b);
}

// حد تاني ممكن يضيف method تانية بالغلط
interface Calculator {
    int calculate(int a, int b);
    int anotherMethod();  // ❌ كده بقى مش Functional Interface!
}

// ✅ مع الـ annotation - حماية!
@FunctionalInterface
interface SafeCalculator {
    int calculate(int a, int b);
    // int anotherMethod();  // ❌ Compile Error!
}
```

## أمثلة على Valid و Invalid Functional Interfaces

```java
// ✅ Valid - method واحدة abstract
@FunctionalInterface
interface Runnable {
    void run();
}

// ✅ Valid - method واحدة abstract + default methods
@FunctionalInterface
interface Comparator<T> {
    int compare(T o1, T o2);
    
    default Comparator<T> reversed() {
        return (o1, o2) -> compare(o2, o1);
    }
}

// ✅ Valid - equals من Object مش بتتحسب
@FunctionalInterface
interface MyInterface {
    void doWork();
    boolean equals(Object obj);  // من Object
}

// ❌ Invalid - مفيش abstract methods
@FunctionalInterface
interface NoAbstract {
    default void method() { }
    // Compile Error: No abstract method!
}

// ❌ Invalid - أكتر من abstract method
@FunctionalInterface
interface TwoMethods {
    void method1();
    void method2();
    // Compile Error: Multiple abstract methods!
}

// ❌ Invalid - بيـ extend interface تاني بـ abstract method مختلف
interface Parent {
    void parentMethod();
}

@FunctionalInterface
interface Child extends Parent {
    void childMethod();
    // Compile Error: Has two abstract methods!
}

// ✅ Valid - بيـ extend وبيعمل override
interface Parent2 {
    void method();
}

@FunctionalInterface
interface Child2 extends Parent2 {
    @Override
    void method();  // نفس الـ method
}
```

---

# إنشاء Custom Functional Interfaces

## 1. Basic Functional Interface

```java
// بدون parameters، بدون return
@FunctionalInterface
interface Task {
    void execute();
}

// استخدام
Task printHello = () -> System.out.println("Hello!");
Task printTime = () -> System.out.println(LocalTime.now());

printHello.execute();  // Hello!
printTime.execute();   // 10:30:45.123
```

## 2. مع Parameter واحد

```java
// Parameter واحد، مفيش return
@FunctionalInterface
interface Printer<T> {
    void print(T item);
}

Printer<String> stringPrinter = s -> System.out.println("String: " + s);
Printer<Integer> intPrinter = n -> System.out.println("Number: " + n);

stringPrinter.print("Hello");  // String: Hello
intPrinter.print(42);          // Number: 42

// Parameter واحد، مع return
@FunctionalInterface
interface Transformer<T, R> {
    R transform(T input);
}

Transformer<String, Integer> lengthGetter = s -> s.length();
Transformer<Integer, String> intToString = n -> "Value: " + n;

System.out.println(lengthGetter.transform("Hello"));  // 5
System.out.println(intToString.transform(42));        // Value: 42
```

## 3. مع Multiple Parameters

```java
// 2 Parameters
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

Calculator add = (a, b) -> a + b;
Calculator subtract = (a, b) -> a - b;
Calculator multiply = (a, b) -> a * b;
Calculator divide = (a, b) -> a / b;
Calculator power = (a, b) -> (int) Math.pow(a, b);

System.out.println(add.calculate(10, 5));       // 15
System.out.println(subtract.calculate(10, 5)); // 5
System.out.println(multiply.calculate(10, 5)); // 50
System.out.println(divide.calculate(10, 5));   // 2
System.out.println(power.calculate(2, 3));     // 8

// 3 Parameters
@FunctionalInterface
interface TriFunction<A, B, C, R> {
    R apply(A a, B b, C c);
}

TriFunction<Integer, Integer, Integer, Integer> sum3 = (a, b, c) -> a + b + c;
TriFunction<String, String, String, String> concat3 = (a, b, c) -> a + b + c;

System.out.println(sum3.apply(1, 2, 3));              // 6
System.out.println(concat3.apply("A", "B", "C"));     // ABC
```

## 4. مع Default Methods

```java
@FunctionalInterface
interface Validator<T> {
    boolean validate(T value);
    
    // Default method للـ AND logic
    default Validator<T> and(Validator<T> other) {
        return value -> this.validate(value) && other.validate(value);
    }
    
    // Default method للـ OR logic
    default Validator<T> or(Validator<T> other) {
        return value -> this.validate(value) || other.validate(value);
    }
    
    // Default method للـ NOT logic
    default Validator<T> negate() {
        return value -> !this.validate(value);
    }
}

// استخدام
Validator<String> notNull = s -> s != null;
Validator<String> notEmpty = s -> !s.isEmpty();
Validator<String> minLength5 = s -> s.length() >= 5;

// Combining validators
Validator<String> validUsername = notNull
    .and(notEmpty)
    .and(minLength5);

System.out.println(validUsername.validate(null));      // false
System.out.println(validUsername.validate(""));        // false
System.out.println(validUsername.validate("abc"));     // false
System.out.println(validUsername.validate("ahmed"));   // true
```

## 5. مع Static Factory Methods

```java
@FunctionalInterface
interface Matcher<T> {
    boolean matches(T value);
    
    // Static factory methods
    static <T> Matcher<T> isNull() {
        return value -> value == null;
    }
    
    static <T> Matcher<T> notNull() {
        return value -> value != null;
    }
    
    static <T> Matcher<T> equalTo(T expected) {
        return value -> Objects.equals(value, expected);
    }
    
    static Matcher<String> startsWith(String prefix) {
        return value -> value != null && value.startsWith(prefix);
    }
    
    static Matcher<String> endsWith(String suffix) {
        return value -> value != null && value.endsWith(suffix);
    }
    
    static Matcher<String> contains(String substring) {
        return value -> value != null && value.contains(substring);
    }
    
    static <T extends Comparable<T>> Matcher<T> greaterThan(T threshold) {
        return value -> value != null && value.compareTo(threshold) > 0;
    }
    
    static <T extends Comparable<T>> Matcher<T> lessThan(T threshold) {
        return value -> value != null && value.compareTo(threshold) < 0;
    }
    
    // Default combining methods
    default Matcher<T> and(Matcher<T> other) {
        return value -> this.matches(value) && other.matches(value);
    }
    
    default Matcher<T> or(Matcher<T> other) {
        return value -> this.matches(value) || other.matches(value);
    }
    
    default Matcher<T> negate() {
        return value -> !this.matches(value);
    }
}

// استخدام
Matcher<String> emailMatcher = Matcher.<String>notNull()
    .and(Matcher.contains("@"))
    .and(Matcher.endsWith(".com").or(Matcher.endsWith(".org")));

System.out.println(emailMatcher.matches("test@example.com"));  // true
System.out.println(emailMatcher.matches("test@example.org"));  // true
System.out.println(emailMatcher.matches("test@example.net"));  // false
System.out.println(emailMatcher.matches(null));                // false

Matcher<Integer> ageMatcher = Matcher.<Integer>notNull()
    .and(Matcher.greaterThan(0))
    .and(Matcher.lessThan(150));

System.out.println(ageMatcher.matches(25));   // true
System.out.println(ageMatcher.matches(-5));   // false
System.out.println(ageMatcher.matches(200));  // false
```

## 6. مع Multiple Type Parameters

```java
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
    
    // Chaining converters
    default <V> Converter<F, V> andThen(Converter<T, V> after) {
        return from -> after.convert(this.convert(from));
    }
    
    // Static identity converter
    static <T> Converter<T, T> identity() {
        return t -> t;
    }
}

// استخدام
Converter<String, Integer> stringToInt = Integer::parseInt;
Converter<Integer, Double> intToDouble = Integer::doubleValue;
Converter<Double, String> doubleToString = d -> String.format("%.2f", d);

// Chaining
Converter<String, String> stringToFormattedDouble = stringToInt
    .andThen(intToDouble)
    .andThen(doubleToString);

System.out.println(stringToFormattedDouble.convert("42"));  // "42.00"
```

---

# Built-in Functional Interfaces بالتفصيل

## java.util.function Package

### 1. Predicate<T> - للـ Testing

```java
import java.util.function.Predicate;

// الـ interface definition
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    
    default Predicate<T> and(Predicate<? super T> other);
    default Predicate<T> or(Predicate<? super T> other);
    default Predicate<T> negate();
    static <T> Predicate<T> isEqual(Object targetRef);
    static <T> Predicate<T> not(Predicate<? super T> target);  // Java 11+
}

// أمثلة عملية
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isLessThan100 = n -> n < 100;

// Basic usage
System.out.println(isPositive.test(5));   // true
System.out.println(isPositive.test(-5));  // false

// Combining with and()
Predicate<Integer> isPositiveAndEven = isPositive.and(isEven);
System.out.println(isPositiveAndEven.test(4));   // true
System.out.println(isPositiveAndEven.test(3));   // false
System.out.println(isPositiveAndEven.test(-4));  // false

// Combining with or()
Predicate<Integer> isPositiveOrEven = isPositive.or(isEven);
System.out.println(isPositiveOrEven.test(3));    // true (positive)
System.out.println(isPositiveOrEven.test(-4));   // true (even)
System.out.println(isPositiveOrEven.test(-3));   // false

// Using negate()
Predicate<Integer> isNotPositive = isPositive.negate();
System.out.println(isNotPositive.test(-5));  // true
System.out.println(isNotPositive.test(5));   // false

// Using Predicate.not() - Java 11+
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = Predicate.not(String::isEmpty);

// Using isEqual()
Predicate<String> isHello = Predicate.isEqual("Hello");
System.out.println(isHello.test("Hello"));  // true
System.out.println(isHello.test("World"));  // false

// Complex example
Predicate<String> isValidEmail = s -> s != null 
    && !s.isEmpty() 
    && s.contains("@") 
    && s.contains(".");

List<String> emails = Arrays.asList(
    "valid@email.com",
    "invalid",
    null,
    "",
    "another@test.org"
);

List<String> validEmails = emails.stream()
    .filter(isValidEmail)
    .collect(Collectors.toList());
System.out.println(validEmails);  // [valid@email.com, another@test.org]
```

### 2. Function<T, R> - للـ Transformation

```java
import java.util.function.Function;

// الـ interface definition
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before);
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after);
    static <T> Function<T, T> identity();
}

// أمثلة عملية
Function<String, Integer> stringLength = String::length;
Function<String, String> toUpperCase = String::toUpperCase;
Function<Integer, Integer> square = n -> n * n;
Function<Integer, String> intToString = n -> "Number: " + n;

// Basic usage
System.out.println(stringLength.apply("Hello"));    // 5
System.out.println(toUpperCase.apply("hello"));     // HELLO
System.out.println(square.apply(5));                // 25
System.out.println(intToString.apply(42));          // Number: 42

// Using andThen() - تنفيذ من اليسار لليمين
Function<String, Integer> getLengthThenSquare = stringLength.andThen(square);
System.out.println(getLengthThenSquare.apply("Hello"));  // 25 (5²)

// Using compose() - تنفيذ من اليمين لليسار
Function<String, Integer> squareAfterLength = square.compose(stringLength);
System.out.println(squareAfterLength.apply("Hello"));    // 25 (5²)

// Chaining multiple functions
Function<String, String> processString = 
    ((Function<String, String>) String::trim)
    .andThen(String::toLowerCase)
    .andThen(s -> s.replace(" ", "_"));

System.out.println(processString.apply("  Hello World  "));  // hello_world

// identity()
Function<String, String> identity = Function.identity();
System.out.println(identity.apply("Hello"));  // Hello

// Practical example - Data transformation pipeline
Function<Map<String, Object>, String> extractName = 
    map -> (String) map.get("name");
Function<String, String> formatName = 
    name -> name.substring(0, 1).toUpperCase() + name.substring(1).toLowerCase();
Function<String, String> addGreeting = 
    name -> "Hello, " + name + "!";

Function<Map<String, Object>, String> greetUser = extractName
    .andThen(formatName)
    .andThen(addGreeting);

Map<String, Object> user = Map.of("name", "AHMED", "age", 25);
System.out.println(greetUser.apply(user));  // Hello, Ahmed!
```

### 3. Consumer<T> - للـ Side Effects

```java
import java.util.function.Consumer;

// الـ interface definition
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    
    default Consumer<T> andThen(Consumer<? super T> after);
}

// أمثلة عملية
Consumer<String> printer = System.out::println;
Consumer<String> logger = s -> System.out.println("[LOG] " + s);
Consumer<List<String>> listClearer = List::clear;

// Basic usage
printer.accept("Hello!");           // Hello!
logger.accept("Application started"); // [LOG] Application started

// Using andThen()
Consumer<String> printAndLog = printer.andThen(logger);
printAndLog.accept("Test");
// Output:
// Test
// [LOG] Test

// Practical example - Processing pipeline
Consumer<StringBuilder> addHeader = sb -> sb.insert(0, "=== REPORT ===\n");
Consumer<StringBuilder> addFooter = sb -> sb.append("\n=== END ===");
Consumer<StringBuilder> addTimestamp = sb -> sb.append("\nGenerated: " + LocalDateTime.now());

Consumer<StringBuilder> formatReport = addHeader
    .andThen(addFooter)
    .andThen(addTimestamp);

StringBuilder report = new StringBuilder("Sales data here");
formatReport.accept(report);
System.out.println(report);

// With collections
List<String> names = Arrays.asList("أحمد", "محمد", "علي");
names.forEach(printer);

// Modifying objects
Consumer<Person> setDefaultAge = person -> person.setAge(18);
Consumer<Person> setDefaultCity = person -> person.setCity("Cairo");
Consumer<Person> initializePerson = setDefaultAge.andThen(setDefaultCity);

Person person = new Person("أحمد");
initializePerson.accept(person);
```

### 4. Supplier<T> - للـ Generation

```java
import java.util.function.Supplier;

// الـ interface definition
@FunctionalInterface
public interface Supplier<T> {
    T get();
}

// أمثلة عملية
Supplier<Double> randomSupplier = Math::random;
Supplier<LocalDateTime> nowSupplier = LocalDateTime::now;
Supplier<UUID> uuidSupplier = UUID::randomUUID;
Supplier<List<String>> listFactory = ArrayList::new;
Supplier<int[]> arrayFactory = () -> new int[10];

// Basic usage
System.out.println(randomSupplier.get());  // 0.7234...
System.out.println(nowSupplier.get());     // 2024-01-15T10:30:00
System.out.println(uuidSupplier.get());    // a1b2c3d4-e5f6-...

// Factory pattern
List<String> newList = listFactory.get();
newList.add("item1");

// Lazy evaluation
Supplier<ExpensiveObject> lazyObject = () -> {
    System.out.println("Creating expensive object...");
    return new ExpensiveObject();
};

System.out.println("Before calling get()");
// Nothing printed yet - lazy!
ExpensiveObject obj = lazyObject.get();  // Now it's created
System.out.println("After calling get()");

// With Optional - orElseGet vs orElse
Optional<String> optional = Optional.empty();

// orElse - ALWAYS evaluates the argument
String result1 = optional.orElse(expensiveOperation());  // expensiveOperation() called!

// orElseGet - Only evaluates if empty
String result2 = optional.orElseGet(() -> expensiveOperation());  // Called only if empty

// Caching with Supplier (Memoization)
class MemoizedSupplier<T> implements Supplier<T> {
    private final Supplier<T> delegate;
    private T cached;
    private boolean computed = false;
    
    public MemoizedSupplier(Supplier<T> delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public T get() {
        if (!computed) {
            cached = delegate.get();
            computed = true;
        }
        return cached;
    }
    
    public static <T> Supplier<T> memoize(Supplier<T> supplier) {
        return new MemoizedSupplier<>(supplier);
    }
}

Supplier<Double> memoizedRandom = MemoizedSupplier.memoize(Math::random);
System.out.println(memoizedRandom.get());  // 0.7234
System.out.println(memoizedRandom.get());  // 0.7234 - same value!
```

### 5. BiFunction<T, U, R>

```java
import java.util.function.BiFunction;

// الـ interface definition
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
    
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after);
}

// أمثلة عملية
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
BiFunction<Integer, Integer, Integer> multiply = (a, b) -> a * b;
BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);
BiFunction<String, String, String> concat = (a, b) -> a + " " + b;

// Basic usage
System.out.println(add.apply(5, 3));           // 8
System.out.println(multiply.apply(5, 3));      // 15
System.out.println(repeat.apply("Ha", 3));     // HaHaHa
System.out.println(concat.apply("Hello", "World")); // Hello World

// Using andThen()
BiFunction<Integer, Integer, Integer> addThenDouble = 
    add.andThen(result -> result * 2);
System.out.println(addThenDouble.apply(5, 3));  // 16 ((5+3)*2)

// With Map operations
Map<String, Integer> scores = new HashMap<>();
scores.put("أحمد", 85);
scores.put("محمد", 90);

// compute - BiFunction<Key, OldValue, NewValue>
scores.compute("أحمد", (name, score) -> score + 10);
System.out.println(scores.get("أحمد"));  // 95

// merge - BiFunction<OldValue, NewValue, Result>
scores.merge("أحمد", 5, (oldVal, newVal) -> oldVal + newVal);
System.out.println(scores.get("أحمد"));  // 100

// Practical example
BiFunction<List<Integer>, Predicate<Integer>, List<Integer>> filter = 
    (list, predicate) -> list.stream()
        .filter(predicate)
        .collect(Collectors.toList());

List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> evens = filter.apply(numbers, n -> n % 2 == 0);
System.out.println(evens);  // [2, 4, 6, 8, 10]
```

### 6. BiConsumer<T, U>

```java
import java.util.function.BiConsumer;

// الـ interface definition
@FunctionalInterface
public interface BiConsumer<T, U> {
    void accept(T t, U u);
    
    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after);
}

// أمثلة عملية
BiConsumer<String, Integer> printPerson = 
    (name, age) -> System.out.println(name + " عمره " + age + " سنة");

BiConsumer<Map<String, Integer>, String> incrementValue = 
    (map, key) -> map.computeIfPresent(key, (k, v) -> v + 1);

BiConsumer<List<String>, String> addIfNotExists = 
    (list, item) -> { if (!list.contains(item)) list.add(item); };

// Basic usage
printPerson.accept("أحمد", 25);  // أحمد عمره 25 سنة

// With Map.forEach
Map<String, Integer> ages = new HashMap<>();
ages.put("أحمد", 25);
ages.put("محمد", 30);
ages.put("علي", 28);

ages.forEach((name, age) -> System.out.println(name + ": " + age));
// أحمد: 25
// محمد: 30
// علي: 28

// Chaining with andThen()
BiConsumer<String, Integer> printAndLog = printPerson
    .andThen((name, age) -> System.out.println("[Logged] " + name));

printAndLog.accept("أحمد", 25);
// أحمد عمره 25 سنة
// [Logged] أحمد

// Practical example - Building objects
BiConsumer<Person, String> setName = Person::setName;
BiConsumer<Person, Integer> setAge = (person, age) -> person.setAge(age);

Person person = new Person();
setName.accept(person, "أحمد");
setAge.accept(person, 25);
```

### 7. BiPredicate<T, U>

```java
import java.util.function.BiPredicate;

// الـ interface definition
@FunctionalInterface
public interface BiPredicate<T, U> {
    boolean test(T t, U u);
    
    default BiPredicate<T, U> and(BiPredicate<? super T, ? super U> other);
    default BiPredicate<T, U> or(BiPredicate<? super T, ? super U> other);
    default BiPredicate<T, U> negate();
}

// أمثلة عملية
BiPredicate<String, Integer> hasMinLength = (s, len) -> s.length() >= len;
BiPredicate<String, Integer> hasMaxLength = (s, len) -> s.length() <= len;
BiPredicate<Integer, Integer> isGreater = (a, b) -> a > b;
BiPredicate<Integer, Integer> isDivisible = (a, b) -> a % b == 0;
BiPredicate<String, String> startsWith = String::startsWith;

// Basic usage
System.out.println(hasMinLength.test("Hello", 3));  // true
System.out.println(hasMinLength.test("Hi", 3));     // false
System.out.println(isGreater.test(10, 5));          // true
System.out.println(isDivisible.test(10, 3));        // false

// Combining
BiPredicate<String, Integer> validLength = hasMinLength.and(hasMaxLength);
// ده مش هيشتغل مباشرة لأن hasMaxLength بتاخد max مش min
// محتاجين نعيد التفكير...

BiPredicate<String, Integer> hasExactLength = (s, len) -> s.length() == len;
System.out.println(hasExactLength.test("Hello", 5));  // true

// Practical example
BiPredicate<Person, Integer> isOlderThan = (person, age) -> person.getAge() > age;
BiPredicate<Person, String> livesIn = (person, city) -> person.getCity().equals(city);

List<Person> people = Arrays.asList(
    new Person("أحمد", 25, "Cairo"),
    new Person("محمد", 35, "Alexandria"),
    new Person("علي", 28, "Cairo")
);

// Filter people older than 26 living in Cairo
people.stream()
    .filter(p -> isOlderThan.test(p, 26) && livesIn.test(p, "Cairo"))
    .forEach(System.out::println);
```

### 8. UnaryOperator<T> و BinaryOperator<T>

```java
import java.util.function.UnaryOperator;
import java.util.function.BinaryOperator;

// UnaryOperator = Function<T, T>
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    static <T> UnaryOperator<T> identity();
}

// BinaryOperator = BiFunction<T, T, T>
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator);
    static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator);
}

// UnaryOperator examples
UnaryOperator<Integer> square = n -> n * n;
UnaryOperator<Integer> increment = n -> n + 1;
UnaryOperator<Integer> negate = n -> -n;
UnaryOperator<String> toUpper = String::toUpperCase;
UnaryOperator<String> trim = String::trim;
UnaryOperator<String> reverse = s -> new StringBuilder(s).reverse().toString();

System.out.println(square.apply(5));       // 25
System.out.println(increment.apply(5));    // 6
System.out.println(toUpper.apply("hello")); // HELLO
System.out.println(reverse.apply("hello")); // olleh

// Chaining UnaryOperators
UnaryOperator<String> processString = trim
    .andThen(toUpper)
    .andThen(reverse);
System.out.println(processString.apply("  hello  "));  // OLLEH

// With List.replaceAll
List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
numbers.replaceAll(square);
System.out.println(numbers);  // [1, 4, 9, 16, 25]

List<String> names = new ArrayList<>(Arrays.asList("ahmed", "mohamed", "ali"));
names.replaceAll(toUpper);
System.out.println(names);  // [AHMED, MOHAMED, ALI]

// BinaryOperator examples
BinaryOperator<Integer> add = (a, b) -> a + b;
BinaryOperator<Integer> multiply = (a, b) -> a * b;
BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
BinaryOperator<Integer> min = (a, b) -> a < b ? a : b;
BinaryOperator<String> concat = (a, b) -> a + b;
BinaryOperator<String> longer = (a, b) -> a.length() >= b.length() ? a : b;

System.out.println(add.apply(5, 3));       // 8
System.out.println(multiply.apply(5, 3));  // 15
System.out.println(max.apply(5, 3));       // 5
System.out.println(longer.apply("Hi", "Hello"));  // Hello

// With Stream.reduce
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);

int sum = nums.stream().reduce(0, add);
System.out.println(sum);  // 15

int product = nums.stream().reduce(1, multiply);
System.out.println(product);  // 120

int maximum = nums.stream().reduce(Integer.MIN_VALUE, max);
System.out.println(maximum);  // 5

// Using minBy and maxBy
BinaryOperator<String> shortestString = BinaryOperator.minBy(Comparator.comparingInt(String::length));
BinaryOperator<String> longestString = BinaryOperator.maxBy(Comparator.comparingInt(String::length));

List<String> words = Arrays.asList("apple", "pie", "banana", "kiwi");
String shortest = words.stream().reduce(shortestString).orElse("");
String longest = words.stream().reduce(longestString).orElse("");

System.out.println(shortest);  // pie
System.out.println(longest);   // banana
```

---

# Composing Functional Interfaces

## Function Composition

```java
// andThen vs compose

Function<Integer, Integer> addOne = x -> x + 1;
Function<Integer, Integer> multiplyTwo = x -> x * 2;

// andThen: addOne THEN multiplyTwo
// (3 + 1) * 2 = 8
Function<Integer, Integer> addThenMultiply = addOne.andThen(multiplyTwo);
System.out.println(addThenMultiply.apply(3));  // 8

// compose: multiplyTwo THEN addOne (reversed order!)
// (3 * 2) + 1 = 7
Function<Integer, Integer> multiplyThenAdd = addOne.compose(multiplyTwo);
System.out.println(multiplyThenAdd.apply(3));  // 7

// Visual representation:
// andThen: input → addOne → multiplyTwo → output
// compose: input → multiplyTwo → addOne → output
```

## Predicate Composition

```java
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isSmall = n -> n < 100;

// AND - كل الشروط لازم تتحقق
Predicate<Integer> isPositiveEvenSmall = isPositive.and(isEven).and(isSmall);

System.out.println(isPositiveEvenSmall.test(50));   // true
System.out.println(isPositiveEvenSmall.test(150));  // false (not small)
System.out.println(isPositiveEvenSmall.test(-4));   // false (not positive)
System.out.println(isPositiveEvenSmall.test(51));   // false (not even)

// OR - شرط واحد يكفي
Predicate<Integer> isPositiveOrEven = isPositive.or(isEven);

System.out.println(isPositiveOrEven.test(5));    // true (positive)
System.out.println(isPositiveOrEven.test(-4));   // true (even)
System.out.println(isPositiveOrEven.test(-3));   // false

// NEGATE - عكس الشرط
Predicate<Integer> isNotPositive = isPositive.negate();

System.out.println(isNotPositive.test(5));   // false
System.out.println(isNotPositive.test(-5));  // true

// Complex composition
Predicate<String> isValidPassword = 
    ((Predicate<String>) s -> s != null)
    .and(s -> s.length() >= 8)
    .and(s -> s.matches(".*[A-Z].*"))    // has uppercase
    .and(s -> s.matches(".*[a-z].*"))    // has lowercase
    .and(s -> s.matches(".*[0-9].*"))    // has digit
    .and(s -> s.matches(".*[!@#$%].*")); // has special char

System.out.println(isValidPassword.test("Password1!"));  // true
System.out.println(isValidPassword.test("password"));    // false
```

## Consumer Composition

```java
Consumer<String> print = System.out::println;
Consumer<String> log = s -> System.out.println("[LOG] " + s);
Consumer<String> save = s -> System.out.println("[SAVE] " + s);

// Chaining consumers
Consumer<String> processMessage = print.andThen(log).andThen(save);

processMessage.accept("Hello");
// Output:
// Hello
// [LOG] Hello
// [SAVE] Hello

// Practical example - Notification system
Consumer<User> sendEmail = user -> System.out.println("Email sent to " + user.getEmail());
Consumer<User> sendSMS = user -> System.out.println("SMS sent to " + user.getPhone());
Consumer<User> sendPush = user -> System.out.println("Push sent to " + user.getDeviceId());
Consumer<User> logNotification = user -> System.out.println("Notification logged for " + user.getName());

Consumer<User> notifyAll = sendEmail.andThen(sendSMS).andThen(sendPush).andThen(logNotification);

User user = new User("أحمد", "ahmed@email.com", "01012345678", "device123");
notifyAll.accept(user);
```

---

# Primitive Specializations

## ليه Primitive Specializations؟

عشان نتجنب الـ **Boxing/Unboxing overhead** اللي بيحصل لما نستخدم Generics مع primitives.

```java
// ❌ مع Generics - فيه Boxing/Unboxing
Function<Integer, Integer> square1 = n -> n * n;
int result1 = square1.apply(5);  // Boxing: 5 → Integer, Unboxing: Integer → int

// ✅ مع Primitive Specialization - مفيش Boxing
IntUnaryOperator square2 = n -> n * n;
int result2 = square2.applyAsInt(5);  // No boxing!
```

## الـ Primitive Functional Interfaces

### For int

```java
// IntPredicate
IntPredicate isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4));  // true

// IntFunction<R> - int → R
IntFunction<String> intToString = n -> "Value: " + n;
System.out.println(intToString.apply(42));  // "Value: 42"

// ToIntFunction<T> - T → int
ToIntFunction<String> stringLength = String::length;
System.out.println(stringLength.applyAsInt("Hello"));  // 5

// IntConsumer
IntConsumer printInt = n -> System.out.println("Number: " + n);
printInt.accept(42);  // "Number: 42"

// IntSupplier
IntSupplier randomInt = () -> (int)(Math.random() * 100);
System.out.println(randomInt.getAsInt());  // Random 0-99

// IntUnaryOperator - int → int
IntUnaryOperator square = n -> n * n;
IntUnaryOperator increment = n -> n + 1;
System.out.println(square.applyAsInt(5));  // 25

// IntBinaryOperator - (int, int) → int
IntBinaryOperator add = (a, b) -> a + b;
IntBinaryOperator max = Integer::max;
System.out.println(add.applyAsInt(5, 3));  // 8
System.out.println(max.applyAsInt(5, 3));  // 5

// ObjIntConsumer<T> - (T, int) → void
ObjIntConsumer<String> printWithIndex = (s, i) -> System.out.println(i + ": " + s);
printWithIndex.accept("Hello", 0);  // "0: Hello"

// IntStream usage
int[] numbers = {1, 2, 3, 4, 5};
int sum = Arrays.stream(numbers)
    .filter(isEven)
    .map(square)
    .sum();
System.out.println(sum);  // 4 + 16 = 20
```

### For long

```java
// LongPredicate
LongPredicate isPositive = n -> n > 0;

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

// ObjLongConsumer<T>
ObjLongConsumer<String> printWithTime = (s, time) -> 
    System.out.println(s + " at " + time);
```

### For double

```java
// DoublePredicate
DoublePredicate isPositive = d -> d > 0;

// DoubleFunction<R>
DoubleFunction<String> doubleToString = d -> String.format("%.2f", d);

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
DoubleBinaryOperator addDoubles = Double::sum;
DoubleBinaryOperator power = Math::pow;

// ObjDoubleConsumer<T>
ObjDoubleConsumer<String> printWithValue = (s, d) -> 
    System.out.println(s + ": " + d);

// Example usage
DoubleStream.of(1.0, 4.0, 9.0, 16.0)
    .filter(isPositive)
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

// Usage
int i = 42;
long l = intToLong.applyAsLong(i);
double d = intToDouble.applyAsDouble(i);
```

## جدول الـ Primitive Specializations

| Generic Interface | int | long | double |
|-------------------|-----|------|--------|
| `Predicate<T>` | `IntPredicate` | `LongPredicate` | `DoublePredicate` |
| `Function<T,R>` | `IntFunction<R>` | `LongFunction<R>` | `DoubleFunction<R>` |
| `Consumer<T>` | `IntConsumer` | `LongConsumer` | `DoubleConsumer` |
| `Supplier<T>` | `IntSupplier` | `LongSupplier` | `DoubleSupplier` |
| `UnaryOperator<T>` | `IntUnaryOperator` | `LongUnaryOperator` | `DoubleUnaryOperator` |
| `BinaryOperator<T>` | `IntBinaryOperator` | `LongBinaryOperator` | `DoubleBinaryOperator` |
| `ToXFunction<T>` | `ToIntFunction<T>` | `ToLongFunction<T>` | `ToDoubleFunction<T>` |

---

# Functional Interfaces مع Generics

## 1. Simple Generic Interface

```java
@FunctionalInterface
interface Transformer<T> {
    T transform(T input);
}

Transformer<String> upperCase = String::toUpperCase;
Transformer<Integer> doubleIt = n -> n * 2;

System.out.println(upperCase.transform("hello"));  // HELLO
System.out.println(doubleIt.transform(5));         // 10
```

## 2. Multiple Type Parameters

```java
@FunctionalInterface
interface Mapper<T, R> {
    R map(T input);
}

Mapper<String, Integer> stringToLength = String::length;
Mapper<Integer, String> intToHex = Integer::toHexString;

System.out.println(stringToLength.map("Hello"));  // 5
System.out.println(intToHex.map(255));            // ff
```

## 3. Bounded Type Parameters

```java
// Upper bound - T لازم يـ extend Number
@FunctionalInterface
interface NumberProcessor<T extends Number> {
    double process(T number);
}

NumberProcessor<Integer> intProcessor = Integer::doubleValue;
NumberProcessor<Double> doubleProcessor = d -> d * 2;

System.out.println(intProcessor.process(42));      // 42.0
System.out.println(doubleProcessor.process(3.14)); // 6.28

// Multiple bounds
@FunctionalInterface
interface ComparableProcessor<T extends Comparable<T> & Serializable> {
    int compare(T a, T b);
}

ComparableProcessor<String> stringComparer = String::compareTo;
System.out.println(stringComparer.compare("a", "b"));  // -1
```

## 4. Wildcard in Functional Interface

```java
// Consumer with wildcard
Consumer<? super String> printer = System.out::println;
printer.accept("Hello");

// Function with wildcards
Function<? extends Number, String> numberToString = Object::toString;

// Predicate with wildcard
Predicate<? super Integer> isPositive = n -> ((Number)n).doubleValue() > 0;
```

## 5. Generic Methods in Functional Interface

```java
@FunctionalInterface
interface GenericFactory<T> {
    T create();
    
    // Generic default method
    default <R> R createAndTransform(Function<T, R> transformer) {
        return transformer.apply(create());
    }
    
    // Generic static method
    static <T> GenericFactory<List<T>> listFactory() {
        return ArrayList::new;
    }
}

GenericFactory<String> stringFactory = () -> "Hello";
Integer length = stringFactory.createAndTransform(String::length);
System.out.println(length);  // 5

GenericFactory<List<String>> listFactory = GenericFactory.listFactory();
List<String> list = listFactory.create();
```

---

# Functional Interfaces مع Exceptions

## المشكلة

```java
// Built-in functional interfaces مش بتدعم checked exceptions!
List<String> paths = Arrays.asList("file1.txt", "file2.txt");

// ❌ Compile Error!
// paths.forEach(path -> Files.delete(Path.of(path)));
// Unhandled exception: IOException
```

## الحل 1: Custom Functional Interface with Exception

```java
@FunctionalInterface
interface ThrowingConsumer<T, E extends Exception> {
    void accept(T t) throws E;
}

@FunctionalInterface
interface ThrowingFunction<T, R, E extends Exception> {
    R apply(T t) throws E;
}

@FunctionalInterface
interface ThrowingSupplier<T, E extends Exception> {
    T get() throws E;
}

@FunctionalInterface
interface ThrowingRunnable<E extends Exception> {
    void run() throws E;
}

// استخدام
ThrowingConsumer<String, IOException> deleteFile = 
    path -> Files.delete(Path.of(path));

ThrowingFunction<String, String, IOException> readFile = 
    path -> Files.readString(Path.of(path));
```

## الحل 2: Wrapper Methods

```java
public class ThrowingWrapper {
    
    // Wrap ThrowingConsumer to Consumer
    public static <T> Consumer<T> wrap(ThrowingConsumer<T, Exception> consumer) {
        return t -> {
            try {
                consumer.accept(t);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        };
    }
    
    // Wrap ThrowingFunction to Function
    public static <T, R> Function<T, R> wrap(ThrowingFunction<T, R, Exception> function) {
        return t -> {
            try {
                return function.apply(t);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        };
    }
    
    // Wrap ThrowingSupplier to Supplier
    public static <T> Supplier<T> wrap(ThrowingSupplier<T, Exception> supplier) {
        return () -> {
            try {
                return supplier.get();
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        };
    }
}

// استخدام
List<String> paths = Arrays.asList("file1.txt", "file2.txt");
paths.forEach(ThrowingWrapper.wrap(path -> Files.delete(Path.of(path))));

List<String> contents = paths.stream()
    .map(ThrowingWrapper.wrap(path -> Files.readString(Path.of(path))))
    .collect(Collectors.toList());
```

## الحل 3: Either/Result Pattern

```java
class Result<T> {
    private final T value;
    private final Exception error;
    
    private Result(T value, Exception error) {
        this.value = value;
        this.error = error;
    }
    
    public static <T> Result<T> success(T value) {
        return new Result<>(value, null);
    }
    
    public static <T> Result<T> failure(Exception error) {
        return new Result<>(null, error);
    }
    
    public boolean isSuccess() {
        return error == null;
    }
    
    public T getValue() {
        if (error != null) throw new IllegalStateException("Result is failure");
        return value;
    }
    
    public Exception getError() {
        return error;
    }
    
    public T getOrElse(T defaultValue) {
        return isSuccess() ? value : defaultValue;
    }
    
    public <R> Result<R> map(Function<T, R> mapper) {
        if (isSuccess()) {
            try {
                return Result.success(mapper.apply(value));
            } catch (Exception e) {
                return Result.failure(e);
            }
        }
        return Result.failure(error);
    }
    
    // Safe execution wrapper
    public static <T> Result<T> of(ThrowingSupplier<T, Exception> supplier) {
        try {
            return Result.success(supplier.get());
        } catch (Exception e) {
            return Result.failure(e);
        }
    }
}

// استخدام
List<String> paths = Arrays.asList("file1.txt", "file2.txt", "nonexistent.txt");

List<Result<String>> results = paths.stream()
    .map(path -> Result.of(() -> Files.readString(Path.of(path))))
    .collect(Collectors.toList());

// Process results
results.forEach(result -> {
    if (result.isSuccess()) {
        System.out.println("Content: " + result.getValue());
    } else {
        System.out.println("Error: " + result.getError().getMessage());
    }
});

// Filter successful results
List<String> successfulReads = results.stream()
    .filter(Result::isSuccess)
    .map(Result::getValue)
    .collect(Collectors.toList());
```

---

# Design Patterns مع Functional Interfaces

## 1. Strategy Pattern

```java
// Traditional Strategy Pattern
interface PaymentStrategy {
    void pay(double amount);
}

class CreditCardPayment implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("Paid " + amount + " using Credit Card");
    }
}

// Functional approach - أبسط!
@FunctionalInterface
interface PaymentStrategy {
    void pay(double amount);
    
    // Pre-built strategies
    static PaymentStrategy creditCard() {
        return amount -> System.out.println("Credit Card: $" + amount);
    }
    
    static PaymentStrategy paypal(String email) {
        return amount -> System.out.println("PayPal (" + email + "): $" + amount);
    }
    
    static PaymentStrategy cash() {
        return amount -> System.out.println("Cash: $" + amount);
    }
}

class PaymentProcessor {
    private PaymentStrategy strategy;
    
    public void setStrategy(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void processPayment(double amount) {
        strategy.pay(amount);
    }
}

// استخدام
PaymentProcessor processor = new PaymentProcessor();

processor.setStrategy(PaymentStrategy.creditCard());
processor.processPayment(100);  // Credit Card: $100

processor.setStrategy(PaymentStrategy.paypal("user@email.com"));
processor.processPayment(50);   // PayPal (user@email.com): $50

// Custom strategy with lambda
processor.setStrategy(amount -> System.out.println("Bitcoin: ฿" + (amount / 50000)));
processor.processPayment(100);  // Bitcoin: ฿0.002
```

## 2. Template Method Pattern

```java
@FunctionalInterface
interface DataProcessor<T, R> {
    R process(T data);
    
    // Template method with hooks
    default R processWithLogging(T data, Consumer<String> logger) {
        logger.accept("Starting processing...");
        R result = process(data);
        logger.accept("Processing complete.");
        return result;
    }
    
    default R processWithValidation(T data, Predicate<T> validator) {
        if (!validator.test(data)) {
            throw new IllegalArgumentException("Invalid data");
        }
        return process(data);
    }
    
    default R processWithRetry(T data, int maxRetries) {
        Exception lastException = null;
        for (int i = 0; i < maxRetries; i++) {
            try {
                return process(data);
            } catch (Exception e) {
                lastException = e;
            }
        }
        throw new RuntimeException("Failed after " + maxRetries + " retries", lastException);
    }
}

// استخدام
DataProcessor<String, Integer> lengthProcessor = String::length;

// With logging
Integer result = lengthProcessor.processWithLogging("Hello", System.out::println);
// Starting processing...
// Processing complete.
// result = 5

// With validation
Integer result2 = lengthProcessor.processWithValidation(
    "Hello",
    s -> s != null && !s.isEmpty()
);
```

## 3. Builder Pattern

```java
class EmailBuilder {
    private String to;
    private String from;
    private String subject;
    private String body;
    private List<String> attachments = new ArrayList<>();
    
    public EmailBuilder with(Consumer<EmailBuilder> builderFunction) {
        builderFunction.accept(this);
        return this;
    }
    
    public EmailBuilder to(String to) { this.to = to; return this; }
    public EmailBuilder from(String from) { this.from = from; return this; }
    public EmailBuilder subject(String subject) { this.subject = subject; return this; }
    public EmailBuilder body(String body) { this.body = body; return this; }
    public EmailBuilder attachment(String attachment) { 
        this.attachments.add(attachment); 
        return this; 
    }
    
    public Email build() {
        return new Email(to, from, subject, body, attachments);
    }
    
    // Fluent API with functions
    public EmailBuilder transform(UnaryOperator<EmailBuilder> transformer) {
        return transformer.apply(this);
    }
}

// استخدام
Email email = new EmailBuilder()
    .with(b -> b.to("recipient@email.com"))
    .with(b -> b.from("sender@email.com"))
    .with(b -> b.subject("Hello"))
    .with(b -> b.body("This is the body"))
    .with(b -> {
        b.attachment("file1.pdf");
        b.attachment("file2.pdf");
    })
    .build();

// Reusable transformers
UnaryOperator<EmailBuilder> addDefaultSender = b -> b.from("noreply@company.com");
UnaryOperator<EmailBuilder> addFooter = b -> b.body(b.body + "\n\n-- Sent from MyApp");

Email email2 = new EmailBuilder()
    .to("user@email.com")
    .subject("News")
    .body("Content")
    .transform(addDefaultSender)
    .transform(addFooter)
    .build();
```

## 4. Chain of Responsibility Pattern

```java
@FunctionalInterface
interface Handler<T> {
    boolean handle(T request);
    
    default Handler<T> then(Handler<T> next) {
        return request -> {
            if (this.handle(request)) {
                return true;
            }
            return next.handle(request);
        };
    }
    
    static <T> Handler<T> chain(Handler<T>... handlers) {
        return request -> {
            for (Handler<T> handler : handlers) {
                if (handler.handle(request)) {
                    return true;
                }
            }
            return false;
        };
    }
}

// استخدام
class Request {
    String type;
    int amount;
    
    Request(String type, int amount) {
        this.type = type;
        this.amount = amount;
    }
}

Handler<Request> managerHandler = request -> {
    if (request.amount <= 1000) {
        System.out.println("Manager approved: $" + request.amount);
        return true;
    }
    return false;
};

Handler<Request> directorHandler = request -> {
    if (request.amount <= 5000) {
        System.out.println("Director approved: $" + request.amount);
        return true;
    }
    return false;
};

Handler<Request> ceoHandler = request -> {
    System.out.println("CEO approved: $" + request.amount);
    return true;
};

Handler<Request> approvalChain = managerHandler.then(directorHandler).then(ceoHandler);

approvalChain.handle(new Request("purchase", 500));   // Manager approved: $500
approvalChain.handle(new Request("purchase", 3000));  // Director approved: $3000
approvalChain.handle(new Request("purchase", 10000)); // CEO approved: $10000
```

## 5. Observer Pattern

```java
class EventEmitter<T> {
    private final List<Consumer<T>> listeners = new ArrayList<>();
    
    public void subscribe(Consumer<T> listener) {
        listeners.add(listener);
    }
    
    public void unsubscribe(Consumer<T> listener) {
        listeners.remove(listener);
    }
    
    public void emit(T event) {
        listeners.forEach(listener -> listener.accept(event));
    }
    
    // Filtered subscription
    public void subscribe(Predicate<T> filter, Consumer<T> listener) {
        listeners.add(event -> {
            if (filter.test(event)) {
                listener.accept(event);
            }
        });
    }
}

// استخدام
EventEmitter<String> emitter = new EventEmitter<>();

emitter.subscribe(msg -> System.out.println("Listener 1: " + msg));
emitter.subscribe(msg -> System.out.println("Listener 2: " + msg));

// Filtered listener - only messages starting with "ALERT"
emitter.subscribe(
    msg -> msg.startsWith("ALERT"),
    msg -> System.out.println("ALERT Handler: " + msg)
);

emitter.emit("Hello");
// Listener 1: Hello
// Listener 2: Hello

emitter.emit("ALERT: System down!");
// Listener 1: ALERT: System down!
// Listener 2: ALERT: System down!
// ALERT Handler: ALERT: System down!
```

## 6. Decorator Pattern

```java
@FunctionalInterface
interface TextProcessor {
    String process(String text);
    
    default TextProcessor andThen(TextProcessor after) {
        return text -> after.process(this.process(text));
    }
    
    default TextProcessor compose(TextProcessor before) {
        return text -> this.process(before.process(text));
    }
    
    // Decorators as static methods
    static TextProcessor trim() {
        return String::trim;
    }
    
    static TextProcessor toUpperCase() {
        return String::toUpperCase;
    }
    
    static TextProcessor toLowerCase() {
        return String::toLowerCase;
    }
    
    static TextProcessor addPrefix(String prefix) {
        return text -> prefix + text;
    }
    
    static TextProcessor addSuffix(String suffix) {
        return text -> text + suffix;
    }
    
    static TextProcessor replaceSpaces(String replacement) {
        return text -> text.replace(" ", replacement);
    }
    
    static TextProcessor truncate(int maxLength) {
        return text -> text.length() > maxLength 
            ? text.substring(0, maxLength) + "..." 
            : text;
    }
}

// استخدام
TextProcessor slugify = TextProcessor.trim()
    .andThen(TextProcessor.toLowerCase())
    .andThen(TextProcessor.replaceSpaces("-"));

System.out.println(slugify.process("  Hello World  "));  // hello-world

TextProcessor formatTitle = TextProcessor.trim()
    .andThen(TextProcessor.toUpperCase())
    .andThen(TextProcessor.addPrefix("=== "))
    .andThen(TextProcessor.addSuffix(" ==="));

System.out.println(formatTitle.process("  my title  "));  // === MY TITLE ===
```

---

# Best Practices

## 1. استخدم @FunctionalInterface دايماً

```java
// ✅ Good
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

// ❌ Bad - مفيش protection
interface Calculator {
    int calculate(int a, int b);
}
```

## 2. استخدم Built-in Interfaces لما ينفع

```java
// ❌ Don't create custom interface for common patterns
@FunctionalInterface
interface StringValidator {
    boolean isValid(String s);
}

// ✅ Use Predicate<String> instead
Predicate<String> isValid = s -> s != null && !s.isEmpty();
```

## 3. Keep Lambda Simple

```java
// ❌ Too complex
Function<String, String> complex = s -> {
    String trimmed = s.trim();
    String lower = trimmed.toLowerCase();
    String replaced = lower.replace(" ", "_");
    return replaced;
};

// ✅ Extract to method or chain
Function<String, String> simple = ((Function<String, String>) String::trim)
    .andThen(String::toLowerCase)
    .andThen(s -> s.replace(" ", "_"));

// Or extract to named method
private String processString(String s) {
    return s.trim().toLowerCase().replace(" ", "_");
}
Function<String, String> simple2 = this::processString;
```

## 4. استخدم Method Reference لما يكون أوضح

```java
// ✅ Clear method reference
list.forEach(System.out::println);
list.stream().map(String::toUpperCase);

// ❌ Unnecessary lambda
list.forEach(s -> System.out.println(s));
list.stream().map(s -> s.toUpperCase());
```

## 5. Handle Nulls Properly

```java
// ❌ May throw NullPointerException
Function<String, Integer> length = s -> s.length();

// ✅ Null-safe
Function<String, Integer> safeLength = s -> s == null ? 0 : s.length();

// ✅ Or use Optional
Function<String, Integer> optionalLength = s -> 
    Optional.ofNullable(s).map(String::length).orElse(0);
```

## 6. Avoid Side Effects in Functions

```java
// ❌ Bad - side effect in Function
List<String> processed = new ArrayList<>();
Function<String, String> badFunction = s -> {
    processed.add(s);  // Side effect!
    return s.toUpperCase();
};

// ✅ Good - pure function
Function<String, String> goodFunction = String::toUpperCase;
```

## 7. استخدم Primitive Specializations للـ Performance

```java
// ❌ Boxing overhead
Function<Integer, Integer> square1 = n -> n * n;

// ✅ No boxing
IntUnaryOperator square2 = n -> n * n;
```

---

# أسئلة الانترفيو

## أسئلة نظرية

### س1: إيه هو الـ Functional Interface وإيه شروطه؟

**الإجابة:**
Functional Interface هو interface فيه **abstract method واحدة بس** (Single Abstract Method - SAM). ده الأساس اللي بيخلي Lambda تشتغل.

**الشروط:**
- Abstract method واحدة بس
- Default methods مش بتتحسب
- Static methods مش بتتحسب
- Methods من Object class (equals, hashCode, toString) مش بتتحسب

```java
@FunctionalInterface
interface Valid {
    void doSomething();           // SAM
    default void helper() {}       // مش بتتحسب
    static void util() {}          // مش بتتحسب
    boolean equals(Object o);      // من Object
}
```

---

### س2: إيه الفرق بين Function و UnaryOperator؟

**الإجابة:**

| Function<T, R> | UnaryOperator<T> |
|----------------|------------------|
| Input type T | Input type T |
| Output type R (أي نوع) | Output type T (نفس النوع) |
| `Function<String, Integer>` | `UnaryOperator<String>` |

```java
// Function - ممكن نوع مختلف
Function<String, Integer> length = String::length;

// UnaryOperator - لازم نفس النوع
UnaryOperator<String> toUpper = String::toUpperCase;

// UnaryOperator extends Function<T, T>
```

---

### س3: إيه الفرق بين Predicate و Function<T, Boolean>؟

**الإجابة:**

| Predicate<T> | Function<T, Boolean> |
|--------------|---------------------|
| Method: `test(T t)` | Method: `apply(T t)` |
| Returns primitive `boolean` | Returns `Boolean` wrapper |
| No boxing | Boxing overhead |
| Has `and()`, `or()`, `negate()` | Only `andThen()`, `compose()` |

```java
// Predicate - أفضل للـ boolean operations
Predicate<String> isEmpty = String::isEmpty;
boolean result = isEmpty.test("");  // No boxing

// Function<T, Boolean> - فيه boxing
Function<String, Boolean> isEmpty2 = String::isEmpty;
Boolean result2 = isEmpty2.apply("");  // Boxing
```

---

### س4: إزاي تعمل composition لـ multiple Predicates؟

**الإجابة:**

```java
Predicate<String> notNull = s -> s != null;
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> validLength = s -> s.length() >= 5 && s.length() <= 20;
Predicate<String> noSpaces = s -> !s.contains(" ");

// AND - كل الشروط
Predicate<String> validUsername = notNull
    .and(notEmpty)
    .and(validLength)
    .and(noSpaces);

// OR - شرط واحد
Predicate<String> startsWithAOrB = 
    ((Predicate<String>) s -> s.startsWith("A"))
    .or(s -> s.startsWith("B"));

// NEGATE - عكس
Predicate<String> hasSpaces = noSpaces.negate();

// Predicate.not() - Java 11+
Predicate<String> isNotEmpty = Predicate.not(String::isEmpty);
```

---

### س5: إيه الفرق بين `andThen` و `compose` في Function؟

**الإجابة:**

```java
Function<Integer, Integer> addOne = x -> x + 1;
Function<Integer, Integer> multiplyTwo = x -> x * 2;

// andThen: left to right
// addOne THEN multiplyTwo
// 3 → (3+1) → (4*2) → 8
Function<Integer, Integer> f1 = addOne.andThen(multiplyTwo);
System.out.println(f1.apply(3));  // 8

// compose: right to left
// multiplyTwo THEN addOne
// 3 → (3*2) → (6+1) → 7
Function<Integer, Integer> f2 = addOne.compose(multiplyTwo);
System.out.println(f2.apply(3));  // 7
```

**Visual:**
```
andThen: input → f1 → f2 → output
compose: input → f2 → f1 → output
```

---

### س6: ليه نستخدم Primitive Specializations؟

**الإجابة:**
عشان نتجنب **Boxing/Unboxing overhead**:

```java
// ❌ With generics - Boxing
Function<Integer, Integer> square1 = n -> n * n;
// int → Integer (boxing) → Integer → int (unboxing)

// ✅ With primitive specialization - No boxing
IntUnaryOperator square2 = n -> n * n;
// int → int (direct)
```

**Performance difference:**
- Boxing creates new objects
- More GC pressure
- Slower execution

---

### س7: إزاي تتعامل مع Checked Exceptions في Functional Interfaces؟

**الإجابة:**

```java
// المشكلة: built-in interfaces مش بتدعم checked exceptions

// الحل 1: Custom throwing interface
@FunctionalInterface
interface ThrowingFunction<T, R, E extends Exception> {
    R apply(T t) throws E;
}

// الحل 2: Wrapper
public static <T, R> Function<T, R> wrap(ThrowingFunction<T, R, Exception> f) {
    return t -> {
        try {
            return f.apply(t);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    };
}

// استخدام
List<String> paths = Arrays.asList("file1.txt", "file2.txt");
paths.stream()
    .map(wrap(path -> Files.readString(Path.of(path))))
    .forEach(System.out::println);
```

---

### س8: إيه الفرق بين Consumer و BiConsumer؟

**الإجابة:**

| Consumer<T> | BiConsumer<T, U> |
|-------------|------------------|
| Input واحد | 2 inputs |
| `accept(T t)` | `accept(T t, U u)` |
| `list.forEach(consumer)` | `map.forEach(biConsumer)` |

```java
// Consumer - input واحد
Consumer<String> printer = System.out::println;
List<String> list = Arrays.asList("a", "b");
list.forEach(printer);

// BiConsumer - 2 inputs
BiConsumer<String, Integer> printWithIndex = 
    (s, i) -> System.out.println(i + ": " + s);
    
Map<String, Integer> map = Map.of("a", 1, "b", 2);
map.forEach(printWithIndex);
```

---

### س9: إمتى تستخدم Supplier؟

**الإجابة:**

1. **Lazy evaluation:**
```java
Optional<String> opt = Optional.empty();
// orElseGet بياخد Supplier - lazy
String result = opt.orElseGet(() -> expensiveOperation());
```

2. **Factory pattern:**
```java
Supplier<List<String>> listFactory = ArrayList::new;
List<String> newList = listFactory.get();
```

3. **Deferred execution:**
```java
Supplier<LocalDateTime> now = LocalDateTime::now;
// القيمة هتتحسب لما نـ call get()
```

4. **Configuration/Default values:**
```java
public void process(Supplier<Config> configSupplier) {
    Config config = configSupplier.get();
}
```

---

### س10: إيه هو Function.identity() وإمتى نستخدمه؟

**الإجابة:**
`Function.identity()` بترجع function بتاخد input وترجعه زي ما هو:

```java
Function<String, String> identity = Function.identity();
// equivalent to: s -> s

// Use cases:

// 1. Collectors.toMap
Map<String, String> map = list.stream()
    .collect(Collectors.toMap(
        Function.identity(),  // key = element itself
        String::toUpperCase   // value = uppercase
    ));

// 2. Grouping
Map<String, List<String>> grouped = list.stream()
    .collect(Collectors.groupingBy(Function.identity()));

// 3. When transformation is optional
Function<String, String> transform = shouldTransform 
    ? String::toUpperCase 
    : Function.identity();
```

---

## أسئلة كود

### س11: اكتب Functional Interface للـ Validation مع combining

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
    
    default Validator<T> or(Validator<T> other) {
        return value -> {
            ValidationResult result = this.validate(value);
            if (result.isValid()) return result;
            return other.validate(value);
        };
    }
    
    static <T> Validator<T> from(Predicate<T> predicate, String errorMessage) {
        return value -> predicate.test(value) 
            ? ValidationResult.valid() 
            : ValidationResult.invalid(errorMessage);
    }
}

record ValidationResult(boolean isValid, String message) {
    static ValidationResult valid() { return new ValidationResult(true, ""); }
    static ValidationResult invalid(String msg) { return new ValidationResult(false, msg); }
}

// استخدام
Validator<String> notNull = Validator.from(s -> s != null, "Cannot be null");
Validator<String> notEmpty = Validator.from(s -> !s.isEmpty(), "Cannot be empty");
Validator<String> minLength = Validator.from(s -> s.length() >= 5, "Too short");

Validator<String> passwordValidator = notNull.and(notEmpty).and(minLength);
ValidationResult result = passwordValidator.validate("abc");
// ValidationResult[isValid=false, message=Too short]
```

---

### س12: اكتب Functional Interface للـ Retry Logic

```java
@FunctionalInterface
interface RetryableOperation<T> {
    T execute() throws Exception;
    
    default T executeWithRetry(int maxRetries, long delayMs) {
        Exception lastException = null;
        for (int i = 0; i <= maxRetries; i++) {
            try {
                return execute();
            } catch (Exception e) {
                lastException = e;
                if (i < maxRetries) {
                    System.out.println("Attempt " + (i+1) + " failed, retrying...");
                    try { Thread.sleep(delayMs); } catch (InterruptedException ie) { }
                }
            }
        }
        throw new RuntimeException("Failed after " + maxRetries + " retries", lastException);
    }
    
    default T executeWithExponentialBackoff(int maxRetries) {
        Exception lastException = null;
        for (int i = 0; i <= maxRetries; i++) {
            try {
                return execute();
            } catch (Exception e) {
                lastException = e;
                if (i < maxRetries) {
                    long delay = (long) Math.pow(2, i) * 1000;
                    System.out.println("Attempt " + (i+1) + " failed, waiting " + delay + "ms...");
                    try { Thread.sleep(delay); } catch (InterruptedException ie) { }
                }
            }
        }
        throw new RuntimeException("Failed after " + maxRetries + " retries", lastException);
    }
}

// استخدام
RetryableOperation<String> fetchData = () -> {
    // Simulated API call that might fail
    if (Math.random() < 0.7) throw new IOException("Network error");
    return "Success!";
};

String result = fetchData.executeWithRetry(3, 1000);
```

---

### س13: اكتب Pipeline Pattern باستخدام Functional Interfaces

```java
class Pipeline<T> {
    private final List<Function<T, T>> stages = new ArrayList<>();
    
    public Pipeline<T> addStage(Function<T, T> stage) {
        stages.add(stage);
        return this;
    }
    
    public Pipeline<T> addStage(UnaryOperator<T> stage) {
        stages.add(stage);
        return this;
    }
    
    public T execute(T input) {
        T result = input;
        for (Function<T, T> stage : stages) {
            result = stage.apply(result);
        }
        return result;
    }
    
    public Pipeline<T> addConditionalStage(Predicate<T> condition, Function<T, T> stage) {
        stages.add(value -> condition.test(value) ? stage.apply(value) : value);
        return this;
    }
    
    // Combine two pipelines
    public Pipeline<T> merge(Pipeline<T> other) {
        Pipeline<T> merged = new Pipeline<>();
        merged.stages.addAll(this.stages);
        merged.stages.addAll(other.stages);
        return merged;
    }
}

// استخدام
Pipeline<String> textPipeline = new Pipeline<String>()
    .addStage(String::trim)
    .addStage(String::toLowerCase)
    .addStage(s -> s.replaceAll("\\s+", " "))
    .addConditionalStage(
        s -> s.length() > 50,
        s -> s.substring(0, 50) + "..."
    );

String result = textPipeline.execute("  HELLO   WORLD  ");
System.out.println(result);  // "hello world"
```

---

### س14: اكتب Event System باستخدام Functional Interfaces

```java
class EventBus<E> {
    private final Map<Class<?>, List<Consumer<Object>>> handlers = new HashMap<>();
    
    public <T extends E> void subscribe(Class<T> eventType, Consumer<T> handler) {
        handlers.computeIfAbsent(eventType, k -> new ArrayList<>())
            .add(event -> handler.accept((T) event));
    }
    
    public <T extends E> void publish(T event) {
        List<Consumer<Object>> eventHandlers = handlers.get(event.getClass());
        if (eventHandlers != null) {
            eventHandlers.forEach(handler -> handler.accept(event));
        }
    }
    
    // Filter-based subscription
    public <T extends E> void subscribe(Class<T> eventType, 
                                        Predicate<T> filter, 
                                        Consumer<T> handler) {
        subscribe(eventType, event -> {
            if (filter.test(event)) {
                handler.accept(event);
            }
        });
    }
}

// Events
record UserCreated(String username, String email) {}
record UserDeleted(String username) {}
record OrderPlaced(String orderId, double amount) {}

// استخدام
EventBus<Object> bus = new EventBus<>();

bus.subscribe(UserCreated.class, e -> 
    System.out.println("New user: " + e.username()));

bus.subscribe(OrderPlaced.class, 
    order -> order.amount() > 100,  // Filter: only high-value orders
    order -> System.out.println("High-value order: " + order.orderId()));

bus.publish(new UserCreated("ahmed", "ahmed@email.com"));
bus.publish(new OrderPlaced("ORD-001", 150));  // Will trigger
bus.publish(new OrderPlaced("ORD-002", 50));   // Won't trigger (filtered)
```

---

### س15: اكتب Memoization Decorator

```java
class Memoizer {
    
    // Memoize Function
    public static <T, R> Function<T, R> memoize(Function<T, R> function) {
        Map<T, R> cache = new ConcurrentHashMap<>();
        return input -> cache.computeIfAbsent(input, function);
    }
    
    // Memoize BiFunction
    public static <T, U, R> BiFunction<T, U, R> memoize(BiFunction<T, U, R> function) {
        Map<List<Object>, R> cache = new ConcurrentHashMap<>();
        return (t, u) -> cache.computeIfAbsent(
            Arrays.asList(t, u),
            key -> function.apply(t, u)
        );
    }
    
    // Memoize Supplier (single value)
    public static <T> Supplier<T> memoize(Supplier<T> supplier) {
        AtomicReference<T> cache = new AtomicReference<>();
        AtomicBoolean computed = new AtomicBoolean(false);
        return () -> {
            if (!computed.get()) {
                synchronized (cache) {
                    if (!computed.get()) {
                        cache.set(supplier.get());
                        computed.set(true);
                    }
                }
            }
            return cache.get();
        };
    }
    
    // Memoize with expiration
    public static <T, R> Function<T, R> memoizeWithExpiry(
            Function<T, R> function, 
            long ttlMs) {
        Map<T, CacheEntry<R>> cache = new ConcurrentHashMap<>();
        return input -> {
            CacheEntry<R> entry = cache.get(input);
            if (entry != null && !entry.isExpired()) {
                return entry.value;
            }
            R result = function.apply(input);
            cache.put(input, new CacheEntry<>(result, ttlMs));
            return result;
        };
    }
    
    private record CacheEntry<T>(T value, long expiryTime) {
        CacheEntry(T value, long ttlMs) {
            this(value, System.currentTimeMillis() + ttlMs);
        }
        boolean isExpired() {
            return System.currentTimeMillis() > expiryTime;
        }
    }
}

// استخدام
Function<Integer, Integer> slowFibonacci = n -> {
    if (n <= 1) return n;
    return slowFibonacci.apply(n - 1) + slowFibonacci.apply(n - 2);
};

Function<Integer, Integer> fastFibonacci = Memoizer.memoize(slowFibonacci);

System.out.println(fastFibonacci.apply(40));  // Fast with memoization
```

---

## نصائح للانترفيو 💡

1. **اعرف الـ Built-in Interfaces كويس** - Predicate, Function, Consumer, Supplier
2. **افهم الفرق بين Primitive و Generic versions** - ليه وإمتى نستخدم كل نوع
3. **اتدرب على Composition** - `and`, `or`, `andThen`, `compose`
4. **اعرف إزاي تتعامل مع Exceptions** في Functional Interfaces
5. **كن جاهز تكتب Custom Functional Interfaces** مع default و static methods
6. **افهم Design Patterns** وإزاي تـ implement بـ Functional style
7. **اعرف الفرق بين Method Reference و Lambda** وإمتى تستخدم كل واحد

---

## موارد إضافية 📚

- [java.util.function Package](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)
- [Oracle Functional Interfaces Tutorial](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
- [Effective Java - Item 44: Favor standard functional interfaces](https://www.oreilly.com/library/view/effective-java/9780134686097/)

---

*تم إعداد هذا الدليل بالعامية المصرية عشان يكون سهل الفهم والمراجعة* 🇪🇬
