الـ **Text Block** بيسمحلك تكتب String على أكتر من سطر **من غير ما تحتاج تعمل Escaping** لعلامات الاقتباس (`\"`) أو السطر الجديد (`\n`).

### Syntax

بتستخدم ثلاثة علامات اقتباس مزدوجة متتالية لتبدأ وتنهي الـ Block: """

```java
String query = """
    SELECT
        id, name
    FROM
        users
    WHERE
        status = 'ACTIVE';
""";
```

### قواعد الـ Formatting (Incidental Whitespace)

أهم نقطة في الـ Text Blocks هي إن الـ JVM بتهتم تلقائياً بتنظيف الـ **Whitespace** عشان الكود يكون نظيف. بنسمي المسافات اللي بتضيفها عشان تنسق الكود بتاعك **Incidental Whitespace**.

- الـ JVM بتمسح الـ **Incidental Whitespace** دي تلقائياً من بداية كل سطر عشان الكود النهائي يكون مظبوط.