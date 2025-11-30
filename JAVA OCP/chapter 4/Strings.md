# # 📌 Java — Creating and Manipulating Strings

## ## 🔥 1. What is a String?

- الـ **String** في Java هو **كائن (Object)** بيمثل نص.
    
- من فئة **java.lang.String**.
    
- باستخدام علامتي الاقتباس `" "`.
    

---

## ## 🧊 2. String is Immutable

**Immutable = لا يمكن تغييره بعد إنشائه.**

أي تعديل = إنشاء Object جديد.

مثال:

`String s = "Hello";
s = s + " World";
// تم إنشاء object جديد`

### ✔️ ليه immutable؟

- تحسين الأداء مع **String Pool**
    
- Thread-safe
    
- يستخدم في الـ caching
    
- يحافظ على ثبات البيانات
    

---

## ## 🧠 3. String Pool

منطقة داخل الـ Heap اسمها:

### **String Constant Pool**

- النصوص المكتوبة بطريقة literal تروح للـ pool:
    
    `String a = "Java"; String b = "Java"; // يشاور على نفس ال object`
    
- لكن:
    
    `String c = new String("Java");`
    
    ده يعمل Object جديد خارج الـ pool.
    

---

# # ✂️ 4. أشهر عمليات Manipulation على Strings

## ## ✔️ length()

يرجع طول النص.

`"Maher".length(); // 5`

## ## ✔️ charAt(index)

يرجع حرف بمكان معين.

`"Hello".charAt(1); // e`

## ## ✔️ substring(start, end)

قطع جزء من النص.

`"Maher".substring(0, 3); // Mah`

## ## ✔️ contains()

للتحقق هل كلمة موجودة داخل النص.

`"Hello Java".contains("Java"); // true`

## ## ✔️ equals() / equalsIgnoreCase()

`"java".equals("java"); // true "JAVA".equalsIgnoreCase("java"); // true`

## ## ✔️ startsWith() / endsWith()

## ## ✔️ toUpperCase() / toLowerCase()

## ## ✔️ trim()

يشيل المسافات من البداية والنهاية.

## ## ✔️ replace()

`"Java".replace("a", "o"); // Jovo`

---

# # 🔥 5. String vs StringBuilder vs StringBuffer

## ## ✔️ String

- ❌ Immutable
    
- ❌ بطيء في التعديل الكثير
    
- ✔️ آمن (Thread-safe)
    

---

## ## ✔️ StringBuilder

- ✔️ Mutable
    
- ✔️ أسرع
    
- ❌ غير آمن للـ threads
    

مثال:

`StringBuilder sb = new StringBuilder(); sb.append("Maher"); sb.append(" Java");`

---

## ## ✔️ StringBuffer

- ✔️ Mutable
    
- ✔️ Thread-safe
    
- ❌ أبطأ من StringBuilder
    

---

# # 🧪 مقارنة سريعة

|النوع|Mutable؟|السرعة|Thread-safe|
|---|---|---|---|
|String|❌ لأ|متوسط|✔️ نعم|
|StringBuilder|✔️ نعم|🔥 سريع|❌ لأ|
|StringBuffer|✔️ نعم|أبطأ شوية|✔️ نعم|

---

# # 🎤 6. أشهر أسئلة Interview

## ## ❓ 1. ليه String immutable؟

**الإجابة:**

- security
    
- thread-safety
    
- استخدام String pool
    
- تحسين الأداء
    

---

## ## ❓ 2. الفرق بين `"Java"` و `new String("Java")`؟

- `"Java"` → يروح للـ String Pool
    
- `new String("Java")` → يعمل object جديد خارج الـ pool
    

---

## ## ❓ 3. إمتى تستخدم StringBuilder؟

لما تعمل concatenation كتير جوه loops.

---

## ## ❓ 4. ليه StringBuilder أسرع؟

عشان **مش immutable** فمش بيعمل object جديد كل مرة.

---

## ## ❓ 5. يعني إيه String Pool؟

منطقة داخل الذاكرة لتخزين النصوص وإعادة استخدامها.

---

# # 🎯 الخلاصة

- String immutable
    
- String pool لتعظيم الأداء
    
- استخدم StringBuilder عند التعديلات الكتيرة
    
- مهم جدًا تحفظ الفروقات للانترفيو