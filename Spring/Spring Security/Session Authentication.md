يعني إيه Session Authentication؟
اول ما المستخدم يسجّل دخوله بنجاح، السيرفر بيحتفظ بمعلومة إنه "مسجل دخول" في Session، وبيستخدمها بعد كده عشان يفتكر المستخدم ده.

خطوات Session Auth:

1. المستخدم يدخل **username + password**
    
2. السيرفر يتأكد إن البيانات صح ✅
    
3. السيرفر بيعمل **Session جديدة**
    
4. بيبعت للمستخدم **Session ID** في Cookie
    
5. بعد كده، كل Request بيبعت الـ Session ID ده تلقائيًا
    
6. السيرفر يعرف المستخدم مين بناءً على الـ Session

الـ **Cookie** هي اللي بتخزن Session ID في المتصفح، زي:
```ini
JSESSIONID=823hdh7e3hd8h3d9d
```


|الميزة|ليه مفيدة؟|
|---|---|
|سهلة التنفيذ|Spring بيجهزلك كل حاجة|
|مش محتاج Token|كله داخلي|
|آمنة نسبيًا|طالما HTTPS شغال|

العيوب 

|العيب|ليه مشكلة؟|
|---|---|
|**مش scalable قوي**|لأن كل Session محفوظة على السيرفر|
|**مش مناسب للموبايل**|لازم Cookie، والموبايل مش دايمًا بيساعد في كده|
|**CSRF معرض له**|لأن الـ Cookies بتتبعت تلقائيًا|

## ==**Spring Security & Session Auth**==

الSpring Security بـ **الـ default** بيشتغل بنظام **Session Authentication**.

يعني:
- أول ما تسجل دخول → بيتخزن Session على السيرفر
- وبيرجعلك Cookie فيها الـ `JSESSIONID`
- وكل ما تبعت request → Spring يفتكرك عن طريق الـ session دي
إعداد بسيط:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults()) // Login form = Session
            .logout(Customizer.withDefaults());   // Logout بيخلص الـ session

        return http.build();
    }
}
```


لو ما عملتش أي إعدادات خاصة، Spring هيتعامل أوتوماتيك بـ Session-based Auth.