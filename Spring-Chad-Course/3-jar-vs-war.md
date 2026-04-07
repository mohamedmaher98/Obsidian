# 📦 الفرق بين JAR و WAR بالتفصيل

> شرح مبسط من الصفر — بتشتغل Java؟ لازم تفهم الفرق ده.

---

## الصورة الكاملة في سطر واحد

```
JAR = مكتبة أو تطبيق مستقل بيشتغل لوحده
WAR = تطبيق ويب محتاج سيرفر (زي Tomcat) عشان يشتغل
```

---

## 🫙 أولاً: ما هو JAR؟

**JAR = Java ARchive**

تخيل إنك عندك مجموعة ملفات `.class` (كود Java المترجم) + Resources (صور، ملفات إعدادات...) — الـ JAR هو ببساطة ملف **ZIP** بيلمّ كل الحاجات دي في ملف واحد.

> 💡 فعلاً لو غيّرت امتداد `.jar` لـ `.zip` وفتحته، هتلاقي كل الملفات جوّاه!

### هيكل الـ JAR من الداخل:

```
my-app.jar
├── META-INF/
│   └── MANIFEST.MF          ← معلومات الـ JAR (مين الـ Main Class؟)
├── com/
│   └── namasoft/
│       └── Main.class        ← كود Java المترجم
└── application.properties    ← Resources
```

### الـ MANIFEST.MF — بطاقة هوية الـ JAR:

```
Manifest-Version: 1.0
Main-Class: com.namasoft.Main        ← ده اللي بيقول java -jar "ابدأ من هنا"
Class-Path: libs/spring-core.jar
Created-By: Apache Maven 3.9.0
```

---

### أنواع JAR:

#### 1. Library JAR — مكتبة بتستخدمها في مشروع تاني

```
spring-core-5.3.27.jar       ← مش بيشتغل لوحده — بتضيفه كـ Dependency
hibernate-core-5.6.jar
gson-2.10.1.jar
```

مش بيبقى فيه `Main-Class` في الـ MANIFEST — لأنه مش مفروض يتشغّل لوحده.

#### 2. Executable JAR — تطبيق كامل يشتغل لوحده

```bash
java -jar my-app.jar     ← بيشتغل مباشرة بدون أي حاجة تانية
```

#### 3. Fat JAR / Uber JAR — نوع خاص من الـ Executable JAR

الـ **Fat JAR** بيلمّ جوّاه:
- كود الـ App بتاعك
- كل الـ Dependencies (Spring, Hibernate...) كلها مدموجة جوّاه

```
my-fat-app.jar  (ممكن يوصل 80MB+)
├── com/namasoft/Main.class       ← كودك
├── org/springframework/...       ← Spring كامل جوّاه!
├── org/hibernate/...             ← Hibernate كامل جوّاه!
└── META-INF/MANIFEST.MF
```

> ✅ **ميزته:** ملف واحد بس — حطّه على أي جهاز فيه Java وشغّله. مش محتاج أي حاجة تانية.
> ❌ **عيبه:** حجمه كبير وبيتعمل من الصفر كل Build.

Spring Boot بيعمل Fat JAR بالظبط بـ `spring-boot-maven-plugin`.

### ازاي Maven يعمل JAR؟

```xml
<!-- في pom.xml -->
<packaging>jar</packaging>
```

```bash
mvn package
# النتيجة: target/my-app-1.0.jar
```

---

## 🏺 ثانياً: ما هو WAR؟

**WAR = Web Application ARchive**

الـ WAR هو تطبيق ويب كامل — فيه Servlets، JSPs، HTML، CSS، JS، وكل حاجة. لكن **مش بيشتغل لوحده** — محتاج **Web Container** (زي Tomcat أو JBoss أو WildFly) عشان يشغّله.

### هيكل الـ WAR من الداخل:

```
nama-erp.war
│
├── META-INF/
│   └── MANIFEST.MF
│
├── WEB-INF/                              ← ❗ الجزء الأهم — محمي من المتصفح
│   │
│   ├── web.xml                           ← إعداد التطبيق (Deployment Descriptor)
│   │
│   ├── classes/                          ← كود Java المترجم بتاعك
│   │   └── com/namasoft/
│   │       ├── controllers/
│   │       │   └── HomeController.class
│   │       └── services/
│   │           └── UserService.class
│   │
│   └── lib/                              ← كل الـ Dependencies هنا
│       ├── spring-core.jar
│       ├── hibernate-core.jar
│       ├── jackson-databind.jar
│       └── ...
│
├── index.html                            ← Static Files — بيتوصلها المتصفح
├── css/
│   └── style.css
├── js/
│   └── app.js
└── images/
    └── logo.png
```

> 🔒 **مهم جداً:** أي حاجة جوّا `WEB-INF/` **مش بيقدر المتصفح يوصلها مباشرة** — يعني لو حد كتب `http://mysite.com/WEB-INF/web.xml` مش هيشوف حاجة. ده أمان مقصود.

### الـ web.xml — مفتاح الـ WAR:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee" version="3.0">

    <display-name>Nama ERP</display-name>

    <!-- تسجيل الـ DispatcherServlet بتاع Spring -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

> 💡 في Spring Boot، الـ `web.xml` ممكن تتعوض بـ `WebApplicationInitializer` — بس في المشاريع الـ Classic Enterprise زي Nama ERP، الـ `web.xml` هو الطريقة الأصلية.

### ازاي Maven يعمل WAR؟

```xml
<!-- في pom.xml -->
<packaging>war</packaging>
```

```bash
mvn package
# النتيجة: target/nama-erp-1.0.war
```

---

## ⚖️ المقارنة التفصيلية: JAR vs WAR

| | JAR | WAR |
|-|-----|-----|
| **الاسم الكامل** | Java ARchive | Web Application ARchive |
| **الغرض الأساسي** | مكتبة أو تطبيق مستقل | تطبيق ويب |
| **يشتغل إزاي؟** | `java -jar app.jar` | بتنزله على Tomcat |
| **محتاج Web Server؟** | ❌ لا | ✅ نعم (Tomcat/JBoss...) |
| **فيه Static Files؟** | نادراً | ✅ HTML/CSS/JS |
| **فيه WEB-INF؟** | ❌ | ✅ |
| **فيه web.xml؟** | ❌ | ✅ (أو بديله) |
| **الـ Dependencies** | جوّاه (Fat JAR) أو خارجيه | دايماً في `WEB-INF/lib/` |
| **الامتداد** | `.jar` | `.war` |
| **في pom.xml** | `<packaging>jar</packaging>` | `<packaging>war</packaging>` |
| **بعد البناء** | `target/app.jar` | `target/app.war` |

---

## 🚀 Deploy إزاي؟

### JAR — التشغيل المباشر:

```bash
# أبسط طريقة
java -jar nama-app-1.0.jar

# مع إعدادات JVM
java -Xms512m -Xmx2g -jar nama-app-1.0.jar

# مع إعدادات Spring Boot
java -jar nama-app-1.0.jar --server.port=8080 --spring.profiles.active=prod
```

### WAR — النزول على Tomcat:

```
الخطوات:
──────────────────────────────────────────────────
1. بتبني الـ WAR:
   mvn clean package

2. بتاخد الملف:
   target/nama-erp.war

3. بتحطه في مجلد webapps على Tomcat:
   TOMCAT_HOME/webapps/nama-erp.war

4. Tomcat بيفكّه أوتوماتيك ويعمل مجلد:
   TOMCAT_HOME/webapps/nama-erp/

5. بتدخل عليه من المتصفح:
   http://localhost:8080/nama-erp/
──────────────────────────────────────────────────
```

> 💡 لو سمّيت الـ WAR بـ `ROOT.war` — التطبيق بيبقى على `http://localhost:8080/` مباشرة بدون اسم.

---

## 🌀 Spring Boot وقصة الاتنين

Spring Boot جاب معه تحوّل كبير — خلى الـ **JAR يقدر يشتغل زي الـ WAR**!

### ازاي؟ — Embedded Server (سيرفر مدموج)

Spring Boot بيحشر Tomcat **جوّا الـ JAR** نفسه!

```
الطريقة القديمة (WAR):
┌─────────────────────────────────────────┐
│  Tomcat (خارجي على السيرفر)            │
│  ├── webapps/                           │
│  │   └── nama-erp.war  ← تنزله هنا    │
└─────────────────────────────────────────┘

الطريقة الحديثة (Spring Boot JAR):
┌─────────────────────────────────────────┐
│  nama-erp.jar                           │
│  ├── BOOT-INF/classes/    ← كودك       │
│  ├── BOOT-INF/lib/                      │
│  │   ├── spring-web.jar                 │
│  │   ├── hibernate.jar                  │
│  │   └── tomcat-embed-core.jar ← 😲    │
│  └── META-INF/                          │
└─────────────────────────────────────────┘
          ↓
   java -jar nama-erp.jar   (Tomcat بيشتغل جوّاه!)
```

### هل ممكن Spring Boot يعمل WAR بدل JAR؟

**نعم!** ومهم تعرف ده — خصوصاً لو عندك Tomcat خارجي زي Nama ERP:

```xml
<!-- في pom.xml -->
<packaging>war</packaging>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>   <!-- ← مهم: Tomcat هيجي من السيرفر مش من الـ JAR -->
    </dependency>
</dependencies>
```

```java
// لازم تعمل كده بدل الـ main() العادي
public class Application extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }
}
```

---

## 🗂️ وفيه كمان EAR — للمشاريع الأضخم

**EAR = Enterprise ARchive**

ده النوع التالت — بيلمّ أكتر من WAR + JAR في ملف واحد بيتنزل على **Application Server** كامل (مش بس Tomcat).

```
company-system.ear
├── nama-erp.war          ← تطبيق الويب
├── nama-batch.jar        ← تطبيق Batch Jobs يشتغل في الخلفية
├── nama-shared.jar       ← مكتبات مشتركة بين الموديولات
└── META-INF/
    └── application.xml   ← إعداد الـ EAR
```

| | Tomcat | JBoss / WildFly / WebSphere |
|-|--------|-----------------------------|
| **يشغّل WAR؟** | ✅ | ✅ |
| **يشغّل EAR؟** | ❌ | ✅ |
| **EJB Support؟** | ❌ | ✅ |
| **JTA (Distributed Transactions)؟** | محدود | ✅ |
| **الثقل** | خفيف | ثقيل |

> 💡 الـ EAR بقى نادر في مشاريع جديدة — معظم المشاريع الحديثة بتفضّل Microservices + Spring Boot JAR بدل EAR الثقيل.

---

## 🧭 متى تستخدم إيه؟ — خريطة القرار

```
┌─────────────────────────────────────────────────────┐
│ إيه اللي بتعمله؟                                   │
└───────────────────────┬─────────────────────────────┘
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
    مكتبة تشاركها   تطبيق مستقل   تطبيق ويب
    مع مشاريع       أو Microservice
    تانية
          │             │             │
          ↓             ↓             ↓
    JAR (Library)  JAR (Fat JAR   على Tomcat
                   Spring Boot)   خارجي؟
                                      │
                               ┌──────┴──────┐
                               ↓             ↓
                              نعم            لا
                               │             │
                               ↓             ↓
                              WAR      JAR (Spring Boot
                                       Embedded Tomcat)
```

### ملخص الاختيار:

| الحالة | الاختيار |
|--------|----------|
| مكتبة تشاركها مع مشاريع تانية | `JAR` (Library) |
| تطبيق مستقل / Microservice | `JAR` (Fat JAR) |
| تطبيق ويب على Tomcat خارجي | `WAR` |
| Nama ERP على Tomcat | `WAR` ✅ |
| نظام Enterprise على JBoss/WildFly | `EAR` |

---

## 📌 خلاصة في جدول واحد

| | JAR | WAR | EAR |
|-|-----|-----|-----|
| **اختصار** | Java ARchive | Web ARchive | Enterprise ARchive |
| **بيشتغل على** | JVM مباشرة | Tomcat / Jetty | JBoss / WildFly |
| **بيتشغّل بـ** | `java -jar` | Deploy على السيرفر | Deploy على AppServer |
| **Static Files** | ❌ | ✅ | عن طريق WAR جوّاه |
| **في pom.xml** | `jar` | `war` | `ear` |
| **الانتشار** | كتير جداً | كتير | نادر (Legacy) |

---

> 📌 **نصيحة نهائية:** Nama ERP شغّال على Tomcat خارجي ومتعمل **WAR** — ده النهج الـ Classic Enterprise الصح لمشروع بالحجم ده. لو يوم من الأيام قررتوا تعملوا Microservices جنبه، هتلاقي نفسك بتعمل **JAR** لكل Service.
