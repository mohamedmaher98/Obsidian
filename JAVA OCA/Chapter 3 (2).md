### ==Core Java APIs==
 هي مجموعة من الـ Java Standard Edition (Java SE وغالبًا بنشتغل بيهم حتى من غير ما نحس
 ### أهم **Core Java APIs**:

1. **java.lang**
    
    - ده الباكيج الأساسي اللي فيه حاجات زي:
        
        - `String`, `Math`, `Object`, `System`, `Thread`, `Exception`
            
    - بيتم استيراده تلقائيًا.
        
2. **java.util**
    
    - أدوات المساعدة (Utilities):
        
        - Collections API: `List`, `Set`, `Map`, `Queue`
            
        - `Date`, `Calendar`, `Random`, `Scanner`
            
3. **java.io**
    
    - للتعامل مع الإدخال والإخراج (Input/Output):
        
        - `File`, `FileInputStream`, `BufferedReader`, `PrintWriter`
            
4. **java.nio**
    
    - بديل أسرع وأحدث لـ `java.io` للتعامل مع الملفات والـ buffers.
        
5. **java.net**
    
    - للتعامل مع الشبكات:
        
        - `Socket`, `URL`, `HttpURLConnection`
            
6. **java.sql**
    
    - للتعامل مع قواعد البيانات:
        
        - `Connection`, `Statement`, `ResultSet`, `PreparedStatement`
            
7. **java.time** (من Java 8)
    
    - للتعامل مع التواريخ والأوقات بشكل أكثر كفاءة:
        
        - `LocalDate`, `LocalTime`, `LocalDateTime`, `Duration`
            
8. **java.math**
    
    - العمليات الحسابية الدقيقة:
        
        - `BigInteger`, `BigDecimal`
---
### ==Wrapper Classes==

هي عبارة عن كلاسات بتغلف ال **primitive types**
لاأن في حاجات في Java بتشتغل مع الكائنات بس (زي الـ Collections مثلاً)، ومينفعش تستخدم معاها types زي `int` أو `boolean` مباشرة. علشان كده بنستخدم الـ **Wrapper**.

![[Pasted image 20250504142158.png]]

| الفايدة                | الشرح                                                                            |
| ---------------------- | -------------------------------------------------------------------------------- |
| التعامل مع Collections | الـ Collections (زي `ArrayList`, `HashMap`) بتشتغل مع Objects بس، مش primitives. |
| إمكانية استخدام null   | الـ Wrapper يقدر يشيل null، لكن الـ primitive لأ.                                |
| أدوات مساعدة           | كل Wrapper فيه methods قوية زي `parseInt()`, `valueOf()`, `compareTo()`, إلخ.    |
|                        |                                                                                  |
|                        |                                                                                  |
> ✅ بدون الـ Wrapper Classes، مش هينفع تحط `int` في `ArrayList`.
### ==Autoboxing==

تحويل تلقائي من primitive إلى Wrapper.
يعني من الصغير للكبير 
Autoboxing/Unboxing :بيخلّي الكود أنضف وأسهل بدون الحاجة لتحويل يدوي.
![[Pasted image 20250504143452.png]]

|الحالة|ليه تستخدم Wrapper؟|
|---|---|
|تخزين أرقام في List أو Map|لأنها بتقبل كائنات بس|
|تحويل String لرقم|تستخدم `parseInt` أو `valueOf`|
|تحتاج null كقيمة افتراضية|Wrapper يقدر يشيل null|
|التعامل مع أدوات Java الحديثة (Stream, Lambda)|بتحتاج Objects مش primitives|

---
### ==تحويل ArrayList إلى Array
```java
List<String> list = new ArrayList<>();
list.add("hawk");
list.add("robin");

Object[] objArray = list.toArray(); // نوعه Object
----------------------------------------------------------------------------
### الطريقة 2: باستخدام `toArray(new Type[0])

`String[] strArray = list.toArray(new String[0]); System.out.println(strArray.length); // 2- ✅ دي الطريقة الصح لأنك بتحصل على `String[]` مش `Object[]`.
    
- تقدر تستخدم أي عدد كـ حجم مبدأي (`new String[0]`, `new String[10]`...)، Java هتعرف تعدل الحجم لو لزم.
  ``` 
---
```java
public class ArrayListConvert {
    public static void main(String[] args) {
        // List to Array
        List<String> birds = new ArrayList<>();
        birds.add("hawk");
        birds.add("robin");
        String[] birdArray = birds.toArray(new String[0]);

        // Array to List (backed list)
        String[] animals = {"cat", "dog"};
        List<String> animalList = Arrays.asList(animals);
        animalList.set(1, "lion");
        animals[0] = "tiger";

        System.out.println(Arrays.toString(animals));     // [tiger, lion]
        System.out.println(animalList);                   // [tiger, lion]
    }
}

```
### ==Sorting==

```java
import java.util.*;

public class SortExample {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>();
        numbers.add(99);
        numbers.add(5);
        numbers.add(81);

        Collections.sort(numbers); // الفرز هنا

        System.out.println(numbers); // [5, 81, 99]
    }
}
```
لو عايز ترتيب تنازلي → تقدر تستخدم:
numbers.sort(Collections.reverseOrder());

---
### ==Date And Time== 
عشان تتعامل معاها بتحتاج تعمل استيراد ل 
import java.time.*;
- **LocalDate**:
    
    - يحتوي فقط على التاريخ (دون الوقت أو المنطقة الزمنية).
        
    - مثال: يوم عيد ميلادك.
        
- **LocalTime**:
    
    - يحتوي فقط على الوقت (دون التاريخ أو المنطقة الزمنية).
        
    - مثال: منتصف الليل.
        
- **LocalDateTime**:
    
    - يحتوي على كلاً من التاريخ والوقت (دون المنطقة الزمنية).
        
    - مثال: الساعة الثانية عشرة منتصف الليل في رأس السنة.
      - **LocalDate**:  
    يعرض فقط التاريخ الحالي (بدون وقت).
    
    `System.out.println(LocalDate.now());`
    
- **LocalTime**:  
    يعرض الوقت الحالي (بدون تاريخ).
    
    `System.out.println(LocalTime.now());`
    
- **LocalDateTime**:  
    يعرض كلاً من التاريخ والوقت الحاليين.
    `System.out.println(LocalDateTime.now());`
    

### **المخرجات المتوقعة**:

عند تشغيل هذه الأكواد في **20 يناير الساعة 12:45 مساءً**، ستظهر النتائج كما يلي:

- **LocalDate**: `2015-01-20`
    
- **LocalTime**: `12:45:18.401`
    
- **LocalDateTime**: `2015-01-20T12:45:18.401`
  
  في Java، **التواريخ والأوقات** هي فئات غير قابلة للتغيير (immutable). هذا يعني أنه بعد تعديل التاريخ أو الوقت باستخدام طرق مثل `plusDays()` أو `minusDays()`, يجب عليك تعيين النتيجة إلى متغير مرجعي جديد، لأن الكائن الأصل لا يتغير.
  ```java
LocalDate date = LocalDate.of(2014, Month.JANUARY, 20);
System.out.println(date); // 2014-01-20

date = date.plusDays(2);  // إضافة يومين
System.out.println(date); // 2014-01-22

date = date.plusWeeks(1); // إضافة أسبوع
System.out.println(date); // 2014-01-29

date = date.plusMonths(1); // إضافة شهر
System.out.println(date); // 2014-02-28

date = date.plusYears(5);  // إضافة خمس سنوات
System.out.println(date); // 2019-02-28

```
### ==الملخص== 
### **ملخص الفصل عن Strings و Arrays و ArrayLists و Dates في Java**

#### **1. Strings:**

- **Strings** هي تسلسلات ثابتة من الأحرف (immutable). هذا يعني أنه لا يمكن تعديل الكائن بعد إنشائه.
    
- **المُشغل الجديد (new)** ليس ضروريًا عند إنشاء String جديد.
    
- **المُشغل (+)** يستخدم للدمج بين سلسلتين من النصوص، حيث ينشئ String جديد يدمج النصوص.
    
- **String Pool**: يتم تخزين String literals في **String pool** لتقليل استهلاك الذاكرة وتحسين الأداء.
    
- **طرق مهمة في String**:
    
    - `charAt()`: للحصول على الحرف في موضع معين.
        
    - `concat()`: لدمج نصوص.
        
    - `equals()`: للمقارنة بين النصوص.
        
    - `replace()`: لاستبدال جزء من النص.
        
    - `substring()`: لاستخراج جزء من النص.
        
    - `toLowerCase()` و `toUpperCase()` لتحويل النصوص إلى حروف صغيرة أو كبيرة.
        
    - `trim()`: لإزالة الفراغات في البداية والنهاية.
        

#### **2. StringBuilder و StringBuffer:**

- **StringBuilder**: هي تسلسلات متغيرة من الأحرف (mutable). يمكن تعديل النصوص في الكائن دون الحاجة لإنشاء كائن جديد.
    
    - **StringBuffer**: نفس **StringBuilder** ولكن **Thread-safe** (آمن لاستخدامه في بيئة متعددة الخيوط).
        
- **طرق مهمة في StringBuilder**:
    
    - `append()`: لإضافة نص إلى النص الحالي.
        
    - `delete()`, `deleteCharAt()`: لحذف نصوص.
        
    - `reverse()`: لعكس النص.
        
    - `toString()`: لتحويل **StringBuilder** إلى **String**.
        
- **الفرق بين String و StringBuilder**:
    
    - عند مقارنة **String** باستخدام `==`, يتم التحقق مما إذا كان الكائنين يشيران إلى نفس الكائن في **String pool**.
        
    - عند مقارنة **StringBuilder** باستخدام `==`, يتم التحقق مما إذا كانا يشيران إلى نفس الكائن.
        

#### **3. Arrays:**

- **Arrays**: هي مناطق ثابتة من الذاكرة على heap تحتوي على بيانات من نوع معين (مثل int[] أو String[]).
    
    - **Arrays.sort()**: لترتيب المصفوفات.
        
    - **Arrays.binarySearch()**: للبحث في مصفوفة مرتبة.
        
    - **المرجعية (varargs)**: يمكن تمرير عدد غير ثابت من المعاملات باستخدام `...`.
        
    - **المصفوفات متعددة الأبعاد**: يمكن أن تحتوي على مصفوفات فرعية بأحجام مختلفة.
        

#### **4. ArrayLists:**

- **ArrayLists**: هي هياكل بيانات يمكن تعديل حجمها بمرور الوقت.
    
    - **طريقة `add()`**: لإضافة عنصر.
        
    - **طريقة `remove()`**: لإزالة عنصر.
        
    - **طريقة `contains()`**: للتحقق مما إذا كان العنصر موجودًا.
        
    - **طريقة `size()`**: للحصول على حجم **ArrayList**.
        
    - **طريقة `set()`**: لتعديل عنصر في موضع معين.
        

#### **5. Dates في Java:**

- **LocalDate**: يحتوي على التاريخ فقط (بدون وقت أو منطقة زمنية).
    
- **LocalTime**: يحتوي على الوقت فقط (بدون تاريخ أو منطقة زمنية).
    
- **LocalDateTime**: يحتوي على التاريخ والوقت ولكن بدون منطقة زمنية.
    
- **طرق مهمة للعمل مع Dates**:
    
    - `plusXXX()` و `minusXXX()`: لإضافة أو خصم أيام، شهور، أو سنوات.
        
    - **Period**: تمثل فترة زمنية يمكن إضافتها أو خصمها من **LocalDate** أو **LocalDateTime**.
        
    - **DateTimeFormatter**: لتنسيق التاريخ والوقت بالطريقة المطلوبة.
        

#### **نقاط هامة:**

- **تاريخ وأوقات غير قابلة للتغيير**: كما هو الحال مع **String**، يتم إنشاء كائنات التاريخ والوقت الجديدة دون تعديل الكائنات الأصلية.
    
- **إنشاء الكائنات**: يتم إنشاء الكائنات باستخدام طرق مثل `LocalDate.now()` أو `LocalDate.of()`.