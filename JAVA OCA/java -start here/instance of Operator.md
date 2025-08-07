

## 📝 تعريف بسيط

الـ `instanceof` هو **operator** في Java بيستخدم علشان تعرف إذا كان **كائن معين (object)** من نوع (class أو interface) معين.

يعني بتسأل: "هل الكائن ده تابع للفئة دي؟"

بيرجع: `true` أو `false`

---

## 📌 الصيغة العامة

```java
object instanceof ClassName
```

لو `object` فعلاً من النوع `ClassName` أو من subclass ليها → يرجع `true`

---

## 🧪 مثال بسيط

```java
String name = "Ali";
System.out.println(name instanceof String); // true
```

```java
Object obj = new String("hello");
System.out.println(obj instanceof String); // true
System.out.println(obj instanceof Object); // true
System.out.println(obj instanceof Integer); // false
```

---

## ✅ مثال مع كلاس مخصص

```java
class Animal {}
class Dog extends Animal {}

public class Test {
    public static void main(String[] args) {
        Animal a = new Dog();
        System.out.println(a instanceof Dog);    // true
        System.out.println(a instanceof Animal); // true
    }
}
```

---

## ⚠️ ملاحظات مهمة

- مفيد جدًا في **التحقق قبل الكاست (casting)**:
    

```java
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}
```

- لو `obj` = null → النتيجة دايمًا `false`.
    

```java
String s = null;
System.out.println(s instanceof String); // false
```

---

## 🔗 نوتات مرتبطة

- [[Type Casting (Explicit vs Implicit)]]
    
- [[Classes and Objects]]
    
- [[Polymorphism]]