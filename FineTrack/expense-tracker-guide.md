# دليل بناء مشروع تتبع النفقات والدخل من الصفر
## SQL Server + Spring Boot + Vue.js

---

## المقدمة

هذا الدليل سيأخذك خطوة بخطوة من الفكرة للمشروع الكامل. المشروع ده مش بس هيفيدك في السي في، لكن هيديك خبرة حقيقية في:
- تصميم قواعد البيانات
- بناء RESTful APIs
- تطوير Single Page Applications
- التعامل مع الأمان والمصادقة
- كتابة اختبارات
- النشر والتوثيق

---

## الجزء الأول: مرحلة التخطيط والتفكير (قبل كتابة أي كود)

### 1.1 تحديد نطاق المشروع (Scope Definition)

قبل ما تكتب أي كود، لازم تجاوب على الأسئلة دي:

**أسئلة أساسية:**
- مين المستخدم المستهدف؟ (شخص واحد؟ عائلة؟ شركة صغيرة؟)
- إيه المشاكل اللي المشروع بيحلها؟
- إيه الفرق بين مشروعك وأي Excel sheet عادي؟

**للمشروع بتاعك، اقترح النطاق ده:**
```
المستخدم: فرد أو عائلة صغيرة
المشكلة: صعوبة تتبع الفلوس ومعرفة فين بتروح
الحل: نظام بسيط لتسجيل كل حركة مالية مع تقارير ذكية
```

### 1.2 تحديد الميزات (Feature List)

**قسّم الميزات لـ 3 مراحل:**

#### المرحلة الأولى (MVP - الحد الأدنى للمنتج):
- [ ] تسجيل دخول/خروج المستخدم
- [ ] إضافة/تعديل/حذف معاملة مالية (دخل أو مصروف)
- [ ] تصنيف المعاملات (طعام، مواصلات، راتب، إلخ)
- [ ] عرض قائمة المعاملات مع فلترة بالتاريخ
- [ ] عرض الرصيد الحالي

#### المرحلة الثانية (Core Features):
- [ ] تقارير شهرية (إجمالي الدخل/المصروفات)
- [ ] رسوم بيانية (Charts) للمصروفات حسب التصنيف
- [ ] إدارة الديون (مين مديون لمين وبكام)
- [ ] تنبيهات عند تجاوز ميزانية معينة
- [ ] تصدير البيانات لـ Excel

#### المرحلة الثالثة (Advanced Features):
- [ ] أهداف ادخار
- [ ] معاملات متكررة (راتب شهري، إيجار)
- [ ] عملات متعددة
- [ ] مشاركة الحساب مع أفراد العائلة
- [ ] تطبيق موبايل (PWA)

**نصيحة مهمة:** ابدأ بالمرحلة الأولى فقط وخلصها كاملة قبل ما تفكر في أي ميزة تانية.

### 1.3 رسم الشاشات (Wireframes)

قبل التصميم، ارسم على ورق أو باستخدام أداة بسيطة:

**الشاشات الأساسية:**
1. صفحة تسجيل الدخول
2. لوحة التحكم الرئيسية (Dashboard)
3. قائمة المعاملات
4. نموذج إضافة/تعديل معاملة
5. صفحة التقارير
6. صفحة الديون
7. صفحة الإعدادات

**أدوات مقترحة للـ Wireframes:**
- Excalidraw (مجاني وبسيط)
- Figma (مجاني للمشاريع الصغيرة)
- حتى ورقة وقلم!

---

## الجزء الثاني: تصميم قاعدة البيانات

### 2.1 تحديد الكيانات (Entities)

من الميزات اللي حددناها، الكيانات الأساسية هي:

```
1. User (المستخدم)
2. Category (التصنيف)
3. Transaction (المعاملة المالية)
4. Debt (الدين)
5. Budget (الميزانية) - للمرحلة الثانية
```

### 2.2 تصميم الجداول

```sql
-- =============================================
-- جدول المستخدمين
-- =============================================
CREATE TABLE users (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    email NVARCHAR(255) NOT NULL UNIQUE,
    password_hash NVARCHAR(255) NOT NULL,
    full_name NVARCHAR(100) NOT NULL,
    currency NVARCHAR(3) DEFAULT 'EGP',
    created_at DATETIME2 DEFAULT GETDATE(),
    updated_at DATETIME2 DEFAULT GETDATE(),
    is_active BIT DEFAULT 1
);

-- =============================================
-- جدول التصنيفات
-- =============================================
CREATE TABLE categories (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    name NVARCHAR(50) NOT NULL,
    type NVARCHAR(10) NOT NULL CHECK (type IN ('INCOME', 'EXPENSE')),
    icon NVARCHAR(50), -- اسم الأيقونة من مكتبة الأيقونات
    color NVARCHAR(7), -- لون بصيغة HEX مثل #FF5733
    is_default BIT DEFAULT 0,
    created_at DATETIME2 DEFAULT GETDATE(),
    
    CONSTRAINT FK_categories_user FOREIGN KEY (user_id) 
        REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT UQ_category_name_user UNIQUE (user_id, name, type)
);

-- =============================================
-- جدول المعاملات المالية
-- =============================================
CREATE TABLE transactions (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    category_id BIGINT NOT NULL,
    amount DECIMAL(18,2) NOT NULL,
    type NVARCHAR(10) NOT NULL CHECK (type IN ('INCOME', 'EXPENSE')),
    description NVARCHAR(500),
    transaction_date DATE NOT NULL,
    created_at DATETIME2 DEFAULT GETDATE(),
    updated_at DATETIME2 DEFAULT GETDATE(),
    
    CONSTRAINT FK_transactions_user FOREIGN KEY (user_id) 
        REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT FK_transactions_category FOREIGN KEY (category_id) 
        REFERENCES categories(id)
);

-- Index للبحث السريع بالتاريخ
CREATE INDEX IX_transactions_date ON transactions(user_id, transaction_date);
CREATE INDEX IX_transactions_type ON transactions(user_id, type);

-- =============================================
-- جدول الديون
-- =============================================
CREATE TABLE debts (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    person_name NVARCHAR(100) NOT NULL,
    amount DECIMAL(18,2) NOT NULL,
    type NVARCHAR(10) NOT NULL CHECK (type IN ('OWE', 'OWED')), -- OWE = أنا مديون، OWED = هو مديون لي
    description NVARCHAR(500),
    due_date DATE,
    is_settled BIT DEFAULT 0,
    settled_date DATE,
    created_at DATETIME2 DEFAULT GETDATE(),
    updated_at DATETIME2 DEFAULT GETDATE(),
    
    CONSTRAINT FK_debts_user FOREIGN KEY (user_id) 
        REFERENCES users(id) ON DELETE CASCADE
);

-- =============================================
-- جدول الميزانيات (للمرحلة الثانية)
-- =============================================
CREATE TABLE budgets (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    category_id BIGINT,
    amount DECIMAL(18,2) NOT NULL,
    period NVARCHAR(10) NOT NULL CHECK (period IN ('MONTHLY', 'WEEKLY', 'YEARLY')),
    start_date DATE NOT NULL,
    created_at DATETIME2 DEFAULT GETDATE(),
    
    CONSTRAINT FK_budgets_user FOREIGN KEY (user_id) 
        REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT FK_budgets_category FOREIGN KEY (category_id) 
        REFERENCES categories(id)
);

-- =============================================
-- بيانات افتراضية للتصنيفات
-- =============================================
-- هذه ستُضاف برمجياً عند إنشاء مستخدم جديد
/*
التصنيفات الافتراضية للمصروفات:
- طعام ومشروبات
- مواصلات
- تسوق
- فواتير
- صحة
- ترفيه
- تعليم
- أخرى

التصنيفات الافتراضية للدخل:
- راتب
- عمل حر
- استثمارات
- هدايا
- أخرى
*/
```

### 2.3 العلاقات بين الجداول (ERD)

```
┌─────────────┐       ┌──────────────┐       ┌──────────────┐
│    users    │       │  categories  │       │ transactions │
├─────────────┤       ├──────────────┤       ├──────────────┤
│ id (PK)     │──┐    │ id (PK)      │──┐    │ id (PK)      │
│ email       │  │    │ user_id (FK) │  │    │ user_id (FK) │
│ password    │  └───>│ name         │  └───>│ category_id  │
│ full_name   │       │ type         │       │ amount       │
│ ...         │       │ ...          │       │ type         │
└─────────────┘       └──────────────┘       │ ...          │
      │                                       └──────────────┘
      │
      │           ┌──────────────┐
      │           │    debts     │
      │           ├──────────────┤
      └──────────>│ id (PK)      │
                  │ user_id (FK) │
                  │ person_name  │
                  │ amount       │
                  │ ...          │
                  └──────────────┘
```

---

## الجزء الثالث: تصميم الـ API

### 3.1 مبادئ تصميم RESTful API

```
- استخدم الأسماء وليس الأفعال: /transactions وليس /getTransactions
- استخدم HTTP Methods بشكل صحيح:
  GET    = جلب بيانات
  POST   = إنشاء جديد
  PUT    = تحديث كامل
  PATCH  = تحديث جزئي
  DELETE = حذف
- استخدم Status Codes المناسبة:
  200 = نجاح
  201 = تم الإنشاء
  400 = خطأ في الطلب
  401 = غير مصرح
  404 = غير موجود
  500 = خطأ في السيرفر
```

### 3.2 قائمة الـ Endpoints

```yaml
# =============================================
# Authentication
# =============================================
POST   /api/auth/register          # تسجيل مستخدم جديد
POST   /api/auth/login             # تسجيل دخول
POST   /api/auth/logout            # تسجيل خروج
POST   /api/auth/refresh           # تجديد التوكن
GET    /api/auth/me                # بيانات المستخدم الحالي

# =============================================
# Categories
# =============================================
GET    /api/categories             # جلب كل التصنيفات
GET    /api/categories/{id}        # جلب تصنيف معين
POST   /api/categories             # إنشاء تصنيف
PUT    /api/categories/{id}        # تحديث تصنيف
DELETE /api/categories/{id}        # حذف تصنيف

# =============================================
# Transactions
# =============================================
GET    /api/transactions           # جلب المعاملات (مع pagination و filtering)
GET    /api/transactions/{id}      # جلب معاملة معينة
POST   /api/transactions           # إنشاء معاملة
PUT    /api/transactions/{id}      # تحديث معاملة
DELETE /api/transactions/{id}      # حذف معاملة

# Query Parameters للـ GET /api/transactions:
# - page: رقم الصفحة (default: 0)
# - size: عدد العناصر (default: 20)
# - type: INCOME أو EXPENSE
# - categoryId: فلترة بالتصنيف
# - startDate: بداية الفترة
# - endDate: نهاية الفترة
# - sort: الترتيب (مثال: transactionDate,desc)

# =============================================
# Debts
# =============================================
GET    /api/debts                  # جلب كل الديون
GET    /api/debts/{id}             # جلب دين معين
POST   /api/debts                  # إنشاء دين
PUT    /api/debts/{id}             # تحديث دين
PATCH  /api/debts/{id}/settle      # تسوية دين
DELETE /api/debts/{id}             # حذف دين

# =============================================
# Reports & Statistics
# =============================================
GET    /api/reports/summary        # ملخص (إجمالي الدخل والمصروفات)
GET    /api/reports/by-category    # تفصيل حسب التصنيف
GET    /api/reports/monthly        # تقرير شهري
GET    /api/reports/trends         # اتجاهات على مدار الوقت

# Query Parameters للتقارير:
# - startDate: بداية الفترة
# - endDate: نهاية الفترة
# - type: INCOME أو EXPENSE

# =============================================
# Dashboard
# =============================================
GET    /api/dashboard              # بيانات لوحة التحكم الرئيسية
# يرجع: الرصيد، آخر المعاملات، ملخص الشهر، الديون المستحقة
```

### 3.3 أمثلة على Request/Response

```json
// POST /api/transactions
// Request:
{
  "categoryId": 1,
  "amount": 150.50,
  "type": "EXPENSE",
  "description": "غداء مع الأصدقاء",
  "transactionDate": "2025-01-20"
}

// Response (201 Created):
{
  "id": 42,
  "categoryId": 1,
  "categoryName": "طعام ومشروبات",
  "amount": 150.50,
  "type": "EXPENSE",
  "description": "غداء مع الأصدقاء",
  "transactionDate": "2025-01-20",
  "createdAt": "2025-01-20T14:30:00"
}

// GET /api/transactions?page=0&size=10&type=EXPENSE&startDate=2025-01-01
// Response:
{
  "content": [
    {
      "id": 42,
      "categoryId": 1,
      "categoryName": "طعام ومشروبات",
      "amount": 150.50,
      "type": "EXPENSE",
      "description": "غداء مع الأصدقاء",
      "transactionDate": "2025-01-20",
      "createdAt": "2025-01-20T14:30:00"
    }
    // ... more items
  ],
  "page": 0,
  "size": 10,
  "totalElements": 45,
  "totalPages": 5
}

// GET /api/reports/summary?startDate=2025-01-01&endDate=2025-01-31
// Response:
{
  "period": {
    "startDate": "2025-01-01",
    "endDate": "2025-01-31"
  },
  "totalIncome": 15000.00,
  "totalExpense": 8500.00,
  "balance": 6500.00,
  "transactionCount": 45
}

// GET /api/reports/by-category?type=EXPENSE&startDate=2025-01-01&endDate=2025-01-31
// Response:
{
  "period": {
    "startDate": "2025-01-01",
    "endDate": "2025-01-31"
  },
  "type": "EXPENSE",
  "total": 8500.00,
  "categories": [
    {
      "categoryId": 1,
      "categoryName": "طعام ومشروبات",
      "amount": 3200.00,
      "percentage": 37.6,
      "transactionCount": 25
    },
    {
      "categoryId": 2,
      "categoryName": "مواصلات",
      "amount": 1500.00,
      "percentage": 17.6,
      "transactionCount": 12
    }
    // ... more categories
  ]
}
```

---

## الجزء الرابع: هيكل مشروع Spring Boot

### 4.1 إنشاء المشروع

**استخدم Spring Initializr:**
- اذهب إلى https://start.spring.io
- اختر:
  - Project: Maven
  - Language: Java
  - Spring Boot: 3.2.x (أحدث إصدار مستقر)
  - Group: com.yourname
  - Artifact: expense-tracker
  - Packaging: Jar
  - Java: 21

**Dependencies المطلوبة:**
```
- Spring Web
- Spring Data JPA
- Spring Security
- Spring Validation
- SQL Server Driver
- Lombok
- MapStruct (أضفه يدوياً)
```

### 4.2 هيكل المجلدات

```
src/main/java/com/yourname/expensetracker/
├── ExpenseTrackerApplication.java
├── config/
│   ├── SecurityConfig.java
│   ├── JwtConfig.java
│   └── WebConfig.java
├── controller/
│   ├── AuthController.java
│   ├── CategoryController.java
│   ├── TransactionController.java
│   ├── DebtController.java
│   └── ReportController.java
├── service/
│   ├── AuthService.java
│   ├── CategoryService.java
│   ├── TransactionService.java
│   ├── DebtService.java
│   └── ReportService.java
├── repository/
│   ├── UserRepository.java
│   ├── CategoryRepository.java
│   ├── TransactionRepository.java
│   └── DebtRepository.java
├── entity/
│   ├── User.java
│   ├── Category.java
│   ├── Transaction.java
│   └── Debt.java
├── dto/
│   ├── request/
│   │   ├── LoginRequest.java
│   │   ├── RegisterRequest.java
│   │   ├── TransactionRequest.java
│   │   └── ...
│   └── response/
│       ├── ApiResponse.java
│       ├── TransactionResponse.java
│       ├── ReportSummaryResponse.java
│       └── ...
├── mapper/
│   ├── TransactionMapper.java
│   └── CategoryMapper.java
├── exception/
│   ├── GlobalExceptionHandler.java
│   ├── ResourceNotFoundException.java
│   └── BusinessException.java
├── security/
│   ├── JwtTokenProvider.java
│   ├── JwtAuthenticationFilter.java
│   └── UserDetailsServiceImpl.java
└── util/
    └── DateUtils.java

src/main/resources/
├── application.yml
├── application-dev.yml
├── application-prod.yml
└── db/
    └── migration/  (لو هتستخدم Flyway)
        ├── V1__create_users_table.sql
        ├── V2__create_categories_table.sql
        └── ...
```

### 4.3 أمثلة على الكود

#### Entity Example:
```java
@Entity
@Table(name = "transactions")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Transaction {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id", nullable = false)
    private Category category;
    
    @Column(nullable = false, precision = 18, scale = 2)
    private BigDecimal amount;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 10)
    private TransactionType type;
    
    @Column(length = 500)
    private String description;
    
    @Column(name = "transaction_date", nullable = false)
    private LocalDate transactionDate;
    
    @CreationTimestamp
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
}
```

#### Repository Example:
```java
public interface TransactionRepository extends JpaRepository<Transaction, Long> {
    
    Page<Transaction> findByUserId(Long userId, Pageable pageable);
    
    @Query("""
        SELECT t FROM Transaction t 
        WHERE t.user.id = :userId 
        AND t.transactionDate BETWEEN :startDate AND :endDate
        AND (:type IS NULL OR t.type = :type)
        AND (:categoryId IS NULL OR t.category.id = :categoryId)
        """)
    Page<Transaction> findByFilters(
        @Param("userId") Long userId,
        @Param("startDate") LocalDate startDate,
        @Param("endDate") LocalDate endDate,
        @Param("type") TransactionType type,
        @Param("categoryId") Long categoryId,
        Pageable pageable
    );
    
    @Query("""
        SELECT SUM(t.amount) FROM Transaction t 
        WHERE t.user.id = :userId 
        AND t.type = :type
        AND t.transactionDate BETWEEN :startDate AND :endDate
        """)
    BigDecimal sumByTypeAndDateRange(
        @Param("userId") Long userId,
        @Param("type") TransactionType type,
        @Param("startDate") LocalDate startDate,
        @Param("endDate") LocalDate endDate
    );
    
    @Query("""
        SELECT t.category.id, t.category.name, SUM(t.amount), COUNT(t)
        FROM Transaction t 
        WHERE t.user.id = :userId 
        AND t.type = :type
        AND t.transactionDate BETWEEN :startDate AND :endDate
        GROUP BY t.category.id, t.category.name
        ORDER BY SUM(t.amount) DESC
        """)
    List<Object[]> sumByCategoryAndDateRange(
        @Param("userId") Long userId,
        @Param("type") TransactionType type,
        @Param("startDate") LocalDate startDate,
        @Param("endDate") LocalDate endDate
    );
}
```

#### Service Example:
```java
@Service
@RequiredArgsConstructor
@Transactional
public class TransactionService {
    
    private final TransactionRepository transactionRepository;
    private final CategoryRepository categoryRepository;
    private final TransactionMapper transactionMapper;
    
    public TransactionResponse create(Long userId, TransactionRequest request) {
        Category category = categoryRepository.findByIdAndUserId(request.getCategoryId(), userId)
            .orElseThrow(() -> new ResourceNotFoundException("Category not found"));
        
        Transaction transaction = Transaction.builder()
            .user(User.builder().id(userId).build())
            .category(category)
            .amount(request.getAmount())
            .type(request.getType())
            .description(request.getDescription())
            .transactionDate(request.getTransactionDate())
            .build();
        
        Transaction saved = transactionRepository.save(transaction);
        return transactionMapper.toResponse(saved);
    }
    
    @Transactional(readOnly = true)
    public Page<TransactionResponse> findAll(Long userId, TransactionFilter filter, Pageable pageable) {
        Page<Transaction> transactions = transactionRepository.findByFilters(
            userId,
            filter.getStartDate(),
            filter.getEndDate(),
            filter.getType(),
            filter.getCategoryId(),
            pageable
        );
        return transactions.map(transactionMapper::toResponse);
    }
    
    public TransactionResponse update(Long userId, Long id, TransactionRequest request) {
        Transaction transaction = transactionRepository.findById(id)
            .filter(t -> t.getUser().getId().equals(userId))
            .orElseThrow(() -> new ResourceNotFoundException("Transaction not found"));
        
        if (!transaction.getCategory().getId().equals(request.getCategoryId())) {
            Category category = categoryRepository.findByIdAndUserId(request.getCategoryId(), userId)
                .orElseThrow(() -> new ResourceNotFoundException("Category not found"));
            transaction.setCategory(category);
        }
        
        transaction.setAmount(request.getAmount());
        transaction.setType(request.getType());
        transaction.setDescription(request.getDescription());
        transaction.setTransactionDate(request.getTransactionDate());
        
        Transaction saved = transactionRepository.save(transaction);
        return transactionMapper.toResponse(saved);
    }
    
    public void delete(Long userId, Long id) {
        Transaction transaction = transactionRepository.findById(id)
            .filter(t -> t.getUser().getId().equals(userId))
            .orElseThrow(() -> new ResourceNotFoundException("Transaction not found"));
        
        transactionRepository.delete(transaction);
    }
}
```

#### Controller Example:
```java
@RestController
@RequestMapping("/api/transactions")
@RequiredArgsConstructor
public class TransactionController {
    
    private final TransactionService transactionService;
    
    @GetMapping
    public ResponseEntity<Page<TransactionResponse>> getAll(
            @AuthenticationPrincipal UserDetails userDetails,
            @Valid TransactionFilter filter,
            Pageable pageable) {
        Long userId = ((CustomUserDetails) userDetails).getId();
        Page<TransactionResponse> transactions = transactionService.findAll(userId, filter, pageable);
        return ResponseEntity.ok(transactions);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<TransactionResponse> getById(
            @AuthenticationPrincipal UserDetails userDetails,
            @PathVariable Long id) {
        Long userId = ((CustomUserDetails) userDetails).getId();
        TransactionResponse transaction = transactionService.findById(userId, id);
        return ResponseEntity.ok(transaction);
    }
    
    @PostMapping
    public ResponseEntity<TransactionResponse> create(
            @AuthenticationPrincipal UserDetails userDetails,
            @Valid @RequestBody TransactionRequest request) {
        Long userId = ((CustomUserDetails) userDetails).getId();
        TransactionResponse transaction = transactionService.create(userId, request);
        return ResponseEntity.status(HttpStatus.CREATED).body(transaction);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<TransactionResponse> update(
            @AuthenticationPrincipal UserDetails userDetails,
            @PathVariable Long id,
            @Valid @RequestBody TransactionRequest request) {
        Long userId = ((CustomUserDetails) userDetails).getId();
        TransactionResponse transaction = transactionService.update(userId, id, request);
        return ResponseEntity.ok(transaction);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(
            @AuthenticationPrincipal UserDetails userDetails,
            @PathVariable Long id) {
        Long userId = ((CustomUserDetails) userDetails).getId();
        transactionService.delete(userId, id);
        return ResponseEntity.noContent().build();
    }
}
```

### 4.4 الـ application.yml

```yaml
spring:
  application:
    name: expense-tracker
  
  datasource:
    url: jdbc:sqlserver://localhost:1433;databaseName=expense_tracker;encrypt=true;trustServerCertificate=true
    username: ${DB_USERNAME:sa}
    password: ${DB_PASSWORD:YourPassword}
    driver-class-name: com.microsoft.sqlserver.jdbc.SQLServerDriver
  
  jpa:
    hibernate:
      ddl-auto: validate  # استخدم Flyway للـ migrations
    show-sql: false
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.SQLServerDialect
  
  jackson:
    default-property-inclusion: non_null
    serialization:
      write-dates-as-timestamps: false

# JWT Configuration
jwt:
  secret: ${JWT_SECRET:your-256-bit-secret-key-here-make-it-long-enough}
  expiration: 86400000  # 24 hours in milliseconds
  refresh-expiration: 604800000  # 7 days

# Server Configuration
server:
  port: 8080
  servlet:
    context-path: /

# Logging
logging:
  level:
    com.yourname.expensetracker: DEBUG
    org.springframework.security: DEBUG
```

---

## الجزء الخامس: هيكل مشروع Vue.js

### 5.1 إنشاء المشروع

```bash
npm create vue@latest expense-tracker-frontend

# اختر:
# ✔ Add TypeScript? Yes
# ✔ Add JSX Support? No
# ✔ Add Vue Router? Yes
# ✔ Add Pinia? Yes
# ✔ Add Vitest? Yes
# ✔ Add ESLint? Yes
# ✔ Add Prettier? Yes
```

### 5.2 هيكل المجلدات

```
src/
├── main.ts
├── App.vue
├── assets/
│   └── styles/
│       ├── main.css
│       └── variables.css
├── components/
│   ├── common/
│   │   ├── AppButton.vue
│   │   ├── AppInput.vue
│   │   ├── AppModal.vue
│   │   ├── AppCard.vue
│   │   ├── LoadingSpinner.vue
│   │   └── EmptyState.vue
│   ├── layout/
│   │   ├── AppHeader.vue
│   │   ├── AppSidebar.vue
│   │   └── AppFooter.vue
│   ├── transactions/
│   │   ├── TransactionList.vue
│   │   ├── TransactionItem.vue
│   │   ├── TransactionForm.vue
│   │   └── TransactionFilter.vue
│   ├── categories/
│   │   ├── CategoryList.vue
│   │   └── CategoryForm.vue
│   ├── debts/
│   │   ├── DebtList.vue
│   │   └── DebtForm.vue
│   ├── reports/
│   │   ├── SummaryCard.vue
│   │   ├── ExpenseChart.vue
│   │   └── TrendChart.vue
│   └── dashboard/
│       ├── BalanceCard.vue
│       ├── RecentTransactions.vue
│       └── QuickActions.vue
├── views/
│   ├── auth/
│   │   ├── LoginView.vue
│   │   └── RegisterView.vue
│   ├── DashboardView.vue
│   ├── TransactionsView.vue
│   ├── CategoriesView.vue
│   ├── DebtsView.vue
│   ├── ReportsView.vue
│   └── SettingsView.vue
├── stores/
│   ├── auth.ts
│   ├── transactions.ts
│   ├── categories.ts
│   ├── debts.ts
│   └── reports.ts
├── composables/
│   ├── useApi.ts
│   ├── useAuth.ts
│   ├── useNotification.ts
│   └── useFormatters.ts
├── services/
│   ├── api.ts          # Axios instance
│   ├── authService.ts
│   ├── transactionService.ts
│   ├── categoryService.ts
│   ├── debtService.ts
│   └── reportService.ts
├── types/
│   ├── auth.ts
│   ├── transaction.ts
│   ├── category.ts
│   ├── debt.ts
│   └── api.ts
├── router/
│   └── index.ts
└── utils/
    ├── formatters.ts
    ├── validators.ts
    └── constants.ts
```

### 5.3 أمثلة على الكود

#### API Service (src/services/api.ts):
```typescript
import axios, { type AxiosInstance, type InternalAxiosRequestConfig } from 'axios'
import { useAuthStore } from '@/stores/auth'
import router from '@/router'

const api: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8080/api',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// Request interceptor - إضافة التوكن لكل طلب
api.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const authStore = useAuthStore()
    if (authStore.token) {
      config.headers.Authorization = `Bearer ${authStore.token}`
    }
    return config
  },
  (error) => Promise.reject(error)
)

// Response interceptor - التعامل مع الأخطاء
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const authStore = useAuthStore()
    
    if (error.response?.status === 401) {
      authStore.logout()
      router.push('/login')
    }
    
    return Promise.reject(error)
  }
)

export default api
```

#### Transaction Service (src/services/transactionService.ts):
```typescript
import api from './api'
import type { Transaction, TransactionRequest, TransactionFilter, PagedResponse } from '@/types'

export const transactionService = {
  async getAll(filter: TransactionFilter, page = 0, size = 20): Promise<PagedResponse<Transaction>> {
    const params = new URLSearchParams()
    params.append('page', page.toString())
    params.append('size', size.toString())
    
    if (filter.type) params.append('type', filter.type)
    if (filter.categoryId) params.append('categoryId', filter.categoryId.toString())
    if (filter.startDate) params.append('startDate', filter.startDate)
    if (filter.endDate) params.append('endDate', filter.endDate)
    
    const response = await api.get(`/transactions?${params}`)
    return response.data
  },
  
  async getById(id: number): Promise<Transaction> {
    const response = await api.get(`/transactions/${id}`)
    return response.data
  },
  
  async create(data: TransactionRequest): Promise<Transaction> {
    const response = await api.post('/transactions', data)
    return response.data
  },
  
  async update(id: number, data: TransactionRequest): Promise<Transaction> {
    const response = await api.put(`/transactions/${id}`, data)
    return response.data
  },
  
  async delete(id: number): Promise<void> {
    await api.delete(`/transactions/${id}`)
  }
}
```

#### Pinia Store (src/stores/transactions.ts):
```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { transactionService } from '@/services/transactionService'
import type { Transaction, TransactionRequest, TransactionFilter } from '@/types'

export const useTransactionStore = defineStore('transactions', () => {
  // State
  const transactions = ref<Transaction[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)
  const currentPage = ref(0)
  const totalPages = ref(0)
  const totalElements = ref(0)
  
  // Getters
  const incomeTransactions = computed(() => 
    transactions.value.filter(t => t.type === 'INCOME')
  )
  
  const expenseTransactions = computed(() => 
    transactions.value.filter(t => t.type === 'EXPENSE')
  )
  
  // Actions
  async function fetchTransactions(filter: TransactionFilter = {}, page = 0) {
    loading.value = true
    error.value = null
    
    try {
      const response = await transactionService.getAll(filter, page)
      transactions.value = response.content
      currentPage.value = response.page
      totalPages.value = response.totalPages
      totalElements.value = response.totalElements
    } catch (err: any) {
      error.value = err.response?.data?.message || 'حدث خطأ في جلب المعاملات'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  async function createTransaction(data: TransactionRequest) {
    loading.value = true
    error.value = null
    
    try {
      const transaction = await transactionService.create(data)
      transactions.value.unshift(transaction)
      return transaction
    } catch (err: any) {
      error.value = err.response?.data?.message || 'حدث خطأ في إضافة المعاملة'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  async function updateTransaction(id: number, data: TransactionRequest) {
    loading.value = true
    error.value = null
    
    try {
      const transaction = await transactionService.update(id, data)
      const index = transactions.value.findIndex(t => t.id === id)
      if (index !== -1) {
        transactions.value[index] = transaction
      }
      return transaction
    } catch (err: any) {
      error.value = err.response?.data?.message || 'حدث خطأ في تحديث المعاملة'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  async function deleteTransaction(id: number) {
    loading.value = true
    error.value = null
    
    try {
      await transactionService.delete(id)
      transactions.value = transactions.value.filter(t => t.id !== id)
    } catch (err: any) {
      error.value = err.response?.data?.message || 'حدث خطأ في حذف المعاملة'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  return {
    // State
    transactions,
    loading,
    error,
    currentPage,
    totalPages,
    totalElements,
    // Getters
    incomeTransactions,
    expenseTransactions,
    // Actions
    fetchTransactions,
    createTransaction,
    updateTransaction,
    deleteTransaction
  }
})
```

#### Component Example (src/components/transactions/TransactionForm.vue):
```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useCategoryStore } from '@/stores/categories'
import type { TransactionRequest, TransactionType } from '@/types'

// Props
const props = defineProps<{
  initialData?: TransactionRequest
  loading?: boolean
}>()

// Emits
const emit = defineEmits<{
  submit: [data: TransactionRequest]
  cancel: []
}>()

// Stores
const categoryStore = useCategoryStore()

// Form data
const form = ref<TransactionRequest>({
  categoryId: props.initialData?.categoryId || 0,
  amount: props.initialData?.amount || 0,
  type: props.initialData?.type || 'EXPENSE',
  description: props.initialData?.description || '',
  transactionDate: props.initialData?.transactionDate || new Date().toISOString().split('T')[0]
})

// Computed
const filteredCategories = computed(() => 
  categoryStore.categories.filter(c => c.type === form.value.type)
)

const isValid = computed(() => 
  form.value.categoryId > 0 && 
  form.value.amount > 0 && 
  form.value.transactionDate
)

// Methods
function handleTypeChange(type: TransactionType) {
  form.value.type = type
  form.value.categoryId = 0 // Reset category when type changes
}

function handleSubmit() {
  if (isValid.value) {
    emit('submit', { ...form.value })
  }
}

// Lifecycle
onMounted(() => {
  categoryStore.fetchCategories()
})
</script>

<template>
  <form @submit.prevent="handleSubmit" class="transaction-form">
    <!-- Transaction Type -->
    <div class="form-group">
      <label>نوع المعاملة</label>
      <div class="type-buttons">
        <button 
          type="button"
          :class="['type-btn', { active: form.type === 'EXPENSE' }]"
          @click="handleTypeChange('EXPENSE')"
        >
          مصروف
        </button>
        <button 
          type="button"
          :class="['type-btn', { active: form.type === 'INCOME' }]"
          @click="handleTypeChange('INCOME')"
        >
          دخل
        </button>
      </div>
    </div>
    
    <!-- Amount -->
    <div class="form-group">
      <label for="amount">المبلغ</label>
      <input
        id="amount"
        v-model.number="form.amount"
        type="number"
        step="0.01"
        min="0"
        required
        placeholder="0.00"
      />
    </div>
    
    <!-- Category -->
    <div class="form-group">
      <label for="category">التصنيف</label>
      <select id="category" v-model="form.categoryId" required>
        <option value="0" disabled>اختر التصنيف</option>
        <option 
          v-for="category in filteredCategories" 
          :key="category.id" 
          :value="category.id"
        >
          {{ category.name }}
        </option>
      </select>
    </div>
    
    <!-- Date -->
    <div class="form-group">
      <label for="date">التاريخ</label>
      <input
        id="date"
        v-model="form.transactionDate"
        type="date"
        required
      />
    </div>
    
    <!-- Description -->
    <div class="form-group">
      <label for="description">الوصف (اختياري)</label>
      <textarea
        id="description"
        v-model="form.description"
        rows="2"
        placeholder="أضف ملاحظة..."
      />
    </div>
    
    <!-- Actions -->
    <div class="form-actions">
      <button type="button" class="btn-secondary" @click="emit('cancel')">
        إلغاء
      </button>
      <button 
        type="submit" 
        class="btn-primary" 
        :disabled="!isValid || loading"
      >
        {{ loading ? 'جاري الحفظ...' : 'حفظ' }}
      </button>
    </div>
  </form>
</template>

<style scoped>
.transaction-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.type-buttons {
  display: flex;
  gap: 0.5rem;
}

.type-btn {
  flex: 1;
  padding: 0.75rem;
  border: 2px solid var(--border-color);
  border-radius: 8px;
  background: white;
  cursor: pointer;
  transition: all 0.2s;
}

.type-btn.active {
  border-color: var(--primary-color);
  background: var(--primary-light);
  color: var(--primary-color);
}

.form-actions {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
  margin-top: 1rem;
}
</style>
```

---

## الجزء السادس: الأمان والمصادقة (Authentication)

### 6.1 استراتيجية JWT

```
تدفق المصادقة:
1. المستخدم يرسل email/password
2. السيرفر يتحقق ويرجع Access Token + Refresh Token
3. الفرونت يحفظ التوكن في memory (وليس localStorage للأمان)
4. كل طلب يحتوي على Authorization header
5. عند انتهاء Access Token، يستخدم Refresh Token للتجديد
```

### 6.2 Spring Security Config

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    
    private final JwtAuthenticationFilter jwtAuthFilter;
    private final UserDetailsService userDetailsService;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(List.of("http://localhost:5173")); // Vue dev server
        configuration.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "PATCH"));
        configuration.setAllowedHeaders(List.of("*"));
        configuration.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) 
            throws Exception {
        return config.getAuthenticationManager();
    }
}
```

### 6.3 نصائح أمنية مهمة

```
✅ افعل:
- استخدم HTTPS في الإنتاج
- خزّن كلمات المرور مشفرة بـ BCrypt
- استخدم Prepared Statements لمنع SQL Injection
- حدد صلاحية التوكن (24 ساعة مثلاً)
- تحقق من ملكية البيانات في كل طلب
- استخدم Rate Limiting

❌ لا تفعل:
- لا تخزن التوكن في localStorage (عرضة لـ XSS)
- لا تضع معلومات حساسة في التوكن
- لا تظهر رسائل خطأ تفصيلية للمستخدم
- لا تثق بأي input من المستخدم
```

---

## الجزء السابع: الاختبارات

### 7.1 اختبارات Spring Boot

```java
// Unit Test للـ Service
@ExtendWith(MockitoExtension.class)
class TransactionServiceTest {
    
    @Mock
    private TransactionRepository transactionRepository;
    
    @Mock
    private CategoryRepository categoryRepository;
    
    @Mock
    private TransactionMapper transactionMapper;
    
    @InjectMocks
    private TransactionService transactionService;
    
    @Test
    void create_ShouldReturnTransaction_WhenValidRequest() {
        // Given
        Long userId = 1L;
        TransactionRequest request = new TransactionRequest();
        request.setCategoryId(1L);
        request.setAmount(new BigDecimal("100.00"));
        request.setType(TransactionType.EXPENSE);
        request.setTransactionDate(LocalDate.now());
        
        Category category = new Category();
        category.setId(1L);
        
        Transaction savedTransaction = new Transaction();
        savedTransaction.setId(1L);
        
        TransactionResponse expectedResponse = new TransactionResponse();
        expectedResponse.setId(1L);
        
        when(categoryRepository.findByIdAndUserId(1L, userId))
            .thenReturn(Optional.of(category));
        when(transactionRepository.save(any(Transaction.class)))
            .thenReturn(savedTransaction);
        when(transactionMapper.toResponse(savedTransaction))
            .thenReturn(expectedResponse);
        
        // When
        TransactionResponse result = transactionService.create(userId, request);
        
        // Then
        assertNotNull(result);
        assertEquals(1L, result.getId());
        verify(transactionRepository).save(any(Transaction.class));
    }
    
    @Test
    void create_ShouldThrowException_WhenCategoryNotFound() {
        // Given
        Long userId = 1L;
        TransactionRequest request = new TransactionRequest();
        request.setCategoryId(999L);
        
        when(categoryRepository.findByIdAndUserId(999L, userId))
            .thenReturn(Optional.empty());
        
        // When & Then
        assertThrows(ResourceNotFoundException.class, 
            () -> transactionService.create(userId, request));
    }
}

// Integration Test للـ Controller
@SpringBootTest
@AutoConfigureMockMvc
class TransactionControllerIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    @WithMockUser(username = "test@test.com")
    void createTransaction_ShouldReturn201() throws Exception {
        TransactionRequest request = new TransactionRequest();
        request.setCategoryId(1L);
        request.setAmount(new BigDecimal("100.00"));
        request.setType(TransactionType.EXPENSE);
        request.setTransactionDate(LocalDate.now());
        
        mockMvc.perform(post("/api/transactions")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").exists())
            .andExpect(jsonPath("$.amount").value(100.00));
    }
}
```

### 7.2 اختبارات Vue.js

```typescript
// Component Test
import { describe, it, expect, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import { createPinia, setActivePinia } from 'pinia'
import TransactionForm from '@/components/transactions/TransactionForm.vue'

describe('TransactionForm', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })
  
  it('renders correctly', () => {
    const wrapper = mount(TransactionForm)
    
    expect(wrapper.find('form').exists()).toBe(true)
    expect(wrapper.find('input[type="number"]').exists()).toBe(true)
    expect(wrapper.find('select').exists()).toBe(true)
  })
  
  it('emits submit event with form data', async () => {
    const wrapper = mount(TransactionForm)
    
    await wrapper.find('input[type="number"]').setValue(100)
    await wrapper.find('input[type="date"]').setValue('2025-01-20')
    await wrapper.find('form').trigger('submit')
    
    expect(wrapper.emitted('submit')).toBeTruthy()
  })
  
  it('disables submit button when form is invalid', () => {
    const wrapper = mount(TransactionForm)
    
    const submitButton = wrapper.find('button[type="submit"]')
    expect(submitButton.attributes('disabled')).toBeDefined()
  })
})

// Store Test
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useTransactionStore } from '@/stores/transactions'
import { transactionService } from '@/services/transactionService'

vi.mock('@/services/transactionService')

describe('Transaction Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
    vi.clearAllMocks()
  })
  
  it('fetches transactions successfully', async () => {
    const mockResponse = {
      content: [{ id: 1, amount: 100, type: 'EXPENSE' }],
      page: 0,
      totalPages: 1,
      totalElements: 1
    }
    
    vi.mocked(transactionService.getAll).mockResolvedValue(mockResponse)
    
    const store = useTransactionStore()
    await store.fetchTransactions()
    
    expect(store.transactions).toHaveLength(1)
    expect(store.transactions[0].amount).toBe(100)
  })
  
  it('handles error when fetching fails', async () => {
    vi.mocked(transactionService.getAll).mockRejectedValue(new Error('Network error'))
    
    const store = useTransactionStore()
    
    await expect(store.fetchTransactions()).rejects.toThrow()
    expect(store.error).toBeTruthy()
  })
})
```

---

## الجزء الثامن: النشر (Deployment)

### 8.1 إعداد Docker

```dockerfile
# Backend Dockerfile
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY target/expense-tracker-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```dockerfile
# Frontend Dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourStrong!Password
    ports:
      - "1433:1433"
    volumes:
      - sqlserver_data:/var/opt/mssql

  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:sqlserver://db:1433;databaseName=expense_tracker;encrypt=true;trustServerCertificate=true
      - SPRING_DATASOURCE_USERNAME=sa
      - SPRING_DATASOURCE_PASSWORD=YourStrong!Password
      - JWT_SECRET=your-production-secret
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  sqlserver_data:
```

### 8.2 خيارات الاستضافة

```
خيارات مجانية/رخيصة للـ Portfolio:

1. Railway.app (مفضل للمبتدئين)
   - يدعم Docker
   - سهل الإعداد
   - $5 credits شهرياً مجاناً

2. Render.com
   - Backend: Web Service
   - Database: PostgreSQL مجاني (بديل عن SQL Server)
   - Frontend: Static Site

3. Vercel (للفرونت فقط)
   - ممتاز لـ Vue.js
   - مجاني تماماً

4. Azure
   - لديك credits مجانية كمطور
   - أفضل خيار لـ SQL Server
```

---

## الجزء التاسع: التوثيق

### 9.1 README.md للمشروع

```markdown
# 💰 Expense Tracker

نظام متكامل لتتبع النفقات والدخل مبني بـ Spring Boot + Vue.js

## 🚀 المميزات

- ✅ تسجيل المعاملات المالية
- ✅ تصنيف المصروفات والدخل
- ✅ تقارير شهرية مفصلة
- ✅ إدارة الديون
- ✅ رسوم بيانية تفاعلية
- ✅ واجهة عربية كاملة

## 🛠️ التقنيات المستخدمة

### Backend
- Java 21
- Spring Boot 3.2
- Spring Security + JWT
- Spring Data JPA
- SQL Server

### Frontend
- Vue 3 + Composition API
- TypeScript
- Pinia (State Management)
- Vue Router
- Chart.js

## 📦 التشغيل المحلي

### المتطلبات
- Java 21+
- Node.js 20+
- SQL Server

### Backend
```bash
cd backend
./mvnw spring-boot:run
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

## 📸 Screenshots

[أضف صور للتطبيق]

## 📄 License

MIT
```

### 9.2 API Documentation

استخدم **Swagger/OpenAPI**:

```java
// أضف للـ pom.xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

الآن يمكنك الوصول للتوثيق عبر: `http://localhost:8080/swagger-ui.html`

---

## الجزء العاشر: خطة العمل المقترحة

### جدول زمني مقترح (8-12 أسبوع)

```
الأسبوع 1-2: التخطيط والتصميم
├── رسم Wireframes
├── تصميم قاعدة البيانات
├── كتابة API Specification
└── إعداد بيئة التطوير

الأسبوع 3-4: Backend الأساسي
├── إعداد Spring Boot Project
├── إنشاء Entities و Repositories
├── Authentication (JWT)
└── CRUD للتصنيفات والمعاملات

الأسبوع 5-6: Frontend الأساسي
├── إعداد Vue Project
├── تصميم Layout الرئيسي
├── صفحات Auth (Login/Register)
└── قائمة المعاملات + Form

الأسبوع 7-8: الميزات المتقدمة
├── التقارير والإحصائيات
├── الرسوم البيانية
├── إدارة الديون
└── Dashboard

الأسبوع 9-10: التحسينات
├── كتابة Tests
├── تحسين UX
├── معالجة الأخطاء
└── تحسين الأداء

الأسبوع 11-12: النشر والتوثيق
├── إعداد Docker
├── النشر على السيرفر
├── كتابة README
└── تحضير للـ Portfolio
```

---

## نصائح ختامية

### ✅ نصائح للنجاح:

1. **ابدأ بسيط**: خلص MVP الأول قبل أي ميزة إضافية
2. **Git من اليوم الأول**: commit بشكل منتظم مع رسائل واضحة
3. **لا تتوقف عند المشاكل**: جوجل و Stack Overflow أصدقائك
4. **اطلب Code Review**: شارك كودك مع زملائك
5. **وثّق كل شيء**: المشروع الموثق جيداً يفرق في المقابلات

### 📚 مصادر للتعلم:

- Spring Boot: [Baeldung](https://www.baeldung.com)
- Vue.js: [Vue.js Documentation](https://vuejs.org)
- SQL: [SQLServerTutorial](https://www.sqlservertutorial.net)

### 💡 أفكار للتطوير المستقبلي:

- تطبيق موبايل بـ Flutter أو React Native
- OCR لقراءة الفواتير
- ربط مع البنوك
- تقارير متقدمة بـ AI
- Multi-tenancy للشركات

---

**بالتوفيق في مشروعك! 🚀**

لو عندك أي سؤال أثناء التطوير، أنا موجود للمساعدة.
