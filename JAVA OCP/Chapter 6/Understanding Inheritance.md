# Chapter 6 — Understanding Inheritance (OCP Java)

## Inheritance Basics
Inheritance يعني إن `class` يرث خواص وسلوك `class` تاني باستخدام `extends`.  
Java بتدعم **Single Inheritance** للكلاسات.  
الهدف من Inheritance:
- إعادة استخدام الكود
- تحقيق Polymorphism
- تقليل التكرار وتحسين الصيانة

---

## Reference vs Object (أهم فكرة)
عند كتابة:
Animal a = new Dog();

- **Reference**: نوع المتغير (`Animal`)
- **Object الحقيقي**: الكائن اللي اتعمل (`Dog`)

Java بتستخدم كل واحد في وقت مختلف.

---

## Compile Time vs Runtime
القواعد الأساسية:
- **Variables** → Compile Time → Reference
- **static methods** → Compile Time → Reference
- **Instance methods** → Runtime → Object
- **Casting** → Runtime → Object

---

## Method Overriding
علشان method تعمل override لازم:
- نفس الاسم
- نفس الـ parameters
- Return type نفس النوع أو subclass (Covariant)
- Access level مينفعش يقل (ينفع يزيد)

ممنوع override:
- final methods
- static methods
- private methods

ملاحظة:  
`static methods` = Hiding مش Overriding

---

## Polymorphism
لو عندك:
Animal a = new Dog();

واستدعيت:
a.speak();

اللي يتنفذ هو method بتاع `Dog`  
لأن **methods بتتحسم وقت التشغيل حسب الـ object الحقيقي**.

---

## Variables & static (Hiding)
لو Parent و Child عندهم variable بنفس الاسم:
- اللي يتاخد هو بتاع **Reference**
- مش بتاع الـ Object

نفس الكلام مع `static methods`.

---

## Casting (أخطر جزء)
Casting يعني تحويل reference من Parent لـ Child.

### Safe Casting
Animal a = new Dog();  
Dog d = (Dog) a; ✔️

### Runtime Error
Animal a = new Animal();  
Dog d = (Dog) a; ❌ ClassCastException

**القاعدة الذهبية**:  
ينفع تعمل cast فقط لو الـ object الحقيقي من النوع ده.

للأمان:
if (a instanceof Dog)

---

## Constructors مع Inheritance
- أول سطر في أي constructor لازم يكون:
  - `super()` أو `this()`
- Constructor الـ Parent دايمًا بيتنفذ الأول
- لو Parent مفيهوش constructor فاضي، لازم تنادي `super(parameters)`

---

## super Keyword
تُستخدم لـ:
- استدعاء constructor من Parent
- استدعاء method من Parent داخل Child

---

## final مع Inheritance
- `final class` → لا يمكن توريثه
- `final method` → لا يمكن عمل override
- `final variable` → لا يمكن تغيير قيمته

---

## القواعد الذهبية (احفظهم)
- Method → Runtime → Object
- Variable → Compile → Reference
- static → Compile → Reference
- Casting → الـ Object الحقيقي هو الحكم
- Parent constructor دايمًا الأول
- Access level مينفعش يقل

---

## مثال شامل للفهم
A a = new B();

- a.x → من A
- a.staticMethod → من A
- a.instanceMethod → من B
- ((B)a).childMethod → من B (طالما الـ object فعلاً B)

---

## الخلاصة
**Java بتهتم إنت عملت `new` إيه  
مش سميت المتغير إيه**
