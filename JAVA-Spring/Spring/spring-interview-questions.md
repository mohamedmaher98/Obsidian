# Spring Framework - أسئلة الانترفيو الشاملة

## المستوى الأول: الأساسيات (Junior Level)

### 1. إيه هو Spring Framework وليه بنستخدمه؟

**الإجابة المتوقعة:**
Spring هو framework لـ Java بيوفر بنية تحتية شاملة لبناء تطبيقات enterprise. أهم ميزة فيه هي **Inversion of Control (IoC)** اللي بتخلي Spring يدير إنشاء الـ objects وربطها ببعض بدل ما المبرمج يعمل كده يدوي.

الفوائد الرئيسية:
- **Loose Coupling** عن طريق Dependency Injection
- **Modularity** - تقدر تستخدم الأجزاء اللي محتاجها بس
- **Testing سهل** - تقدر تبدل أي component بـ mock
- **Community كبير** ودعم واسع

---

### 2. إيه الفرق بين Spring و Spring Boot؟

**الإجابة المتوقعة:**

| Spring | Spring Boot |
|--------|-------------|
| محتاج configuration كتير | Auto-configuration |
| محتاج تختار وتضبط server | Server مدمج (Tomcat/Jetty) |
| محتاج تعمل XML أو Java config | Convention over configuration |
| مرونة كاملة | رأي افتراضي (opinionated) مع إمكانية التغيير |

Spring Boot هو طبقة فوق Spring بتقلل الـ boilerplate code. بدل ما تكتب 100 سطر configuration، Spring Boot بيعمل auto-configuration بناءً على الـ dependencies اللي في الـ classpath.

**سؤال متابعة محتمل:** "إيه اللي `@SpringBootApplication` بتعمله؟"
- `@Configuration` - الكلاس ده مصدر للـ Bean definitions
- `@EnableAutoConfiguration` - فعّل الـ auto-configuration
- `@ComponentScan` - دور على الـ components في الـ package الحالي وتحته

---

### 3. إيه هو Dependency Injection وإيه أنواعه؟

**الإجابة المتوقعة:**
DI يعني إن الـ object مش بينشئ الـ dependencies بتاعته بنفسه، بل بيستقبلها من مصدر خارجي (الـ Spring Container).

**الأنواع:**

1. **Constructor Injection** (الأفضل):
```java
@Service
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

2. **Setter Injection**:
```java
@Autowired
public void setPaymentService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

3. **Field Injection** (مش مفضلة):
```java
@Autowired
private PaymentService paymentService;
```

**سؤال متابعة:** "ليه Constructor Injection أحسن؟"
- الـ dependencies بتبقى `final` (immutable)
- مستحيل تنشئ object من غير الـ dependencies المطلوبة
- أسهل في الـ unit testing
- بتوضح الـ dependencies بشكل صريح

---

### 4. إيه الفرق بين `@Component` و `@Service` و `@Repository` و `@Controller`؟

**الإجابة المتوقعة:**
كلهم بيعملوا نفس الحاجة الأساسية: بيسجلوا الـ class كـ Bean في Spring Container. الفرق هو **الدلالة (semantics)**:

- `@Component` - أي class عام
- `@Service` - business logic layer
- `@Repository` - data access layer (وبيضيف ترجمة تلقائية لـ database exceptions)
- `@Controller` - web layer (بيستقبل HTTP requests ويرجع views)
- `@RestController` = `@Controller` + `@ResponseBody` (بيرجع JSON/XML مباشرة)

---

### 5. إيه هو Bean Scope وإيه أنواعه؟

**الإجابة المتوقعة:**

| Scope | الشرح |
|-------|-------|
| `singleton` (الافتراضي) | instance واحد في كل الـ application context |
| `prototype` | instance جديد كل مرة يتطلب |
| `request` | instance جديد لكل HTTP request (web فقط) |
| `session` | instance جديد لكل HTTP session (web فقط) |
| `application` | instance واحد لكل ServletContext |

**سؤال متابعة:** "لو Singleton Bean بيعتمد على Prototype Bean، إيه المشكلة؟"
المشكلة إن الـ Prototype Bean هيتنشئ مرة واحدة بس (وقت إنشاء الـ Singleton). الحل: استخدم `@Lookup` أو `ObjectFactory<T>` أو `Provider<T>`.

---

### 6. إيه الفرق بين `@Autowired` و `@Qualifier` و `@Primary`؟

**الإجابة المتوقعة:**

- `@Autowired` - بتقول لـ Spring "حقن الـ dependency هنا"
- `@Qualifier("name")` - لما يكون فيه أكتر من implementation لنفس الـ interface، بتحدد مين فيهم
- `@Primary` - بتحدد الـ implementation الافتراضي

```java
public interface NotificationService { }

@Service
@Primary  // ده الافتراضي
public class EmailNotification implements NotificationService { }

@Service
public class SmsNotification implements NotificationService { }

// استخدام:
@Autowired
private NotificationService service; // هياخد EmailNotification (لأنها @Primary)

@Autowired
@Qualifier("smsNotification")
private NotificationService smsService; // هياخد SmsNotification تحديداً
```

---

### 7. إيه هو `@Configuration` و `@Bean`؟

**الإجابة المتوقعة:**

`@Configuration` بتقول إن الـ class ده مصدر لتعريف Beans.
`@Bean` بتتحط على method جوه `@Configuration` class وبترجع object هيتسجل كـ Bean.

```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

بنستخدمها لما:
- الـ class من library خارجية (ما نقدرش نحط عليه `@Component`)
- محتاجين تعمل initialization معين
- محتاجين تتحكم في طريقة الإنشاء

---

## المستوى التاني: متوسط (Mid-Level)

### 8. اشرح الـ Spring Bean Lifecycle بالتفصيل

**الإجابة المتوقعة:**

```
1. Instantiation     → Spring بينشئ الـ object
2. Populate Props    → بيحقن الـ dependencies
3. BeanNameAware     → بيبلغ الـ Bean باسمه
4. BeanFactoryAware  → بيدي الـ Bean reference للـ BeanFactory
5. ApplicationContextAware → بيدي الـ Bean reference للـ ApplicationContext
6. @PostConstruct    → initialization logic
7. InitializingBean.afterPropertiesSet()
8. Custom init-method
  ────── الـ Bean شغال ──────
9. @PreDestroy       → cleanup logic
10. DisposableBean.destroy()
11. Custom destroy-method
```

الأهم عملياً: `@PostConstruct` و `@PreDestroy`.

---

### 9. اشرح `@Transactional` بالتفصيل

**الإجابة المتوقعة:**

`@Transactional` بتخلي Spring يلف الـ method في database transaction. لو الـ method نجح، بيعمل `commit`. لو رمى exception، بيعمل `rollback`.

**النقاط المهمة:**
- بالافتراضي بتعمل rollback لـ **unchecked exceptions** (RuntimeException) بس
- لازم تكون على **public method**
- **مش بتشتغل** لو ناديت method من جوه نفس الـ class (بسبب الـ Proxy)
- ممكن تكون على الـ class (بتطبق على كل الـ methods)

```java
@Transactional(
    readOnly = true,                          // تحسين أداء للقراءة
    timeout = 30,                             // timeout بالثواني
    rollbackFor = Exception.class,            // rollback لكل الـ exceptions
    isolation = Isolation.READ_COMMITTED,     // مستوى العزل
    propagation = Propagation.REQUIRED        // سلوك التداخل
)
```

**سؤال متابعة كلاسيكي:** "ليه `@Transactional` مش بتشتغل لو ناديت method من نفس الـ class؟"
لأن Spring بيستخدم **Proxy pattern**. النداء الداخلي بيتجاوز الـ Proxy ويروح للـ method مباشرة.

---

### 10. إيه هي الـ Transaction Propagation Types؟

**الإجابة المتوقعة:**

| Type | الشرح |
|------|-------|
| `REQUIRED` (الافتراضي) | استخدم transaction موجودة، أو اعمل واحدة جديدة |
| `REQUIRES_NEW` | دايماً اعمل transaction جديدة (علّق الحالية) |
| `SUPPORTS` | لو فيه transaction استخدمها، لو مفيش اشتغل من غيرها |
| `NOT_SUPPORTED` | اشتغل من غير transaction (علّق الحالية لو فيه) |
| `MANDATORY` | لازم يكون فيه transaction موجودة، غير كده ارمي exception |
| `NEVER` | لازم ما يكونش فيه transaction، غير كده ارمي exception |
| `NESTED` | اعمل savepoint جوه الـ transaction الحالية |

**مثال عملي:**
```java
@Transactional
public void processOrder(Order order) {
    orderRepository.save(order);  // جزء من الـ transaction الرئيسية

    try {
        auditService.logAction(order);  // REQUIRES_NEW - transaction منفصلة
    } catch (Exception e) {
        // لو الـ audit فشل، الـ order مش هيتأثر
    }
}

@Service
public class AuditService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logAction(Order order) { ... }
}
```

---

### 11. إيه الفرق بين JPA و Hibernate و Spring Data JPA؟

**الإجابة المتوقعة:**

- **JPA** (Jakarta Persistence API): specification/معيار بيحدد إزاي نربط Java objects بـ database. هو مجرد interfaces وAnnotations.
- **Hibernate**: implementation للـ JPA. هو اللي بيعمل الشغل الفعلي.
- **Spring Data JPA**: طبقة فوق JPA/Hibernate بتسهل الاستخدام. بتوفر Repository interfaces جاهزة وبتولد queries من أسماء الـ methods.

```
Spring Data JPA  →  يستخدم  →  JPA (specification)  →  ينفذه  →  Hibernate (implementation)
```

---

### 12. اشرح الـ N+1 Problem وإزاي تحلها

**الإجابة المتوقعة:**

**المشكلة:**
لو عندك 100 Category وكل واحدة فيها Products:
```java
List<Category> categories = categoryRepository.findAll(); // Query 1: SELECT * FROM categories
for (Category cat : categories) {
    cat.getProducts(); // 100 queries إضافية! كل واحدة SELECT * FROM products WHERE category_id = ?
}
// المجموع: 1 + 100 = 101 query!
```

**الحلول:**

1. **JOIN FETCH في JPQL:**
```java
@Query("SELECT c FROM Category c JOIN FETCH c.products")
List<Category> findAllWithProducts();
// query واحدة بس: SELECT c.*, p.* FROM categories c JOIN products p ON ...
```

2. **@EntityGraph:**
```java
@EntityGraph(attributePaths = {"products"})
List<Category> findAll();
```

3. **@BatchSize على الـ Entity:**
```java
@OneToMany
@BatchSize(size = 20)  // بيجيب products لـ 20 category في query واحدة
private List<Product> products;
```

---

### 13. إيه الفرق بين FetchType.LAZY و EAGER؟

**الإجابة المتوقعة:**

- **LAZY**: ما بيجيبش البيانات المرتبطة إلا لما تطلبها فعلاً. ده الأفضل في معظم الحالات.
- **EAGER**: بيجيب كل البيانات المرتبطة مع الـ query الأصلي. ممكن يسبب مشاكل أداء.

```java
@ManyToOne(fetch = FetchType.LAZY)   // ما تجيبش الـ Category إلا لما أنادي getCategory()
private Category category;

@OneToMany(fetch = FetchType.EAGER)  // جيب كل الـ Products دايماً (عادةً سيء)
private List<Product> products;
```

**الافتراضي:**
- `@ManyToOne` و `@OneToOne` → EAGER
- `@OneToMany` و `@ManyToMany` → LAZY

**نصيحة في الانترفيو:** قول إنك دايماً بتستخدم LAZY وبتستخدم JOIN FETCH لما محتاج البيانات.

---

### 14. اشرح Spring Security Filter Chain

**الإجابة المتوقعة:**

كل HTTP request بيعدي على سلسلة من الـ Filters قبل ما يوصل للـ Controller:

```
Request → [SecurityContextPersistenceFilter]
        → [UsernamePasswordAuthenticationFilter]
        → [BasicAuthenticationFilter]
        → [ExceptionTranslationFilter]
        → [FilterSecurityInterceptor]
        → Controller
```

كل Filter ليه دور:
1. **SecurityContextPersistence**: بيحمل الـ SecurityContext من الـ session
2. **Authentication Filter**: بيتحقق من الـ credentials
3. **ExceptionTranslation**: بيحول الـ security exceptions لـ HTTP responses
4. **FilterSecurityInterceptor**: بيتحقق من الـ authorization

---

### 15. إيه الفرق بين Authentication و Authorization؟

**الإجابة المتوقعة:**

- **Authentication** (هوية): مين أنت؟ التحقق من هوية المستخدم (username/password, JWT, OAuth)
- **Authorization** (صلاحيات): إيه اللي مسموحلك تعمله؟ التحقق من الصلاحيات (roles, permissions)

```java
// Authentication: التحقق من الهوية
http.formLogin();  // أو httpBasic() أو JWT

// Authorization: التحقق من الصلاحيات
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
    .anyRequest().authenticated()
);
```

---

### 16. إيه هو AOP واستخداماته في Spring؟

**الإجابة المتوقعة:**

AOP (Aspect Oriented Programming) بيسمحلك تضيف سلوك مشترك (cross-cutting concerns) لأكتر من class من غير ما تعدل فيهم.

**المصطلحات:**
- **Aspect**: الـ class اللي فيه الـ cross-cutting logic
- **Advice**: الكود اللي بيتنفذ (Before, After, Around)
- **Pointcut**: تعبير بيحدد فين الـ Advice يتطبق
- **JoinPoint**: نقطة التنفيذ (عادةً method call)

**الاستخدامات:**
- Logging
- Security (`@PreAuthorize`)
- Transaction management (`@Transactional`)
- Caching (`@Cacheable`)
- Performance monitoring
- Exception handling

Spring بينفذ AOP عن طريق **Proxy pattern** (JDK Dynamic Proxy لـ interfaces، CGLIB لـ classes).

---

## المستوى التالت: متقدم (Senior Level)

### 17. اشرح كيف Spring Auto-Configuration بتشتغل

**الإجابة المتوقعة:**

1. `@EnableAutoConfiguration` بتفعّل الـ mechanism
2. Spring Boot بيقرأ ملف `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
3. كل auto-configuration class بيستخدم `@Conditional` annotations:

```java
@AutoConfiguration
@ConditionalOnClass(DataSource.class)              // لو الـ class موجود في classpath
@ConditionalOnProperty(name = "spring.datasource.url")  // لو الـ property موجود
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean  // بس لو المستخدم ما عرفش Bean بنفسه
    public DataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
    }
}
```

**المفتاح:** `@ConditionalOnMissingBean` - يعني لو أنت عرفت Bean بنفسك، Spring Boot مش هيتدخل.

---

### 18. اشرح Spring Circular Dependency Problem

**الإجابة المتوقعة:**

**المشكلة:**
```java
@Service
public class ServiceA {
    public ServiceA(ServiceB b) { }  // محتاج B
}

@Service
public class ServiceB {
    public ServiceB(ServiceA a) { }  // محتاج A
}
// مين يتنشئ الأول؟ → Exception!
```

**الحلول:**
1. **إعادة التصميم** (الأفضل) - فيه مشكلة في الـ design
2. **@Lazy** - تأخير إنشاء أحدهم:
```java
public ServiceA(@Lazy ServiceB b) { }
```
3. **Setter Injection** بدل Constructor Injection
4. **استخدام Events** بدل النداء المباشر
5. **استخراج الـ shared logic** في class تالت

**ملاحظة مهمة:** Spring Boot 3.x مش بيسمح بـ circular dependencies بالافتراضي.

---

### 19. اشرح الفرق بين الـ `ApplicationContext` و `BeanFactory`

**الإجابة المتوقعة:**

`BeanFactory` هو الـ IoC container الأساسي. `ApplicationContext` بيورث منه وبيضيف ميزات إضافية:

| Feature | BeanFactory | ApplicationContext |
|---------|-------------|-------------------|
| Bean instantiation/wiring | ✅ | ✅ |
| Automatic BeanPostProcessor | ❌ | ✅ |
| Automatic BeanFactoryPostProcessor | ❌ | ✅ |
| MessageSource (i18n) | ❌ | ✅ |
| ApplicationEvent publishing | ❌ | ✅ |
| Environment abstraction | ❌ | ✅ |
| Eager vs Lazy | Lazy بالافتراضي | Eager بالافتراضي |

عملياً دايماً بنستخدم `ApplicationContext`.

---

### 20. إيه هو Spring WebFlux وإيه الفرق بينه وبين Spring MVC؟

**الإجابة المتوقعة:**

| Spring MVC | Spring WebFlux |
|-----------|----------------|
| Synchronous/Blocking | Asynchronous/Non-blocking |
| Thread-per-request | Event loop (Netty) |
| Servlet API | Reactive Streams |
| `@Controller` returns objects | `@Controller` returns `Mono<T>` / `Flux<T>` |
| مناسب لمعظم التطبيقات | مناسب للـ high concurrency / streaming |

```java
// MVC (blocking)
@GetMapping("/products")
public List<Product> getProducts() {
    return productService.findAll(); // بيستنى النتيجة
}

// WebFlux (non-blocking)
@GetMapping("/products")
public Flux<Product> getProducts() {
    return productService.findAll(); // بيرجع stream من البيانات
}
```

---

### 21. اشرح الـ Spring Boot Starter concept

**الإجابة المتوقعة:**

الـ Starters هي مجموعات من الـ dependencies مجمعة مع بعض. بدل ما تضيف 10 dependencies يدوي، بتضيف starter واحد.

```xml
<!-- بدل ما تضيف spring-web, jackson, tomcat, validation كلهم -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- بدل ما تضيف spring-data-jpa, hibernate, HikariCP كلهم -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

الـ Starters الأساسية:
- `spring-boot-starter-web` - Web applications
- `spring-boot-starter-data-jpa` - Database with JPA
- `spring-boot-starter-security` - Security
- `spring-boot-starter-test` - Testing
- `spring-boot-starter-validation` - Bean validation
- `spring-boot-starter-actuator` - Monitoring

---

### 22. إيه هو Spring Cloud وإيه أهم مكوناته؟

**الإجابة المتوقعة:**

Spring Cloud بيوفر أدوات لبناء microservices:

| Component | الوظيفة |
|-----------|---------|
| **Spring Cloud Config** | إدارة الإعدادات مركزياً |
| **Eureka / Consul** | Service Discovery |
| **Spring Cloud Gateway** | API Gateway |
| **Resilience4j** | Circuit Breaker, Rate Limiting |
| **Spring Cloud OpenFeign** | Declarative REST Client |
| **Spring Cloud Stream** | Messaging (Kafka, RabbitMQ) |
| **Spring Cloud Sleuth / Micrometer** | Distributed Tracing |

---

### 23. اشرح كيف تتعامل مع Database Migration في Spring Boot

**الإجابة المتوقعة:**

بنستخدم **Flyway** أو **Liquibase**:

**Flyway:**
```
src/main/resources/db/migration/
├── V1__create_users_table.sql
├── V2__add_email_to_users.sql
└── V3__create_orders_table.sql
```

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    password VARCHAR(255) NOT NULL
);
```

```properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
```

الفكرة: كل ملف بيتنفذ مرة واحدة بس بالترتيب. Flyway بيتتبع إيه اللي اتنفذ.

---

### 24. اشرح الـ Caching Abstraction في Spring

**الإجابة المتوقعة:**

```java
@EnableCaching  // في الـ Configuration

@Cacheable("products")         // خزّن النتيجة - لو اتنادى تاني بنفس الـ parameters، ارجع من الـ cache
@CachePut("products")          // حدّث الـ cache
@CacheEvict("products")        // امسح من الـ cache
@CacheEvict(value = "products", allEntries = true)  // امسح كل الـ cache
```

**Cache Providers:**
- Simple (ConcurrentHashMap - الافتراضي)
- Caffeine (أفضل in-memory)
- Redis (distributed)
- EhCache

**سؤال متابعة:** "إيه المشكلة لو الـ cached data اتغيرت من مصدر تاني؟"
الـ cache هتبقى outdated. لازم تستخدم TTL (Time To Live) أو distributed cache events.

---

### 25. اشرح الـ Spring Profiles بالتفصيل

**الإجابة المتوقعة:**

Profiles بتسمحلك يكون عندك إعدادات مختلفة لبيئات مختلفة:

```yaml
# application.yml (الإعدادات المشتركة)
app:
  name: MyApp

---
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
logging:
  level:
    root: DEBUG

---
# application-prod.yml
spring:
  datasource:
    url: jdbc:mysql://prod-server/mydb
logging:
  level:
    root: WARN
```

**تفعيل Profile:**
```bash
java -jar app.jar --spring.profiles.active=prod
# أو
export SPRING_PROFILES_ACTIVE=prod
```

**Conditional Beans:**
```java
@Bean
@Profile("dev")
public DataSource devDataSource() { ... }

@Bean
@Profile("prod")
public DataSource prodDataSource() { ... }
```

---

## أسئلة Scenario-Based (السيناريوهات العملية)

### 26. لو عندك REST API بطيء، إيه الخطوات اللي هتعملها؟

**الإجابة المتوقعة:**

1. **حدد الـ bottleneck**: استخدم Actuator metrics أو APM tool
2. **Database queries**:
   - تحقق من N+1 problem
   - أضف indexes
   - استخدم pagination
   - استخدم `@Query` بدل method names لـ complex queries
3. **Caching**: أضف `@Cacheable` للبيانات اللي ما بتتغيرش كتير
4. **Async processing**: استخدم `@Async` للعمليات اللي مش لازم تكون في نفس الـ request
5. **Connection pooling**: تأكد إن HikariCP متظبط
6. **Response optimization**: استخدم DTOs بدل ما ترجع الـ entity كامل

---

### 27. إزاي تتعامل مع exceptions في Spring Boot REST API؟

**الإجابة المتوقعة:**

```java
// 1. Custom Exceptions
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

// 2. Global Exception Handler
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        return new ErrorResponse("VALIDATION_FAILED", errors);
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        return new ErrorResponse("INTERNAL_ERROR", "Something went wrong");
    }
}

// 3. Error Response DTO
public record ErrorResponse(String code, Object message) { }
```

---

### 28. إزاي تأمن REST API بـ JWT؟

**الإجابة المتوقعة:**

الخطوات:
1. المستخدم بيبعت `POST /auth/login` بـ username/password
2. السيرفر بيتحقق وبيرجع JWT token
3. المستخدم بيبعت الـ token في كل request: `Authorization: Bearer <token>`
4. Filter بيتحقق من الـ token ويحط الـ user في SecurityContext

المكونات:
- **JwtTokenProvider**: إنشاء والتحقق من الـ tokens
- **JwtAuthenticationFilter**: Filter بيعترض كل request ويتحقق من الـ token
- **SecurityConfig**: إعدادات الحماية

```java
// JwtAuthenticationFilter
public class JwtAuthFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain chain) {
        String token = extractToken(request);
        if (token != null && jwtProvider.validateToken(token)) {
            Authentication auth = jwtProvider.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        chain.doFilter(request, response);
    }
}
```

---

### 29. إزاي تعمل Unit Test و Integration Test في Spring Boot؟

**الإجابة المتوقعة:**

**Unit Test** (بدون Spring - سريع):
```java
@ExtendWith(MockitoExtension.class)
class ProductServiceTest {
    @Mock ProductRepository repository;
    @InjectMocks ProductService service;

    @Test
    void shouldFindProduct() {
        when(repository.findById(1L)).thenReturn(Optional.of(new Product("Phone")));
        Product result = service.getProduct(1L);
        assertEquals("Phone", result.getName());
    }
}
```

**Integration Test** (مع Spring - أبطأ):
```java
@SpringBootTest
@AutoConfigureMockMvc
class ProductControllerTest {
    @Autowired MockMvc mockMvc;

    @Test
    void shouldReturnProducts() throws Exception {
        mockMvc.perform(get("/api/products"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$").isArray());
    }
}
```

**Sliced Tests** (أجزاء من Spring - وسط):
- `@WebMvcTest` - Controller layer بس
- `@DataJpaTest` - Repository layer بس
- `@JsonTest` - JSON serialization بس

---

### 30. إيه هو Spring Boot Actuator وليه مهم؟

**الإجابة المتوقعة:**

Actuator بيوفر endpoints جاهزة لمراقبة وإدارة التطبيق في Production:

```
/actuator/health      → حالة التطبيق (UP/DOWN) + Database + Disk Space
/actuator/metrics     → JVM memory, CPU, HTTP request counts
/actuator/info        → معلومات عن التطبيق (version, git commit)
/actuator/env         → Environment properties
/actuator/loggers     → التحكم في log levels وقت التشغيل
/actuator/threaddump  → Thread dump للـ debugging
/actuator/httptrace   → آخر HTTP requests
```

بيتكامل مع:
- Prometheus + Grafana للـ monitoring dashboards
- Spring Boot Admin لـ UI
- Custom health indicators

---

## أسئلة Tricky (الأسئلة الخادعة)

### 31. هل `@Transactional` بتشتغل على private methods؟

**لا.** لأن Spring AOP بيستخدم Proxy pattern واللي بيشتغل على public methods بس. لو حطيتها على private method مش هتعمل حاجة ومش هيطلع error - هتتجاهل بهدوء.

---

### 32. إيه الفرق بين `@Controller` و `@RestController`؟

`@RestController` = `@Controller` + `@ResponseBody`

يعني كل method في `@RestController` بترجع الـ object مباشرة كـ JSON/XML، بينما `@Controller` بيرجع اسم view (HTML template).

---

### 33. لو عندك Singleton Bean فيه mutable state، إيه المشكلة؟

**Thread safety!** الـ Singleton بيتشارك بين كل الـ threads. لو فيه mutable state:
- ممكن يحصل race conditions
- البيانات تتلخبط بين الـ requests

الحل: ما تحطش state في Singleton beans، أو استخدم `ThreadLocal` أو synchronized.

---

### 34. إيه الفرق بين `findById` و `getById` (أو `getReferenceById`) في JpaRepository؟

- `findById(id)` → بيعمل `SELECT` query فوراً وبيرجع `Optional<T>`
- `getReferenceById(id)` → بيرجع **proxy** (lazy reference) من غير ما يروح للـ database. بيروح للـ database بس لما تستخدم الـ object فعلاً. لو الـ entity مش موجود، بيرمي `EntityNotFoundException` وقت الاستخدام مش وقت النداء.

---

### 35. إيه الفرق بين `save()` و `saveAndFlush()` في JpaRepository؟

- `save()` → بيحط الـ entity في الـ persistence context. الـ SQL ممكن يتنفذ بعدين (عند الـ commit أو flush)
- `saveAndFlush()` → بيعمل `save()` + بيبعت الـ SQL للـ database فوراً. مفيد لما محتاج تتأكد إن الـ data اتكتبت (مثلاً عشان auto-generated ID)

---

### 36. إيه هو Spring Boot DevTools وإيه اللي بيعمله؟

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
</dependency>
```

- **Automatic Restart**: لما تغير كود، التطبيق بيعمل restart تلقائي
- **LiveReload**: الـ browser بيعمل refresh تلقائي
- **Relaxed Caching**: بيعطل الـ caching في الـ development
- **Better Error Pages**: صفحات error أوضح
- **بيتعطل تلقائياً في Production** (لما تعمل `java -jar`)

---

## أسئلة Design و Architecture

### 37. إيه الفرق بين Monolith و Microservices وإمتى تستخدم كل واحد؟

**Monolith:**
- تطبيق واحد فيه كل حاجة
- أبسط في البداية
- أسهل في الـ debugging
- مناسب للفرق الصغيرة والمشاريع اللي بتبدأ

**Microservices:**
- كل service مستقلة بـ database خاص
- كل service ممكن تتنشر وتتوسع لوحدها
- أعقد (networking, distributed transactions, monitoring)
- مناسب للفرق الكبيرة والتطبيقات اللي محتاجة scale

**القاعدة:** ابدأ بـ Monolith (أو Modular Monolith) وفصّل لـ Microservices لما تحتاج فعلاً.

---

### 38. إيه هو DTO Pattern وليه مهم؟

**الإجابة:**
DTO (Data Transfer Object) هو object بيستخدم لنقل البيانات بين الطبقات.

**ليه مش بنرجع الـ Entity مباشرة؟**
1. **Security**: ما ترجعش fields حساسة (password, internal IDs)
2. **Performance**: ما ترجعش بيانات مش محتاجها الـ client
3. **Decoupling**: تغيير الـ Entity ما يأثرش على الـ API
4. **Circular references**: تتجنب مشكلة LazyInitializationException و infinite recursion

```java
// Entity (في الـ database)
@Entity
public class User {
    private Long id;
    private String username;
    private String password;     // ما ينفعش يترجع!
    private String internalNote; // داخلي
}

// DTO (للـ API response)
public record UserResponse(Long id, String username) { }

// Mapping
public UserResponse toResponse(User user) {
    return new UserResponse(user.getId(), user.getUsername());
}
```

---

### 39. اشرح الـ 3-Tier Architecture في Spring

```
┌──────────────────────┐
│   Presentation       │  @RestController / @Controller
│   (Web Layer)        │  - HTTP handling, validation
├──────────────────────┤
│   Business Logic     │  @Service
│   (Service Layer)    │  - Business rules, transactions
├──────────────────────┤
│   Data Access        │  @Repository
│   (Repository Layer) │  - Database operations
└──────────────────────┘
```

**القواعد:**
- Controller ما يتعاملش مع الـ Database مباشرة
- Repository ما يحتويش على business logic
- Service هو اللي بيربط بينهم
- كل طبقة بتتعامل مع الطبقة اللي تحتها بس

---

### 40. إيه الفرق بين `@RequestParam` و `@PathVariable` و `@RequestBody`؟

```java
// @PathVariable - جزء من الـ URL path
// GET /api/products/5
@GetMapping("/products/{id}")
public Product getProduct(@PathVariable Long id) { }

// @RequestParam - query parameter
// GET /api/products?category=electronics&page=2
@GetMapping("/products")
public List<Product> getProducts(@RequestParam String category,
                                  @RequestParam(defaultValue = "0") int page) { }

// @RequestBody - الـ JSON body من الـ request
// POST /api/products  +  {"name": "Phone", "price": 999}
@PostMapping("/products")
public Product createProduct(@RequestBody CreateProductRequest request) { }
```

---

## نصائح للانترفيو

1. **ما تحفظش - افهم.** المحاور بيعرف يفرق بين اللي فاهم واللي حافظ.

2. **اربط بالعملي.** لما تشرح مفهوم، ادي مثال من مشروع حقيقي اشتغلت عليه.

3. **اعترف لما ما تعرفش.** أحسن من إنك تألف إجابة غلط. قول "مش متأكد بس أعتقد..." أو "مش عارف بس هعمل research."

4. **اسأل أسئلة.** لما السؤال مش واضح، اسأل. ده بيبين إنك بتفكر مش بتحفظ.

5. **اتكلم عن الـ trade-offs.** مفيش حل مثالي. كل حل ليه مزايا وعيوب. بيّن إنك بتفهم الـ trade-offs.

6. **اعرف الـ Spring Boot version اللي اشتغلت عليه.** فيه فروقات بين Spring Boot 2.x و 3.x (مثلاً Jakarta EE بدل javax).
