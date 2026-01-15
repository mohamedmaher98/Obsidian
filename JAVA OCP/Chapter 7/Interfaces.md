# Interfaces in Java – Declaring, Implementing & Using (Advanced)

## يعني إيه Interface؟
Interface في Java تمثل **Contract / Capability**  
بتقول:
- إيه اللي الكلاس يقدر يعمله (what can do)
- مش إيه هو من جوّه (not what is)

---

## Declaring an Interface
```java
interface Flyable {
    void fly();
}
```

- كل methods تلقائيًا `public abstract`
- مفيش constructors
- مفيش state

---

## Implementing an Interface
```java
class Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("Flying");
    }
}
```

قواعد:
- لازم تطبّق كل methods
- الميثود لازم تبقى `public`
- وإلا الكلاس يبقى `abstract`

---

## Multiple Interfaces
```java
class Duck implements Flyable, Swimmable {
    public void fly() {}
    public void swim() {}
}
```

مسموح لأن:
- Interfaces مفيهاش state
- مفيش constructors
- مفيش ambiguity زي classes

---

## Interface extends Interface
```java
interface A {
    void a();
}

interface B {
    void b();
}

interface C extends A, B {
}
```

- Interface تقدر تورّث من أكتر من Interface
- التنفيذ يكون في الكلاس

---

## Default Methods (Java 8+)
```java
interface Logger {
    default void log(String msg) {
        System.out.println(msg);
    }
}
```

تُستخدم لما:
- نضيف behavior جديد
- من غير ما نكسّر implementations قديمة

---

## Static Methods in Interface
```java
interface Utils {
    static void print(String msg) {
        System.out.println(msg);
    }
}
```

- لا تُعمل override
- تُنادى باسم الـ interface:
```java
Utils.print("Hello");
```

---

## Variables in Interface
```java
interface Config {
    int TIMEOUT = 30;
}
```

أي variable في interface هو:
`public static final`

---

## Interface vs Abstract Class (Design)
- Interface → contract / capability
- Abstract Class → identity + shared logic

---

## خلاصة
- Interface = عقد
- Implement = التزام
- Default = behavior اختياري
- Static = helper خاص
- Variables = constants فقط
