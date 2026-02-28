# Spring Framework - الدليل الشامل قبل ما تبدأ

## الفكرة الأساسية: ليه Spring أصلاً؟

تخيل إنك بتبني تطبيق Java كبير. عندك عشرات الـ classes اللي بتعتمد على بعض. الـ `OrderService` محتاج `PaymentService` و `InventoryService` و `EmailService`. وكل واحد فيهم محتاج `Database` و `Logger`. لو أنت هتعمل كل ده يدوي، هتلاقي نفسك بتكتب كود مليان `new` في كل مكان، والتغيير في حاجة واحدة بيأثر على كل حاجة تانية.

فاSpring جه يحل المشكلة دي. بيقولك: "أنت قولي إيه اللي محتاجه، وأنا هوصله ليك." الفكرة دي اسمها **Inversion of Control (IoC)** - بدل ما أنت تتحكم في إنشاء الـ objects، Spring هو اللي بيتحكم.

---

## الأساسيات اللي لازم تفهمها الأول

### 1. Inversion of Control (IoC) - قلب التحكم

في الطريقة التقليدية:
```java
public class OrderService {
    private PaymentService paymentService = new PaymentService(); // أنت بتنشئه
    private InventoryService inventoryService = new InventoryService(); // وده كمان
}
```

المشكلة هنا إن `OrderService` مربوط بشكل صلب بـ `PaymentService`. لو عايز تغير الـ implementation أو تعمل testing، محتاج تعدل الكود نفسه.

في Spring:
```java
public class OrderService {
    private PaymentService paymentService; // Spring هو اللي هيوصله ليك

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService; // Spring بيبعته في الـ constructor
    }
}
```

أنت مش بتنشئ حاجة. Spring بيشوف إنك محتاج `PaymentService` وبيديهولك. ده معناه إنك تقدر تبدل الـ implementation من غير ما تغير سطر في `OrderService`.

### 2. Dependency Injection (DI) - حقن التبعيات

ده الاسم العملي لـ IoC. يعني Spring بـ"يحقن" (يدخل) الـ dependencies في الـ classes بتاعتك. فيه 3 طرق:

#### Constructor Injection (الأفضل والأكثر استخداماً)
```java
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;

    @Autowired // ممكن تشيلها لو constructor واحد بس
    public OrderService(PaymentService paymentService, InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
}
```
**ليه الأفضل؟** لأن الـ dependencies بتبقى `final` (ما تتغيرش)، والـ object ما يتنشئش من غيرها. ده بيخلي الكود أكثر أماناً.

#### Setter Injection
```java
@Service
public class OrderService {
    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```
بتستخدمها لما الـ dependency تكون اختيارية (optional).

#### Field Injection (تجنبها)
```java
@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService; // مباشرة على الـ field
}
```
شكلها أبسط لكنها سيئة لأنها بتخلي الـ testing صعب وبتخفي الـ dependencies.

### 3. Spring Container (ApplicationContext) - الحاوية

الـ Spring Container هو الـ "صندوق الكبير" اللي بيحتوي على كل الـ objects (اللي اسمها **Beans**). لما التطبيق بيشتغل:

1. Spring بيقرأ الـ configuration (annotations أو XML)
2. بينشئ كل الـ Beans اللي محتاجها
3. بيربطها ببعض (يحقن الـ dependencies)
4. بيخليها جاهزة للاستخدام

```
[ApplicationContext]
├── OrderService (bean)
├── PaymentService (bean)
├── InventoryService (bean)
├── EmailService (bean)
└── DataSource (bean)
```

---

## الـ Beans - أهم مفهوم في Spring

### إيه هو الـ Bean؟

الـ Bean هو أي object بيديره Spring. أنت بتقول لـ Spring "أنا عايزك تنشئ object من الـ class ده وتديره" وخلاص.

### إزاي تعرّف Bean؟

#### الطريقة 1: Stereotype Annotations (الأكثر شيوعاً)
```java
@Component          // أي class عادي
@Service            // class فيه business logic
@Repository         // class بيتعامل مع الـ database
@Controller         // class بيستقبل HTTP requests
@RestController     // زي @Controller بس بيرجع JSON مباشرة
```

كل دي في الآخر بتعمل نفس الحاجة: بتقول لـ Spring "اعمل Bean من الـ class ده." الفرق بينهم هو **الدلالة** - بتوضح الغرض من الـ class.

```java
@Service
public class PaymentService {
    // Spring هينشئ object من الكلاس ده تلقائياً
}

@Repository
public class OrderRepository {
    // Spring عارف إن ده بيتعامل مع الـ database
    // وبيعمل حاجات إضافية زي ترجمة الـ exceptions
}
```

#### الطريقة 2: @Bean Method في @Configuration class
```java
@Configuration
public class AppConfig {

    @Bean
    public DataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://localhost/mydb");
        ds.setUsername("root");
        return ds;
    }

    @Bean
    public EmailService emailService() {
        return new EmailService("smtp.gmail.com", 587);
    }
}
```
بتستخدم الطريقة دي لما:
- الـ class مش بتاعك (من library خارجية)
- محتاج تعمل configuration معين وقت الإنشاء
- محتاج تتحكم في طريقة الإنشاء

### Bean Scope - عمر الـ Bean

```java
@Component
@Scope("singleton")  // ده الافتراضي - object واحد بس في كل التطبيق
public class PaymentService { }

@Component
@Scope("prototype")  // object جديد كل مرة حد يطلبه
public class ShoppingCart { }

// في Web Applications بس:
@Scope("request")    // object جديد لكل HTTP request
@Scope("session")    // object جديد لكل user session
```

**الـ Singleton هو الافتراضي** ودي حاجة مهمة جداً تفهمها. يعني لو عندك `PaymentService`، Spring بينشئ object واحد بس وكل اللي بيطلبوه بياخدوا نفس الـ object. علشان كده:
- ما تحطش state (بيانات متغيرة) في الـ singleton beans
- لو محتاج state لكل request، استخدم `prototype` أو `request` scope

### Bean Lifecycle - دورة حياة الـ Bean

```java
@Component
public class DatabaseConnection {

    @PostConstruct  // بيتنفذ بعد ما Spring ينشئ الـ Bean ويحقن كل الـ dependencies
    public void init() {
        System.out.println("تم الاتصال بالـ database");
    }

    @PreDestroy  // بيتنفذ قبل ما التطبيق يقفل
    public void cleanup() {
        System.out.println("تم قطع الاتصال بالـ database");
    }
}
```

الترتيب:
1. الSpring بينشئ الـ object (constructor)
2. بيحقن الـ dependencies
3. بينادي `@PostConstruct`
4. ... الـ Bean شغال ...
5. التطبيق بيقفل → بينادي `@PreDestroy`

---

## Spring Boot - البداية السريعة

### الفرق بين Spring و Spring Boot

**Spring** هو الـ framework الأساسي. محتاج تعمل configuration كتير.

**Spring Boot** هو طبقة فوق Spring بتعملك auto-configuration. بدل ما تكتب 50 سطر configuration، بتكتب سطرين وخلاص.

```java
@SpringBootApplication  // دي بتشغل كل السحر
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

`@SpringBootApplication` دي اختصار لـ 3 annotations:
- `@Configuration` - الـ class ده فيه configuration
- `@EnableAutoConfiguration` - خلي Spring Boot يعمل auto-configuration
- `@ComponentScan` - دور على كل الـ @Component/@Service/etc. في الـ package ده وتحته

### application.properties / application.yml

ده ملف الإعدادات الرئيسي:

```properties
# application.properties
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=update
```

أو بصيغة YAML (أوضح للقراءة):
```yaml
# application.yml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: secret
  jpa:
    hibernate:
      ddl-auto: update
```

### Profiles - بيئات مختلفة

```properties
# application-dev.properties (بيئة التطوير)
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/devdb

# application-prod.properties (بيئة الإنتاج)
server.port=80
spring.datasource.url=jdbc:mysql://prod-server:3306/proddb
```

بتشغل profile معين:
```bash
java -jar app.jar --spring.profiles.active=dev
```

أو في الـ properties:
```properties
spring.profiles.active=dev
```

---

## Spring MVC - بناء Web Applications

### الفكرة الأساسية

Spring MVC بيتبع نمط **Model-View-Controller**:
- **Model**: البيانات
- **View**: الـ HTML/JSON اللي بيتبعت للمستخدم
- **Controller**: اللي بيستقبل الـ request ويرجع الـ response

### REST Controllers

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    // GET /api/products
    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll();
    }

    // GET /api/products/5
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.findById(id);
    }

    // POST /api/products
    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        return productService.save(product);
    }

    // PUT /api/products/5
    @PutMapping("/{id}")
    public Product updateProduct(@PathVariable Long id, @RequestBody Product product) {
        product.setId(id);
        return productService.save(product);
    }

    // DELETE /api/products/5
    @DeleteMapping("/{id}")
    public void deleteProduct(@PathVariable Long id) {
        productService.delete(id);
    }
}
```

### الـ Annotations المهمة في Controllers

```java
@GetMapping("/search")
public List<Product> search(
    @RequestParam String name,           // /search?name=phone
    @RequestParam(required = false, defaultValue = "0") int page,  // اختياري مع قيمة افتراضية
    @RequestHeader("Authorization") String token,  // من الـ HTTP headers
    @CookieValue("sessionId") String session       // من الـ cookies
) {
    return productService.search(name, page);
}
```

### ResponseEntity - تحكم كامل في الـ Response

```java
@GetMapping("/{id}")
public ResponseEntity<Product> getProduct(@PathVariable Long id) {
    Product product = productService.findById(id);
    if (product == null) {
        return ResponseEntity.notFound().build();           // 404
    }
    return ResponseEntity.ok(product);                      // 200 + body
}

@PostMapping
public ResponseEntity<Product> createProduct(@RequestBody Product product) {
    Product saved = productService.save(product);
    URI location = URI.create("/api/products/" + saved.getId());
    return ResponseEntity.created(location).body(saved);    // 201 + location header
}
```

### Exception Handling - التعامل مع الأخطاء

```java
// Exception class مخصص
public class ProductNotFoundException extends RuntimeException {
    public ProductNotFoundException(Long id) {
        super("Product not found: " + id);
    }
}

// Global Exception Handler
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ProductNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(404, ex.getMessage());
        return ResponseEntity.status(404).body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse(500, "Internal Server Error");
        return ResponseEntity.status(500).body(error);
    }
}
```

### Request Validation - التحقق من البيانات

```java
public class CreateProductRequest {
    @NotBlank(message = "الاسم مطلوب")
    private String name;

    @Positive(message = "السعر لازم يكون أكبر من صفر")
    private BigDecimal price;

    @Size(min = 5, max = 500, message = "الوصف لازم يكون بين 5 و 500 حرف")
    private String description;

    @Email(message = "البريد الإلكتروني غير صالح")
    private String supplierEmail;
}

@PostMapping
public Product createProduct(@Valid @RequestBody CreateProductRequest request) {
    // لو الـ validation فشل، Spring بيرجع 400 Bad Request تلقائياً
    return productService.create(request);
}
```

---

## Spring Data JPA - التعامل مع الـ Database

### إيه هو JPA؟

**JPA** (Java Persistence API) هو معيار لربط الـ Java objects بجداول الـ database. **Hibernate** هو الـ implementation الأشهر. **Spring Data JPA** بيسهل استخدام JPA بشكل كبير.

### الـ Entity - ربط Class بجدول

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String name;

    @Column(precision = 10, scale = 2)
    private BigDecimal price;

    @Column(columnDefinition = "TEXT")
    private String description;

    @Enumerated(EnumType.STRING)
    private ProductStatus status;

    @CreatedDate
    private LocalDateTime createdAt;

    // Getters و Setters
}
```

### العلاقات بين الجداول

```java
// One-to-Many: مثلاً Category فيها Products كتير
@Entity
public class Category {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL)
    private List<Product> products = new ArrayList<>();
}

@Entity
public class Product {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToOne(fetch = FetchType.LAZY)  // LAZY = ما تجيبش الـ Category إلا لما أطلبها
    @JoinColumn(name = "category_id")
    private Category category;
}

// Many-to-Many: Product ممكن يكون في أكتر من Order والعكس
@Entity
public class Order {
    @ManyToMany
    @JoinTable(
        name = "order_products",
        joinColumns = @JoinColumn(name = "order_id"),
        inverseJoinColumns = @JoinColumn(name = "product_id")
    )
    private Set<Product> products = new HashSet<>();
}
```

### الـ Repository - التعامل مع الـ Database بدون SQL

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    // Spring بيولد الـ SQL تلقائياً من اسم الـ method!

    List<Product> findByName(String name);
    // SELECT * FROM products WHERE name = ?

    List<Product> findByPriceLessThan(BigDecimal price);
    // SELECT * FROM products WHERE price < ?

    List<Product> findByNameContainingIgnoreCase(String keyword);
    // SELECT * FROM products WHERE LOWER(name) LIKE LOWER('%keyword%')

    List<Product> findByCategoryNameAndPriceBetween(String categoryName, BigDecimal min, BigDecimal max);
    // SELECT p FROM products p JOIN categories c ON p.category_id = c.id
    // WHERE c.name = ? AND p.price BETWEEN ? AND ?

    Optional<Product> findByNameAndStatus(String name, ProductStatus status);

    long countByStatus(ProductStatus status);

    boolean existsByName(String name);

    List<Product> findTop10ByOrderByPriceDesc();
    // SELECT * FROM products ORDER BY price DESC LIMIT 10
}
```

### Queries مخصصة

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    // JPQL - شبه SQL بس على الـ objects
    @Query("SELECT p FROM Product p WHERE p.price > :minPrice AND p.category.name = :category")
    List<Product> findExpensiveByCategory(@Param("minPrice") BigDecimal minPrice,
                                          @Param("category") String category);

    // Native SQL - لو محتاج SQL حقيقي
    @Query(value = "SELECT * FROM products WHERE MATCH(name, description) AGAINST(:keyword)",
           nativeQuery = true)
    List<Product> fullTextSearch(@Param("keyword") String keyword);

    // Update query
    @Modifying
    @Transactional
    @Query("UPDATE Product p SET p.status = :status WHERE p.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") ProductStatus status);
}
```

### الـ Methods اللي بتيجي جاهزة مع JpaRepository

```java
// مش محتاج تعرف أي method - دول جاهزين:
productRepository.save(product);              // INSERT أو UPDATE
productRepository.saveAll(productList);       // حفظ مجموعة
productRepository.findById(1L);              // SELECT by ID → Optional<Product>
productRepository.findAll();                  // SELECT *
productRepository.findAll(Sort.by("price")); // SELECT * ORDER BY price
productRepository.findAll(PageRequest.of(0, 10)); // Pagination
productRepository.count();                    // COUNT(*)
productRepository.deleteById(1L);            // DELETE
productRepository.existsById(1L);            // EXISTS
```

### Pagination و Sorting

```java
@GetMapping
public Page<Product> getProducts(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "name") String sortBy
) {
    Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
    return productRepository.findAll(pageable);
}
// الـ response بيرجع:
// {
//   "content": [...],      // البيانات
//   "totalElements": 150,  // العدد الكلي
//   "totalPages": 15,      // عدد الصفحات
//   "number": 0,           // الصفحة الحالية
//   "size": 10             // حجم الصفحة
// }
```

---

## الـ Service Layer - طبقة الـ Business Logic

### ليه محتاج Service Layer؟

الـ Controller ما ينفعش يحتوي على business logic. الـ Repository ما ينفعش يحتوي على business logic. الـ Service هو اللي بيحتوي على كل الـ business logic.

```
Controller  →  Service  →  Repository  →  Database
(HTTP)         (Logic)     (Data Access)   (Storage)
```

```java
@Service
@Transactional(readOnly = true)  // كل الـ methods read-only بالافتراضي
public class OrderService {

    private final OrderRepository orderRepository;
    private final ProductRepository productRepository;
    private final PaymentService paymentService;

    public OrderService(OrderRepository orderRepository,
                       ProductRepository productRepository,
                       PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.productRepository = productRepository;
        this.paymentService = paymentService;
    }

    public Order getOrder(Long id) {
        return orderRepository.findById(id)
            .orElseThrow(() -> new OrderNotFoundException(id));
    }

    @Transactional  // ده بيعمل override للـ readOnly - هنا بنكتب في الـ database
    public Order createOrder(CreateOrderRequest request) {
        // 1. التحقق من المنتجات
        List<Product> products = productRepository.findAllById(request.getProductIds());
        if (products.size() != request.getProductIds().size()) {
            throw new InvalidOrderException("بعض المنتجات غير موجودة");
        }

        // 2. حساب الإجمالي
        BigDecimal total = products.stream()
            .map(Product::getPrice)
            .reduce(BigDecimal.ZERO, BigDecimal::add);

        // 3. إنشاء الطلب
        Order order = new Order();
        order.setProducts(new HashSet<>(products));
        order.setTotal(total);
        order.setStatus(OrderStatus.PENDING);

        // 4. معالجة الدفع
        paymentService.processPayment(order);

        // 5. حفظ الطلب
        return orderRepository.save(order);
        // لو حصل أي exception، كل اللي فوق بيترجع (rollback) تلقائياً
    }
}
```

---

## @Transactional - إدارة المعاملات

### ليه مهمة؟

تخيل إنك بتحول فلوس من حساب لحساب:
1. خصم من الحساب الأول ✅
2. إضافة للحساب التاني ❌ (حصل error)

من غير Transaction، الفلوس اتخصمت بس ما اتضافتش! مع `@Transactional`، لو أي خطوة فشلت، كل حاجة بترجع زي ما كانت.

```java
@Transactional
public void transferMoney(Long fromAccount, Long toAccount, BigDecimal amount) {
    Account from = accountRepository.findById(fromAccount).orElseThrow();
    Account to = accountRepository.findById(toAccount).orElseThrow();

    from.setBalance(from.getBalance().subtract(amount));  // خصم
    to.setBalance(to.getBalance().add(amount));            // إضافة

    accountRepository.save(from);
    accountRepository.save(to);
    // لو حصل exception في أي سطر، كل حاجة بترجع (rollback)
}
```

### خيارات @Transactional

```java
@Transactional(
    readOnly = true,                    // للقراءة فقط (أسرع)
    timeout = 30,                       // timeout بالثواني
    rollbackFor = Exception.class,      // اعمل rollback لأي exception
    noRollbackFor = EmailException.class, // ما تعملش rollback لده
    propagation = Propagation.REQUIRED  // السلوك مع transactions تانية
)
```

### Propagation - سلوك الـ Transaction مع transactions تانية

```java
// REQUIRED (الافتراضي): استخدم transaction موجودة أو اعمل واحدة جديدة
@Transactional(propagation = Propagation.REQUIRED)

// REQUIRES_NEW: دايماً اعمل transaction جديدة (علق الحالية)
@Transactional(propagation = Propagation.REQUIRES_NEW)

// SUPPORTS: لو فيه transaction استخدمها، لو مفيش اشتغل من غير
@Transactional(propagation = Propagation.SUPPORTS)

// NOT_SUPPORTED: اشتغل من غير transaction حتى لو فيه واحدة
@Transactional(propagation = Propagation.NOT_SUPPORTED)
```

---

## AOP - Aspect Oriented Programming

### الفكرة

عندك حاجات بتتكرر في كل مكان: logging، security checks، performance monitoring. بدل ما تحطها في كل method، بتحطها في مكان واحد وSpring بيطبقها تلقائياً.

```java
@Aspect
@Component
public class LoggingAspect {

    // قبل أي method في أي Service class
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Calling: " + joinPoint.getSignature().getName());
    }

    // بعد أي method يرجع بنجاح
    @AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("Returned: " + result);
    }

    // حوالين الـ method (قبل وبعد) - الأقوى
    @Around("execution(* com.example.service.*.*(..))")
    public Object measureTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();  // نفذ الـ method الأصلي
        long duration = System.currentTimeMillis() - start;
        System.out.println(joinPoint.getSignature() + " took " + duration + "ms");
        return result;
    }
}
```

### إزاي Spring بيطبق AOP؟

Spring بيعمل **Proxy** حوالين الـ Bean بتاعك. يعني لما حد ينادي `orderService.createOrder()`، مش بينادي الـ method مباشرة. بينادي الـ proxy اللي بيعمل الـ logging/security/transaction الأول، وبعدين ينادي الـ method الأصلي.

```
Client → [Proxy: logging → security → transaction] → OrderService.createOrder()
```

**مهم جداً**: بسبب الـ Proxy ده:
- `@Transactional` و `@Async` وغيرها ما بتشتغلش لو ناديت method من جوه نفس الـ class
- لازم النداء يكون من class تاني عشان يعدي على الـ Proxy

```java
@Service
public class OrderService {
    @Transactional
    public void createOrder() { ... }

    public void doSomething() {
        this.createOrder(); // ⚠️ الـ @Transactional مش هتشتغل! النداء الداخلي بيتجاوز الـ Proxy
    }
}
```

---

## Spring Security - الحماية والأمان

### الإعداد الأساسي

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())  // عادةً بتتعطل في REST APIs
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()       // متاح للجميع
                .requestMatchers("/api/admin/**").hasRole("ADMIN")   // للـ admin بس
                .requestMatchers(HttpMethod.POST, "/api/products").hasRole("MANAGER")
                .anyRequest().authenticated()                        // الباقي محتاج login
            )
            .httpBasic(Customizer.withDefaults());   // Basic Auth
            // أو
            // .formLogin(Customizer.withDefaults()); // Form Login

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();  // تشفير كلمات المرور
    }
}
```

### UserDetailsService - من أين يجيب بيانات المستخدمين

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));

        return org.springframework.security.core.userdetails.User
            .withUsername(user.getUsername())
            .password(user.getPassword())  // لازم يكون مشفر بـ BCrypt
            .roles(user.getRoles().toArray(new String[0]))
            .build();
    }
}
```

### JWT Authentication (الأكثر استخداماً في REST APIs)

```java
// الفكرة العامة:
// 1. المستخدم بيبعت username + password
// 2. السيرفر بيتحقق وبيرجع JWT token
// 3. المستخدم بيبعت الـ token في كل request بعد كده
// 4. السيرفر بيتحقق من الـ token ويعرف مين المستخدم

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @PostMapping("/login")
    public AuthResponse login(@RequestBody LoginRequest request) {
        // التحقق من البيانات
        Authentication auth = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword())
        );
        // إنشاء JWT token
        String token = jwtTokenProvider.generateToken(auth);
        return new AuthResponse(token);
    }
}
```

### @PreAuthorize - حماية على مستوى الـ Method

```java
@Service
public class ProductService {

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteProduct(Long id) { ... }

    @PreAuthorize("hasRole('MANAGER') or #userId == authentication.principal.id")
    public void updateProduct(Long userId, Product product) { ... }

    @PreAuthorize("isAuthenticated()")
    public List<Product> getProducts() { ... }
}
```

---

## Spring Events - نظام الأحداث

### الفكرة

بدل ما الـ classes تنادي بعض مباشرة، ممكن تبعت "أحداث" واللي مهتم يسمع.

```java
// 1. تعريف الـ Event
public class OrderCreatedEvent {
    private final Order order;

    public OrderCreatedEvent(Order order) {
        this.order = order;
    }

    public Order getOrder() { return order; }
}

// 2. بعت الـ Event
@Service
public class OrderService {
    private final ApplicationEventPublisher eventPublisher;

    public OrderService(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        Order order = // ... إنشاء الطلب
        orderRepository.save(order);

        eventPublisher.publishEvent(new OrderCreatedEvent(order));
        return order;
    }
}

// 3. استقبل الـ Event
@Component
public class EmailNotificationListener {

    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        // ابعت إيميل تأكيد
        emailService.sendOrderConfirmation(event.getOrder());
    }
}

@Component
public class InventoryListener {

    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        // قلل المخزون
        inventoryService.reduceStock(event.getOrder().getProducts());
    }
}
```

### @TransactionalEventListener

```java
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void onOrderCreated(OrderCreatedEvent event) {
    // بيتنفذ بعد ما الـ transaction تنجح
    // لو الـ transaction عملت rollback، الـ listener ده مش هيتنفذ
}
```

---

## Scheduling - تنفيذ مهام دورية

```java
@Configuration
@EnableScheduling
public class SchedulingConfig { }

@Component
public class ScheduledTasks {

    @Scheduled(fixedRate = 60000)  // كل 60 ثانية
    public void checkPendingOrders() {
        // فحص الطلبات المعلقة
    }

    @Scheduled(fixedDelay = 30000)  // 30 ثانية بعد ما الـ method السابق يخلص
    public void processQueue() {
        // معالجة الطابور
    }

    @Scheduled(cron = "0 0 8 * * MON-FRI")  // الساعة 8 صباحاً أيام العمل
    public void sendDailyReport() {
        // تقرير يومي
    }

    @Scheduled(cron = "0 0 0 1 * *")  // أول يوم في كل شهر
    public void monthlyCleanup() {
        // تنظيف شهري
    }
}
```

---

## @Async - تنفيذ غير متزامن

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.initialize();
        return executor;
    }
}

@Service
public class EmailService {

    @Async  // بتتنفذ في thread منفصل - مش بتعطل الـ caller
    public void sendWelcomeEmail(String email) {
        // ده بياخد وقت... بس مش هيعطل الـ response
        // الكود اللي ناداني هيكمل شغله عادي
    }

    @Async
    public CompletableFuture<EmailResult> sendEmailWithResult(String email) {
        // لو محتاج ترجع نتيجة
        EmailResult result = // ... ابعت الإيميل
        return CompletableFuture.completedFuture(result);
    }
}
```

---

## Caching - التخزين المؤقت

```java
@Configuration
@EnableCaching
public class CacheConfig { }

@Service
public class ProductService {

    @Cacheable("products")  // أول مرة بيجيب من الـ database، المرات اللي بعدها من الـ cache
    public Product getProduct(Long id) {
        return productRepository.findById(id).orElseThrow();
    }

    @Cacheable(value = "products", key = "#name + '-' + #page")
    public List<Product> searchProducts(String name, int page) {
        return productRepository.findByNameContaining(name, PageRequest.of(page, 10));
    }

    @CachePut(value = "products", key = "#product.id")  // حدث الـ cache
    public Product updateProduct(Product product) {
        return productRepository.save(product);
    }

    @CacheEvict(value = "products", key = "#id")  // امسح من الـ cache
    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }

    @CacheEvict(value = "products", allEntries = true)  // امسح كل الـ cache
    public void clearCache() { }
}
```

---

## Spring Actuator - مراقبة التطبيق

### الإعداد

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
management.endpoints.web.exposure.include=health,info,metrics,env
management.endpoint.health.show-details=always
```

### الـ Endpoints الجاهزة

```
GET /actuator/health     → حالة التطبيق (UP/DOWN)
GET /actuator/info       → معلومات عن التطبيق
GET /actuator/metrics    → مقاييس الأداء
GET /actuator/env        → متغيرات البيئة
GET /actuator/beans      → كل الـ Beans المسجلة
GET /actuator/mappings   → كل الـ URL mappings
```

---

## Configuration Properties - إعدادات مخصصة

```java
@ConfigurationProperties(prefix = "app.email")
public class EmailProperties {
    private String host;
    private int port;
    private String from;
    private boolean enabled;

    // Getters و Setters
}
```

```properties
app.email.host=smtp.gmail.com
app.email.port=587
app.email.from=noreply@example.com
app.email.enabled=true
```

```java
@Configuration
@EnableConfigurationProperties(EmailProperties.class)
public class AppConfig { }

@Service
public class EmailService {
    private final EmailProperties emailProperties;

    public EmailService(EmailProperties emailProperties) {
        this.emailProperties = emailProperties;
        // emailProperties.getHost() → "smtp.gmail.com"
    }
}
```

### @Value - للقيم البسيطة

```java
@Service
public class MyService {
    @Value("${app.name}")
    private String appName;

    @Value("${app.max-retries:3}")  // قيمة افتراضية = 3
    private int maxRetries;

    @Value("${app.feature.enabled:false}")
    private boolean featureEnabled;
}
```

---

## Filters و Interceptors - اعتراض الـ Requests

### Filter (مستوى Servlet - قبل Spring)

```java
@Component
public class RequestLoggingFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        System.out.println("Request: " + httpRequest.getMethod() + " " + httpRequest.getRequestURI());

        long start = System.currentTimeMillis();
        chain.doFilter(request, response);  // كمّل للـ filter اللي بعده أو الـ controller
        long duration = System.currentTimeMillis() - start;

        System.out.println("Response time: " + duration + "ms");
    }
}
```

### Interceptor (مستوى Spring - أقوى)

```java
@Component
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // قبل ما الـ Controller يشتغل
        String token = request.getHeader("Authorization");
        if (token == null) {
            response.setStatus(401);
            return false;  // أوقف - ما تكملش
        }
        return true;  // كمّل
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                          Object handler, ModelAndView modelAndView) {
        // بعد ما الـ Controller يخلص وقبل ما الـ response يتبعت
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                               Object handler, Exception ex) {
        // بعد ما كل حاجة تخلص
    }
}

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthInterceptor())
                .addPathPatterns("/api/**")          // طبقه على الـ paths دي
                .excludePathPatterns("/api/auth/**"); // إلا دي
    }
}
```

---

## Testing في Spring

### Unit Testing (بدون Spring)

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private ProductRepository productRepository;

    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldCreateOrderSuccessfully() {
        // Given
        Product product = new Product(1L, "Phone", new BigDecimal("999.99"));
        when(productRepository.findAllById(List.of(1L))).thenReturn(List.of(product));
        when(orderRepository.save(any(Order.class))).thenAnswer(inv -> inv.getArgument(0));

        // When
        CreateOrderRequest request = new CreateOrderRequest(List.of(1L));
        Order result = orderService.createOrder(request);

        // Then
        assertNotNull(result);
        assertEquals(new BigDecimal("999.99"), result.getTotal());
        verify(orderRepository).save(any(Order.class));
    }

    @Test
    void shouldThrowWhenProductNotFound() {
        when(productRepository.findAllById(anyList())).thenReturn(List.of());

        CreateOrderRequest request = new CreateOrderRequest(List.of(999L));

        assertThrows(InvalidOrderException.class, () -> orderService.createOrder(request));
    }
}
```

### Integration Testing (مع Spring)

```java
@SpringBootTest
@AutoConfigureMockMvc
class ProductControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ProductRepository productRepository;

    @BeforeEach
    void setUp() {
        productRepository.deleteAll();
    }

    @Test
    void shouldReturnAllProducts() throws Exception {
        productRepository.save(new Product(null, "Phone", new BigDecimal("999")));
        productRepository.save(new Product(null, "Laptop", new BigDecimal("1999")));

        mockMvc.perform(get("/api/products"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$", hasSize(2)))
            .andExpect(jsonPath("$[0].name").value("Phone"));
    }

    @Test
    void shouldCreateProduct() throws Exception {
        String json = """
            {"name": "Tablet", "price": 599.99}
            """;

        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(json))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.name").value("Tablet"));
    }
}
```

### Repository Testing

```java
@DataJpaTest  // بيشغل Spring بس للـ JPA - أسرع من @SpringBootTest
class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @Test
    void shouldFindByNameContaining() {
        productRepository.save(new Product(null, "iPhone 15", new BigDecimal("999")));
        productRepository.save(new Product(null, "Samsung Galaxy", new BigDecimal("899")));

        List<Product> results = productRepository.findByNameContainingIgnoreCase("iphone");

        assertEquals(1, results.size());
        assertEquals("iPhone 15", results.get(0).getName());
    }
}
```

---

## Design Patterns اللي Spring بيستخدمها

| Pattern | فين في Spring | الشرح |
|---------|-------------|-------|
| **Singleton** | كل Bean بالافتراضي | object واحد بس في كل التطبيق |
| **Factory** | `BeanFactory` / `ApplicationContext` | بينشئ الـ objects |
| **Proxy** | AOP, `@Transactional` | بيلف الـ objects بطبقة إضافية |
| **Template Method** | `JdbcTemplate`, `RestTemplate` | بيوفر هيكل ثابت مع تخصيص |
| **Observer** | `ApplicationEvent` | نظام الأحداث |
| **Strategy** | `HandlerMapping`, `ViewResolver` | تبديل الخوارزميات |
| **Decorator** | Filter Chain | إضافة سلوك بدون تعديل |

---

## الهيكل المثالي لمشروع Spring Boot

```
src/main/java/com/example/myapp/
├── MyAppApplication.java              # نقطة البداية
├── config/                            # الإعدادات
│   ├── SecurityConfig.java
│   ├── WebConfig.java
│   └── CacheConfig.java
├── controller/                        # HTTP endpoints
│   ├── ProductController.java
│   └── OrderController.java
├── service/                           # Business logic
│   ├── ProductService.java
│   └── OrderService.java
├── repository/                        # Data access
│   ├── ProductRepository.java
│   └── OrderRepository.java
├── entity/                            # Database entities
│   ├── Product.java
│   └── Order.java
├── dto/                               # Data Transfer Objects
│   ├── CreateProductRequest.java
│   └── ProductResponse.java
├── exception/                         # Custom exceptions
│   ├── ProductNotFoundException.java
│   └── GlobalExceptionHandler.java
└── util/                              # Utility classes
    └── DateUtils.java

src/main/resources/
├── application.yml
├── application-dev.yml
└── application-prod.yml
```

---

## ملخص الـ Annotations الأهم

### Core
| Annotation | الوظيفة |
|-----------|---------|
| `@Component` | عرّف Bean عام |
| `@Service` | عرّف Bean لـ business logic |
| `@Repository` | عرّف Bean للـ data access |
| `@Controller` | عرّف Bean لـ HTTP (يرجع views) |
| `@RestController` | عرّف Bean لـ HTTP (يرجع JSON) |
| `@Configuration` | class فيه إعدادات وتعريف beans |
| `@Bean` | عرّف Bean في method |
| `@Autowired` | اطلب من Spring يحقن dependency |
| `@Qualifier("name")` | حدد أي implementation لما يكون فيه أكتر من واحد |
| `@Primary` | خلي الـ Bean ده الافتراضي |
| `@Value("${key}")` | اقرأ قيمة من الـ properties |
| `@Scope("prototype")` | غيّر scope الـ Bean |
| `@Lazy` | ما تنشئش الـ Bean غير لما يتطلب |

### Web
| Annotation | الوظيفة |
|-----------|---------|
| `@RequestMapping` | حدد الـ URL الأساسي |
| `@GetMapping` | HTTP GET |
| `@PostMapping` | HTTP POST |
| `@PutMapping` | HTTP PUT |
| `@DeleteMapping` | HTTP DELETE |
| `@PathVariable` | قيمة من الـ URL path |
| `@RequestParam` | قيمة من الـ query string |
| `@RequestBody` | الـ body كـ object |
| `@ResponseStatus` | حدد HTTP status code |
| `@Valid` | فعّل الـ validation |

### Data
| Annotation | الوظيفة |
|-----------|---------|
| `@Entity` | ربط class بجدول database |
| `@Table` | حدد اسم الجدول |
| `@Id` | primary key |
| `@GeneratedValue` | auto-increment |
| `@Column` | إعدادات العمود |
| `@OneToMany` | علاقة واحد لكثير |
| `@ManyToOne` | علاقة كثير لواحد |
| `@ManyToMany` | علاقة كثير لكثير |
| `@Transactional` | إدارة المعاملات |
| `@Query` | كتابة query مخصص |

### Testing
| Annotation | الوظيفة |
|-----------|---------|
| `@SpringBootTest` | شغّل كل Spring للـ testing |
| `@DataJpaTest` | شغّل JPA بس |
| `@WebMvcTest` | شغّل Web layer بس |
| `@MockBean` | بدّل Bean بـ mock |
| `@AutoConfigureMockMvc` | جهّز MockMvc |

---

## ترتيب المذاكرة المقترح

1. **Java أساسيات** - لو مش متمكن منها (OOP, Collections, Streams, Generics)
2. **IoC و DI** - ده أساس كل حاجة في Spring
3. **Spring Boot** - الإعداد والتشغيل
4. **Spring MVC** - بناء REST APIs
5. **Spring Data JPA** - التعامل مع الـ Database
6. **Validation و Exception Handling** - التعامل مع الأخطاء
7. **Spring Security** - الحماية
8. **Testing** - كتابة الاختبارات
9. **AOP و Events** - مفاهيم متقدمة
10. **Caching, Async, Scheduling** - تحسين الأداء
11. **Actuator و Monitoring** - مراقبة التطبيق

---

## نصائح مهمة قبل ما تبدأ

**افهم الـ magic**: Spring بيعمل حاجات كتير "وراك". لما تفهم الـ Proxy pattern و IoC container، كل حاجة هتبقى منطقية.

**ابدأ بمشروع صغير**: ما تحاولش تتعلم كل حاجة مرة واحدة. اعمل CRUD بسيط وابني عليه.

**اقرأ الـ error messages**: Spring errors طويلة بس فيها معلومات مفيدة. دور على السطر اللي فيه `Caused by:`.

**استخدم Spring Initializr**: [start.spring.io](https://start.spring.io) - ده بيولدلك مشروع جاهز بالـ dependencies اللي محتاجها.

**الـ documentation الرسمية**: Spring عنده من أفضل الـ documentation في عالم البرمجة. ارجعلها دايماً.
