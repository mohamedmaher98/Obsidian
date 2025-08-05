الJWT هو **توكن رقمي** (مثل بطاقة دخول) بيتم إنشاؤه بعد ما المستخدم يعمل **Login**، وبيحتوي على **معلومات المستخدم** (مثل اسمه، الصلاحيات، ومدة الصلاحية).
ال**JWT** أو **JSON Web Token** هو ملف صغير بيتبعت مع كل طلب HTTP علشان يثبت إن المستخدم مسجل دخوله.

بيكون فيه معلومات مشفّرة زي:  
✅ اسم المستخدم  
✅ تاريخ الانتهاء  
✅ بيانات تانية لو عايز
### **مقارنة بسيطة:**
|الطريقة التقليدية (Session)|JWT|
|---|---|
|تخزين بيانات المستخدم في **السيرفر** (مثل Redis أو Database).|البيانات كلها **مشفرة في التوكن نفسه**، والسيرفر مبيحتفظش بيها.|
|يحتاج **جلسة (Session)** علشان يتعرف على المستخدم.|**مستقل (Stateless)**، مفيش حاجة بتتخزن في السيرفر.|
|مناسب للتطبيقات الكلاسيكية (مثل المواقع بالـ Server-Side Rendering).|مناسب لـ **APIs** و **التطبيقات الحديثة** (زي React, Angular, Mobile Apps).|
## **ليه بنستخدم JWT؟ 💡**

1.عشان **Stateless**: مفيش حاجة بتتخزن في السيرفر، فبيقلل الحمل عليه.
2. **آمن**: البيانات مشفرة (مش بتكون واضحة).
3. **سريع**: مبيحتلش جلسات (Sessions) في الذاكرة.
4. **مناسب لـ Microservices**: لأن كل خدمة (Service) ممكن تفحص التوكن من غير ما تتواصل مع خدمة تانية.
## **هيكل JWT (إيه اللي جوا التوكن؟) 🧐**
الJWT بيتكون من 3 أجزاء، كل جزء بيتفصل بنقطة (`.`):

xxxxx.yyyyy.zzzzz

**Header (الجزء الأول - xxxxx)**:
بيحوي نوع التوكن (`JWT`) وخوارزمية التشفير (مثل `HS256`).
- مثال:
```java
{
  "alg": "HS256",
  "typ": "JWT"
}
```
**Payload (الجزء الثاني - yyyyy)**:

- هنا البيانات اللي هتخزنها (زي `userId`, `username`, `role`).
- فيه 3 أنواع من البيانات:
    - ال**Registered Claims**: معايير معروفة (زي `exp` للانتهاء، `iss` للمصدر).
    - ال**Public Claims**: بيانات عامة.
    - ال**Private Claims**: بيانات خاصة بتطبيقك.
- مثال:
 ```json
 {
  "sub": "1234567890",  // Subject (مثل User ID)
  "name": "Ahmed",
  "role": "ADMIN",
  "iat": 1516239022,    // تاريخ الإنشاء
  "exp": 1516242622     // تاريخ الانتهاء
}
```
**Signature (الجزء الثالث - zzzzz)**:
- ده الجزء اللي بيخلي التوكن آمن، لأنه بيتعمل عن طريق:
- HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret_key)
لو حد غير في الـ Header أو الـ Payload، هيتلغى التوكن.##

## **إزاي بيشتغل JWT مع Spring Boot**

**(1) إضافة Dependencies في `pom.xml`**

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

### **(2) إنشاء JWT Service**

```java
import io.jsonwebtoken.*;
import org.springframework.stereotype.Service;
import java.util.Date;

@Service
public class JwtService {
    private String SECRET_KEY = "mySuperSecretKey123!@#"; // المفتاح السري

    // إنشاء توكن
    public String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username) // المستخدم
                .setIssuedAt(new Date()) // تاريخ الإنشاء
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 10)) // انتهاء بعد 10 ساعات
                .signWith(SignatureAlgorithm.HS256, SECRET_KEY) // التشفير
                .compact();
    }

    // فحص التوكن
    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    // استخراج اسم المستخدم من التوكن
    public String extractUsername(String token) {
        return Jwts.parser()
                .setSigningKey(SECRET_KEY)
                .parseClaimsJws(token)
                .getBody()
                .getSubject();
    }
}
```
- الكلاس ده معمول علشان يبقى Service تقدر تستخدمه في أي حتة في Spring Boot.

وظيفته:
    
    1. يعمل Token
    2. يتحقق منه
    3. يطلع منه اسم المستخدم

private String SECRET_KEY = "mySuperSecretKey123!@#";


- ده المفتاح السري اللي بيستخدمه علشان يوقّع (يشفّر) الـ JWT.
- لازم يكون سري ومتخزن في مكان آمن (زي environment variable مش كده في الكود).

## ✍️ دالة إنشاء التوكن

```java
public String generateToken(String username) {
    return Jwts.builder()
        .setSubject(username)
        .setIssuedAt(new Date())
        .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 10))
        .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
        .compact();
}

```

|الكود|المعنى بالمصري|
|---|---|
|`.setSubject(username)`|هنا بتحط اسم المستخدم في التوكن.|
|`.setIssuedAt(new Date())`|بتحط تاريخ النهارده كـ تاريخ إصدار التوكن.|
|`.setExpiration(...)`|بتحط تاريخ الانتهاء (هنا بعد 10 ساعات من دلوقتي).|
|`.signWith(...)`|بتشفّر التوكن بالمفتاح السري باستخدام خوارزمية `HS256`.|
|`.compact()`|بتكوّن التوكن النهائي كـ String.|

## ✅ دالة فحص التوكن

```java
public boolean validateToken(String token) {
    try {
        Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token);
        return true;
    } catch (Exception e) {
        return false;
    }
}

```
- بتحاول تفك التوكن باستخدام المفتاح السري.
- لو اتفك بنجاح → التوكن سليم → بيرجع `true`.
- لو حصلت مشكلة (زي Expired / Signature غلط) → بيرجع `false`


- ## 🧍‍♂️ دالة استخراج اسم المستخدم من التوكن
```java
public String extractUsername(String token) {
    return Jwts.parser()
        .setSigningKey(SECRET_KEY)
        .parseClaimsJws(token)
        .getBody()
        .getSubject();
}
```
- بتفك التوكن وتجيب الـ **payload** (الـ data اللي جواه).
- وبعد كده بتجيب الـ **subject** اللي هو اسم المستخدم

 ## **مثال عملي سريع:**
 ### 1. أول ما المستخدم يسجّل دخول:
- تبعتله `token = generateToken("ahmed")`
### 2. كل مرة يبعَت request:
- بيبعت التوكن في Header زي كده:
- Authorization: Bearer eyJhbGciOi...
### 3. أنت في الـ Backend:
- تستخدم `validateToken(token)` علشان تتأكد إنه سليم.
- لو تمام، تستخدم `extractUsername(token)` علشان تعرف مين اللي باعت الطلب.