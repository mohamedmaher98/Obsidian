# Java Generics - الدليل الشامل

## يعني إيه Generics؟

تخيل إنك عندك صندوق، الصندوق ده ممكن تحط فيه أي حاجة - تفاح، كتب، أقلام. بس المشكلة إنك لما تيجي تطلع حاجة من الصندوق مش هتعرف هي إيه غير لما تشوفها.

الـ Generics بتحل المشكلة دي - بتخليك تقول "الصندوق ده للتفاح بس" فلما تطلع حاجة منه تبقى متأكد إنها تفاحة.

```java
// من غير Generics - صندوق لأي حاجة
List list = new ArrayList();
list.add("نص");
list.add(123);          // مفيش مشكلة وقت الـ compile
String s = (String) list.get(1); // ClassCastException وقت الـ runtime!

// مع Generics - صندوق محدد النوع
List<String> list = new ArrayList<>();
list.add("نص");
list.add(123);          // Compile error! الكومبايلر بيمنعك
String s = list.get(0); // مفيش حاجة لـ casting
```

---

## 1. Generic Classes

### الشكل الأساسي

```java
public class Box<T> {
    private T content;

    public void put(T item) {
        this.content = item;
    }

    public T get() {
        return content;
    }
}
```

ال `T` هنا اسمه **Type Parameter** - يعني "نوع هنحدده بعدين". لما تيجي تستخدم الكلاس:

```java
Box<String> stringBox = new Box<>();
stringBox.put("أهلاً");
String value = stringBox.get(); // مفيش casting

Box<Integer> intBox = new Box<>();
intBox.put(42);
Integer num = intBox.get();
```

### أكتر من Type Parameter

```java
public class Pair<K, V> {
    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
}

// الاستخدام
Pair<String, Integer> age = new Pair<>("أحمد", 25);
Pair<Integer, List<String>> data = new Pair<>(1, List.of("a", "b"));
```

### تسمية الـ Type Parameters (Convention)

| الحرف | المعنى |
|-------|--------|
| `T` | Type (نوع عام) |
| `E` | Element (عنصر في Collection) |
| `K` | Key (مفتاح) |
| `V` | Value (قيمة) |
| `N` | Number (رقم) |
| `R` | Return type (نوع الإرجاع) |
| `S, U` | أنواع إضافية لما T مش كفاية |

---

## 2. Generic Methods

مش لازم الكلاس كله يبقى Generic - ممكن method واحدة بس:

```java
public class ArrayUtils {

    // الـ <T> قبل الـ return type هي اللي بتعرّف الـ type parameter
    public static <T> T getFirst(T[] array) {
        if (array == null || array.length == 0)
            return null;
        return array[0];
    }

    public static <T> List<T> arrayToList(T[] array) {
        List<T> list = new ArrayList<>();
        for (T item : array) {
            list.add(item);
        }
        return list;
    }
}

// الاستخدام - الكومبايلر بيستنتج النوع لوحده (Type Inference)
String first = ArrayUtils.getFirst(new String[]{"a", "b", "c"});
Integer firstNum = ArrayUtils.getFirst(new Integer[]{1, 2, 3});

// أو ممكن تحدد النوع صراحةً
String first = ArrayUtils.<String>getFirst(new String[]{"a", "b"});
```

### Generic Method في Generic Class

```java
public class Box<T> {
    private T content;

    // U هنا مختلف عن T تبع الكلاس
    public <U> boolean hasSameType(Box<U> other) {
        return this.content.getClass() == other.content.getClass();
    }
}
```

---

## 3. Bounded Type Parameters (تحديد حدود النوع)

### Upper Bound - `extends`

لما تقول `<T extends Number>` يعني T لازم يكون Number أو أي حاجة بترث منه:

```java
// T لازم يكون Number أو subclass منه (Integer, Double, etc.)
public class MathBox<T extends Number> {
    private T value;

    public MathBox(T value) {
        this.value = value;
    }

    public double getDoubleValue() {
        // بما إن T extends Number، نقدر نستخدم methods بتاعة Number
        return value.doubleValue();
    }
}

MathBox<Integer> intBox = new MathBox<>(10);     // OK
MathBox<Double> doubleBox = new MathBox<>(3.14);  // OK
MathBox<String> strBox = new MathBox<>("text");   // Compile Error!
```

### Multiple Bounds

```java
// T لازم يكون extends Comparable و implements Serializable
public class SortableBox<T extends Comparable<T> & Serializable> {
    private T value;

    public boolean isGreaterThan(T other) {
        return value.compareTo(other) > 0;
    }
}
```

**القاعدة**: Class أول حاجة، بعد كده Interfaces. وبنستخدم `&` للفصل.

```java
// صح - الكلاس أولاً
<T extends Number & Comparable<T> & Serializable>

// غلط - Interface قبل Class
<T extends Comparable<T> & Number>  // Compile Error!
```

### مثال عملي - إيجاد أكبر عنصر

```java
public static <T extends Comparable<T>> T findMax(List<T> list) {
    if (list.isEmpty())
        throw new IllegalArgumentException("List is empty");

    T max = list.get(0);
    for (T item : list) {
        if (item.compareTo(max) > 0) {
            max = item;
        }
    }
    return max;
}

int maxInt = findMax(List.of(3, 1, 4, 1, 5)); // 5
String maxStr = findMax(List.of("banana", "apple", "cherry")); // "cherry"
```

---

## 4. Wildcards (الرموز العامة)

الـ Wildcards بتستخدم لما تبقى **بتستقبل** Generic type مش بتعرّفه.

### `?` - Unbounded Wildcard

يعني "أي نوع":

```java
// بتقبل List من أي نوع
public static void printAll(List<?> list) {
    for (Object item : list) {
        System.out.println(item);
    }
}

printAll(List.of(1, 2, 3));          // OK
printAll(List.of("a", "b", "c"));    // OK
printAll(List.of(true, false));      // OK
```

### `? extends T` - Upper Bounded Wildcard (القراءة)

يعني "T أو أي حاجة بترث من T" - **للقراءة بس**:

```java
// بتقبل List<Number> أو List<Integer> أو List<Double>
public static double sum(List<? extends Number> list) {
    double total = 0;
    for (Number num : list) {    // القراءة آمنة - كل حاجة Number
        total += num.doubleValue();
    }
    return total;
}

List<Integer> ints = List.of(1, 2, 3);
List<Double> doubles = List.of(1.1, 2.2, 3.3);

sum(ints);    // OK - Integer extends Number
sum(doubles); // OK - Double extends Number
```

**ليه مينفعش تكتب فيها؟**

```java
List<? extends Number> list = new ArrayList<Integer>();
// list.add(3.14);  // Compile Error! ممكن تكون List<Integer>
// list.add(1);     // Compile Error! ممكن تكون List<Double>
list.add(null);     // ده الحاجة الوحيدة اللي مسموحة
```

### `? super T` - Lower Bounded Wildcard (الكتابة)

يعني "T أو أي حاجة T بيرث منها" - **للكتابة**:

```java
// بتقبل List<Integer> أو List<Number> أو List<Object>
public static void addNumbers(List<? super Integer> list) {
    list.add(1);     // OK - Integer مضمون إنه يتقبل
    list.add(2);     // OK
    list.add(3);     // OK
}

List<Integer> ints = new ArrayList<>();
List<Number> nums = new ArrayList<>();
List<Object> objs = new ArrayList<>();

addNumbers(ints);  // OK
addNumbers(nums);  // OK
addNumbers(objs);  // OK
```

**ليه القراءة منها مش مضمونة؟**

```java
List<? super Integer> list = new ArrayList<Number>();
Object item = list.get(0); // بترجع Object بس - مش عارفين النوع الحقيقي
```

### PECS - Producer Extends, Consumer Super

دي القاعدة الذهبية:

| الحالة | استخدم | السبب |
|--------|--------|-------|
| بتقرأ من الـ Collection (Producer) | `? extends T` | عارف إن كل حاجة هي T أو subtype |
| بتكتب في الـ Collection (Consumer) | `? super T` | عارف إن T مقبول دايماً |
| بتقرأ وتكتب | `T` بدون wildcard | محتاج النوع المحدد |

```java
// مثال عملي - نسخ من source لـ destination
public static <T> void copy(List<? extends T> source, List<? super T> dest) {
    for (T item : source) {  // source = Producer = extends
        dest.add(item);       // dest = Consumer = super
    }
}

List<Integer> ints = List.of(1, 2, 3);
List<Number> nums = new ArrayList<>();
copy(ints, nums); // OK! Integer extends Number
```

---

## 5. Type Erasure (مسح الأنواع)

### إيه اللي بيحصل وقت الـ Compile؟

Java بتشيل كل الـ Generic type information وقت الـ compilation. ده اسمه **Type Erasure**.

```java
// اللي إنت كاتبه
public class Box<T> {
    private T content;
    public T get() { return content; }
}

// اللي الكومبايلر بيحوله لـ (Bytecode)
public class Box {
    private Object content;
    public Object get() { return content; }
}
```

لو فيه bound:

```java
// اللي إنت كاتبه
public class MathBox<T extends Number> {
    private T value;
}

// بعد الـ Erasure
public class MathBox {
    private Number value;  // بيتحول لأول bound
}
```

### نتائج الـ Type Erasure

**1. مينفعش تعمل `new T()` أو `new T[]`:**

```java
public class Box<T> {
    // Compile Error! - مش عارفين T إيه وقت الـ runtime
    // T item = new T();
    // T[] array = new T[10];

    // الحل - استخدم Class<T>
    public T createInstance(Class<T> clazz) throws Exception {
        return clazz.getDeclaredConstructor().newInstance();
    }
}
```

**2. مينفعش تعمل `instanceof` مع Generic type:**

```java
// Compile Error!
// if (obj instanceof List<String>) { }

// الصح
if (obj instanceof List<?>) { }
```

**3. مينفعش تعمل overload بأنواع Generic مختلفة:**

```java
public class Printer {
    // Compile Error! - بعد الـ erasure الاتنين بيبقوا نفس الـ signature
    // public void print(List<String> list) { }
    // public void print(List<Integer> list) { }

    // الحل
    public void printStrings(List<String> list) { }
    public void printIntegers(List<Integer> list) { }
}
```

**4. Static members مينفعش يستخدموا Type Parameter بتاع الكلاس:**

```java
public class Box<T> {
    // Compile Error!
    // private static T value;
    // public static T getValue() { return value; }

    // بس ممكن يبقى عندهم Type Parameter خاص بيهم
    public static <U> U doSomething(U input) { return input; } // OK
}
```

---

## 6. Generic Interfaces

```java
public interface Repository<T, ID> {
    T findById(ID id);
    List<T> findAll();
    void save(T entity);
    void delete(T entity);
}

// تطبيق مع تحديد الأنواع
public class UserRepository implements Repository<User, Long> {
    @Override
    public User findById(Long id) { /* ... */ }

    @Override
    public List<User> findAll() { /* ... */ }

    @Override
    public void save(User entity) { /* ... */ }

    @Override
    public void delete(User entity) { /* ... */ }
}

// أو تطبيق مع الحفاظ على الـ Generic
public class GenericRepository<T> implements Repository<T, Long> {
    @Override
    public T findById(Long id) { /* ... */ }
    // ...
}
```

### Comparable - أشهر Generic Interface

```java
public class Student implements Comparable<Student> {
    private String name;
    private double gpa;

    @Override
    public int compareTo(Student other) {
        return Double.compare(this.gpa, other.gpa);
    }
}

List<Student> students = new ArrayList<>();
Collections.sort(students); // بيستخدم compareTo
```

---

## 7. Generic Inheritance و Subtypes

### قاعدة مهمة جداً

`List<Integer>` مش subtype من `List<Number>` حتى لو `Integer extends Number`!

```java
List<Number> nums;
List<Integer> ints = List.of(1, 2, 3);

// Compile Error! مش نفس النوع
// nums = ints;

// ليه؟ لأن لو ده كان مسموح:
// nums.add(3.14); // كنا هنضيف Double في List<Integer>!
```

**الحل بالـ Wildcards:**

```java
List<? extends Number> nums = ints; // OK - بس للقراءة بس
```

### Generic Class Inheritance

```java
public class Animal { }
public class Dog extends Animal { }

public class Box<T> { }
public class SpecialBox<T> extends Box<T> { }

// ده شغال - SpecialBox extends Box
Box<Dog> box = new SpecialBox<Dog>(); // OK

// ده مش شغال - Box<Dog> مش subtype من Box<Animal>
// Box<Animal> box = new Box<Dog>(); // Error!
```

---

## 8. Recursive Type Bounds

لما النوع بيشير لنفسه:

```java
// T لازم يكون Comparable مع نفسه
public class SortedList<T extends Comparable<T>> {
    private List<T> items = new ArrayList<>();

    public void add(T item) {
        items.add(item);
        Collections.sort(items);
    }
}
```

### الـ Builder Pattern مع Generics

```java
public abstract class Builder<T extends Builder<T>> {
    private String name;

    @SuppressWarnings("unchecked")
    public T withName(String name) {
        this.name = name;
        return (T) this;  // بيرجع النوع الفعلي مش Builder
    }
}

public class UserBuilder extends Builder<UserBuilder> {
    private int age;

    public UserBuilder withAge(int age) {
        this.age = age;
        return this;
    }
}

// Method chaining بيشتغل صح
UserBuilder builder = new UserBuilder()
    .withName("أحمد")   // بيرجع UserBuilder مش Builder
    .withAge(25);       // فنقدر نستخدم withAge
```

---

## 9. Generic Constructors

```java
public class Converter {
    // Constructor عنده Type Parameter خاص بيه
    public <T> Converter(T item) {
        System.out.println("Converting: " + item.getClass().getName());
    }
}

Converter c1 = new Converter("text");
Converter c2 = new Converter(123);
```

---

## 10. Intersection Types

```java
// T لازم يكون implements Serializable و Comparable في نفس الوقت
public static <T extends Serializable & Comparable<T>> void process(T item) {
    // نقدر نستخدم methods من الاتنين
}
```

---

## 11. Generic Records (Java 16+)

```java
public record Pair<A, B>(A first, B second) { }

Pair<String, Integer> pair = new Pair<>("age", 25);
String key = pair.first();
Integer value = pair.second();
```

---

## 12. Diamond Operator `<>`

من Java 7، مش محتاج تكرر الأنواع:

```java
// قبل Java 7
Map<String, List<Integer>> map = new HashMap<String, List<Integer>>();

// من Java 7 - الكومبايلر بيستنتج
Map<String, List<Integer>> map = new HashMap<>();

// من Java 10 - var كمان
var map = new HashMap<String, List<Integer>>();
```

---

## 13. أمثلة عملية متقدمة

### Generic DAO Pattern

```java
public abstract class GenericDAO<T, ID extends Serializable> {
    private Class<T> entityClass;

    public GenericDAO(Class<T> entityClass) {
        this.entityClass = entityClass;
    }

    public T findById(ID id) {
        return entityManager.find(entityClass, id);
    }

    public List<T> findAll() {
        return entityManager
            .createQuery("FROM " + entityClass.getName(), entityClass)
            .getResultList();
    }

    public void save(T entity) {
        entityManager.persist(entity);
    }
}

public class UserDAO extends GenericDAO<User, Long> {
    public UserDAO() {
        super(User.class);
    }
}
```

### Generic Event System

```java
public interface EventListener<E> {
    void onEvent(E event);
}

public class EventBus {
    private Map<Class<?>, List<EventListener<?>>> listeners = new HashMap<>();

    public <E> void register(Class<E> eventType, EventListener<E> listener) {
        listeners.computeIfAbsent(eventType, k -> new ArrayList<>()).add(listener);
    }

    @SuppressWarnings("unchecked")
    public <E> void publish(E event) {
        List<EventListener<?>> list = listeners.get(event.getClass());
        if (list != null) {
            for (EventListener<?> listener : list) {
                ((EventListener<E>) listener).onEvent(event);
            }
        }
    }
}

// الاستخدام
EventBus bus = new EventBus();
bus.register(OrderCreated.class, event -> {
    System.out.println("Order: " + event.getOrderId());
});
```

### Generic Response Wrapper

```java
public class ApiResponse<T> {
    private boolean success;
    private T data;
    private String errorMessage;

    public static <T> ApiResponse<T> ok(T data) {
        ApiResponse<T> response = new ApiResponse<>();
        response.success = true;
        response.data = data;
        return response;
    }

    public static <T> ApiResponse<T> error(String message) {
        ApiResponse<T> response = new ApiResponse<>();
        response.success = false;
        response.errorMessage = message;
        return response;
    }
}

ApiResponse<User> response = ApiResponse.ok(new User("أحمد"));
ApiResponse<List<Order>> orders = ApiResponse.ok(orderList);
ApiResponse<Void> error = ApiResponse.error("Not found");
```

### Type-Safe Heterogeneous Container

```java
public class TypeSafeMap {
    private Map<Class<?>, Object> map = new HashMap<>();

    public <T> void put(Class<T> type, T value) {
        map.put(type, value);
    }

    public <T> T get(Class<T> type) {
        return type.cast(map.get(type));
    }
}

TypeSafeMap config = new TypeSafeMap();
config.put(String.class, "hello");
config.put(Integer.class, 42);
config.put(List.class, List.of(1, 2, 3));

String s = config.get(String.class);    // "hello" - type safe
Integer i = config.get(Integer.class);  // 42 - no casting needed
```

---

## 14. أخطاء شائعة وحلولها

### 1. Raw Types

```java
// غلط - Raw type (من غير Generic)
List list = new ArrayList();
list.add("text");
list.add(123);  // مفيش حماية

// صح
List<String> list = new ArrayList<>();
```

### 2. نسيان الـ Diamond

```java
// Warning - Raw type على اليمين
List<String> list = new ArrayList(); // يفضل new ArrayList<>()
```

### 3. Wildcard Capture Error

```java
public static void swap(List<?> list, int i, int j) {
    // Compile Error! مش عارفين نوع العنصر
    // list.set(i, list.get(j));

    // الحل - Helper method
    swapHelper(list, i, j);
}

private static <T> void swapHelper(List<T> list, int i, int j) {
    T temp = list.get(i);
    list.set(i, list.get(j));
    list.set(j, temp);
}
```

### 4. Generic Array Creation

```java
// Compile Error!
// List<String>[] array = new List<String>[10];

// الحل
@SuppressWarnings("unchecked")
List<String>[] array = (List<String>[]) new List<?>[10];

// أو الأفضل - استخدم List of Lists
List<List<String>> listOfLists = new ArrayList<>();
```

---

## 15. ملخص سريع

| المفهوم | الصيغة | الاستخدام |
|---------|--------|-----------|
| Generic Class | `class Box<T>` | كلاس بيشتغل مع أي نوع |
| Generic Method | `<T> T method(T param)` | method بنوع مرن |
| Bounded Type | `<T extends Number>` | تحديد الأنواع المقبولة |
| Unbounded Wildcard | `<?>` | أي نوع - للقراءة بس |
| Upper Bounded | `<? extends T>` | T أو subtypes - Producer |
| Lower Bounded | `<? super T>` | T أو supertypes - Consumer |
| Multiple Bounds | `<T extends A & B>` | لازم يحقق كل الشروط |
| Diamond | `new Box<>()` | الكومبايلر يستنتج النوع |

### القواعد الذهبية

1. **PECS**: Producer = `extends`، Consumer = `super`
2. **Type Erasure**: الأنواع بتتمسح وقت الـ runtime
3. **Invariance**: `List<Dog>` مش `List<Animal>`
4. **مينفعش**: `new T()` أو `new T[]` أو `instanceof T`
5. **استخدم Wildcards** لما تبقى بتستقبل Generic type
6. **استخدم Type Parameters** لما تبقى بتعرّف Generic type
