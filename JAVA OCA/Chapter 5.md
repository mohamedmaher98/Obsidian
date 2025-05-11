==Class Design==
### **What is Inheritance?**
- Allows a **child class (subclass)** to inherit **fields and methods** from a **parent class (superclass)**.
- Child classes automatically get:
    - All `public` and `protected` members (methods, variables).
    - Default (package-private) members if in the same package.

Types of inheritance :
a- single inheritance 
	a class can extend only one parent class.
b-Multilevel inheritance 
	a child class can further be inherited by another class 
```java
class Animal { }
class Mammal extends Animal { }
class Dog extends Mammal { } // Dog → Mammal → Animal
```
c- no multiple inheritance (unlike c++)
	java dos not allow multiple inheritance for the class to avoid complexity
```java
class A { }
class B { }
class C extends A, B { } // ERROR: Java doesn't support this!
```
d-  **Multiple Inheritance via Interfaces**
- A class can implement **multiple interfaces** (Java's workaround).
```java
interface Swimmable { }
interface Walkable { }
class Duck implements Swimmable, Walkable { } // Valid
```
---
 **The `final` Keyword**
 - Prevents a class from being inherited.
 **Why Java Avoids Multiple Inheritance?**
 **Ambiguity (Diamond Problem)**: If two parents have the same method, which one should the child inherit?
 ```java
 class A { void print() { System.out.println("A"); } }
class B { void print() { System.out.println("B"); } }
class C extends A, B { } // Which print() should C use? Java avoids this!
```
solution
Use **interfaces** (since they don’t contain implementation until Java 8 `default` methods).

---
كل كلاس بتكتبه في Java، سواء كتبته بـ `extends` أو لأ، هو في الحقيقة بيورث (implicitly) من `java.lang.Object`.
لأن `Object` بيحتوي على شوية دوال أساسية موجودة في كل كلاس في Java، زي:
toString()	لتحويل الكائن إلى String
equals(Object obj)	للمقارنة بين كائنين
hashCode()	يرجّع رقم يستخدم في الـ hashing
getClass()	يرجّع كلاس الكائن
clone()	يعمل نسخة من الكائن (لو الكلاس بيدعمها)
finalize()	(قديم – لما الجاربيج كوليكتور يخلص على الكائن

---
### **==Defining Constructor==** 
كل كلاس لازم يكون عنده على الأقل Constructor واحد.
- لو مكتبتش Constructor، الكومبايلر هيضيف **default constructor** تلقائيًا:
 في حالة الوراثة:
- **أول سطر في أي Constructor** لازم يكون:
    - إما `this(...)` لنداء Constructor تاني داخل نفس الكلاس.
    - أو `super(...)` لنداء Constructor في الكلاس الأب.
      ولازم يكونوا اول سطر في الConstructor 

---
### ==**Understanding Compiler Enhancements**==
تحسينات الكومبايلر عند تعريف الـ Constructors
1. الكومبايلر بيضيف `super()` تلقائيًا
لو إنت ما كتبتش أي constructor في الكلاس، أو كتبت constructor بدون `super()`، الكومبايلر هيضيف:
```java
public class Donkey {
    public Donkey() {
        super(); // ← يتم إضافتها تلقائيًا
    }
}
//طالما الأب فيه no-argument constructor، الدنيا تمام.

```
لما الأب **مافيهوش no-arg constructor**
```java
public class Mammal {
    public Mammal(int age) { }
}

public class Elephant extends Mammal { } // ❌ Error!
```
لازم نضيف constructor بنفسنا، ونديله القيمة اللي الأب محتاجها:
```java
public class Elephant extends Mammal {
    public Elephant() {
        super(10); // ✅ بنستدعي الـ constructor الصح
    }
}```
قواعد مهمة في الكونستركتور فالوراثة

1️⃣	أول سطر في أي constructor لازم يكون super(...) أو this(...)
2️⃣	ما ينفعش تستخدم super() بعد أول سطر
3️⃣	لو ما كتبتش super()، الكومبايلر هيضيف super() تلقائيًا
4️⃣	لو الأب مافيهوش no-arg constructor، والابن مافيهوش constructor، الكود مش هيكمل
5️⃣	لازم تستدعي constructor موجود في الأب بشكل صريح لو الأب مافيهوش no-arg

---
استدعاء الـ Constructors
- لكمبايلر بيضيف `super()` لو مش مكتوبة، بشرط إن فيه constructor بدون باراميتر في الأب.
- التنفيذ بيبدأ من الأعلى في شجرة الوراثة وينزل لتحت
    - الأب الأول  
    - ثم الأب الثاني
    - وهكذا
    - وأخيرًا الابن
**الوصول لأعضاء الأب**
- تقدر توصل لأي عضو في الأب لو كان:
    - `public`
    - `protected`
    - `default` لو في نفس الـ package
- مينفعش توصل لأي عضو `private` مباشرة، لكن ممكن توصله بـ `public` أو `protected` method.
- عندك 3 طرق لكتابة الوصول:
    
    - `this.variable`
    - `super.variable`
    - `variable` (الاسم مباشرة)
---
 الفرق بين `this` و `super` في Java

| الاستخدام              | `this`                            | `super`                             |
|------------------------|-----------------------------------|-------------------------------------|
| يشير إلى               | الكائن الحالي                     | الأب المباشر للكائن الحالي          |
| يصل إلى                | كل أعضاء الكلاس الحالي             | فقط الأعضاء اللي وُرثت من الأب       |
| يستخدم مع              | المتغيرات، الدوال، constructors   | المتغيرات، الدوال، constructors     |
| الوصول لمتغير خاص بالابن | ✅ متاح                           | ❌ غير متاح                          |
- الفرق مابين super , super()

| العنصر            | `super()`                                         | `super`                                        |
| ----------------- | ------------------------------------------------- | ---------------------------------------------- |
| نوعه              | استدعاء (Call)                                    | كلمة محجوزة (Keyword)                          |
| وظيفته            | يستدعي **constructor** من الكلاس الأب             | يستخدم للوصول إلى **متحوّلات أو دوال** من الأب |
| مكان الاستخدام    | فقط في **أول سطر** في constructor في الكلاس الابن | يمكن استخدامه في أي مكان داخل الكلاس           |
| هل يحتاج أقواس () | نعم، لأنها عملية استدعاء constructor              | لا، لأنه يشير إلى أعضاء الأب                   |
| مثال صالح         | `super();`                                        | `super.getAge();`                              |
| مثال غير صالح     | `super;` ❌ أو `super().getAge();` ❌               | -                                              |

---
# ==**Overriding Methods in Java**==

**1. What is Method Overriding?**

- **التعريف**:  
    عملية إعادة تعريف ميثود موجودة في الكلاس الأب (Parent Class) في الكلاس الابن (Child Class) **بنفس الاسم والتوقيع (Signature)**.
- **الغرض**:  
    تعديل أو تحسين سلوك الميثود في الكلاس الابن مع الحفاظ على اسمها ووظيفتها الأساسية.
**How to Override a Method?**
 **الشروط الأساسية:**
1. **نفس الاسم (Name)**.
2. **نفس الباراميترات (Parameters)**.
3. **نفس نوع الإرجاع (Return Type)** أو نوع فرعي (Covariant Return Type).
4. **مستوى الوصول (Access Modifier)** مساوي أو أكثر وصولاً (مثال: `protected` في الأب → `public` في الابن).
5. **لا ترمي استثناءات (Exceptions)** جديدة أو أوسع من ميثود الأب.
**** rاستخدام `super` لاستدعاء ميثود الأب****

ذا أردت تنفيذ **جزء من منطق الأب** قبل أو بعد التعديل في الابن
```java
class Wolf extends Canine {
    @Override
    public double getAverageWeight() {
        return super.getAverageWeight() + 20; // استدعاء ميثود الأب أولاً
    }
}
```
مقارنة بين Overriding و Overloading

| المعيار            | **Overriding**                               | **Overloading**                                 |
|--------------------|---------------------------------------------|------------------------------------------------|
| العلاقة بين الميثودات | بين الأب والابن                              | في نفس الكلاس                                  |
| التوقيع            | نفس الاسم والباراميترات                    | نفس الاسم، باراميترات مختلفة                  |
| نوع الإرجاع        | نفس النوع أو subtype                        | يمكن أن يكون مختلفاً                          |
| مستوى الوصول       | مساوي أو أكثر وصولاً                         | أي مستوى وصول                                  |
| الاستثناءات        | لا يمكن توسيعها                              | يمكن تغييرها                                   |
**Covariant Return Types (أنواع الإرجاع المتغيرة**
```java
class Animal {
    public Animal reproduce() { return new Animal(); }
}

class Dog extends Animal {
    @Override
    public Dog reproduce() { // إرجاع Dog بدلاً من Animal
        return new Dog();
    }
}
```
**ملخص أهم النقاط**

1. Overriding يتطلب **تطابق التوقيع**.
2. استخدم `super.methodName()` لاستدعاء ميثود الأب.
    
3. تجنب **Recursive Calls** (لا تحذف `super` عند الحاجة لاستدعاء الأب).
4. `@Override` يساعد في اكتشاف الأخطاء مبكراً
5. يمكن تغيير نوع الإرجاع إذا كان **subtype** (Covariant Return Types).


**ملخص النقاط الرئيسية**
1. التجاوز يتطلب **تطابق التوقيع** تمامًا
2. استخدام `super.اسم_الدالة()` لاستدعاء دالة الأب
3. تجنب **الاستدعاء الذاتي اللانهائي**
4. استخدام `@Override` للتحقق من صحة التجاوز
5. إمكانية تغيير نوع الإرجاع إذا كان **فرعيًا**
---
```java
What is the output of the following code? (Choose all that apply)
1: interface HasTail { int getTailLength(); }
2: abstract class Puma implements HasTail {
3: protected int getTailLength() {return 4;}
4: }
5: public class Cougar extends Puma {
6: public static void main(String[] args) {
7: Puma puma = new Puma();
8: System.out.println(puma.getTailLength());
9: }
10:
11: public int getTailLength(int length) {return 2;}
12: }
```
---
## 🧠 Redeclaring Private Methods in Java

### ✅ المفهوم الأساسي:

- **الـ private methods لا يمكن عمل override لها** في الكلاس الابن لأنها غير مرئية أصلاً خارج الكلاس الأب.
    
- لكن تقدر تعمل **redeclare** (إعادة تعريف) لطريقة بنفس الاسم (أو مختلفة في التوقيع) في الكلاس الابن.

### الفرق بين Override و Redeclare:

|الحالة|الوصف|
|---|---|
|`public/protected` method|لازم تتبع قواعد **overriding** (نفس التوقيع، نفس أو أضيق access level، إلخ).|
|`private` method|لا يمكن عمل override، لكن ممكن تعمل **method جديدة** بنفس الاسم|

لما تعمل method بنفس اسم method خاصة في الكلاس الأب:

- دي تعتبر method **مستقلة تمامًا**.
    
- **مش override**، وبالتالي مفيش أي قواعد من قواعد الـ overriding بتطبق.
    
- حتى لو غيرت الـ return type، الكود هيشتغل عادي.

مثال : 
```java 
public class Camel {
    private String getNumberOfHumps() {
        return "Undefined";
    }
}

public class BactrianCamel extends Camel {
    private int getNumberOfHumps() {
        return 2;
    }
}

```
- الـ method في `BactrianCamel` **مش مرتبطة** باللي في `Camel`.
    
	- لأن `getNumberOfHumps()` في `Camel` خاصة (private)، مش ممكن تشوفها أو تورثها.
---
## 🛡️ Hiding Static Methods in Java

### ✅ المفهوم الأساسي
- **إخفاء (hiding)** يحدث عندما يعرّف الكلاس الابن **static method** بنفس الاسم والتوقيع (signature) الموجودين في الكلاس الأب.
- يختلف عن **overriding** لأن both methods تكون static، ولا تدخل قواعد الـ overriding (الذي يخص instance methods).

---

### 📋 خمس قواعد لإخفاء static methods
1. **نفس التوقيع**  
   Method في الكلاس الابن لازم يكون لها نفس الاسم والقائمة من المعاملات (parameters) زي الأب.

2. **Accessibility**  
   Method في الابن لازم تكون على نفس مستوى الوصول أو أكثر اتساعًا (public ≥ protected ≥ default ≥ private) من الأب.

3. **استثناءات (Exceptions)**  
   لا يجوز أن ترمي Method في الابن checked exception جديد أو أوسع من اللي يرمى في الأب.

4. **Covariant Return**  
   لو في return value، لازم تكون نفس النوع أو subtype (لأنها static) — القاعدة نفسها للـ overriding.

5. **static keyword**  
   - لو الأب عامل method على إنها `static`، الابن **لازم** يعرّف method بنفس الـ `static`.  
   - ولو الأب **مش** عامل method `static`، الابن **لازم** يسيبها instance method (بدون `static`).

---

### 🧪 مثال صحيح (Compiles & Runs)
```java
public class Bear {
    public static void eat() {
        System.out.println("Bear is eating");
    }
}

public class Panda extends Bear {
    public static void eat() {
        System.out.println("Panda bear is chewing");
    }
    public static void main(String[] args) {
        Panda.eat(); // Panda bear is chewing
    }
}

//wrong ex:

public class Bear {
    public static void sneeze() { … }
    public void hibernate() { … }
}

public class Panda extends Bear {
    public void sneeze() { // ERROR: should be static to hide
        System.out.println("…");
    }
    public static void hibernate() { // ERROR: should be instance to override
        System.out.println("…");
    }
}
```
- س**sneeze()** في الأب static، لكن الابن معرفها instance ⇒ خطأ.
    
- س**hibernate()** في الأب instance، لكن الابن معرفها static ⇒ خطأ.

> تجنب إخفاء static methods في الكود الحقيقي، لأنها بتخلق التباس وصعوبة في القراءة.  
> إللي بيجهزوا للـ OCP exam ممكن يقابلوها، لكن في الشغل الأفضل تلتزم بأساليب واضحة وبسيطة.


----
##  ==Overriding vs. Hiding Methods in Java==

### ✅ المفهوم الأساسي
| العنصر              | Overriding                           | Hiding (للـ static methods)              |
|---------------------|---------------------------------------|------------------------------------------|
| نوع الميثود         | Instance method                      | Static method                            |
| أي ميثود يتم استدعاؤها؟ | حسب **نوع الكائن (object type)**        | حسب **نوع المرجع (reference type)**       |
| يحصل في وقت؟        | Runtime (ديناميكي - Polymorphism)    | Compile-time (ثابت)                     |
| إمكانية الوصول للأب؟ | ممكن استخدام `super.method()`         | ممكن استخدام `ParentClass.method()`      |

---

### 🧪 مثال على **Hiding**:
```java
public class Marsupial {
    public static boolean isBiped() {
        return false;
    }
    public void getMarsupialDescription() {
        System.out.println("Marsupial walks on two legs: " + isBiped());
    }
}

public class Kangaroo extends Marsupial {
    public static boolean isBiped() {
        return true;
    }
    public void getKangarooDescription() {
        System.out.println("Kangaroo hops on two legs: " + isBiped());
    }

    public static void main(String[] args) {
        Kangaroo joey = new Kangaroo();
        joey.getMarsupialDescription();   // false
        joey.getKangarooDescription();    // true
    }
}
```
#### 🧠 الملاحظات:

- `getMarsupialDescription()` بتستدعي النسخة من `Marsupial.isBiped()` لأنها تابعة للكلاس الأب.
    
- `getKangarooDescription()` بتستدعي نسخة `Kangaroo.isBiped()` لأنها تابعة للكلاس الابن.
    
- كل نسخة من `isBiped()` مستقلة عن التانية — **no polymorphism**.
```java
class Marsupial {
    public boolean isBiped() {
        return false;
    }
    public void getMarsupialDescription() {
        System.out.println("Marsupial walks on two legs: " + isBiped());
    }
}

public class Kangaroo extends Marsupial {
    public boolean isBiped() {
        return true;
    }
    public void getKangarooDescription() {
        System.out.println("Kangaroo hops on two legs: " + isBiped());
    }

    public static void main(String[] args) {
        Kangaroo joey = new Kangaroo();
        joey.getMarsupialDescription();   // true
        joey.getKangarooDescription();    // true
    }
}
```
- هنا حصل **polymorphism** لأن `isBiped()` تم **overriding**.
    
- حتى لما `getMarsupialDescription()` (اللي في الأب) بتستدعي `isBiped()`، بيتم تنفيذ نسخة الابن.
---
##  ==Final Methods in Java==

### ✅ القاعدة الأساسية:
ال`final` methods لا يمكن **تجاوزها (override)** في الكلاس الابن، سواء كانت:
- Instance methods
- أو Static methods (لا يمكن حتى إخفاؤها - hiding)

---

### 🧪 مثال:
```java
public class Bird {
    public final boolean hasFeathers() {
        return true;
    }
}

public class Penguin extends Bird {
    public final boolean hasFeathers() { // ❌ DOES NOT COMPILE
        return false;
    }
}
```
#### ❗ النتيجة:

- الكود لا يتم تجميعه (Compile Error) لأن `hasFeathers()` في الأب معلّمة بـ `final`.
    
- كتابة `final` مرة تانية في الكلاس الابن **لا تُغير شيء** — لسه ممنوع override.
    

---

### ❓ لماذا نستخدم final في الميثود؟

- لضمان سلوك معين لا يمكن تغييره بواسطة الكلاسات الابنة.
    
- حماية الـ API من التعديل في أماكن غير مرغوب فيها.
    
- تحسين الأداء في بعض الحالات، حيث يمكن للمترجم تحسين تنفيذ الميثود لأنه يعلم أنها لن تتغير.
    

---

### ⚠️ تنبيه عملي:

> لا تستخدم `final` على الميثودات إلا إذا كنت **واثق 100%** إن السلوك يجب ألا يتغير، لأن ده بيقلل من مرونة الكود.

|العنصر|التأثير عند استخدام `final`|
|---|---|
|Instance method|لا يمكن عمل override في الكلاس الابن|
|Static method|لا يمكن عمل hiding في الكلاس الابن|
|مفيد في|ضمان سلوك ثابت للميثود في كل الكلاسات|
|يُستخدم بحذر؟|✅ نعم – لا تستخدمه إلا عند الضرورة|

---
##  ==Inheriting & Hiding Variables in Java==

### 📌 القاعدة الأساسية:
- **الـ variables لا يمكن عمل override لها** في Java.
- لكن يمكن **إخفاؤها (hide)** في الكلاس الابن لو لها نفس الاسم.
- بيتم إنشاء **نسختين** من المتغير: واحدة في الأب وواحدة في الابن.

---

### 🧪 مثال عملي:
```java
public class Rodent {
    protected int tailLength = 4;

    public void getRodentDetails() {
        System.out.println("[parentTail=" + tailLength + "]");
    }
}

public class Mouse extends Rodent {
    protected int tailLength = 8;

    public void getMouseDetails() {
        System.out.println("[tail=" + tailLength + ", parentTail=" + super.tailLength + "]");
    }

    public static void main(String[] args) {
        Mouse mouse = new Mouse();
        mouse.getRodentDetails();
        mouse.getMouseDetails();
    }
}
[parentTail=4]
[tail=8, parentTail=4]
```
لأان - **الـ variables لا يمكن عمل override لها** في Java.
### 🧠 ملحوظات مهمة:

- استخدام `super.variableName` بيرجع قيمة المتغير في الكلاس الأب.
    
- الوصول للمتغير بيعتمد على **نوع الريفرنس**، مش نوع الكائن نفسه.
- **إخفاء المتغيرات (Variable Hiding)** يعتبر ممارسة سيئة جدًا (Bad Practice)!
- - بيخلي الكود صعب القراءة.
- ممكن يسبب أخطاء منطقية لما تستخدم الريفرنس من نوع الأب.
مثال
```java
public class Animal {
    public int length = 2;
}

public class Jellyfish extends Animal {
    public int length = 5;

    public static void main(String[] args) {
        Jellyfish jellyfish = new Jellyfish();
        Animal animal = new Jellyfish();

        System.out.println(jellyfish.length); // 5
        System.out.println(animal.length);    // 2 ❗
    }
}

```
- لا تكرر نفس اسم المتغير الموجود في الكلاس الأب داخل الكلاس الابن، **إلا إذا كان `private`**.
    
- حاول دائمًا استخدام أسماء مختلفة لتجنب الالتباس.

| العنصر            | القاعدة                                |
| ----------------- | -------------------------------------- |
| Instance variable | يتم إخفاؤه فقط (Hide)                  |
| Static variable   | يتم إخفاؤه فقط (Hide)                  |
| الوصول للمتغير    | يعتمد على نوع الريفرنس وليس نوع الكائن |
| best practice     | لا تستخدم نفس الاسم للمتغير في الابن   |

---
## ==Abstract Classes and Methods in Java==

---

### 🔑 تعريف:
- ال`abstract class`: كلاس لا يمكن إنشاء كائن منه (No instantiation).
-ال `abstract method`: دالة بدون body، يجب تنفيذها (override) في الكلاس الابن.

---

### 🧪 مثال عملي:
```java
public abstract class Animal {
    protected int age;

    public void eat() {
        System.out.println("Animal is eating");
    }

    public abstract String getName(); // no body
}

public class Swan extends Animal {
    public String getName() {
        return "Swan";
    }
}~```
### ✅ قواعد مهمة:

| القاعدة                                           | الشرح                        |
| ------------------------------------------------- | ---------------------------- |
| ✅ كلاس abstract ممكن يحتوي على methods عادية      | مثل `eat()`                  |
| ✅ ممكن ما يكونش فيه أي abstract method            | عادي!                        |
| ❌ لا يمكن إنشاء كائن من abstract class            | `new Animal()` ❌             |
| ❌ال abstract method لازم تكون داخل abstract class | مش في كلاس عادي              |
| ❌ال abstract method ما ينفعش يكون ليها body       | `{}` ❌                       |
| ❌ال abstract class ما ينفعش يكون `final`          | لأنك لازم تورثه              |
| ال❌ abstract method ما ينفعش تكون `final`         | عشان لازم تتعمل لها override |
| ❌ال abstract method ما ينفعش تكون `private`       | الكلاس الابن مش هيقدر يشوفها |

### 💡 نصيحة:

- استخدم `abstract class` لما تحب توفر **سلوك مشترك** مع إجبار الأبناء على تنفيذ وظائف معينة.
    
- لو كل الوظائف المفروض يطبقها الابن فقط، فكر في **interface** بدلًا من abstract class.

---
### ✅ ما هو الـ `abstract` في Java؟

كلمة `abstract` في Java ممكن تتكتب على:

1. **الكلاسات (classes)**: علشان نقول إن الكلاس ده مش ينفع نعمل منه كائن (object) مباشرة.
    
2. **الدوال (methods)**: علشان نقول إن الدالة دي لازم يتم تنفيذها (implement) في الكلاس اللي يرث (extends) الكلاس الأب.
---
### 🧱 أولاً: الـ Abstract Class

#### ✅ تعريف:

كلاس بيتم استخدامه كأساس (template) لكلاسات تانية. الكلاس ده ممكن يحتوي على:

- دوال عادية (مكتملة).
    
- دوال abstract (غير مكتملة).
    
- متغيرات.

❌ مش ممكن تعمل منه object مباشرة:
```java
abstract class Animal {
    public void eat() {
        System.out.println("Animal is eating");
    }
    public abstract String getName(); // لازم يتنفذ في كلاس الوريث
}

```
✅ الوريث لازم ينفذ كل الدوال الـ abstract:
```java
class Dog extends Animal {
    public String getName() {
        return "Dog";
    }
}
```
### 🧠 ثانياً: الـ Abstract Method

### ✅ تعريف:

دالة ماعندهاش جسم (body) ولازم أي كلاس يرث الكلاس الأب ينفذها.
public abstract class Animal {
    public abstract void makeSound(); // مفيش {} لأن مفيش جسم
}
****
✅ لازم تتنفذ في كلاس غير abstract:
public class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow");
    }
}

| الحالة                                     | النتيجة |
| ------------------------------------------ | ------- |
| الabstract class فيه abstract method؟      | ✅ عادي  |
| الabstract class من غير abstract method؟   | ✅ عادي  |
| الmethod abstract في class عادي؟           | ❌ Error |
| الmethod abstract وعندها body؟             | ❌ Error |
| الmethod abstract ومعاها final أو private؟ | ❌ Error |
| الclass abstract ومعاها final؟             | ❌ Error |

---
### 🧭 الفرق بين abstract و interface؟

|المقارنة|abstract class|interface|
|---|---|---|
|هل فيه متغيرات؟|✅ ممكن|✅ لكن لازم يكونوا ثابتين (final static)|
|هل فيه دوال عادية؟|✅ ممكن|✅ من Java 8 باستخدام default|
|هل ينفع يرث كلاس تاني؟|❌ بس واحد|✅ تقدر تورث أكتر من interface|
### 💡 ليه نستخدم abstract؟

- علشان تعمل تصميم عام لكلاسات شبه بعض.
- تجبر المطورين اللي هيستخدموا الكلاس إنهم يكتبوا دوال معينة.
- تعزز من **مبدأ البرمجة بالكائنات** OOP وخصوصًا **الوراثة** و**التعدد الشكلي (Polymorphism)**.

|العنصر|abstract class|interface|
|---|---|---|
|يمكن يحتوي على|متغيرات + دوال عادية + abstract|دوال abstract فقط (Java 8+: default & static)|
|يرث منه كلاس واحد فقط|✅ نعم|❌ لا|
|يطبق منه أكتر من واحد|❌ لا|✅ نعم|
|فيه constructor؟|✅ نعم|❌ لا|
|ينفع يحتوي على body؟|✅ في الدوال العادية|✅ من Java 8 في default methods|
|الكلمة المفتاحية|`abstract class ClassName`|`interface InterfaceName`|

---
### ✅ **أولاً: استخدام `abstract class` (كلاس مجرد)**

تستخدم `abstract class` لما تكون بتوصف **كائنات حقيقية تشترك في صفات وسلوك أساسي مشترك**، وعايز توفّر **سلوك افتراضي (default behavior)** مع إمكانية تغييره.

#### ✅ استخدم `abstract class` لما:

1. ✅ الكلاسات المشاركة **لها علاقة واضحة "is-a"** (مثلاً: Dog is an Animal).
    
2. ✅ عايز توفّر **كود مشترك** بين الكلاسات (مثلاً: دالة `eat()` لكل الحيوانات).
    
3. ✅ محتاج تكتب **constructor** لتجهيز بيانات مشترك.
    
4. ✅ فيه **متغيرات حالة (instance variables)** مشتركة بين الكلاسات.
    
5. ❌ مش محتاج الوراثة المتعددة (لأن الكلاس بيورث من كلاس واحد بس).
    

### مثال نظري:

لو عندك كلاس `Vehicle`، وكل المركبات بتتحرك وتستهلك وقود، يبقى `Vehicle` تبقى `abstract class`، وتخلي `Car` و`Truck` يورثوا منها.

### ✅ **ثانيًا: استخدام `interface` (واجهة)**

تستخدم `interface` لما تكون بتوصف **سلوك أو قدرات مجردة يمكن إضافتها لأي كائن**، حتى لو مفيش علاقة "is-a" واضحة.

### ✅ استخدم `interface` لما:

1. ✅ بتوصف **قدرة/وظيفة** (مثل: `Flyable`, `Serializable`, `Movable`) مش كائن.
    
2. ✅ محتاج **تطبق الوراثة المتعددة** (كلاس واحد يطبق أكتر من `interface`).
    
3. ✅ عايز تفصل بين **ما يجب فعله** و**كيفية تنفيذه**.
    
4. ❌ مش محتاج تخزن حالات أو بيانات داخلية.
    

### مثال نظري:

لو عندك كائنات مختلفة (طائر، طيارة، حشرة) وكلهم يقدروا يطيروا، تقدر تعمل `interface Flyable` وتخليهم كلهم يطبقوه، حتى لو مفيش علاقة بينهم مباشرة.

|الحالة|استخدم `abstract class`|استخدم `interface`|
|---|---|---|
|فيه كود مشترك بين الكلاسات؟|✅ نعم|❌ غالبًا لأ|
|عايز وراثة متعددة؟|❌ لا|✅ نعم|
|الكائنات ليها علاقة وراثية "is-a"?|✅ نعم|❌ مش شرط|
|بتوصف وظيفة/سلوك مجرد؟|❌ لأ غالبًا|✅ نعم|
|عايز تستخدم constructor؟|✅ نعم|❌ لا|
|فيه متغيرات حالة؟|✅ ممكن|❌ لأ|-

---
