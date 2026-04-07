# 🍃 دليلك الشامل لملفات مشروع Spring Boot

> شرح مبسط من الصفر — هنمشي على كل ملف وكل مجلد ونشرح إيه دوره بالظبط.

---

## 📁 أولاً: الشكل الكامل لمشروع Spring Boot

لما بتعمل مشروع Spring Boot جديد (من [start.spring.io](https://start.spring.io) مثلاً)، هيجيلك كده:

```
my-spring-app/
│
├── 📄 pom.xml                              ← إدارة الـ Dependencies (لو Maven)
│
├── 📁 src/
│   ├── 📁 main/
│   │   ├── 📁 java/
│   │   │   └── com/namasoft/myapp/
│   │   │       ├── 📄 MyAppApplication.java     ← نقطة بداية التطبيق
│   │   │       ├── 📁 controller/               ← طبقة الـ API
│   │   │       ├── 📁 service/                  ← طبقة الـ Business Logic
│   │   │       ├── 📁 repository/               ← طبقة الـ Database
│   │   │       ├── 📁 model/  (أو entity/)      ← الـ Data Classes
│   │   │       └── 📁 config/                   ← إعدادات خاصة
│   │   │
│   │   └── 📁 resources/
│   │       ├── 📄 application.properties        ← إعدادات التطبيق الرئيسية
│   │       ├── 📁 static/                       ← CSS, JS, Images
│   │       └── 📁 templates/                    ← HTML Templates (Thymeleaf)
│   │
│   └── 📁 test/
│       └── 📁 java/
│           └── com/namasoft/myapp/
│               └── 📄 MyAppApplicationTests.java
│
├── 📄 .gitignore
├── 📄 .mvnw  /  mvnw.cmd                   ← Maven Wrapper
└── 📁 .mvn/
    └── wrapper/
        └── maven-wrapper.properties
```

---

## 🔍 ثانياً: شرح كل ملف بالتفصيل

---

### 1️⃣ `MyAppApplication.java` — قلب التطبيق ونقطة البداية

ده **أول ملف** لازم تفهمه — ده اللي بيشغّل كل حاجة.

```java
package com.namasoft.myapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication                           // ← Annotation واحدة بتعمل ماجيك كتير!
public class MyAppApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyAppApplication.class, args);  // ← بيشغّل التطبيق
    }
}
```

#### `@SpringBootApplication` — إيه اللي بيعمله بالظبط؟

دي في الحقيقة **3 Annotations في واحدة**:

```
@SpringBootApplication
    ├── @SpringBootConfiguration   ← بيقول "الكلاس ده فيه إعدادات Spring"
    ├── @EnableAutoConfiguration   ← ← ← الماجيك الحقيقي! (هنشرحه دلوقتي)
    └── @ComponentScan             ← بيدوّر على الـ Components في نفس الـ Package وتحتيه
```

#### `@EnableAutoConfiguration` — إيه الماجيك ده؟

Spring Boot بيبص على الـ Dependencies اللي حاطّها في الـ `pom.xml` ويعمل إعدادات أوتوماتيك:

```
لو حاطّ spring-boot-starter-web    → يعمل Tomcat + MVC أوتوماتيك
لو حاطّ spring-boot-starter-data-jpa → يعمل EntityManager + Transactions أوتوماتيك
لو حاطّ spring-boot-starter-security → يفعّل Security أوتوماتيك
```

بدل Spring Boot كنت هتكتب مئات الأسطر من الـ XML Config يدوياً!

---

### 2️⃣ `pom.xml` — مدير المشروع

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <modelVersion>4.0.0</modelVersion>

    <!-- ① وراثة إعدادات Spring Boot الأساسية -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <!-- ② هوية مشروعك -->
    <groupId>com.namasoft</groupId>
    <artifactId>my-app</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>   <!-- أو war لو هتنزله على Tomcat خارجي -->

    <!-- ③ إصدار Java -->
    <properties>
        <java.version>17</java.version>
    </properties>

    <!-- ④ المكتبات اللي محتاجها -->
    <dependencies>

        <!-- Web: REST APIs + Tomcat مدموج -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Database: JPA + Hibernate -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- SQL Server Driver -->
        <dependency>
            <groupId>com.microsoft.sqlserver</groupId>
            <artifactId>mssql-jdbc</artifactId>
        </dependency>

        <!-- للـ Tests -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <!-- ⑤ Plugin عشان يعمل Fat JAR -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

#### الـ `spring-boot-starter-parent` — ليه مهم جداً؟

ده Parent POM بيجيبلك:
- إصدارات المكتبات كلها متوافقة مع بعض — **مش محتاج تكتب `<version>` لأي Starter**
- إعدادات Maven plugins جاهزة
- إعدادات الـ Encoding والـ Java version

```xml
<!-- بدون parent: لازم تكتب version لكل حاجة -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.2.0</version>   ← محتاج تحدده يدوي وتتأكد من التوافق
</dependency>

<!-- مع parent: Spring Boot بيتكفّل بالإصدارات -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- مفيش version! Spring Boot اختارها أوتوماتيك ✅ -->
</dependency>
```

---

### 3️⃣ `application.properties` — ملف الإعدادات الرئيسي

ده الملف اللي بتحط فيه كل إعدادات التطبيق — Database، Port، Logging، وأي حاجة تانية.

```properties
# ── Server ──────────────────────────────────────
server.port=8080
server.servlet.context-path=/api        # التطبيق يبقى على /api/...

# ── Database (SQL Server) ────────────────────────
spring.datasource.url=jdbc:sqlserver://localhost:1433;databaseName=NamaDB
spring.datasource.username=sa
spring.datasource.password=YourPassword
spring.datasource.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver

# ── JPA / Hibernate ──────────────────────────────
spring.jpa.hibernate.ddl-auto=update          # update / create / validate / none
spring.jpa.show-sql=true                      # اطبع الـ SQL في الـ Console
spring.jpa.properties.hibernate.format_sql=true

# ── Logging ──────────────────────────────────────
logging.level.root=INFO
logging.level.com.namasoft=DEBUG              # Debug لكودك بس
logging.file.name=logs/app.log

# ── Custom Properties (تعمّلها بنفسك) ───────────
app.name=Nama ERP
app.version=2.5.0
app.max-upload-size=10MB
```

#### `application.properties` vs `application.yml` — إيه الفرق؟

نفس الحاجة بالظبط — بس شكل مختلف:

```properties
# application.properties
spring.datasource.url=jdbc:sqlserver://localhost:1433
spring.datasource.username=sa
spring.jpa.show-sql=true
```

```yaml
# application.yml
spring:
  datasource:
    url: jdbc:sqlserver://localhost:1433
    username: sa
  jpa:
    show-sql: true
```

الـ YAML أقل تكرار وأوضح في التنظيم — بس الاتنين شغّالين. Spring Boot بيقرا الاتنين.

---

### 4️⃣ Profiles — إعدادات مختلفة لكل بيئة

مشكلة شائعة: إزاي تشغّل التطبيق بـ Database مختلفة على Dev و Production؟

**الحل: Profiles!**

```
resources/
├── application.properties           ← إعدادات مشتركة بين كل البيئات
├── application-dev.properties       ← إعدادات بيئة التطوير
├── application-staging.properties   ← إعدادات بيئة الاختبار
└── application-prod.properties      ← إعدادات الـ Production
```

```properties
# application.properties (مشترك)
app.name=Nama ERP
spring.profiles.active=dev           ← النشط افتراضياً هو dev

# application-dev.properties
spring.datasource.url=jdbc:sqlserver://localhost:1433;databaseName=NamaDB_DEV
spring.jpa.show-sql=true
logging.level.root=DEBUG

# application-prod.properties
spring.datasource.url=jdbc:sqlserver://prod-server:1433;databaseName=NamaDB
spring.jpa.show-sql=false
logging.level.root=WARN
```

#### تفعيل Profile معين:

```bash
# وقت التشغيل
java -jar app.jar --spring.profiles.active=prod

# أو عن طريق Environment Variable
export SPRING_PROFILES_ACTIVE=prod
java -jar app.jar
```

---

### 5️⃣ طبقات المشروع (Layers) — الـ Package Structure

Spring Boot بيشجّعك تنظّم كودك في طبقات. كل طبقة ليها مسؤولية واحدة بس.

```
com.namasoft.myapp/
│
├── 📁 controller/      ← طبقة الـ API (بتستقبل الـ HTTP Requests)
├── 📁 service/         ← طبقة الـ Business Logic
├── 📁 repository/      ← طبقة الـ Database
├── 📁 model/           ← الـ Entities (جداول الـ Database)
├── 📁 dto/             ← Data Transfer Objects (بيانات بتتبعت للـ API)
└── 📁 config/          ← إعدادات خاصة
```

#### رحلة الـ Request من الأول للآخر:

```
المتصفح / Client
      ↓ HTTP Request
  Controller          ← بيستقبل الـ Request ويحوّله
      ↓
   Service            ← بيعمل الـ Business Logic
      ↓
  Repository          ← بيتكلم مع الـ Database
      ↓
   Database           ← SQL Server / MySQL / ...
      ↑
  Repository          ← بيرجع البيانات
      ↑
   Service            ← بيعالج البيانات
      ↑
  Controller          ← بيحوّلها لـ JSON ويرجعها
      ↑
المتصفح / Client      ← HTTP Response (JSON)
```

---

### 6️⃣ `@Controller` و `@RestController` — طبقة الـ API

```java
package com.namasoft.myapp.controller;

import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController                          // ← Controller + بيرجع JSON أوتوماتيك
@RequestMapping("/api/users")            // ← كل الـ Endpoints تبدأ بـ /api/users
public class UserController {

    private final UserService userService;

    // Constructor Injection (الأفضل دايماً)
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping                          // GET /api/users
    public List<UserDTO> getAllUsers() {
        return userService.findAll();
    }

    @GetMapping("/{id}")                 // GET /api/users/5
    public UserDTO getUserById(@PathVariable Long id) {
        return userService.findById(id);
    }

    @PostMapping                         // POST /api/users
    public UserDTO createUser(@RequestBody UserDTO userDTO) {
        return userService.save(userDTO);
    }

    @PutMapping("/{id}")                 // PUT /api/users/5
    public UserDTO updateUser(@PathVariable Long id, @RequestBody UserDTO userDTO) {
        return userService.update(id, userDTO);
    }

    @DeleteMapping("/{id}")              // DELETE /api/users/5
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

#### `@Controller` vs `@RestController`:

| | `@Controller` | `@RestController` |
|-|--------------|-------------------|
| **بيرجع إيه؟** | HTML Page (View) | JSON / XML |
| **متى تستخدمه؟** | لو بتعمل MVC مع Thymeleaf | لو بتعمل REST API |
| **يحتاج `@ResponseBody`؟** | ✅ على كل Method | ❌ مدموجة فيه |

---

### 7️⃣ `@Service` — طبقة الـ Business Logic

```java
package com.namasoft.myapp.service;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;

@Service                                  // ← بيقول لـ Spring "الكلاس ده Component"
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public List<UserDTO> findAll() {
        return userRepository.findAll()
                .stream()
                .map(this::toDTO)
                .toList();
    }

    public UserDTO findById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("User not found: " + id));
        return toDTO(user);
    }

    @Transactional                        // ← لو فيه أكتر من عملية Database، بيعملهم في Transaction واحدة
    public UserDTO save(UserDTO dto) {
        User user = toEntity(dto);
        User saved = userRepository.save(user);
        return toDTO(saved);
    }

    // Helper Methods
    private UserDTO toDTO(User user) { /* ... */ }
    private User toEntity(UserDTO dto) { /* ... */ }
}
```

---

### 8️⃣ `@Repository` — طبقة الـ Database

```java
package com.namasoft.myapp.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.util.List;

// Spring Data JPA بيعمل الـ Implementation أوتوماتيك! مش محتاج تكتب SQL
public interface UserRepository extends JpaRepository<User, Long> {

    // Spring بيفهم الاسم ويعمل SQL أوتوماتيك ✨
    List<User> findByEmail(String email);
    List<User> findByNameContainingIgnoreCase(String name);
    List<User> findByActiveTrue();

    // أو تكتب JPQL بنفسك
    @Query("SELECT u FROM User u WHERE u.department.id = :deptId AND u.active = true")
    List<User> findActiveUsersByDepartment(Long deptId);

    // أو Native SQL لو محتاج
    @Query(value = "SELECT * FROM users WHERE YEAR(created_at) = :year", nativeQuery = true)
    List<User> findUsersCreatedInYear(int year);
}
```

#### الـ Methods الجاهزة في `JpaRepository`:

```java
userRepository.findAll()              // كل الـ Records
userRepository.findById(id)          // بـ ID
userRepository.save(user)            // Insert أو Update
userRepository.delete(user)          // حذف
userRepository.count()               // العدد الكلي
userRepository.existsById(id)        // موجود ولا لأ؟
```

---

### 9️⃣ `@Entity` (Model) — تمثيل جدول الـ Database

```java
package com.namasoft.myapp.model;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity                                    // ← بيقول لـ Hibernate "ده جدول في الـ DB"
@Table(name = "users")                     // ← اسم الجدول (لو مختلف عن اسم الكلاس)
public class User {

    @Id                                    // ← Primary Key
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // ← Auto Increment
    private Long id;

    @Column(name = "full_name", nullable = false, length = 100)
    private String name;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(name = "is_active")
    private boolean active = true;

    @ManyToOne                             // ← علاقة: User ينتمي لـ Department واحد
    @JoinColumn(name = "department_id")
    private Department department;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders;            // ← علاقة: User عنده كتير Orders

    @CreationTimestamp
    private LocalDateTime createdAt;

    @UpdateTimestamp
    private LocalDateTime updatedAt;

    // Getters & Setters...
}
```

---

### 🔟 `DTO` — Data Transfer Object

```java
package com.namasoft.myapp.dto;

// DTO = البيانات اللي بتبعتها للـ Client (مش Entity كاملة)
// بيحميك من إنك تكشف بيانات حساسة زي الـ Password
public class UserDTO {

    private Long id;
    private String name;
    private String email;
    private String departmentName;
    // ملاحظة: مفيش password هنا! ✅

    // Constructors, Getters, Setters...
}
```

#### ليه DTO مهم؟

```
❌ بدون DTO: بترجع الـ Entity كاملة → بتكشف كل حاجة حتى الـ Password وبيانات حساسة
✅ مع DTO:   بترجع بس اللي الـ Client يحتاجه → آمن ومنظّم
```

---

### 1️⃣1️⃣ `config/` — إعدادات خاصة

```java
package com.namasoft.myapp.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration                          // ← بيقول لـ Spring "الكلاس ده فيه إعدادات"
public class AppConfig {

    // Bean = Object بيتعمله Spring وبيديه لأي حد يطلبه (Dependency Injection)
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
        return mapper;
    }
}
```

---

### 1️⃣2️⃣ `static/` و `templates/` — الـ Frontend

```
resources/
├── static/                  ← ملفات بتتخدم مباشرة بدون معالجة
│   ├── css/
│   │   └── style.css        ← http://localhost:8080/css/style.css
│   ├── js/
│   │   └── app.js           ← http://localhost:8080/js/app.js
│   └── images/
│       └── logo.png         ← http://localhost:8080/images/logo.png
│
└── templates/               ← HTML بيتعالجه Thymeleaf قبل ما يتبعت للمتصفح
    ├── index.html
    ├── users/
    │   ├── list.html
    │   └── form.html
    └── layout/
        └── base.html
```

> 💡 لو بتعمل REST API بس (وعندك Vue.js أو React Frontend منفصل)، مش هتحتاج `templates/` خالص.

---

### 1️⃣3️⃣ `MyAppApplicationTests.java` — الـ Tests

```java
package com.namasoft.myapp;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest                          // ← بيشغّل الـ Application Context كامل للـ Test
class MyAppApplicationTests {

    @Test
    void contextLoads() {
        // أبسط Test — بيتأكد إن التطبيق بيشتغل من غير أخطاء
    }
}
```

---

### 1️⃣4️⃣ Maven Wrapper (`mvnw`) — Maven من غير تثبيت

```
mvnw        ← للـ Linux / Mac
mvnw.cmd    ← للـ Windows
```

دول بيخلّوك تشغّل Maven **بدون ما يكون مثبّت على جهازك**. مفيد جداً في:
- CI/CD Pipelines
- أجهزة جديدة في الفريق
- ضمان إن كل الفريق بيستخدم نفس إصدار Maven

```bash
# بدل:
mvn clean install

# بتكتب:
./mvnw clean install        # Linux/Mac
mvnw.cmd clean install      # Windows
```

---

### 1️⃣5️⃣ `.gitignore` — إيه اللي مش بتحطه على Git

```gitignore
# Maven build output
target/

# IDE files
.idea/
*.iml
.eclipse/
.settings/

# OS files
.DS_Store
Thumbs.db

# Secrets (مهم جداً!)
application-prod.properties
*.env
```

> ⚠️ **تحذير:** أي ملف فيه Passwords أو API Keys (زي `application-prod.properties`) **لازم يكون في .gitignore** — متحطهوش على Git أبداً!

---

## 🧩 الـ Annotations الأساسية — مرجع سريع

| Annotation | بيتحط على | وظيفته |
|-----------|-----------|--------|
| `@SpringBootApplication` | Main Class | بيشغّل كل حاجة |
| `@RestController` | Class | Controller بيرجع JSON |
| `@Controller` | Class | Controller بيرجع HTML |
| `@Service` | Class | Business Logic Layer |
| `@Repository` | Interface/Class | Database Layer |
| `@Component` | Class | Generic Spring Component |
| `@Configuration` | Class | فيه Beans وإعدادات |
| `@Entity` | Class | جدول في الـ Database |
| `@Bean` | Method | بيعمل Object يديره Spring |
| `@Autowired` | Field/Constructor | Dependency Injection |
| `@Transactional` | Method/Class | Database Transaction |
| `@GetMapping` | Method | HTTP GET |
| `@PostMapping` | Method | HTTP POST |
| `@PutMapping` | Method | HTTP PUT |
| `@DeleteMapping` | Method | HTTP DELETE |
| `@RequestBody` | Parameter | بياخد JSON من الـ Request |
| `@PathVariable` | Parameter | بياخد قيمة من الـ URL |
| `@RequestParam` | Parameter | بياخد Query Parameter |

---

## 🌊 الـ Starters — المكتبات الجاهزة

الـ Starter هو مجموعة Dependencies مجمّعة في اسم واحد — بدل ما تضيف 10 مكتبات، بتضيف Starter واحد:

| Starter | بيجيبلك |
|---------|---------|
| `spring-boot-starter-web` | Tomcat + Spring MVC + Jackson (JSON) |
| `spring-boot-starter-data-jpa` | Hibernate + Spring Data + Connection Pool |
| `spring-boot-starter-security` | Spring Security كامل |
| `spring-boot-starter-test` | JUnit + Mockito + AssertJ |
| `spring-boot-starter-mail` | JavaMail للبريد الإلكتروني |
| `spring-boot-starter-cache` | Caching abstraction |
| `spring-boot-starter-validation` | Bean Validation (Hibernate Validator) |
| `spring-boot-starter-actuator` | Monitoring endpoints (Health, Metrics) |
| `spring-boot-starter-websocket` | WebSocket + STOMP |

---

## 🗺️ خريطة المشروع الكاملة — من Request لـ Response

```
① Client يبعت HTTP Request
        ↓
② Filter Chain (Security, Logging...)
        ↓
③ DispatcherServlet  ← بيوزّع الـ Requests على الـ Controllers الصح
        ↓
④ @RestController    ← بياخد الـ Request ويحوّله
        ↓
⑤ @Service           ← بيعمل الـ Business Logic
        ↓
⑥ @Repository        ← بيتكلم مع الـ Database
        ↓
⑦ Database           ← SQL Server
        ↑
⑥ @Repository        ← بيرجع البيانات
        ↑
⑤ @Service           ← بيحوّل Entity لـ DTO
        ↑
④ @RestController    ← Jackson بيحوّل الـ DTO لـ JSON
        ↑
② Filter Chain
        ↑
① Client استلم HTTP Response (JSON)
```

---

## 📌 ملخص نهائي

```
pom.xml                   → إدارة المكتبات والبناء
MyAppApplication.java     → نقطة البداية + Auto Configuration
application.properties    → كل إعدادات التطبيق
@Controller / @RestController → بيستقبل الـ HTTP Requests
@Service                  → Business Logic
@Repository               → Database Access
@Entity                   → تمثيل الجداول
DTO                       → البيانات اللي بتبعتها للـ Client
static/                   → CSS, JS, Images
templates/                → HTML (لو بتستخدم Thymeleaf)
test/                     → Unit & Integration Tests
```

> 📌 **نصيحة:** في Nama ERP اللي شغّال Spring + Hibernate على Tomcat خارجي، معظم الـ Concepts دي موجودة — بس من غير Spring Boot's Auto Configuration. Spring Boot بيأتمت الـ Setup اللي أنتوا بتعملوه يدوياً في الـ XML Config.
