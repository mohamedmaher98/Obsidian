
## 1) ApplicationContext

الApplicationContext هو الـ IoC Container في Spring.

مسؤول عن: - يعمل الـ Beans - يحقن الـ Dependencies - يدير الـ
Lifecycle - يدير الـ Scopes

في Spring Boot: SpringApplication.run() هو اللي يعمل ApplicationContext
تلقائيًا.

------------------------------------------------------------------------

## 2) أنواع الـ Container

### BeanFactory

-   أبسط نوع
-   Lazy loading افتراضيًا
-   نادر الاستخدام حاليًا

### ApplicationContext

-   الأشهر
-   فيه دعم Events
-   فيه دعم AOP
-   فيه دعم Web
-   هو المستخدم في Spring Boot

ممكن يبقى فيه Parent و Child Context في تطبيقات الويب.

------------------------------------------------------------------------

## 3) Servlet و DispatcherServlet

### Servlet

كلاس Java بيستقبل HTTP request ويرجع HTTP response. Tomcat يشغل Servlets
فقط.

### DispatcherServlet

هو Front Controller في Spring MVC. بيستقبل كل الطلبات ويوجهها للـ
Controller المناسب.

Flow الطلب:

Client ↓ Tomcat (Servlet Container) ↓ DispatcherServlet ↓ Controller ↓
Service ↓ Repository ↓ Database

------------------------------------------------------------------------

## 4) Server vs Service

Server = البرنامج اللي بيستقبل الطلبات (Tomcat)

Service = كلاس فيه Business Logic

------------------------------------------------------------------------

## 5) Bean Scope

Scope = عدد النسخ وعمر الـ Bean

### Singleton (الافتراضي)

-   نسخة واحدة لكل ApplicationContext
-   يعيش طول عمر التطبيق
-   مناسب للـ Services و Repositories

### Prototype

-   نسخة جديدة كل مرة نطلب Bean
-   Spring لا يدير مرحلة التدمير بعد الإنشاء

### Request Scope

-   نسخة لكل HTTP request

### Session Scope

-   نسخة لكل User Session

### Application Scope

-   نسخة واحدة لكل Web Application

------------------------------------------------------------------------

## 6) Stateful vs Stateless

### Stateless

-   مفيش متغيرات Field بتخزن بيانات متغيرة
-   آمن مع Singleton

### Stateful

-   فيه Fields بتتغير
-   خطر لو Singleton في Web App

Model classes زي User طبيعي تبقى Stateful. لكن Service classes المفروض
تبقى Stateless.

------------------------------------------------------------------------

## 7) Bean Lifecycle

مراحل حياة الـ Bean (Singleton):

1)  Instantiation (Spring يعمل object)
2)  Dependency Injection
3)  @PostConstruct
4)  Bean in use
5)  @PreDestroy (عند إغلاق التطبيق)

@PostConstruct تتنفذ مرة واحدة في Singleton. @PreDestroy لا تعمل مع
Prototype.

------------------------------------------------------------------------

## 8) REST API

REST API: - بيعتمد على HTTP - بيرجع JSON غالبًا - يستخدم GET / POST /
PUT / DELETE

أنواع أخرى: - SOAP (XML - قديم) - GraphQL (تحديد البيانات المطلوبة) -
gRPC (سريع - Microservices)

------------------------------------------------------------------------

# الخلاصة النهائية

Spring Boot Application =

-   ApplicationContext يدير Beans
-   Embedded Tomcat يشغل Servlet Container
-   DispatcherServlet يوجه الطلبات
-   Controllers تستقبل HTTP
-   Services تنفذ Business Logic
-   Repositories تتعامل مع Database
-   Scopes تحدد عدد النسخ
-   Lifecycle يدير بداية ونهاية الـ Beans

أنت دلوقتي فاهم المعمارية الكاملة من أول الطلب لحد إدارة عمر الـ Bean.
