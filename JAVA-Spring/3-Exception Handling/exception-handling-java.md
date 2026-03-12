# Exception Handling in Java - شرح شامل

## جدول المحتويات

1. [ايه هو الـ Exception؟](#1---ايه-هو-الـ-exception)
2. [التسلسل الهرمي للـ Exceptions](#2---التسلسل-الهرمي)
3. [أنواع الـ Exceptions](#3---أنواع-الـ-exceptions)
4. [try-catch](#4---try-catch)
5. [finally](#5---finally)
6. [throw و throws](#6---throw-و-throws)
7. [Custom Exceptions](#7---custom-exceptions)
8. [try-with-resources](#8---try-with-resources)
9. [Multi-catch](#9---multi-catch)
10. [أهم الـ Exceptions الشائعة](#10---أهم-الـ-exceptions-الشائعة)
11. [Best Practices](#11---best-practices)

---

## 1 - ايه هو الـ Exception؟

الـ Exception هو **حدث غير طبيعي** بيحصل أثناء تشغيل البرنامج وبيقطع التدفق الطبيعي للكود. بدل ما البرنامج يقع ويقفل، Java بتديك طريقة إنك **تمسك الخطأ وتتعامل معاه**.

```java
// بدون Exception Handling - البرنامج هيقع
int result = 10 / 0; // ArithmeticException!

// مع Exception Handling - البرنامج هيكمل
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("مينفعش تقسم على صفر!");
}
```

---

## 2 - التسلسل الهرمي

```
                    Object
                      |
                  Throwable
                 /         \
            Error          Exception
              |            /        \
        (أخطاء خطيرة)   RuntimeException   (Checked Exceptions)
                          |                     |
                   (Unchecked Exceptions)   IOException
                          |                 SQLException
                   NullPointerException     FileNotFoundException
                   ArrayIndexOutOfBounds    ClassNotFoundException
                   ArithmeticException
                   ClassCastException
                   IllegalArgumentException
```

### Throwable
- الـ class الأب لكل الأخطاء والاستثناءات.

### Error
- أخطاء **خطيرة جداً** مش المفروض تتعامل معاها في الكود.
- أمثلة: `OutOfMemoryError`, `StackOverflowError`

### Exception
- أخطاء **تقدر تتعامل معاها** في الكود بتاعك.

---

## 3 - أنواع الـ Exceptions

### أ) Checked Exceptions (Compile-time)

- الـ compiler **بيجبرك** تتعامل معاها.
- لازم تحطها في `try-catch` أو تعمل `throws` في الـ method.
- بتحصل بسبب **ظروف خارجية** (ملف مش موجود، مشكلة في الشبكة).

```java
// مثال: قراءة ملف - لازم تتعامل مع الـ Exception
import java.io.*;

public class CheckedExample {
    public static void main(String[] args) {
        // الكود ده مش هيعمل compile من غير try-catch أو throws
        try {
            FileReader file = new FileReader("test.txt");
            BufferedReader reader = new BufferedReader(file);
            String line = reader.readLine();
            reader.close();
        } catch (IOException e) {
            System.out.println("مشكلة في قراءة الملف: " + e.getMessage());
        }
    }
}
```

### ب) Unchecked Exceptions (Runtime)

- الـ compiler **مش بيجبرك** تتعامل معاها.
- بتورث من `RuntimeException`.
- بتحصل بسبب **أخطاء في الكود** (logic errors).

```java
public class UncheckedExample {
    public static void main(String[] args) {
        // NullPointerException
        String text = null;
        // text.length(); // هيقع runtime error

        // ArrayIndexOutOfBoundsException
        int[] arr = {1, 2, 3};
        // arr[5] = 10; // هيقع runtime error

        // ArithmeticException
        // int result = 10 / 0; // هيقع runtime error
    }
}
```

---

## 4 - try-catch

الأساس في التعامل مع الـ Exceptions.

```java
try {
    // الكود اللي ممكن يعمل مشكلة
    int[] numbers = {1, 2, 3};
    System.out.println(numbers[5]);
} catch (ArrayIndexOutOfBoundsException e) {
    // التعامل مع المشكلة
    System.out.println("الـ index مش موجود في الـ array!");
    System.out.println("رسالة الخطأ: " + e.getMessage());
}
```

### أكتر من catch block

تقدر تمسك أنواع مختلفة من الـ Exceptions:

```java
try {
    int[] arr = {1, 2, 3};
    int result = arr[2] / 0;
} catch (ArithmeticException e) {
    System.out.println("خطأ حسابي: " + e.getMessage());
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("خطأ في الـ index: " + e.getMessage());
} catch (Exception e) {
    // الـ catch العام - لازم يكون الأخير
    System.out.println("خطأ عام: " + e.getMessage());
}
```

> **ملاحظة مهمة:** لازم تحط الـ Exception الأكثر تحديداً الأول، والأعم في الآخر. يعني `Exception` دايماً في الآخر.

---

## 5 - finally

بلوك `finally` بيتنفذ **دايماً** سواء حصل exception أو لا. بيُستخدم لتنظيف الـ resources.

```java
FileReader file = null;
try {
    file = new FileReader("data.txt");
    // قراءة الملف
} catch (FileNotFoundException e) {
    System.out.println("الملف مش موجود!");
} finally {
    // الكود ده هيتنفذ في كل الحالات
    System.out.println("بلوك finally اتنفذ");
    if (file != null) {
        try {
            file.close();
        } catch (IOException e) {
            System.out.println("مشكلة في إغلاق الملف");
        }
    }
}
```

### حالات تنفيذ finally

```java
// حالة 1: مفيش exception - finally هيتنفذ
try {
    System.out.println("كل حاجة تمام");
} finally {
    System.out.println("finally اتنفذ"); // هيتنفذ
}

// حالة 2: في exception اتمسك - finally هيتنفذ
try {
    int x = 1 / 0;
} catch (ArithmeticException e) {
    System.out.println("اتمسك");
} finally {
    System.out.println("finally اتنفذ"); // هيتنفذ
}

// حالة 3: في exception ماتمسكش - finally هيتنفذ برضو!
try {
    int x = 1 / 0;
} finally {
    System.out.println("finally اتنفذ"); // هيتنفذ قبل ما البرنامج يقع
}
```

---

## 6 - throw و throws

### throw

بتستخدمها عشان **ترمي exception بنفسك** يدوياً.

```java
public class ThrowExample {

    public static int divide(int a, int b) {
        if (b == 0) {
            throw new ArithmeticException("مينفعش تقسم على صفر!");
        }
        return a / b;
    }

    public static void validateAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException("العمر مينفعش يكون سالب!");
        }
        if (age < 18) {
            throw new IllegalArgumentException("لازم يكون 18 سنة أو أكبر!");
        }
        System.out.println("العمر مقبول");
    }

    public static void main(String[] args) {
        try {
            validateAge(-5);
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

### throws

بتُستخدم في **تعريف الـ method** عشان تقول إن الـ method دي **ممكن ترمي exception** واللي بيستدعيها لازم يتعامل معاه.

```java
import java.io.*;

public class ThrowsExample {

    // الـ method دي ممكن ترمي IOException
    public static String readFile(String path) throws IOException {
        BufferedReader reader = new BufferedReader(new FileReader(path));
        String content = reader.readLine();
        reader.close();
        return content;
    }

    // ممكن ترمي أكتر من نوع
    public static void process(String path) throws IOException, IllegalArgumentException {
        if (path == null) {
            throw new IllegalArgumentException("المسار مينفعش يكون null");
        }
        String content = readFile(path);
        System.out.println(content);
    }

    public static void main(String[] args) {
        try {
            String content = readFile("data.txt");
            System.out.println(content);
        } catch (IOException e) {
            System.out.println("مشكلة: " + e.getMessage());
        }
    }
}
```

### الفرق بين throw و throws

| `throw` | `throws` |
|---------|----------|
| بتُستخدم **جوه الـ method** | بتُستخدم في **تعريف الـ method** |
| بترمي **exception واحد** | بتعلن عن **exception أو أكتر** |
| بتيجي مع **object** من الـ exception | بتيجي مع **اسم الـ class** |
| `throw new Exception("msg")` | `void method() throws Exception` |

---

## 7 - Custom Exceptions

تقدر تعمل **exceptions خاصة بيك** عشان تعبر عن أخطاء معينة في الـ application بتاعك.

### Custom Checked Exception

```java
// لازم تورث من Exception
public class InsufficientBalanceException extends Exception {

    private double amount;
    private double balance;

    public InsufficientBalanceException(double amount, double balance) {
        super("الرصيد مش كفاية! محتاج: " + amount + " - الرصيد الحالي: " + balance);
        this.amount = amount;
        this.balance = balance;
    }

    public double getAmount() { return amount; }
    public double getBalance() { return balance; }
}
```

### Custom Unchecked Exception

```java
// بتورث من RuntimeException
public class InvalidEmailException extends RuntimeException {

    public InvalidEmailException(String email) {
        super("الإيميل ده مش صحيح: " + email);
    }
}
```

### استخدام الـ Custom Exceptions

```java
public class BankAccount {
    private double balance;

    public BankAccount(double balance) {
        this.balance = balance;
    }

    // Checked - اللي بيستدعي لازم يتعامل معاه
    public void withdraw(double amount) throws InsufficientBalanceException {
        if (amount > balance) {
            throw new InsufficientBalanceException(amount, balance);
        }
        balance -= amount;
        System.out.println("تم السحب بنجاح. الرصيد: " + balance);
    }

    // Unchecked - مش لازم يتعامل معاه
    public void setEmail(String email) {
        if (!email.contains("@")) {
            throw new InvalidEmailException(email);
        }
        System.out.println("تم تسجيل الإيميل: " + email);
    }

    public static void main(String[] args) {
        BankAccount account = new BankAccount(1000);

        try {
            account.withdraw(1500);
        } catch (InsufficientBalanceException e) {
            System.out.println(e.getMessage());
            System.out.println("كنت محتاج: " + e.getAmount());
        }
    }
}
```

---

## 8 - try-with-resources

اتضافت في **Java 7**. بتقفل الـ resources تلقائياً من غير ما تحتاج `finally`. الـ resource لازم ينفّذ `AutoCloseable` interface.

```java
// الطريقة القديمة (قبل Java 7)
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("data.txt"));
    String line = reader.readLine();
    System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (reader != null) {
        try {
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// الطريقة الجديدة - try-with-resources
try (BufferedReader reader = new BufferedReader(new FileReader("data.txt"))) {
    String line = reader.readLine();
    System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
}
// reader اتقفل تلقائياً!
```

### أكتر من resource

```java
try (
    FileInputStream input = new FileInputStream("input.txt");
    FileOutputStream output = new FileOutputStream("output.txt")
) {
    int data;
    while ((data = input.read()) != -1) {
        output.write(data);
    }
} catch (IOException e) {
    e.printStackTrace();
}
// الاتنين اتقفلوا تلقائياً!
```

### عمل Custom AutoCloseable

```java
public class DatabaseConnection implements AutoCloseable {
    private String name;

    public DatabaseConnection(String name) {
        this.name = name;
        System.out.println("اتصال بـ " + name + " تم فتحه");
    }

    public void query(String sql) {
        System.out.println("بينفذ: " + sql);
    }

    @Override
    public void close() {
        System.out.println("اتصال بـ " + name + " تم إغلاقه");
    }
}

// الاستخدام
try (DatabaseConnection db = new DatabaseConnection("MyDB")) {
    db.query("SELECT * FROM users");
}
// الاتصال اتقفل تلقائياً
```

---

## 9 - Multi-catch

اتضافت في **Java 7**. بتمسك أكتر من نوع exception في `catch` واحد.

```java
try {
    // كود ممكن يرمي أنواع مختلفة
    String text = "abc";
    int number = Integer.parseInt(text);
} catch (NumberFormatException | ArithmeticException | NullPointerException e) {
    // التعامل مع أي واحد منهم
    System.out.println("حصل خطأ: " + e.getMessage());
}
```

> **ملاحظة:** الأنواع في الـ multi-catch مينفعش يكون واحد منهم subclass للتاني.

---

## 10 - أهم الـ Exceptions الشائعة

### Unchecked (Runtime)

| Exception | السبب | مثال |
|-----------|-------|------|
| `NullPointerException` | استخدام متغير قيمته `null` | `String s = null; s.length();` |
| `ArrayIndexOutOfBoundsException` | index خارج حدود الـ array | `int[] a = {1,2}; a[5];` |
| `StringIndexOutOfBoundsException` | index خارج حدود الـ String | `"hi".charAt(10);` |
| `ArithmeticException` | خطأ حسابي (القسمة على صفر) | `10 / 0;` |
| `NumberFormatException` | تحويل string مش رقم لرقم | `Integer.parseInt("abc");` |
| `ClassCastException` | casting غلط | `Object o = "hi"; Integer i = (Integer) o;` |
| `IllegalArgumentException` | argument مش صحيح للـ method | حسب الـ validation |
| `IllegalStateException` | الـ object في حالة مش مناسبة | استدعاء method في وقت غلط |
| `StackOverflowError` | recursion لا نهائي | method بتنادي نفسها بدون شرط وقف |
| `ConcurrentModificationException` | تعديل collection أثناء التكرار | حذف عنصر من list أثناء for-each |

### Checked

| Exception | السبب |
|-----------|-------|
| `IOException` | مشكلة في الـ I/O (قراءة/كتابة) |
| `FileNotFoundException` | الملف مش موجود |
| `SQLException` | مشكلة في قاعدة البيانات |
| `ClassNotFoundException` | الـ class مش موجود |
| `InterruptedException` | thread اتقاطع |

---

## 11 - Best Practices

### 1. امسك الـ Exception المحدد مش العام

```java
// غلط
try {
    readFile("data.txt");
} catch (Exception e) {
    e.printStackTrace();
}

// صح
try {
    readFile("data.txt");
} catch (FileNotFoundException e) {
    System.out.println("الملف مش موجود");
} catch (IOException e) {
    System.out.println("مشكلة في القراءة");
}
```

### 2. متبلعش الـ Exception

```java
// غلط - بتخبي المشكلة
try {
    riskyOperation();
} catch (Exception e) {
    // مفيش حاجة! الخطأ اتبلع
}

// صح
try {
    riskyOperation();
} catch (Exception e) {
    logger.error("حصلت مشكلة", e);
    // أو على الأقل
    e.printStackTrace();
}
```

### 3. استخدم finally أو try-with-resources لتنظيف الـ Resources

```java
// الأفضل - try-with-resources
try (Connection conn = getConnection()) {
    // استخدم الاتصال
}
```

### 4. متستخدمش Exceptions للتحكم في البرنامج

```java
// غلط
try {
    int value = Integer.parseInt(input);
} catch (NumberFormatException e) {
    // مش رقم - عادي
}

// صح - تحقق الأول
if (input.matches("\\d+")) {
    int value = Integer.parseInt(input);
}
```

### 5. حط رسالة واضحة في الـ Exception

```java
// غلط
throw new IllegalArgumentException("خطأ");

// صح
throw new IllegalArgumentException(
    "العمر لازم يكون بين 0 و 150، لكن القيمة المدخلة: " + age
);
```

### 6. استخدم Exception Chaining لما تعمل re-throw

```java
try {
    readConfig();
} catch (IOException e) {
    // حافظ على الـ original exception كـ cause
    throw new ConfigException("فشل تحميل الإعدادات", e);
}
```

### 7. متعملش throw في finally

```java
// غلط - ممكن يخبي الـ exception الأصلي
try {
    riskyOperation();
} finally {
    throw new RuntimeException("خطأ!"); // هيخبي أي exception من try
}
```

---

## ملخص سريع

```
try       → حط فيه الكود اللي ممكن يعمل مشكلة
catch     → امسك الـ exception واتعامل معاه
finally   → كود بيتنفذ دايماً (تنظيف)
throw     → ارمي exception بنفسك
throws    → علّن إن الـ method ممكن ترمي exception
```

```java
// الشكل الكامل
public void myMethod() throws MyException {
    try (Resource res = new Resource()) {
        if (problem) {
            throw new MyException("في مشكلة!");
        }
        // كود عادي
    } catch (SpecificException e) {
        // تعامل مع الخطأ المحدد
        logger.error(e.getMessage(), e);
    } catch (GeneralException e) {
        // تعامل مع الخطأ العام
        throw new MyException("حصل خطأ", e);
    } finally {
        // تنظيف إضافي لو محتاج
    }
}
```
