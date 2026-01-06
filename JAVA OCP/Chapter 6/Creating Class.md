# Creating Classes — Java (Interview + OCP)

## Class vs Object
- ال**Class** هو Template / Blueprint بنعرّف فيه الـ fields والـ methods.
- ال**Object** هو instance من الـ class، وكل object ليه نسخته الخاصة من الـ instance variables.

---

## Components of a Class
أي class ممكن يحتوي على:
- Fields (variables)
- Methods
- Constructors
- static / instance blocks

---

## Constructor
- اسمه لازم يكون نفس اسم الكلاس
- **مالوش return type**
- بيتنادي تلقائيًا مع `new`
- مينفعش يكون `static`
- لو كتبت أي constructor → Java **مش** بتعمل default constructor

---

## Constructor Overloading
- ينفع overload constructor
- يعتمد على:
  - عدد الـ parameters
  - نوعهم
  - ترتيبهم
- ❌ return type مش جزء من الـ overloading

---

## this Keyword
استخدامات `this`:
- تمييز field من parameter  
  `this.name = name;`
- استدعاء constructor تاني  
  `this(10);`
- الإشارة للـ object الحالي

---

## Access Modifiers
- `private` → داخل نفس الكلاس فقط
- `default` → داخل نفس package
- `protected` → نفس package + subclasses
- `public` → أي مكان

قاعدة مهمة:
> الmember مينفعش يبقى أوسع من الـ class نفسه

---

## static vs instance
- **static**
  - بتتشارك بين كل الـ objects
  - بتتنادي باسم الكلاس
- **instance**
  - لكل object نسخة خاصة

تغيير static variable من أي object → يتغير عند الكل.

---

## Initialization Order
الترتيب الصحيح:
1. static variables / static blocks
2. instance variables
3. instance blocks
4. constructor

---

## final مع Creating Classes
- ال`final variable` → لازم تتعمل initialize (عند التعريف أو في constructor)
- `الfinal method` → مينفعش override
- `الfinal class` → مينفعش يتورث

---

## Traps مهمة في Interviews
- الConstructor ≠ Method
- وجود return type → method مش constructor
- الstatic methods → Hiding مش Overriding
- الdefault constructor بيتشال لو كتبت واحد
- الfinal variable لازم تتعمل initialize

---

## مثال Interview شامل
```java
class A {
    static int x = 1;
    int y = 2;
}

A a1 = new A();
A a2 = new A();

```


---
