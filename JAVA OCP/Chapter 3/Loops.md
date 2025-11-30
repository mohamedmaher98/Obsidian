while loop
افضل لف طول ما الشرط true”
```java
while (condition) {
    // code
}
int i = 1;

while (i <= 5) {
    System.out.println(i);
    i++;   // متنساش تزود القيمة!
}

```

---
do-while loop
نفّذ مرة واحدة على الأقل… وبعدين اسأل

```java
do {
    // code
} while (condition);


int x = 10;

do {
    System.out.println(x);
    x++;
} while (x < 10);
```

---
for loop
الأفضل لما تعرف “عدد مرات التكرار”

```java
for (initialization; condition; increment) {
    // code
}


```
- كل حاجة في سطر واحد (البداية + الشرط + الزيادة)
    
- أحسن اختيار في loops العدّادية
---
**Enhanced For Loop (for-each)**

```java
int[] numbers = {10, 20, 30};

for (int n : numbers) {
    System.out.println(n);
}



```
## 💡 مناسب جدًا لـ:

- Arrays
    
- Lists
    
- Sets
    
- أي Iterable
    

## ❌ لكن:

- مش ينفع تعدّل index
    
- مش تقدر تـ"skip" لعنصر قبله أو بعده غير باستخدام continue
    
- مش ينفع تستخدمه لو عايز تعرف رقم العنصر (index)
---
**Infinite Loops**

```java
while (true) {
    // runs forever
}

for(;;) {
    // runs forever
}


```

---
سادسًا: **loop control keywords**
## 1) **break**

```java
while(true) {
    if(x == 5) break;
}
يخرج فورًا من اللوب
```


2) **continue**
```java
for(int i=1; i<=5; i++) {
    if(i == 3) continue;
    System.out.println(i);
}
1245
```

3) **labelled break / continue** (متقدّم شوية)

```java
outer:
for(int i=1; i<=3; i++){
    for(int j=1; j<=3; j++){
        if(j == 2) break outer;
    }
}
```


---
QQQ
## ✔️ 1) الفرق بين while و do-while؟

- while → ممكن مايتنفّذش
    
- do-while → لازم يتنفّذ مرة
    

## ✔️ 2) إمتى تستخدم for بدل while؟

لما تكون عارف عدد مرات التكرار.

## ✔️ 3) هل ينفع تكتب for loop بدون condition؟

أيوه → infinite loop

---
## 5) إيه الفرق بين for العادي و for-each؟

- for العادي → عندك index
    
- for-each → للقراءة فقط
## ✔️ 6) هل ينفع تحذف عنصر من List وانت في for-each؟

❌ لأ → يعمل Exception  
✔️ لازم تستخدم Iterator
