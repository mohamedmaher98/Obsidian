# 🟢 Static vs Instance in Java

## 1️⃣ تعريف

- **الInstance variable / method** → مرتبط بكل object جديد.
    
-ال **Static variable / method** → مرتبط بالكلاس نفسه، مش بأي object.
    
- كل objects بتشارك نفس الـ static variable أو يمكنهم الوصول لنفس الـ static method.

## الوصول للـ Static Members

### 3.1 بالاسم الكلاسيكي

`System.out.println(Snake.hiss); Snake.allGiraffeComeOut();`

### 3.2 عن طريق object (مسموح لكن مش مستحسن)

```java
`Snake s = new Snake();
 System.out.println(s.hiss);  // صح 
 s = null; 
 System.out.println(s.hiss);  // برضه صح`

**مهم:** compiler يعتمد على **type reference** مش object نفسه.
```

## تحديث Static Variables
```java
Snake.hiss = 4;
Snake snake1 = new Snake();
Snake snake2 = new Snake();

snake1.hiss = 6;
snake2.hiss = 5;

System.out.println(Snake.hiss); // 5 → shared

```

## قواعد Static vs Instance

|قاعدة|Static|Instance|
|---|---|---|
|ينادي method بدون object؟|✔️|❌|
|ينادي variable بدون object؟|✔️|❌|
|ينادي instance method جوه static method؟|❌|ممكن مع object|
|ينادي static method جوه instance method؟|✔️|✔️|
|ينادي instance variable جوه static method؟|❌|✔️ مع object|

الحلول لو محتاج instance variable جوه static method
1. **خلي variable static**
    

`static String name = "Sammy";`

2. **اعمل object**
    

`MantaRay ray = new MantaRay(); ray.third();`

---
## Static Initializers in Java
## 1️⃣ تعريف

-ال **Static initializer block** هو كود داخل `{ }` مع كلمة `static`
    
- يُنفذ **مرة واحدة فقط** عندما يتم **تحميل الكلاس لأول مرة**
    
- يستخدم عادةً لتهيئة **static variables** خاصة الـ `final` التي لا يمكن تعيينها عند الإعلان
-
---
ملخص
# 🟢 Java Static – Reference

## 1️⃣ Instance vs Static

| نوع العضو | ينتمي لـ | متى يمكن الوصول؟ |
|-----------|----------|-----------------|
| Instance variable / method | Object | بعد إنشاء object |
| Static variable / method | Class | بدون إنشاء object |

### مثال
```java
class Penguin {
    String name;                 // Instance
    static String tallest;       // Static
}

public class Test {
    public static void main(String[] args) {
        Penguin p1 = new Penguin();
        p1.name = "Lilly";
        Penguin.tallest = "Lilly";

        Penguin p2 = new Penguin();
        p2.name = "Willy";
        Penguin.tallest = "Willy";

        System.out.println(p1.name);      // Lilly
        System.out.println(p2.name);      // Willy
        System.out.println(Penguin.tallest); // Willy
    }
}
```

## 2️⃣ Access Static Members

`System.out.println(Snake.hiss);       // أفضل طريقة Snake s = new Snake(); System.out.println(s.hiss);           // ممكن لكن مش مستحسن s = null; System.out.println(s.hiss);           // برضه صح`

**ملاحظة:** Compiler يعتمد على **type reference** وليس object نفسه.

---

## 3️⃣ Updating Static Variables

`Snake.hiss = 4; Snake snake1 = new Snake(); Snake snake2 = new Snake();  snake1.hiss = 6; snake2.hiss = 5;  System.out.println(Snake.hiss); // 5 → shared بين كل objects`

---

## 4️⃣ Static vs Instance Rules

|قاعدة|Static|Instance|
|---|---|---|
|ينادي method بدون object؟|✔️|❌|
|ينادي variable بدون object؟|✔️|❌|
|ينادي instance method جوه static method؟|❌|ممكن مع object|
|ينادي static method جوه instance method؟|✔️|✔️|
|ينادي instance variable جوه static method؟|❌|✔️ مع object|

---

## 5️⃣ Static Constants (final)

- Static variable اللي **مش عايزين يتغير** → `static final`
    
- Convention: **UPPER_CASE_WITH_UNDERSCORES**
    

`class ZooPen {     private static final int NUM_BUCKETS = 45;     // NUM_BUCKETS = 5; ❌ لا يسمح }`

### Static final Arrays / Objects

`private static final String[] treats = new String[10];  treats[0] = "popcorn"; // ✅ ممكن تعديل المحتوى // treats = new String[5]; ❌ لا إعادة تعيين reference`

---

## 6️⃣ Static Initializers

- Blocks تعمل مرة واحدة عند **تحميل الكلاس**
    
- تستخدم لتعيين **static variables** خاصة الـ `final`
    
```java
private static final int NUM_SECONDS_PER_MINUTE; 
private static final int NUM_MINUTES_PER_HOUR;
 private static final int NUM_SECONDS_PER_HOUR;
   static {     NUM_SECONDS_PER_MINUTE = 60;     NUM_MINUTES_PER_HOUR = 60; }  static {     NUM_SECONDS_PER_HOUR = NUM_SECONDS_PER_MINUTE * NUM_MINUTES_PER_HOUR; }
   
```

```
### قواعد مهمة

|النوع|يمكن تعيينه؟|مثال|
|---|---|---|
|static variable عادي|✅ أي عدد مرات|`private static int one;`|
|static final variable بدون قيمة عند الإعلان|✅ مرة واحدة في static block|`private static final int two; static { two = 2; }`|
|static final variable مع قيمة عند الإعلان|❌ لا يسمح بإعادة التعيين|`private static final int three = 3;`|
|static final variable بدون قيمة ولا تعيين|❌ خطأ كومبايل|`private static final int four; // DOES NOT COMPILE`|

---

## 7️⃣ نصائح للامتحان

- أي object تستخدمه للوصول لـ static → **المهم type reference مش object نفسه**
    
- Static method لا يمكنها الوصول لـ instance variable أو instance method بدون object
    
- تحديث static variable يؤثر على كل objects
    
- Static final → immutable reference، لكن محتوى array/object ممكن يتغير
  
  ```

# 🟢 Passing Data among Methods – Java

## 1️⃣ Java is Pass-by-Value

- كل قيمة أو reference لما تبعت method → نسخة (copy) بتتبعت  
- **Assignments داخل الميثود لا تؤثر على المتغير الأصلي**  

### مثال مع primitive
```java
public static void main(String[] args) {
    int num = 4;
    newNumber(num);
    System.out.print(num); // 4
}

public static void newNumber(int num) {
    num = 8; // مجرد نسخة محلية، المتغير الأصلي ما اتغيرش
}

# 📘 Chapter Summary – Methods & Variables in Java

## 1️⃣ Declaring Methods

- Methods تبدأ بـ **access modifiers**: `private`, package (default), `protected`, `public`  
- Optional specifiers: `static` (covered here), others in future chapters  
- بعد كده: **return type**, method name, parameter list  
- Method signature = method name + parameter list  
- Optional **exceptions list** بعد parameter list  
- Method body داخل `{ }` (مستبعدة للـ abstract methods)

---

## 2️⃣ Access Modifiers

| Modifier   | Accessible From                     |
|------------|------------------------------------|
| private    | نفس الكلاس فقط                     |
| package    | نفس الحزمة فقط                     |
| protected  | نفس الحزمة + subclasses           |
| public     | أي مكان                             |

---

## 3️⃣ Static Methods & Variables

- **Static members** مشتركة بين كل الـ instances  
- **Instance members** ممكن ينادوا static members  
- **Static members** لا يمكنهم نداء instance members بدون instance  
- يمكن استخدام **static imports** لاستدعاء static members مباشرة  

---

## 4️⃣ Final Modifier

- يمكن تطبيقه على **local, instance, static variables**  
- Local variable **effectively final** إذا لم تتغير بعد التعيين  
- اختبار سريع: ضيف `final` → لو الكود يشتغل يبقى effectively final

---

## 5️⃣ Passing Data Among Methods

- Java = **pass-by-value**  
- تغيير قيمة parameters داخل الميثود لا يؤثر على المتغير الأصلي  
- تعديل **object parameter نفسه** يظهر خارج الميثود  

**مثال:**  
```java
StringBuilder sb = new StringBuilder("Hello");
modify(sb);
System.out.println(sb); // HelloWorld
