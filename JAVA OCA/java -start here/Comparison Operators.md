الـ **Comparison Operators** في Java بتستخدمها لما تحب تقارن بين حاجتين، وبتديلك ناتج **true** أو **false**. يعني بتسأل السؤال: "هل الشرط ده صح ولا غلط؟"

---

## 📌 أنواع Comparison Operators

### ✅ == (يساوي؟)

هل القيمتين زي بعض؟

```
int x = 5;
int y = 5;
System.out.println(x == y); // true
```

### ✅ `!=` (لا يساوي؟)

هل القيمتين مختلفين؟

```
int x = 5;
int y = 3;
System.out.println(x != y); // true
```

### ✅ `>` (أكبر من؟)

```
int age = 20;
System.out.println(age > 18); // true
```

### ✅ `<` (أصغر من؟)

```
int age = 16;
System.out.println(age < 18); // true
```

### ✅ `>=` (أكبر من أو يساوي؟)

```
int score = 90;
System.out.println(score >= 90); // true
```

### ✅ `<=` (أصغر من أو يساوي؟)

```
int temp = 30;
System.out.println(temp <= 40); // true
```

---

## ⚠️ ملاحظات مهمة

- النتائج دايمًا Boolean: يا `true` يا `false`.
    
- لما تقارن Strings أو Objects ما تستخدمش = =، استخدم `.equals()`.
    

```
String name1 = "Ali";
String name2 = new String("Ali");
System.out.println(name1 == name2); // false
System.out.println(name1.equals(name2)); // true
```

---

## 🔗 نوتات مرتبطة