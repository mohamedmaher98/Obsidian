# Spring Core — ملخص الشرح بالتفصيل (IoC / DI / Factory / Container / Annotations)

> مكتوب بالعمايه المصريه + أمثلة Java عشان تثبت الفهم **مش حفظ**.

---

## 0) الفكرة اللي بدأنا منها
كتير مننا وهو بيكتب Java بيعمل كده:

```java
class OrderService {
    private PaymentService paymentService = new PaymentService(); // new جوا الكلاس
}
```

ده معناه إن `OrderService` ماسك **مسؤوليتين**:
1) شغله الأساسي (Orders)  
2) إنه يخلق `PaymentService` بنفسه  

وده بيعمل مشاكل: ربط جامد (Tight Coupling)، صعوبة Test، صعوبة تبديل implementations.

---

## 1) Inversion of Control (IoC) يعني إيه؟
**IoC = عكس التحكم**  
يعني بدل ما الكلاس يتحكم في **إنشاء** اللي محتاجه (dependencies) بنفسه، يخلي حد تاني يديله اللي محتاجه جاهز.

### قبل IoC (مفيش IoC)
الكلاس بيقول:
> أنا محتاج B → يبقى هعمل `new B()`

مثال:

```java
class B {}

class A {
    private B b = new B();  // A هي اللي خلقت B
}
```

ده **مش IoC**.

### بعد IoC (فيه IoC)
الكلاس بيقول:
> أنا محتاج B → اللي هيخلقني يديني B جاهزة

```java
class B {}

class A {
    private final B b;

    public A(B b) {         // A بتاخد B جاهزة
        this.b = b;
    }
}
```

واستخدامه (Manual DI):

```java
B b = new B();
A a = new A(b);
```

**التحكم اتعكس**: إنشاء `B` خرج برا `A`.

---

## 2) Dependency Injection (DI) يعني إيه؟
**DI = أسلوب/طريقة لتنفيذ IoC**  
يعني بدل ما الكلاس يخلق dependencies، إحنا “بنحقنها” له من بره.

> كل DI هو IoC، بس IoC فكرة عامة وممكن يتعمل بطرق تانية.

---

## 3) أنواع DI الثلاثة (بأمثلة العربية/الموتور)

### 3.1) Constructor Injection (الأفضل غالبًا)
**المعنى:** العربية ما ينفعش تتخلق إلا بالموتور.

```java
interface Engine { void start(); }

class DieselEngine implements Engine {
    public void start() { System.out.println("Diesel Engine started"); }
}

class Car {
    private final Engine engine;

    public Car(Engine engine) { // لازم ييجي هنا
        this.engine = engine;
    }

    public void drive() {
        engine.start();
        System.out.println("Car is moving");
    }
}
```

استخدام:

```java
Car car = new Car(new DieselEngine());
car.drive();
```

**ليه الأفضل؟**
- بيجبرك توفر dependency الأساسية
- واضح جدًا إن الكلاس محتاجها
- ممتاز للـ unit testing
- تقدر تخليها `final` (immutability)

> قاعدة: لو الحاجة **أساسية** والكلاس مش هيشتغل من غيرها → Constructor.

---

### 3.2) Setter Injection
**المعنى:** ممكن أعمل العربية وبعدين أركبلها موتور (اختياري/يتغير).

```java
class Car {
    private Engine engine;

    public void setEngine(Engine engine) { // ممكن تتحط بعدين
        this.engine = engine;
    }

    public void drive() {
        if (engine == null) throw new IllegalStateException("No engine!");
        engine.start();
        System.out.println("Car is moving");
    }
}
```

استخدام:

```java
Car car = new Car();
car.setEngine(new DieselEngine());
car.drive();
```

**تستخدمه إمتى؟**
- dependency اختيارية
- أو ممكن تتغير أثناء التشغيل

**عيوبه:**
- ممكن تنسى تعمل set
- ممكن تستخدم الكلاس قبل ما تحط dependency

> قاعدة: لو الحاجة **اختيارية** أو ممكن تتغير → Setter.

---

### 3.3) Field Injection (شائع في Spring.. بس مش مفضل)
**المعنى:** Spring “يحط” dependency جوه field مباشرة من غير ما تبينها في constructor أو setter.

```java
@Component
class Car {
    @Autowired
    private Engine engine; // Spring هيحطها هنا
}
```

**هو بيعمل إيه؟**  
Spring بيخلق `Car` وبعدين يعمل injection للـ field “من وراك” كأنه بيعمل:

```java
car.engine = engine;
```

**ليه مش مفضل؟**
- مش واضح dependency من الـ API بتاع الكلاس
- صعب في unit testing بدون Spring
- غالبًا مش immutable

> Best practice في Spring: **Constructor Injection**.

---

## 4) Factory Pattern (ليه بنستخدمه؟)
Factory = “مصنع” بيصنع object بناءً على اختيار/شرط بدل ما تكرر `new + if/else` في كل مكان.

### المشكلة من غير Factory
```java
Engine engine;
if (type.equals("diesel")) engine = new DieselEngine();
else if (type.equals("electric")) engine = new ElectricEngine();
else engine = new HybridEngine();
```

### الحل: Factory واحدة
```java
class EngineFactory {
    public static Engine create(String type) {
        if (type.equals("diesel")) return new DieselEngine();
        if (type.equals("electric")) return new ElectricEngine();
        return new HybridEngine();
    }
}
```

واستخدام:

```java
Engine engine = EngineFactory.create("diesel");
Car car = new Car(engine); // هنا DI (manual) + Factory
```

### الفرق بين Factory و DI
- **Factory**: بتنقل قرار الإنشاء لمكان واحد (بس انت لسه بتستدعي المصنع)
- **DI/IoC**: بتشيل مسؤولية الإنشاء من الكلاس خالص (والقرار غالبًا يبقى عند Container زي Spring)

---

## 5) Spring Container (ApplicationContext)
Spring Container هو “المدير” اللي:
- يخلق الـ objects (Beans)
- يربطهم ببعض (DI)
- يدير lifecycle
- يدير scopes

أشهر Container في Spring: **ApplicationContext**

### يعني إيه Bean؟
أي object Spring بيخلقه ويديره اسمه **Bean**.

### Spring بيعمل إيه عند تشغيل التطبيق؟
1) يعمل scan على الكلاسات اللي عليها:
   - `@Component`, `@Service`, `@Repository`, `@Controller`, `@RestController`
2) يخلق instances
3) يشوف dependencies في constructors
4) يربطهم ببعض
5) يخزنهم داخل ApplicationContext

### getBean (فكرة)
```java
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
Car car = ctx.getBean(Car.class);
```

### Singleton (الافتراضي)
غالبًا نفس الـ bean بيتعمل مرة واحدة للتطبيق كله (Singleton scope).

---

## 6) لو فيه كذا Bean من نفس النوع (Ambiguity)
لو عندك:

```java
@Component class DieselEngine implements Engine {}
@Component class ElectricEngine implements Engine {}
```

و `Car` محتاجة `Engine`:

```java
public Car(Engine engine) { ... }
```

هنا Spring مش هيعرف يختار مين → يحصل مشكلة (Ambiguous beans)

**حلول شائعة:**
- `@Primary` على implementation الافتراضي
- أو `@Qualifier("beanName")` لتحديد المطلوب
- أو حقن `Map<String, Engine>` / `List<Engine>` لو عايز كلهم

(الفكرة: لازم تساعد Spring يحدد الاختيار)

---

## 7) الفرق بين @Component و @Service و @Repository و @Controller و @RestController
### الحقيقة الأساسية
كلهم في الأساس **@Component** (يعني كلهم Beans)  
بس بيدّوا معنى + سلوك إضافي بسيط في بعض الحالات.

### 7.1) @Component
- لأي class “عام” مش واضح هو أنهي layer

### 7.2) @Service
- معناها: ده business logic
- غالبًا مفيش سلوك إضافي، بس تنظيم ومعنى

### 7.3) @Repository
- معناها: طبقة الداتا/الداتابيز
- **ميزة مهمة:** Spring يعمل exception translation
  (يحّول exceptions بتاعة DB لـ DataAccessException)

### 7.4) @Controller
- طبقة الويب (Spring MVC)
- غالبًا بيرجع View (صفحات HTML) في MVC التقليدي

### 7.5) @RestController
- اختصار لـ: `@Controller + @ResponseBody`
- يعني بيرجع JSON مباشرة (Web APIs)

جدول سريع:

| Annotation | Layer | بيرجع إيه | ميزة إضافية |
|---|---|---|---|
| `@Component` | عام | — | لا |
| `@Service` | Business | — | معنى تنظيمي |
| `@Repository` | Data | — | Exception Translation |
| `@Controller` | Web MVC | View | سلوك MVC |
| `@RestController` | Web API | JSON | ResponseBody تلقائي |

---

## 8) Mini E-commerce مثال (نطبق اختيار نوع DI)
### Interfaces
```java
interface PaymentProcessor { void pay(double amount); }
interface ProductRepository { Product findById(Long id); }
```

### Implementations
```java
class PaypalProcessor implements PaymentProcessor {
    public void pay(double amount) { System.out.println("PayPal: " + amount); }
}
```

### Services
**الحاجات الأساسية** → Constructor  
**الاختيارية** → Setter

```java
class DiscountService {
    double applyDiscount(double amount) { return amount * 0.9; }
}

class NotificationService {
    void sendConfirmation() { System.out.println("Confirmation sent"); }
}

class OrderService {

    private final ProductRepository productRepository;    // أساسي
    private final PaymentProcessor paymentProcessor;      // أساسي

    private DiscountService discountService;              // اختياري
    private NotificationService notificationService;      // اختياري

    public OrderService(ProductRepository productRepository,
                        PaymentProcessor paymentProcessor) {
        this.productRepository = productRepository;
        this.paymentProcessor = paymentProcessor;
    }

    public void setDiscountService(DiscountService discountService) {
        this.discountService = discountService;
    }

    public void setNotificationService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void placeOrder(Long productId) {
        Product product = productRepository.findById(productId);
        double price = product.getPrice();

        if (discountService != null) {
            price = discountService.applyDiscount(price);
        }

        paymentProcessor.pay(price);

        if (notificationService != null) {
            notificationService.sendConfirmation();
        }
    }
}
```

---

## 9) قواعد سريعة (تثبيت الفهم)
- لو شايف `new` لdependency جوه الكلاس → غالبًا **مش IoC**
- لو الكلاس بياخد dependency جاهزة من بره → **IoC**
- dependency أساسية → **Constructor Injection**
- dependency اختيارية/بتتغير → **Setter Injection**
- Field Injection سهل بس مش أفضل practice → خليك في Constructor

---

## 10) المصطلحات في سطر واحد
- **IoC**: التحكم في إنشاء dependencies خرج برا الكلاس  
- **DI**: طريقة حقن dependencies للكلاس  
- **Factory**: مكان مركزي يقرر ويخلق implementation مناسب  
- **ApplicationContext**: الـ Container اللي بيدير Beans و DI في Spring  

---
