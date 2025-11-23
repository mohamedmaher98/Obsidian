تمام، هشرحلك الكود ده خطوة بخطوة بطريقة بسيطة مع أمثلة عملية:

## الهدف العام من الكود:

الكود ده بيعمل نظام أمان (Security System) لتطبيق Spring Boot، بحيث المؤسسات (Organizations) تقدر تسجل دخول باستخدام Reference ID وكلمة مرور.

---

## 1. نموذج البيانات (Organization Model):

java

```java
@Entity
public class Organization extends BaseEntity {
    private String referenceId;      // زي كود المؤسسة
    private String organizationName; // اسم المؤسسة  
    private String password;         // كلمة المرور المشفرة
}
```

**الشرح:**

- ده زي جدول في قاعدة البيانات
- كل مؤسسة ليها كود خاص بيها (referenceId) زي "ORG001"
- واسم المؤسسة زي "شركة الأهرام"
- وكلمة مرور مشفرة

---

## 2. مستودع البيانات (Repository):

java

```java
public interface OrganizationRepository extends JpaRepository<Organization, Long> {
    Optional<Organization> findByReferenceId(String referenceId);
}
```

**الشرح:**

- ده بيتعامل مع قاعدة البيانات
- زي لما تقول له "دور على المؤسسة اللي كودها ORG001"
- يرجعلك المؤسسة لو موجودة، أو null لو مش موجودة

---

## 3. مقدم المصادقة المخصص (CustomAuthenticationProvider):

java

```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {
    // المكونات المطلوبة
    private OrganizationRepository repository;
    private PasswordEncoder passwordEncoder;
}
```

**الشرح المبسط:** ده زي البواب بتاع العمارة، لما حد يجي يدخل بيسأله:

- إيه اسمك (Reference ID)؟
- إيه كلمة المرور؟

### عملية التحقق (authenticate method):

java

```java
public Authentication authenticate(Authentication authentication) {
    String referenceId = authentication.getName();     // ياخد الكود
    String password = authentication.getCredentials().toString(); // ياخد كلمة المرور
    
    // يدور على المؤسسة في قاعدة البيانات
    Optional<Organization> organization = repository.findByReferenceId(referenceId);
    
    if (organization.isPresent()) { // لو المؤسسة موجودة
        if (passwordEncoder.matches(password, organization.get().getPassword())) {
            // كلمة المرور صح - يسمحله يدخل
            List<GrantedAuthority> authorityList = new ArrayList<>();
            authorityList.add(new SimpleGrantedAuthority("organization_user"));
            return new UsernamePasswordAuthenticationToken(referenceId, password, authorityList);
        } else {
            // كلمة مرور غلط
            throw new BadCredentialsException("Invalid password");
        }
    } else {
        // المؤسسة مش مسجلة
        throw new BadCredentialsException("Invalid organization you must be registered");
    }
}
```

**مثال عملي:**

1. مؤسسة تبعت: referenceId = "ORG001", password = "123456"
2. البواب يدور على "ORG001" في قاعدة البيانات
3. يلاقيها، يشوف كلمة المرور المحفوظة مشفرة
4. يقارن "123456" بكلمة المرور المشفرة
5. لو متطابقة، يديلها تصريح دخول بصلاحية "organization_user"

---

## 4. إعدادات الأمان (SecurityConfig):

java

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http, CustomAuthenticationProvider customAuthenticationProvider) {
        http
            .csrf(AbstractHttpConfigurer::disable) // يلغي CSRF للAPI
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/auth/**").permitAll()    // السماح للجميع
                .requestMatchers("/api/public/**").permitAll()  // السماح للجميع
                .requestMatchers("/actuator/**").permitAll()    // للمراقبة
                .requestMatchers("/swagger-ui.html", "/swagger-ui/**", "/v3/api-docs/**").permitAll() // للوثائق
                .anyRequest().authenticated() // باقي الطلبات تحتاج تسجيل دخول
            )
            .authenticationProvider(customAuthenticationProvider) // استخدام المصادقة المخصصة
            .httpBasic(Customizer.withDefaults()); // يستخدم HTTP Basic Auth
    }
}
```

**الشرح بالأمثلة:**

### الطرق المفتوحة (مش محتاجة تسجيل دخول):

- `/api/auth/login` - تسجيل الدخول
- `/api/public/info` - معلومات عامة
- `/swagger-ui.html` - صفحة الوثائق

### الطرق المحمية (محتاجة تسجيل دخول):

- `/api/users/list` - قائمة المستخدمين
- `/api/reports/sales` - تقارير المبيعات

---

## 5. تشفير كلمة المرور:

java

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

**الشرح:**

- بدل ما نحفظ كلمة المرور زي ما هي "123456"
- نحفظها مشفرة زي "2a$10 N.zmdr9k7uOCQQVVpnyt..."
- عشان لو حد سرق قاعدة البيانات ميقدرش يشوف كلمات المرور الحقيقية

---

## 6. إعدادات CORS:

java

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    configuration.setAllowedOrigins(Arrays.asList("http://localhost:4200"));
    // ...
}
```

**الشرح:**

- يسمح للموقع اللي على `http://localhost:4200` (عادة Angular) يستدعي الAPI
- زي لما تسمح لصديق معين يكلمك بس من رقم معين

---

## 7. الكونترولر (TestController):

java

```java
@RestController
@RequestMapping("/api/test")
public class TestController {
    @GetMapping("/start")
    public String test() {
        return "yeassssOOyess";
    }
}
```

**إيه اللي بيحصل لما تستدعي `/api/test/start`:**

### الحالة الحالية (الطريق مُعلق):

java

```java
// .requestMatchers("/api/test/**").permitAll() // معلق بـ //
```

1. المستخدم يبعت GET request لـ `/api/test/start`
2. SecurityFilter يشوف إن الطريق ده مش في القائمة المفتوحة
3. يطلب تسجيل دخول (HTTP 401 Unauthorized)
4. المستخدم لازم يبعت Authorization header زي:
    
    ```
    Authorization: Basic T1JHMDAxOjEyMzQ1Ng==
    ```
    
5. CustomAuthenticationProvider يفك التشفير ويتحقق
6. لو صح، يسمحله يوصل للكونترولر
7. الكونترولر يرجع "yeassssOOyess"

### لو شلت التعليق:

java

```java
.requestMatchers("/api/test/**").permitAll()
```

هيخلي أي حد يقدر يوصل للطريق ده من غير تسجيل دخول.

---

## ملخص سير العمل:

1. **التسجيل**: مؤسسة تسجل في النظام بـ referenceId و password
2. **التشفير**: كلمة المرور تتحفظ مشفرة في قاعدة البيانات
3. **تسجيل الدخول**: المؤسسة تبعت credentials في الـ Authorization header
4. **التحقق**: CustomAuthenticationProvider يتحقق من البيانات
5. **السماح**: لو البيانات صح، يسمحلها تستخدم الـ APIs المحمية
6. **الرد**: الكونترولر يشتغل ويرجع الاستجابة

**مثال كامل لاستدعاء الAPI:**

bash

```bash
curl -X GET "http://localhost:8080/api/test/start" \
     -H "Authorization: Basic T1JHMDAxOjEyMzQ1Ng=="
```

النظام ده بيضمن إن بس المؤسسات المسجلة والمخولة تقدر تستخدم APIs المحمية.