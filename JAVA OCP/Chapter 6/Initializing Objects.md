
## يعني إيه Initializing Object؟
الInitialization هو ترتيب إعطاء القيم للـ object عند إنشاءه باستخدام `new`.
بيحدد:
- إمتى المتغيرات تاخد قيم
- إمتى الـ blocks تتنفذ
- إمتى الـ constructor يشتغل
- وإيه اللي يحصل الأول

---

## مكونات Initialization
أي object في Java بيمر بالمراحل دي:
1. Default values
2. Field initializers
3. Instance initializer blocks
4. Constructor

---

## Default Values
أول ما Java تخصص memory:
- int → 0
- double → 0.0
- boolean → false
- reference → null

```java
class A {
    int x;
    String s;
}

```

### Field Initializers

class A {
    int x = 5;
    String s = "Hi";
}

## Instance Initializer Block

Block بيتنفذ مع كل `new` وقبل الـ constructor.
```java
class A {
    {
        System.out.print("Block ");
    }
}

```

### Constructor
آخر خطوة في initialization.

```java
class A {
    A() {
        System.out.print("Constructor ");
    }
}

```

---
## الترتيب داخل نفس الكلاس

عند إنشاء object:

1. Field initializers
    
2. Instance initializer blocks
    
3. Constructor
    

---

## Initialization مع Inheritance
عند `new Child()`:

### Class Loading (مرة واحدة):

1. Parent static fields / blocks
    
2. Child static fields / blocks
    

### Object Creation (كل مرة):

3. Parent instance fields
    
4. Parent instance blocks
    
5. Parent constructor
    
6. Child instance fields
    
7. Child instance blocks
    
8. Child constructor
---
## static vs instance

- static:
    
    - يتنفذ وقت تحميل الكلاس
        
    - مرة واحدة
        
    - مش جزء من object initialization
        
- instance:
    
    - يتنفذ مع كل `new`
        
    - جزء من object lifecycle

---
## static Field Initializer
static field initializer يعتبر static initialization.

---
مثال شامل 
```java
class A {
    static { System.out.print("A0 "); }
    { System.out.print("A1 "); }
    A() { System.out.print("A2 "); }
}

class B extends A {
    static { System.out.print("B0 "); }
    { System.out.print("B1 "); }
    B() { System.out.print("B2 "); }
}

new B();

```

---
### الناتج:

A0 B0 A1 A2 B1 B2

---
## القاعدة الذهبية

Class Loading:  
static Parent → static Child

Object Creation:  
instance Parent → constructor Parent → instance Child → constructor Child

---
