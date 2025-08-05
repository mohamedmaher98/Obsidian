
يعني إيه Token Authentication؟

هي طريقة لتسجيل الدخول وتفويض الوصول من غير ما السيرفر يحتفظ بأي Session.  
السيرفر بيصدر **Token** (زي مفتاح مؤقت) بعد تسجيل الدخول، والمستخدم بيبعته في كل طلب

خطوات Token Auth:
1. المستخدم يدخل **username + password**
2. السيرفر يتأكد من البيانات ✅
3. السيرفر بيولّد **Token (عادة JWT)**
4. السيرفر يبعت الـ Token للمستخدم (في Response)
5. المستخدم يخزنه (عادة في LocalStorage أو SessionStorage)
6. بعد كده، كل Request المستخدم يبعت فيه الـ Token ده في Header:
    
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6...
```
السيرفر يتحقق من صلاحية الـ Token ويديله الصلاحيات المطلوبة

 إيه هو JWT؟

**JWT = JSON Web Token**  
ده نوع من التوكن بيحتوي على بيانات اليوزر (زي id، role...) وموقع عليه بتوقيع (signature) علشان ما يتزوّرش.

مميزاته 

|الميزة|الشرح|
|---|---|
|**Stateless**|السيرفر مش بيحتفظ بحاجة (Scalable)|
|**سهل في الـ APIs**|مناسب للـ Mobile & SPA apps|
|**ينفع مع Microservices**|كل سيرفيس يقدر يتحقق بنفسه من الـ token|

عيوبه

|العيب|الشرح|
|---|---|
|**لازم تتحقق من التوكن في كل Request**|مفيش كاش ولا Session|
|**Token يتسرب؟ خطر كبير**|لازم تحفظه كويس|
|**لازم تأمن التخزين**|في الموبايل أو المتصفح|

==**تطبيق في Spring Boot (Token Auth بـ JWT)**==


### . تضيف dependencies:


```xml
`<dependency>     <groupId>io.jsonwebtoken</groupId>     <artifactId>jjwt</artifactId>     <version>0.9.1</version> </dependency>`
```
### 2. كلاس توليد التوكن:


```java
@Component public class JwtUtil {     private final String secret = "mySecretKey";      public String generateToken(String username) {         return Jwts.builder()             .setSubject(username)             .setIssuedAt(new Date())             .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60)) // 1 ساعة             .signWith(SignatureAlgorithm.HS256, secret)             .compact();     }      public String extractUsername(String token) {         return Jwts.parser().setSigningKey(secret).parseClaimsJws(token)             .getBody().getSubject();     } }`
```
### 3. فلتر JWT:

```java
`public class JwtFilter extends OncePerRequestFilter {      @Autowired     private JwtUtil jwtUtil;      @Override     protected void doFilterInternal(HttpServletRequest request,                                     HttpServletResponse response,                                     FilterChain filterChain)                                     throws ServletException, IOException {          String authHeader = request.getHeader("Authorization");          if (authHeader != null && authHeader.startsWith("Bearer ")) {             String token = authHeader.substring(7);             String username = jwtUtil.extractUsername(token);             // اعمل Set للـ Authentication هنا (مبسطة مؤقتًا)         }          filterChain.doFilter(request, response);     } }`
```
### 4. إعداد الـ Security:


```java
`@Configuration @EnableWebSecurity public class SecurityConfig {      @Bean     public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {         http.csrf().disable()             .authorizeHttpRequests(auth -> auth                 .requestMatchers("/auth/**").permitAll()                 .anyRequest().authenticated()             )             .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))             .addFilterBefore(new JwtFilter(), UsernamePasswordAuthenticationFilter.class);          return http.build();     } }`
```


| المقارنة  | Session Auth         | Token Auth (JWT)                         |
| --------- | -------------------- | ---------------------------------------- |
| التخزين   | الSession في السيرفر | الToken في الكلاينت (مثلاً localStorage) |
| مناسب لـ  | تطبيقات ويب عادية    | الAPIs / موبايل / SPA                    |
| Scalable؟ | لأ                   | ✅                                        |
| الأمان    | كويس مع HTTPS و CSRF | لازم تحمي التوكن بنفسك                   |
