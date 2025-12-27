## Extending a Class

لما تعمل inheritance، الكلاس الابن بيرث الحاجات بتاعت الأب حسب الـ access modifiers

# Creating Classes & Access Modifiers

## 🧩 Extending a Class
- ال`extends` بتخلي كلاص تورث من كلاص تانية.
- ال`private` مش بيتورث، `protected` بيتورث.
- نقدر نوصل للـ private عن طريق getters/setters.

## 🏷️ Class Access Modifiers
-ال `public`: ممكن يوصلها أي كلاص.
- ال`(default)`: نفس الـ package بس.
- ❌ لا تستخدم `private` أو `protected` مع top-level classes.

|نوع الكلاس|public|protected|private|
|---|---|---|---|
|Top-Level|✅|❌|❌|
|Inner Class|✅|✅|✅|

- Nested classes ممكن يبقى عندها أي modifier.

## 💡 this Reference
- ال`this` معناها الـ object الحالي.
- بتستخدم لو فيه اسم parameter زي اسم الـ variable في الكلاص.
- ❌ال `length = length;` → غلط.
- ✅ `this.length = length;`.

## ⚠️ Common Exam Traps
- private members مش متورثة.
- `this` مش مسموحة في static methods.
- top-level protected/private class → compilation error.
- `var = var;` مش بيعمل اللي المفروض يعمله.

## 📝 Recap Example
```java
public void setColor(String color) {
    this.color = color; // Correct
}
```

# Using `super` Reference in Java

## 🧩 Concept
- `super` بترمز للكلاس الأب (parent class).
- تستخدم للوصول لمتغير أو ميثود اتعرّف في الأب.
- شبه `this`، لكن بتستبعد أعضاء الكلاس الحالية.

## ⚙️ Example

## 🟡 Key Differences Between this and super

| Feature | this | super |
|----------|------|-------|
| يرمز إلى؟ | الكائن الحالي | كائن الأب |
| يوصّل إلى؟ | الكلاس الحالية + الموروثة | الموروثة فقط |
| فين يستخدم؟ | instance methods | instance methods |
| الهدف | تمييز الـinstance الحالية | الوصول لنسخة الأب |

## 🧠 Exam Tips
- super مش بيشوف المتغيرات غير الموروثة.
- متغير بنفس الاسم = variable hiding.
- ممنوع استخدام super في static methods.
- super() بيستدعي constructor الأب.

## 🔍 Output Example
