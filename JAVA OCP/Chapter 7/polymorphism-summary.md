
## يعني إيه Polymorphism؟

الـ Polymorphism معناها إن الـ Object الواحد ممكن ياخد أشكال كتير. يعني الـ Object بتاعك ممكن تشاور عليه بأكتر من نوع Reference:

لReference من نفس نوع الـ Object
- لReference من الـ Superclass بتاعته
- اReference من Interface هو بيـ implement

### مثال عملي

```java
public class Primate {
    public boolean hasHair() {
        return true;
    }
}

public interface HasTail {
    public abstract boolean isTailStriped();
}

public class Lemur extends Primate implements HasTail {
    public boolean isTailStriped() {
        return false;
    }
    public int age = 10;
    
    public static void main(String[] args) {
        Lemur lemur = new Lemur();
        System.out.println(lemur.age);        // 10
        
        HasTail hasTail = lemur;
        System.out.println(hasTail.isTailStriped());  // false
        
        Primate primate = lemur;
        System.out.println(primate.hasHair());  // true
    }
}
```

**الفكرة هنا إيه؟** إحنا عملنا Object واحد بس من نوع `Lemur`، بس قدرنا نشاور عليه بـ 3 أنواع مختلفة: `Lemur` و `HasTail` و `Primate`.

---

## الفرق بين الـ Object والـ Reference

### القاعدة الذهبية

الـ Object في الذاكرة **مابيتغيرش**! اللي بيتغير هو الـ Reference اللي بنشاور بيه على الـ Object.

فكر فيها كده: عندك صاحبك أحمد (ده الـ Object)، ممكن تناديه بـ "أحمد" أو "دكتور أحمد" أو "باشمهندس" (دي الـ References). أحمد نفسه مابيتغيرش، بس حسب إنت بتناديه بإيه، هتقدر تطلب منه حاجات معينة.

### القواعد المهمة

| القاعدة | الشرح |
|---------|-------|
| نوع الـ Object | بيحدد إيه الـ properties اللي موجودة في الذاكرة |
| نوع الـ Reference | بيحدد إيه الـ methods والـ variables اللي تقدر توصلها |

### مثال على المشكلة

```java
HasTail hasTail = new Lemur();
System.out.println(hasTail.age);  // ❌ DOES NOT COMPILE

Primate primate = new Lemur();
System.out.println(primate.isTailStriped());  // ❌ DOES NOT COMPILE
```

**ليه مش بيشتغل؟**

- لما شاورت على الـ Lemur بـ Reference من نوع `HasTail`، الـ compiler بيقولك "أنا مش شايف غير اللي في الـ Interface دي"
- الـ `age` موجود في الـ Object في الذاكرة، بس الـ Reference مش شايفه!

---

## الـ Casting (التحويل)

### أنواع الـ Casting

#### 1. Implicit Cast (من تحت لفوق) ✅
من الـ Subtype للـ Supertype - **مش محتاج cast**

```java
Lemur lemur = new Lemur();
Primate primate = lemur;  // تمام، مش محتاج cast
```

#### 2. Explicit Cast (من فوق لتحت) ⚠️
من الـ Supertype للـ Subtype - **لازم cast**

```java
Primate primate = new Lemur();
Lemur lemur2 = (Lemur)primate;  // لازم الـ cast
Lemur lemur3 = primate;  // ❌ DOES NOT COMPILE - فين الـ cast؟
```

### قواعد الـ Casting

| القاعدة | التوضيح |
|---------|---------|
| من Subtype لـ Supertype | مش محتاج cast |
| من Supertype لـ Subtype | لازم explicit cast |
| Cast غلط وقت الـ Runtime | هيرمي `ClassCastException` |
| Cast لنوع ملوش علاقة | الـ Compiler هيرفض من الأول |

### مثال على Cast مرفوض

```java
public class Bird {}
public class Fish {
    public static void main(String[] args) {
        Fish fish = new Fish();
        Bird bird = (Bird)fish;  // ❌ DOES NOT COMPILE
    }
}
```

**ليه؟** لأن `Fish` و `Bird` ملهمش أي علاقة ببعض!

### حالة خاصة: الـ Interfaces

الـ Compiler بيبقى أكثر تساهلاً مع الـ Interfaces لأن ممكن Subclass يـ implement الـ Interface حتى لو الـ Parent مش بيعمل كده.

```java
interface Canine {}
interface Dog {}
class Wolf implements Canine {}

public class BadCasts {
    public static void main(String[] args) {
        Wolf wolfy = new Wolf();
        Dog badWolf = (Dog)wolfy;  // ✅ Compiles بس ❌ ClassCastException at runtime!
    }
}
```

**الاستثناء:** لو الـ Class معمولها `final`، الـ Compiler يقدر يرفض:

```java
final class Wolf implements Canine {}
// دلوقتي الـ cast لـ Dog مش هيـ compile لأن مفيش subclass ممكن يـ implement Dog
```

---

## الـ instanceof Operator

بنستخدمه عشان نتأكد إن الـ Object من نوع معين قبل ما نعمل Cast، وده بيحمينا من الـ `ClassCastException`.

### مثال على المشكلة

```java
class Rodent {}
public class Capybara extends Rodent {
    public static void main(String[] args) {
        Rodent rodent = new Rodent();
        var capybara = (Capybara)rodent;  // 💥 ClassCastException!
    }
}
```

### الحل باستخدام instanceof

```java
if (rodent instanceof Capybara c) {
    // دلوقتي تقدر تستخدم c بأمان
}
```

---

## الـ Method Overriding والـ Polymorphism

### القاعدة المهمة جداً

لما تعمل Override لـ method، **كل** الـ calls ليها هتروح للـ version الجديدة، حتى لو الـ call جاي من الـ Parent class!

### مثال مهم

```java
class Penguin {
    public int getHeight() { return 3; }
    public void printInfo() {
        System.out.print(this.getHeight());
    }
}

public class EmperorPenguin extends Penguin {
    public int getHeight() { return 8; }
    
    public static void main(String[] fish) {
        new EmperorPenguin().printInfo();  // بيطبع 8 مش 3!
    }
}
```

**ليه طبع 8؟** لأن الـ Object في الذاكرة هو `EmperorPenguin`، فلما `printInfo()` نادت `getHeight()`، راحت للـ overridden version.

### لو عايز تنادي الـ Parent version

استخدم `super` في الـ Child class:

```java
public class EmperorPenguin extends Penguin {
    public int getHeight() { return 8; }
    
    public void printInfo() {
        System.out.print(super.getHeight());  // هيطبع 3
    }
}
```

---

## الفرق بين Overriding و Hiding

### جدول المقارنة

| الخاصية | Overriding | Hiding |
|---------|------------|--------|
| بيحصل في | Instance methods | Static methods و Variables |
| بيعتمد على | نوع الـ Object | نوع الـ Reference |
| Polymorphism | ✅ أيوه | ❌ لأ |

### مثال عملي مهم جداً

```java
class Marsupial {
    protected int age = 2;
    public static boolean isBiped() {
        return false;
    }
}

public class Kangaroo extends Marsupial {
    protected int age = 6;
    public static boolean isBiped() {
        return true;
    }
    
    public static void main(String[] args) {
        Kangaroo joey = new Kangaroo();
        Marsupial moey = joey;
        
        System.out.println(joey.isBiped());   // true
        System.out.println(moey.isBiped());   // false
        System.out.println(joey.age);          // 6
        System.out.println(moey.age);          // 2
    }
}
```

**إيه اللي حصل؟**

- عندنا Object **واحد** بس من نوع `Kangaroo`
- الـ static method `isBiped()` **hidden مش overridden**، فالنتيجة بتعتمد على نوع الـ Reference
- الـ variable `age` برضو **hidden**، فنفس الكلام

### نصيحة عملية ⚠️

**ماتعملش Hide لـ members في الكود بتاعك!** ده بيخلي الكود صعب جداً في الفهم والصيانة. لو عندك variable أو static method في الـ Parent، اختار اسم مختلف في الـ Child.

---

## ملخص سريع للامتحان 📝

1. **Polymorphism**: الـ Object الواحد ممكن يتشاور عليه بأنواع Reference مختلفة
2. **الـ Object مابيتغيرش**: اللي بيتغير هو الـ Reference بس
3. **Casting لفوق**: مش محتاج explicit cast
4. **Casting لتحت**: لازم explicit cast
5. **instanceof**: استخدمه قبل الـ casting عشان تتجنب الـ ClassCastException
6. **Method Overriding**: بيعتمد على نوع الـ Object
7. **Member Hiding**: بيعتمد على نوع الـ Reference
8. **ماتعملش Hide**: سيئة برمجياً!

---

## رسم توضيحي 🎨

```
┌─────────────────────────────────────────────────────────┐
│                    الذاكرة (Memory)                      │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Lemur Object                        │   │
│  │  ┌─────────────────────────────────────────┐   │   │
│  │  │  من Primate: hasHair()                   │   │   │
│  │  │  من HasTail: isTailStriped()             │   │   │
│  │  │  من Lemur: age = 10                      │   │   │
│  │  └─────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                         ▲
         ┌───────────────┼───────────────┐
         │               │               │
    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
    │ Lemur   │    │ HasTail │    │ Primate │
    │Reference│    │Reference│    │Reference│
    └─────────┘    └─────────┘    └─────────┘
    شايف كل     شايف            شايف
    حاجة        isTailStriped   hasHair
               بس              بس
```

**خلاص كده! بالتوفيق في الامتحان! 🚀**
