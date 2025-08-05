S
ال Spring Security هو إطار عمل قوي يُستخدم لتأمين تطبيقات **Java/Spring Boot**، حيث يوفر **مصادقة (Authentication)** و**تفويض (Authorization)** للمستخدمين. سنشرحه خطوة بخطوة مع أمثلة عملية.

### ==**what is Spring Security**==

ال**Spring Security** هو إطار عمل (framework) بيقدم أدوات لحماية تطبيقك:

- تسجيل الدخول (Authentication)
    
- الصلاحيات (Authorization)
    
- منع الوصول غير المصرح به
    
- التشفير (Encryption)
    
- حماية من هجمات شهيرة (مثل CSRF، Session Fixation)
  
 ## **==Dependency**==

 ```xml
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
-بمجرد إضافته، يصبح التطبيق مؤمنًا تلقائيًا ببيانات افتراضية
    اسم المستخدم: `user`
- كلمة السر: تُطبع في الـ Console (تبدأ بـ `Using generated security password`

## **==Authentication vs Authorization==**

| المصطلح            | المعنى                                                     |
| ------------------ | ---------------------------------------------------------- |
| **Authentication** | "إنت مين؟" → تسجيل الدخول (Username + Password)            |
| **Authorization**  | "مسموحلك تشوف إيه؟" → صلاحيات الوصول (Roles / Permissions) |
## ==SecurityFilterChain==

في Spring Security، فيه سلسلة من الـ **Filters** بتعدي على كل طلب HTTP، وبتقرر:
- هل الشخص ده مسجل دخول؟
- هل ليه صلاحية يشوف الصفحة دي؟
- هل محتوى الـ request آمن؟
ال`SecurityFilterChain` هو اللي بيتحكم في ترتيب وخصائص الفلاتر دي.

## ==HttpSecurity==

ال`HttpSecurity` هو الكائن اللي بتحدد بيه السياسات الأمنية:
- إيه الصفحات المسموح بيها؟
- شكل تسجيل الدخول عامل إزاي؟
- تعمل تسجيل خروج إزاي؟
زي ما استخدمته هنا:

## ==Roles==

في **Spring Security**، الـ **Roles** معناها "أدوار المستخدمين" (User Roles) وبتستخدم عشان تتحكم في **صلاحيات الوصول** (Authorization) لكل مستخدم حسب دوره.

**خطوات إضافة Role**
1. إنشاء Enum أو جدول في قاعدة البيانات يمثل الـ Roles.
2. ربط الـ Role بالمستخدم.
3. تعديل `UserDetailsService` ليرجع الـ Roles.
4. تأمين الـ Endpoints باستخدام الـ Roles.
5. (اختياري) استخدام JWT مع الـ Roles.
 🛠️ 1. إنشاء الـ Role
 ✅ لو هتستخدم Enum (بسيط وسريع):

```java
public enum Role {
ROLE_USER,
ROLE_ADMIN }
```
 أو 
```java
@Entity
@Table(name = "roles")
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;  // زي: ROLE_USER أو ROLE_ADMIN

    // Constructors
    public Role() {}

    public Role(String name) {
        this.name = name;
    }

    // Getters & Setters
}
```

**تربطه بكلاس يوزر**
```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue
    private Long id;

    private String username;
    private String password;

    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();

    // Constructors, Getters, Setters
}
```
كده كل يوزر ممكن يكون ليه أكتر من Role، زي:

- يوزر عنده: `ROLE_USER` و `ROLE_MANAGER`
- يوزر تاني عنده: `ROLE_ADMIN` بس

 **تعديل `UserDetailsService` علشان يرجع الصلاحيات:**

```java
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));

    List<GrantedAuthority> authorities = user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority(role.getName()))
            .collect(Collectors.toList());

    return new org.springframework.security.core.userdetails.User(
            user.getUsername(),
            user.getPassword(),
            authorities
    );
}
```


## ==**`UserDetails`**==

كلاس `UserDetails` في Spring Security هو **interface** (واجهة) بيستخدمه Spring علشان يتعامل مع بيانات المستخدم أثناء عملية **المصادقة (Authentication)**

 طيب ليه بنستخدم `UserDetails`؟

لما Spring يعمل تسجيل دخول (Login)، هو محتاج يعرف:
- اسم المستخدم (username)
- الباسورد (password)
- الأدوار (roles/authorities)
- 
- هل الحساب مفعل؟ منتهي؟... إلخ

فبدل ما يطلب منك تكتب كل ده بنفسك، بيقولك: "اعملي كلاس ينفذ `UserDetails` وأنا هتعامل معاه".

 شكل الواجهة (interface) `UserDetails`:

```java
public interface UserDetails {
    String getUsername();
    String getPassword();
    Collection<? extends GrantedAuthority> getAuthorities();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}```

كلاس خاص بيا ينفذ `UserDetails`
```java
public class CustomUserDetails implements UserDetails {

    private User user;  // كيان المستخدم بتاعك اللي جاي من قاعدة البيانات

    public CustomUserDetails(User user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority(role.getName()))
            .collect(Collectors.toList());
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

**بيتربط** في `UserDetailsService`:

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
        return new CustomUserDetails(user);
    }
}

```

- ال`UserDetails`: واجهة بيستخدمها Spring Security علشان يتعامل مع بيانات المستخدم.
- إنت بتعمل كلاس ينفذها (`CustomUserDetails`) علشان تحط فيه معلومات المستخدم بتاعك.
- الSpring بيرجع للكلاس ده أثناء تسجيل الدخول، وبيستخدم البيانات اللي فيه.

---


## **==`GrantedAuthority`==**
كلمة **`GrantedAuthority`** معناها ببساطة:

> "صلاحية أو إذن (Permission) معينة اتمنحت للمستخدم."

يعني هي الطريقة اللي Spring Security بيعرف بيها:

- المستخدم ده يقدر يدخل صفحة معينة؟
    
- يقدر ينفذ أكشن معين؟
    
- هل هو Admin؟ ولا User عادي؟ وهكذا...
- هو عبارة عن **interface** بسيط جدًا:

```java
public interface GrantedAuthority {
    String getAuthority();
}
```

```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
    return user.getRoles().stream()
        .map(role -> new SimpleGrantedAuthority(role.getName()))
        .collect(Collectors.toList());
}
```

احنا بنحوّل الـ Roles لـ GrantedAuthority علشان Spring Security يعرف يستخدمه

- ## **==Create Specified Login Page**==

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()  // أي شخص يمكنه الوصول
                .requestMatchers("/admin/**").hasRole("ADMIN")  // فقط للمدير
                .anyRequest().authenticated()  // الباقي يحتاج تسجيل دخول
            )
            .formLogin(form -> form
                .loginPage("/login")  // صفحة تسجيل الدخول المخصصة
                .permitAll()
            )
            .logout(logout -> logout
                .permitAll()
            );
        return http.build();
    }
}```
الكلاس ده هو مسؤول عن إعداد **أمان (Security)** التطبيق باستخدام **Spring Security**.
الكود ده بيستخدم طريقة حديثة اسمها **Lambda DSL** 

- ال`@Configuration`: معناها إن الـ class ده بيحتوي على Beans بيتم تحميلها في السياق (context) بتاع Spring.    
- ال`@EnableWebSecurity`: دي بتفعّل إعدادات الـ Spring Security.
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

```

- دي Bean مسؤولة عن إعداد سلسلة الفلاتر اللي بتتحكم في الأمن (Authorization, Authentication).
-ال `HttpSecurity` هو API بيسمح لك تحدد سياسات الأمن لكل طلب HTTP.


```java
http
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/public/**").permitAll()
        .requestMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
    )

```

|السطر|المعنى|
|---|---|
|`.requestMatchers("/public/**").permitAll()`|أي طلب على رابط يبدأ بـ `/public/` مسموح به لأي حد حتى لو مش مسجل دخوله.|
|`.requestMatchers("/admin/**").hasRole("ADMIN")`|فقط المستخدمين اللي ليهم الـ role "ADMIN" يقدروا يدخلوا روابط تبدأ بـ `/admin/`.|
|`.anyRequest().authenticated()`|أي طلب تاني غير اللي فوق لازم المستخدم يكون مسجل دخوله.|

**إعداد صفحة تسجيل الدخول (Login)**
```java
.formLogin(form -> form
    .loginPage("/login")  // صفحة مخصصة لتسجيل الدخول
    .permitAll()
)
```

- ال`formLogin`: بيفعل تسجيل الدخول من خلال **نموذج (form)**.
- ال`.loginPage("/login")`: بيقول إن صفحة تسجيل الدخول الخاصة بالمستخدم موجودة على الرابط `/login`.
- ال`.permitAll()`: بيسمح للجميع بالدخول على صفحة تسجيل الدخول (بدون ما يكونوا مسجلين).
---
تسجيل الخروج
```java
  .logout(logout -> logout
                .permitAll()
            );
        //بيفعل ميزة تسجيل الخروج، وبيخليها متاحة لأي حد من غير قيود.
        return http.build();
        
    }
    
```
return http.build(); 
بيرجع الكائن `SecurityFilterChain` بعد بناء كل الإعدادات.

لو عايز تضيف صلاحيات تانيه
```java 
.requestMatchers("/manager/**").hasRole("MANAGER")
```


## **==أنواع المصادقة في Spring Security==**

- **Form-based Authentication (نموذج تسجيل دخول)**
مثال :
```hava
http
  .authorizeHttpRequests(auth -> auth
      .anyRequest().authenticated()
  )
  .formLogin();  // يفعّل صفحة login

```
- المستخدم يفتح الموقع.
- يدخل اسم المستخدم والباسورد.
ال- Spring يتحقق ويرجعه للصفحة المطلوبة.

- **Basic Authentication**

مثال :
```java
http
  .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
  .httpBasic();  // يفعّل Basic Auth
```
مفيد في: أدوات Postman / REST APIs بسيطة.
🚫 **عيبه**: الباسورد بيعدي في كل طلب، حتى لو مشفر.

- **JWT Authentication (توكن مشفّر)**
(الأشهر في REST APIs)
ده النوع المنتشر حاليًا.
✅ الفكرة
- المستخدم يسجل الدخول (username + password)
- السيرفر يرجع JWT Token
- المستخدم يبعته في كل طلب في Header
🧠 مفيش سيشن، كله Stateless.
✅ مناسب جدًا لـ:
- الREST APIs
- الFrontend منفصل (React, Angular)

- **OAuth2 / Social Login (Google, Facebook, ...)**
مصادقة عبر Google, Facebook, GitHub...
الSpring بيدعمها بسهولة:
http
  .oauth2Login();
مناسب للمواقع اللي عايزة تسجيل دخول سريع بدون تسجيل يدوي.
مثال: Login باستخدام Google.

- **LDAP Authentication**
- **Token-based Authentication (غير JWT)**
- **Session-based Authentication**
- **Custom Authentication**

|النوع|الأفضل لـ|فيه سيشن؟|التعقيد|الأمان|
|---|---|---|---|---|
|Form Login|مواقع كلاسيكية|✅ نعم|بسيط|متوسط|
|Basic Auth|API تجريبية|❌ لا|بسيط جدًا|ضعيف|
|JWT|REST APIs|❌ لا|متوسط|قوي|
|OAuth2|سوشيال لوجن|❌ لا|متوسط-عالي|قوي جدًا|
|LDAP|أنظمة شركات|ممكن|متوسط|قوي|
|Token (غير JWT)|REST APIs|❌ لا|متوسط|حسب التنفيذ|
|Session|مواقع قديمة|✅ نعم|بسيط|متوسط|
|Custom|أي حاجة مخصصة|حسب التصميم|عالي|حسبك انت 😎|

---
## (Authorization) - التحكم في الصلاحيات
يتم تحديد الصلاحيات باستخدام:

- `hasRole("ADMIN")` → للمدير فقط.
    
- `hasAnyRole("USER", "ADMIN")` → للمستخدم أو المدير.
    
- `permitAll()` → متاح للجميع.
    
- `denyAll()` → ممنوع للجميع.

```java
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
    .requestMatchers("/public/**").permitAll()
    .anyRequest().authenticated()
);
```