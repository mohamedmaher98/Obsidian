# 🛣️ خريطة طريق شاملة لتعلم Spring Security من الصفر

## 📚 المرحلة الأولى: الأساسيات النظرية (1-2 أسابيع)

### 🎯 الأهداف:

- فهم مفاهيم الأمان الأساسية
- تعلم المصطلحات المهمة
- فهم أنواع التهديدات الشائعة

### 📖 المواضيع:

#### 1. مبادئ الأمان الأساسية

- **CIA Triad**: Confidentiality, Integrity, Availability
- **Authentication vs Authorization**
- **Principal, Credentials, Authorities**
- **Session Management**

#### 2. تهديدات الأمان الشائعة

- **OWASP Top 10**
- **SQL Injection**
- **Cross-Site Scripting (XSS)**
- **Cross-Site Request Forgery (CSRF)**
- **Session Hijacking**

#### 3. أنواع المصادقة

- **Username/Password**
- **Token-based (JWT)**
- **OAuth 2.0**
- **Multi-Factor Authentication (MFA)**

### 📝 المشاريع العملية:

- [ ] اقرأ عن OWASP Top 10 وتعرف على كل نوع تهديد
- [ ] اعمل بحث عن حوادث أمان شهيرة وتعلم منها

---

## 🏗️ المرحلة الثانية: إعداد البيئة وأول تطبيق (1 أسبوع)

### 🎯 الأهداف:

- إنشاء مشروع Spring Boot مع Spring Security
- فهم التكوين الأساسي
- تشغيل أول تطبيق محمي

### 📖 المواضيع:

#### 1. إعداد المشروع

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

#### 2. التكوين الافتراضي

- **Default Security Configuration**
- **Auto-generated Password**
- **Default Login Page**
- **Default User (user)**

#### 3. أول تطبيق

```java
@SpringBootApplication
public class SecurityDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SecurityDemoApplication.class, args);
    }
}

@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello Secured World!";
    }
}
```

### 📝 المشاريع العملية:

- [ ] أنشئ مشروع Spring Boot جديد
- [ ] أضف Spring Security dependency
- [ ] اعمل controller بسيط واختبر الحماية الافتراضية
- [ ] اختبر الوصول للتطبيق بالمستخدم الافتراضي

---

## ⚙️ المرحلة الثالثة: التكوين الأساسي (2-3 أسابيع)

### 🎯 الأهداف:

- تخصيص إعدادات الأمان
- إنشاء مستخدمين مخصصين
- فهم SecurityFilterChain

### 📖 المواضيع:

#### 1. Security Configuration Class

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults())
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }
}
```

#### 2. إنشاء مستخدمين في الذاكرة

```java
@Bean
public InMemoryUserDetailsManager userDetailsService() {
    UserDetails user = User.withDefaultPasswordEncoder()
        .username("user")
        .password("password")
        .roles("USER")
        .build();
    return new InMemoryUserDetailsManager(user);
}
```

#### 3. تشفير كلمات المرور

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

#### 4. URL Security Configuration

```java
.authorizeHttpRequests(authz -> authz
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
    .requestMatchers("/public/**").permitAll()
    .anyRequest().authenticated()
)
```

### 📝 المشاريع العملية:

- [ ] أنشئ SecurityConfig class مخصص
- [ ] اعمل مستخدمين في الذاكرة بأدوار مختلفة
- [ ] أنشئ controllers مختلفة لكل دور (admin, user, public)
- [ ] اختبر الوصول بمستخدمين مختلفين
- [ ] جرب أنواع مختلفة من PasswordEncoder

---

## 🗄️ المرحلة الرابعة: التكامل مع قاعدة البيانات (2-3 أسابيع)

### 🎯 الأهداف:

- حفظ المستخدمين في قاعدة البيانات
- تنفيذ UserDetailsService مخصص
- إدارة الأدوار والصلاحيات

### 📖 المواضيع:

#### 1. إعداد قاعدة البيانات

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true)
    private String username;
    
    private String password;
    
    private boolean enabled;
    
    @ManyToMany(fetch = FetchType.EAGER)
    private Set<Role> roles;
}

@Entity
@Table(name = "roles")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
}
```

#### 2. UserRepository

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

#### 3. Custom UserDetailsService

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        
        return org.springframework.security.core.userdetails.User.builder()
            .username(user.getUsername())
            .password(user.getPassword())
            .authorities(getAuthorities(user.getRoles()))
            .build();
    }
}
```

#### 4. تكوين AuthenticationProvider

```java
@Bean
public DaoAuthenticationProvider authenticationProvider() {
    DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
    authProvider.setUserDetailsService(userDetailsService);
    authProvider.setPasswordEncoder(passwordEncoder());
    return authProvider;
}
```

### 📝 المشاريع العملية:

- [ ] أنشئ User وRole entities
- [ ] اعمل repositories للمستخدمين والأدوار
- [ ] نفذ CustomUserDetailsService
- [ ] أنشئ صفحات تسجيل مستخدمين جدد
- [ ] اعمل نظام إدارة الأدوار
- [ ] اختبر تسجيل الدخول بمستخدمين من قاعدة البيانات

---

## 🔐 المرحلة الخامسة: المصادقة المتقدمة (3-4 أسابيع)

### 🎯 الأهداف:

- تنفيذ JWT Authentication
- فهم OAuth 2.0
- إنشاء REST API محمية

### 📖 المواضيع:

#### 1. JWT (JSON Web Tokens)

```java
@Component
public class JwtUtil {
    
    private String secret = "mySecret";
    private int jwtExpiration = 86400000; // 24 hours
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return createToken(claims, userDetails.getUsername());
    }
    
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = getUsernameFromToken(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
}
```

#### 2. JWT Authentication Filter

```java
@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException {
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized");
    }
}

@Component
public class JwtRequestFilter extends OncePerRequestFilter {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Autowired
    private JwtUtil jwtUtil;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, 
                                   FilterChain chain) throws ServletException, IOException {
        // JWT validation logic
    }
}
```

#### 3. Authentication Controller

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest loginRequest) {
        // Authenticate user and generate JWT
    }
    
    @PostMapping("/register")
    public ResponseEntity<?> register(@RequestBody RegisterRequest registerRequest) {
        // Register new user
    }
}
```

#### 4. OAuth 2.0 Integration

```java
@EnableOAuth2Client
public class OAuth2Config {
    
    @Bean
    public OAuth2RestTemplate oauth2RestTemplate() {
        return new OAuth2RestTemplate(googleResource());
    }
}
```

### 📝 المشاريع العملية:

- [ ] نفذ JWT utility class
- [ ] أنشئ JWT authentication filter
- [ ] اعمل login/register endpoints
- [ ] أنشئ REST API محمية بـ JWT
- [ ] اختبر JWT authentication مع Postman
- [ ] جرب تكامل OAuth 2.0 مع Google/Facebook

---

## 🛡️ المرحلة السادسة: Method Security (2 أسابيع)

### 🎯 الأهداف:

- تأمين Methods على مستوى Service layer
- فهم Annotations الخاصة بالأمان
- تنفيذ Fine-grained Authorization

### 📖 المواضيع:

#### 1. تفعيل Method Security

```java
@Configuration
@EnableMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig {
}
```

#### 2. Pre/Post Authorize Annotations

```java
@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    @PreAuthorize("hasRole('USER') or hasRole('ADMIN')")
    @PostAuthorize("returnObject.username == authentication.name or hasRole('ADMIN')")
    public User getUserById(Long id) {
        return userRepository.findById(id).orElse(null);
    }
    
    @PreAuthorize("@userSecurity.canModifyUser(authentication, #userId)")
    public User updateUser(Long userId, User user) {
        // Update user logic
    }
}
```

#### 3. Custom Security Expressions

```java
@Component("userSecurity")
public class UserSecurity {
    
    public boolean canModifyUser(Authentication authentication, Long userId) {
        String currentUsername = authentication.getName();
        // Custom logic to check if user can modify another user
        return true; // or false based on business logic
    }
}
```

#### 4. Secured و RolesAllowed Annotations

```java
@Secured({"ROLE_USER", "ROLE_ADMIN"})
public void secureMethod() {
    // Secure method implementation
}

@RolesAllowed({"ADMIN"})
public void adminOnlyMethod() {
    // Admin only method
}
```

### 📝 المشاريع العملية:

- [ ] فعل method security في تطبيقك
- [ ] استخدم @PreAuthorize و @PostAuthorize في service methods
- [ ] أنشئ custom security expressions
- [ ] اختبر method security مع مستخدمين مختلفين
- [ ] اعمل unit tests للmethod security

---

## 🌐 المرحلة السابعة: Web Security المتقدم (2-3 أسابيع)

### 🎯 الأهداف:

- فهم وحماية من CSRF
- تكوين CORS بشكل صحيح
- تأمين Session Management
- حماية من XSS

### 📖 المواضيع:

#### 1. CSRF Protection

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .ignoringRequestMatchers("/api/public/**")
            );
        return http.build();
    }
}
```

#### 2. CORS Configuration

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    configuration.setAllowedOriginPatterns(Arrays.asList("https://*.example.com"));
    configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE"));
    configuration.setAllowedHeaders(Arrays.asList("*"));
    configuration.setAllowCredentials(true);
    
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", configuration);
    return source;
}
```

#### 3. Session Management

```java
http
    .sessionManagement(session -> session
        .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // For REST APIs
        .maximumSessions(1)
        .maxSessionsPreventsLogin(false)
        .sessionRegistry(sessionRegistry())
    )
    .rememberMe(remember -> remember
        .tokenValiditySeconds(86400)
        .userDetailsService(userDetailsService)
    );
```

#### 4. Security Headers

```java
http
    .headers(headers -> headers
        .frameOptions(HeadersConfigurer.FrameOptionsConfig::deny)
        .contentTypeOptions(Customizer.withDefaults())
        .httpStrictTransportSecurity(hsts -> hsts
            .maxAgeInSeconds(31536000)
            .includeSubdomains(true)
        )
        .addHeaderWriter(new XFrameOptionsHeaderWriter(XFrameOptionsHeaderWriter.XFrameOptionsMode.SAMEORIGIN))
    );
```

### 📝 المشاريع العملية:

- [ ] نفذ CSRF protection مع Angular/React frontend
- [ ] كون CORS للسماح لdomains محددة فقط
- [ ] اعمل session timeout mechanism
- [ ] أضف security headers للresponses
- [ ] اختبر الحماية من XSS attacks

---

## 🔧 المرحلة الثامنة: Customization متقدم (3 أسابيع)

### 🎯 الأهداف:

- إنشاء Authentication Provider مخصص
- تخصيص Login/Logout Process
- إدارة الاستثناءات الأمنية
- تسجيل الأحداث الأمنية

### 📖 المواضيع:

#### 1. Custom Authentication Provider

```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();
        
        // Custom authentication logic
        User user = userService.findByUsername(username);
        if (user != null && passwordEncoder.matches(password, user.getPassword())) {
            List<GrantedAuthority> authorities = getUserAuthorities(user);
            return new UsernamePasswordAuthenticationToken(username, password, authorities);
        }
        
        throw new BadCredentialsException("Authentication failed");
    }
    
    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

#### 2. Custom Login Success/Failure Handlers

```java
@Component
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
                                       Authentication authentication) throws IOException {
        // Custom success logic
        String redirectUrl = determineTargetUrl(authentication);
        response.sendRedirect(redirectUrl);
    }
}

@Component
public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {
    
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
                                       AuthenticationException exception) throws IOException {
        // Custom failure logic
        response.sendRedirect("/login?error=true");
    }
}
```

#### 3. Security Event Listening

```java
@Component
public class AuthenticationEventListener {
    
    @EventListener
    public void handleAuthenticationSuccess(AuthenticationSuccessEvent event) {
        String username = event.getAuthentication().getName();
        // Log successful authentication
        logger.info("User {} successfully authenticated", username);
    }
    
    @EventListener
    public void handleAuthenticationFailure(AbstractAuthenticationFailureEvent event) {
        String username = event.getAuthentication().getName();
        // Log failed authentication
        logger.warn("Authentication failed for user {}", username);
    }
}
```

#### 4. Custom Access Denied Handler

```java
@Component
public class CustomAccessDeniedHandler implements AccessDeniedHandler {
    
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
                      AccessDeniedException accessDeniedException) throws IOException {
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        response.getWriter().write("Access Denied: " + accessDeniedException.getMessage());
    }
}
```

### 📝 المشاريع العملية:

- [ ] أنشئ custom authentication provider للتحقق من نوع معين من المستخدمين
- [ ] نفذ custom login success handler يوجه المستخدمين حسب أدوارهم
- [ ] أضف security event logging للمراقبة
- [ ] اعمل custom access denied pages
- [ ] اختبر كل الcustom handlers

---

## 🏢 المرحلة التاسعة: Enterprise Features (2-3 أسابيع)

### 🎯 الأهداف:

- تكامل مع LDAP/Active Directory
- تنفيذ Single Sign-On (SSO)
- إدارة Sessions في البيئة الموزعة
- تأمين Microservices

### 📖 المواضيع:

#### 1. LDAP Integration

```java
@Configuration
public class LdapConfig {
    
    @Bean
    public LdapAuthenticationProvider ldapAuthenticationProvider() {
        return new LdapAuthenticationProvider(
            new BindAuthenticator(contextSource()),
            new DefaultLdapAuthoritiesPopulator(contextSource(), "ou=groups")
        );
    }
    
    @Bean
    public LdapContextSource contextSource() {
        LdapContextSource source = new LdapContextSource();
        source.setUrl("ldap://localhost:389");
        source.setBase("dc=example,dc=com");
        return source;
    }
}
```

#### 2. SAML 2.0 Configuration

```java
@EnableSaml2Login
public class Saml2Config {
    
    @Bean
    public RelyingPartyRegistrationRepository relyingPartyRegistrations() {
        return new InMemoryRelyingPartyRegistrationRepository(
            RelyingPartyRegistration.withRegistrationId("saml2")
                .assertionConsumerServiceLocation("/login/saml2/sso/{registrationId}")
                .build()
        );
    }
}
```

#### 3. Distributed Session Management

```java
@Configuration
@EnableRedisHttpSession
public class SessionConfig {
    
    @Bean
    public LettuceConnectionFactory connectionFactory() {
        return new LettuceConnectionFactory(
            new RedisStandaloneConfiguration("localhost", 6379)
        );
    }
}
```

#### 4. Microservices Security

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig {
    
    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }
    
    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("secret");
        return converter;
    }
}
```

### 📝 المشاريع العملية:

- [ ] جرب تكامل LDAP مع OpenLDAP أو Active Directory
- [ ] نفذ SAML SSO مع identity provider
- [ ] اعمل distributed sessions باستخدام Redis
- [ ] أنشئ microservices محمية بـ OAuth 2.0
- [ ] اختبر SSO بين تطبيقات متعددة

---

## 🧪 المرحلة العاشرة: Testing والDebugging (2 أسابيع)

### 🎯 الأهداف:

- كتابة Unit Tests للSecurity
- تنفيذ Integration Tests
- اختبار الأمان (Penetration Testing)
- Debugging مشاكل الأمان

### 📖 المواضيع:

#### 1. Security Testing

```java
@SpringBootTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@WithMockUser(roles = "ADMIN")
class SecurityTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    @WithMockUser(roles = "USER")
    public void testUserAccess() throws Exception {
        mockMvc.perform(get("/api/user/profile"))
                .andExpect(status().isOk());
    }
    
    @Test
    @WithMockUser(roles = "USER")
    public void testAdminAccessDenied() throws Exception {
        mockMvc.perform(get("/api/admin/users"))
                .andExpect(status().isForbidden());
    }
}
```

#### 2. Authentication Testing

```java
@Test
public void testAuthentication() throws Exception {
    String username = "testuser";
    String password = "testpass";
    
    mockMvc.perform(post("/api/auth/login")
            .contentType(MediaType.APPLICATION_JSON)
            .content("{\"username\":\"" + username + "\",\"password\":\"" + password + "\"}"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.token").exists());
}
```

#### 3. Security Debugging

```java
// Enable security debugging
@EnableWebSecurity(debug = true)
public class SecurityConfig {
    // Configuration
}
```

#### 4. Performance Testing

```java
@Test
public void testAuthenticationPerformance() {
    long startTime = System.currentTimeMillis();
    
    // Perform multiple authentication requests
    for (int i = 0; i < 1000; i++) {
        // Authentication logic
    }
    
    long endTime = System.currentTimeMillis();
    assertTrue("Authentication performance is acceptable", (endTime - startTime) < 5000);
}
```

### 📝 المشاريع العملية:

- [ ] اكتب unit tests لكل security components
- [ ] اعمل integration tests للauthentication flow
- [ ] استخدم OWASP ZAP للpenetration testing
- [ ] اختبر أداء المصادقة تحت حمولة عالية
- [ ] اعمل security audit لتطبيقك

---

## 🚀 المرحلة النهائية: Best Practices ومشروع شامل (3-4 أسابيع)

### 🎯 الأهداف:

- تطبيق جميع المعارف في مشروع واحد
- تعلم أفضل الممارسات
- إعداد للإنتاج (Production Ready)

### 📖 المواضيع:

#### 1. Security Best Practices

- **Principle of Least Privilege**
- **Defense in Depth**
- **Security by Design**
- **Regular Security Updates**
- **Secure Configuration Management**

#### 2. Production Checklist

- [ ] HTTPS إجباري
- [ ] Security Headers مكونة
- [ ] Rate Limiting مطبق
- [ ] Logging وMonitoring مفعل
- [ ] Secrets Management آمن
- [ ] Regular Security Audits

#### 3. Monitoring والAlerting

```java
@Component
public class SecurityMonitor {
    
    @EventListener
    public void handleFailedLogin(AuthenticationFailureEvent event) {
        // Send alert if too many failed attempts
        securityAlertService.checkForBruteForceAttack(event);
    }
}
```

### 📝 المشروع النهائي:

**إنشاء نظام إدارة المحتوى (CMS) محمي بالكامل:**

- [ ] **المستخدمين**: تسجيل، تسجيل دخول، إدارة profiles
- [ ] **الأدوار**: Admin, Editor, Author, Reader
- [ ] **المحتوى**: إنشاء، تعديل، نشر، حذف مقالات
- [ ] **الصلاحيات**: تحكم دقيق في من يقدر يعمل إيه
- [ ] **API Security**: REST APIs محمية بـ JWT
- [ ] **File Upload**: رفع ملفات بأمان
- [ ] **Audit Trail**: تسجيل كل العمليات
- [ ] **Rate Limiting**: منع إساءة الاستخدام
- [ ] **Email Integration**: تفعيل الحسابات، إعادة تعيين كلمة المرور
- [ ] **Two-Factor Authentication**: أمان إضافي
- [ ] **Social Login**: تسجيل دخول بـ Google/Facebook
- [ ] **Admin Dashboard**: لوحة تحكم للإدارة
- [ ] **Responsive Design**: يعمل على الموبايل والكمبيوتر
- [ ] **Production Deployment**: نشر على AWS/Heroku

---

## 📅 جدول زمني مقترح (16-20 أسبوع)

|المرحلة|المدة|التركيز|
|---|---|---|
|1|1-2 أسابيع|الأساسيات النظرية|
|2|1 أسبوع|إعداد البيئة|
|3|2-3 أسابيع|التكوين الأساسي|
|4|2-3 أسابيع|قاعدة البيانات|
|5|3-4 أسابيع|المصادقة المتقدمة|
|6|2 أسابيع|Method Security|
|7|2-3 أسابيع|Web Security|
|8|3 أسابيع|Customization|
|9|2-3 أسابيع|Enterprise Features|
|10|2 أسابيع|Testing|
|11|3-4 أسابيع|المشروع النهائي|

---

## 📚 المصادر المقترحة

### الكتب:

1. **Spring Security in Action** - Laurentiu Spilca
2. **Pro Spring Security** - Carlo Scarioni, Massimo Nardone
3. **Spring Boot in Action** - Craig Walls

### الكورسات Online:

1. **Spring Security Zero to Master** - Udemy
2. **Spring Security Fundamentals** - Pluralsight
3. **OAuth 2.0 and Spring Security** - LinkedIn Learning

### المواقع والوثائق:

1. **Spring Security Reference** - docs.spring.io
2. **OWASP Web Security** - owasp.org
3. **Baeldung Spring Security** - baeldung.com

### أدوات مفيدة:

1. **Postman** - اختبار APIs
2. **OWASP ZAP** - اختبار الأمان
3. **JWT.io** - فحص JWT tokens
4. **Burp Suite** - اختبار الثغرات

---

## 🎯 نصائح للنجاح:

1. **اعمل hands-on projects** مع كل مرحلة
2. **اختبر كل حاجة** عملياً قبل الانتقال للمرحلة التالية
3. **اقرأ الأخطاء بعناية** - هي أحسن