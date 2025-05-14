
# ⚙️ Spring Framework Runtime شرح مبسط

الـ **Spring Runtime** هو البيئة اللي بيشتغل فيها التطبيق بتاعك وقت التشغيل، وبيضم حاجات كتير زي الـ IoC Container, Bean Lifecycle, AOP, Event Handling، إلخ.

---

## ✅ المكونات الأساسية في Spring Runtime

| المكون | الشرح بالمصري |
|--------|----------------|
| IoC Container | المسؤول عن إدارة الـ Beans: ينشئهم، يحقنهم، ويتحكم في دورة حياتهم |
| Bean Lifecycle | سلسلة خطوات الـ Bean بيمر بيها من أول ما يتخلق لحد ما يتدمّر |
| Dependency Injection | آلية Spring في إنه يحقن الاعتماديات تلقائيًا في الكلاسات |
| ApplicationContext | النسخة المتطورة من BeanFactory، وبتشغل كل حاجة Runtime |
| AOP (Aspect Oriented Programming) | تقدر تحقن لوجيك إضافي (زي Logging أو Security) من غير ما تعدل الكود الأساسي |
| Events & Listeners | Spring بيسمحلك تبعت Events وتستقبلها بـ Listeners زي ما بتحصل حاجات في التطبيق |

---

## 🧪 مثال عملي على Runtime

### 1. Bean وتعريفه
```java
@Component
public class EmployeeService {
    public void print() {
        System.out.println("خدمة الموظف شغالة");
    }
}
```

### 2. Bean بتستخدم Bean تاني (Dependency Injection)
```java
@Component
public class Payroll {

    private final EmployeeService employeeService;

    @Autowired
    public Payroll(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    public void run() {
        employeeService.print();
    }
}
```

### 3. التشغيل في الـ Runtime
```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(App.class, args);
        Payroll payroll = context.getBean(Payroll.class);
        payroll.run(); // هيطبع: خدمة الموظف شغالة
    }
}
```

---

## 🔁 Bean Lifecycle مراحل دورة حياة الـ Bean

1. Spring ينشئ الـ Bean (Constructor)
2. يحقن الاعتماديات (@Autowired)
3. ينفذ `@PostConstruct` لو فيه
4. يشتغل الكود بتاعك عادي
5. قبل ما التطبيق يقفل، ينفذ `@PreDestroy`

---

## 🌀 AOP مثال سريع

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.*.*(..))")
    public void logBefore() {
        System.out.println("قبل ما الميثود تشتغل");
    }
}
```

---

## 📣 Event Example

```java
@Component
public class MyListener {

    @EventListener
    public void handleContextRefresh(ContextRefreshedEvent event) {
        System.out.println("التطبيق اشتغل!");
    }
}
```

---

> 💡 **ملحوظة:** كل المكونات دي بتشتغل في Runtime أوتوماتيك، بمجرد ما Spring Boot يشغّل الـ Application.

