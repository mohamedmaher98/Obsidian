# 🔒 Working with Variables in Lambdas في Java - دليل شامل بالمصري

## المحتويات
- [مقدمة عن Variable Capture](#مقدمة-عن-variable-capture)
- [أنواع المتغيرات في Lambda](#أنواع-المتغيرات-في-lambda)
- [Effectively Final بالتفصيل](#effectively-final-بالتفصيل)
- [الفرق بين Lambda و Anonymous Class](#الفرق-بين-lambda-و-anonymous-class)
- [Closures في Java](#closures-في-java)
- [مشاكل شائعة وحلولها](#مشاكل-شائعة-وحلولها)
- [Best Practices](#best-practices)
- [أسئلة الانترفيو](#أسئلة-الانترفيو)

---

# مقدمة عن Variable Capture

## يعني إيه Variable Capture؟

الـ **Variable Capture** (أو **Closure**) هو قدرة الـ Lambda إنها توصل لـ variables من الـ scope اللي حواليها (enclosing scope).

```java
public void example() {
    String message = "Hello";  // Local variable
    
    // الـ Lambda بتـ "capture" الـ variable message
    Runnable r = () -> System.out.println(message);
    
    r.run();  // Hello
}
```

## ليه مهم نفهم Variable Capture؟

```java
// ❌ ده مش هيشتغل!
public void problematicExample() {
    int counter = 0;
    
    List<String> names = Arrays.asList("أحمد", "محمد", "علي");
    
    names.forEach(name -> {
        counter++;  // ❌ Compile Error!
        System.out.println(counter + ". " + name);
    });
}
// Error: Local variable counter defined in an enclosing scope 
// must be final or effectively final
```

**السبب:** الـ Lambda ممكن تتنفذ في وقت تاني أو thread تاني، فلازم القيم تكون ثابتة عشان نتجنب race conditions ومشاكل concurrency.

---

# أنواع المتغيرات في Lambda

## 1. Lambda Parameters

الـ Parameters بتاعة الـ Lambda نفسها - دي عادية ومفيش قيود عليها.

```java
// Single parameter
Consumer<String> printer = s -> System.out.println(s);
//                          ↑ parameter

// Multiple parameters
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
//                                           ↑  ↑ parameters

// With explicit types
Function<String, Integer> length = (String s) -> s.length();

// Parameters are mutable within lambda body
Consumer<StringBuilder> modifier = sb -> {
    sb.append("Modified");  // ✅ OK - modifying the object
    sb = new StringBuilder();  // ✅ OK - reassigning parameter
    // لكن ده مش هيأثر على الـ original reference
};
```

## 2. Local Variables (من الـ Enclosing Scope)

الـ Local variables من الـ method اللي فيها الـ Lambda - **لازم تكون effectively final**.

```java
public void localVariables() {
    // ✅ Effectively final - مش بيتغير
    String greeting = "Hello";
    
    // ✅ Explicitly final
    final int multiplier = 2;
    
    // ❌ مش effectively final - بيتغير
    int counter = 0;
    counter = 1;  // ده خلاه مش effectively final
    
    Consumer<String> greet = name -> {
        System.out.println(greeting + " " + name);  // ✅ OK
        System.out.println(multiplier * 10);        // ✅ OK
        // System.out.println(counter);             // ❌ Compile Error!
    };
}
```

## 3. Instance Variables (Fields)

الـ Instance variables (fields) - **مفيش قيود عليها**!

```java
public class InstanceVariableExample {
    private int instanceCounter = 0;  // Instance variable
    private String prefix = "Item";
    
    public void process() {
        List<String> items = Arrays.asList("A", "B", "C");
        
        items.forEach(item -> {
            instanceCounter++;           // ✅ OK - مفيش مشكلة!
            prefix = "Modified";         // ✅ OK - ممكن تغيره!
            System.out.println(prefix + " " + instanceCounter + ": " + item);
        });
        
        System.out.println("Total: " + instanceCounter);  // 3
    }
}
```

## 4. Static Variables

الـ Static variables - **مفيش قيود عليها** برضو!

```java
public class StaticVariableExample {
    private static int staticCounter = 0;
    private static String staticPrefix = "Global";
    
    public void process() {
        List<String> items = Arrays.asList("X", "Y", "Z");
        
        items.forEach(item -> {
            staticCounter++;              // ✅ OK
            staticPrefix = "Changed";     // ✅ OK
            System.out.println(staticPrefix + " " + staticCounter);
        });
    }
}
```

## جدول ملخص

| نوع المتغير | يمكن قراءته؟ | يمكن تعديله؟ | ملاحظات |
|-------------|--------------|--------------|---------|
| Lambda Parameter | ✅ | ✅ | عادي زي أي parameter |
| Local Variable | ✅ | ❌ | لازم effectively final |
| Instance Variable | ✅ | ✅ | مفيش قيود |
| Static Variable | ✅ | ✅ | مفيش قيود |

---

# Effectively Final بالتفصيل

## يعني إيه Effectively Final؟

الـ variable يكون **effectively final** لما:
1. مش معرف بـ `final` keyword
2. **بس** مش بيتغير بعد ما يتعرف (initialized)

```java
public void effectivelyFinalExamples() {
    // ✅ Effectively final - مش بيتغير أبداً
    int a = 10;
    
    // ✅ Explicitly final
    final int b = 20;
    
    // ❌ NOT effectively final - بيتغير
    int c = 30;
    c = 40;
    
    // ❌ NOT effectively final - بيتغير بعدين
    int d = 50;
    
    Runnable r = () -> {
        System.out.println(a);  // ✅ OK
        System.out.println(b);  // ✅ OK
        // System.out.println(c);  // ❌ Error - c مش effectively final
        // System.out.println(d);  // ❌ Error - d هيتغير بعدين
    };
    
    d = 60;  // ده خلى d مش effectively final
}
```

## حالات Effectively Final

### ✅ الحالات اللي بتشتغل

```java
public void validCases() {
    // Case 1: Simple assignment, never changed
    String name = "أحمد";
    Consumer<Void> c1 = v -> System.out.println(name);  // ✅
    
    // Case 2: Assigned from parameter
    public void greet(String greeting) {
        Consumer<String> c2 = name -> System.out.println(greeting + " " + name);  // ✅
    }
    
    // Case 3: Assigned from method return
    int length = "Hello".length();
    IntSupplier s1 = () -> length;  // ✅
    
    // Case 4: Assigned in single branch (if only one path)
    final int value;
    if (condition) {
        value = 10;
    } else {
        value = 20;
    }
    IntSupplier s2 = () -> value;  // ✅ - final variable with definite assignment
    
    // Case 5: Loop variable in enhanced for (each iteration is new)
    List<String> names = Arrays.asList("A", "B", "C");
    List<Runnable> tasks = new ArrayList<>();
    for (String n : names) {
        tasks.add(() -> System.out.println(n));  // ✅ - n is effectively final per iteration
    }
    
    // Case 6: Reference type - reference is final, content can change
    List<String> list = new ArrayList<>();
    Consumer<String> adder = item -> list.add(item);  // ✅
}
```

### ❌ الحالات اللي مش بتشتغل

```java
public void invalidCases() {
    // Case 1: Reassignment
    int x = 10;
    x = 20;  // ❌ مش effectively final
    // Runnable r = () -> System.out.println(x);  // Error!
    
    // Case 2: Increment/Decrement
    int counter = 0;
    counter++;  // ❌
    // Runnable r = () -> System.out.println(counter);  // Error!
    
    // Case 3: Compound assignment
    int sum = 0;
    sum += 10;  // ❌
    // Runnable r = () -> System.out.println(sum);  // Error!
    
    // Case 4: Assignment in loop
    int i = 0;
    while (i < 10) {
        i++;  // ❌
    }
    // Runnable r = () -> System.out.println(i);  // Error!
    
    // Case 5: Traditional for loop variable
    List<Runnable> tasks = new ArrayList<>();
    for (int j = 0; j < 5; j++) {  // j is modified each iteration
        // tasks.add(() -> System.out.println(j));  // ❌ Error!
    }
    
    // Case 6: Modified after lambda definition
    int value = 10;
    Runnable r = () -> System.out.println(value);  // Would be OK...
    value = 20;  // ❌ ...but this makes it not effectively final!
}
```

## ليه Java بتفرض Effectively Final؟

### السبب 1: Concurrency Safety

```java
public void concurrencyProblem() {
    int counter = 0;
    
    // لو سمحنا بده...
    Runnable task = () -> {
        // counter++;  // ❌ مش مسموح
    };
    
    // الـ Lambda ممكن تتنفذ في thread تاني
    new Thread(task).start();
    
    // وفي نفس الوقت الـ main thread ممكن يغير counter
    // counter = 100;
    
    // Race condition! إيه القيمة الصح؟
}
```

### السبب 2: Capture Semantics

```java
public void captureSemantics() {
    int value = 10;
    
    // الـ Lambda بتـ capture COPY من القيمة
    // مش reference للـ variable
    Runnable r = () -> System.out.println(value);
    
    // لو سمحنا بالتغيير، هيحصل confusion
    // value = 20;
    // الـ Lambda هتطبع 10 ولا 20؟
    
    r.run();
}
```

### السبب 3: Scope Lifetime

```java
public Runnable createRunnable() {
    int localValue = 42;  // Local variable
    
    // الـ Lambda ممكن تعيش أكتر من الـ method
    return () -> System.out.println(localValue);
    
    // بعد الـ method ترجع، localValue المفروض يتشال من الـ stack
    // بس الـ Lambda لسه محتاجاه!
    // الحل: Java بتعمل copy من القيمة في الـ Lambda
}

// استخدام
Runnable r = createRunnable();  // Method انتهت
r.run();  // بس الـ Lambda لسه شغالة! (بتستخدم الـ copy)
```

---

# الفرق بين Lambda و Anonymous Class

## 1. الفرق في `this`

```java
public class ThisDifference {
    private String name = "Outer";
    
    public void demonstrate() {
        // ===== Anonymous Inner Class =====
        // this = الـ Anonymous class نفسه
        Runnable anon = new Runnable() {
            private String name = "Anonymous";
            
            @Override
            public void run() {
                System.out.println(this.name);           // "Anonymous"
                System.out.println(ThisDifference.this.name);  // "Outer"
            }
        };
        
        // ===== Lambda Expression =====
        // this = الـ Enclosing class (ThisDifference)
        Runnable lambda = () -> {
            System.out.println(this.name);  // "Outer"
            // مفيش حاجة اسمها "lambda's this"
        };
        
        anon.run();    // Anonymous
        lambda.run();  // Outer
    }
}
```

## 2. الفرق في Shadowing

```java
public class ShadowingDifference {
    private int value = 100;
    
    public void demonstrate() {
        int value = 200;  // Local variable - shadows field
        
        // ===== Anonymous Inner Class =====
        // ممكن تعمل shadowing
        Runnable anon = new Runnable() {
            private int value = 300;  // ✅ Shadows outer
            
            @Override
            public void run() {
                System.out.println(value);       // 300 (own field)
                System.out.println(this.value);  // 300 (own field)
            }
        };
        
        // ===== Lambda Expression =====
        // مش ممكن تعمل shadowing للـ local variables
        Runnable lambda = () -> {
            // int value = 400;  // ❌ Compile Error! Can't shadow
            System.out.println(value);       // 200 (local variable)
            System.out.println(this.value);  // 100 (outer field)
        };
        
        anon.run();    // 300
        lambda.run();  // 200
    }
}
```

## 3. الفرق في Variable Capture

```java
public class CaptureDifference {
    
    public void demonstrate() {
        int localVar = 10;
        
        // ===== Anonymous Inner Class =====
        // نفس القاعدة - لازم effectively final
        Runnable anon = new Runnable() {
            @Override
            public void run() {
                System.out.println(localVar);  // ✅ OK if effectively final
                // localVar = 20;  // ❌ Can't modify
            }
        };
        
        // ===== Lambda Expression =====
        // نفس القاعدة بالظبط
        Runnable lambda = () -> {
            System.out.println(localVar);  // ✅ OK if effectively final
            // localVar = 20;  // ❌ Can't modify
        };
        
        // الاتنين بيتصرفوا زي بعض في الموضوع ده
    }
}
```

## 4. الفرق في الـ Scope

```java
public class ScopeDifference {
    
    public void demonstrate() {
        String outerVar = "outer";
        
        // ===== Anonymous Inner Class =====
        // عنده scope خاص بيه
        Runnable anon = new Runnable() {
            String innerVar = "inner";  // Own variable
            
            @Override
            public void run() {
                System.out.println(outerVar);  // Access outer
                System.out.println(innerVar);  // Access own
            }
        };
        
        // ===== Lambda Expression =====
        // مفيش scope خاص - بيشارك نفس الـ scope
        Runnable lambda = () -> {
            // String outerVar = "shadow";  // ❌ Can't shadow!
            System.out.println(outerVar);  // Access outer
        };
    }
}
```

## جدول المقارنة

| الخاصية | Anonymous Class | Lambda |
|---------|-----------------|--------|
| `this` | يشير للـ Anonymous class | يشير للـ Enclosing class |
| Shadowing | ✅ ممكن | ❌ مش ممكن |
| Own fields | ✅ ممكن | ❌ مش ممكن |
| Own scope | ✅ نعم | ❌ لا (يشارك الـ enclosing scope) |
| Effectively final | ✅ مطلوب | ✅ مطلوب |
| Multiple methods | ✅ ممكن | ❌ SAM only |

---

# Closures في Java

## يعني إيه Closure؟

الـ **Closure** هو function (أو Lambda) بتـ "تقفل على" (close over) variables من الـ scope اللي اتعرفت فيه.

```java
public class ClosureExample {
    
    public Function<Integer, Integer> createMultiplier(int factor) {
        // factor هو local variable
        
        // الـ Lambda بتـ "close over" factor
        return x -> x * factor;
        
        // حتى بعد الـ method ترجع، الـ Lambda فاكرة factor
    }
    
    public void demonstrate() {
        Function<Integer, Integer> triple = createMultiplier(3);
        Function<Integer, Integer> double_ = createMultiplier(2);
        
        System.out.println(triple.apply(5));   // 15
        System.out.println(double_.apply(5));  // 10
        
        // كل Lambda فاكرة الـ factor بتاعها!
    }
}
```

## Closure مع State

```java
public class ClosureWithState {
    
    // ✅ Using instance variable for mutable state
    public Consumer<String> createAccumulator() {
        StringBuilder accumulated = new StringBuilder();  // Local but reference type
        
        return item -> {
            accumulated.append(item).append(", ");
            System.out.println("Accumulated: " + accumulated);
        };
    }
    
    // ✅ Using wrapper for counter
    public Consumer<String> createCounter() {
        int[] counter = {0};  // Array wrapper trick
        
        return item -> {
            counter[0]++;
            System.out.println(counter[0] + ". " + item);
        };
    }
    
    public void demonstrate() {
        Consumer<String> accumulator = createAccumulator();
        accumulator.accept("A");  // Accumulated: A, 
        accumulator.accept("B");  // Accumulated: A, B, 
        accumulator.accept("C");  // Accumulated: A, B, C, 
        
        Consumer<String> counter = createCounter();
        counter.accept("X");  // 1. X
        counter.accept("Y");  // 2. Y
        counter.accept("Z");  // 3. Z
    }
}
```

## Closure في Factory Pattern

```java
public class ClosureFactory {
    
    // Predicate factory
    public static Predicate<Integer> greaterThan(int threshold) {
        return n -> n > threshold;  // Closes over threshold
    }
    
    public static Predicate<Integer> between(int min, int max) {
        return n -> n >= min && n <= max;  // Closes over min and max
    }
    
    // Function factory
    public static Function<String, String> prefixer(String prefix) {
        return s -> prefix + s;  // Closes over prefix
    }
    
    public static Function<String, String> wrapper(String before, String after) {
        return s -> before + s + after;  // Closes over before and after
    }
    
    // Consumer factory
    public static Consumer<String> logger(String tag) {
        return msg -> System.out.println("[" + tag + "] " + msg);
    }
    
    public static void main(String[] args) {
        // Using the factories
        Predicate<Integer> above10 = greaterThan(10);
        Predicate<Integer> between5And15 = between(5, 15);
        
        System.out.println(above10.test(15));      // true
        System.out.println(between5And15.test(7)); // true
        
        Function<String, String> addMr = prefixer("Mr. ");
        Function<String, String> htmlBold = wrapper("<b>", "</b>");
        
        System.out.println(addMr.apply("أحمد"));        // Mr. أحمد
        System.out.println(htmlBold.apply("Important")); // <b>Important</b>
        
        Consumer<String> infoLog = logger("INFO");
        Consumer<String> errorLog = logger("ERROR");
        
        infoLog.accept("Started");   // [INFO] Started
        errorLog.accept("Failed!");  // [ERROR] Failed!
    }
}
```

---

# مشاكل شائعة وحلولها

## المشكلة 1: Counter في Loop

### ❌ المشكلة

```java
public void problem1() {
    List<String> names = Arrays.asList("أحمد", "محمد", "علي");
    int counter = 0;
    
    names.forEach(name -> {
        counter++;  // ❌ Compile Error!
        System.out.println(counter + ". " + name);
    });
}
```

### ✅ الحل 1: Array Wrapper

```java
public void solution1a() {
    List<String> names = Arrays.asList("أحمد", "محمد", "علي");
    int[] counter = {0};  // Array wrapper
    
    names.forEach(name -> {
        counter[0]++;  // ✅ OK - نغير content مش reference
        System.out.println(counter[0] + ". " + name);
    });
}
```

### ✅ الحل 2: AtomicInteger

```java
public void solution1b() {
    List<String> names = Arrays.asList("أحمد", "محمد", "علي");
    AtomicInteger counter = new AtomicInteger(0);
    
    names.forEach(name -> {
        int index = counter.incrementAndGet();  // ✅ Thread-safe!
        System.out.println(index + ". " + name);
    });
}
```

### ✅ الحل 3: IntStream with index

```java
public void solution1c() {
    List<String> names = Arrays.asList("أحمد", "محمد", "علي");
    
    IntStream.range(0, names.size())
        .forEach(i -> System.out.println((i + 1) + ". " + names.get(i)));
}
```

### ✅ الحل 4: Traditional for loop

```java
public void solution1d() {
    List<String> names = Arrays.asList("أحمد", "محمد", "علي");
    
    for (int i = 0; i < names.size(); i++) {
        System.out.println((i + 1) + ". " + names.get(i));
    }
}
```

---

## المشكلة 2: Loop Variable Capture

### ❌ المشكلة

```java
public void problem2() {
    List<Runnable> tasks = new ArrayList<>();
    
    for (int i = 0; i < 5; i++) {
        tasks.add(() -> System.out.println(i));  // ❌ Compile Error!
    }
    // i is modified each iteration, not effectively final
}
```

### ✅ الحل 1: Final copy inside loop

```java
public void solution2a() {
    List<Runnable> tasks = new ArrayList<>();
    
    for (int i = 0; i < 5; i++) {
        final int index = i;  // Create final copy
        tasks.add(() -> System.out.println(index));  // ✅ OK
    }
    
    tasks.forEach(Runnable::run);  // 0, 1, 2, 3, 4
}
```

### ✅ الحل 2: Enhanced for loop

```java
public void solution2b() {
    List<Runnable> tasks = new ArrayList<>();
    List<Integer> numbers = Arrays.asList(0, 1, 2, 3, 4);
    
    for (Integer num : numbers) {  // num is effectively final per iteration
        tasks.add(() -> System.out.println(num));  // ✅ OK
    }
    
    tasks.forEach(Runnable::run);  // 0, 1, 2, 3, 4
}
```

### ✅ الحل 3: IntStream

```java
public void solution2c() {
    List<Runnable> tasks = IntStream.range(0, 5)
        .mapToObj(i -> (Runnable) () -> System.out.println(i))
        .collect(Collectors.toList());
    
    tasks.forEach(Runnable::run);  // 0, 1, 2, 3, 4
}
```

---

## المشكلة 3: Accumulating Results

### ❌ المشكلة

```java
public void problem3() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    int sum = 0;
    
    numbers.forEach(n -> {
        sum += n;  // ❌ Compile Error!
    });
}
```

### ✅ الحل 1: Use reduce()

```java
public void solution3a() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    
    int sum = numbers.stream()
        .reduce(0, Integer::sum);
    
    System.out.println(sum);  // 15
}
```

### ✅ الحل 2: Use sum() for primitives

```java
public void solution3b() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    
    int sum = numbers.stream()
        .mapToInt(Integer::intValue)
        .sum();
    
    System.out.println(sum);  // 15
}
```

### ✅ الحل 3: AtomicInteger (إذا لازم forEach)

```java
public void solution3c() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    AtomicInteger sum = new AtomicInteger(0);
    
    numbers.forEach(n -> sum.addAndGet(n));
    
    System.out.println(sum.get());  // 15
}
```

### ✅ الحل 4: Array wrapper

```java
public void solution3d() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    int[] sum = {0};
    
    numbers.forEach(n -> sum[0] += n);
    
    System.out.println(sum[0]);  // 15
}
```

---

## المشكلة 4: Building a String

### ❌ المشكلة

```java
public void problem4() {
    List<String> words = Arrays.asList("Hello", "World", "Java");
    String result = "";
    
    words.forEach(word -> {
        result += word + " ";  // ❌ Compile Error!
    });
}
```

### ✅ الحل 1: StringBuilder

```java
public void solution4a() {
    List<String> words = Arrays.asList("Hello", "World", "Java");
    StringBuilder result = new StringBuilder();
    
    words.forEach(word -> result.append(word).append(" "));
    
    System.out.println(result.toString().trim());  // Hello World Java
}
```

### ✅ الحل 2: String.join()

```java
public void solution4b() {
    List<String> words = Arrays.asList("Hello", "World", "Java");
    
    String result = String.join(" ", words);
    
    System.out.println(result);  // Hello World Java
}
```

### ✅ الحل 3: Collectors.joining()

```java
public void solution4c() {
    List<String> words = Arrays.asList("Hello", "World", "Java");
    
    String result = words.stream()
        .collect(Collectors.joining(" "));
    
    System.out.println(result);  // Hello World Java
}
```

### ✅ الحل 4: reduce()

```java
public void solution4d() {
    List<String> words = Arrays.asList("Hello", "World", "Java");
    
    String result = words.stream()
        .reduce("", (a, b) -> a.isEmpty() ? b : a + " " + b);
    
    System.out.println(result);  // Hello World Java
}
```

---

## المشكلة 5: Conditional Flag

### ❌ المشكلة

```java
public void problem5() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    boolean found = false;
    
    numbers.forEach(n -> {
        if (n > 3) {
            found = true;  // ❌ Compile Error!
        }
    });
}
```

### ✅ الحل 1: anyMatch()

```java
public void solution5a() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    
    boolean found = numbers.stream()
        .anyMatch(n -> n > 3);
    
    System.out.println(found);  // true
}
```

### ✅ الحل 2: AtomicBoolean

```java
public void solution5b() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    AtomicBoolean found = new AtomicBoolean(false);
    
    numbers.forEach(n -> {
        if (n > 3) {
            found.set(true);
        }
    });
    
    System.out.println(found.get());  // true
}
```

### ✅ الحل 3: boolean[] wrapper

```java
public void solution5c() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    boolean[] found = {false};
    
    numbers.forEach(n -> {
        if (n > 3) {
            found[0] = true;
        }
    });
    
    System.out.println(found[0]);  // true
}
```

---

## المشكلة 6: Finding Maximum/Minimum

### ❌ المشكلة

```java
public void problem6() {
    List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 9);
    int max = Integer.MIN_VALUE;
    
    numbers.forEach(n -> {
        if (n > max) {
            max = n;  // ❌ Compile Error!
        }
    });
}
```

### ✅ الحل 1: max() / min()

```java
public void solution6a() {
    List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 9);
    
    int max = numbers.stream()
        .max(Integer::compareTo)
        .orElse(Integer.MIN_VALUE);
    
    int min = numbers.stream()
        .min(Integer::compareTo)
        .orElse(Integer.MAX_VALUE);
    
    System.out.println("Max: " + max);  // 9
    System.out.println("Min: " + min);  // 1
}
```

### ✅ الحل 2: reduce()

```java
public void solution6b() {
    List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 9);
    
    int max = numbers.stream()
        .reduce(Integer.MIN_VALUE, Integer::max);
    
    int min = numbers.stream()
        .reduce(Integer.MAX_VALUE, Integer::min);
    
    System.out.println("Max: " + max);  // 9
    System.out.println("Min: " + min);  // 1
}
```

### ✅ الحل 3: IntStream

```java
public void solution6c() {
    List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 9);
    
    IntSummaryStatistics stats = numbers.stream()
        .mapToInt(Integer::intValue)
        .summaryStatistics();
    
    System.out.println("Max: " + stats.getMax());    // 9
    System.out.println("Min: " + stats.getMin());    // 1
    System.out.println("Sum: " + stats.getSum());    // 23
    System.out.println("Avg: " + stats.getAverage()); // 3.83...
}
```

---

## المشكلة 7: Multiple Values

### ❌ المشكلة

```java
public void problem7() {
    List<String> words = Arrays.asList("apple", "banana", "cherry");
    int count = 0;
    int totalLength = 0;
    
    words.forEach(word -> {
        count++;           // ❌ Error!
        totalLength += word.length();  // ❌ Error!
    });
}
```

### ✅ الحل 1: Custom accumulator class

```java
public void solution7a() {
    List<String> words = Arrays.asList("apple", "banana", "cherry");
    
    class Accumulator {
        int count = 0;
        int totalLength = 0;
    }
    
    Accumulator acc = new Accumulator();
    
    words.forEach(word -> {
        acc.count++;
        acc.totalLength += word.length();
    });
    
    System.out.println("Count: " + acc.count);           // 3
    System.out.println("Total Length: " + acc.totalLength); // 17
}
```

### ✅ الحل 2: reduce with custom object

```java
public void solution7b() {
    List<String> words = Arrays.asList("apple", "banana", "cherry");
    
    record Stats(int count, int totalLength) {
        Stats add(String word) {
            return new Stats(count + 1, totalLength + word.length());
        }
        Stats combine(Stats other) {
            return new Stats(count + other.count, totalLength + other.totalLength);
        }
    }
    
    Stats result = words.stream()
        .reduce(
            new Stats(0, 0),
            (stats, word) -> stats.add(word),
            Stats::combine
        );
    
    System.out.println("Count: " + result.count());           // 3
    System.out.println("Total Length: " + result.totalLength()); // 17
}
```

### ✅ الحل 3: Array wrappers

```java
public void solution7c() {
    List<String> words = Arrays.asList("apple", "banana", "cherry");
    
    int[] count = {0};
    int[] totalLength = {0};
    
    words.forEach(word -> {
        count[0]++;
        totalLength[0] += word.length();
    });
    
    System.out.println("Count: " + count[0]);           // 3
    System.out.println("Total Length: " + totalLength[0]); // 17
}
```

---

# Best Practices

## 1. فضّل Stream Operations على Mutating State

```java
// ❌ Bad - Mutating state
public int sumBad(List<Integer> numbers) {
    int[] sum = {0};
    numbers.forEach(n -> sum[0] += n);
    return sum[0];
}

// ✅ Good - Functional approach
public int sumGood(List<Integer> numbers) {
    return numbers.stream()
        .mapToInt(Integer::intValue)
        .sum();
}
```

## 2. استخدم Atomic Classes للـ Thread Safety

```java
// ❌ Bad - Not thread-safe
public void processParallelBad(List<String> items) {
    int[] counter = {0};
    items.parallelStream().forEach(item -> {
        counter[0]++;  // Race condition!
        process(item);
    });
}

// ✅ Good - Thread-safe
public void processParallelGood(List<String> items) {
    AtomicInteger counter = new AtomicInteger(0);
    items.parallelStream().forEach(item -> {
        counter.incrementAndGet();  // Thread-safe
        process(item);
    });
}

// ✅ Better - Use count()
public long processParallelBetter(List<String> items) {
    return items.parallelStream()
        .peek(this::process)
        .count();
}
```

## 3. تجنب Side Effects في Lambdas

```java
// ❌ Bad - Side effects
public List<String> processBad(List<String> items) {
    List<String> results = new ArrayList<>();
    items.stream()
        .filter(s -> s.length() > 3)
        .forEach(s -> results.add(s.toUpperCase()));  // Side effect!
    return results;
}

// ✅ Good - Pure functional
public List<String> processGood(List<String> items) {
    return items.stream()
        .filter(s -> s.length() > 3)
        .map(String::toUpperCase)
        .collect(Collectors.toList());
}
```

## 4. استخدم Method References لما يكون الكود أوضح

```java
// ✅ Method reference - clearer
list.forEach(System.out::println);
list.stream().map(String::toUpperCase);

// ❌ Lambda - unnecessary verbosity
list.forEach(s -> System.out.println(s));
list.stream().map(s -> s.toUpperCase());
```

## 5. خلي الـ Lambdas قصيرة

```java
// ❌ Bad - Too long
items.stream()
    .filter(item -> {
        boolean valid = item != null;
        if (valid) {
            valid = !item.isEmpty();
        }
        if (valid) {
            valid = item.length() > 3;
        }
        return valid;
    })
    .collect(Collectors.toList());

// ✅ Good - Extract to method
items.stream()
    .filter(this::isValidItem)
    .collect(Collectors.toList());

private boolean isValidItem(String item) {
    return item != null && !item.isEmpty() && item.length() > 3;
}

// ✅ Or use Predicate composition
Predicate<String> notNull = Objects::nonNull;
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> longEnough = s -> s.length() > 3;

items.stream()
    .filter(notNull.and(notEmpty).and(longEnough))
    .collect(Collectors.toList());
```

## 6. Document Captured Variables

```java
// ✅ Good - Clear what's captured
public Function<Integer, Integer> createMultiplier(int factor) {
    // factor is captured - must be effectively final
    return x -> x * factor;
}

// ✅ Good - Document mutable state
public Consumer<String> createLogger(String prefix) {
    // Using StringBuilder for mutable accumulation
    StringBuilder log = new StringBuilder();
    
    return message -> {
        log.append(prefix).append(": ").append(message).append("\n");
        System.out.print(log);
    };
}
```

---

# أسئلة الانترفيو

## أسئلة نظرية

### س1: يعني إيه Effectively Final؟

**الإجابة:**
الـ variable يكون **effectively final** لما مش معرف بـ `final` keyword، **بس** مش بيتغير بعد ما يتعرف.

```java
// ✅ Effectively final
int x = 10;  // معرف ومش هيتغير

// ❌ Not effectively final
int y = 10;
y = 20;  // اتغير!

// في الـ Lambda
Runnable r = () -> {
    System.out.println(x);  // ✅ OK
    // System.out.println(y);  // ❌ Error
};
```

**السبب:** الـ Lambda بتـ capture **copy** من القيمة. لو القيمة ممكن تتغير، هيحصل confusion إيه القيمة الصح.

---

### س2: ليه الـ Local Variables لازم تكون Effectively Final في Lambda؟

**الإجابة:**

**السبب 1: Capture Semantics**
```java
int value = 10;
Runnable r = () -> System.out.println(value);
// الـ Lambda بتاخد COPY من value
// لو سمحنا بالتغيير، الـ copy مش هتتغير
```

**السبب 2: Concurrency Safety**
```java
int counter = 0;
Runnable r = () -> counter++;  // لو سمحنا...

new Thread(r).start();  // Thread 1
new Thread(r).start();  // Thread 2
// Race condition!
```

**السبب 3: Scope Lifetime**
```java
Runnable createTask() {
    int local = 42;
    return () -> System.out.println(local);
    // بعد الـ return، local المفروض يتشال من Stack
    // بس الـ Lambda محتاجاه!
}
```

---

### س3: إيه الفرق في `this` بين Lambda و Anonymous Class؟

**الإجابة:**

```java
public class Example {
    String name = "Outer";
    
    void test() {
        // Anonymous Class - this = الـ anonymous instance
        Runnable anon = new Runnable() {
            String name = "Anon";
            public void run() {
                System.out.println(this.name);  // "Anon"
            }
        };
        
        // Lambda - this = الـ enclosing class
        Runnable lambda = () -> {
            System.out.println(this.name);  // "Outer"
        };
    }
}
```

---

### س4: إزاي تتعامل مع Counter في Lambda؟

**الإجابة:**

```java
// ❌ مش هيشتغل
int counter = 0;
list.forEach(item -> counter++);

// ✅ الحل 1: Array wrapper
int[] counter = {0};
list.forEach(item -> counter[0]++);

// ✅ الحل 2: AtomicInteger (thread-safe)
AtomicInteger counter = new AtomicInteger(0);
list.forEach(item -> counter.incrementAndGet());

// ✅ الحل 3: count() method
long count = list.stream().count();

// ✅ الحل 4: IntStream with index
IntStream.range(0, list.size())
    .forEach(i -> System.out.println((i+1) + ". " + list.get(i)));
```

---

### س5: إيه الفرق بين Instance Variables و Local Variables في Lambda؟

**الإجابة:**

| Instance Variables | Local Variables |
|-------------------|-----------------|
| ممكن تتقرأ وتتعدل | ممكن تتقرا بس |
| مفيش قيود | لازم effectively final |
| موجودة في Heap | موجودة في Stack |
| الـ Lambda بتوصلها عن طريق `this` | الـ Lambda بتاخد copy |

```java
class Example {
    int instanceVar = 0;  // Instance variable
    
    void method() {
        int localVar = 0;  // Local variable
        
        Runnable r = () -> {
            instanceVar++;  // ✅ OK
            // localVar++;  // ❌ Error
        };
    }
}
```

---

### س6: يعني إيه Closure في Java؟

**الإجابة:**
الـ **Closure** هو Lambda (أو function) بتـ "تقفل على" variables من الـ scope اللي اتعرفت فيه.

```java
Function<Integer, Integer> createMultiplier(int factor) {
    // factor is "closed over" by the lambda
    return x -> x * factor;
}

Function<Integer, Integer> triple = createMultiplier(3);
Function<Integer, Integer> double_ = createMultiplier(2);

triple.apply(5);   // 15 - factor = 3
double_.apply(5);  // 10 - factor = 2
```

كل Lambda فاكرة الـ `factor` بتاعها حتى بعد الـ method ترجع.

---

### س7: إمتى نستخدم AtomicInteger بدل int[] wrapper؟

**الإجابة:**

```java
// int[] wrapper - للـ single-threaded scenarios
int[] counter = {0};
list.forEach(item -> counter[0]++);  // ✅ OK for sequential stream

// AtomicInteger - للـ multi-threaded scenarios
AtomicInteger counter = new AtomicInteger(0);
list.parallelStream().forEach(item -> counter.incrementAndGet());  // ✅ Thread-safe
```

**القاعدة:**
- `int[]` → Sequential streams, single thread
- `AtomicInteger` → Parallel streams, multiple threads

---

## أسئلة كود

### س8: صحح الكود ده

```java
// ❌ الكود الغلط
public void process(List<String> items) {
    int total = 0;
    String result = "";
    boolean found = false;
    
    items.forEach(item -> {
        total += item.length();
        result += item + ",";
        if (item.startsWith("A")) {
            found = true;
        }
    });
    
    System.out.println("Total: " + total);
    System.out.println("Result: " + result);
    System.out.println("Found: " + found);
}
```

**الإجابة:**

```java
// ✅ الحل 1: Using Stream operations
public void processSolution1(List<String> items) {
    int total = items.stream()
        .mapToInt(String::length)
        .sum();
    
    String result = items.stream()
        .collect(Collectors.joining(","));
    
    boolean found = items.stream()
        .anyMatch(item -> item.startsWith("A"));
    
    System.out.println("Total: " + total);
    System.out.println("Result: " + result);
    System.out.println("Found: " + found);
}

// ✅ الحل 2: Using wrappers
public void processSolution2(List<String> items) {
    int[] total = {0};
    StringBuilder result = new StringBuilder();
    boolean[] found = {false};
    
    items.forEach(item -> {
        total[0] += item.length();
        result.append(item).append(",");
        if (item.startsWith("A")) {
            found[0] = true;
        }
    });
    
    System.out.println("Total: " + total[0]);
    System.out.println("Result: " + result);
    System.out.println("Found: " + found[0]);
}

// ✅ الحل 3: Using a custom accumulator
public void processSolution3(List<String> items) {
    class Accumulator {
        int total = 0;
        StringBuilder result = new StringBuilder();
        boolean found = false;
    }
    
    Accumulator acc = new Accumulator();
    
    items.forEach(item -> {
        acc.total += item.length();
        acc.result.append(item).append(",");
        if (item.startsWith("A")) {
            acc.found = true;
        }
    });
    
    System.out.println("Total: " + acc.total);
    System.out.println("Result: " + acc.result);
    System.out.println("Found: " + acc.found);
}
```

---

### س9: اكتب Closure Factory للـ Validators

```java
public class ValidatorFactory {
    
    // String length validator
    public static Predicate<String> minLength(int min) {
        return s -> s != null && s.length() >= min;
    }
    
    public static Predicate<String> maxLength(int max) {
        return s -> s != null && s.length() <= max;
    }
    
    public static Predicate<String> lengthBetween(int min, int max) {
        return minLength(min).and(maxLength(max));
    }
    
    // Pattern validator
    public static Predicate<String> matches(String regex) {
        Pattern pattern = Pattern.compile(regex);  // Compile once
        return s -> s != null && pattern.matcher(s).matches();
    }
    
    // Contains validator
    public static Predicate<String> contains(String substring) {
        return s -> s != null && s.contains(substring);
    }
    
    // Number range validator
    public static Predicate<Integer> between(int min, int max) {
        return n -> n >= min && n <= max;
    }
    
    // Combine validators
    @SafeVarargs
    public static <T> Predicate<T> allOf(Predicate<T>... predicates) {
        return Arrays.stream(predicates)
            .reduce(t -> true, Predicate::and);
    }
    
    public static void main(String[] args) {
        // Password validator
        Predicate<String> validPassword = allOf(
            minLength(8),
            maxLength(20),
            matches(".*[A-Z].*"),  // Has uppercase
            matches(".*[a-z].*"),  // Has lowercase
            matches(".*[0-9].*")   // Has digit
        );
        
        System.out.println(validPassword.test("Password1"));   // true
        System.out.println(validPassword.test("weak"));        // false
        
        // Email validator
        Predicate<String> validEmail = allOf(
            minLength(5),
            contains("@"),
            matches(".*@.*\\..*")
        );
        
        System.out.println(validEmail.test("test@example.com")); // true
        System.out.println(validEmail.test("invalid"));          // false
        
        // Age validator
        Predicate<Integer> validAge = between(0, 150);
        
        System.out.println(validAge.test(25));   // true
        System.out.println(validAge.test(-5));   // false
        System.out.println(validAge.test(200));  // false
    }
}
```

---

### س10: حل مشكلة Loop Variable Capture

```java
// ❌ المشكلة
public List<Supplier<Integer>> createSuppliersBad() {
    List<Supplier<Integer>> suppliers = new ArrayList<>();
    
    for (int i = 0; i < 5; i++) {
        suppliers.add(() -> i);  // ❌ Error: i is not effectively final
    }
    
    return suppliers;
}
```

**الإجابة:**

```java
// ✅ الحل 1: Final copy
public List<Supplier<Integer>> createSuppliers1() {
    List<Supplier<Integer>> suppliers = new ArrayList<>();
    
    for (int i = 0; i < 5; i++) {
        final int value = i;  // Final copy for each iteration
        suppliers.add(() -> value);
    }
    
    return suppliers;
}

// ✅ الحل 2: IntStream
public List<Supplier<Integer>> createSuppliers2() {
    return IntStream.range(0, 5)
        .mapToObj(i -> (Supplier<Integer>) () -> i)
        .collect(Collectors.toList());
}

// ✅ الحل 3: Enhanced for with list
public List<Supplier<Integer>> createSuppliers3() {
    List<Supplier<Integer>> suppliers = new ArrayList<>();
    
    for (Integer i : Arrays.asList(0, 1, 2, 3, 4)) {
        // i is effectively final per iteration
        suppliers.add(() -> i);
    }
    
    return suppliers;
}

// Test
public static void main(String[] args) {
    List<Supplier<Integer>> suppliers = createSuppliers1();
    
    suppliers.forEach(s -> System.out.println(s.get()));
    // 0, 1, 2, 3, 4
}
```

---

### س11: اكتب Thread-Safe Counter باستخدام Lambda

```java
public class ThreadSafeCounter {
    
    // ✅ Using AtomicInteger
    public void countWithAtomic(List<String> items) {
        AtomicInteger counter = new AtomicInteger(0);
        AtomicInteger total = new AtomicInteger(0);
        
        items.parallelStream().forEach(item -> {
            counter.incrementAndGet();
            total.addAndGet(item.length());
        });
        
        System.out.println("Count: " + counter.get());
        System.out.println("Total length: " + total.get());
    }
    
    // ✅ Using LongAdder (better for high contention)
    public void countWithAdder(List<String> items) {
        LongAdder counter = new LongAdder();
        LongAdder total = new LongAdder();
        
        items.parallelStream().forEach(item -> {
            counter.increment();
            total.add(item.length());
        });
        
        System.out.println("Count: " + counter.sum());
        System.out.println("Total length: " + total.sum());
    }
    
    // ✅ Better: Using Stream operations (no mutable state)
    public void countFunctional(List<String> items) {
        long count = items.parallelStream().count();
        
        int total = items.parallelStream()
            .mapToInt(String::length)
            .sum();
        
        System.out.println("Count: " + count);
        System.out.println("Total length: " + total);
    }
    
    // ✅ Using reduce for complex accumulation
    public void countWithReduce(List<String> items) {
        record Stats(long count, int totalLength) {
            Stats add(String item) {
                return new Stats(count + 1, totalLength + item.length());
            }
            Stats combine(Stats other) {
                return new Stats(count + other.count, totalLength + other.totalLength);
            }
        }
        
        Stats result = items.parallelStream()
            .reduce(
                new Stats(0, 0),
                (stats, item) -> stats.add(item),
                Stats::combine
            );
        
        System.out.println("Count: " + result.count());
        System.out.println("Total length: " + result.totalLength());
    }
}
```

---

### س12: شرح الفرق في الـ Output

```java
public class OutputDifference {
    private String field = "Field";
    
    public void test() {
        String local = "Local";
        
        // Case 1: Anonymous class
        new Thread(new Runnable() {
            private String field = "Inner Field";
            
            @Override
            public void run() {
                System.out.println("1: " + this.field);
                System.out.println("2: " + OutputDifference.this.field);
                System.out.println("3: " + local);
            }
        }).start();
        
        // Case 2: Lambda
        new Thread(() -> {
            System.out.println("4: " + this.field);
            System.out.println("5: " + field);
            System.out.println("6: " + local);
        }).start();
    }
}
```

**الإجابة:**

```
Output:
1: Inner Field    ← Anonymous class's own field
2: Field          ← Outer class's field
3: Local          ← Captured local variable

4: Field          ← this refers to OutputDifference
5: Field          ← same as this.field
6: Local          ← Captured local variable
```

**الشرح:**
- **Anonymous class:** `this` يشير للـ anonymous instance، عشان كده `this.field` = "Inner Field"
- **Lambda:** `this` يشير للـ enclosing class (OutputDifference)، عشان كده `this.field` = "Field"
- **local** في الحالتين هو الـ captured local variable

---

## نصائح للانترفيو 💡

1. **افهم Effectively Final كويس** - ده سؤال شائع جداً
2. **اعرف الفرق في `this`** بين Lambda و Anonymous Class
3. **اتدرب على الـ Workarounds** (Array wrapper, AtomicInteger, etc.)
4. **فهم ليه Java بتفرض القيود دي** (Concurrency, Capture semantics)
5. **فضّل Stream operations** على mutating state
6. **اعرف إمتى تستخدم Atomic classes** للـ thread safety
7. **افهم Closures** وإزاي تعمل Factory methods

---

## موارد إضافية 📚

- [Oracle Lambda Expressions Tutorial](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
- [JLS - Lambda Expressions](https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.27)
- [Effective Java - Item 42: Prefer lambdas to anonymous classes](https://www.oreilly.com/library/view/effective-java/9780134686097/)

---

*تم إعداد هذا الدليل بالعامية المصرية عشان يكون سهل الفهم والمراجعة* 🇪🇬
