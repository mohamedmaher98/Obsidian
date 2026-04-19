# Roadmap: Advanced Java Fundamentals
## هدف: قراءة وفهم أي كود — مع التطبيق على بيئة ERP

---

## نظرة عامة

| المرحلة | الأسابيع | المحور الرئيسي |
|---|---|---|
| 1 | 1–2 | JVM Internals & Memory Model |
| 2 | 3–4 | Advanced OOP & Design Philosophy |
| 3 | 5–6 | Collections Internals & Big O |
| 4 | 7–9 | Concurrency & Multithreading |
| 5 | 10–11 | Reflection & Annotations |
| 6 | 12–14 | Design Patterns in Enterprise |

---

## المرحلة الأولى — أسبوع 1–2
### JVM Internals: Memory, Stack, Heap, GC

**المشكلة الجوهرية التي ستحلها:**
لما تقرأ كود Legacy وتشوف `static` field، أو `ThreadLocal`، أو كائن بيتعمل جوه loop — مش هتفهم خطورته أو سلوكه من غير فهم أين يعيش في الذاكرة.

---

### الأسبوع الأول — Memory Architecture

**المفاهيم المستهدفة:**

- **Method Area / Metaspace:** فين بيتخزن الـ class metadata، والـ `static` fields، والـ bytecode.
- **Heap:** الـ Young Generation (Eden + Survivors) والـ Old Generation — وليه الفصل ده موجود أصلاً.
- **Stack:** كل thread عنده stack خاص، كل method call بتعمل stack frame جديد فيه الـ local variables والـ operand stack.
- **PC Register & Native Method Stack:** للاكتمال النظري.

**المرجع:**
- *"Understanding the JVM: Advanced Features and Best Practices"* — Charlie Hunt & Binu John (الفصول 2–4)
- كبديل مجاني: مقالات Oracle الرسمية عن JVM Architecture + فيديوهات Heinz Kabutz على Javaspecialists.eu

---

### الأسبوع الثاني — Garbage Collection

**المفاهيم المستهدفة:**

- كيف يحدد الـ GC الـ "live objects" عبر الـ **GC Roots** (stack variables, static fields, JNI references).
- الفرق بين **Serial / Parallel / G1 / ZGC** وإمتى يُستخدم كل منهم.
- مفهوم **Stop-The-World** وأثره على الـ Performance.
- **Memory Leaks في Java:** كيف يحدثون رغم وجود GC (static collections, unclosed resources, listeners).

**المرجع:**
- *"Java Performance: The Definitive Guide"* — Scott Oaks (الفصل 6–7)

---

### التمرين العملي — المرحلة الأولى

**الهدف: تحليل كود، مش كتابة كود.**

افتح أي Service class في الـ ERP بتاعكم وجاوب على الأسئلة دي:

1. كل variable محلية في الـ methods — هتعيش في **Stack** ولا **Heap**؟ ليه؟
2. في الـ `static` fields في الـ class — هتتخزن فين بالظبط؟ إيه خطورة لو بيُضاف عليها data مع الوقت؟
3. لو في أي `List` أو `Map` بتتعمل كـ `static` ومش بتتفرغ — ده Memory Leak محتمل. حدده.
4. ابحث عن أي استخدام لـ `ThreadLocal` في الكود — فسّر ليه المطور استخدمه هنا.

---

### التطبيق على بيئة ERP

في الـ Hibernate Session Management: فهم إن كل Hibernate Session بتعمل Objects في الـ Heap وبتحتفظ بـ First-Level Cache — ده بيفسر ليه الـ `LazyInitializationException` بيحصل لما تحاول توصل لـ entity بعد ما الـ Session اتقفلت.

في الـ Spring Beans: الـ Singleton beans بتعيش في الـ Metaspace/Heap طول عمر التطبيق — لو فيه state داخل الـ singleton bean ده مشكلة كبيرة في الـ multi-user ERP.

---
---

## المرحلة الثانية — أسبوع 3–4
### Advanced OOP: SOLID, Composition, Interface Hidden Behavior

**المشكلة الجوهرية التي ستحلها:**
قراءة كود Legacy بيكون صعب لأن المطورين كتبوه بأساليب مختلفة — بعضها OOP سليم وبعضها مش سليم. لازم تعرف تميّز الاتنين وتفهم الـ intent وراء كل design.

---

### الأسبوع الثالث — SOLID Principles

**المفاهيم المستهدفة:**

- **Single Responsibility:** كل class ليها سبب واحد للتغيير. كيف تكتشف انتهاكه في Legacy code؟
- **Open/Closed:** الكود مفتوح للتوسعة مغلق للتعديل. الـ `if/else` الطويلة دي غالباً انتهاك.
- **Liskov Substitution:** لو عندك `Animal animal = new Dog()` — المفروض الكود يشتغل صح من غير ما يعرف إنه `Dog`.
- **Interface Segregation:** Interface كبيرة بتخلي classes تـ implement methods مش بتحتاجها.
- **Dependency Inversion:** الكود بيعتمد على Abstractions مش على Implementations — ده أساس فهم Spring DI.

**المرجع:**
- *"Clean Code"* — Robert C. Martin (الفصول 9–11)
- *"Agile Software Development: Principles, Patterns, and Practices"* — Robert C. Martin (الفصول المخصصة لـ SOLID)

---

### الأسبوع الرابع — Composition & Interface Secrets

**المفاهيم المستهدفة:**

- **Composition over Inheritance:** إيه المشاكل الحقيقية للـ deep inheritance hierarchies في الكود القديم؟
- **Default Methods في Interfaces (Java 8+):** كيف أضافت Java سلوك للـ interfaces وأثر ذلك على الـ multiple inheritance.
- **Marker Interfaces:** مثل `Serializable` و `Cloneable` — بتأثر على سلوك الـ JVM من غير ما فيها methods.
- **Functional Interfaces:** الـ `@FunctionalInterface` وعلاقتها بالـ Lambdas.

**المرجع:**
- *"Effective Java"* — Joshua Bloch (Items 18, 19, 20, 22)

---

### التمرين العملي — المرحلة الثانية

**الهدف: تشخيص انتهاكات OOP في Legacy Code.**

خذ أي Class طويلة (300+ سطر) من الـ ERP وجاوب:

1. هل الـ class بتعمل أكتر من "مسؤولية واحدة"؟ اذكر المسؤوليات اللي لقيتها.
2. في الـ inheritance hierarchy — لو `class B extends A`، هل `B` فعلاً هي نوع `A` منطقياً؟ ولا مجرد reuse للكود؟
3. ابحث عن `instanceof` checks — دي غالباً انتهاك لـ Liskov أو إشارة لـ design مش سليم.
4. افحص الـ interfaces المُنفَّذة — هل الـ class بتستخدم كل الـ methods ولا في methods بتعمل `throw UnsupportedOperationException`؟

---

### التطبيق على بيئة ERP

في Hibernate Entities: الـ `@Entity` classes اللي بترث من base entity — افهم إيه اللي بيورثه الـ child فعلاً وإيه الـ Hibernate behavior الخاص بالـ inheritance strategy المستخدمة (`SINGLE_TABLE`، `JOINED`، `TABLE_PER_CLASS`).

في Spring Services: الـ Dependency Injection هو تطبيق مباشر لـ Dependency Inversion Principle — لما تشوف `@Autowired` فافهم إن Spring بيـ inject الـ implementation عبر الـ interface.

---
---

## المرحلة الثالثة — أسبوع 5–6
### Collections Framework: Big O & Internal Implementation

**المشكلة الجوهرية التي ستحلها:**
لما تشوف `HashMap` بدل `TreeMap` أو `LinkedList` بدل `ArrayList` في الكود — مش هتفهم ليه المطور اختار ده إلا لو عارف التكلفة الحقيقية لكل operation.

---

### الأسبوع الخامس — Big O لكل Collection

| Collection | Get | Add | Remove | Contains | Notes |
|---|---|---|---|---|---|
| ArrayList | O(1) | O(1) amortized | O(n) | O(n) | أسرع في random access |
| LinkedList | O(n) | O(1) | O(1) | O(n) | أسرع في add/remove من المنتصف |
| HashMap | O(1) avg | O(1) avg | O(1) avg | O(1) avg | يتدهور لـ O(n) في worst case |
| TreeMap | O(log n) | O(log n) | O(log n) | O(log n) | مرتب دايماً |
| HashSet | O(1) avg | O(1) avg | O(1) avg | O(1) avg | مبني على HashMap |
| PriorityQueue | O(log n) | O(log n) | O(n) | O(n) | min-heap internally |

---

### الأسبوع السادس — Internal Implementation

**المفاهيم المستهدفة:**

**HashMap Internals:**
- الـ `hashCode()` بيُحدد الـ bucket.
- الـ `equals()` بيُحدد الـ exact key جوه الـ bucket.
- الـ **Load Factor** (default 0.75) وعملية الـ **Rehashing**.
- الـ Java 8 تحسين: لما الـ bucket يتجاوز 8 elements بيتحول من LinkedList لـ Red-Black Tree.

**ArrayList Internals:**
- بيبدأ بـ initial capacity = 10.
- لما يمتلئ بيعمل new array بحجم `1.5x` وبيعمل `System.arraycopy`.
- ليه `add(index, element)` هو O(n) — لأنه بيشيل كل اللي بعده.

**المرجع:**
- *"Java Generics and Collections"* — Maurice Naftalin & Philip Wadler
- مصدر مجاني ممتاز: قراءة الـ Source Code مباشرة من OpenJDK على GitHub

---

### التمرين العملي — المرحلة الثالثة

**الهدف: تحليل اختيارات الـ Collections في الكود الموجود.**

1. ابحث في الكود عن كل استخدام لـ `HashMap` — هل الـ key class عندها `hashCode()` و `equals()` مُعرَّفين بشكل صحيح؟ (في Hibernate Entities ده مهم جداً)
2. ابحث عن `List` بتتستخدم كـ "lookup" بـ `contains()` — ده O(n)، هل ممكن تكون `Set` أفضل؟
3. ابحث عن أي loop بتُضيف لـ ArrayList داخلها — هل في `ensureCapacity` بيُستخدم؟ لو لا، ليه؟
4. ابحث عن `synchronized` على Collections — وقارن بـ `ConcurrentHashMap` إذا وجدت.

---

### التطبيق على بيئة ERP

في Hibernate: الـ `equals()` و `hashCode()` في الـ Entities مش مجرد best practice — الـ First-Level Cache (Identity Map) يعتمد عليهم مباشرة لتحديد إذا كان الـ entity موجود في الـ session أم لا.

---
---

## المرحلة الرابعة — أسبوع 7–9
### Concurrency & Multithreading

**المشكلة الجوهرية التي ستحلها:**
الـ Concurrency bugs من أصعب الأخطاء اكتشافاً في الـ Legacy Code — هي بتظهر بشكل متقطع وتحت load عالي فقط. فهم المبادئ هيخليك تشوف الـ bug المحتمل قبل ما يحصل.

---

### الأسبوع السابع — Thread Lifecycle & Synchronization

**المفاهيم المستهدفة:**

- **Thread States:** NEW → RUNNABLE → BLOCKED → WAITING → TIMED_WAITING → TERMINATED
- **Race Condition:** متى يحدث وكيف تتعرف عليه في الكود؟
- **`synchronized`:** الـ intrinsic lock — بيعمل ايه بالظبط على الـ object monitor.
- **`volatile`:** يضمن visibility مش atomicity — الفرق المهم.
- **`AtomicInteger` وأخواتها:** كيف تعمل بدون locks عبر CAS (Compare-And-Swap).
- **Deadlock:** الشروط الأربعة لحدوثه وكيف تكتشفه في thread dump.

**المرجع:**
- *"Java Concurrency in Practice"* — Brian Goetz et al. (الكتاب الأساسي في الموضوع ده — الفصول 1–5)

---

### الأسبوع الثامن — Executors & Thread Pools

**المفاهيم المستهدفة:**

- **ThreadPool:** ليه إنشاء thread لكل request مشكلة (overhead عالي).
- **`ExecutorService`:** `FixedThreadPool`، `CachedThreadPool`، `ScheduledThreadPool` — الفرق بينهم.
- **`Future` و `Callable`:** الفرق عن `Runnable` وكيف تحصل على نتيجة من thread.
- **`BlockingQueue`:** كيف تعمل الـ thread pools داخلياً.

**المرجع:**
- *"Java Concurrency in Practice"* — الفصول 6–8

---

### الأسبوع التاسع — CompletableFuture & Modern Concurrency

**المفاهيم المستهدفة:**

- **`CompletableFuture`:** الفرق عن `Future` العادي — non-blocking composition.
- **`thenApply` vs `thenCompose` vs `thenCombine`:** متى تستخدم كل واحدة.
- **Exception Handling في async code:** `exceptionally()` و `handle()`.
- **Virtual Threads (Java 21+):** المفهوم الأساسي وأثره على ERP applications.

**المرجع:**
- *"Modern Java in Action"* — Raoul-Gabriel Urma et al. (الفصل 15–16)

---

### التمرين العملي — المرحلة الرابعة

**الهدف: تحليل الـ Concurrency risks في الكود الموجود.**

1. ابحث عن كل الـ `static` fields اللي مش `final` — كل واحدة منهم هي Shared Mutable State محتملة.
2. ابحث عن `synchronized` methods — هل الـ lock على الـ object كله ضروري ولا ممكن يكون أضيق؟
3. ابحث عن `HashMap` بتتستخدم من أكتر من thread — لازم تكون `ConcurrentHashMap`.
4. افحص أي Singleton pattern في الكود — هل هو thread-safe؟ هل بيستخدم double-checked locking بشكل صحيح؟

---

### التطبيق على بيئة ERP

في Tomcat: كل HTTP request بييجي على thread منفصل من الـ thread pool. أي `static` mutable field في الـ Spring beans هو Race Condition محتمل في بيئة الـ multi-user ERP. الـ Hibernate `SessionFactory` هي thread-safe لكن الـ `Session` نفسها **مش** thread-safe.

---
---

## المرحلة الخامسة — أسبوع 10–11
### Reflection API & Annotations

**المشكلة الجوهرية التي ستحلها:**
Spring و Hibernate مبيان منهم كود — بيشتغلوا بـ "magic". الـ Reflection هو الـ magic. لما تفهمه هتقرأ الـ Framework code وتفهم "إيه اللي بيحصل فعلاً" لما بتكتب `@Autowired` أو `@Entity`.

---

### الأسبوع العاشر — Reflection API

**المفاهيم المستهدفة:**

- **`Class<?>` object:** كل class في Java عندها object من نوع `Class` — ده بوابة الـ Reflection.
- **`getDeclaredFields()` / `getDeclaredMethods()`:** الفرق بين `declared` (كل حاجة) و `public` (المعلنة فقط).
- **`setAccessible(true)`:** كيف Hibernate بيوصل لـ private fields مباشرة من غير getters.
- **Dynamic Proxy:** `java.lang.reflect.Proxy` — كيف Spring بيعمل الـ AOP interceptors.
- **الـ Performance cost:** ليه الـ Reflection أبطأ من الـ Direct call وكيف Frameworks بتـ cache النتائج.

**المرجع:**
- *"Java Reflection in Action"* — Ira Forman & Nate Forman
- كبديل: Oracle Java Documentation على Reflection + قراءة مصدر Spring `BeanFactory`

---

### الأسبوع الحادي عشر — Annotations

**المفاهيم المستهدفة:**

- **Annotation Retention Policies:** `SOURCE` (للـ compiler فقط) / `CLASS` (في bytecode) / `RUNTIME` (متاح بالـ Reflection).
- **Annotation Targets:** على الـ field، method، class، parameter.
- **Processing Annotations بالـ Reflection:** `getAnnotation()` / `isAnnotationPresent()`.
- **AnnotationProcessor (Compile-time):** مثل Lombok — بيشتغل قبل الـ compilation مش في الـ runtime.

**كيف تقرأ كود Spring بعد ده:**
- `@Autowired` → Spring بيسكن الـ ApplicationContext classes بالـ Reflection، لاقي الـ annotation، inject الـ bean.
- `@Transactional` → Spring بيعمل Dynamic Proxy للـ class، لما method بـ `@Transactional` بتتكال، الـ proxy بيفتح transaction الأول.
- `@Entity` في Hibernate → Hibernate بيعمل scan للـ classpath، لاقي الـ annotation، بنى الـ mapping من الـ fields.

**المرجع:**
- *"Spring in Action"* — Craig Walls (الفصول المخصصة للـ AOP و DI internals)

---

### التمرين العملي — المرحلة الخامسة

**الهدف: تتبع كيف Framework يعالج annotation موجودة في الكود.**

1. خذ أي `@Service` class في الـ ERP — تتبع خطوات Spring من `@ComponentScan` حتى تسجيل الـ bean في الـ ApplicationContext.
2. خذ أي method بـ `@Transactional` — ارسم بالخطوات ما الذي يحدث عند استدعائها (AOP Proxy → Transaction Manager → Actual Method → Commit/Rollback).
3. خذ أي `@Entity` — افهم كيف Hibernate يبني الـ `ClassMetadata` منها باستخدام Reflection على الـ fields والـ annotations.

---

### التطبيق على بيئة ERP

فهم Reflection يفسر سلوكيات مهمة: ليه Hibernate محتاج no-arg constructor في الـ Entities (بيعملها instance بالـ Reflection)، وليه الـ Spring Bean creation بيبطّأ عند الـ startup (بيـ scan وبيعمل Reflection على كل الـ classes).

---
---

## المرحلة السادسة — أسبوع 12–14
### Design Patterns in Enterprise Systems

**المشكلة الجوهرية التي ستحلها:**
الـ Legacy Code مكتوب على أنماط تصميمية. من غير معرفتها، بتقرأ كود ولا بتفهم الـ intent. بعد ما تعرفها، بتقرأ نفس الكود وبتقول "آه، ده Decorator Pattern" — وفجأة كل حاجة واضحة.

---

### الأسبوع الثاني عشر — Creational Patterns

| Pattern | المشكلة اللي بيحلها | وين بتلاقيه في ERP/Spring |
|---|---|---|
| **Singleton** | object واحد للـ application كلها | Spring Beans (default scope) |
| **Factory Method** | إنشاء objects بدون تحديد الـ class | `SessionFactory`، `BeanFactory` |
| **Abstract Factory** | عائلة من الـ objects المترابطة | JDBC Drivers، JPA Providers |
| **Builder** | بناء objects معقدة خطوة بخطوة | `CriteriaBuilder` في JPA |
| **Prototype** | نسخ objects موجودة | `BeanDefinition.clone()` في Spring |

---

### الأسبوع الثالث عشر — Structural Patterns

| Pattern | المشكلة اللي بيحلها | وين بتلاقيه في ERP/Spring |
|---|---|---|
| **Proxy** | تحكم في الوصول لـ object | Spring AOP، Hibernate Lazy Loading |
| **Decorator** | إضافة سلوك بدون inheritance | Java I/O Streams، Spring Security Filters |
| **Adapter** | ربط interfaces غير متوافقة | JDBC Adapters، Spring MVC HandlerAdapter |
| **Facade** | واجهة بسيطة لنظام معقد | Service Layer في ERP |
| **Composite** | معاملة objects والـ collections موحدة | UI Component trees، Permission hierarchies |

---

### الأسبوع الرابع عشر — Behavioral Patterns

| Pattern | المشكلة اللي بيحلها | وين بتلاقيه في ERP/Spring |
|---|---|---|
| **Observer** | إشعار عدة objects عند تغيير حالة | Spring Events، `ApplicationListener` |
| **Strategy** | تغيير الخوارزمية في runtime | Sort strategies، Payment processors |
| **Template Method** | هيكل خوارزمية ثابت مع خطوات قابلة للتغيير | `JdbcTemplate`، `HibernateTemplate` |
| **Chain of Responsibility** | تمرير request عبر سلسلة من handlers | Servlet Filters، Spring Security Filter Chain |
| **Command** | تغليف العملية كـ object | `Runnable`، Task queues في ERP |

**المرجع:**
- *"Design Patterns: Elements of Reusable Object-Oriented Software"* — Gang of Four (المرجع الأساسي)
- *"Head First Design Patterns"* — Freeman & Robson (أسهل في القراءة والفهم)

---

### التمرين العملي — المرحلة السادسة

**الهدف: تحديد الأنماط في الكود الموجود.**

1. افتح أي **Filter** أو **Interceptor** في الـ ERP — حدد هل هو Decorator أم Chain of Responsibility وبرر.
2. افتح الـ **Service Layer** — حدد الـ Facade pattern وما هو الـ "complexity" اللي بيخبيه.
3. ابحث عن أي `*Template` class في الكود (أو في الـ Spring libraries المستخدمة) — افهم كيف Template Method Pattern محقق.
4. ابحث عن أي `*Factory` أو `*Builder` class — حدد لماذا اختار المطور هذا الـ pattern بدل الـ `new` المباشر.

---

### التطبيق على بيئة ERP

الـ ERP systems هي تجمع أنماط: الـ Workflow engine غالباً Chain of Responsibility، الـ Reporting engine غالباً Strategy + Template Method، الـ Integration layer غالباً Adapter، والـ Permission system غالباً Proxy + Composite.

---
---

## خلاصة المسار التراكمي

```
أسبوع 1–2:   أنت تفهم أين يعيش كل object
أسبوع 3–4:   أنت تفهم ليه الكود مهيكل بالطريقة دي
أسبوع 5–6:   أنت تفهم التكلفة الحقيقية لكل سطر كود
أسبوع 7–9:   أنت تفهم الـ hidden bugs في الكود المتعدد الـ threads
أسبوع 10–11: أنت تفهم الـ "magic" اللي Spring و Hibernate بيعملوه
أسبوع 12–14: أنت تفهم الـ intent وراء كل design decision
```

---

## نصيحة منهجية

لكل موضوع، قبل ما تقرأ المرجع — حاول تفتح كود حقيقي من الـ ERP وتسأل نفسك "إيه اللي مش فاهمه هنا؟". الأسئلة دي هتخلي قراءتك للمرجع أكتر تركيزاً وأسرع استيعاباً بكتير من القراءة بدون سياق.

---

*Roadmap prepared for: Java Developer (1.5 years experience) — ERP Domain*
*Focus: Code Reading & Understanding over Code Writing*
