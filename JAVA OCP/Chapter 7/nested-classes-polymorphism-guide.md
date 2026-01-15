# 📦 Nested Classes & Polymorphism في Java - دليل شامل بالمصري

## المحتويات
- [الجزء الأول: Nested Classes](#الجزء-الأول-nested-classes)
  - [أنواع الـ Nested Classes](#أنواع-الـ-nested-classes)
  - [Static Nested Class](#1-static-nested-class)
  - [Member Inner Class](#2-member-inner-class)
  - [Local Inner Class](#3-local-inner-class)
  - [Anonymous Inner Class](#4-anonymous-inner-class)
- [الجزء الثاني: Polymorphism](#الجزء-الثاني-polymorphism)
  - [Compile-Time Polymorphism](#1-compile-time-polymorphism-method-overloading)
  - [Runtime Polymorphism](#2-runtime-polymorphism-method-overriding)
  - [Upcasting و Downcasting](#upcasting-و-downcasting)
  - [Polymorphism مع Interfaces](#polymorphism-مع-interfaces)
- [أسئلة الانترفيو](#أسئلة-الانترفيو)

---

# الجزء الأول: Nested Classes

## مقدمة

### يعني إيه Nested Class؟

الـ **Nested Class** هو class معرف جوه class تاني. الفكرة إنك بتجمع classes مع بعض لما يكونوا مرتبطين منطقياً.

```java
class OuterClass {
    // ده الـ Nested Class
    class InnerClass {
        
    }
}
```

### ليه نستخدم Nested Classes؟

1. **Logical Grouping**: تجميع classes مرتبطة ببعض
2. **Encapsulation**: إخفاء implementation details
3. **Readability**: كود أسهل في القراءة والصيانة
4. **Access**: الـ Inner class يقدر يوصل لـ private members

---

## أنواع الـ Nested Classes

```
Nested Classes
├── Static Nested Class
└── Non-Static (Inner Classes)
    ├── Member Inner Class
    ├── Local Inner Class
    └── Anonymous Inner Class
```

---

## 1. Static Nested Class

### التعريف

ده class عادي بس معرف جوه class تاني ومعاه كلمة `static`. مش محتاج instance من الـ Outer Class عشان تستخدمه.

```java
public class University {
    private static String universityName = "جامعة القاهرة";
    private String dean = "د. أحمد";
    private static int totalStudents = 50000;
    
    // Static Nested Class
    public static class Department {
        private String name;
        private int studentCount;
        
        public Department(String name, int studentCount) {
            this.name = name;
            this.studentCount = studentCount;
        }
        
        public void displayInfo() {
            // ✅ يقدر يوصل للـ static members (حتى private)
            System.out.println("الجامعة: " + universityName);
            System.out.println("إجمالي الطلاب: " + totalStudents);
            System.out.println("القسم: " + name);
            System.out.println("طلاب القسم: " + studentCount);
            
            // ❌ مش يقدر يوصل للـ instance members
            // System.out.println(dean);  // Compile Error!
        }
        
        public static void showUniversityName() {
            System.out.println(universityName);
        }
    }
}
```

### الاستخدام

```java
// إنشاء instance - مش محتاج instance من University
University.Department cs = new University.Department("علوم الحاسب", 500);
cs.displayInfo();

// استدعاء static method
University.Department.showUniversityName();

// Import للتبسيط
import com.example.University.Department;
Department math = new Department("رياضيات", 300);
```

### خصائص الـ Static Nested Class

| الخاصية | القيمة |
|---------|--------|
| Access to outer static members | ✅ (حتى private) |
| Access to outer instance members | ❌ |
| Can have static members | ✅ |
| Can have instance members | ✅ |
| Needs outer instance | ❌ |
| Can be public/private/protected | ✅ |

### متى تستخدم Static Nested Class؟

```java
// 1. Builder Pattern
public class HttpRequest {
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    
    private HttpRequest(Builder builder) {
        this.url = builder.url;
        this.method = builder.method;
        this.headers = builder.headers;
    }
    
    public static class Builder {
        private String url;
        private String method = "GET";
        private Map<String, String> headers = new HashMap<>();
        
        public Builder url(String url) {
            this.url = url;
            return this;
        }
        
        public Builder method(String method) {
            this.method = method;
            return this;
        }
        
        public Builder header(String key, String value) {
            headers.put(key, value);
            return this;
        }
        
        public HttpRequest build() {
            return new HttpRequest(this);
        }
    }
}

// الاستخدام
HttpRequest request = new HttpRequest.Builder()
    .url("https://api.example.com")
    .method("POST")
    .header("Content-Type", "application/json")
    .build();
```

```java
// 2. Helper/Utility Class
public class StringUtils {
    
    public static class Validator {
        public static boolean isEmail(String str) {
            return str != null && str.contains("@");
        }
        
        public static boolean isNumeric(String str) {
            return str != null && str.matches("\\d+");
        }
    }
    
    public static class Formatter {
        public static String capitalize(String str) {
            if (str == null || str.isEmpty()) return str;
            return str.substring(0, 1).toUpperCase() + str.substring(1);
        }
    }
}

// الاستخدام
StringUtils.Validator.isEmail("test@example.com");
StringUtils.Formatter.capitalize("hello");
```

---

## 2. Member Inner Class

### التعريف

ده class معرف جوه class تاني من غير `static`. بيكون مرتبط بـ instance من الـ Outer Class.

```java
public class Car {
    private String brand;
    private String model;
    private int year;
    
    public Car(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
    }
    
    // Member Inner Class
    public class Engine {
        private int horsepower;
        private String type;
        private boolean running = false;
        
        public Engine(int horsepower, String type) {
            this.horsepower = horsepower;
            this.type = type;
        }
        
        public void start() {
            running = true;
            // ✅ يقدر يوصل لكل حاجة في الـ Outer Class
            System.out.println("بدء تشغيل محرك " + brand + " " + model);
            System.out.println("سنة الصنع: " + year);
            System.out.println("نوع المحرك: " + type);
            System.out.println("القوة: " + horsepower + " حصان");
        }
        
        public void stop() {
            running = false;
            System.out.println("إيقاف محرك " + brand);
        }
        
        // الوصول للـ outer this
        public Car getCar() {
            return Car.this;
        }
    }
    
    // Factory method
    public Engine createEngine(int hp, String type) {
        return new Engine(hp, type);
    }
}
```

### الاستخدام

```java
// الطريقة الأولى: new على outer instance
Car car = new Car("تويوتا", "كامري", 2024);
Car.Engine engine = car.new Engine(200, "V6");
engine.start();

// الطريقة الثانية: Factory method
Car car2 = new Car("هوندا", "أكورد", 2023);
Car.Engine engine2 = car2.createEngine(180, "Inline-4");
engine2.start();
```

### Shadowing

```java
public class ShadowExample {
    private int x = 10;  // Outer's x
    
    class Inner {
        private int x = 20;  // Inner's x
        
        void display(int x) {  // Parameter x
            System.out.println("Parameter x: " + x);
            System.out.println("Inner's x: " + this.x);
            System.out.println("Outer's x: " + ShadowExample.this.x);
        }
    }
}

// الاستخدام
ShadowExample outer = new ShadowExample();
ShadowExample.Inner inner = outer.new Inner();
inner.display(30);
// Output:
// Parameter x: 30
// Inner's x: 20
// Outer's x: 10
```

### خصائص الـ Member Inner Class

| الخاصية | القيمة |
|---------|--------|
| Access to outer static members | ✅ |
| Access to outer instance members | ✅ (حتى private) |
| Can have static members | ❌ (except static final constants) |
| Can have instance members | ✅ |
| Needs outer instance | ✅ |
| Access to OuterClass.this | ✅ |

### Real-World Example: Iterator Pattern

```java
public class CustomArrayList<E> implements Iterable<E> {
    private Object[] elements;
    private int size = 0;
    
    public CustomArrayList(int capacity) {
        elements = new Object[capacity];
    }
    
    public void add(E element) {
        if (size < elements.length) {
            elements[size++] = element;
        }
    }
    
    @SuppressWarnings("unchecked")
    public E get(int index) {
        return (E) elements[index];
    }
    
    public int size() {
        return size;
    }
    
    @Override
    public Iterator<E> iterator() {
        return new ArrayIterator();
    }
    
    // Member Inner Class - Iterator
    private class ArrayIterator implements Iterator<E> {
        private int currentIndex = 0;
        
        @Override
        public boolean hasNext() {
            return currentIndex < size;  // Access to outer's size
        }
        
        @Override
        @SuppressWarnings("unchecked")
        public E next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            return (E) elements[currentIndex++];  // Access to outer's elements
        }
    }
}

// الاستخدام
CustomArrayList<String> list = new CustomArrayList<>(10);
list.add("أحمد");
list.add("محمد");
list.add("علي");

for (String name : list) {
    System.out.println(name);
}
```

---

## 3. Local Inner Class

### التعريف

ده class معرف جوه **method** أو **block**. مش مرئي برا الـ scope ده.

```java
public class Calculator {
    private String name = "الآلة الحاسبة";
    
    public void performOperations(final int x, final int y) {
        final String operation = "حساب";
        
        // Local Inner Class
        class MathOperation {
            public int add() {
                // ✅ Access to outer class members
                System.out.println(name);
                // ✅ Access to effectively final local variables
                System.out.println("العملية: " + operation);
                return x + y;
            }
            
            public int multiply() {
                return x * y;
            }
            
            public int subtract() {
                return x - y;
            }
        }
        
        // استخدام الـ Local Class
        MathOperation op = new MathOperation();
        System.out.println("الجمع: " + op.add());
        System.out.println("الضرب: " + op.multiply());
        System.out.println("الطرح: " + op.subtract());
    }
}
```

### Effectively Final Variables

```java
public void example() {
    int count = 10;           // ✅ effectively final
    String name = "أحمد";     // ✅ effectively final
    int[] numbers = {1, 2};   // ✅ effectively final (reference)
    
    // count = 20;  // ❌ لو عملت ده، count مش هيكون effectively final
    
    class LocalClass {
        void process() {
            System.out.println(count);  // ✅ شغال
            System.out.println(name);   // ✅ شغال
            numbers[0] = 100;           // ✅ شغال - بنغير content مش reference
        }
    }
    
    new LocalClass().process();
}
```

### خصائص الـ Local Inner Class

| الخاصية | القيمة |
|---------|--------|
| Scope | جوه الـ method/block بس |
| Access modifiers | ❌ مش مسموح |
| Access to outer members | ✅ |
| Access to local variables | ✅ (effectively final only) |
| Can be static | ❌ |
| Reusable | ❌ |

### متى تستخدم Local Inner Class؟

```java
// عندما تحتاج class لمرة واحدة بس مع access لـ local variables
public List<String> filterData(List<String> data, String prefix, int minLength) {
    
    class Filter {
        boolean matches(String item) {
            return item.startsWith(prefix) && item.length() >= minLength;
        }
    }
    
    Filter filter = new Filter();
    List<String> result = new ArrayList<>();
    
    for (String item : data) {
        if (filter.matches(item)) {
            result.add(item);
        }
    }
    
    return result;
}
```

---

## 4. Anonymous Inner Class

### التعريف

ده class من غير اسم! بيتعرف ويتعمله instantiation في نفس الوقت. بيستخدم لـ:
- Implement interface
- Extend class

```java
// الـ Syntax الأساسي
new SuperTypeOrInterface() {
    // class body
};
```

### أمثلة

#### مع Interface

```java
interface Greeting {
    void sayHello();
    void sayGoodbye();
}

public class Main {
    public static void main(String[] args) {
        // Anonymous Inner Class implementing interface
        Greeting arabicGreeting = new Greeting() {
            @Override
            public void sayHello() {
                System.out.println("أهلاً وسهلاً!");
            }
            
            @Override
            public void sayGoodbye() {
                System.out.println("مع السلامة!");
            }
        };
        
        arabicGreeting.sayHello();
        arabicGreeting.sayGoodbye();
    }
}
```

#### مع Abstract Class

```java
abstract class Animal {
    protected String name;
    
    public Animal(String name) {
        this.name = name;
    }
    
    abstract void makeSound();
    
    public void sleep() {
        System.out.println(name + " نايم...");
    }
}

public class Main {
    public static void main(String[] args) {
        // Anonymous class extending abstract class
        Animal cat = new Animal("قطة") {
            @Override
            void makeSound() {
                System.out.println(name + " بتقول: مياو!");
            }
        };
        
        cat.makeSound();
        cat.sleep();
    }
}
```

#### مع Concrete Class

```java
class Button {
    private String label;
    
    public Button(String label) {
        this.label = label;
    }
    
    public void click() {
        System.out.println("Button clicked: " + label);
    }
}

// Override method في Anonymous class
Button specialButton = new Button("Special") {
    @Override
    public void click() {
        System.out.println("🎉 Special button activated!");
        super.click();
    }
};
```

### الاستخدامات الشائعة

#### Event Listeners (Swing/JavaFX)

```java
// Swing
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("تم الضغط على الزر!");
    }
});

// JavaFX
button.setOnAction(new EventHandler<ActionEvent>() {
    @Override
    public void handle(ActionEvent event) {
        System.out.println("Button clicked!");
    }
});
```

#### Comparator

```java
List<Person> people = Arrays.asList(
    new Person("أحمد", 25),
    new Person("محمد", 30),
    new Person("علي", 20)
);

// ترتيب بالعمر
Collections.sort(people, new Comparator<Person>() {
    @Override
    public int compare(Person p1, Person p2) {
        return Integer.compare(p1.getAge(), p2.getAge());
    }
});
```

#### Thread/Runnable

```java
// Runnable
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Thread شغال!");
    }
});
thread.start();

// أو مباشرة
new Thread(new Runnable() {
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println("Count: " + i);
        }
    }
}).start();
```

### Anonymous Class vs Lambda

```java
// Functional Interface
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

// Anonymous Inner Class
Calculator add1 = new Calculator() {
    @Override
    public int calculate(int a, int b) {
        return a + b;
    }
};

// Lambda Expression (أبسط وأنظف)
Calculator add2 = (a, b) -> a + b;

// Method Reference
Calculator add3 = Integer::sum;
```

#### متى تستخدم Anonymous Class بدل Lambda؟

```java
// 1. لما تحتاج أكتر من method
interface MultiMethod {
    void method1();
    void method2();
}

MultiMethod obj = new MultiMethod() {
    @Override
    public void method1() { }
    
    @Override
    public void method2() { }
};

// 2. لما تحتاج state (fields)
Runnable r = new Runnable() {
    private int count = 0;  // ✅ ممكن في Anonymous
    
    @Override
    public void run() {
        count++;
    }
};

// 3. لما تحتاج this reference للـ Anonymous class نفسه
Runnable r2 = new Runnable() {
    @Override
    public void run() {
        System.out.println(this.getClass().getName());  // Anonymous class
    }
};
```

### خصائص الـ Anonymous Inner Class

| الخاصية | القيمة |
|---------|--------|
| Has a name | ❌ |
| Reusable | ❌ |
| Can implement interface | ✅ (one only) |
| Can extend class | ✅ (one only) |
| Can have constructor | ❌ (uses instance initializer) |
| Can have static members | ❌ (except constants) |
| Access to outer members | ✅ |
| Access to local variables | ✅ (effectively final) |

---

## جدول مقارنة شامل

| الخاصية | Static Nested | Member Inner | Local Inner | Anonymous |
|---------|---------------|--------------|-------------|-----------|
| **موقع التعريف** | Class level | Class level | Method/Block | Expression |
| **Static members** | ✅ | ❌ | ❌ | ❌ |
| **Access to outer instance** | ❌ | ✅ | ✅ | ✅ |
| **Access to outer static** | ✅ | ✅ | ✅ | ✅ |
| **Access to local vars** | N/A | N/A | ✅ (final) | ✅ (final) |
| **Has name** | ✅ | ✅ | ✅ | ❌ |
| **Needs outer instance** | ❌ | ✅ | ✅ | ✅ |
| **Reusable** | ✅ | ✅ | ❌ | ❌ |
| **Access modifiers** | ✅ | ✅ | ❌ | ❌ |

---

# الجزء الثاني: Polymorphism

## مقدمة

### يعني إيه Polymorphism؟

كلمة **Polymorphism** من اليوناني ومعناها "أشكال كتير" (Poly = كتير، Morph = شكل).

في البرمجة: **نفس الـ interface/method ممكن يتصرف بطرق مختلفة حسب الـ object**.

### مثال من الحياة الواقعية

```
الزر "Play" ▶️ في الموبايل:
├── على أغنية → يشغل موسيقى 🎵
├── على فيديو → يشغل فيديو 🎬
├── على بودكاست → يشغل صوت 🎙️
└── على لعبة → يبدأ اللعب 🎮

نفس الفعل (Play) بيعمل حاجات مختلفة حسب السياق!
```

---

## أنواع الـ Polymorphism

```
Polymorphism
├── Compile-Time (Static)
│   └── Method Overloading
│
└── Runtime (Dynamic)
    ├── Method Overriding
    └── Interface Implementation
```

---

## 1. Compile-Time Polymorphism (Method Overloading)

### التعريف

نفس اسم الـ method بس بـ **parameters مختلفة**. الـ compiler بيحدد أنهي method يستدعي.

```java
public class Printer {
    
    // Overloaded print methods
    public void print(String text) {
        System.out.println("String: " + text);
    }
    
    public void print(int number) {
        System.out.println("Integer: " + number);
    }
    
    public void print(double number) {
        System.out.println("Double: " + number);
    }
    
    public void print(String text, int times) {
        for (int i = 0; i < times; i++) {
            System.out.println(text);
        }
    }
    
    public void print(int... numbers) {
        System.out.print("Numbers: ");
        for (int n : numbers) {
            System.out.print(n + " ");
        }
        System.out.println();
    }
}

// الاستخدام
Printer printer = new Printer();
printer.print("مرحباً");           // String: مرحباً
printer.print(42);                 // Integer: 42
printer.print(3.14);               // Double: 3.14
printer.print("Hi", 3);            // Hi (3 مرات)
printer.print(1, 2, 3, 4, 5);      // Numbers: 1 2 3 4 5
```

### قواعد الـ Overloading

```java
public class OverloadingRules {
    
    // ✅ عدد parameters مختلف
    void method() { }
    void method(int a) { }
    void method(int a, int b) { }
    
    // ✅ نوع parameters مختلف
    void calculate(int a) { }
    void calculate(double a) { }
    void calculate(String a) { }
    
    // ✅ ترتيب parameters مختلف
    void process(int a, String b) { }
    void process(String a, int b) { }
    
    // ❌ Return type بس مختلف - Compile Error!
    // int getValue() { return 1; }
    // double getValue() { return 1.0; }  // ❌
    
    // ❌ Access modifier بس مختلف - Compile Error!
    // public void doSomething(int a) { }
    // private void doSomething(int a) { }  // ❌
    
    // ❌ Exception بس مختلفة - Compile Error!
    // void save(String data) { }
    // void save(String data) throws IOException { }  // ❌
}
```

### Type Promotion في Overloading

```java
public class TypePromotionExample {
    
    void show(int x) {
        System.out.println("int: " + x);
    }
    
    void show(long x) {
        System.out.println("long: " + x);
    }
    
    void show(float x) {
        System.out.println("float: " + x);
    }
    
    void show(double x) {
        System.out.println("double: " + x);
    }
}

TypePromotionExample t = new TypePromotionExample();

byte b = 10;
t.show(b);    // int: 10 (byte → int)

short s = 20;
t.show(s);    // int: 20 (short → int)

char c = 'A';
t.show(c);    // int: 65 (char → int)

t.show(100);  // int: 100
t.show(100L); // long: 100
t.show(1.5f); // float: 1.5
t.show(1.5);  // double: 1.5
```

### هرم الـ Type Promotion

```
           double
              ↑
           float
              ↑
           long
              ↑
            int  ←── char
              ↑
           short
              ↑
           byte
```

### Ambiguity في Overloading

```java
public class AmbiguityExample {
    
    void method(int a, long b) {
        System.out.println("int, long");
    }
    
    void method(long a, int b) {
        System.out.println("long, int");
    }
}

AmbiguityExample obj = new AmbiguityExample();
obj.method(10, 20L);  // ✅ int, long
obj.method(10L, 20);  // ✅ long, int
// obj.method(10, 20);  // ❌ Compile Error - Ambiguous!
```

---

## 2. Runtime Polymorphism (Method Overriding)

### التعريف

الـ subclass بيعمل **implementation جديد** لـ method موجود في الـ parent. الـ JVM بيحدد أنهي method يستدعي **وقت الـ runtime**.

```java
// Parent Class
class Shape {
    protected String color;
    
    public Shape(String color) {
        this.color = color;
    }
    
    public double getArea() {
        return 0;
    }
    
    public void display() {
        System.out.println("شكل بلون: " + color);
    }
}

// Child Classes
class Circle extends Shape {
    private double radius;
    
    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }
    
    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public void display() {
        System.out.println("دائرة " + color + " بنصف قطر " + radius);
    }
}

class Rectangle extends Shape {
    private double width, height;
    
    public Rectangle(String color, double width, double height) {
        super(color);
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double getArea() {
        return width * height;
    }
    
    @Override
    public void display() {
        System.out.println("مستطيل " + color + " " + width + "x" + height);
    }
}

// الاستخدام - Polymorphism!
public class Main {
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle("أحمر", 5),
            new Rectangle("أزرق", 4, 6),
            new Circle("أخضر", 3)
        };
        
        for (Shape shape : shapes) {
            shape.display();
            System.out.println("المساحة: " + shape.getArea());
            System.out.println("---");
        }
    }
}
```

### قواعد الـ Overriding

#### 1. Method Signature

```java
class Parent {
    void method(int x) { }
}

class Child extends Parent {
    @Override
    void method(int x) { }  // ✅ نفس الـ signature بالظبط
    
    // void method(long x) { }  // ❌ ده Overloading مش Overriding
}
```

#### 2. Return Type (Covariant)

```java
class Parent {
    Number getValue() {
        return 0;
    }
    
    Parent getInstance() {
        return new Parent();
    }
}

class Child extends Parent {
    @Override
    Integer getValue() {  // ✅ Integer is subtype of Number
        return 0;
    }
    
    @Override
    Child getInstance() {  // ✅ Child is subtype of Parent
        return new Child();
    }
}
```

#### 3. Access Modifier

```java
class Parent {
    protected void method() { }
}

class Child extends Parent {
    @Override
    public void method() { }     // ✅ public > protected
    
    @Override
    protected void method() { }  // ✅ same access
    
    // @Override
    // private void method() { }  // ❌ private < protected
}
```

**ترتيب الـ Access Modifiers:**
```
private < default < protected < public
(أضيق)                         (أوسع)
```

#### 4. Exceptions

```java
class Parent {
    void method() throws IOException { }
}

class Child extends Parent {
    @Override
    void method() throws FileNotFoundException { }  // ✅ أضيق (subclass)
    
    @Override
    void method() throws IOException { }  // ✅ نفسها
    
    @Override
    void method() { }  // ✅ مفيش exception خالص
    
    // @Override
    // void method() throws Exception { }  // ❌ أوسع
}
```

### جدول قواعد الـ Overriding

| القاعدة | مسموح | ممنوع |
|---------|-------|-------|
| **Signature** | نفسها بالظبط | مختلفة |
| **Return type** | نفسه أو subtype | supertype |
| **Access** | نفسه أو أوسع | أضيق |
| **Exceptions** | نفسها أو أقل/أضيق | أكتر/أوسع |
| **final methods** | - | ❌ ممنوع |
| **static methods** | - | ❌ hiding مش overriding |
| **private methods** | - | ❌ مش موروثة |

### @Override Annotation

```java
class Parent {
    void doSomething() { }
}

class Child extends Parent {
    @Override  // ✅ يساعد الـ compiler يتحقق
    void doSomething() { }
    
    @Override  // ❌ Compile Error - مفيش method بالاسم ده في الـ Parent
    void doSomethng() { }  // خطأ إملائي!
}
```

---

## Upcasting و Downcasting

### Upcasting (Implicit)

تحويل من Child type لـ Parent type - **تلقائي وآمن**.

```java
class Animal {
    void eat() { System.out.println("Animal eating"); }
}

class Dog extends Animal {
    @Override
    void eat() { System.out.println("Dog eating"); }
    void bark() { System.out.println("Woof!"); }
}

// Upcasting
Animal animal = new Dog();  // ✅ تلقائي
animal.eat();    // Dog eating (polymorphism!)
// animal.bark();  // ❌ Compile Error - Animal مش عنده bark()
```

### Downcasting (Explicit)

تحويل من Parent type لـ Child type - **يدوي ومحتاج حذر**.

```java
Animal animal = new Dog();

// Downcasting - يدوي
Dog dog = (Dog) animal;  // ✅ شغال لأن الـ actual object هو Dog
dog.bark();              // ✅ Woof!

// ⚠️ خطر!
Animal animal2 = new Animal();
// Dog dog2 = (Dog) animal2;  // ❌ ClassCastException at runtime!
```

### Safe Downcasting مع instanceof

```java
public void processAnimal(Animal animal) {
    animal.eat();  // Polymorphic call
    
    // Safe downcasting
    if (animal instanceof Dog) {
        Dog dog = (Dog) animal;
        dog.bark();
    } else if (animal instanceof Cat) {
        Cat cat = (Cat) animal;
        cat.meow();
    }
}

// Java 16+ Pattern Matching for instanceof
public void processAnimalModern(Animal animal) {
    if (animal instanceof Dog dog) {
        dog.bark();  // dog متاح مباشرة
    } else if (animal instanceof Cat cat) {
        cat.meow();
    }
}

// Java 21+ Pattern Matching with switch
public void processAnimalSwitch(Animal animal) {
    switch (animal) {
        case Dog dog -> dog.bark();
        case Cat cat -> cat.meow();
        case null -> System.out.println("No animal");
        default -> System.out.println("Unknown animal");
    }
}
```

---

## Polymorphism مع Interfaces

```java
// Interfaces
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

// Implementations
class Duck implements Flyable, Swimmable {
    @Override
    public void fly() {
        System.out.println("البطة بتطير 🦆");
    }
    
    @Override
    public void swim() {
        System.out.println("البطة بتعوم 🦆");
    }
}

class Airplane implements Flyable {
    @Override
    public void fly() {
        System.out.println("الطيارة بتطير ✈️");
    }
}

class Fish implements Swimmable {
    @Override
    public void swim() {
        System.out.println("السمكة بتعوم 🐟");
    }
}

// Polymorphic usage
public class Main {
    public static void main(String[] args) {
        // Polymorphism through interface
        List<Flyable> flyingThings = Arrays.asList(
            new Duck(),
            new Airplane()
        );
        
        for (Flyable f : flyingThings) {
            f.fly();  // كل واحد هيطير بطريقته!
        }
        
        List<Swimmable> swimmingThings = Arrays.asList(
            new Duck(),
            new Fish()
        );
        
        for (Swimmable s : swimmingThings) {
            s.swim();
        }
    }
}
```

---

## Method Hiding (Static Methods)

الـ **static methods مش بيتعملها override** - بيتعملها **hiding**!

```java
class Parent {
    public static void staticMethod() {
        System.out.println("Parent static");
    }
    
    public void instanceMethod() {
        System.out.println("Parent instance");
    }
}

class Child extends Parent {
    // Method Hiding (مش Overriding!)
    public static void staticMethod() {
        System.out.println("Child static");
    }
    
    @Override
    public void instanceMethod() {
        System.out.println("Child instance");
    }
}

// الفرق المهم
Parent p = new Child();

p.staticMethod();    // Parent static ← Reference type!
p.instanceMethod();  // Child instance ← Actual object type!

Child c = new Child();
c.staticMethod();    // Child static
c.instanceMethod();  // Child instance
```

### الفرق بين Overriding و Hiding

| Overriding | Hiding |
|------------|--------|
| Instance methods | Static methods |
| Runtime binding | Compile-time binding |
| Based on object type | Based on reference type |
| Uses `@Override` | No annotation |
| Polymorphic | Not polymorphic |

---

## Virtual Method Invocation

```java
class Employee {
    public void work() {
        System.out.println("Employee working...");
    }
}

class Developer extends Employee {
    @Override
    public void work() {
        System.out.println("Developer coding...");
    }
    
    public void debug() {
        System.out.println("Developer debugging...");
    }
}

class Manager extends Employee {
    @Override
    public void work() {
        System.out.println("Manager managing...");
    }
    
    public void meetClient() {
        System.out.println("Manager meeting client...");
    }
}

// Virtual Method Table (VMT) في الـ runtime
public class Company {
    public static void main(String[] args) {
        Employee[] team = {
            new Developer(),
            new Manager(),
            new Developer()
        };
        
        for (Employee emp : team) {
            emp.work();  // JVM يحدد الـ actual method في runtime
        }
    }
}
// Output:
// Developer coding...
// Manager managing...
// Developer coding...
```

---

## Real-World Example: Payment System

```java
// Abstract base class
abstract class PaymentMethod {
    protected String accountId;
    protected BigDecimal balance;
    
    public PaymentMethod(String accountId, BigDecimal balance) {
        this.accountId = accountId;
        this.balance = balance;
    }
    
    // Template method
    public final boolean pay(BigDecimal amount) {
        if (!validate()) {
            System.out.println("❌ فشل التحقق");
            return false;
        }
        if (!hasSufficientFunds(amount)) {
            System.out.println("❌ رصيد غير كافي");
            return false;
        }
        return processPayment(amount);
    }
    
    // Methods to override
    protected abstract boolean validate();
    protected abstract boolean processPayment(BigDecimal amount);
    
    protected boolean hasSufficientFunds(BigDecimal amount) {
        return balance.compareTo(amount) >= 0;
    }
}

// Concrete implementations
class CreditCard extends PaymentMethod {
    private String cardNumber;
    private String cvv;
    
    public CreditCard(String accountId, BigDecimal limit, 
                      String cardNumber, String cvv) {
        super(accountId, limit);
        this.cardNumber = cardNumber;
        this.cvv = cvv;
    }
    
    @Override
    protected boolean validate() {
        return cardNumber.length() == 16 && cvv.length() == 3;
    }
    
    @Override
    protected boolean processPayment(BigDecimal amount) {
        System.out.println("💳 دفع " + amount + " بالبطاقة: ****" + 
                          cardNumber.substring(12));
        balance = balance.subtract(amount);
        return true;
    }
}

class BankTransfer extends PaymentMethod {
    private String bankCode;
    private String iban;
    
    public BankTransfer(String accountId, BigDecimal balance,
                        String bankCode, String iban) {
        super(accountId, balance);
        this.bankCode = bankCode;
        this.iban = iban;
    }
    
    @Override
    protected boolean validate() {
        return iban.length() == 29;  // Egypt IBAN length
    }
    
    @Override
    protected boolean processPayment(BigDecimal amount) {
        System.out.println("🏦 تحويل " + amount + " من بنك: " + bankCode);
        balance = balance.subtract(amount);
        return true;
    }
}

class DigitalWallet extends PaymentMethod {
    private String phoneNumber;
    
    public DigitalWallet(String accountId, BigDecimal balance,
                         String phoneNumber) {
        super(accountId, balance);
        this.phoneNumber = phoneNumber;
    }
    
    @Override
    protected boolean validate() {
        return phoneNumber.matches("^01[0125]\\d{8}$");  // Egyptian mobile
    }
    
    @Override
    protected boolean processPayment(BigDecimal amount) {
        System.out.println("📱 دفع " + amount + " من محفظة: " + phoneNumber);
        balance = balance.subtract(amount);
        return true;
    }
}

// Usage with Polymorphism
public class PaymentProcessor {
    public static void processOrder(PaymentMethod payment, BigDecimal amount) {
        if (payment.pay(amount)) {
            System.out.println("✅ تمت العملية بنجاح!");
        } else {
            System.out.println("❌ فشلت العملية");
        }
    }
    
    public static void main(String[] args) {
        PaymentMethod[] methods = {
            new CreditCard("ACC001", new BigDecimal("5000"), 
                          "1234567890123456", "123"),
            new BankTransfer("ACC002", new BigDecimal("10000"),
                            "NBE", "EG380019000500000000000000000"),
            new DigitalWallet("ACC003", new BigDecimal("2000"),
                             "01012345678")
        };
        
        BigDecimal orderAmount = new BigDecimal("500");
        
        for (PaymentMethod method : methods) {
            System.out.println("\n--- Processing ---");
            processOrder(method, orderAmount);
        }
    }
}
```

---

# 🎯 أسئلة الانترفيو

## الجزء الأول: Nested Classes

### س1: إيه أنواع الـ Nested Classes في Java؟
**الإجابة:**
```
Nested Classes
├── Static Nested Class (static modifier)
└── Inner Classes (non-static)
    ├── Member Inner Class (at class level)
    ├── Local Inner Class (inside method/block)
    └── Anonymous Inner Class (no name, one-time use)
```

---

### س2: إيه الفرق بين Static Nested Class و Member Inner Class؟

**الإجابة:**

| Static Nested | Member Inner |
|---------------|--------------|
| معاه `static` | من غير `static` |
| مش محتاج outer instance | محتاج outer instance |
| يوصل للـ static members بس | يوصل لكل الـ members |
| `new Outer.Nested()` | `outer.new Inner()` |
| ممكن يكون فيه static members | مش ممكن (إلا constants) |

```java
class Outer {
    private static int staticVar = 1;
    private int instanceVar = 2;
    
    static class StaticNested {
        void method() {
            System.out.println(staticVar);    // ✅
            // System.out.println(instanceVar);  // ❌
        }
    }
    
    class MemberInner {
        void method() {
            System.out.println(staticVar);    // ✅
            System.out.println(instanceVar);  // ✅
        }
    }
}

// Usage
Outer.StaticNested sn = new Outer.StaticNested();  // No outer instance needed

Outer outer = new Outer();
Outer.MemberInner mi = outer.new MemberInner();    // Outer instance required
```

---

### س3: يعني إيه Effectively Final؟

**الإجابة:**
Effectively Final هو variable مش معرف كـ `final` بس مش بيتغير بعد ما يتعرف. الـ Local Inner Classes و Anonymous Classes لازم توصل لـ effectively final variables بس.

```java
void method() {
    int x = 10;        // effectively final
    int y = 20;
    y = 30;            // ده بيخلي y مش effectively final
    
    class Local {
        void print() {
            System.out.println(x);  // ✅ x effectively final
            // System.out.println(y);  // ❌ y مش effectively final
        }
    }
}
```

**السبب:** الـ inner class ممكن يعيش أكتر من الـ method، فالـ Java بتعمل copy للـ variables، ولو اتغيروا هيحصل inconsistency.

---

### س4: إزاي توصل للـ Outer class this من Inner class؟

**الإجابة:**
باستخدام `OuterClassName.this`:

```java
class Outer {
    private int value = 10;
    
    class Inner {
        private int value = 20;
        
        void show(int value) {
            System.out.println(value);              // parameter: 30
            System.out.println(this.value);         // Inner's: 20
            System.out.println(Outer.this.value);   // Outer's: 10
        }
    }
}

Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
inner.show(30);
```

---

### س5: إمتى تستخدم كل نوع من الـ Nested Classes؟

**الإجابة:**

| النوع | متى تستخدمه |
|-------|-------------|
| **Static Nested** | Helper classes، Builder pattern، مش محتاج outer instance |
| **Member Inner** | Iterator، محتاج access لـ outer instance |
| **Local Inner** | class لمرة واحدة جوه method |
| **Anonymous** | Callbacks، Event listeners، implementation لمرة واحدة |

---

### س6: إيه الفرق بين Anonymous Inner Class و Lambda؟

**الإجابة:**

```java
// Anonymous Inner Class
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

// Lambda
Runnable r2 = () -> System.out.println("Hello");
```

| Anonymous Class | Lambda |
|-----------------|--------|
| أي interface | Functional interface بس |
| ممكن يكون فيه state | Stateless |
| `this` = anonymous class | `this` = enclosing class |
| ممكن يعمل implement لأكتر من method | method واحدة بس |
| أطول | أقصر وأنظف |

---

### س7: هل ممكن Inner Class يكون private؟

**الإجابة:**
أيوه، الـ **Member Inner Class** و **Static Nested Class** ممكن يكونوا private/protected/public/default.

```java
class Outer {
    private class PrivateInner { }      // ✅ مرئي جوه Outer بس
    protected class ProtectedInner { }  // ✅
    public class PublicInner { }        // ✅
    class DefaultInner { }              // ✅
    
    private static class PrivateStatic { }  // ✅
}
```

لكن **Local Inner Class** و **Anonymous Class** مش ممكن يكون فيهم access modifiers.

---

### س8: اكتب كود لـ Iterator باستخدام Member Inner Class

```java
public class CustomStack<E> implements Iterable<E> {
    private Object[] elements;
    private int size = 0;
    
    public CustomStack(int capacity) {
        elements = new Object[capacity];
    }
    
    public void push(E item) {
        if (size < elements.length) {
            elements[size++] = item;
        }
    }
    
    @SuppressWarnings("unchecked")
    public E pop() {
        if (size > 0) {
            return (E) elements[--size];
        }
        throw new EmptyStackException();
    }
    
    @Override
    public Iterator<E> iterator() {
        return new StackIterator();
    }
    
    // Member Inner Class
    private class StackIterator implements Iterator<E> {
        private int current = size - 1;  // Start from top
        
        @Override
        public boolean hasNext() {
            return current >= 0;
        }
        
        @Override
        @SuppressWarnings("unchecked")
        public E next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            return (E) elements[current--];
        }
    }
}
```

---

## الجزء الثاني: Polymorphism

### س9: إيه الفرق بين Overloading و Overriding؟

**الإجابة:**

| Overloading | Overriding |
|-------------|------------|
| نفس الـ class أو parent/child | Parent و Child بس |
| نفس الاسم، parameters مختلفة | نفس الـ signature بالظبط |
| Compile-time | Runtime |
| Return type ممكن يختلف | Return type نفسه أو covariant |
| Access modifier أي حاجة | نفسه أو أوسع |
| مش polymorphism حقيقي | Polymorphism الحقيقي |

```java
// Overloading
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }  // نفس الاسم، parameters مختلفة
}

// Overriding
class Animal {
    void speak() { System.out.println("..."); }
}

class Dog extends Animal {
    @Override
    void speak() { System.out.println("Woof!"); }  // نفس الـ signature
}
```

---

### س10: يعني إيه Covariant Return Type؟

**الإجابة:**
الـ overriding method ممكن يرجع **subtype** من الـ return type بتاع الـ parent method.

```java
class Animal {
    Animal reproduce() {
        return new Animal();
    }
}

class Dog extends Animal {
    @Override
    Dog reproduce() {  // ✅ Dog is subtype of Animal
        return new Dog();
    }
}

// الفايدة: مش محتاج casting
Dog dog = new Dog();
Dog puppy = dog.reproduce();  // ✅ مباشرة من غير casting
```

---

### س11: هل ممكن نعمل Override لـ static method؟

**الإجابة:**
**لا!** الـ static methods بيتعملها **hiding** مش overriding.

```java
class Parent {
    static void staticMethod() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    static void staticMethod() {  // Hiding, not overriding
        System.out.println("Child");
    }
}

Parent p = new Child();
p.staticMethod();  // "Parent" ← based on reference type, not object type
```

الـ Hiding بيتحدد بالـ **reference type** وقت الـ compile.
الـ Overriding بيتحدد بالـ **actual object type** وقت الـ runtime.

---

### س12: إيه هو Dynamic Method Dispatch؟

**الإجابة:**
Dynamic Method Dispatch (أو Virtual Method Invocation) هو الآلية اللي الـ JVM بيستخدمها عشان يحدد أنهي **overridden method** يستدعي في الـ **runtime** بناءً على الـ actual object type.

```java
class Shape {
    void draw() { System.out.println("Drawing shape"); }
}

class Circle extends Shape {
    @Override
    void draw() { System.out.println("Drawing circle"); }
}

// Dynamic dispatch
Shape s = new Circle();  // Reference: Shape, Object: Circle
s.draw();  // Runtime: JVM يشوف الـ actual object type (Circle)
           // Output: "Drawing circle"
```

---

### س13: إيه الفرق بين Upcasting و Downcasting؟

**الإجابة:**

```java
class Animal { }
class Dog extends Animal {
    void bark() { }
}

// Upcasting: Child → Parent (implicit, safe)
Animal animal = new Dog();  // ✅ تلقائي

// Downcasting: Parent → Child (explicit, risky)
Dog dog = (Dog) animal;     // ✅ شغال لأن actual object هو Dog
dog.bark();

// ⚠️ خطر
Animal animal2 = new Animal();
// Dog dog2 = (Dog) animal2;  // ❌ ClassCastException!

// Safe downcasting
if (animal2 instanceof Dog d) {
    d.bark();  // ✅ Safe
}
```

| Upcasting | Downcasting |
|-----------|-------------|
| Implicit | Explicit (casting) |
| دايماً آمن | ممكن يفشل |
| Child → Parent | Parent → Child |
| بنخسر child methods | بنكسب child methods |

---

### س14: ليه الـ private methods مش بيتعملها Override؟

**الإجابة:**
لأن الـ **private methods مش موروثة**! الـ child class مش بيشوفها أصلاً.

```java
class Parent {
    private void privateMethod() {
        System.out.println("Parent private");
    }
    
    public void callPrivate() {
        privateMethod();
    }
}

class Child extends Parent {
    // ده method جديد، مش override!
    private void privateMethod() {
        System.out.println("Child private");
    }
}

Child c = new Child();
c.callPrivate();  // "Parent private" ← الـ Parent's method
```

---

### س15: ليه الـ final methods مش بيتعملها Override؟

**الإجابة:**
لأن `final` معناها "النهائي" - مش ممكن يتغير.

```java
class Parent {
    final void finalMethod() {
        System.out.println("Can't override this!");
    }
}

class Child extends Parent {
    // @Override
    // void finalMethod() { }  // ❌ Compile Error!
}
```

**أسباب استخدام final methods:**
1. **Security**: منع تغيير behavior حساس
2. **Performance**: الـ JVM ممكن يعمل inlining
3. **Design**: التأكيد إن الـ implementation مش هيتغير

---

### س16: إيه الفرق بين Abstract Class و Interface في الـ Polymorphism؟

**الإجابة:**

```java
// Abstract Class
abstract class Vehicle {
    protected String brand;
    
    public Vehicle(String brand) {
        this.brand = brand;
    }
    
    abstract void start();  // Abstract method
    
    void stop() {           // Concrete method
        System.out.println(brand + " stopped");
    }
}

// Interface
interface Drivable {
    void drive();           // implicitly public abstract
    
    default void park() {   // default method (Java 8+)
        System.out.println("Parking...");
    }
}

// Usage
class Car extends Vehicle implements Drivable {
    public Car(String brand) {
        super(brand);
    }
    
    @Override
    void start() {
        System.out.println(brand + " car starting");
    }
    
    @Override
    public void drive() {
        System.out.println("Driving " + brand);
    }
}
```

| Abstract Class | Interface |
|----------------|-----------|
| Single inheritance | Multiple implementation |
| Can have constructors | No constructors |
| Can have instance fields | Only constants |
| Can have any access | Only public (mostly) |
| "is-a" relationship | "can-do" capability |

---

### س17: اكتب مثال للـ Polymorphism مع Strategy Pattern

```java
// Strategy Interface
interface PricingStrategy {
    double calculatePrice(double basePrice, int quantity);
}

// Concrete Strategies
class RegularPricing implements PricingStrategy {
    @Override
    public double calculatePrice(double basePrice, int quantity) {
        return basePrice * quantity;
    }
}

class BulkPricing implements PricingStrategy {
    @Override
    public double calculatePrice(double basePrice, int quantity) {
        if (quantity >= 10) {
            return basePrice * quantity * 0.9;  // 10% discount
        }
        return basePrice * quantity;
    }
}

class VIPPricing implements PricingStrategy {
    @Override
    public double calculatePrice(double basePrice, int quantity) {
        return basePrice * quantity * 0.8;  // 20% discount always
    }
}

// Context
class ShoppingCart {
    private PricingStrategy strategy;
    
    public void setStrategy(PricingStrategy strategy) {
        this.strategy = strategy;
    }
    
    public double checkout(double basePrice, int quantity) {
        return strategy.calculatePrice(basePrice, quantity);
    }
}

// Usage - Polymorphism in action!
ShoppingCart cart = new ShoppingCart();

cart.setStrategy(new RegularPricing());
System.out.println(cart.checkout(100, 5));   // 500.0

cart.setStrategy(new BulkPricing());
System.out.println(cart.checkout(100, 15));  // 1350.0 (10% off)

cart.setStrategy(new VIPPricing());
System.out.println(cart.checkout(100, 5));   // 400.0 (20% off)
```

---

### س18: إيه هو الـ Liskov Substitution Principle (LSP)؟

**الإجابة:**
الـ LSP بيقول: **Objects of a superclass should be replaceable with objects of its subclasses without breaking the application.**

يعني أي method بتاخد Parent، لازم تشتغل صح لو بعتتلها أي Child.

```java
// ❌ Violating LSP
class Rectangle {
    protected int width, height;
    
    public void setWidth(int w) { width = w; }
    public void setHeight(int h) { height = h; }
    public int getArea() { return width * height; }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int w) {
        width = w;
        height = w;  // Force square behavior
    }
    
    @Override
    public void setHeight(int h) {
        width = h;
        height = h;
    }
}

// المشكلة
void resize(Rectangle r) {
    r.setWidth(5);
    r.setHeight(10);
    assert r.getArea() == 50;  // ❌ Fails for Square!
}
```

```java
// ✅ Following LSP
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    private int width, height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public int getArea() { return width * height; }
}

class Square implements Shape {
    private int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    @Override
    public int getArea() { return side * side; }
}
```

---

### س19: اشرح الـ instanceof مع Pattern Matching (Java 16+)

```java
// Before Java 16
void process(Object obj) {
    if (obj instanceof String) {
        String s = (String) obj;
        System.out.println(s.length());
    }
}

// Java 16+ Pattern Matching
void processModern(Object obj) {
    if (obj instanceof String s) {
        System.out.println(s.length());  // s متاح مباشرة
    }
    
    // مع condition
    if (obj instanceof String s && s.length() > 5) {
        System.out.println("Long string: " + s);
    }
}

// Java 21+ Switch Pattern Matching
String describe(Object obj) {
    return switch (obj) {
        case Integer i -> "Integer: " + i;
        case Long l -> "Long: " + l;
        case Double d -> "Double: " + d;
        case String s when s.isEmpty() -> "Empty string";
        case String s -> "String: " + s;
        case null -> "null value";
        default -> "Unknown: " + obj.getClass();
    };
}
```

---

### س20: إيه الفرق بين Method Overriding و Method Hiding؟

**الإجابة:**

```java
class Parent {
    static void staticMethod() {
        System.out.println("Parent static");
    }
    
    void instanceMethod() {
        System.out.println("Parent instance");
    }
}

class Child extends Parent {
    static void staticMethod() {      // HIDING
        System.out.println("Child static");
    }
    
    @Override
    void instanceMethod() {           // OVERRIDING
        System.out.println("Child instance");
    }
}

Parent ref = new Child();
ref.staticMethod();   // "Parent static" ← Hiding (reference type)
ref.instanceMethod(); // "Child instance" ← Overriding (object type)
```

| Aspect | Overriding | Hiding |
|--------|------------|--------|
| Methods | Instance | Static |
| Binding | Runtime | Compile-time |
| Based on | Object type | Reference type |
| Polymorphic | ✅ Yes | ❌ No |
| `@Override` | ✅ Use it | ❌ Don't use |

---

## نصائح للانترفيو 💡

### Nested Classes:
1. اعرف الفرق بين الأربع أنواع كويس
2. افهم effectively final وليه مهم
3. اعرف إمتى تستخدم كل نوع
4. كن جاهز تكتب Iterator pattern

### Polymorphism:
1. اعرف الفرق بين Overloading و Overriding بالتفصيل
2. افهم قواعد الـ Overriding (return type, access, exceptions)
3. اشرح Runtime vs Compile-time polymorphism
4. كن جاهز تكتب examples للـ Strategy أو Template pattern
5. اعرف LSP كويس

---

## موارد إضافية 📚

- [Oracle Java Tutorials - Nested Classes](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html)
- [Oracle Java Tutorials - Polymorphism](https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html)
- [Effective Java by Joshua Bloch](https://www.oreilly.com/library/view/effective-java/9780134686097/)

---

*تم إعداد هذا الدليل بالعامية المصرية عشان يكون سهل الفهم والمراجعة* 🇪🇬
