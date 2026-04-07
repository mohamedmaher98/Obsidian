# Spring Boot — الـ Controller والـ Request Flow
> مذاكرة شاملة من الأول للآخر

---

## 📌 الفهرس
1. [الـ Controller — هو إيه؟](#-الـ-controller--هو-إيه)
2. [الـ Annotations الأساسية](#-الـ-annotations-الأساسية)
3. [الـ Request Mapping](#-الـ-request-mapping)
4. [إزاي تاخد البيانات من الـ Request](#-إزاي-تاخد-البيانات-من-الـ-request)
5. [الـ Response — بترد إزاي؟](#-الـ-response--بترد-إزاي)
6. [الـ Validation](#-الـ-validation)
7. [الـ Exception Handling](#-الـ-exception-handling)
8. [Controller احترافي — مثال كامل](#-controller-احترافي--مثال-كامل)
9. [رحلة الـ Request — من أول لآخر](#-رحلة-الـ-request--من-أول-لآخر)
10. [الـ DispatcherServlet بالتفصيل](#-الـ-dispatcherservlet-بالتفصيل)
11. [الـ HandlerMapping بالتفصيل](#-الـ-handlermapping-بالتفصيل)

---

## 🔹 الـ Controller — هو إيه؟

تخيل إن التطبيق بتاعك عبارة عن مطعم:
- **الـ Client** (المتصفح أو أي تطبيق) هو الزبون اللي بيطلب
- **الـ Controller** هو الـ Waiter — بياخد الطلب، يوديه للمطبخ، ويرجع بالأكل
- **الـ Service** هو المطبخ اللي بيعمل الشغل الحقيقي

> ⚠️ الـ Controller مش بيعمل business logic — دوره إنه **يستقبل الـ HTTP request، يفهمه، ويرد عليه.**

---

## 🔹 الـ Annotations الأساسية

### `@Controller` vs `@RestController`

```java
@Controller
public class PageController {
    // ده بيرجع اسم HTML page (View)
    @GetMapping("/home")
    public String home() {
        return "home"; // بيدور على home.html
    }
}
```

```java
@RestController
public class ApiController {
    // ده بيرجع Data (JSON/XML) مباشرةً
    @GetMapping("/users")
    public List<User> getUsers() {
        return userService.findAll();
    }
}
```

> 💡 `@RestController` = `@Controller` + `@ResponseBody` على كل method

---

## 🔹 الـ Request Mapping

### `@RequestMapping` — الأم

```java
@RestController
@RequestMapping("/api/users")  // Base path لكل الـ methods جوه
public class UserController {
    // كل الـ endpoints هتبقى /api/users/...
}
```

### الـ Shortcuts (اللي بتستخدمها يومياً)

| Annotation | HTTP Method | الاستخدام |
|---|---|---|
| `@GetMapping` | GET | جيب بيانات |
| `@PostMapping` | POST | أنشئ حاجة جديدة |
| `@PutMapping` | PUT | عدّل كل الـ resource |
| `@PatchMapping` | PATCH | عدّل جزء منه بس |
| `@DeleteMapping` | DELETE | احذف |

---

## 🔹 إزاي تاخد البيانات من الـ Request

### 1. `@PathVariable` — من الـ URL نفسه

```java
// GET /api/users/42
@GetMapping("/{id}")
public User getById(@PathVariable Long id) {
    return userService.findById(id);
}

// لو الاسم مختلف
@GetMapping("/{userId}")
public User getById(@PathVariable("userId") Long id) { ... }
```

### 2. `@RequestParam` — من الـ Query String

```java
// GET /api/users?page=0&size=10&name=ahmed
@GetMapping
public List<User> getAll(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(required = false) String name
) { ... }
```

### 3. `@RequestBody` — من الـ Body (JSON)

```java
// POST /api/users
// Body: { "name": "Ahmed", "email": "ahmed@test.com" }
@PostMapping
public User create(@RequestBody UserDto dto) {
    return userService.create(dto);
}
```

### 4. `@RequestHeader` — من الـ Headers

```java
@GetMapping("/profile")
public User getProfile(@RequestHeader("Authorization") String token) {
    // مثلاً لو بتعمل validation يدوي
}
```

---

## 🔹 الـ Response — بترد إزاي؟

### الطريقة البسيطة — رجّع Object مباشرةً

```java
@GetMapping("/{id}")
public User getById(@PathVariable Long id) {
    return userService.findById(id); // Spring هيحوله JSON أوتوماتيك
}
```

### الطريقة الاحترافية — `ResponseEntity`

```java
@GetMapping("/{id}")
public ResponseEntity<User> getById(@PathVariable Long id) {
    User user = userService.findById(id);

    if (user == null) {
        return ResponseEntity.notFound().build();         // 404
    }

    return ResponseEntity.ok(user);                       // 200 + body
}

@PostMapping
public ResponseEntity<User> create(@RequestBody UserDto dto) {
    User created = userService.create(dto);
    return ResponseEntity.status(HttpStatus.CREATED).body(created); // 201
}
```

> 💡 `ResponseEntity` بيديك تحكم كامل في الـ **Status Code + Headers + Body**

---

## 🔹 الـ Validation

```java
@PostMapping
public ResponseEntity<User> create(@Valid @RequestBody UserDto dto) {
    // @Valid بتشغّل الـ annotations اللي على الـ DTO
    return ResponseEntity.ok(userService.create(dto));
}
```

```java
public class UserDto {
    @NotBlank(message = "الاسم مطلوب")
    private String name;

    @Email(message = "إيميل غلط")
    private String email;

    @Min(value = 18, message = "السن لازم 18+")
    private int age;
}
```

---

## 🔹 الـ Exception Handling

### `@ExceptionHandler` — جوّا الـ Controller

```java
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<String> handleNotFound(UserNotFoundException ex) {
    return ResponseEntity.status(404).body(ex.getMessage());
}
```

### `@ControllerAdvice` — Global (الأفضل)

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(404)
            .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors()
            .stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return ResponseEntity.badRequest().body(new ErrorResponse("VALIDATION", message));
    }
}
```

---

## 🔹 Controller احترافي — مثال كامل

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor  // Lombok - بيعمل Constructor injection
public class UserController {

    private final UserService userService;

    @GetMapping
    public ResponseEntity<Page<UserDto>> getAll(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return ResponseEntity.ok(userService.findAll(page, size));
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }

    @PostMapping
    public ResponseEntity<UserDto> create(@Valid @RequestBody CreateUserRequest request) {
        UserDto created = userService.create(request);
        URI location = URI.create("/api/v1/users/" + created.getId());
        return ResponseEntity.created(location).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<UserDto> update(
            @PathVariable Long id,
            @Valid @RequestBody UpdateUserRequest request) {
        return ResponseEntity.ok(userService.update(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build(); // 204
    }
}
```

---

## 🔹 رحلة الـ Request — من أول لآخر

### الصورة الكاملة

```
Client (Browser/Mobile/Postman)
         │
         │  HTTP Request
         ▼
┌─────────────────────┐
│      Network        │
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│   Tomcat (Server)   │  ◄── بيستقبل الـ TCP connection
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│   Filter Chain      │  ◄── أول حاجة بتشوف الـ request
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ DispatcherServlet   │  ◄── قلب Spring MVC
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│    Interceptors     │  ◄── قبل الـ Controller
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│    Controller       │  ◄── بياخد الـ request ويفككه
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│      Service        │  ◄── الـ Business Logic
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│    Repository       │  ◄── بيتكلم مع الـ Database
└─────────────────────┘
         │
         ▼
    (نفس الطريق بالعكس للـ Response)
```

### كل خطوة بالتفصيل

**①ال Tomcat** — بيستقبل الـ TCP connection وبيحوّل الـ HTTP الخام لـ `HttpServletRequest` Object وبيعمل Thread من الـ thread pool.

**②ال Filter Chain** — بتشتغل قبل Spring نفسه يشوف الـ request.

```java
@Component
public class LoggingFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request,
                         ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {
        // ✅ هنا قبل الـ request يكمل
        System.out.println("Request جه: " + ((HttpServletRequest)request).getRequestURI());

        chain.doFilter(request, response); // ← كمّل للخطوة الجاية

        // ✅ هنا بعد الـ response يرجع
        System.out.println("Response اتبعت");
    }
}
```

**③ال DispatcherServlet** — بيسأل HandlerMapping مين المسؤول عن الـ Request ده.

**④ الInterceptors** — بتشتغل قبل الـ Controller مباشرةً وبعده.

```java
@Component
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) {
        System.out.println("هيدخل Controller دلوقتي");
        return true; // true = كمّل | false = وقّف هنا
    }

    @Override
    public void postHandle(...) { }  // بعد الـ Controller، قبل الـ Response

    @Override
    public void afterCompletion(...) { }  // بعد ما الـ Response اتبعت خالص
}
```

**الفرق بين Filter و Interceptor:**

| | Filter | Interceptor |
|---|---|---|
| بيشتغل قبل | Spring نفسه | الـ Controller بس |
| بيعرف مين الـ Controller؟ | لأ | آه |
| مكانه | Servlet level | Spring level |

**⑤ الController** — بياخد الـ Request ويفككه (PathVariable, RequestBody, إلخ).

**⑥ الService** — الـ Business Logic.

**⑦ الRepository** — بيتكلم مع الـ Database عن طريق Hibernate/JPA.

**⑧ الرجوع** — نفس الطريق بالعكس، Jackson بيحوّل الـ Object لـ JSON.

---

## 🔹 الـ DispatcherServlet بالتفصيل

### هو إيه؟

```
Java EE
  └── Servlet API
        └── HttpServlet  ← الـ Standard
              └── FrameworkServlet  ← Spring
                    └── DispatcherServlet  ← ده اللي بنتكلم عنه
```

هو **Front Controller** — كل Request لازم يعدي منه. مش بيعمل الشغل بنفسه، بس بيعرف مين المسؤول عن إيه.

### بيشتغل إزاي من جوّا؟

```
Request
  │
  ▼
① HandlerMapping      ← "مين المسؤول؟"
  │
  ▼
② HandlerAdapter      ← "إزاي أكلّمه؟"
  │
  ▼
③ Handler (Controller) ← "اشتغل يا باشا"
  │
  ▼
④ HandlerExceptionResolver ← "في error؟ اتعامل معاه"
  │
  ▼
⑤ ViewResolver        ← "ارسم الـ Response إزاي؟"
  │
  ▼
Response
```

### ① HandlerMapping

بيرجع مش بس الـ method — بيرجع:

```
HandlerMapping رجّع:
{
  handler:      UserController.getById(),
  interceptors: [AuthInterceptor, LoggingInterceptor]
}
```

### ② HandlerAdapter — الـ Magic الحقيقي

بيعرف يـ"يترجم" الـ Request ويحط كل argument في مكانه الصح:

```
URL: /api/users/42
     │
     ▼
HandlerAdapter شاف إن الـ Method محتاجة @PathVariable Long id
     │
     ▼
استخرج "42" من الـ URL وحوّلها من String لـ Long
     │
     ▼
استدعى الـ Method: getById(42L)
```

| Annotation | الـ Resolver بتاعه |
|---|---|
| `@PathVariable` | PathVariableMethodArgumentResolver |
| `@RequestParam` | RequestParamMethodArgumentResolver |
| `@RequestBody` | RequestResponseBodyMethodProcessor |
| `@RequestHeader` | RequestHeaderMethodArgumentResolver |

### ③ HandlerExceptionResolver

```
UserNotFoundException اتـ throw
         │
         ▼
DispatcherServlet بيمسك الـ Exception
         │
         ├── @ExceptionHandler في نفس الـ Controller؟ → يستخدمه
         └── @ControllerAdvice موجودة؟ → يستخدمها
```

### ④ ViewResolver

```
@RestController  →  Jackson بيحوّل Object لـ JSON
@Controller      →  Thymeleaf/JSP بيحوّل لـ HTML
```

### في DispatcherServlet واحد بس!

```
كل الـ Requests
    │    │    │
    ▼    ▼    ▼
  DispatcherServlet واحد
    │
    بيوزّع على Controllers كتير
```

---

## 🔹 الـ HandlerMapping بالتفصيل

### إمتى بيتبني؟

**وقت الـ Startup — مش وقت الـ Request:**

```
التطبيق بدأ
     │
     ▼
Spring بيمسح كل الـ Classes
     │
     ▼
لقى @RestController أو @Controller ؟
     │
     ▼
بيمسح كل الـ Methods جوّاه
     │
     ▼
لقى @GetMapping / @PostMapping إلخ ؟
     │
     ▼
بيسجّل الـ Mapping في جدول جوّاه
```

### الجدول اللي بيبنيه

```
┌─────────────────────────────┬──────────┬──────────────────────────────┐
│ URL Pattern                 │ Method   │ Handler                      │
├─────────────────────────────┼──────────┼──────────────────────────────┤
│ /api/users                  │ GET      │ UserController.getAll()      │
│ /api/users/{id}             │ GET      │ UserController.getById()     │
│ /api/users                  │ POST     │ UserController.create()      │
│ /api/users/{id}             │ PUT      │ UserController.update()      │
│ /api/users/{id}             │ DELETE   │ UserController.delete()      │
└─────────────────────────────┴──────────┴──────────────────────────────┘
```

### بيطابق على أكتر من الـ URL

```
Request
   ├── URL:         /api/users/42
   ├── HTTP Method: GET
   ├── Headers:     Content-Type, Accept
   └── Params:      ?page=0
```

```java
@GetMapping("/api/users")
public List<User> getAll() { }                         // GET + /api/users

@PostMapping("/api/users")
public User create() { }                               // POST + /api/users

@GetMapping(value = "/api/users", params = "active=true")
public List<User> getActive() { }                      // GET + param معين

@GetMapping(value = "/api/users", headers = "X-API-Version=2")
public List<User> getAllV2() { }                       // GET + header معين
```

### الـ URL Matching

```java
@GetMapping("/api/users")          // Exact Match — بس /api/users بالظبط
@GetMapping("/api/users/{id}")     // Path Variable — أي حاجة في مكان {id}
@GetMapping("/api/*/details")      // نجمة وحدة = segment واحد بس
@GetMapping("/api/**")             // نجمتين = أي حاجة بعد كده
```

### لو في Conflict — مين يكسب؟

```
① Exact Match يكسب دايماً
② Path Variable بعده
③ Wildcard آخر واحد
```

```java
@GetMapping("/api/users/profile")   // ① Exact
@GetMapping("/api/users/{id}")      // ② Path Variable
@GetMapping("/api/users/*")         // ③ Wildcard
```

```
GET /api/users/profile  →  يروح لـ ① (Exact Match)
GET /api/users/42       →  يروح لـ ② (Path Variable)
```

### لو ملقاش Handler

```
Request: GET /api/something-weird
              │
              ▼
HandlerMapping ملقاش أي Match
              │
              ▼
Response: 404 Not Found
```

---

## 🔹 الصورة النهائية الكاملة

```
GET /api/users/42
       │
       ▼
  Tomcat
  (استقبل الـ Connection، عمل Thread)
       │
       ▼
  Filter Chain
  (Spring Security بيتحقق من الـ Token)
       │
       ▼
  DispatcherServlet
       │
       ├──► HandlerMapping:
       │    "GET + /api/users/{id} → UserController.getById()"
       │
       ├──► Interceptors preHandle()
       │
       ├──► HandlerAdapter:
       │    استخرج id=42 من الـ URL، حوّله Long
       │    استدعى getById(42L)
       │
       ├──► Controller:
       │    getById(42) → بيكلم Service
       │         │
       │         └──► Service → Repository → Database
       │                   رجع User Object
       │
       ├──► Interceptors postHandle()
       │
       ├──► MessageConverter (Jackson):
       │    User Object → {"id":42,"name":"Ahmed"}
       │
       └──► Filter afterCompletion()
       │
       ▼
  Response: 200 OK + {"id":42,"name":"Ahmed"}
       │
       ▼
  Client ✅
```

---

> 📝 **ملاحظة:** الـ flow ده بيحصل لكل Request بيجي للتطبيق — فهمه كويس هو أساس فهم Spring Boot كله.
