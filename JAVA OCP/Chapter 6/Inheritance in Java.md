# Member Inheritance in Java

## الفكرة العامة

ال**Inheritance** في Java معناها إن الكلاس الابن (Subclass) بياخد **Members** من الكلاس الأب (Superclass) باستخدام الكلمة المفتاحية `extends`. الـ Members اللي بتتورّث هي **fields و methods** فقط، أما **constructors لا تُورّث**. الهدف من التوريث هو إعادة استخدام الكود، تقليل التكرار، وتنظيم العلاقات بين الكلاسات.

	`class A {
	     int x;  
	        void show() {} 
	        } 
	         class B extends A {
	              int y; }`

في المثال ده، الكلاس `B` ورث المتغير `x` والميثود `show()` من الكلاس `A`.

---

## الAccess Modifiers والتوريث

التوريث في Java بيختلف حسب **Access Modifier**:

- `public` → يُورّث ويمكن الوصول إليه من أي مكان
    
- `protected` → يُورّث ويمكن الوصول إليه داخل الكلاس الابن
    
- `default` (من غير modifier) → يُورّث **فقط داخل نفس package**
    
- `private` → ❌ لا يُورّث لأن الكلاس الابن لا يراه
    

> ملاحظة: الـ `private members` يمكن الوصول لها بشكل غير مباشر عن طريق `public` أو `protected methods` في الكلاس الأب.

---

## الInheriting Methods و Method Overriding

الميثود الموروثة يمكن استخدامها كما هي، أو إعادة تعريفها في الكلاس الابن باستخدام **method overriding**.  
الـ overriding يعني كتابة ميثود في الكلاس الابن بنفس الاسم ونفس الـ parameters.

**شروط overriding**:

- نفس الاسم ونفس التوقيع (parameters)
    
- نفس أو أوسع access modifier
    
- لا يمكن عمل override لـ `private` أو `final`
    
- لا يمكن عمل override لـ `static methods`
    

`class A {     void test() {} }  class B extends A {     @Override     void test() {} }`

الميثودز تخضع للـ **polymorphism**، يعني التنفيذ بيتم حسب **نوع الـ object وقت التشغيل (runtime)** وليس نوع الـ reference.

---

## الFields مع Inheritance

الـ **fields لا تخضع للـ polymorphism**.  
الوصول للـ field يتم حسب **نوع الـ reference** وليس نوع الـ object.

`A ref = new B(); 
ref.x;   // x الخاصة بـ A`

لو الكلاس الابن عرّف field بنفس اسم الأب، ده يسمى **field hiding**.

---

## super Keyword

الكلمة المفتاحية `super` تُستخدم للوصول لأعضاء الكلاس الأب.

- استدعاء constructor الأب:
    

`super();`

- الوصول لـ field أو method من الأب عند وجود تعارض أسماء:
    

`super.x; super.test();`

> `super()` لازم تكون **أول سطر** داخل constructor الكلاس الابن.

---

## الConstructors و Inheritance

- ❌ Constructors لا تُورّث
    
- ✔️ الConstructor الأب يُستدعى تلقائيًا قبل Constructor الابن
    
- لو الأب عنده constructor بباراميتر، يجب استدعاؤه صراحة باستخدام `super(parameters)`
    
ترتيب التنفيذ:

`Parent → Child`

---

## static Members و Inheritance

الـ `static members` مرتبطة بالكلاس وليس بالـ object، لذلك:

- لا يحدث لها overriding
    
- يحدث لها **method hiding**
    
- يتم تحديدها حسب **نوع الـ reference**
    

`A ref = new B(); ref.staticMethod(); // تنفذ method الخاصة بـ A`

---

## final مع Inheritance

- `الfinal method` → ❌ لا يمكن عمل override
    
- `الfinal class` → ❌ لا يمكن لأي كلاس يرث منها
    
- ال`final variable` → لا يمكن تغيير قيمتها
    

محاولة override لـ `final method` تؤدي إلى **Compile-time error**.

---

## الخلاصة

-ال Java تستخدم `extends` للتوريث
    
-ال `public` و `protected` و `default` members تُورّث
    
-ال `private` members لا تُورّث
    
-ال Constructors لا تُورّث
    
- الMethods تخضع للـ polymorphism
    
- الFields و static methods تعتمد على نوع الـ reference
    
- `الsuper` للوصول لأعضاء الأب
    
- ال`final` تمنع التعديل أو التوريث
 ---
  ---
  