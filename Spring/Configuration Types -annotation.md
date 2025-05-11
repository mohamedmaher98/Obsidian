 1️⃣ XML-based Configuration

```java
<beans xmlns="http://www.springframework.org/schema/beans">
    <bean id="employee" class="com.example.Employee"/>
</beans>


// in main class 
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
Employee emp = context.getBean("employee", Employee.class);

```
2️⃣ Annotation-based Configuration (الطريقة الحديثة)

```java
@Component
public class Employee {
}

// in main class 
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
Employee emp = context.getBean(Employee.class);

//in appConfig 
@ComponentScan("com.example")
@Configuration
public class AppConfig {
}
```

==3 - Java-based Configuration (الأكتر مرونة)==

```java 
@Configuration
public class AppConfig {

    @Bean
    public Employee employee() {
        return new Employee();
    }
}

// in Main class 
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
Employee emp = context.getBean(Employee.class);

```

## 🛠 Spring Configuration Types

| النوع | الوصف | مثال |
|------|--------|------|
| **XML-based Configuration** | تعريف Beans في ملفات XML | `<bean id="..." class="..."/>` |
| **Annotation-based Configuration** | استخدام أنوتيشن زي `@Component`, `@Autowired`, `@ComponentScan` | `@Component`, `@Configuration`, `@Autowired` |
| **Java-based Configuration** | تعريف Beans باستخدام `@Bean` في كلاس عليه `@Configuration` | `@Configuration`, `@Bean` |

---
### ==**Annotation Types**== 


## 🧩 أنواع الأنوتيشن في Spring

### ✅ Component Annotations
| Annotation | الاستخدام |
|-----------|------------|
| `@Component` | تعريف Bean عام |
| `@Service` | منطق البزنس |
| `@Repository` | طبقة البيانات |
| `@Controller` | الـ MVC Controller |
| `@RestController` | API Controller (JSON/XML) |

---

### ✅ Injection Annotations
| Annotation | الاستخدام |
|-----------|------------|
| `@Autowired` | حقن تلقائي |
| `@Qualifier("name")` | تحديد Bean معين |
| `@Inject` | بديل لـ Autowired |
| `@Value("${key}")` | حقن قيمة من خصائص |

---

### ✅ Configuration Annotations
| Annotation | الاستخدام |
|-----------|------------|
| `@Configuration` | كلاس إعدادات |
| `@Bean` | تعريف Bean يدوي |
| `@ComponentScan` | تحديد مكان Beans |
| `@PropertySource` | تحميل ملف خصائص خارجي |

---

### ✅ Lifecycle Annotations
| Annotation | الاستخدام |
|-----------|------------|
| `@PostConstruct` | بعد الإنشاء |
| `@PreDestroy` | قبل التدمير |

---

### ✅ أخرى مهمة
| Annotation | الاستخدام |
|-----------|------------|
| `@Transactional` | المعاملات |
| `@EnableAutoConfiguration` | إعداد تلقائي في Spring Boot |
| `@SpringBootApplication` | Main Anno في Spring Boot |

---
##### ==🧩 أولًا: Component Annotations (أنوتيشنات تعريف Beans)==

|Annotation|الشرح بالمصري|
|---|---|
|`@Component`|دي معناها إن الكلاس ده Spring هيستخدمه كـ Bean، يعني هينشئ منه object ويديره بنفسه. كأنك بتقوله: "اعتبر الكلاس ده من عندك".|
|`@Service`|نفس فكرة `@Component`، بس معمولة مخصوص للكلاسات اللي فيها منطق البزنس (Business Logic). يعني خدمات، حسابات، تعاملات... إلخ.|
|`@Repository`|دي للكلاسات اللي بتتعامل مع الداتا (يعني بتوصل بقاعدة البيانات). Spring بيفهم إن الكلاس ده هيبقى مسؤول عن الـ DAO.|
|`@Controller`|بتستخدمها في تطبيقات الويب علشان تعرف الكلاس اللي هيتعامل مع طلبات الـ HTTP ويرجع صفحات HTML.|
|`@RestController`|زي `@Controller` بالظبط، لكن بترجع بيانات (زي JSON) بدل صفحات، وبتستخدمها في REST APIs.|
##### **==🧩 تانيًا: Injection Annotations (حقن الاعتماديات – DI)==**
| Annotation               | الشرح                                                                                                                              |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| `@Autowired`             | دي أهم أنوتيشن في الحقن (DI). معناها: "يا Spring، هاتلي نسخة من الـ Bean ده وحطها هنا". بتحقن object من نوع معين جوه كلاس تاني.    |
| `@Qualifier("beanName")` | لو عندك أكتر من Bean من نفس النوع، وSpring محتار يختار أنهي واحد، ساعتها بتستخدم `@Qualifier` عشان تحددله اسم الـ Bean اللي عايزه. |
| `@Inject`                | دي زي `@Autowired` بالظبط، بس جاية من Java نفسها (javax). مش مشهورة أوي في Spring، بس ينفع تستخدمها.                               |
| `@Value("${key}")`       | بتستخدمها لو عايز تحقن قيمة من ملف الخصائص (properties file)، زي URL أو كلمة سر.                                                   |


##### ==**🧩 ثالثًا: Configuration Annotations (أنوتيشنات الإعدادات)**==

| Annotation                           | الشرح                                                                                              |
| ------------------------------------ | -------------------------------------------------------------------------------------------------- |
| `@Configuration`                     | بتقول لـ Spring إن الكلاس ده فيه إعدادات و Beans. زي ما يكون config file بس بكود Java.             |
| `@Bean`                              | بتستخدمها جوه كلاس عليه `@Configuration` عشان تعرف Bean بنفسك، مش عن طريق `@Component`.            |
| `@ComponentScan("package")`          | دي بتقول لـ Spring يروح يبص جوا الباكيج دي ويدوّر على كلاس عليه `@Component`, `@Service`, ... إلخ. |
| `@PropertySource("file.properties")` | بتستخدمها لو عايز تحمل ملف خصائص خارجي وتستخدم القيم اللي فيه مع `@Value`.                         |
##### **==🧩 رابعًا: Lifecycle Annotations (دورة حياة الـ Bean)==**

|Annotation|الشرح بالمصري|
|---|---|
|`@PostConstruct`|دي بتتنفذ **أول ما Spring يخلص بناء الـ Bean** (يعني بعد الـ Constructor). تحط فيها كود تهيئة أو إعداد أولي.|
|`@PreDestroy`|دي بتتنفذ **قبل ما Spring يدمر الـ Bean** (لما التطبيق يقفل). تحط فيها أي كود تنظيف أو حفظ.|

##### ==**🧩 أنوتيشنات مهمة في Spring Boot**==

| Annotation                 | الشرح بالمصري                                                                                                                        |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `@Transactional`           | دي معناها: "شغل الكود اللي جوا دي كمعاملة واحدة". يعني لو حصل خطأ، كل اللي اتعمل يتراجع (Rollback). بتستخدمها مع الداتا.             |
| `@EnableAutoConfiguration` | عSpring Boot بيحاول يظبط نفسه أوتوماتيك بناءً على الـ dependencies اللي عندك، والأنوتيشن دي بتفعّل الخاصية دي.                       |
| `@SpringBootApplication`   | دي بتجمع 3 أنوتيشنات مهمة (`@Configuration`, `@ComponentScan`, `@EnableAutoConfiguration`) في أنوتيشن واحدة، وبتحطها على المين كلاس. |

---

# 🧩 شرح أنوتيشنات Spring مع أمثلة

---

## ✅ Component Annotations

| Annotation | الشرح بالمصري |
|-----------|----------------|
| `@Component` | Spring بيعتبر الكلاس ده Bean ويديره بنفسه |
| `@Service` | Bean خاص بالـ Business Logic |
| `@Repository` | Bean بيتعامل مع قاعدة البيانات (DAO) |
| `@Controller` | بيتعامل مع صفحات الويب والـ HTTP Requests |
| `@RestController` | زي `@Controller` لكن بيرجع JSON (لـ APIs) |

### 🧪 مثال:
```java
@Component
public class MyComponent {}

@Service
public class EmployeeService {}

@Repository
public class EmployeeRepository {}

@Controller
public class WebController {}

@RestController
public class ApiController {}
```

---

## ✅ Injection Annotations

| Annotation | الشرح بالمصري |
|-----------|----------------|
| `@Autowired` | Spring يحقن Bean تلقائيًا (حسب النوع) |
| `@Qualifier("name")` | تحدد أي Bean يتحقن لو فيه أكتر من واحد |
| `@Inject` | زي `@Autowired` لكنها جاية من Java |
| `@Value("${key}")` | تحقن قيمة من ملف خصائص (properties) |

### 🧪 مثال:
```java
@Service
public class EmployeeService {
    
    @Autowired
    private EmployeeRepository repo;

    @Autowired
    @Qualifier("specialBean")
    private MyBean special;

    @Value("${app.name}")
    private String appName;
}
```

---

## ✅ Configuration Annotations

| Annotation | الشرح بالمصري |
|-----------|----------------|
| `@Configuration` | كلاس إعدادات فيه Beans |
| `@Bean` | تعريف Bean يدويًا |
| `@ComponentScan("package")` | يدوّر على Beans في باكيج معينة |
| `@PropertySource("file.properties")` | يحمل ملف خصائص خارجي |

### 🧪 مثال:
```java
@Configuration
@ComponentScan("com.example")
@PropertySource("classpath:app.properties")
public class AppConfig {

    @Bean
    public Employee employeeBean() {
        return new Employee();
    }
}
```

---

## ✅ Lifecycle Annotations

| Annotation | الشرح بالمصري |
|-----------|----------------|
| `@PostConstruct` | كود بيتنفذ بعد ما الـ Bean يتجهز |
| `@PreDestroy` | كود بيتنفذ قبل ما الـ Bean يتدمّر |

### 🧪 مثال:
```java
@Component
public class InitExample {

    @PostConstruct
    public void init() {
        System.out.println("Bean جاهز!");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Bean بيتقفل");
    }
}
```

---

## ✅ Spring Boot Annotations

| Annotation | الشرح بالمصري |
|-----------|----------------|
| `@Transactional` | تشغيل الكود كـ Transaction (Rollback لو حصل خطأ) |
| `@EnableAutoConfiguration` | Spring Boot يظبط نفسه تلقائيًا |
| `@SpringBootApplication` | بتجمع `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan` |

### 🧪 مثال:
```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}

@Service
@Transactional
public class OrderService {
    public void createOrder() {
        // كل العملية دي تعتبر Transaction
    }
}
```

---

> 📝 **نصيحة:** الأنوتيشنز بتخلي Spring ذكي جدًا في التعامل مع الكلاسات، فاهم هو هيستخدم إيه وإزاي. لما تفهم كل أنوتيشن وإمتى تستخدمها، بتكتب كود أنضف وأوضح.
