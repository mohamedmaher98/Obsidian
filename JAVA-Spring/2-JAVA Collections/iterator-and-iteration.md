# Iterator و Iteration في Java Collections

## المقدمة

تخيل إن عندك درج فيه ملفات كتير، وعايز تعدي على كل ملف واحد واحد - تفتحه، تشوف اللي جواه، وبعدين تنتقل للي بعده. الـ Iterator في Java هو بالظبط الفكرة دي: أداة بتخليك تمشي على عناصر أي Collection واحد واحد، من غير ما تحتاج تعرف هي متخزنة إزاي من جوا.

---

## الفرق بين Iterable و Iterator

قبل ما ندخل في التفاصيل، لازم نفهم إن في حاجتين مختلفين:

### Iterable (الحاجة اللي تقدر تلف عليها)

الـ `Iterable<T>` هو interface بيقول: "أنا حاجة تقدر تلف عليها". أي class بينفذ الـ interface ده لازم يوفر method واحدة:

```java
public interface Iterable<T> {
    Iterator<T> iterator();
}
```

كل الـ Collections في Java (مثل `List`, `Set`, `Queue`) بتنفذ `Iterable`. وده السبب إنك تقدر تستخدم for-each loop عليهم.

### Iterator (الأداة اللي بتلف بيها)

الـ `Iterator<T>` هو interface بيمثل الأداة نفسها اللي بتستخدمها عشان تتحرك بين العناصر:

```java
public interface Iterator<T> {
    boolean hasNext();  // في عنصر تاني ولا لأ؟
    T next();           // هات العنصر الجاي
    void remove();      // امسح العنصر الحالي (optional)
}
```

يعني الـ Iterable بيقولك "هاتلك Iterator"، والـ Iterator هو اللي بيعمل الشغل الفعلي.

---

## الطرق المختلفة للـ Iteration

### 1. for-each Loop (الطريقة الأبسط)

```java
List<String> names = List.of("Ahmed", "Sara", "Omar");

for (String name : names) {
    System.out.println(name);
}
```

الـ compiler بيحول الكود ده لـ Iterator من وراك. يعني الكود اللي فوق ده بيتحول لحاجة زي كده:

```java
Iterator<String> it = names.iterator();
while (it.hasNext()) {
    String name = it.next();
    System.out.println(name);
}
```

**امتى تستخدمها؟** لما عايز تقرأ كل العناصر بس، من غير ما تحتاج تعدل أو تمسح أي حاجة.

### 2. Iterator بشكل صريح

```java
List<String> names = new ArrayList<>(List.of("Ahmed", "Sara", "Omar"));

Iterator<String> it = names.iterator();
while (it.hasNext()) {
    String name = it.next();
    if (name.equals("Sara")) {
        it.remove(); // نقدر نمسح بأمان أثناء اللف
    }
}
// names = ["Ahmed", "Omar"]
```

**امتى تستخدمها؟** لما محتاج تمسح عناصر أثناء اللف. لو حاولت تمسح من الـ List مباشرة جوا for-each loop، هتاخد `ConcurrentModificationException`.

### 3. ListIterator (للـ Lists بس)

الـ `ListIterator` هو نسخة محسنة من الـ Iterator خاصة بالـ Lists، وبيديك قدرات إضافية:

```java
List<String> names = new ArrayList<>(List.of("Ahmed", "Sara", "Omar"));

ListIterator<String> lit = names.listIterator();

// التحرك للأمام
while (lit.hasNext()) {
    int index = lit.nextIndex();  // بيديك الـ index كمان
    String name = lit.next();
    System.out.println(index + ": " + name);
}

// التحرك للخلف!
while (lit.hasPrevious()) {
    String name = lit.previous();
    System.out.println("رجوع: " + name);
}
```

#### قدرات الـ ListIterator الإضافية

```java
ListIterator<String> lit = names.listIterator();

lit.next();        // Ahmed
lit.set("Khaled"); // غير Ahmed لـ Khaled
lit.next();        // Sara
lit.add("Mona");   // أضف Mona بعد Sara

// names = ["Khaled", "Sara", "Mona", "Omar"]
```

الـ methods المتاحة:

| Method | الوظيفة |
|---|---|
| `hasNext()` | في عنصر قدام؟ |
| `next()` | هات العنصر الجاي |
| `hasPrevious()` | في عنصر ورا؟ |
| `previous()` | هات العنصر اللي فات |
| `nextIndex()` | index العنصر الجاي |
| `previousIndex()` | index العنصر اللي فات |
| `remove()` | امسح آخر عنصر اترجع |
| `set(E e)` | غير آخر عنصر اترجع |
| `add(E e)` | أضف عنصر في المكان الحالي |

### 4. forEach() Method (Java 8+)

```java
List<String> names = List.of("Ahmed", "Sara", "Omar");

names.forEach(name -> System.out.println(name));

// أو باستخدام method reference
names.forEach(System.out::println);
```

### 5. Stream API (Java 8+)

```java
List<String> names = List.of("Ahmed", "Sara", "Omar", "Ali");

// فلترة وتحويل
List<String> result = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .toList();
// result = ["AHMED", "SARA", "OMAR"]
```

### 6. الـ for Loop التقليدي (بالـ index)

```java
List<String> names = List.of("Ahmed", "Sara", "Omar");

for (int i = 0; i < names.size(); i++) {
    System.out.println(i + ": " + names.get(i));
}
```

::: warning
الطريقة دي كويسة مع `ArrayList` لإن الوصول بالـ index سريع O(1). لكن مع `LinkedList` الوصول بالـ index بطيء O(n) لإنه بيمشي من الأول كل مرة، فاستخدم Iterator بدلها.
:::

---

## Iteration على أنواع Collections المختلفة

### Map Iteration

الـ `Map` مش بينفذ `Iterable` بشكل مباشر، لكن بيوفرلك ٣ طرق تلف عليه:

```java
Map<String, Integer> scores = Map.of("Ahmed", 95, "Sara", 88, "Omar", 72);

// 1. اللف على الـ entries (الأكثر استخداما)
for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

// 2. اللف على الـ keys بس
for (String name : scores.keySet()) {
    System.out.println(name);
}

// 3. اللف على الـ values بس
for (Integer score : scores.values()) {
    System.out.println(score);
}

// 4. forEach مع lambda
scores.forEach((name, score) -> {
    System.out.println(name + " جاب " + score);
});
```

### Set Iteration

```java
Set<String> colors = Set.of("Red", "Green", "Blue");

for (String color : colors) {
    System.out.println(color);
}
// الترتيب مش مضمون في HashSet
```

### Queue / Deque Iteration

```java
Queue<String> queue = new LinkedList<>(List.of("First", "Second", "Third"));

// اللف العادي (من غير ما تشيل عناصر)
for (String item : queue) {
    System.out.println(item);
}

// لو عايز تستهلك الـ queue (تشيل كل عنصر بعد ما تقرأه)
while (!queue.isEmpty()) {
    String item = queue.poll();
    System.out.println("Processing: " + item);
}
```

---

## ConcurrentModificationException - المشكلة الشهيرة

### المشكلة

```java
List<String> names = new ArrayList<>(List.of("Ahmed", "Sara", "Omar"));

// ده هيرمي ConcurrentModificationException!
for (String name : names) {
    if (name.equals("Sara")) {
        names.remove(name); // تعديل الـ List أثناء اللف
    }
}
```

لما بتعدل Collection أثناء ما Iterator شغال عليها، الـ Iterator بيكتشف إن في حاجة اتغيرت ورا ضهره وبيرمي exception. الفكرة دي اسمها **fail-fast behavior**.

### الحلول

#### الحل 1: استخدم `Iterator.remove()`

```java
Iterator<String> it = names.iterator();
while (it.hasNext()) {
    String name = it.next();
    if (name.equals("Sara")) {
        it.remove(); // آمن - بيعدل من خلال الـ Iterator نفسه
    }
}
```

#### الحل 2: استخدم `removeIf()` (Java 8+)

```java
names.removeIf(name -> name.equals("Sara"));
```

الطريقة دي أنظف وأبسط، وبتستخدم Iterator من جواها.

#### الحل 3: اجمع العناصر اللي عايز تمسحها واحذفها بعدين

```java
List<String> toRemove = new ArrayList<>();
for (String name : names) {
    if (name.equals("Sara")) {
        toRemove.add(name);
    }
}
names.removeAll(toRemove);
```

#### الحل 4: استخدم Streams

```java
names = names.stream()
    .filter(name -> !name.equals("Sara"))
    .collect(Collectors.toList());
```

---

## كتابة Iterator خاص بيك

لو عندك data structure خاصة بيك وعايز تقدر تلف عليها، لازم تنفذ `Iterable` و `Iterator`.

### مثال: Range Iterator

تخيل إنك عايز تعمل حاجة زي `range(1, 5)` اللي في Python:

```java
public class Range implements Iterable<Integer> {
    private final int start;
    private final int end;

    public Range(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    public Iterator<Integer> iterator() {
        return new RangeIterator();
    }

    private class RangeIterator implements Iterator<Integer> {
        private int current = start;

        @Override
        public boolean hasNext() {
            return current < end;
        }

        @Override
        public Integer next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            return current++;
        }
    }
}
```

واستخدامه:

```java
for (int num : new Range(1, 5)) {
    System.out.println(num); // 1, 2, 3, 4
}
```

### مثال: Iterator على شجرة ثنائية

```java
public class BinaryTree<T> implements Iterable<T> {
    private T value;
    private BinaryTree<T> left, right;

    @Override
    public Iterator<T> iterator() {
        // In-order traversal
        List<T> elements = new ArrayList<>();
        inOrder(this, elements);
        return elements.iterator();
    }

    private void inOrder(BinaryTree<T> node, List<T> list) {
        if (node == null) return;
        inOrder(node.left, list);
        list.add(node.value);
        inOrder(node.right, list);
    }
}
```

---

## Spliterator (Java 8+)

الـ `Spliterator` (اختصار Split-Iterator) هو Iterator متقدم اتصمم خصيصا عشان يشتغل مع الـ parallel processing والـ Streams.

### الفرق بينه وبين Iterator العادي

| الخاصية | Iterator | Spliterator |
|---|---|---|
| التقسيم | لأ | ايوا - يقدر يقسم نفسه لجزئين |
| حجم العناصر | معرفوش | يقدر يقدر الحجم |
| Parallel | لأ | ايوا |
| الاستخدام | for-each, while | Streams |

### إزاي بيشتغل

```java
List<String> names = List.of("Ahmed", "Sara", "Omar", "Ali", "Mona", "Khaled");

Spliterator<String> spliterator1 = names.spliterator();
Spliterator<String> spliterator2 = spliterator1.trySplit();
// spliterator2 أخد نص العناصر
// spliterator1 فاضله النص التاني

// كل واحد يقدر يشتغل في thread مختلف
spliterator1.forEachRemaining(System.out::println);
spliterator2.forEachRemaining(System.out::println);
```

### خصائص الـ Spliterator

```java
Spliterator<String> sp = names.spliterator();

// التحقق من الخصائص
sp.characteristics(); // بيرجع int فيه flags

// الخصائص المتاحة:
// ORDERED    - العناصر ليها ترتيب محدد
// DISTINCT   - مفيش عناصر مكررة
// SORTED     - العناصر مترتبة
// SIZED      - عارفين عدد العناصر
// NONNULL    - مفيش null values
// IMMUTABLE  - مش هيتغير
// CONCURRENT - آمن مع threads
// SUBSIZED   - الأجزاء المقسومة ليها حجم معروف

// تقدير عدد العناصر
long size = sp.estimateSize(); // 6
```

---

## Enumeration (القديم)

الـ `Enumeration` هو سلف الـ Iterator، كان موجود من Java 1.0 مع الـ legacy collections زي `Vector` و `Hashtable`:

```java
Vector<String> vector = new Vector<>(List.of("A", "B", "C"));

Enumeration<String> e = vector.elements();
while (e.hasMoreElements()) {
    System.out.println(e.nextElement());
}
```

### الفرق بين Enumeration و Iterator

| الخاصية | Enumeration | Iterator |
|---|---|---|
| متى ظهر | Java 1.0 | Java 1.2 |
| المسح أثناء اللف | لأ | ايوا (`remove()`) |
| Fail-fast | لأ | ايوا |
| أسماء الـ methods | `hasMoreElements()`, `nextElement()` | `hasNext()`, `next()` |
| الاستخدام | Legacy فقط | الاستخدام الحالي |

::: warning
متستخدمش `Enumeration` في كود جديد. استخدم `Iterator` أو for-each بدله.
:::

---

## مقارنة الأداء

### أي طريقة Iteration أسرع؟

| الطريقة | ArrayList | LinkedList | HashSet |
|---|---|---|---|
| for-each | سريع | سريع | سريع |
| Iterator | سريع | سريع | سريع |
| for (index) | سريع | **بطيء جدا** | غير متاح |
| forEach() | سريع | سريع | سريع |
| Stream | overhead بسيط | overhead بسيط | overhead بسيط |
| Parallel Stream | ممتاز للبيانات الكبيرة | مش مثالي | حسب الحجم |

::: tip
في الحالات العادية، الفرق بين الطرق المختلفة بسيط جدا. اختار الطريقة اللي بتخلي الكود أوضح وأسهل في القراءة. لو عندك ملايين العناصر وعايز أداء، ابدأ قيس بالـ benchmarks.
:::

---

## أنماط شائعة ومفيدة

### Pattern 1: تحويل Iterator لـ Stream

```java
Iterator<String> it = someOldApi.getIterator();

// تحويل Iterator لـ Stream
Stream<String> stream = StreamSupport.stream(
    Spliterators.spliteratorUnknownSize(it, Spliterator.ORDERED),
    false
);

List<String> result = stream.filter(s -> s.length() > 3).toList();
```

### Pattern 2: Iterator مع حالة (Stateful Iterator)

```java
// Iterator بيقرأ من ملف سطر سطر
public class FileLineIterator implements Iterator<String>, AutoCloseable {
    private BufferedReader reader;
    private String nextLine;

    public FileLineIterator(Path path) throws IOException {
        this.reader = Files.newBufferedReader(path);
        this.nextLine = reader.readLine();
    }

    @Override
    public boolean hasNext() {
        return nextLine != null;
    }

    @Override
    public String next() {
        if (!hasNext()) throw new NoSuchElementException();
        String current = nextLine;
        try {
            nextLine = reader.readLine();
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
        return current;
    }

    @Override
    public void close() throws IOException {
        reader.close();
    }
}
```

### Pattern 3: Filtering Iterator (Decorator Pattern)

```java
public class FilteringIterator<T> implements Iterator<T> {
    private final Iterator<T> source;
    private final Predicate<T> predicate;
    private T nextItem;
    private boolean hasNext;

    public FilteringIterator(Iterator<T> source, Predicate<T> predicate) {
        this.source = source;
        this.predicate = predicate;
        advance();
    }

    private void advance() {
        while (source.hasNext()) {
            T item = source.next();
            if (predicate.test(item)) {
                nextItem = item;
                hasNext = true;
                return;
            }
        }
        hasNext = false;
    }

    @Override
    public boolean hasNext() {
        return hasNext;
    }

    @Override
    public T next() {
        if (!hasNext) throw new NoSuchElementException();
        T result = nextItem;
        advance();
        return result;
    }
}

// الاستخدام:
Iterator<String> filtered = new FilteringIterator<>(
    names.iterator(),
    name -> name.startsWith("A")
);
```

### Pattern 4: Composite Iterator (لف على أكتر من Collection)

```java
public class CompositeIterator<T> implements Iterator<T> {
    private final Queue<Iterator<T>> iterators;

    @SafeVarargs
    public CompositeIterator(Iterator<T>... iterators) {
        this.iterators = new LinkedList<>(Arrays.asList(iterators));
    }

    @Override
    public boolean hasNext() {
        while (!iterators.isEmpty()) {
            if (iterators.peek().hasNext()) {
                return true;
            }
            iterators.poll();
        }
        return false;
    }

    @Override
    public T next() {
        if (!hasNext()) throw new NoSuchElementException();
        return iterators.peek().next();
    }
}

// الاستخدام: اللف على ٣ lists كأنهم list واحدة
Iterator<String> combined = new CompositeIterator<>(
    list1.iterator(), list2.iterator(), list3.iterator()
);
```

---

## Thread Safety والـ Concurrent Collections

الـ Iterators العادية مش thread-safe. لو عندك أكتر من thread بيعدلوا في Collection، استخدم الـ Concurrent Collections:

### ConcurrentHashMap

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("A", 1);
map.put("B", 2);

// الـ Iterator هنا weakly consistent
// مش هيرمي ConcurrentModificationException
// لكن ممكن ميعرضش تعديلات حصلت بعد ما الـ Iterator اتعمل
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    map.put("C", 3); // آمن - مش هيرمي exception
}
```

### CopyOnWriteArrayList

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>(List.of("A", "B", "C"));

// الـ Iterator بيشتغل على snapshot من الـ List
for (String item : list) {
    list.add("D"); // آمن - الـ Iterator شغال على نسخة قديمة
}
```

::: warning
`CopyOnWriteArrayList` بيعمل نسخة من الـ array كل ما تعدل فيه. ده كويس لو القراءة أكتر من الكتابة بكتير، لكن لو بتعدل كتير هيبقى بطيء ويستهلك memory.
:::

---

## ملخص سريع: امتى تستخدم إيه

| الموقف | استخدم |
|---|---|
| قراءة كل العناصر | for-each loop |
| محتاج تمسح أثناء اللف | `Iterator` + `remove()` أو `removeIf()` |
| محتاج تمشي للأمام وللخلف | `ListIterator` |
| محتاج تعدل أثناء اللف | `ListIterator` + `set()` / `add()` |
| فلترة وتحويل | Streams |
| Parallel processing | Parallel Streams / `Spliterator` |
| Thread safety | Concurrent Collections |
| Collection كبيرة + lazy evaluation | Streams |
