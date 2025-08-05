Methods and Encapsulation
Working with Methods and Encapsulation
■ Create methods with arguments and return values; including
overloaded methods
■ Apply the static keyword to methods and fields
■ Create and overload constructors; include impact on default
constructors
■ Apply access modifiers
■ Apply encapsulation principles to a class
■ Determine the effect upon object references and primitive
values when they are passed into methods that change the
values
✓ Working with Selected classes from the Java API
■ Write a simple Lambda expression that consumes a Lambda
Predicate expression

---
### ==Designing Methods==

![[Pasted image 20250505125134.png]]

| Element                    | Example                     | Required?             |
| -------------------------- | --------------------------- | --------------------- |
| Access modifier            | `public`                    | No                    |
| Optional specifier         | `final`                     | No                    |
| Return type                | void                        | yes                   |
| Method name                | nap                         | yes                   |
| Parameter list             | int minutes                 | yes but can be empity |
| Optional exception<br>list | throws InterruptedExecption | no                    |
| Method body                | { // body}                  | yes but can be empity |
|                            |                             |                       |

---
### ==Access Modifiers==

Java offers four choices of access modifier:
public :: The method can be called from any class.
private  :: The method can only be called from within the same class.
protected :: The method can only be called from classes in the same package or subclasses.
Default (Package Private)  :: Access The method can only be called from classes in the same
package. This one is tricky because there is no keyword for default access. You simply omit
the access modifier.

==protected== : 

- نفس **الباكدج (package)**.
    
- **وأيضًا في الكلاسات اللي بتورّث (subclasses)** حتى لو في **باكدج مختلف**.

==Default :==
لو **ما كتبتش أي access modifier** (يعني لا `public` ولا `protected` ولا `private`) → يبقى الوصول **default**
- والـ **default access** معناه إن العنصر **متاح بس داخل نفس الـ package** فقط.
    
- **مش مسموح** للكلاسات اللي في باكدجات تانية توصل له، حتى لو ورثت منه.

| Access Modifier | داخل نفس الكلاس | داخل نفس الـ package | في الـ subclass (وراثة) | في أي مكان (عام) |
| --------------- | --------------- | -------------------- | ----------------------- | ---------------- |
| *default*       | ✅               | ✅                    | ❌                       | ❌                |
| `protected`     | ✅               | ✅                    | ✅                       | ❌                |
| `public`        | ✅               | ✅                    | ✅                       | ✅                |
| `private`       | ✅               | ❌                    | ❌                       | ❌                |

---
### ==Optional Specifiers==

هي كلمات مفتاحية (keywords) **اختيارية** ممكن تضيفها لتعطي خصائص معينة للمتغير أو الميثود أو الكلاس، لكنها **ليست مطلوبة** لبناء الكود.

| Specifier             | المعنى / الاستخدام                                                    | مثال                            |
| --------------------- | --------------------------------------------------------------------- | ------------------------------- |
| `final`               | يمنع التغيير (على المتغير، الميثود، أو الكلاس)                        | `final int x = 5;`              |
| `static`              | يربط العنصر بالكلاس بدلاً من الكائن                                   | `static int counter = 0;`       |
| `abstract`            | يعلن أن الكلاس أو الميثود غير مكتمل                                   | `abstract void draw();`         |
| `synchronized`        | يمنع التزامن المتعدد (multi-thread access)                            | `synchronized void update() {}` |
| `native`              | يشير إلى أن الميثود مكتوب بلغة غير Java                               | `native void read();`           |
| `strictfp`            | يفرض دقة ثابتة للعمليات الحسابية العائمة                              | `strictfp class Calculator {}`  |
| `transient`           | يستثني المتغير من serialization                                       | `transient int tempData;`       |
| `volatile`            | يضمن أن القراءة والكتابة على المتغير تحدث من الذاكرة الرئيسية مباشرةً | `volatile boolean flag;`        |
| `default` (interface) | لتعريف ميثود افتراضي داخل interface                                   | `default void log() {}`         |

==final== 
متغير	    لا يمكن تغيير قيمته بعد التهيئة	
ميثود  	لا يمكن override لها في subclass
كلاس	لا يمكن توريثه
==static==
متغير  	قيمة مشتركة بين كل الكائنات
ميثود  	يُستدعى بدون كائن
كلاس داخلي	Nested class مستقل عن الكائن الخارجي	

==abstract== 

تعني "غير مكتمل" – لازم يتم توريثه واستكماله.
ميثود 	مفيش له body، لازم يتنفذ في subclass
كلاس	فيه ميثودات abstract أو غير مكتملة	
==default==
في `default` (فقط داخل `interface`)

🧩 Optional Specifiers in Java – استخداماتها ومكان استخدامها

| Specifier      | المعنى / التأثير                                                | مثال                                 | ينفع مع كلاس | ينفع مع ميثود | ينفع مع متغير |
|----------------|------------------------------------------------------------------|--------------------------------------|:------------:|:-------------:|:-------------:|
| `final`        | يمنع التغيير أو التوريث أو override                              | `final int x = 5;`                   | ✅           | ✅            | ✅            |
| `static`       | يربط العنصر بالكلاس بدلاً من الكائن                              | `static int counter = 0;`           | ❌           | ✅            | ✅            |
| `abstract`     | يجب توريثه واستكماله – لا يحتوي على تنفيذ                       | `abstract void draw();`             | ✅           | ✅            | ❌            |
| `synchronized` | يمنع تعارض الـ threads على نفس الميثود                           | `synchronized void update() {}`     | ❌           | ✅            | ❌            |
| `native`       | الميثود مكتوب بلغة تانية (مثل C/C++) – يُستخدم مع JNI           | `native void read();`               | ❌           | ✅            | ❌            |
| `strictfp`     | يضمن دقة موحدة للعمليات العائمة (float/double) عبر الأنظمة      | `strictfp class Calc {}`            | ✅           | ✅            | ❌            |
| `transient`    | يستثني المتغير من Serialization                                  | `transient String password;`        | ❌           | ❌            | ✅            |
| `volatile`     | يضمن قراءة/كتابة مباشرة من الذاكرة في الـ multi-threading        | `volatile boolean flag;`            | ❌           | ❌            | ✅            |
| `default`      | يسمح بتنفيذ داخل interface (من Java 8 فما فوق)                   | `default void log() {}`             | ❌           | ✅ (في interface فقط) | ❌     |

---
### ==Return Type==
- **كل ميثود لازم يكون له return type**.
    
    - لو الميثود مش بيرجع حاجة → لازم يكون `void`.
        
- **لو return type مش void** → لازم يكون فيه `return` statement بيرجع قيمة من النوع ده.
    
- **لو return type هو void**:
    
    - ينفع تكتب `return;` (من غير قيمة).
        
    - أو متكتبش `return` خالص.
---
### **==Method Name==**
- **لقواعد**:
    
    - اسم الميثود يجب أن يحتوي فقط على حروف، أرقام، أو الرموز `$` و `_`.
        
    - **لا يمكن أن يبدأ الاسم برقم**.
        
    - **لا يمكن أن يكون الاسم محجوزًا (reserved word)**.
        
    - **القاعدة التقليدية**: يبدأ الاسم بحرف صغير، لكن ده مش إلزامي.
        
- **أمثلة**:
    
    - `public void walk1() { }` – اسم ميثود صحيح.
        
    - `public void 2walk() { }` – غير صحيح، لأنه لا يمكن أن يبدأ الاسم برقم.
        
    - `public walk3 void() { }` – غير صحيح، لأن الميثود لا بد أن تبدأ بالـ return type وليس العكس.
        
    - `public void Walk_$() { }` – صحيح، رغم أن الأسماء التي تبدأ بحرف كبير ليست مفضلة، لكنها قانونية.
        
    - `public void() { }` – غير صحيح، لأن اسم الميثود مفقود.
---
### **==Parameter List==**
- **الملاحظات**:
    
    - القائمة تعتبر جزءًا إلزاميًا في تعريف الميثود، لكن مش ضروري تحتوي على معاملات.
        
    - **إذا كان هناك معاملات، فالمعاملات مفصولة بفاصلة**.
        
- **أمثلة**:
    
    - `public void walk1() { }` – ميثود بدون معاملات، لكن القوسين لازم يكونوا موجودين.
        
    - `public void walk2 { }` – غير صحيح، لأن الأقواس مفقودة.
        
    - `public void walk3(int a) { }` – ميثود صحيحة مع معامل واحد.
        
    - `public void walk4(int a; int b) { }` – غير صحيح، لأن المعاملات مفصولة بـ `;` بدل من `,`.
        
    - `public void walk5(int a, int b) { }` – ميثود صحيحة مع معاملات مفصولة بـ 
---
### ==**Optional Exception List**== 

 3. **قائمة الاستثناءات (Optional Exception List)**:

- **الملاحظات**:
    
    - يمكنك تعريف استثناءات يتم رميها داخل الميثود باستخدام الكلمة المفتاحية `throws`.
        
    - هذه القائمة اختيارية ويمكن أن تحتوي على أكثر من استثناء مفصولين بفواصل.
        
- **أمثلة**:
    
    - `public void zeroExceptions() { }` – ميثود بدون استثناءات.
        
    - `public void oneException() throws IllegalArgumentException { }` – ميثود فيها استثناء واحد.
        
    - `public void twoExceptions() throws IllegalArgumentException, InterruptedException { }` – ميثود فيها استثناءين.

----
### ==**Varargs**==

الـ **Varargs** يسمح لك بتمرير عدد غير محدد من المعاملات إلى الميثود كأنها مصفوفة. لكن هناك بعض القواعد التي يجب اتباعها عند استخدامها.
- **يجب أن يكون الـ Varargs هو آخر عنصر في قائمة المعاملات**. بمعنى آخر، لا يمكن أن تضع أي معاملة بعد الـ Varargs في تعريف الميثود.
    
- **يمكن أن يكون لديك Varargs واحد فقط في الميثود**. لا يمكنك استخدام أكثر من واحد.
ex : 
public void walk1(int... nums) { }
public void walk2(int start, int... nums) { }
 **استدعاء الميثود مع Varargs**:

عند استدعاء ميثود تحتوي على Varargs، لديك عدة خيارات:

- يمكنك تمرير مصفوفة.
    
- يمكنك تمرير عناصر فردية وستقوم Java بإنشاء المصفوفة تلقائيًا.
    
- يمكنك ترك الـ Varargs فارغًا، وJava ستقوم بإنشاء مصفوفة فارغة.
    

#### **أمثلة على استدعاء الميثود مع Varargs:**
public static void walk(int start, int... nums) {
    System.out.println(nums.length);  // الطول المصفوفة nums
}

public static void main(String[] args) {
    walk(1);                  // 0
    walk(1, 2);               // 1
    walk(1, 2, 3);            // 2
    walk(1, new int[] {4, 5}); // 2
}
walk(1, null); // Throws a NullPointerException

 **الملخص**:

- **Varargs** يجب أن يكون في آخر المعاملات في الميثود.
    
- يمكنك استدعاء الميثود مع Varargs باستخدام قائمة من القيم أو مصفوفة.
    
- **Java** تخلق مصفوفة تلقائيًا للقيم الممررة عبر Varargs، وإذا لم يتم تمرير أي قيمة، يتم إنشاء مصفوفة فارغة.
    
- يمكن أن يسبب `null` مع Varargs مشاكل (مثل `NullPointerException`).
---


---
### ==Applying Access Modifiers==

**Private:**
Only code in the same class can call private methods or access
private fields.

**Default**:
only classes in the package may access it 

**Protected Access** :
Protected access allows everything that default (package private) access allows and more.
The protected access modifier adds the ability to access members of a parent class.

**Public Access :**
 public means anyone can access the member from anywhere.

---
### **==Static methods and fields Calls==**

- **الـ static method** زيها زي أي method من ناحية إن **الكود نفسه مش بيتكرر**.
- لما تستدعي method، سواء static أو non-static:
    - **البارامترات (parameters)** و **المتغيرات المحلية (local variables)** بتتحط في **الـ Stack**، وده بيكون **نسخة مستقلة** لكل استدعاء.
- لكن الكود نفسه (جسم الدالة) موجود **مرة واحدة في الذاكرة**، ومش بيتكرر مع كل استدعاء.
- public class Test {
    public static void printSum(int a, int b) {
        int sum = a + b;
        System.out.println(sum);
    }
}
- الكود بتاع `printSum` موجود **مرة واحدة**.
- لكن:
    - أول استدعاء هيحجز `a=3`, `b=4`, `sum=7` في الـ Stack. 
    - تاني استدعاء هيحجز `a=5`, `b=6`, `sum=11` في Stack جديد.
---
instance VS static 
- ال**instance method / variable**: دي بتتخزن جوه الكائن (object)، يعني لازم تكون عامل `new` عشان تقدر توصل لها.
- ال**static method / variable**: دي بتتخزن مرة واحدة بس على مستوى الكلاس نفسه، مش الكائن، يعن تقدر تستخدمها من غير ما تعمل `new`.
- ال"Static context معندوش access على أي حاجة غير static. إنما Instance context يقدر يشوف كل حاجة."
---
### ==Static Initialization Block==

هو بلوك كود بيبدأ بـ `static {}`، وبيشتغل **مرة واحدة بس** عند أول استخدام للكلاس (سواء بإنشاء object أو استدعاء أي method أو variable static).
مفيد لما يكون عندك متغير `static` (أو `static final`) ومحتاج تعمله **تهيئة initialization** بناءً على عمليات حسابية أو منطق معقد.
private static final int NUM_SECONDS_PER_HOUR;
static {
    int numSecondsPerMinute = 60;
    int numMinutesPerHour = 60;
    NUM_SECONDS_PER_HOUR = numSecondsPerMinute * numMinutesPerHour;
}
امتي استخدمه 
أحيانًا محتاج تعيّن قيمة لمتغير `static final`، بس القيمة دي **مش مباشرة**، محتاج تحسبها.
static final int NUM_SECONDS;

static {
    int x = getAge;
    int y = getAge2;
    NUM_SECONDS = x * y;
}

|`final int x = 5;`|✅|اتعينت على طول|
|`final int x; x = 5;`|✅|اتعينت مرة واحدة بعدين|
|`final int x; x = 5; x = 10;`|❌|اتعينت مرتين|
|`static final int x; static { x = 5; }`|✅|اتعينت مرة واحدة في البلوك|
|`static final int x; static { x = 5; x = 6; }`|❌|اتعينت مرتين|

حاول تتجنب البلوكات عشان بتصعب القراءة فالكود 
امتي استخدمهم بجد 
لما تكون عندك **متغير `static`** (يعني يخص الكلاس كله)، وعايز تهيئه بطريقة **معقدة شويه أو أكتر من سطر**.
```java
private static final List<String> NAMES;

static {
    NAMES = new ArrayList<>();
    NAMES.add("Ahmed");
    NAMES.add("Sara");
    NAMES.add("Youssef");
}

```
ليه استخدمنا `static {}` هنا؟
- لأن التهيئة مش ممكن تتعمل في سطر واحد.
- لازم أضيف عناصر جوه `ArrayList`.
---
==Static Imports==
يعني إيه `static import`؟
بشكل طبيعي، لما تستخدم دالة `static` من كلاس تاني، لازم تكتب اسم الكلاس.
Math.pow(2, 3);
لكن لو عملت **static import**، تقدر تستخدم الدالة **مباشرة بدون اسم الكلاس**.
import static java.lang.Math.pow;
pow(2.3);
لما يكون عندك **عدة استخدامات لمتغير أو دالة static** (زي `PI`، `pow()`, أو `out.println()`).
```java
import static java.lang.Math.*;

public class Test {
    public static void main(String[] args) {
        double result = pow(2, 3); // 8.0
        double piValue = PI;      // 3.1415...
        System.out.println(result);
        System.out.println(piValue);
    }
}
```
لو استخدمت static imports كتير أو من كذا كلاس، الكود ممكن **يبقى مش واضح ومربك**. فخلي استخدامك له **بشكل معتدل**.
ال import العادي
استيراد كلاس بالكامل
الstatic import
استيراد دالة أو متغير static فقط

---
### **==Passing Data Among Methods==**

**الفرق بين تعديل القيمة نفسها** وتعديل **الكائن اللي القيمة بتشاور عليه** لما تبعت داتا لدالة في Java، وده جزء مهم جدًا في فهم إزاي Java بتتعامل مع **الأنواع البدائية (primitives)** و **الكائنات (objects)**.
java is pass by value language

Primitive Types
public static void main(String[] args) {
    int num = 4;
    newNumber(num);
    System.out.println(num); // 4
}
public static void newNumber(int num) {
    num = 8;
}

- نا `num = 4` في المين.
- لما تبعت `num` للدالة، Java بتبعت **نسخة من القيمة**.
- جوه الدالة، `num = 8` بيأثر على النسخة، مش الأصل.
- عشان كده الطباعة بتكون `4`
-في Java، لما تبعت متغير لدالة، لو كان المتغير **قيمة بدائية (primitive)** زي `int`، `char`، أو `boolean`، فده بيتبعت **نسخة من القيمة**. يعني أي تغييرات بتحصل في المتغير داخل الدالة مابتأثرش على المتغير في المكان اللي استدعاك فيه.

أما لو كان المتغير **كائن (object)** زي `String` أو `StringBuilder`، فبيبقى فيه نوع من "الإشارة" للمكان اللي الكائن موجود فيه، لكن في حالة الكائنات غير القابلة للتغيير (زي `String`)، حتى لو حاولت تعيد تعيين المتغير، مافيش تأثير على المتغير اللي في المين. أما لو الكائن **قابل للتغيير** (زي `StringBuilder`)، أي تعديل يحصل عليه بيأثر على الكائن في المين.

---
### **==Overloading Methods==**

creating methods with the same signature in the same class. Method overloading occurs when there are different method signatures with the same name but different type parameters.
هو لما تعمل أكثر من method بنفس الاسم **لكن**:
- عدد البراميترز يختلف.
- أو نوع البراميترز يختلف.
- أو ترتيبهم يختلف.
public void fly(int[] lengths) { }
public void fly(int... lengths) { } // ❌ DOES NOT COMPILE
غلط لانهم نفس الحاجه
الفرق
|`int[] lengths`|لازم تبعت Array|`fly(new int[]{1, 2, 3});`|
|`int... lengths`|ممكن تبعت أرقام مباشرة أو Array|`fly(1, 2, 3);` أو `fly(new int[]{1,2});`|

==Autoboxing==
يعني إن جافا بتحول القيم البدائية (زي `int`) تلقائيًا إلى كائنات (`Integer`) عند اللزوم.
public void fly(Integer numMiles) { }
fly(3); // ✅ يعمل بسبب autoboxing: int → Integer

public void fly(int numMiles) { }
public void fly(Integer numMiles) { }

 وقتها جافا تفضل:

- ✅ `fly(int)` لأنها **أكثر تحديدًا (more specific)**.
- ❌ مش هتلجأ لـ autoboxing طالما فيه نسخة أنسب.
- ## 🔁 قاعدة عامة:

> **Java always chooses the most specific method first**  
> وبعد كده:
> - توسع من نوع صغير لكبير (مثلاً `int` → `long`)
> - ثم autoboxing (مثلاً `int` → `Integer`)
> - ثم استخدام `Object`

Java بتعمل **تحويل واحد فقط** (One conversion)
public static void play(Long l) { }
public static void play(Long... l) { }

public static void main(String[] args) {
    play(4);   // ❌ لا يُسمح
    play(4L);  // ✅ يستدعي play(Long l)
}
- `4` هو `int`. عشان يتحول لـ `Long`، لازم يمر بـ:
    - `int` → `long` (widening)
    - `long` → `Long` (autoboxing)
- Java **مش هتعمل خطوتين تحويل**. لازم تكون خطوة واحدة فقط.
---
### **==Creating Constructor==**


لما تعمل `new Bunny()`:
Java يحجز مساحة في الذاكرة (memory allocation).
ينادي الـ constructor اللي اسمه زي اسم الكلاس.
الـ constructor يبدأ يشتغل ويهيّأ المتغيرات.
الفرق بين
this.variable = parameter; ✅ صح
parameter = this.variable; ❌ خطأ، عكس الاتجاه.

==Default Constructor==
#### Java بتعمل Constructor لوحدها، لكن بشرط:

> **لو أنت ما كتبتش أي constructor، Java بتضيف default constructor بنفسها.**
فايدة الـ private constructor:
- بيستخدم لما تكون الكلاس utility (زي `Math`).
- أو في design patterns زي Singleton، لما تحب تتحكم في عدد النسخ.
==Default Constructor== 
#### تقدر تعمل أكتر من Constructor في نفس الكلاس، بشرط:

> **الاختلاف في قائمة الباراميترات (عددًا أو نوعًا).**

 قاعدة ذهبية:

> لو هتستخدم `this(...)` لاستدعاء Constructor تاني، لازم يكون **أول سطر** فعلي (non-comment) في الـ constructor.

this
- بيستخدم لاستدعاء **Constructor آخر** داخل **نفس الكلاس**.
- لازم يكون **أول سطر** في الـ constructor
- public class Animal {
    private String name;
    private int age;

    public Animal(String name) {
        this(name, 0); // 👈 استدعاء constructor آخر في نفس الكلاس
    }

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

Super
- بيستخدم لاستدعاء **Constructor في الكلاس الأب (superclass)**.
- برضه لازم يكون **أول سطر** في الـ constructor
- مهم جدًا لو الـ superclass معندوش constructor افتراضي (بدون باراميتر).
public class Animal {
    public Animal(String name) {
        System.out.println("Animal: " + name);
    }
}

public class Dog extends Animal {
    public Dog() {
        super("Buddy"); // 👈 استدعاء constructor من الكلاس الأب
        System.out.println("Dog is created");
    }
}
Animal: Buddy
Dog is created

---
==Constructor Chaining==
دي تقنية في الجافا بنستخدمها لما عندنا أكثر من كونستركتور في نفس الكلاس، وبنخليهم يستدعوا بعض بدل ما نكرر الكود
```java
public class Mouse {
    private int numTeeth;
    private int numWhiskers;
    private int weight;
    
    // الكونستركتور الأول (بأقل باراميترز)
    public Mouse(int weight) {
        this(weight, 16); // بيستدعي الكونستركتور اللي بعده
    }
    
    // الكونستركتور الثاني
    public Mouse(int weight, int numTeeth) {
        this(weight, numTeeth, 6); // بيستدعي الكونستركتور الرئيسي
    }
    
    // الكونستركتور الرئيسي اللي بيعمل كل الشغل
    public Mouse(int weight, int numTeeth, int numWhiskers) {
        this.weight = weight;
        this.numTeeth = numTeeth;
        this.numWhiskers = numWhiskers;
    }
}
```
 ليه بنستخدمها؟

1. **تجنب تكرار الكود**: بدل ما نكتب نفس الكود في كل كونستركتور
    
2. **مرونة أكثر**: نعرف ندي قيم افتراضية للباراميترز المطلوبة
    
3. **تنظيم الكود**: كل الكونستركتورز بتستدعي الأخير اللي بيعمل الإسناد الفعلي
    
 شروط استخدامها:

- لازم استدعاء `this()` يكون أول حاجة في الكونستركتور
    
- مش ممكن تستخدم `this()` و `super()` مع بعض في نفس الكونستركتور
    
- بتشتغل من الأبسط للأكمل (من أقل باراميترز لأكتر باراميترز)
مثال توضيحي:
لما ننفذ `new Mouse(15)`:
1. بيستدعي الكونستركتور الأول (`Mouse(int weight)`)
2. الكونستركتور الأول بيستدعي الثاني ويديله الوزن + عدد أسنان افتراضي (16)
3. الكونستركتور الثاني بيستدعي الثالث ويديله الوزن + عدد الأسنان + عدد شوارب افتراضي (6)
4. الكونستركتور الثالث بيقوم بعملية الإسناد الفعلية للقيم
النتيجة النهائية: الفأر هيبقى عنده:
- وزن = 15 (القيمة اللي دخلناها)
- عدد أسنان = 16 (القيمة الافتراضية)
- عدد شوارب = 6 (القيمة الافتراضية)
---
### **==Final Fields==**

 مفهوم الـ `final` للمتغيرات:

- لما نعمل متغير `final`، معناه إنه لازم يتعين قيمته **مرة واحدة بس** وماينفعش تتغير بعد كده
- فيه 3 أماكن ممكن نعيّن فيها قيمة المتغيرات `final`:
    1. عند التعريف مباشرة
    2. في الـ instance initializer
    3. في الكونستركتور (المهم يتعين قبل ما الكونستركتور يخلص)
```java
public class MouseHouse {
    private final int volume;  // هيتعين في الكونستركتور
    private final String name = "The Mouse House";  // اتعينت عند التعريف
    
    public MouseHouse(int length, int width, int height) {
        volume = length * width * height;  // التعيين هنا
    }
}
```
 قواعد مهمة:

1. **لازم كل الـ final variables يتباصوا قيمة** قبل ما الكونستركتور يخلص
    
2. لو متغير `final` اتعين عند التعريف، **مينفعش** يتعين تاني في الكونستركتور
    
3. لو متغير `final` معندوش قيمة عند التعريف، **لازم** يتعين في كل الكونستركتورز
 ليه نستخدم final variables؟
4. **الأمان**: منع التعديل بعد التعيين
5. **الوضوح**: تعريف المتغيرات اللي مش هتتغير
6. **الأداء**: المترجم ممكن يعمل تحسينات للأداء
ملحوظة: لو حاولت تعديل قيمة `final` بعد ما اتعينت، هتظهر لك **compile error**!
---

### **==Initialization Order**==

 القواعد الأساسية لترتيب التهيئة:

1. **الستاتيك (static) يتم تنفيذه أولاً**:
    
    - المتغيرات الستاتيك
        
    - البلوكات الستاتيك
        
    - يتم تنفيذهم **مرة واحدة فقط** عند تحميل الكلاس
        
2. **الإنستانس (instance) يتم تنفيذه عند إنشاء الكائن**:
    
    - متغيرات الإنستانس
        
    - البلوكات العادية (غير الستاتيك)
        
    - يتم تنفيذهم **كل مرة** ننشئ فيها كائن جديد
        
3. **الكونستركتور يتم تنفيذه أخيراً**
```java
public class InitializationOrderSimple {
    private int x = 10;  // 3. instance variable
    { System.out.println(x); }  // 3. instance initializer
    
    private static int COUNT = 0;  // 1. static variable
    static { System.out.println(COUNT); }  // 1. static initializer
    
    public InitializationOrderSimple() {  // 4. constructor
        System.out.println("constructor");
    }
}
```

```java
public class InitializationOrder {
    private String name = "Torchie";  // 3. instance variable
    { System.out.println(name); }     // 3. instance initializer
    
    private static int COUNT = 0;     // 1. static variable
    static { System.out.println(COUNT); }  // 1. static initializer
    
    { COUNT++; System.out.println(COUNT); }  // 3. instance initializer
    
    public InitializationOrder() {    // 4. constructor
        System.out.println("constructor");
    }
    
    public static void main(String[] args) {
        System.out.println("read to construct");
        new InitializationOrder();
    }
}

```
1. الستاتيك دائماً أولاً وبالترتيب من الأعلى للأسفل
    
2. الإنستانس initializers بالترتيب من الأعلى للأسفل
    
3. الكونستركتور دائماً آخر حاجة
    
4. لو في extends (وراثة)، الكلاس الأب يتم تهيئته أولاً
==---==
### ==Encapsulation **Data**==
ليه بستخدم الgetter وال setter
- المتغير private فمحدش يعدل عليه مباشرة
- الـ setter بيسمحلنا نتحكم في القيم المسموح بيها (مثلًا منع القيم السالبة)
- الـ getter بيسمحلنا نقرأ القيمة من بره الكلاس

|المتغيرات تكون private|`private int numEggs;`|
|لو نوعه boolean نبدأ ب is|`public boolean isHappy()`|
|لو مش boolean نبدأ ب get|`public int getNumEggs()`|
|الـ setter يبدأ ب set|`public void setHappy(boolean happy)`|

---
### ==**Creating Immutable Classes**== 

 إنشاء كلاسات غير قابلة للتغيير (Immutable Classes) بالعربي

 إيه هي الـ Immutable Class؟

دي كلاسات في الجافا **ماينفعش تتعدل** بعد ما يتم إنشاؤها. أي بيانات يتم تخزينها في الكلاس تبقى ثابتة طول عمر **الكائن**.

 ليه نستخدم Immutable Classes؟

1. **الأمان**: مفيش حد يقدر يغير البيانات من بره
    
2. **الثبات**: الكائن هيبقى زي ما أنشأته طول ما هو موجود
    
3. **الكفاءة**: تقلل عدد النسخ في الذاكرة (زي الـ String في الجافا)
    
4. **سهولة التتبع**: مفيش مفاجآت، البيانات متعرفش تتغير

طريقة الانشاء 
```java 
public final class ImmutableSwan {  // 1. الكلاس لازم يكون final
    private final int numberEggs;    // 2. المتغيرات final
    
    // 3. الكونستركتور لتعيين القيم الأولية
    public ImmutableSwan(int numberEggs) {
        this.numberEggs = 50 + numberEggs;
    }
    
    // 4. فقط getters بدون setters
    public int getNumberEggs() {
        return numberEggs;
    }
}
```
## القواعد الأساسية لإنشاء Immutable Class:

1. **إجعل الكلاس `final`**:
    - عشان ماينفعش يتم توريثه وتغيير سلوكه
2. **إجعل كل المتغيرات `private` و `final`**:
    - عشان متتعدلش بعد الإنشاء
3. **لا تضع أي `setters`**:
    - مفيش طرق لتعديل البيانات بعد الإنشاء
4. **لا تسمح بالتعديل على المتغيرات المرجعية**:
    - لو عندك متغير من نوع object (مثل List)، لازم تتأكد إنه ماينفعش يتعدل
5. **إستخدم الكونستركتور لتعيين القيم الأولية**:
    - القيم بتتحدد عند الإنشاء وبس
    ```java
import java.util.List;
import java.util.Collections;

public final class ImmutableStudent {
    private final String name;
    private final List<String> courses;
    
    public ImmutableStudent(String name, List<String> courses) {
        this.name = name;
        // 5. إنشاء نسخة غير قابلة للتعديل من الـ List
        this.courses = Collections.unmodifiableList(new ArrayList<>(courses));
    }
    public String getName() {
        return name;
    }
    public List<String> getCourses() {
        // إرجاع نسخة غير قابلة للتعديل
        return courses;
    }
}
```
## نصائح مهمة:

1. **لو الكلاس فيه متغيرات مرجعية**:
    - لازم تعمل deep copy عند الإدخال والإخراج
    - أو تستخدم Collections.unmodifiable...    
2. **الـ Records في الجافا 16+**:
    - بتساعد في إنشاء immutable classes بسهولة
3. **الفوائد في الـ multithreading**:
    -ال Immutable objects آمنة تمامًا للاستخدام في بيئات الـ multithreading بدون حاجة لـ synchronization
4. **التحسينات**:
    - ممكن تستخدم الـ Builder Pattern لو عندك كتير من المتغيرات
## الخلاصة
الـ Immutable Classes دي من أهم المفاهيم في الجافا، وبتساعد في:
- كتابة كود أكثر أمانًا
- تقليل الأخطاء الجانبية
- تحسين الأداء
- تسهيل فهم الكود وصيانته
-عند كتابة immutable classes:
1. إحذر من مشاركة references للكائنات القابلة للتغيير
```java 
public class NotImmutable {
    private StringBuilder builder;
    
    public NotImmutable(StringBuilder b) {
        builder = b; // المشكلة هنا!
    }
    
    public StringBuilder getBuilder() {
        return builder; // والمشكلة هنا!
    }
الحل 
public NotImmutable(StringBuilder b) {
    this.builder = new StringBuilder(b); // نسخة جديدة
}
public StringBuilder getBuilder() {
    return new StringBuilder(builder); // نرجع نسخة جديدة
}
}
```
1. إستخدم defensive copying عند الإدخال والإخراج
2. إفضل إرجاع أنواع immutable عندما يكون ذلك ممكناً
3. تأكد أن الكلاس لا يوفر أي طريقة لتعديل حالته الداخلية
---
### ==**Record in java**==

إيه هو الـ Record في الجافا؟
الـ Record ده نوع جديد من الكلاسات جافا ابتداءً من الإصدار 16، وهو مصمم خصيصًا لإنشاء **كلاسات غير قابلة للتغيير (Immutable)** بطريقة مختصرة وسهلة.


```java 
public record Student(String name, List<String> courses) {}
ده هو ده 
public final class Student {
    private final String name;
    private final List<String> courses;
    
    // Constructor
    public Student(String name, List<String> courses) {
        this.name = name;
        this.courses = List.copyOf(courses); // إنشاء نسخة غير قابلة للتعديل
    }
    
    // Getters
    public String name() { return name; }
    public List<String> courses() { return courses; }
    
    // equals, hashCode, toString
    @Override public boolean equals(Object o) { ... }
    @Override public int hashCode() { ... }
    @Override public String toString() { ... }
}
```
 المميزات اللي بيوفرها الـ Record:

1. **إختصار كبير في الكود**: سطر واحد بدل كل الكلاس!
    
2. **أوتوماتيك immutable**: كل الحقول بتكون `private` و`final`
    
3. **مفيش setters**: عشان متقدرش تعدل القيم بعد الإنشاء
    
4. **بيجيبلك methods جاهزة**:
    
    - `equals()` و `hashCode()`
        
    - `toString()`
        
5. **constructor جاهز**: بيكون فيه باراميترات لكل الحقول
---
  مشكلة الـ List في المثال:
```java
Student s = new Student("Ahmed", new ArrayList<>(Arrays.asList("Math", "Science")));

// ممكن حد يعمل كده ويعدل الليست الأصلية
s.courses().add("History"); // هتطلع runtime exception
//الحل
public record Student(String name, List<String> courses) {
    public Student {
        courses = List.copyOf(courses); // هتخلي الـ List غير قابلة للتعديل
    }
}

//لو عايز تضيف ميثودات جديدة 
public record Student(String name, List<String> courses) {
    // ممكن تضيف methods
    public boolean hasCourse(String courseName) {
        return courses.contains(courseName);
    }
}

//ممكن تعدل على الكونستركتور من غير ما تكتبه كامل
public record Student(String name, List<String> courses) {
    public Student {
        Objects.requireNonNull(name, "Name cannot be null");
        courses = List.copyOf(courses);
    }
}
```
 متى تستخدم الـ Record؟
- لما يكون عندك **كلاس بيانات** بس (data carrier)
- لما تكون البيانات **مش محتاجة تتغير** بعد الإنشاء
- لما تكون محتاج implementation بسيط لـ `equals` و `hashCode`
متى ما تستخدمهوش؟
- لو محتاج كلاس قابل للتعديل
- لو محتاج توريث (inheritance)
- لو محتاج تخفيض (encapsulation) معقد

الـ Records دي من أحسن الإضافات الحديثة في الجافا، وبتوفر وقت وجهد كتير في كتابة الكلاسات البسيطة

---

## **==Lambda Expression==**
الـ **Lambda Expression** ببساطة هي طريقة إنك تبعت **كود كأنه متغير**. يعني ممكن تبعت **function بدون ما تكتب class كامل**.

زي ما بتبعت رقم أو نص كـ parameter، دلوقتي تقدر تبعت **كود بينفذ حاجة**.
print(animals, new CheckIfHopper());
نا اضطرينا نعمل class اسمه `CheckIfHopper` بس عشان نرجّع `true` لو الحيوان بينط.
مع lambda، تقدر تستبدله بسطر بسيط:
print(animals, a -> a.canHop());
ده كأنك بتقول: “لو الحيوان بينط، اطبعه”، لكن من غير ما تعمل class جديد.
a -> a.canSwim()
- **a**: هو الباراميتر (الـ Animal)
- **a.canSwim()**: هو الكود اللي بيتنفذ (يرجّع true أو false)
وده اسمه "Deferred Execution"
يعني الكود اللي بتكتبه جوه الـ lambda مش بيتنفذ فورًا، بيتخزن ويتنفذ **وقت ما تستدعيه**، زي لما يحصل:

مميزات الـ Lambda:

1. **مش محتاج تكتب نوع الباراميتر**: الجافا بتفهمه أوتوماتيك (Type Inference)
2. **مش محتاج أقواس لو في باراميتر واحد**: `a ->` بدل `(a) ->`
3. **مش محتاج أكتب `return` لو في سطر واحد**: الجافا بترجعه أوتوماتيك
 مفهوم Deferred Executant:
الـ Lambda بيتنفذ **مش فوراً**، لكن لما نحتاجه. في المثال بتاعنا، الكود بيتنفذ جوا الـ `print()` لما يعدي على كل حيوان.
 الخلاصة:
الـ Lambdas في الجافا:
- بتخليك تكتب كود أوضح وأقصر
- بتساعدك تكتب كود بطريقة functional programming
- بتوفر وقتك من كتابة كلاسات كتير لمهام بسيطة
- بتخلي الكود أسهل في الصيانة والتعديل

---
