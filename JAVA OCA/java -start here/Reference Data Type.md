# 🧠 Java - Reference Data Types

## ✅ ما هي؟
أنواع بيانات لا تخزن القيمة نفسها، بل تشير إلى كائن (Object) في الذاكرة.

---

## 📦 أمثلة:

- `String name = "Ali";`
- `int[] arr = {1, 2, 3};`
- `MyClass obj = new MyClass();`
- `ArrayList<String> list = new ArrayList<>();`

---

## 🔍 الفرق عن Primitive:

| الخاصية | Primitive | Reference |
|---------|-----------|-----------|
| القيمة | مباشرة | عنوان لكائن |
| التخزين | Stack | Heap |
| السرعة | أسرع | أبطأ |
| مثال | `int`, `float`, `boolean` | `String`, `Array`, `Class` |

---

## 💡 تذكير:
- `String` هو كائن، لكنه بيتعامل كأنه نوع خاص لأنه Immutable
- التغيير في المرجع لا يغير النسخة الأصلية إلا لو الكائن نفسه قابل للتعديل

---

## 🧪 مثال سريع:

```java
String a = "Hello";
String b = a;
a = "World";
System.out.println(b); // Hello
---
```

ما الفرق الأساسي بين **Primitive Type** و **Reference Type** من ناحية التخزين في الذاكرة؟
### ☕ الفرق بين **String Literal** و **String Object** في Java
```
public class Test {
    public static void main(String[] args) {
        String s1 = "Java";
        String s2 = "Java";
        String s3 = new String("Java");

        System.out.println(s1 == s2);      // true → نفس literal
        System.out.println(s1 == s3);      // false → new object
        System.out.println(s1.equals(s3)); // true → نفس القيمة
    }
}
```


|                                  |         |             |                  |     |
| -------------------------------- | ------- | ----------- | ---------------- | --- |
| `String s = "Java";`             | Literal | String Pool | ✅ ممكن يكون true | ✅   |
| `String s = new String("Java");` | Object  | Heap        | ❌ أكيد false     | ✅   |