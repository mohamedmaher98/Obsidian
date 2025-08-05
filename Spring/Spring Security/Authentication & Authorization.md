الفرق بين **Authentication** و **Authorization**

| المصطلح            | المعنى                                    | مثال بسيط                        |
| ------------------ | ----------------------------------------- | -------------------------------- |
| **Authentication** | التحقق من الشخص ده هو مين؟ (تسجيل الدخول) | أحمد دخل اليوزرنيم والباسورد؟ ✅  |
| **Authorization**  | الشخص ده مسموح له يعمل إيه؟ (صلاحيات)     | أحمد يقدر يدخل صفحة الـ Admin؟ ❌ |

يعني إيه Spring Security

| المكون                      | وظيفته                                |
| --------------------------- | ------------------------------------- |
| `AuthenticationManager`     | بيدير عملية التحقق من الهوية          |
| `UserDetailsService`        | بيوفر بيانات المستخدم من DB أو من كود |
| `PasswordEncoder`           | بيشفر الباسوردات                      |
| `SecurityFilterChain`       | الفلاتر اللي بتحمي الـ requests       |
| `GrantedAuthority` / `Role` | بتحدد الصلاحيات لكل يوزر              |


اعداداته فالتطبيق
in pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```


إعداد مستخدم وباسورد في `application.properties` (أبسط طريقة):

in Properties

```properties
spring.security.user.name=admin
spring.security.user.password=1234
```

3. حماية صفحة معينة:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults());

        return http.build();
    }
}
```

4-الUserDetailsService (لو هتجيب اليوزر من قاعدة البيانات):


ملخص 

|المصطلح|المعنى|
|---|---|
|Authentication|التحقق من الهوية (اليوزر باسورد)|
|Authorization|السماح أو المنع من دخول صفحات معينة|
|Spring Security|إطار لحماية تطبيقك من غير ما تكتب كل حاجة من الصفر|
|UserDetailsService|بيجبلك بيانات اليوزر|
|PasswordEncoder|لتشفير وفك تشفير الباسورد|
|SecurityFilterChain|بيحدد إيه يتأمن وإزاي|