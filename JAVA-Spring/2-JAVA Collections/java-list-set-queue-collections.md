# List, Set, Queue - الدليل الشامل

## الصورة الكبيرة: ليه في أكتر من نوع Collection؟

تخيل إنك بتنظم حاجات في حياتك اليومية:

- ال**List** = زي قايمة المشتريات: فيها ترتيب، وممكن تكتب نفس الحاجة مرتين "عيش، جبنة، عيش"
-ال **Set** = زي مجموعة أرقام تليفونات الأصحاب: مفيش رقم مكرر، ومش مهم الترتيب
-ال **Queue** = زي طابور البنك: اللي يجي الأول يتخدم الأول

كل واحدة فيهم بتحل مشكلة مختلفة، وعشان كده Java وفرتلك التلاتة.

### شجرة الـ Collection Framework

```
                    Collection (interface)
                   /         |           \
                  /          |            \
              List          Set          Queue
             /    \        / | \          / \
            /      \      /  |  \        /   \
     ArrayList  LinkedList  HashSet  TreeSet  PriorityQueue  Deque
                    ↑              |                          |
                    |         LinkedHashSet              ArrayDeque
                    |                                    LinkedList
                    +--- (بينفذ List و Deque الاتنين)
```

---
---

# 1. List - القايمة المرتبة

## الفكرة

الـ List هي collection مرتبة، كل عنصر فيها ليه **index** (رقم ترتيبه)، وبتسمح بالتكرار. يعني تقدر تحط نفس القيمة أكتر من مرة.

خصائص أي List:
- **مرتبة**: العناصر ليها ترتيب ثابت (اللي دخل أول هيفضل أول)
- **بتسمح بالتكرار**: تقدر تحط `"Ahmed"` ١٠ مرات
- **بتسمح بـ null**: تقدر تحط `null` كعنصر
- **Indexed**: كل عنصر ليه index من 0 لـ size-1

---

## أنواع الـ List

### ArrayList

#### إزاي بتشتغل من جوا

الـ ArrayList من جواها عبارة عن **array عادية** بس بتكبر لوحدها. تخيلها كده:

```
العناصر: ["Ahmed", "Sara", "Omar", null, null, null]
                                      ^
                                      |
           المساحة الفاضية (capacity = 6, size = 3)
```

لما بتعمل `new ArrayList<>()` بيحصل الآتي:
1. بيتعمل array فاضية بحجم **10** (الـ default capacity)
2. كل ما تضيف عنصر، بيتحط في أول مكان فاضي
3. لما الـ array تتملي، بيتعمل array جديدة **حجمها 1.5x الحجم القديم** وبيتنقل كل حاجة فيها

```
// المرحلة 1: array حجمها 10
[A, B, C, D, E, F, G, H, I, J]  ← مليانة!

// المرحلة 2: بيتعمل array جديدة حجمها 15
[A, B, C, D, E, F, G, H, I, J, K, _, _, _, _]  ← اتنقلت + اتضاف K
```

عملية النقل دي (اسمها **resize** أو **grow**) مكلفة لإنها بتنسخ كل العناصر. بس بتحصل نادر.

#### أداء الـ ArrayList

| العملية | السرعة | السبب |
|---|---|---|
| `get(index)` | **O(1)** فوري | لإنها array - بيروح للـ index مباشرة |
| `add(element)` | **O(1)** غالبا | بيحط في الآخر، إلا لو محتاج resize |
| `add(index, element)` | **O(n)** بطيء | لإنه بيزحلق كل العناصر اللي بعد الـ index |
| `remove(index)` | **O(n)** بطيء | لإنه بيزحلق كل العناصر عشان يملي الفراغ |
| `contains(element)` | **O(n)** بطيء | بيدور عنصر عنصر من الأول |
| `size()` | **O(1)** فوري | متخزن في متغير |

#### امتى تستخدم ArrayList

- لما بتقرأ أكتر ما بتكتب (أغلب الحالات)
- لما محتاج توصل لعناصر بالـ index كتير
- لما مش بتضيف أو تمسح من النص كتير

```java
// إنشاء
ArrayList<String> names = new ArrayList<>();

// إنشاء بـ capacity محددة (لو عارف العدد التقريبي)
ArrayList<String> names = new ArrayList<>(1000);

// إنشاء من collection تانية
ArrayList<String> copy = new ArrayList<>(existingList);
```

---

### LinkedList

#### إزاي بتشتغل من جوا

الـ LinkedList مختلفة تماما. كل عنصر عبارة عن **Node** (عقدة) فيها ٣ حاجات:
1. القيمة نفسها
2. pointer للعقدة اللي قبلها (prev)
3. pointer للعقدة اللي بعدها (next)

```
null ← [prev|"Ahmed"|next] ⇄ [prev|"Sara"|next] ⇄ [prev|"Omar"|next] → null
         ^                                                    ^
         |                                                    |
       first (head)                                       last (tail)
```

مفيش array خالص. كل عقدة عارفة جارتها بس، زي سلسلة حلقات.

#### أداء الـ LinkedList

| العملية | السرعة | السبب |
|---|---|---|
| `get(index)` | **O(n)** بطيء | لازم يمشي من الأول أو الآخر عقدة عقدة |
| `add(element)` في الآخر | **O(1)** فوري | بيضيف عقدة جديدة عند الـ tail |
| `addFirst(element)` | **O(1)** فوري | بيضيف عقدة جديدة عند الـ head |
| `remove()` من الأول أو الآخر | **O(1)** فوري | بيشيل العقدة ويعدل الـ pointers |
| `remove(index)` من النص | **O(n)** | لازم يمشي للعقدة الأول |
| `contains(element)` | **O(n)** بطيء | بيدور عقدة عقدة |

#### امتى تستخدم LinkedList

- لما بتضيف وتمسح من **أول أو آخر** القايمة كتير
- لما بتستخدمها كـ Queue أو Stack
- لما مش بتوصل بالـ index

::: warning
في **99% من الحالات** الـ ArrayList أحسن من الـ LinkedList. الـ LinkedList بتستهلك memory أكتر (كل عقدة فيها 2 pointers إضافيين) والوصول بالـ index فيها بطيء جدا. استخدمها بس لما يكون عندك سبب واضح.
:::

---

### مقارنة مباشرة: ArrayList vs LinkedList

```
العملية               ArrayList    LinkedList    الفايز
─────────────────────────────────────────────────────────
الوصول بـ index          O(1)         O(n)      ArrayList
الإضافة في الآخر        O(1)*        O(1)      تعادل
الإضافة في الأول        O(n)         O(1)      LinkedList
الإضافة في النص         O(n)         O(1)**    LinkedList**
المسح من الأول          O(n)         O(1)      LinkedList
المسح من الآخر          O(1)         O(1)      تعادل
استهلاك الـ Memory      أقل          أكتر      ArrayList
الـ Cache Performance   ممتاز        ضعيف      ArrayList

*  O(1) amortized - أحيانا O(n) لما بيعمل resize
** O(1) لو واقف على العقدة، لكن الوصول للعقدة نفسه O(n)
```

---

## كل Methods الـ List المهمة بالتفصيل

### الإضافة

```java
List<String> list = new ArrayList<>();

// 1. add - إضافة في الآخر
list.add("Ahmed");       // ["Ahmed"]
list.add("Sara");        // ["Ahmed", "Sara"]
list.add("Omar");        // ["Ahmed", "Sara", "Omar"]

// 2. add بـ index - إضافة في مكان محدد
// كل العناصر اللي بعد الـ index بتتزحلق مكان واحد
list.add(1, "Mona");     // ["Ahmed", "Mona", "Sara", "Omar"]
//                                    ↑ اتحطت هنا، Sara و Omar اتزحلقوا

// 3. addAll - إضافة collection كاملة
List<String> more = List.of("Ali", "Hana");
list.addAll(more);       // ["Ahmed", "Mona", "Sara", "Omar", "Ali", "Hana"]

// 4. addAll بـ index
list.addAll(2, List.of("X", "Y")); // ["Ahmed", "Mona", "X", "Y", "Sara", "Omar", "Ali", "Hana"]
```

### القراءة والبحث

```java
List<String> list = new ArrayList<>(List.of("Ahmed", "Sara", "Omar", "Sara", "Ali"));

// 1. get - جيب العنصر بالـ index
String first = list.get(0);    // "Ahmed"
String third = list.get(2);    // "Omar"
// list.get(10);               // IndexOutOfBoundsException!

// 2. indexOf - إيه الـ index بتاع العنصر ده؟ (أول ظهور)
int idx = list.indexOf("Sara");     // 1 (أول مكان لقاها فيه)
int idx2 = list.indexOf("Khaled");  // -1 (مش موجود)

// 3. lastIndexOf - آخر ظهور
int last = list.lastIndexOf("Sara"); // 3

// 4. contains - موجود ولا لأ؟
boolean hasSara = list.contains("Sara");     // true
boolean hasKhaled = list.contains("Khaled"); // false

// 5. containsAll - كل دول موجودين؟
boolean hasAll = list.containsAll(List.of("Ahmed", "Sara")); // true

// 6. size - العدد
int count = list.size(); // 5

// 7. isEmpty - فاضية ولا لأ؟
boolean empty = list.isEmpty(); // false

// 8. subList - جزء من القايمة (من index لـ index)
// ده VIEW مش نسخة - أي تعديل فيه بيأثر على الأصلية
List<String> sub = list.subList(1, 3); // ["Sara", "Omar"]
//                             ↑ inclusive  ↑ exclusive
```

### التعديل

```java
List<String> list = new ArrayList<>(List.of("Ahmed", "Sara", "Omar"));

// 1. set - غير عنصر في index معين (بيرجع القيمة القديمة)
String old = list.set(1, "Mona"); // old = "Sara", list = ["Ahmed", "Mona", "Omar"]

// 2. replaceAll - غير كل العناصر بـ function
list.replaceAll(name -> name.toUpperCase());
// ["AHMED", "MONA", "OMAR"]

// 3. sort - رتب
list.sort(Comparator.naturalOrder());  // ترتيب أبجدي: ["AHMED", "MONA", "OMAR"]
list.sort(Comparator.reverseOrder());  // عكس أبجدي: ["OMAR", "MONA", "AHMED"]

// ترتيب بشرط مخصص (مثلا بطول الاسم)
list.sort(Comparator.comparingInt(String::length));
```

### المسح

```java
List<String> list = new ArrayList<>(List.of("Ahmed", "Sara", "Omar", "Sara", "Ali"));

// 1. remove بالـ index - امسح العنصر في المكان ده
String removed = list.remove(0); // removed = "Ahmed", list = ["Sara", "Omar", "Sara", "Ali"]

// 2. remove بالـ object - امسح أول ظهور للعنصر ده
boolean wasRemoved = list.remove("Sara"); // true, list = ["Omar", "Sara", "Ali"]
//                                          ↑ مسح أول "Sara" بس

// تحذير مهم مع الأرقام!
List<Integer> nums = new ArrayList<>(List.of(10, 20, 30));
nums.remove(Integer.valueOf(20)); // ← بيمسح القيمة 20
// nums.remove(20);              // ← ده هيحاول يمسح index 20! هيرمي exception

// 3. removeAll - امسح كل العناصر الموجودة في الـ collection دي
list.removeAll(List.of("Omar", "Ali")); // ["Sara"]

// 4. removeIf - امسح كل عنصر ينطبق عليه الشرط
List<String> names = new ArrayList<>(List.of("Ahmed", "Sara", "Omar", "Ali", "Amr"));
names.removeIf(name -> name.startsWith("A"));
// names = ["Sara", "Omar"]

// 5. retainAll - ابقي بس العناصر الموجودة في الـ collection دي (عكس removeAll)
List<String> list2 = new ArrayList<>(List.of("A", "B", "C", "D"));
list2.retainAll(List.of("B", "D", "F"));
// list2 = ["B", "D"]

// 6. clear - امسح كل حاجة
list2.clear(); // []
```

### التحويل

```java
List<String> list = new ArrayList<>(List.of("Ahmed", "Sara", "Omar"));

// 1. toArray - حول لـ array
Object[] arr1 = list.toArray();
String[] arr2 = list.toArray(new String[0]); // الطريقة الصح عشان تحدد النوع

// 2. List.of - عمل Immutable List (مينفعش تتعدل)
List<String> immutable = List.of("A", "B", "C");
// immutable.add("D"); // UnsupportedOperationException!

// 3. List.copyOf - نسخة Immutable من list موجودة
List<String> copy = List.copyOf(list);

// 4. Collections.unmodifiableList - view مش بتتعدل لكن الأصلية لو اتعدلت هتأثر
List<String> unmodifiable = Collections.unmodifiableList(list);
// unmodifiable.add("X"); // Exception!
// لكن لو عدلت list الأصلية، unmodifiable هتتأثر

// 5. نسخة تتعدل من Immutable list
List<String> mutable = new ArrayList<>(List.of("A", "B", "C"));
mutable.add("D"); // شغال
```

---

## أمثلة عملية على الـ List

### مثال 1: تصفية وتحويل

```java
List<String> names = List.of("Ahmed Hassan", "sara ali", "OMAR KHALED", "mona");

// حول كل الأسماء لـ Title Case وصفي الأسماء الكاملة بس
List<String> fullNames = new ArrayList<>();
for (String name : names) {
    if (name.contains(" ")) {
        String[] parts = name.toLowerCase().split(" ");
        String formatted = capitalize(parts[0]) + " " + capitalize(parts[1]);
        fullNames.add(formatted);
    }
}
// fullNames = ["Ahmed Hassan", "Sara Ali", "Omar Khaled"]

// نفس الحاجة بـ Streams
List<String> fullNames2 = names.stream()
    .filter(name -> name.contains(" "))
    .map(name -> {
        String[] parts = name.toLowerCase().split(" ");
        return capitalize(parts[0]) + " " + capitalize(parts[1]);
    })
    .toList();
```

### مثال 2: إزالة المكررات مع الحفاظ على الترتيب

```java
List<String> withDuplicates = List.of("A", "B", "A", "C", "B", "D", "A");

// طريقة 1: بـ LinkedHashSet (أبسط)
List<String> unique = new ArrayList<>(new LinkedHashSet<>(withDuplicates));
// ["A", "B", "C", "D"]

// طريقة 2: بـ Stream
List<String> unique2 = withDuplicates.stream()
    .distinct()
    .toList();
// ["A", "B", "C", "D"]
```

### مثال 3: تقسيم List لمجموعات (Batching)

```java
// عايز أقسم list كبيرة لمجموعات كل واحدة 3 عناصر
List<String> all = List.of("A", "B", "C", "D", "E", "F", "G", "H");

List<List<String>> batches = new ArrayList<>();
for (int i = 0; i < all.size(); i += 3) {
    int end = Math.min(i + 3, all.size());
    batches.add(all.subList(i, end));
}
// batches = [["A","B","C"], ["D","E","F"], ["G","H"]]
```

### مثال 4: ترتيب بأكتر من معيار

```java
List<String> names = new ArrayList<>(List.of("Omar", "Ali", "Ahmed", "Sara", "Mo"));

// رتب بالطول الأول، ولو الطول متساوي رتب أبجدي
names.sort(Comparator.comparingInt(String::length)
                      .thenComparing(Comparator.naturalOrder()));
// ["Mo", "Ali", "Omar", "Sara", "Ahmed"]
//   2     3      4       4        5
//              Omar < Sara (أبجدي)
```

---
---

# 2. Set - المجموعة بدون تكرار

## الفكرة

الـ Set هي collection **مبتسمحش بالتكرار**. لو حاولت تضيف عنصر موجود أصلا، مش هيتضاف. بتستخدم الـ `equals()` و `hashCode()` عشان تعرف العنصر مكرر ولا لأ.

خصائص أي Set:
- **مفيش تكرار**: كل عنصر فريد
- **مفيش index**: متقدرش تقول `set.get(3)` - مفيش الكلام ده
- **الترتيب يعتمد على النوع**: HashSet مفيش ترتيب، LinkedHashSet بترتيب الإضافة، TreeSet مرتب

---

## أنواع الـ Set

### HashSet

#### إزاي بتشتغل من جوا

الـ HashSet من جواها عبارة عن **HashMap** (جدول هاش). كل عنصر بتضيفه بيبقى key في الـ HashMap، والـ value بيكون object وهمي ثابت.

الفكرة الأساسية:

```
أنت بتقول: set.add("Ahmed")

الـ HashSet بيعمل:
1. يحسب hashCode بتاع "Ahmed" → مثلا 63347
2. يستخدم الـ hash عشان يحدد الـ bucket (الخانة) → 63347 % 16 = 3
3. يروح لـ bucket رقم 3 ويدور لو "Ahmed" موجود ولا لأ
4. لو مش موجود → يضيفه. لو موجود → ميعملش حاجة ويرجع false.
```

تخيل الـ HashSet كدرج فيه أدراج صغيرة (buckets):

```
Bucket 0: ["Sara"]
Bucket 1: []
Bucket 2: ["Omar", "Khaled"]  ← لو اتنين ليهم نفس الـ bucket (collision)
Bucket 3: ["Ahmed"]
Bucket 4: []
...
Bucket 15: ["Ali"]
```

لما **عدد العناصر / عدد الـ buckets > 0.75** (الـ load factor)، بيتعمل **rehash**: عدد الـ buckets بيتضاعف وكل العناصر بتتوزع تاني.

#### أداء الـ HashSet

| العملية | السرعة | السبب |
|---|---|---|
| `add(element)` | **O(1)** فوري | hash → bucket → حط |
| `remove(element)` | **O(1)** فوري | hash → bucket → امسح |
| `contains(element)` | **O(1)** فوري | hash → bucket → دور |
| الترتيب | **مفيش** | العناصر بتتوزع حسب الـ hash |

::: tip
عشان الـ HashSet تشتغل صح، لازم الـ objects اللي جواها يكون عندها `hashCode()` و `equals()` متنفذين صح. لو عملت class جديد ونسيت تنفذهم، الـ Set مش هيعرف يكتشف التكرار.
:::

#### القاعدة الذهبية: hashCode و equals

```java
public class Student {
    private int id;
    private String name;

    // لازم الاتنين مع بعض!
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return id == student.id;  // طالب بنفس الـ id يبقى نفس الطالب
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}

Set<Student> students = new HashSet<>();
students.add(new Student(1, "Ahmed"));
students.add(new Student(1, "Ahmed")); // مش هيتضاف - نفس الـ id
students.add(new Student(2, "Sara"));
// students.size() = 2
```

**لو نفذت `equals` من غير `hashCode`**: ممكن عنصرين متساويين يروحوا buckets مختلفة والـ Set ميكتشفش التكرار!

---

### LinkedHashSet

#### إزاي بتشتغل من جوا

الـ LinkedHashSet هي **HashSet + Linked List**. كل عنصر بيتخزن في الـ hash table زي الـ HashSet العادي، بس كمان كل عنصر عنده pointer لـ:
- العنصر اللي اتضاف **قبله**
- العنصر اللي اتضاف **بعده**

```
Hash Table (زي HashSet):
Bucket 0: ["Sara"]
Bucket 3: ["Ahmed"]
Bucket 7: ["Omar"]

Linked List (بتحافظ على ترتيب الإضافة):
"Ahmed" → "Sara" → "Omar"
(أول واحد اتضاف → تاني واحد → تالت واحد)
```

#### أداء الـ LinkedHashSet

| العملية | السرعة | السبب |
|---|---|---|
| `add(element)` | **O(1)** | زي HashSet + تعديل الـ linked list |
| `remove(element)` | **O(1)** | زي HashSet + تعديل الـ linked list |
| `contains(element)` | **O(1)** | زي HashSet بالظبط |
| الترتيب | **ترتيب الإضافة** | بسبب الـ linked list |

#### امتى تستخدمها

لما عايز مزايا الـ HashSet (سرعة + عدم تكرار) **و** عايز تحافظ على ترتيب الإضافة.

```java
Set<String> set = new LinkedHashSet<>();
set.add("Omar");
set.add("Ahmed");
set.add("Sara");

for (String name : set) {
    System.out.println(name);
}
// مضمون الترتيب: Omar, Ahmed, Sara (بترتيب الإضافة)
```

---

### TreeSet

#### إزاي بتشتغل من جوا

الـ TreeSet من جواها عبارة عن **Red-Black Tree** (شجرة ثنائية متوازنة). كل عنصر بيتحط في مكانه الصح عشان الشجرة تفضل مرتبة.

```
          "Mona"
         /      \
      "Ahmed"   "Sara"
        \        /
      "Ali"  "Omar"
```

لما بتضيف عنصر جديد:
1. بيقارنه بالـ root
2. لو أصغر بيروح شمال، لو أكبر بيروح يمين
3. بيكرر لحد ما يلاقي مكانه
4. بيعمل rebalancing لو الشجرة مبقتش متوازنة

#### أداء الـ TreeSet

| العملية | السرعة | السبب |
|---|---|---|
| `add(element)` | **O(log n)** | بيمشي من الـ root لمكانه |
| `remove(element)` | **O(log n)** | بيدور ويمسح ويعمل rebalance |
| `contains(element)` | **O(log n)** | بيمشي في الشجرة |
| الترتيب | **مرتب دايما** | طبيعة الشجرة |

O(log n) يعني: لو عندك مليون عنصر، أقصى عدد مقارنات ≈ 20 بس.

#### العمليات الخاصة بالـ TreeSet

الـ TreeSet بينفذ `NavigableSet` وده بيديه عمليات إضافية قوية:

```java
TreeSet<Integer> scores = new TreeSet<>(List.of(55, 70, 85, 90, 42, 63, 78));
// الشجرة مرتبة تلقائيا: [42, 55, 63, 70, 78, 85, 90]

// 1. first / last - أصغر وأكبر عنصر
int lowest = scores.first();   // 42
int highest = scores.last();   // 90

// 2. lower - أكبر عنصر أصغر من القيمة دي (strictly less)
Integer below70 = scores.lower(70);   // 63

// 3. floor - أكبر عنصر أصغر من أو يساوي القيمة دي
Integer atOrBelow70 = scores.floor(70); // 70

// 4. higher - أصغر عنصر أكبر من القيمة دي (strictly greater)
Integer above70 = scores.higher(70);  // 78

// 5. ceiling - أصغر عنصر أكبر من أو يساوي القيمة دي
Integer atOrAbove70 = scores.ceiling(70); // 70

// 6. headSet - كل العناصر الأصغر من قيمة معينة
SortedSet<Integer> below78 = scores.headSet(78);        // [42, 55, 63, 70]
NavigableSet<Integer> atOrBelow78 = scores.headSet(78, true); // [42, 55, 63, 70, 78]

// 7. tailSet - كل العناصر الأكبر من أو تساوي قيمة معينة
SortedSet<Integer> from70 = scores.tailSet(70);          // [70, 78, 85, 90]
NavigableSet<Integer> above70Only = scores.tailSet(70, false); // [78, 85, 90]

// 8. subSet - العناصر بين قيمتين
SortedSet<Integer> mid = scores.subSet(55, 85);          // [55, 63, 70, 78]
NavigableSet<Integer> mid2 = scores.subSet(55, true, 85, true); // [55, 63, 70, 78, 85]

// 9. descendingSet - نسخة بترتيب عكسي
NavigableSet<Integer> desc = scores.descendingSet();     // [90, 85, 78, 70, 63, 55, 42]

// 10. pollFirst / pollLast - شيل وارجع أصغر/أكبر عنصر
int min = scores.pollFirst(); // 42, scores = [55, 63, 70, 78, 85, 90]
int max = scores.pollLast();  // 90, scores = [55, 63, 70, 78, 85]
```

#### الترتيب المخصص مع TreeSet

```java
// ترتيب عكسي
TreeSet<String> reversed = new TreeSet<>(Comparator.reverseOrder());
reversed.addAll(List.of("Ahmed", "Sara", "Omar"));
// ["Sara", "Omar", "Ahmed"]

// ترتيب بطول النص
TreeSet<String> byLength = new TreeSet<>(Comparator.comparingInt(String::length)
                                                     .thenComparing(Comparator.naturalOrder()));
byLength.addAll(List.of("Ahmed", "Sara", "Omar", "Ali", "Mo"));
// ["Mo", "Ali", "Omar", "Sara", "Ahmed"]
```

::: warning
لو الـ elements مش `Comparable` ومعملتش Comparator، هتاخد `ClassCastException` أول ما تضيف عنصر.
:::

---

### مقارنة أنواع الـ Set

```
الخاصية              HashSet       LinkedHashSet    TreeSet
───────────────────────────────────────────────────────────────
الترتيب              مفيش          ترتيب الإضافة    مرتب (طبيعي أو custom)
add/remove/contains  O(1)          O(1)             O(log n)
التنفيذ من جوا       Hash Table    Hash + LinkedList Red-Black Tree
null values          ايوا (واحد)    ايوا (واحد)       لأ*
استهلاك Memory       أقل           متوسط            أكتر

* TreeSet مش بيقبل null لإنه بيقارن العناصر ببعض
```

---

## كل Methods الـ Set المهمة بالتفصيل

### الأساسيات

```java
Set<String> set = new HashSet<>();

// 1. add - إضافة (بترجع true لو اتضاف فعلا، false لو كان موجود)
boolean added1 = set.add("Ahmed"); // true  - اتضاف
boolean added2 = set.add("Ahmed"); // false - كان موجود أصلا!
set.add("Sara");
set.add("Omar");

// 2. remove - مسح (بترجع true لو كان موجود واتمسح)
boolean removed = set.remove("Sara"); // true
boolean removed2 = set.remove("Khaled"); // false - مكانش موجود

// 3. contains - موجود ولا لأ
boolean has = set.contains("Ahmed"); // true

// 4. size و isEmpty
int count = set.size();      // 2
boolean empty = set.isEmpty(); // false

// 5. clear - امسح كل حاجة
set.clear(); // Set فاضي
```

### عمليات المجموعات الرياضية

دي أقوى حاجة في الـ Set - العمليات الرياضية:

```java
Set<String> teamA = new HashSet<>(Set.of("Ahmed", "Sara", "Omar", "Ali"));
Set<String> teamB = new HashSet<>(Set.of("Sara", "Mona", "Ali", "Khaled"));
```

#### Union (الاتحاد) - كل الأعضاء من الفريقين

```java
Set<String> union = new HashSet<>(teamA);
union.addAll(teamB);
// union = {"Ahmed", "Sara", "Omar", "Ali", "Mona", "Khaled"}
```

#### Intersection (التقاطع) - الأعضاء المشتركين بس

```java
Set<String> intersection = new HashSet<>(teamA);
intersection.retainAll(teamB);
// intersection = {"Sara", "Ali"}
```

#### Difference (الفرق) - الأعضاء في A بس ومش في B

```java
Set<String> onlyInA = new HashSet<>(teamA);
onlyInA.removeAll(teamB);
// onlyInA = {"Ahmed", "Omar"}
```

#### Symmetric Difference (الفرق المتماثل) - اللي في واحد بس مش الاتنين

```java
Set<String> symDiff = new HashSet<>(teamA);
symDiff.addAll(teamB);           // الاتحاد
Set<String> common = new HashSet<>(teamA);
common.retainAll(teamB);         // التقاطع
symDiff.removeAll(common);       // الاتحاد - التقاطع
// symDiff = {"Ahmed", "Omar", "Mona", "Khaled"}
```

#### التحقق من العلاقات

```java
Set<String> small = Set.of("Sara", "Ali");
Set<String> big = Set.of("Ahmed", "Sara", "Omar", "Ali");

// هل small subset من big؟ (كل عناصر small موجودة في big)
boolean isSubset = big.containsAll(small); // true

// هل مفيش عناصر مشتركة؟
boolean noOverlap = Collections.disjoint(teamA, Set.of("X", "Y")); // true
```

---

## أمثلة عملية على الـ Set

### مثال 1: إيجاد المكررات في List

```java
List<String> words = List.of("hello", "world", "hello", "java", "world", "hello");

Set<String> seen = new HashSet<>();
Set<String> duplicates = new HashSet<>();

for (String word : words) {
    if (!seen.add(word)) {  // add بترجع false لو كان موجود
        duplicates.add(word);
    }
}
// duplicates = {"hello", "world"}
```

### مثال 2: أول حرف متكرر في String

```java
String text = "programming";

Set<Character> seen = new HashSet<>();
for (char c : text.toCharArray()) {
    if (!seen.add(c)) {
        System.out.println("أول حرف متكرر: " + c); // 'r'
        break;
    }
}
```

### مثال 3: إزالة المكررات مع الحفاظ على الترتيب

```java
List<Integer> numbers = List.of(5, 3, 8, 3, 1, 5, 9, 1);

// LinkedHashSet بيشيل المكررات ويحافظ على الترتيب
List<Integer> unique = new ArrayList<>(new LinkedHashSet<>(numbers));
// [5, 3, 8, 1, 9]
```

### مثال 4: التحقق من Anagram (كلمتين فيهم نفس الحروف)

```java
public static boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;

    // لو الحروف المختلفة متساوية، هم anagram
    // بس ده مش كافي - لازم نعد الحروف كمان
    Map<Character, Integer> countA = new HashMap<>();
    Map<Character, Integer> countB = new HashMap<>();

    for (char c : a.toLowerCase().toCharArray()) {
        countA.merge(c, 1, Integer::sum);
    }
    for (char c : b.toLowerCase().toCharArray()) {
        countB.merge(c, 1, Integer::sum);
    }

    return countA.equals(countB);
}

isAnagram("listen", "silent"); // true
isAnagram("hello", "world");  // false
```

### مثال 5: TreeSet لنظام درجات

```java
TreeSet<Integer> grades = new TreeSet<>(List.of(95, 87, 72, 63, 55, 91, 78, 84));
// مرتبة تلقائيا: [55, 63, 72, 78, 84, 87, 91, 95]

// الطلبة اللي نجحوا (60 فأكتر)
SortedSet<Integer> passed = grades.tailSet(60);
// [63, 72, 78, 84, 87, 91, 95]

// الطلبة اللي جابوا امتياز (90 فأكتر)
SortedSet<Integer> excellent = grades.tailSet(90);
// [91, 95]

// أعلى درجة أقل من 80
Integer justBelow80 = grades.lower(80);
// 78
```

---
---

# 3. Queue - الطابور

## الفكرة

الـ Queue بتمثل **طابور**: العناصر بتدخل من ناحية وبتطلع من الناحية التانية. في أغلب الحالات، الفكرة هي **FIFO** (First In, First Out) - اللي يدخل الأول يطلع الأول، زي طابور البنك بالظبط.

لكن مش كل الـ Queues كده - الـ PriorityQueue بتطلع العنصر الأهم الأول بغض النظر عن ترتيب الدخول.

---

## مهم جدا: الفرق بين مجموعتين من الـ Methods

الـ Queue عنده **زوجين** من methods لنفس العمليات. الفرق هو إيه اللي بيحصل لما يفشل:

| العملية | بترمي Exception | بترجع قيمة خاصة |
|---|---|---|
| **إضافة** | `add(e)` → IllegalStateException | `offer(e)` → false |
| **شيل من الأول** | `remove()` → NoSuchElementException | `poll()` → null |
| **بص على الأول** | `element()` → NoSuchElementException | `peek()` → null |

::: tip
القاعدة: استخدم **`offer` / `poll` / `peek`** دايما. هم أأمن لإنهم بيرجعوا null أو false بدل ما يرموا exception. استخدم `add` / `remove` / `element` بس لما تكون متأكد إن العملية لازم تنجح.
:::

---

## أنواع الـ Queue

### PriorityQueue

#### إزاي بتشتغل من جوا

الـ PriorityQueue من جواها عبارة عن **Binary Heap** - شجرة ثنائية متخزنة في array.

**خاصية الـ Heap**: كل أب أصغر من أو يساوي ولاده (في الـ Min-Heap).

```
الشجرة:
            10
          /    \
        20      15
       /  \    /
     30   25  18

متخزنة في array:
[10, 20, 15, 30, 25, 18]
 0   1   2   3   4   5

العلاقات:
- أب العنصر i → في index (i-1)/2
- الابن الشمال بتاع i → في index 2*i + 1
- الابن اليمين بتاع i → في index 2*i + 2
```

لما بتضيف عنصر:
1. بيتحط في آخر الـ array
2. بيتقارن بأبوه - لو أصغر بيتبادلوا (**sift up**)
3. بيكرر لحد ما يوصل لمكانه الصح

لما بتشيل عنصر (poll):
1. العنصر الأول (الأصغر) بيطلع
2. آخر عنصر بيتحط مكانه
3. بيتقارن بولاده - لو أكبر من أي واحد فيهم بيتبادل مع الأصغر (**sift down**)
4. بيكرر لحد ما يوصل لمكانه الصح

#### أداء الـ PriorityQueue

| العملية | السرعة | السبب |
|---|---|---|
| `offer(element)` | **O(log n)** | بيعمل sift up في الشجرة |
| `poll()` | **O(log n)** | بيعمل sift down في الشجرة |
| `peek()` | **O(1)** فوري | العنصر الأصغر دايما في الـ root |
| `contains(element)` | **O(n)** بطيء | بيدور في كل الـ array |
| `remove(element)` | **O(n)** بطيء | بيدور الأول وبعدين يعمل sift |

::: warning
الـ PriorityQueue **مش مرتبة بالكامل**! لو عملت for-each عليها، العناصر مش هتطلع مرتبة. بس الـ `poll()` هو اللي بيديك العناصر بالترتيب الصح. ده لإن الـ heap بيضمن إن الأصغر في الأول بس، مش إن كل العناصر مرتبة.
:::

```java
PriorityQueue<Integer> pq = new PriorityQueue<>(List.of(30, 10, 20, 50, 40));

// اللف العادي - مش مرتب!
for (int num : pq) {
    System.out.print(num + " "); // ممكن يطلع: 10 30 20 50 40
}

// لكن poll بترجع بالترتيب
while (!pq.isEmpty()) {
    System.out.print(pq.poll() + " "); // 10 20 30 40 50
}
```

#### استخدامات الـ PriorityQueue

```java
// 1. Min-Heap (الأصغر يطلع الأول) - الـ default
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.addAll(List.of(30, 10, 20, 50, 40));
minHeap.poll(); // 10
minHeap.poll(); // 20

// 2. Max-Heap (الأكبر يطلع الأول)
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.addAll(List.of(30, 10, 20, 50, 40));
maxHeap.poll(); // 50
maxHeap.poll(); // 40

// 3. ترتيب بخاصية معينة
record Task(String name, int priority) {}

PriorityQueue<Task> taskQueue = new PriorityQueue<>(
    Comparator.comparingInt(Task::priority)
);
taskQueue.offer(new Task("Send email", 3));
taskQueue.offer(new Task("Fix critical bug", 1));
taskQueue.offer(new Task("Update docs", 5));
taskQueue.offer(new Task("Deploy to prod", 2));

while (!taskQueue.isEmpty()) {
    Task task = taskQueue.poll();
    System.out.println(task.priority() + ": " + task.name());
}
// 1: Fix critical bug
// 2: Deploy to prod
// 3: Send email
// 5: Update docs
```

---

### ArrayDeque

#### إزاي بتشتغل من جوا

الـ ArrayDeque (اختصار Array Double-Ended Queue) هي **circular array** - array الأول والآخر فيها متوصلين.

```
تخيلها كـ array ملفوفة في دايرة:

          [_, _, "C", "D", "E", _, _, _]
               ^              ^
               |              |
             head           tail

لو ضفنا في الأول:
          [_, "B", "C", "D", "E", _, _, _]
            ^                  ^
            head              tail

لو ضفنا في الآخر:
          [_, "B", "C", "D", "E", "F", _, _]
            ^                       ^
            head                   tail

لو الـ tail وصل لآخر الـ array بيلف من الأول (circular):
          ["G", "B", "C", "D", "E", "F", _, _]
              ^                          ^
              tail                      head
```

الميزة: الإضافة والمسح من الطرفين **O(1)** دايما (إلا لما محتاج resize).

#### أداء الـ ArrayDeque

| العملية | السرعة |
|---|---|
| `offerFirst(e)` / `offerLast(e)` | **O(1)** |
| `pollFirst()` / `pollLast()` | **O(1)** |
| `peekFirst()` / `peekLast()` | **O(1)** |
| `contains(element)` | **O(n)** |
| الوصول بالـ index | **غير متاح** - مفيش `get(i)` |

#### ArrayDeque كـ Stack (LIFO)

```java
// الـ Stack class القديمة في Java مبنية على Vector وهي slow
// استخدم ArrayDeque بدلها

Deque<String> stack = new ArrayDeque<>();

// Push - حط فوق
stack.push("Page 1");
stack.push("Page 2");
stack.push("Page 3");
// الـ Stack: Page 3 (فوق) → Page 2 → Page 1 (تحت)

// Peek - بص على اللي فوق من غير ما تشيله
String top = stack.peek(); // "Page 3"

// Pop - شيل من فوق
String popped = stack.pop(); // "Page 3"
// الـ Stack: Page 2 (فوق) → Page 1 (تحت)
```

#### ArrayDeque كـ Queue (FIFO)

```java
Deque<String> queue = new ArrayDeque<>();

// offer - ادخل من الآخر
queue.offer("Customer 1");
queue.offer("Customer 2");
queue.offer("Customer 3");
// الطابور: Customer 1 (أول واحد) → Customer 2 → Customer 3 (آخر واحد)

// peek - بص على اللي في الأول
String first = queue.peek(); // "Customer 1"

// poll - خدم اللي في الأول
String served = queue.poll(); // "Customer 1"
// الطابور: Customer 2 → Customer 3
```

#### كل methods الـ Deque

```java
Deque<String> deque = new ArrayDeque<>();

// === الإضافة ===
deque.offerFirst("B");  // أضف في الأول → [B]
deque.offerFirst("A");  // أضف في الأول → [A, B]
deque.offerLast("C");   // أضف في الآخر → [A, B, C]
deque.offerLast("D");   // أضف في الآخر → [A, B, C, D]

// === القراءة بدون مسح ===
String first = deque.peekFirst(); // "A"
String last = deque.peekLast();   // "D"

// === المسح ===
String fromFront = deque.pollFirst(); // "A" → [B, C, D]
String fromBack = deque.pollLast();   // "D" → [B, C]

// === Stack methods ===
deque.push("X");     // = offerFirst  → [X, B, C]
deque.pop();         // = pollFirst   → [B, C]
deque.peek();        // = peekFirst   → "B"

// === Queue methods ===
deque.offer("Y");    // = offerLast   → [B, C, Y]
deque.poll();        // = pollFirst   → [C, Y]
deque.peek();        // = peekFirst   → "C"

// === حجم وغيره ===
int size = deque.size();          // 2
boolean empty = deque.isEmpty();  // false
boolean has = deque.contains("C"); // true

// === التحويل ===
deque.clear(); // امسح كل حاجة
```

---

### LinkedList كـ Queue و Deque

الـ LinkedList بتنفذ `List` و `Deque` الاتنين:

```java
// كـ Queue
Queue<String> queue = new LinkedList<>();
queue.offer("A");
queue.offer("B");
queue.poll(); // "A"

// كـ Deque
Deque<String> deque = new LinkedList<>();
deque.offerFirst("B");
deque.offerLast("C");
deque.offerFirst("A");
deque.pollFirst(); // "A"
deque.pollLast();  // "C"
```

::: tip
**ArrayDeque أسرع من LinkedList** كـ Queue و Stack في أغلب الحالات. الـ LinkedList بتستهلك memory أكتر (كل عقدة فيها 2 pointers) والـ cache performance بتاعها أضعف. استخدم ArrayDeque إلا لو محتاج عمليات الـ List (زي الوصول بالـ index).
:::

---

### مقارنة أنواع الـ Queue

```
الخاصية            PriorityQueue   ArrayDeque    LinkedList (as Queue)
──────────────────────────────────────────────────────────────────────
الترتيب            بالأولوية       FIFO/LIFO      FIFO/LIFO
offer/poll         O(log n)        O(1)           O(1)
peek               O(1)            O(1)           O(1)
contains           O(n)            O(n)           O(n)
null values        لأ              لأ             ايوا
Deque support      لأ              ايوا           ايوا
Memory             متوسط           أقل            أكتر
الاستخدام الرئيسي  طابور أولويات   Stack/Queue    Queue + List
```

---

## أمثلة عملية على الـ Queue

### مثال 1: نظام معالجة الطلبات (FIFO Queue)

```java
record Order(int id, String customer, double amount) {}

Queue<Order> orderQueue = new ArrayDeque<>();

// الطلبات بتيجي
orderQueue.offer(new Order(1, "Ahmed", 150.0));
orderQueue.offer(new Order(2, "Sara", 89.50));
orderQueue.offer(new Order(3, "Omar", 220.0));

// المعالجة - اللي طلب الأول يتخدم الأول
while (!orderQueue.isEmpty()) {
    Order order = orderQueue.poll();
    System.out.println("Processing order #" + order.id()
        + " for " + order.customer()
        + " - $" + order.amount());
}
// Processing order #1 for Ahmed - $150.0
// Processing order #2 for Sara - $89.5
// Processing order #3 for Omar - $220.0
```

### مثال 2: Undo/Redo بـ Stack

```java
Deque<String> undoStack = new ArrayDeque<>();
Deque<String> redoStack = new ArrayDeque<>();

// المستخدم بيعمل actions
undoStack.push("Type 'Hello'");
undoStack.push("Type ' World'");
undoStack.push("Make Bold");

// Undo
String lastAction = undoStack.pop(); // "Make Bold"
redoStack.push(lastAction);
System.out.println("Undone: " + lastAction);

// Redo
String redoAction = redoStack.pop(); // "Make Bold"
undoStack.push(redoAction);
System.out.println("Redone: " + redoAction);
```

### مثال 3: إيجاد أكبر K عناصر (Top-K Problem)

```java
// عايز أعرف أعلى 3 درجات من 1000 طالب
int k = 3;
int[] allScores = {72, 95, 88, 41, 63, 99, 77, 84, 91, 55};

// استخدم Min-Heap بحجم k
PriorityQueue<Integer> topK = new PriorityQueue<>();

for (int score : allScores) {
    topK.offer(score);
    if (topK.size() > k) {
        topK.poll(); // شيل الأصغر عشان يفضل أكبر k بس
    }
}

// topK = [91, 95, 99] (أكبر 3 درجات)
while (!topK.isEmpty()) {
    System.out.println(topK.poll());
}
// 91, 95, 99
```

### مثال 4: Sliding Window Maximum

```java
// عندك array وعايز تعرف أكبر عنصر في كل نافذة بحجم k
int[] nums = {1, 3, -1, -3, 5, 3, 6, 7};
int k = 3;

Deque<Integer> deque = new ArrayDeque<>(); // بيخزن indices
List<Integer> result = new ArrayList<>();

for (int i = 0; i < nums.length; i++) {
    // شيل العناصر اللي خرجت من النافذة
    while (!deque.isEmpty() && deque.peekFirst() < i - k + 1) {
        deque.pollFirst();
    }
    // شيل العناصر الأصغر من العنصر الجديد (مش هنحتاجها)
    while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) {
        deque.pollLast();
    }
    deque.offerLast(i);

    // أضف النتيجة لما النافذة تكتمل
    if (i >= k - 1) {
        result.add(nums[deque.peekFirst()]);
    }
}
// result = [3, 3, 5, 5, 6, 7]
// النافذة [1,3,-1]→3  [3,-1,-3]→3  [-1,-3,5]→5  [-3,5,3]→5  [5,3,6]→6  [3,6,7]→7
```

### مثال 5: BFS (Breadth-First Search) باستخدام Queue

```java
// البحث في شجرة مستوى مستوى
record TreeNode(int value, TreeNode left, TreeNode right) {}

void bfs(TreeNode root) {
    if (root == null) return;

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int levelSize = queue.size(); // عدد العناصر في المستوى الحالي

        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            System.out.print(node.value() + " ");

            if (node.left() != null) queue.offer(node.left());
            if (node.right() != null) queue.offer(node.right());
        }
        System.out.println(); // سطر جديد لكل مستوى
    }
}
```

### مثال 6: Merge K Sorted Lists بـ PriorityQueue

```java
// عندك k قوايم مرتبة وعايز تدمجهم في قايمة واحدة مرتبة
List<List<Integer>> lists = List.of(
    List.of(1, 4, 7),
    List.of(2, 5, 8),
    List.of(3, 6, 9)
);

record Entry(int value, int listIndex, int elementIndex) {}

PriorityQueue<Entry> pq = new PriorityQueue<>(
    Comparator.comparingInt(Entry::value)
);

// حط أول عنصر من كل قايمة
for (int i = 0; i < lists.size(); i++) {
    pq.offer(new Entry(lists.get(i).get(0), i, 0));
}

List<Integer> merged = new ArrayList<>();
while (!pq.isEmpty()) {
    Entry smallest = pq.poll();
    merged.add(smallest.value());

    int nextIdx = smallest.elementIndex() + 1;
    if (nextIdx < lists.get(smallest.listIndex()).size()) {
        pq.offer(new Entry(
            lists.get(smallest.listIndex()).get(nextIdx),
            smallest.listIndex(),
            nextIdx
        ));
    }
}
// merged = [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---
---

# الجدول الشامل: امتى تستخدم إيه

## اختيار النوع الرئيسي

```
السؤال                                              الإجابة
──────────────────────────────────────────────────────────────────
محتاج ترتيب + index + تكرار؟                       → List
محتاج عناصر فريدة بدون تكرار؟                      → Set
محتاج معالجة عناصر بترتيب معين (FIFO/أولويات)؟     → Queue
محتاج إضافة/مسح من الطرفين؟                        → Deque
```

## اختيار التنفيذ

```
                  الحالة                                    استخدم
────────────────────────────────────────────────────────────────────────────
List + قراءة كتير بالـ index                           → ArrayList
List + إضافة/مسح من الأول كتير                         → LinkedList (نادر)
List + أي حاجة تانية                                   → ArrayList

Set + مش مهم الترتيب                                   → HashSet
Set + عايز ترتيب الإضافة                               → LinkedHashSet
Set + عايز العناصر مرتبة + عمليات زي floor/ceiling     → TreeSet

Queue + FIFO عادي                                      → ArrayDeque
Queue + أولويات (الأهم يطلع الأول)                     → PriorityQueue
Stack + LIFO                                           → ArrayDeque
Deque + الطرفين                                        → ArrayDeque
```

## جدول الأداء النهائي

```
العملية         ArrayList  LinkedList  HashSet  LinkedHashSet  TreeSet  PriorityQueue  ArrayDeque
────────────────────────────────────────────────────────────────────────────────────────────────────
get(index)      O(1)       O(n)        -        -              -        -              -
add (آخر)       O(1)*      O(1)        -        -              -        -              O(1)
add (أول)       O(n)       O(1)        -        -              -        -              O(1)
add/offer       -          -           O(1)     O(1)           O(log n) O(log n)       O(1)
remove (قيمة)   O(n)       O(n)        O(1)     O(1)           O(log n) O(n)           O(n)
contains        O(n)       O(n)        O(1)     O(1)           O(log n) O(n)           O(n)
poll            -          O(1)        -        -              -        O(log n)       O(1)
peek            -          O(1)        -        -              -        O(1)           O(1)

* amortized - أحيانا O(n) لما بيعمل resize
```
