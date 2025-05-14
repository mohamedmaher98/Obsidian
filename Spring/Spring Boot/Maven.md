# ==**إيه هو Maven؟**==
ل**Maven** هو **Build Tool** يعني أداة بتساعدك في:

- إدارة المكاتب (Dependencies)
    
- تجميع المشروع (Compile)
    
- بناء المشروع (Build)
    
- تشغيل التستات (Tests)
    
- إنتاج ملف `.jar` أو `.war`
    
- تنظيم المشروع بطريقة موحدة
- بيعمل generate documentation for your project 
- بيعمل generate  لل source code 
- بيعمل compiling  source code 
- بيعمل  packaging for source code in jar or war file 
- بينزل المكتبات الي فال pom
- 
----
# **==🧰 بيشتغل إزاي؟==**
كل مشروع Maven بيبقى فيه ملف اسمه: 
pom.xml
ده هو **العقل المدبر**، بيحتوي على كل الإعدادات والمكاتب اللي المشروع محتاجها.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
  
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0</version>
    <packaging>jar</packaging>

    <dependencies>
        <!-- Spring Boot Starter Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>

```
---
# **==هيكل مشروع Maven==**


project-name/
├── src/
│   ├── main/
│   │   └── java/
│   └── test/
│       └── java/
├── target/         ← هنا بيطلع ملف الـ .jar
└── pom.xml         ← ملف الإعدادات

|الأمر|الوظيفة|
|---|---|
|`mvn clean`|يمسح الـ build القديم|
|`mvn compile`|يجمّع الكود|
|`mvn test`|يشغل التستات|
|`mvn package`|يبني المشروع ويعمل `.jar` أو `.war`|
|`mvn install`|يضيف المكتبة للـ local repo|
|`mvn spring-boot:run`|يشغل مشروع Spring Boot (لو استخدمت plugin)|

---
# ==**Spring Boot Dependency Management**== 

السبرينج بوت انت بتحددله ال فيرجن بتاعت السبرينج بوت الي  انت هتتشغل عليه و هو لوحده هيكونترول اي مكتبة هينزلها وهينزل كل المكتبات بفيرجنات متوافقه مع بعض 
لان كل مكتبه لازم يبيق ليها فيرجن متوافقه مع التاني زي مكتبة ال Tom cat  لازم تبقي متوافقه مع مكتبة ال Sql  مثلا

---
# **==Starts==** 

ال**Starters** هي مكاتب جاهزة (Dependencies) Spring عملها عشان توفر عليك وقت.

كل Starter عبارة عن **باكيدج فيها كل اللي مشروعك محتاجه** علشان تبدأ تشتغل على جزء معين (مثلاً: Web, JPA, Security...).
يعني بدل ما تقعد تضيف dependency لكل مكتبة لوحدها (Servlet + Jackson + Tomcat ...)، Spring عملك Starter واحد يجمعهم كلهم.


|Starter|بيستخدم في إيه|
|---|---|
|`spring-boot-starter`|الباكيج الأساسي لكل مشاريع Spring Boot|
|`spring-boot-starter-web`|عشان تعمل APIs أو Web Apps (فيه Tomcat + JSON + Spring MVC)|
|`spring-boot-starter-data-jpa`|عشان تتعامل مع قواعد البيانات باستخدام JPA/Hibernate|
|`spring-boot-starter-security`|لو عايز تضيف تسجيل دخول وصلاحيات|
|`spring-boot-starter-thymeleaf`|لو بتستخدم صفحات HTML بـ Thymeleaf|
|`spring-boot-starter-test`|عشان تكتب وتختبر الكود بتاعك|
|`spring-boot-starter-validation`|عشان تضيف فاليشن (زي @NotNull, @Email)|
فال POM
```xml 
<dependencies>
    <!-- Web Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- JPA Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>


ال <groupId> وال <artifactid>
```
 
دول الي هما الاي دي بتوع اي حاجه سواء مكتبه او مشروع 
 مميزات الـ Starters
- بتوفر وقتك في إعداد المشروع
- مفيش تعارض بين المكاتب، Spring ظبطها
- بتخلي المشروع أنضف وأسهل في الصيانة
---
# ==**🧬 `spring-boot-starter-parent`**==


هو **Parent Project** جاهز معمول من Spring، وإنت بتورث منه في مشروعك علشان تستفيد من:
✅ 1. إدارة نسخ المكاتب (Dependency Versions)
بدل ما تكتب كل مرة النسخة بتاعة كل مكتبة، الـ starter-parent بيحدد لك نسخ موحدة ومستقرة.

✅ 2. إعدادات Build جاهزة (Plugins, Encoding, Java Version, إلخ)
مش لازم تعيد كتابة إعدادات الـ Maven من الأول.

✅ 3. ترتيب المشروع وسهولة الدمج
بيخلي مشروعك compatible مع باقي مشاريع Spring Boot.

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.5</version> <!-- أو أي إصدار بتستخدمه -->
    <relativePath/> <!-- مهم علشان يشتغل صح -->
</parent>


//full 

<project>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>

```
لو مش عايز تستخدم `starter-parent`، ممكن تستخدم `spring-boot-dependencies` كـ BOM (بس ده Advanced شوية وبيحتاج تكتب إعدادات أكتر).
