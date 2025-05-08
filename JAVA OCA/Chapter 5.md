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
