# Wrapper Classes in Java

## تعريف

الـ Wrapper Classes هي Objects بتغلف الـ Primitive Types (زي int) عشان تقدر تستخدمها في سياقات الـ Object-Oriented.

## 1. الوظيفة الأساسية (The Need)

- **تمثيل كـ Object**: تحويل الـ Primitives لـ Objects لتكون متوافقة مع الـ Frameworks.
    
- **Collections & Generics**: لتخزين Primitives في Collections زي ArrayList، لأنها بتقبل Objects فقط.
    

## 2. أمثلة على الـ Wrappers

|Primitive|Wrapper Class|Default Value|
|---|---|---|
|int|Integer|null|
|char|Character|null|
|boolean|Boolean|null|

## 3. آليات التحويل

|   |   |   |
|---|---|---|
|العملية|الوصف|التأثير على الأداء|
|Auto-boxing|تحويل تلقائي من Primitive لـ Wrapper Object|زيادة استخدام Heap وGC|
|Unboxing|تحويل تلقائي من Wrapper Object لـ Primitive|ممكن يسبب NullPointerException لو القيمة null|


### 4. قواعد المقارنة (The Golden Rule) 🥇

مقارنة ال int نتسخدم == لانها بتقارن قيمة 
مقارنة ال integer نستخدم .equals   
