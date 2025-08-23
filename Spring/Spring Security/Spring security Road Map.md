# Spring Security Complete Roadmap

## من المبتدئ للمحترف - دليل شامل ومفصل

---

## 📋 فهرس المحتويات

### 🎯 **المرحلة الأولى: الأساسيات**

1. [مقدمة عن Spring Security](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#intro)
2. [إعداد المشروع الأول](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#setup)
3. [Authentication vs Authorization](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#auth-concepts)
4. [Basic Authentication](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#basic-auth)

### 🔐 **المرحلة الثانية: Authentication**

5. [Form-based Authentication](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#form-auth)
6. [Custom Login Pages](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#custom-login)
7. [UserDetailsService](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#user-details)
8. [Password Encoding](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#password-encoding)
9. [Remember Me](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#remember-me)

### 🛡️ **المرحلة الثالثة: Authorization**

10. [Method-level Security](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#method-security)
11. [URL-based Security](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#url-security)
12. [Role-based Access Control](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#rbac)
13. [Expression-based Access Control](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#expression-access)

### 🌐 **المرحلة الرابعة: Web Security**

14. [CSRF Protection](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#csrf)
15. [Session Management](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#session-mgmt)
16. [CORS Configuration](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#cors)
17. [Security Headers](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#security-headers)

### 🔑 **المرحلة الخامسة: JWT & OAuth**

18. [JWT Implementation](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#jwt)
19. [OAuth2 Basics](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#oauth2-basics)
20. [OAuth2 with Google/Facebook](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#oauth2-social)
21. [Resource Server](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#resource-server)

### 📱 **المرحلة السادسة: المتقدم**

22. [Method Security Advanced](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#method-advanced)
23. [Custom Security Filters](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#custom-filters)
24. [Multi-tenancy Security](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#multi-tenant)
25. [Reactive Security (WebFlux)](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#reactive)

### 🧪 **المرحلة السابعة: Testing & Production**

26. [Security Testing](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#testing)
27. [Performance & Monitoring](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#performance)
28. [Production Best Practices](https://claude.ai/chat/f3b3ac45-dfbb-4644-9d20-eaad0c9c58e6#production)

---

## 🎯 المرحلة الأولى: الأساسيات

### 1. مقدمة عن Spring Security {#intro}

#### ما هو Spring Security؟

الSpring Security هو framework قوي ومرن لحماية التطبيقات المبنية على Spring، يوفر:

- **Authentication**: التحقق من هوية المستخدم
- **Authorization**: التحكم في الصلاحيات
- **Protection**: الحماية من الهجمات الشائعة
- **Integration**: التكامل مع أنظمة خارجية

#### المفاهيم الأساسية:

```java
// Principal: هوية المستخدم
public interface Principal {
    String getName();
}

// Authentication: معلومات التصديق
public interface Authentication extends Principal {
    Collection<? extends GrantedAuthority> getAuthorities();
    Object getCredentials();
    Object getDetails();
    boolean isAuthenticated();
}

// GrantedAuthority: الصلاحيات
public interface GrantedAuthority {
    String getAuthority();
}
```

#### هيكل Spring Security:

```
Request → SecurityFilterChain → Authentication → Authorization → Controller
```

---

### 2. إعداد المشروع الأول {#setup}

#### Dependencies المطلوبة:

```xml
<dependencies>
    <!-- Spring Boot Starter Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- H2 Database for testing -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

#### أول تطبيق Security:

```java
@SpringBootApplication
public class SecurityDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SecurityDemoApplication.class, args);
    }
}

@RestController
public class HomeController {
    
    @GetMapping("/")
    public String home() {
        return "Welcome to Spring Security!";
    }
    
    @GetMapping("/user")
    public String user(Principal principal) {
        return "Hello " + principal.getName();
    }
}
```

#### التشغيل الأول:

عند تشغيل التطبيق، Spring Security يقوم بـ:

1. إنشاء مستخدم افتراضي: `user`
2. إنشاء كلمة مرور عشوائية (تظهر في الـ console)
3. تفعيل Basic Authentication تلقائياً

---

### 3. Authentication vs Authorization {#auth-concepts}

#### Authentication (التصديق):

```java
// من هو المستخدم؟
public class AuthenticationExample {
    
    // مثال على معلومات التصديق
    public class UserCredentials {
        private String username;
        private String password;
        private boolean enabled;
        private boolean accountNonExpired;
        private boolean credentialsNonExpired;
        private boolean accountNonLocked;
    }
}
```

#### Authorization (التفويض):

```java
// ما الذي يُسمح للمستخدم بفعله؟
public class AuthorizationExample {
    
    // مثال على الصلاحيات
    public enum Role {
        ADMIN("ROLE_ADMIN"),
        USER("ROLE_USER"),
        MODERATOR("ROLE_MODERATOR");
        
        private final String authority;
        
        Role(String authority) {
            this.authority = authority;
        }
        
        public String getAuthority() {
            return authority;
        }
    }
}
```

#### الفرق بينهما:

|Authentication|Authorization|
|---|---|
|"من أنت؟"|"ماذا يُسمح لك؟"|
|Username/Password|Roles/Permissions|
|Login Process|Access Control|
|يحدث أولاً|يحدث ثانياً|

---

### 4. Basic Authentication {#basic-auth}

#### ما هو Basic Authentication؟

```java
@Configuration
@EnableWebSecurity
public class BasicSecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());
        
        return http.build();
    }
    
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.withDefaultPasswordEncoder()
            .username("user")
            .password("password")
            .roles("USER")
            .build();
            
        UserDetails admin = User.withDefaultPasswordEncoder()
            .username("admin")
            .password("admin")
            .roles("ADMIN", "USER")
            .build();
            
        return new InMemoryUserDetailsManager(user, admin);
    }
}
```

#### كيف يعمل Basic Authentication:

1. العميل يرسل طلب بدون authorization header
2. الخادم يرد بـ 401 Unauthorized
3. العميل يرسل الطلب مع header: `Authorization: Basic base64(username:password)`
4. الخادم يفك التشفير ويتحقق من البيانات

#### مثال على الاستخدام:

```bash
# طلب بدون authentication
curl http://localhost:8080/user

# طلب مع Basic Auth
curl -u user:password http://localhost:8080/user

# أو باستخدام header مباشرة
curl -H "Authorization: Basic dXNlcjpwYXNzd29yZA==" http://localhost:8080/user
```

---

## 🔐 المرحلة الثانية: Authentication

### 5. Form-based Authentication {#form-auth}

#### إعداد Form Login:

```java
@Configuration
@EnableWebSecurity
public class FormSecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/login", "/css/**", "/js/**").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .loginProcessingUrl("/perform-login")
                .defaultSuccessUrl("/home", true)
                .failureUrl("/login?error=true")
                .usernameParameter("email")
                .passwordParameter("pass")
            )
            .logout(logout -> logout
                .logoutUrl("/perform-logout")
                .logoutSuccessUrl("/login?logout=true")
                .deleteCookies("JSESSIONID")
            );
            
        return http.build();
    }
}
```

#### Controller لصفحة Login:

```java
@Controller
public class LoginController {
    
    @GetMapping("/login")
    public String login(Model model, 
                       @RequestParam(value = "error", required = false) String error,
                       @RequestParam(value = "logout", required = false) String logout) {
        
        if (error != null) {
            model.addAttribute("errorMsg", "Invalid username or password!");
        }
        
        if (logout != null) {
            model.addAttribute("msg", "You've been logged out successfully.");
        }
        
        return "login";
    }
    
    @GetMapping("/home")
    public String home(Authentication authentication, Model model) {
        model.addAttribute("username", authentication.getName());
        model.addAttribute("authorities", authentication.getAuthorities());
        return "home";
    }
}
```

---

### 6. Custom Login Pages {#custom-login}

#### HTML Template (Thymeleaf):

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Login</title>
    <link rel="stylesheet" href="/css/login.css">
</head>
<body>
    <div class="login-container">
        <h2>Login</h2>
        
        <!-- Error Message -->
        <div th:if="${errorMsg}" class="alert alert-danger" th:text="${errorMsg}"></div>
        
        <!-- Success Message -->
        <div th:if="${msg}" class="alert alert-success" th:text="${msg}"></div>
        
        <form th:action="@{/perform-login}" method="post">
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" required>
            </div>
            
            <div class="form-group">
                <label for="pass">Password:</label>
                <input type="password" id="pass" name="pass" required>
            </div>
            
            <div class="form-group">
                <label>
                    <input type="checkbox" name="remember-me"> Remember Me
                </label>
            </div>
            
            <button type="submit">Login</button>
        </form>
        
        <div class="register-link">
            <a th:href="@{/register}">Don't have an account? Register here</a>
        </div>
    </div>
</body>
</html>
```

#### CSS للتصميم:

```css
.login-container {
    max-width: 400px;
    margin: 50px auto;
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.form-group {
    margin-bottom: 15px;
}

.form-group label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
}

.form-group input[type="email"],
.form-group input[type="password"] {
    width: 100%;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

button {
    width: 100%;
    padding: 10px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

.alert {
    padding: 10px;
    margin-bottom: 15px;
    border-radius: 4px;
}

.alert-danger {
    background-color: #f8d7da;
    color: #721c24;
    border: 1px solid #f5c6cb;
}

.alert-success {
    background-color: #d4edda;
    color: #155724;
    border: 1px solid #c3e6cb;
}
```

---

### 7. UserDetailsService {#user-details}

#### إنشاء UserDetailsService مخصص:

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true)
    private String email;
    
    private String password;
    private String firstName;
    private String lastName;
    private boolean enabled = true;
    private boolean accountNonExpired = true;
    private boolean accountNonLocked = true;
    private boolean credentialsNonExpired = true;
    
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    // constructors, getters, setters
}

@Entity
@Table(name = "roles")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Enumerated(EnumType.STRING)
    private RoleName name;
    
    // constructors, getters, setters
}

public enum RoleName {
    ROLE_USER,
    ROLE_ADMIN,
    ROLE_MODERATOR
}
```

#### UserDetails Implementation:

```java
public class UserPrincipal implements UserDetails {
    private Long id;
    private String email;
    private String password;
    private String firstName;
    private String lastName;
    private Collection<? extends GrantedAuthority> authorities;
    private boolean enabled;
    private boolean accountNonExpired;
    private boolean accountNonLocked;
    private boolean credentialsNonExpired;
    
    public UserPrincipal(Long id, String email, String password, 
                        String firstName, String lastName,
                        Collection<? extends GrantedAuthority> authorities,
                        boolean enabled, boolean accountNonExpired,
                        boolean accountNonLocked, boolean credentialsNonExpired) {
        this.id = id;
        this.email = email;
        this.password = password;
        this.firstName = firstName;
        this.lastName = lastName;
        this.authorities = authorities;
        this.enabled = enabled;
        this.accountNonExpired = accountNonExpired;
        this.accountNonLocked = accountNonLocked;
        this.credentialsNonExpired = credentialsNonExpired;
    }
    
    public static UserPrincipal create(User user) {
        List<GrantedAuthority> authorities = user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority(role.getName().name()))
                .collect(Collectors.toList());
                
        return new UserPrincipal(
            user.getId(),
            user.getEmail(),
            user.getPassword(),
            user.getFirstName(),
            user.getLastName(),
            authorities,
            user.isEnabled(),
            user.isAccountNonExpired(),
            user.isAccountNonLocked(),
            user.isCredentialsNonExpired()
        );
    }
    
    @Override
    public String getUsername() {
        return email;
    }
    
    @Override
    public String getPassword() {
        return password;
    }
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }
    
    @Override
    public boolean isEnabled() {
        return enabled;
    }
    
    @Override
    public boolean isAccountNonExpired() {
        return accountNonExpired;
    }
    
    @Override
    public boolean isAccountNonLocked() {
        return accountNonLocked;
    }
    
    @Override
    public boolean isCredentialsNonExpired() {
        return credentialsNonExpired;
    }
    
    // Additional getters
    public Long getId() { return id; }
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
}
```

#### Custom UserDetailsService:

```java
@Service
@Transactional
public class CustomUserDetailsService implements UserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        User user = userRepository.findByEmail(email)
                .orElseThrow(() -> new UsernameNotFoundException("User not found with email: " + email));
                
        return UserPrincipal.create(user);
    }
    
    // Method used by JWT authentication
    public UserDetails loadUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new UsernameNotFoundException("User not found with id: " + id));
                
        return UserPrincipal.create(user);
    }
}
```

---

### 8. Password Encoding {#password-encoding}

#### إعداد Password Encoder:

```java
@Configuration
public class PasswordConfig {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12); // strength 12
    }
    
    // للبيئات المختلفة
    @Bean
    @Profile("dev")
    public PasswordEncoder devPasswordEncoder() {
        return NoOpPasswordEncoder.getInstance(); // للتطوير فقط
    }
    
    @Bean
    @Profile("prod")
    public PasswordEncoder prodPasswordEncoder() {
        return new BCryptPasswordEncoder(15); // أقوى للإنتاج
    }
}
```

#### استخدام Password Encoder:

```java
@Service
public class UserService {
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Autowired
    private UserRepository userRepository;
    
    public User createUser(UserRegistrationDto registrationDto) {
        // التحقق من وجود المستخدم
        if (userRepository.existsByEmail(registrationDto.getEmail())) {
            throw new UserAlreadyExistsException("Email already in use!");
        }
        
        // إنشاء مستخدم جديد
        User user = new User();
        user.setEmail(registrationDto.getEmail());
        user.setFirstName(registrationDto.getFirstName());
        user.setLastName(registrationDto.getLastName());
        
        // تشفير كلمة المرور
        user.setPassword(passwordEncoder.encode(registrationDto.getPassword()));
        
        // إضافة دور افتراضي
        Role userRole = roleRepository.findByName(RoleName.ROLE_USER)
                .orElseThrow(() -> new RuntimeException("User Role not set."));
        user.getRoles().add(userRole);
        
        return userRepository.save(user);
    }
    
    public boolean checkPassword(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }
    
    public void changePassword(Long userId, String oldPassword, String newPassword) {
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new UserNotFoundException("User not found"));
                
        // التحقق من كلمة المرور القديمة
        if (!passwordEncoder.matches(oldPassword, user.getPassword())) {
            throw new InvalidPasswordException("Current password is incorrect");
        }
        
        // تحديث كلمة المرور
        user.setPassword(passwordEncoder.encode(newPassword));
        userRepository.save(user);
    }
}
```

#### أنواع Password Encoders المختلفة:

```java
@Configuration
public class PasswordEncoderExamples {
    
    // BCrypt - الأكثر استخداماً
    @Bean
    public PasswordEncoder bcryptEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    // Argon2 - الأحدث والأقوى
    @Bean
    public PasswordEncoder argon2Encoder() {
        return new Argon2PasswordEncoder();
    }
    
    // SCrypt
    @Bean
    public PasswordEncoder scryptEncoder() {
        return new SCryptPasswordEncoder();
    }
    
    // PBKDF2
    @Bean
    public PasswordEncoder pbkdf2Encoder() {
        return new Pbkdf2PasswordEncoder();
    }
    
    // DelegatingPasswordEncoder - يدعم عدة أنواع
    @Bean
    public PasswordEncoder delegatingEncoder() {
        String idForEncode = "bcrypt";
        Map<String, PasswordEncoder> encoders = new HashMap<>();
        encoders.put(idForEncode, new BCryptPasswordEncoder());
        encoders.put("noop", NoOpPasswordEncoder.getInstance());
        encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
        encoders.put("scrypt", new SCryptPasswordEncoder());
        encoders.put("argon2", new Argon2PasswordEncoder());
        
        return new DelegatingPasswordEncoder(idForEncode, encoders);
    }
}
```

---

### 9. Remember Me {#remember-me}

#### إعداد Remember Me:

```java
@Configuration
@EnableWebSecurity
public class RememberMeConfig {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Autowired
    private DataSource dataSource;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            .rememberMe(remember -> remember
                .tokenRepository(persistentTokenRepository())
                .userDetailsService(userDetailsService)
                .tokenValiditySeconds(24 * 60 * 60) // 24 hours
                .key("mySecretKey")
                .rememberMeParameter("remember-me")
                .rememberMeCookieName("my-remember-me")
            );
            
        return http.build();
    }
    
    @Bean
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();
        tokenRepository.setDataSource(dataSource);
        return tokenRepository;
    }
}
```

#### Database Schema للـ Remember Me:

```sql
CREATE TABLE persistent_logins (
    username VARCHAR(64) NOT NULL,
    series VARCHAR(64) PRIMARY KEY,
    token VARCHAR(64) NOT NULL,
    last_used TIMESTAMP NOT NULL
);
```

#### Entity للـ Persistent Tokens:

```java
@Entity
@Table(name = "persistent_logins")
public class PersistentLogin {
    
    private String username;
    
    @Id
    private String series;
    
    private String token;
    
    @Column(name = "last_used")
    private Date lastUsed;
    
    // constructors, getters, setters
}
```

#### Custom Remember Me Service:

```java
@Service
public class CustomRememberMeService extends PersistentTokenBasedRememberMeServices {
    
    public CustomRememberMeService(String key, UserDetailsService userDetailsService, 
                                  PersistentTokenRepository tokenRepository) {
        super(key, userDetailsService, tokenRepository);
    }
    
    @Override
    protected void onLoginSuccess(HttpServletRequest request, HttpServletResponse response, 
                                 Authentication successfulAuthentication) {
        super.onLoginSuccess(request, response, successfulAuthentication);
        
        // إضافة منطق إضافي عند نجاح تسجيل الدخول
        String username = successfulAuthentication.getName();
        logger.info("Remember me login successful for user: " + username);
        
        // تسجيل معلومات إضافية
        auditService.logRememberMeLogin(username, request.getRemoteAddr());
    }
    
    @Override
    protected UserDetails processAutoLoginCookie(String[] cookieTokens, 
                                               HttpServletRequest request, 
                                               HttpServletResponse response) {
        try {
            UserDetails user = super.processAutoLoginCookie(cookieTokens, request, response);
            
            // تحديث آخر نشاط للمستخدم
            userService.updateLastActivity(user.getUsername());
            
            return user;
        } catch (RememberMeAuthenticationException ex) {
            logger.warn("Remember me authentication failed: " + ex.getMessage());
            throw ex;
        }
    }
}
```

---

## 🛡️ المرحلة الثالثة: Authorization

### 10. Method-level Security {#method-security}

#### تفعيل Method Security:

```java
@Configuration
@EnableMethodSecurity(
    prePostEnabled = true,      // @PreAuthorize, @PostAuthorize
    securedEnabled = true,      // @Secured
    jsr250Enabled = true       // @RolesAllowed
)
public class MethodSecurityConfig {
    
    @Bean
    public MethodSecurityExpressionHandler expressionHandler() {
        DefaultMethodSecurityExpressionHandler handler = 
            new DefaultMethodSecurityExpressionHandler();
        handler.setPermissionEvaluator(new CustomPermissionEvaluator());
        return handler;
    }
}
```

#### أمثلة على Method Security:

```java
@Service
public class BookService {
    
    @Autowired
    private BookRepository bookRepository;
    
    // باستخدام @PreAuthorize
    @PreAuthorize("hasRole('ADMIN')")
    public List<Book> getAllBooks() {
        return bookRepository.findAll();
    }
    
    // التحقق من الملكية
    @PreAuthorize("hasRole('USER') and #book.author.email == authentication.name")
    public Book updateBook(@Param("book") Book book) {
        return bookRepository.save(book);
    }
    
    // التحقق من الصلاحية المعقدة
    @PreAuthorize("hasRole('ADMIN') or (hasRole('MODERATOR') and #book.category != 'RESTRICTED')")
    public void deleteBook(Book book) {
        bookRepository.delete(book);
    }
    
    // باستخدام @PostAuthorize
    @PostAuthorize("returnObject.author.email == authentication.name or hasRole('ADMIN')")
    public Book getBookById(Long id) {
        return bookRepository.findById(id).orElse(null);
    }
    
    // فلترة النتائج
    @PostFilter("filterObject.isPublic or filterObject.author.email == authentication.name")
    public List<Book> getUserBooks() {
        return bookRepository.findAll();
    }
    
    // باستخدام @Secured
    @Secured({"ROLE_USER", "ROLE_ADMIN"})
    public List<Book> getPublicBooks() {
        return bookRepository.findByIsPublicTrue();
    }
    
    // باستخدام @RolesAllowed
    @RolesAllowed("ADMIN")
    public void approveBook(Long bookId) {
        Book book = bookRepository.findById(bookId).orElseThrow();
        book.setApproved(true);
        bookRepository.save(book);
    }
    
    // SpEL expressions معقدة
    @PreAuthorize("@bookSecurityService.canEditBook(#bookId, authentication.name)")
    public Book editBook(Long bookId, BookDto bookDto) {
        Book book = bookRepository.findById(bookId).orElseThrow();
        // update logic
        return bookRepository.save(book);
    }
}
```

#### Custom Permission Evaluator:

```java
@Component
public class CustomPermissionEvaluator implements PermissionEvaluator {
    
    @Autowired
    private BookRepository bookRepository;
    
    @Override
    public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
        if (authentication == null || targetDomainObject == null || !(permission instanceof String)) {
            return false;
        }
        
        String targetType = targetDomainObject.getClass().getSimpleName().toUpperCase();
        return hasPrivilege(authentication, targetType, permission.toString());
    }
    
    @Override
    public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission) {
        if (authentication == null || targetType == null || !(permission instanceof String)) {
            return false;
        }
        
        return hasPrivilege(authentication, targetType.toUpperCase(), permission.toString());
    }
    
    private boolean hasPrivilege(Authentication authentication, String targetType, String permission) {
        UserPrincipal principal = (UserPrincipal) authentication.getPrincipal();
        
        // تحقق من الصلاحيات العامة
        if (hasRole(authentication, "ADMIN")) {
            return true;
        }
        
        // تحقق من صلاحيات محددة حسب نوع الكائن
        switch (
```