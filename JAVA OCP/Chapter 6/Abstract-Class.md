# Abstract Class 
## يعني إيه Abstract Class؟
الـ **Abstract Class** في Java هي كلاس **ماينفعش نعمل منها object**.  
بتتستخدم كـ **كلاس أساس** (Base Class) للكلاسات اللي هتورث منها.

بنكتبها باستخدام الكلمة المفتاحية `abstract`.

```java
abstract class Animal {
    abstract void sound();

    void eat() {
        System.out.println("Animal eats");
    }
}
```

الكلاس دي:
- فيها method مش كاملة (`sound`)
- وفيها method عادية (`eat`)

---

## ليه نستخدم Abstract Class؟
بنستخدم الـ abstract class لما:
- يبقى عندنا فكرة عامة مش مكتملة
- عايزين نفرض على الكلاسات اللي هتورث ميثودز معينة لازم تتنفذ
- وفي نفس الوقت عايزين نحط كود مشترك يتورّث

ببساطة:
> أنا عارف *إيه* اللي لازم يحصل، بس مش عارف *إزاي* يتنفذ لسه

---

## قواعد مهمة جدًا
- ❌ ماينفعش نعمل object من abstract class
- ✔️ ينفع تحتوي abstract methods و methods عادية
- ✔️ ينفع تحتوي fields
- ✔️ ينفع تحتوي constructors
- ✔️ ينفع تورث من abstract class تانية
- ✔️ ينفع تطبق interfaces

```java
Animal a = new Animal(); // ERROR
Animal a = new Dog();    // صح
```

---

## Abstract Methods
الميثود الـ abstract:
- ملهاش body
- لازم تبقى جوه abstract class
- لازم تتعملها override في أول كلاس **مش abstract** (concrete class)

```java
abstract void move();
```

### مينفعش تكون:
- ❌ `private` (الابن مش هيشوفها)
- ❌ `static` (مفيش overriding)
- ❌ `final` (final تمنع الـ override)

---

## الوراثة من Abstract Class
لما كلاس يرث من abstract class:
- لو هو **مش abstract** → لازم يعمل override لكل الـ abstract methods
- لو ماعملش كده → يحصل **Compile-time error**
- لو هو abstract → مش لازم override

```java
class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog barks");
    }
}
```

---

## Constructors في Abstract Class
آه، ينفع abstract class يكون فيها constructor 😎  
بس:
- بيتنادي لما نعمل object من كلاس ابن
- بيستخدم غالبًا لتهيئة fields مشتركة

```java
abstract class Shape {
    int x;

    Shape(int x) {
        this.x = x;
    }
}
```

---

## Polymorphism مع Abstract Class
الـ abstract class بتدعم الـ polymorphism عادي جدًا.

```java
Animal a = new Dog();
a.sound(); // Dog implementation
```

- تنفيذ الميثود بيبقى حسب نوع الـ object
- مش حسب نوع الـ reference

---

## Abstract Class vs Class عادية
| Class عادية | Abstract Class |
|------------|----------------|
| ينفع نعمل object | ❌ |
| كاملة التنفيذ | ممكن تكون ناقصة |
| مفيهاش abstract methods | ينفع |
 
---

## Abstract Class vs Interface (مختصر)
| Abstract Class | Interface |
|---------------|-----------|
| توريث واحد | multiple inheritance |
| ينفع constructors | مفيش |
| ينفع methods عادية | غالبًا abstract |
| ينفع fields | constants بس |

---

## أخطاء شائعة
- ننسى نعمل override للـ abstract method
- نحاول نعمل object من abstract class
- نخلي abstract method `static` أو `final`

---

## الخلاصة
- abstract class = كلاس ناقص
- ماينفعش نعمل منه object
- بيستخدم ككلاس أساس
- بيفرض methods لازم تتنفذ
- ينفع يحوي كود مشترك
- بيدعم polymorphism

