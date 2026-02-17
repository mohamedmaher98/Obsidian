# Map في Java - الدليل الشامل من الصفر

---

## القصة من الأول: ليه محتاجين Map؟

تخيل إنك فاتح دفتر تليفونات. كل صفحة فيها **اسم** وجنبه **رقم التليفون**. لما تدور على "Ahmed"، بتفتح الصفحة بتاعته وتلاقي الرقم على طول.

ده بالظبط الـ Map: **دفتر** بيربط بين حاجتين:
- **Key** (المفتاح) → الاسم
- **Value** (القيمة) → رقم التليفون

```java
Map<String, String> phoneBook = new HashMap<>();
phoneBook.put("Ahmed", "0123456789");
phoneBook.put("Sara",  "0111222333");

String ahmedPhone = phoneBook.get("Ahmed"); // "0123456789"
```

بس الحكاية مش بالبساطة دي. ورا الكواليس في سحر هندسي كبير بيخلي البحث عن أي key من بين مليون key يحصل في **لحظة واحدة**. تعالى نفهم إزاي.

---
---

# الفصل الأول: HashMap - البطل الرئيسي

الـ HashMap هو أكتر نوع Map بيتستخدم في Java. في 90% من الحالات هو اللي هتستخدمه. فخلينا نفهمه من جواه خالص.

## القصة: إزاي HashMap بتشتغل من جوا

### المشكلة اللي بتحلها

تخيل عندك 1,000,000 اسم وعايز تدور على اسم واحد. لو حطيتهم في List، هتدور واحد واحد - ممكن تاخد مليون خطوة. ده **O(n)** وده بطيء جدا.

الـ HashMap بتحل المشكلة دي بحيلة ذكية: بدل ما تدور على العنصر، **بتحسب مكانه**.

### الخطوة 1: الـ Hash Function - السحر

كل object في Java عنده method اسمها `hashCode()` بترجع رقم integer. الرقم ده زي **البصمة** بتاعة الـ object:

```java
"Ahmed".hashCode()  → 63347874
"Sara".hashCode()   → 2554538
"Omar".hashCode()   → 2466830
```

الرقم ده ثابت - كل ما تحسب hashCode لـ "Ahmed" هيطلع نفس الرقم.

### الخطوة 2: الـ Array of Buckets

الـ HashMap من جواها عبارة عن **array** (اسمها table)، حجمها الأولي **16 خانة**. كل خانة اسمها **bucket** (زنبيل):

```
Index:  [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]  [8]  [9]  [10] [11] [12] [13] [14] [15]
         |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
        null null null null null null null null null null null null null null null null
```

### الخطوة 3: الربط - من hashCode للـ bucket

لما بتعمل `put("Ahmed", "0123456789")`:

```
1. احسب الـ hash:     "Ahmed".hashCode() → 63347874
2. حدد الـ bucket:    63347874 % 16 = 2     (باقي القسمة)
3. حط في bucket رقم 2: ["Ahmed" → "0123456789"]
```

```
Index:  [0]  [1]  [2]               [3]  [4] ...
         |    |    |                  |    |
        null null  ["Ahmed"→"012.."] null null
```

لما بتعمل `get("Ahmed")`:

```
1. احسب الـ hash:     "Ahmed".hashCode() → 63347874
2. حدد الـ bucket:    63347874 % 16 = 2
3. روح لـ bucket 2:  لقيت "Ahmed" → ارجع "0123456789"
```

**مروحتش تدور في كل الـ buckets!** رحت للمكان الصح مباشرة. ده السبب إن HashMap سريعة **O(1)**.

### الخطوة 4: التصادم (Collision) - لما اتنين يروحوا نفس الـ bucket

تخيل "Ahmed" و "Ziad" لما نعمل % 16 طلعوا نفس الـ bucket. إيه اللي بيحصل؟

**في Java 8 وبعدها، الحل بيكون على مرحلتين:**

**المرحلة الأولى - Linked List:**
لو عدد العناصر في الـ bucket **أقل من 8**، بيتعملوا linked list:

```
Bucket 2: ["Ahmed"→"012.."] → ["Ziad"→"099.."] → null
```

لما تدور على "Ziad":
1. روح لـ bucket 2
2. قارن "Ahmed".equals("Ziad")? لأ → كمل
3. قارن "Ziad".equals("Ziad")? ايوا → لقيته!

**المرحلة التانية - Red-Black Tree:**
لو عدد العناصر في bucket واحد وصل **8 أو أكتر**، الـ linked list بتتحول لـ **Red-Black Tree**. ليه؟ لإن البحث في linked list هو O(n)، لكن في tree هو O(log n). يعني لو 1000 عنصر في نفس الـ bucket:
- Linked List: 1000 مقارنة
- Tree: ~10 مقارنات بس

```
Bucket 2 (tree mode):
              "Mona"
             /      \
          "Ahmed"   "Ziad"
           /  \
        "Ali" "Khaled"
```

### الخطوة 5: الـ Resize - لما البيت يضيق

الـ HashMap عندها حاجة اسمها **load factor** (عامل التحميل) = **0.75** بشكل افتراضي.

يعني: لما عدد العناصر يوصل **75%** من حجم الـ array، بيتعمل **resize**:

```
الحالة الأولية: 16 bucket، load factor 0.75
→ لما العناصر توصل 16 × 0.75 = 12 عنصر → resize!

الـ resize:
1. عمل array جديدة حجمها الضعف: 32 bucket
2. خد كل عنصر واحسب مكانه الجديد (لإن % 32 غير % 16)
3. حط كل عنصر في مكانه الجديد

بعدين لما توصل 32 × 0.75 = 24 → resize تاني! → 64 bucket
وهكذا...
```

الـ resize مكلفة (O(n) لإنها بتنقل كل العناصر)، بس بتحصل نادر. وعشان كده لو عارف إنك هتحط 10,000 عنصر، الأحسن إنك تقول من الأول:

```java
// بدل ما يعمل resize كذا مرة
Map<String, Integer> map = new HashMap<>(14000);
// 14000 / 0.75 ≈ 10666 capacity → مفيش resize
```

### ملخص HashMap من جوا

```
put("Ahmed", value):
  ┌─────────────────────────────────────────────┐
  │ 1. hashCode("Ahmed") → 63347874             │
  │ 2. hash = spread(63347874) → تعديل بسيط     │
  │ 3. index = hash & (capacity - 1) → 2        │
  │ 4. روح لـ bucket[2]                         │
  │    ├─ لو فاضي → حط Node جديد               │
  │    ├─ لو في nodes ودي key مكرر → غير الvalue │
  │    └─ لو في nodes ودي key جديد → أضف        │
  │        ├─ < 8 nodes → Linked List            │
  │        └─ >= 8 nodes → Red-Black Tree        │
  │ 5. لو size > threshold → resize (ضعّف)      │
  └─────────────────────────────────────────────┘
```

---

## كل Methods الـ HashMap بالتفصيل

### الإضافة والتعديل

```java
Map<String, Integer> scores = new HashMap<>();

// 1. put - حط قيمة (لو الـ key موجود → غير القيمة القديمة وارجعها)
scores.put("Ahmed", 85);      // null (مكانش موجود)
scores.put("Sara", 92);       // null
Integer old = scores.put("Ahmed", 90); // 85 (القيمة القديمة)
// scores = {Ahmed=90, Sara=92}

// 2. putIfAbsent - حط بس لو الـ key مش موجود (مش هيغير الموجود)
scores.putIfAbsent("Ahmed", 100); // 90 → مغيرش حاجة لإن Ahmed موجود
scores.putIfAbsent("Omar", 78);   // null → اتضاف لإنه مكانش موجود
// scores = {Ahmed=90, Sara=92, Omar=78}

// 3. putAll - حط كل عناصر map تانية
Map<String, Integer> more = Map.of("Ali", 88, "Mona", 95);
scores.putAll(more);
// scores = {Ahmed=90, Sara=92, Omar=78, Ali=88, Mona=95}
```

### القراءة والبحث

```java
Map<String, Integer> scores = new HashMap<>(Map.of(
    "Ahmed", 90, "Sara", 92, "Omar", 78
));

// 1. get - هات القيمة (null لو الـ key مش موجود)
Integer ahmedScore = scores.get("Ahmed");  // 90
Integer unknownScore = scores.get("Ziad"); // null

// 2. getOrDefault - هات القيمة أو قيمة بديلة
int score1 = scores.getOrDefault("Ahmed", 0); // 90
int score2 = scores.getOrDefault("Ziad", 0);  // 0 (مش موجود → رجع الـ default)

// 3. containsKey - الـ key موجود؟
boolean hasAhmed = scores.containsKey("Ahmed"); // true
boolean hasZiad = scores.containsKey("Ziad");   // false

// 4. containsValue - القيمة موجودة؟ (بطيء O(n) - بيدور في كل القيم)
boolean has90 = scores.containsValue(90);  // true
boolean has50 = scores.containsValue(50);  // false

// 5. size و isEmpty
int count = scores.size();      // 3
boolean empty = scores.isEmpty(); // false

// 6. keySet - كل الـ keys كـ Set
Set<String> names = scores.keySet(); // [Ahmed, Sara, Omar] (مش بترتيب معين)

// 7. values - كل القيم كـ Collection
Collection<Integer> allScores = scores.values(); // [90, 92, 78]

// 8. entrySet - كل الـ key-value pairs كـ Set
Set<Map.Entry<String, Integer>> entries = scores.entrySet();
// [Ahmed=90, Sara=92, Omar=78]
```

::: warning
ال`keySet()`, `values()`, و `entrySet()` بيرجعوا **views** مش نسخ. يعني لو عدلت الـ Map، الـ views هتتأثر، والعكس صحيح.
:::

### المسح

```java
Map<String, Integer> scores = new HashMap<>(Map.ofEntries(
    Map.entry("Ahmed", 90), Map.entry("Sara", 92),
    Map.entry("Omar", 78), Map.entry("Ali", 88)
));

// 1. remove بالـ key - امسح وارجع القيمة القديمة
Integer removed = scores.remove("Omar"); // 78
Integer removed2 = scores.remove("Ziad"); // null (مكانش موجود)

// 2. remove بالـ key والـ value - امسح بس لو القيمة مطابقة
boolean ok1 = scores.remove("Ahmed", 90);  // true → اتمسح (key=Ahmed, value=90 ✓)
boolean ok2 = scores.remove("Sara", 50);   // false → ما اتمسحش (value مش 50)
// scores = {Sara=92, Ali=88}

// 3. clear - امسح كل حاجة
scores.clear(); // {}
```

### التعديل المتقدم (Java 8+)

دول أقوى methods في الـ Map وبيوفروا عليك كود كتير:

#### replace

```java
Map<String, Integer> scores = new HashMap<>(Map.of("Ahmed", 90, "Sara", 92));

// 1. replace - غير القيمة لو الـ key موجود
scores.replace("Ahmed", 95);  // Ahmed → 95 (اتغير)
scores.replace("Ziad", 80);   // مفيش تأثير (Ziad مش موجود)

// 2. replace بالقيمة القديمة - غير بس لو القيمة الحالية مطابقة
boolean changed = scores.replace("Sara", 92, 98); // true → 92 اتغيرت لـ 98
boolean changed2 = scores.replace("Sara", 50, 99); // false → القيمة مش 50 فمتغيرتش

// 3. replaceAll - غير كل القيم بـ function
scores.replaceAll((name, score) -> score + 5); // كل واحد زاد 5 درجات
```

#### compute - احسب قيمة جديدة

```java
Map<String, Integer> wordCount = new HashMap<>();
String text = "hello world hello java hello world";

// بدون compute:
for (String word : text.split(" ")) {
    if (wordCount.containsKey(word)) {
        wordCount.put(word, wordCount.get(word) + 1);
    } else {
        wordCount.put(word, 1);
    }
}

// مع compute:
wordCount.clear();
for (String word : text.split(" ")) {
    wordCount.compute(word, (key, oldValue) -> oldValue == null ? 1 : oldValue + 1);
}
// {hello=3, world=2, java=1}
```

#### computeIfAbsent - احسب بس لو مش موجود

ده من **أهم** الـ methods. تخيل إنك عايز تجمع أسماء حسب أول حرف:

```java
// بدون computeIfAbsent (كود طويل ومزعج):
Map<Character, List<String>> groups = new HashMap<>();
for (String name : List.of("Ahmed", "Ali", "Sara", "Amr", "Salma")) {
    char firstChar = name.charAt(0);
    if (!groups.containsKey(firstChar)) {
        groups.put(firstChar, new ArrayList<>());
    }
    groups.get(firstChar).add(name);
}

// مع computeIfAbsent (سطر واحد!):
Map<Character, List<String>> groups2 = new HashMap<>();
for (String name : List.of("Ahmed", "Ali", "Sara", "Amr", "Salma")) {
    groups2.computeIfAbsent(name.charAt(0), k -> new ArrayList<>()).add(name);
}
// {A=[Ahmed, Ali, Amr], S=[Sara, Salma]}
```

إزاي بتشتغل خطوة بخطوة:
```
computeIfAbsent('A', k -> new ArrayList<>())

1. هل 'A' موجود في الـ Map؟
   ├─ لأ → نفذ الـ function: new ArrayList<>()
   │       حط النتيجة في الـ Map: 'A' → []
   │       ارجع الـ ArrayList الجديد
   └─ ايوا → ارجع القيمة الموجودة على طول (مش هينفذ الـ function)

2. .add(name) → أضف الاسم للـ ArrayList (سواء الجديد أو الموجود)
```

#### computeIfPresent - احسب بس لو موجود

```java
Map<String, Integer> scores = new HashMap<>(Map.of("Ahmed", 90, "Sara", 85));

// ضاعف الدرجة بس للموجودين
scores.computeIfPresent("Ahmed", (key, val) -> val * 2); // Ahmed → 180
scores.computeIfPresent("Ziad", (key, val) -> val * 2);  // مفيش تأثير (Ziad مش موجود)

// لو الـ function رجعت null → الـ key بيتمسح!
scores.computeIfPresent("Sara", (key, val) -> null); // Sara اتمسحت!
```

#### merge - ادمج قيمة جديدة مع القديمة

```java
Map<String, Integer> scores = new HashMap<>();

// عد الكلمات بـ merge (أنظف من compute):
for (String word : "hello world hello java hello world".split(" ")) {
    scores.merge(word, 1, Integer::sum);
    // لو مش موجود → حط 1
    // لو موجود → اجمع القيمة القديمة + 1
}
// {hello=3, world=2, java=1}

// مثال تاني: اجمع الأرقام
Map<String, Integer> totals = new HashMap<>(Map.of("Ahmed", 100));
totals.merge("Ahmed", 50, Integer::sum);  // Ahmed → 150
totals.merge("Sara", 80, Integer::sum);   // Sara → 80 (مكانتش موجودة)

// لو الـ function رجعت null → الـ key بيتمسح
totals.merge("Ahmed", 0, (oldVal, newVal) -> null); // Ahmed اتمسح
```

### اللف على الـ Map

```java
Map<String, Integer> scores = Map.of("Ahmed", 90, "Sara", 92, "Omar", 78);

// 1. for-each على الـ entrySet (الأكثر استخداما)
for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    String name = entry.getKey();
    int score = entry.getValue();
    System.out.println(name + " = " + score);
}

// 2. forEach مع lambda (أنظف)
scores.forEach((name, score) -> {
    System.out.println(name + " = " + score);
});

// 3. اللف على الـ keys بس
for (String name : scores.keySet()) {
    System.out.println(name);
}

// 4. اللف على الـ values بس
for (int score : scores.values()) {
    System.out.println(score);
}

// 5. بـ Stream
scores.entrySet().stream()
    .filter(e -> e.getValue() > 80)
    .forEach(e -> System.out.println(e.getKey() + " passed!"));
```

---

## أداء HashMap

| العملية | السرعة (Average) | السرعة (Worst) | السبب |
|---|---|---|---|
| `put(key, value)` | **O(1)** | O(n)* | hash → bucket → حط |
| `get(key)` | **O(1)** | O(n)* | hash → bucket → جيب |
| `remove(key)` | **O(1)** | O(n)* | hash → bucket → امسح |
| `containsKey(key)` | **O(1)** | O(n)* | hash → bucket → دور |
| `containsValue(value)` | **O(n)** دايما | O(n) | بيلف على كل القيم |

\* الـ worst case بيحصل لما كل العناصر تروح نفس الـ bucket (hash collision سيئ). في Java 8+ مع الـ tree bins بيبقى O(log n) بدل O(n).

---
---

# الفصل التاني: LinkedHashMap - HashMap بذاكرة

## القصة

الـ HashMap مش بتحافظ على أي ترتيب. يعني لو حطيت Ahmed أول واحد، مش شرط يطلع أول واحد لما تلف على الـ Map.

**بس أحيانا الترتيب مهم.** تخيل إنك بتعمل cache: عايز تفتكر ترتيب الحاجات اللي دخلت، أو ترتيب الحاجات اللي اتوصلت ليها. هنا بييجي دور **LinkedHashMap**.

## إزاي بتشتغل من جوا

الـ LinkedHashMap هي **HashMap + Doubly Linked List**. كل entry جواها بيبقى عنده:
- مكانه في الـ hash table (زي HashMap)
- pointer للـ entry اللي اتضاف **قبله**
- pointer للـ entry اللي اتضاف **بعده**

```
Hash Table (زي HashMap):
Bucket 0: [Sara → 92]
Bucket 3: [Ahmed → 90]
Bucket 7: [Omar → 78]

Linked List (بترتيب الإضافة):
head → [Ahmed → 90] ⇄ [Sara → 92] ⇄ [Omar → 78] ← tail
       أول واحد اتضاف     تاني           تالت
```

```java
Map<String, Integer> map = new LinkedHashMap<>();
map.put("Ahmed", 90);
map.put("Sara", 92);
map.put("Omar", 78);

map.forEach((k, v) -> System.out.println(k));
// Ahmed ← أول واحد اتضاف
// Sara
// Omar  ← آخر واحد اتضاف
// الترتيب مضمون!
```

## الـ Access Order Mode - ترتيب الوصول

الـ LinkedHashMap عندها وضع تاني: بدل ترتيب الإضافة، ترتب بـ **آخر وصول**. كل ما توصل لعنصر (بـ get أو put)، بيروح لآخر الـ linked list:

```java
// الـ true في الآخر = access order mode
Map<String, Integer> map = new LinkedHashMap<>(16, 0.75f, true);
map.put("Ahmed", 90); // الترتيب: Ahmed
map.put("Sara", 92);  // الترتيب: Ahmed → Sara
map.put("Omar", 78);  // الترتيب: Ahmed → Sara → Omar

map.get("Ahmed");      // Ahmed اتوصله → راح لآخر: Sara → Omar → Ahmed
map.get("Sara");       // Sara اتوصلها → راحت لآخر: Omar → Ahmed → Sara

map.forEach((k, v) -> System.out.print(k + " "));
// Omar Ahmed Sara
```

## LRU Cache - التطبيق الأشهر

الـ access order mode بيخلي أقدم عنصر ماتوصلوش يفضل في الأول. ده بالظبط اللي محتاجه لـ **LRU Cache** (Least Recently Used):

```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int maxSize;

    public LRUCache(int maxSize) {
        super(maxSize, 0.75f, true); // true = access order
        this.maxSize = maxSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > maxSize; // لو العدد أكبر من الحد → امسح الأقدم
    }
}
```

```java
LRUCache<String, String> cache = new LRUCache<>(3);

cache.put("page1", "content1");
cache.put("page2", "content2");
cache.put("page3", "content3");
// Cache: [page1, page2, page3]

cache.get("page1"); // page1 اتوصله → راح لآخر
// Cache: [page2, page3, page1]

cache.put("page4", "content4"); // Cache مليان → page2 (الأقدم) اتمسح
// Cache: [page3, page1, page4]

System.out.println(cache.containsKey("page2")); // false → اتمسح!
```

## الأداء

زي الـ HashMap بالظبط في كل العمليات، بس مع overhead بسيط جدا عشان الـ linked list. الـ iteration أسرع من HashMap لإنها بتمشي على الـ linked list مباشرة بدل ما تعدي على buckets فاضية.

---
---

# الفصل التالت: TreeMap - الخريطة المرتبة

## القصة

تخيل إنك عايز تعمل قاموس. مش بس عايز تلاقي كلمة بسرعة - لأ، عايز كمان الكلمات تكون **مرتبة أبجديا**. وعايز تقدر تقول "هاتلي كل الكلمات من 'أ' لـ 'ج'".

الـ HashMap مش هتنفع هنا لإنها مش مرتبة. الـ TreeMap بتحل المشكلة دي.

## إزاي بتشتغل من جوا

الـ TreeMap من جواها عبارة عن **Red-Black Tree** (شجرة ثنائية متوازنة). كل node فيها key-value pair، والشجرة مرتبة بالـ key.

```
              ["Mona" → 95]
              /             \
     ["Ahmed" → 90]    ["Sara" → 92]
       /      \              \
["Ali" → 88] ["Khaled" → 85] ["Ziad" → 77]
```

**خصائص الشجرة:**
- كل node على الشمال أصغر من الأب
- كل node على اليمين أكبر من الأب
- الشجرة **متوازنة** دايما (فرق الارتفاع بين أي فرعين <= 1)

لما بتضيف عنصر:
1. بيقارن الـ key بالـ root
2. لو أصغر → يروح شمال، لو أكبر → يمين
3. بيكرر لحد ما يلاقي مكانه
4. بيتحط ويتعمل **rebalancing** (تلوين وتدوير) عشان الشجرة تفضل متوازنة

## الأداء

| العملية | السرعة | المقارنة بـ HashMap |
|---|---|---|
| `put(key, value)` | **O(log n)** | أبطأ (HashMap = O(1)) |
| `get(key)` | **O(log n)** | أبطأ |
| `remove(key)` | **O(log n)** | أبطأ |
| `containsKey(key)` | **O(log n)** | أبطأ |
| الـ Iteration | **O(n)** مرتب | HashMap مش مرتبة |

O(log n) يعني: مليون عنصر → ~20 مقارنة بس. مش سيئة خالص، بس HashMap أسرع.

## العمليات الخاصة - القوة الحقيقية

الـ TreeMap بتنفذ `NavigableMap` وده بيديها عمليات مش موجودة في أي Map تاني:

```java
TreeMap<Integer, String> students = new TreeMap<>();
students.put(55, "Ali");
students.put(63, "Omar");
students.put(72, "Mona");
students.put(78, "Khaled");
students.put(85, "Sara");
students.put(91, "Ahmed");
students.put(95, "Hana");
// مرتبة تلقائيا: {55=Ali, 63=Omar, 72=Mona, 78=Khaled, 85=Sara, 91=Ahmed, 95=Hana}
```

### أول وآخر عنصر

```java
Map.Entry<Integer, String> first = students.firstEntry(); // 55=Ali
Map.Entry<Integer, String> last = students.lastEntry();   // 95=Hana

Integer firstKey = students.firstKey(); // 55
Integer lastKey = students.lastKey();   // 95
```

### البحث بالقرب - أقرب key لقيمة معينة

```java
// lowerEntry - أكبر entry أصغر من 78 (strictly less)
Map.Entry<Integer, String> below = students.lowerEntry(78); // 72=Mona

// floorEntry - أكبر entry أصغر من أو تساوي 78
Map.Entry<Integer, String> atOrBelow = students.floorEntry(78); // 78=Khaled

// higherEntry - أصغر entry أكبر من 78 (strictly greater)
Map.Entry<Integer, String> above = students.higherEntry(78); // 85=Sara

// ceilingEntry - أصغر entry أكبر من أو تساوي 78
Map.Entry<Integer, String> atOrAbove = students.ceilingEntry(78); // 78=Khaled

// لو بتدور على key مش موجود:
Map.Entry<Integer, String> ceiling80 = students.ceilingEntry(80); // 85=Sara
// مفيش 80 في الشجرة → أقرب حاجة أكبر = 85

// في نسخ بترجع الـ key بس:
Integer lowerKey = students.lowerKey(78);   // 72
Integer higherKey = students.higherKey(78); // 85
```

### التقطيع - خد جزء من الـ Map

```java
// headMap - كل الـ keys الأصغر من قيمة معينة
SortedMap<Integer, String> failing = students.headMap(60);
// {55=Ali}

// headMap inclusive
NavigableMap<Integer, String> upTo60 = students.headMap(60, true);
// {55=Ali} (60 مش موجود أصلا لكن لو كان موجود كان هيتضاف)

// tailMap - كل الـ keys الأكبر من أو تساوي قيمة معينة
SortedMap<Integer, String> excellent = students.tailMap(90);
// {91=Ahmed, 95=Hana}

// subMap - بين قيمتين
SortedMap<Integer, String> mid = students.subMap(70, 90);
// {72=Mona, 78=Khaled, 85=Sara}   ← 70 inclusive, 90 exclusive

NavigableMap<Integer, String> mid2 = students.subMap(70, true, 90, true);
// {72=Mona, 78=Khaled, 85=Sara}   ← 90 مش موجود فمش هيتضاف
```

::: tip
الـ sub-maps دي **views** - يعني لو عدلت الأصلية أو الـ view، التاني هيتأثر. ولو حاولت تضيف key خارج الـ range في الـ view، هتاخد `IllegalArgumentException`.
:::

### ترتيب عكسي ومسح مع إرجاع

```java
// descendingMap - نسخة بترتيب عكسي
NavigableMap<Integer, String> desc = students.descendingMap();
// {95=Hana, 91=Ahmed, 85=Sara, 78=Khaled, 72=Mona, 63=Omar, 55=Ali}

// pollFirstEntry - شيل وارجع أصغر entry
Map.Entry<Integer, String> lowest = students.pollFirstEntry(); // 55=Ali
// students الآن بدون Ali

// pollLastEntry - شيل وارجع أكبر entry
Map.Entry<Integer, String> highest = students.pollLastEntry(); // 95=Hana
```

### ترتيب مخصص

```java
// ترتيب عكسي
TreeMap<String, Integer> reversed = new TreeMap<>(Comparator.reverseOrder());
reversed.put("Ahmed", 90);
reversed.put("Sara", 92);
reversed.put("Omar", 78);
// {Sara=92, Omar=78, Ahmed=90}

// ترتيب case-insensitive
TreeMap<String, Integer> caseInsensitive = new TreeMap<>(String.CASE_INSENSITIVE_ORDER);
caseInsensitive.put("Ahmed", 90);
caseInsensitive.put("ahmed", 95); // بيعتبره نفس الـ key!
// {Ahmed=95}
```

---
---

# الفصل الرابع: أنواع Map تانية مهمة

## Hashtable - الجد القديم

الـ Hashtable هي النسخة القديمة من HashMap من أيام Java 1.0. **متستخدمهاش في كود جديد.**

```java
// قديم - متستخدموش
Hashtable<String, Integer> old = new Hashtable<>();

// الفرق عن HashMap:
// 1. Hashtable = synchronized (thread-safe بس بطيء)
// 2. Hashtable مش بيقبل null keys أو null values
// 3. HashMap أسرع بكتير
```

## EnumMap - خريطة للـ Enums

لما الـ keys بتوعك كلها من نوع Enum، الـ EnumMap أسرع وأوفر في الـ memory من HashMap بمراحل.

### إزاي بتشتغل من جوا

الـ EnumMap من جواها عبارة عن **array بسيطة**. كل enum value ليها index ثابت (ordinal)، فبيستخدمه مباشرة كـ index في الـ array:

```
enum Day { SAT, SUN, MON, TUE, WED, THU, FRI }
           0     1    2    3    4    5    6

Array من جوا:
[null, null, "Work", "Work", "Work", "Work", null]
 SAT   SUN    MON    TUE     WED     THU    FRI
```

مفيش hashing ولا buckets ولا حاجة - **وصول مباشر بالـ index**. عشان كده هي الأسرع.

```java
enum Priority { LOW, MEDIUM, HIGH, CRITICAL }

Map<Priority, List<String>> tasksByPriority = new EnumMap<>(Priority.class);
tasksByPriority.put(Priority.CRITICAL, List.of("Fix server crash"));
tasksByPriority.put(Priority.HIGH, List.of("Deploy hotfix", "Update API"));
tasksByPriority.put(Priority.LOW, List.of("Update docs"));

// اللف بيكون بترتيب الـ enum (مش بترتيب الإضافة ومش random)
tasksByPriority.forEach((priority, tasks) -> {
    System.out.println(priority + ": " + tasks);
});
// LOW: [Update docs]
// HIGH: [Deploy hotfix, Update API]
// CRITICAL: [Fix server crash]
```

## WeakHashMap - الخريطة اللي بتنسى

### القصة

تخيل إنك بتعمل cache: بتحفظ نتائج عمليات مكلفة عشان لو حد طلبها تاني ترجعها بسرعة. بس المشكلة: الـ cache ممكن يكبر ويكبر لحد ما يملي الـ memory.

الـ WeakHashMap بتحل المشكلة دي: لما مفيش حد تاني بيستخدم الـ key (مفيش reference ليه)، الـ Garbage Collector بيمسحه من الـ Map تلقائيا.

```java
WeakHashMap<Object, String> cache = new WeakHashMap<>();

Object key1 = new Object();
Object key2 = new Object();
cache.put(key1, "Data for key1");
cache.put(key2, "Data for key2");
// cache.size() = 2

key1 = null; // مفيش حد بيستخدم key1 خلاص
System.gc(); // الـ GC ممكن يمسح key1 من الـ cache

// cache.size() ممكن يبقى 1 (key1 اتمسح)
```

::: warning
`String` literals مش بيتمسحوا لإنهم interned. يعني `cache.put("hello", value)` مش هيتمسح أبدا. استخدم `new String("hello")` لو عايز السلوك ده (بس نادر ما تحتاجه).
:::

## IdentityHashMap - المقارنة بالـ Reference مش بالـ equals

الـ HashMap العادية بتقارن الـ keys بـ `equals()`. يعني `new String("hello")` و `new String("hello")` يعتبروا نفس الـ key.

الـ IdentityHashMap بتقارن بالـ `==` (نفس الـ object في الـ memory بالظبط):

```java
Map<String, Integer> normal = new HashMap<>();
String a = new String("hello");
String b = new String("hello");
normal.put(a, 1);
normal.put(b, 2);
// normal.size() = 1 → a.equals(b) = true فاعتبرهم نفس الـ key

Map<String, Integer> identity = new IdentityHashMap<>();
identity.put(a, 1);
identity.put(b, 2);
// identity.size() = 2 → a != b (references مختلفة) فاعتبرهم keys مختلفة
```

**امتى تستخدمها؟** نادر جدا. مثلا لما تعمل object graph traversal وعايز تتبع الـ objects اللي زرتها بالـ reference مش بالـ value.

---
---

# الفصل الخامس: Immutable Maps - الخرائط الثابتة

## Map.of و Map.ofEntries (Java 9+)

```java
// عمل Map ثابتة (Immutable) - مينفعش تتعدل
// حد أقصى 10 entries مع Map.of
Map<String, Integer> scores = Map.of(
    "Ahmed", 90,
    "Sara", 92,
    "Omar", 78
);

// لأكتر من 10 entries استخدم Map.ofEntries
Map<String, Integer> moreScores = Map.ofEntries(
    Map.entry("Ahmed", 90),
    Map.entry("Sara", 92),
    Map.entry("Omar", 78),
    Map.entry("Ali", 88),
    Map.entry("Mona", 95),
    Map.entry("Khaled", 72),
    Map.entry("Hana", 91),
    Map.entry("Amr", 84),
    Map.entry("Nour", 87),
    Map.entry("Youssef", 93),
    Map.entry("Layla", 89)
);

// أي محاولة تعديل → UnsupportedOperationException
// scores.put("New", 100);    // Exception!
// scores.remove("Ahmed");    // Exception!

// Map.of مش بتقبل null keys أو values
// Map.of("key", null);       // NullPointerException!

// لو عايز نسخة تتعدل
Map<String, Integer> mutable = new HashMap<>(scores);
mutable.put("New", 100); // شغال
```

## Map.copyOf (Java 10+)

```java
Map<String, Integer> original = new HashMap<>();
original.put("Ahmed", 90);
original.put("Sara", 92);

Map<String, Integer> copy = Map.copyOf(original);
// copy ثابتة - مينفعش تتعدل
// لو original اتعدلت بعد كده، copy مش هتتأثر
```

## Collections.unmodifiableMap

```java
Map<String, Integer> original = new HashMap<>();
original.put("Ahmed", 90);

Map<String, Integer> view = Collections.unmodifiableMap(original);
// view.put("X", 1);  // Exception!

// بس لو عدلت original:
original.put("Sara", 92);
// view هتتأثر! لإنها view مش نسخة مستقلة
System.out.println(view.size()); // 2
```

---
---

# الفصل السادس: Concurrent Maps - خرائط الـ Threads

## المشكلة

لما أكتر من thread بيعدلوا HashMap في نفس الوقت، بتحصل كوارث:
- بيانات بتضيع
- infinite loop (الـ linked list بتعمل دايرة)
- `ConcurrentModificationException`

## ConcurrentHashMap - الحل

### إزاي بتشتغل من جوا

في Java 8+، الـ ConcurrentHashMap بتستخدم تقنية اسمها **CAS (Compare-And-Swap)** مع **synchronized** على مستوى الـ bucket الواحد فقط:

```
Thread 1 بيعدل Bucket 3 ──→ synchronized على Bucket 3 بس
Thread 2 بيعدل Bucket 7 ──→ synchronized على Bucket 7 بس
↑ الاتنين بيشتغلوا في نفس الوقت! مفيش blocking

Thread 3 بيعدل Bucket 3 ──→ مستني Thread 1 يخلص
↑ بس ده بيحصل نادر
```

الفرق عن Hashtable:
- **Hashtable**: بيعمل lock على الـ Map **كلها** → thread واحد بس يشتغل
- **ConcurrentHashMap**: بيعمل lock على **bucket واحد** → threads كتير تشتغل معا

### الاستخدام

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// العمليات العادية - thread-safe
map.put("Ahmed", 90);
map.get("Ahmed");
map.remove("Ahmed");

// العمليات الذرية (atomic) - دي المهمة
// لإن get ثم put في خطوتين مش thread-safe حتى مع ConcurrentHashMap
```

### العمليات الذرية - Atomic Operations

```java
ConcurrentHashMap<String, Integer> counter = new ConcurrentHashMap<>();

// مشكلة: ده مش thread-safe حتى مع ConcurrentHashMap!
// Thread 1 و Thread 2 ممكن يقرأوا نفس القيمة قبل ما أي واحد فيهم يكتب
Integer val = counter.get("visits");
counter.put("visits", val == null ? 1 : val + 1);

// الحل: استخدم العمليات الذرية
counter.merge("visits", 1, Integer::sum);
// ده ذري (atomic) - مفيش race condition

// أو
counter.compute("visits", (key, oldVal) -> oldVal == null ? 1 : oldVal + 1);

// أو
counter.putIfAbsent("visits", 0);
// بس putIfAbsent ثم get ثم put مش atomic ككل
```

### عمليات خاصة بـ ConcurrentHashMap

```java
ConcurrentHashMap<String, Integer> scores = new ConcurrentHashMap<>();
scores.put("Ahmed", 90);
scores.put("Sara", 92);
scores.put("Omar", 78);
scores.put("Ali", 65);
scores.put("Mona", 88);

// 1. Parallel forEach - بيشتغل على threads متعددة
scores.forEach(2, (name, score) -> {   // 2 = parallelism threshold
    System.out.println(Thread.currentThread().getName() + ": " + name);
});

// 2. Parallel search - بيدور ويرجع أول نتيجة
String topStudent = scores.search(2, (name, score) -> {
    return score > 90 ? name : null;  // null يعني "كمل دوّر"
});
// topStudent = "Sara"

// 3. Parallel reduce - بيجمع كل القيم
int total = scores.reduce(2,
    (name, score) -> score,       // mapper: خد الـ score
    Integer::sum                   // reducer: اجمعهم
);
// total = 413
```

---
---

# الفصل السابع: أنماط وأمثلة عملية

## المثال 1: عد تكرار الكلمات (Word Frequency)

```java
String text = "the quick brown fox jumps over the lazy dog the fox";
String[] words = text.split(" ");

// الطريقة الأنظف: merge
Map<String, Integer> freq = new HashMap<>();
for (String word : words) {
    freq.merge(word, 1, Integer::sum);
}
// {the=3, quick=1, brown=1, fox=2, jumps=1, over=1, lazy=1, dog=1}

// الطريقة بـ Streams
Map<String, Long> freq2 = Arrays.stream(words)
    .collect(Collectors.groupingBy(w -> w, Collectors.counting()));
```

## المثال 2: تجميع العناصر (Grouping)

```java
record Student(String name, String department, int grade) {}

List<Student> students = List.of(
    new Student("Ahmed", "CS", 90),
    new Student("Sara", "CS", 85),
    new Student("Omar", "Math", 78),
    new Student("Ali", "Math", 92),
    new Student("Mona", "CS", 88)
);

// تجميع حسب القسم
Map<String, List<Student>> byDept = new HashMap<>();
for (Student s : students) {
    byDept.computeIfAbsent(s.department(), k -> new ArrayList<>()).add(s);
}
// {CS=[Ahmed, Sara, Mona], Math=[Omar, Ali]}

// بـ Streams (أنظف)
Map<String, List<Student>> byDept2 = students.stream()
    .collect(Collectors.groupingBy(Student::department));

// أعلى درجة في كل قسم
Map<String, Optional<Student>> topPerDept = students.stream()
    .collect(Collectors.groupingBy(
        Student::department,
        Collectors.maxBy(Comparator.comparingInt(Student::grade))
    ));
// {CS=Optional[Ahmed(90)], Math=Optional[Ali(92)]}

// متوسط الدرجات في كل قسم
Map<String, Double> avgPerDept = students.stream()
    .collect(Collectors.groupingBy(
        Student::department,
        Collectors.averagingInt(Student::grade)
    ));
// {CS=87.67, Math=85.0}
```

## المثال 3: عكس الـ Map (Invert)

```java
// من key→value لـ value→key
Map<String, String> countryCapital = Map.of(
    "Egypt", "Cairo",
    "Jordan", "Amman",
    "Morocco", "Rabat"
);

Map<String, String> capitalCountry = new HashMap<>();
countryCapital.forEach((country, capital) -> {
    capitalCountry.put(capital, country);
});
// {Cairo=Egypt, Amman=Jordan, Rabat=Morocco}

// بـ Streams
Map<String, String> inverted = countryCapital.entrySet().stream()
    .collect(Collectors.toMap(Map.Entry::getValue, Map.Entry::getKey));
```

::: warning
لو في values مكررة، الـ inversion هيفشل لإن هيبقى في keys مكررة. في الحالة دي لازم تحدد إزاي تتعامل مع التكرار:
```java
Map<String, String> safeInvert = map.entrySet().stream()
    .collect(Collectors.toMap(
        Map.Entry::getValue,
        Map.Entry::getKey,
        (existing, replacement) -> existing // لو في تكرار خد الأول
    ));
```
:::

## المثال 4: عمل Map من List (Indexing)

```java
record Employee(int id, String name, String dept) {}

List<Employee> employees = List.of(
    new Employee(1, "Ahmed", "Engineering"),
    new Employee(2, "Sara", "Marketing"),
    new Employee(3, "Omar", "Engineering")
);

// عمل index بالـ id
Map<Integer, Employee> byId = new HashMap<>();
for (Employee e : employees) {
    byId.put(e.id(), e);
}

// بـ Streams
Map<Integer, Employee> byId2 = employees.stream()
    .collect(Collectors.toMap(Employee::id, e -> e));

// جيب الموظف بالـ id
Employee emp = byId.get(2); // Sara
```

## المثال 5: Two Sum Problem - من أشهر مسائل الـ Interviews

```java
// عندك array من أرقام، لاقي اتنين مجموعهم يساوي target
// ارجع الـ indices بتاعتهم

public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>(); // value → index

    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[] { seen.get(complement), i };
        }
        seen.put(nums[i], i);
    }
    return new int[] {}; // مفيش حل
}

// مثال: nums = [2, 7, 11, 15], target = 9
// i=0: complement = 9-2 = 7, seen = {2→0}
// i=1: complement = 9-7 = 2, seen has 2! → return [0, 1]
```

## المثال 6: MultiMap - key واحد بأكتر من value

Java مفيهاش MultiMap بشكل مدمج، بس سهل تعمله:

```java
// Map<Key, List<Value>>
Map<String, List<String>> studentCourses = new HashMap<>();

// إضافة
studentCourses.computeIfAbsent("Ahmed", k -> new ArrayList<>()).add("Math");
studentCourses.computeIfAbsent("Ahmed", k -> new ArrayList<>()).add("Physics");
studentCourses.computeIfAbsent("Ahmed", k -> new ArrayList<>()).add("CS");
studentCourses.computeIfAbsent("Sara", k -> new ArrayList<>()).add("Math");
studentCourses.computeIfAbsent("Sara", k -> new ArrayList<>()).add("Biology");

// {Ahmed=[Math, Physics, CS], Sara=[Math, Biology]}

// البحث
List<String> ahmedCourses = studentCourses.getOrDefault("Ahmed", List.of());
// [Math, Physics, CS]
```

## المثال 7: Bi-directional Map

```java
// خريطة تقدر تدور فيها بالـ key أو بالـ value
public class BiMap<K, V> {
    private final Map<K, V> forward = new HashMap<>();
    private final Map<V, K> backward = new HashMap<>();

    public void put(K key, V value) {
        forward.put(key, value);
        backward.put(value, key);
    }

    public V getByKey(K key) { return forward.get(key); }
    public K getByValue(V value) { return backward.get(value); }
}

BiMap<String, String> countryCode = new BiMap<>();
countryCode.put("Egypt", "+20");
countryCode.put("Jordan", "+962");

countryCode.getByKey("Egypt");    // "+20"
countryCode.getByValue("+20");    // "Egypt"
```

## المثال 8: حساب الأحرف المتكررة بين كلمتين

```java
public static Map<Character, Integer> commonCharCounts(String a, String b) {
    Map<Character, Integer> countA = new HashMap<>();
    Map<Character, Integer> countB = new HashMap<>();

    for (char c : a.toCharArray()) countA.merge(c, 1, Integer::sum);
    for (char c : b.toCharArray()) countB.merge(c, 1, Integer::sum);

    Map<Character, Integer> common = new HashMap<>();
    countA.forEach((ch, count) -> {
        if (countB.containsKey(ch)) {
            common.put(ch, Math.min(count, countB.get(ch)));
        }
    });
    return common;
}

commonCharCounts("programming", "gaming");
// {g=1, a=1, m=2, i=1, n=1}
```

---
---

# الفصل التامن: الأخطاء الشائعة وإزاي تتجنبها

## الخطأ 1: التعديل أثناء اللف

```java
Map<String, Integer> map = new HashMap<>(Map.of("A", 1, "B", 2, "C", 3));

// ده هيرمي ConcurrentModificationException!
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    if (entry.getValue() < 3) {
        map.remove(entry.getKey()); // تعديل أثناء اللف
    }
}

// الحل 1: استخدم Iterator
Iterator<Map.Entry<String, Integer>> it = map.entrySet().iterator();
while (it.hasNext()) {
    Map.Entry<String, Integer> entry = it.next();
    if (entry.getValue() < 3) {
        it.remove(); // آمن
    }
}

// الحل 2: removeIf (أنظف)
map.entrySet().removeIf(entry -> entry.getValue() < 3);

// الحل 3: اجمع الـ keys وامسحهم بعدين
List<String> toRemove = new ArrayList<>();
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    if (entry.getValue() < 3) toRemove.add(entry.getKey());
}
toRemove.forEach(map::remove);
```

## الخطأ 2: استخدام Mutable Object كـ Key

```java
// خطير! لو غيرت الـ object بعد ما حطيته كـ key، الـ Map هتتوه
List<String> key = new ArrayList<>(List.of("A", "B"));
Map<List<String>, String> map = new HashMap<>();
map.put(key, "found");

System.out.println(map.get(key)); // "found"

key.add("C"); // غيرت الـ key! hashCode اتغير!

System.out.println(map.get(key)); // null!! الـ Map مش لاقياه
// العنصر لسه موجود لكن في bucket قديم بـ hash قديم
// والـ Map بتدور في bucket جديد بـ hash جديد

// القاعدة: استخدم Immutable objects كـ keys دايما
// String, Integer, Long, Enum - كلهم immutable وآمنين
```

## الخطأ 3: الخلط بين null و عدم الوجود

```java
Map<String, Integer> map = new HashMap<>();
map.put("Ahmed", null); // ينفع! HashMap بتقبل null values

Integer val = map.get("Ahmed"); // null ← موجود بس قيمته null
Integer val2 = map.get("Sara"); // null ← مش موجود

// إزاي تفرق؟
map.containsKey("Ahmed"); // true
map.containsKey("Sara");  // false

// أو استخدم getOrDefault
int score = map.getOrDefault("Ahmed", -1); // -1 ← لإن القيمة null
// بس كده مش هتفرق بين null و عدم الوجود

// الأحسن: متحطش null values. لو القيمة مش موجودة، متحطهاش أصلا.
```

## الخطأ 4: افتراض ترتيب HashMap

```java
Map<String, Integer> map = new HashMap<>();
map.put("C", 3);
map.put("A", 1);
map.put("B", 2);

// مش مضمون إن الترتيب هيكون C, A, B أو A, B, C أو أي ترتيب معين
// ممكن يتغير بعد resize كمان!

// لو عايز ترتيب الإضافة → LinkedHashMap
// لو عايز ترتيب أبجدي → TreeMap
```

## الخطأ 5: استخدام == بدل equals مع Integer Keys

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(200, "two hundred");

// ده شغال لإن Java بتعمل cache لـ Integer من -128 لـ 127
Integer key1 = 1;
map.get(key1); // "one" ✓

// ده ممكن يفشل لو استخدمت == لإن 200 خارج الـ cache range
Integer a = 200;
Integer b = 200;
System.out.println(a == b);      // false! (objects مختلفة)
System.out.println(a.equals(b)); // true

// بس الـ HashMap بتستخدم equals() من جواها، فمفيش مشكلة مع get/put
// المشكلة بتظهر لو أنت بتقارن keys بنفسك بـ ==
```

---
---

# الفصل التاسع: جدول المقارنة الشامل

## متى تستخدم أي نوع Map

```
السيناريو                                          استخدم
─────────────────────────────────────────────────────────────────
أغلب الحالات العادية                               → HashMap
عايز ترتيب الإضافة                                 → LinkedHashMap
عايز العناصر مرتبة بالـ key                         → TreeMap
عايز عمليات زي floor/ceiling/range                  → TreeMap
الـ keys من نوع Enum                                → EnumMap
أكتر من thread بيستخدم الـ Map                      → ConcurrentHashMap
عايز الـ GC يمسح entries مش مستخدمة                → WeakHashMap
عايز Map ثابتة مش بتتعدل                           → Map.of() / Map.copyOf()
عايز cache بحجم محدود                              → LinkedHashMap (LRU)
```

## جدول الأداء

```
العملية           HashMap    LinkedHashMap    TreeMap    EnumMap    ConcurrentHashMap
──────────────────────────────────────────────────────────────────────────────────────
put               O(1)       O(1)             O(log n)   O(1)      O(1)
get               O(1)       O(1)             O(log n)   O(1)      O(1)
remove            O(1)       O(1)             O(log n)   O(1)      O(1)
containsKey       O(1)       O(1)             O(log n)   O(1)      O(1)
iteration         غير مرتب   ترتيب إضافة      مرتب      ترتب enum  غير مرتب
null key          ايوا       ايوا             لأ         لأ        لأ
null value        ايوا       ايوا             ايوا       ايوا      لأ
Thread-safe       لأ         لأ               لأ         لأ        ايوا
```

## الـ Methods الشاملة

```
Method                     الوظيفة                                        متاحة في
────────────────────────────────────────────────────────────────────────────────────────
put(K, V)                  حط/غير                                        الكل
get(K)                     هات                                           الكل
remove(K)                  امسح                                          الكل
containsKey(K)             موجود؟                                        الكل
size()                     العدد                                         الكل
isEmpty()                  فاضي؟                                         الكل
clear()                    امسح الكل                                     الكل
keySet()                   كل الـ keys                                   الكل
values()                   كل الـ values                                 الكل
entrySet()                 كل الـ entries                                 الكل
putIfAbsent(K, V)          حط لو مش موجود                                الكل
getOrDefault(K, V)         هات أو default                                الكل
replace(K, V)              غير لو موجود                                  الكل
remove(K, V)               امسح لو key و value مطابقين                   الكل
replaceAll(BiFunction)     غير كل القيم بـ function                      الكل
merge(K, V, BiFunction)    ادمج القيمة الجديدة مع القديمة                الكل
compute(K, BiFunction)     احسب قيمة جديدة                               الكل
computeIfAbsent(K, Func)   احسب لو مش موجود                              الكل
computeIfPresent(K, Func)  احسب لو موجود                                 الكل
forEach(BiConsumer)        لف على كل entry                               الكل
─── خاصة بـ TreeMap / NavigableMap ───
firstKey() / lastKey()     أصغر/أكبر key                                 TreeMap
firstEntry() / lastEntry() أصغر/أكبر entry                               TreeMap
lowerKey(K) / higherKey(K) أقرب أصغر/أكبر key                            TreeMap
floorKey(K) / ceilingKey(K) أقرب أصغر=/أكبر= key                         TreeMap
headMap(K) / tailMap(K)    جزء من الـ Map                                 TreeMap
subMap(K, K)               جزء بين key و key                             TreeMap
descendingMap()            ترتيب عكسي                                    TreeMap
pollFirstEntry/pollLast    شيل وارجع أصغر/أكبر                          TreeMap
```
