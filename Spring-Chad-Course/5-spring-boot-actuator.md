# Spring Boot Actuator — شرح من الصفر 🚀

---

## أولاً: الـ Actuator ده إيه أصلاً؟

تخيل معايا إنك اشتريت عربية جديدة وركّبت فيها **لوحة قيادة (Dashboard)**.

اللوحة دي بتقولك:
- السرعة كام؟
- الوقود كام باقي؟
- درجة حرارة الموتور؟
- فيه مشكلة فين؟

**الSpring Boot Actuator** هو نفس الفكرة بالظبط، بس لتطبيق الـ Spring Boot بتاعك.

هو عبارة عن **"لوحة قيادة"** جاهزة بتضيفها لتطبيقك، وفي الحال بتقدر تعرف:

- التطبيق شغال ولا وقع؟
- بيستهلك كام Memory؟
- الـ Database متصل ولا لأ؟
- الـ Endpoints اللي فيه إيه؟
- الـ Logs على أي Level؟
- وكتير كمان...

وكل ده من غير ما تكتب ولا سطر كود إضافي تقريباً! ✨

---

## تانياً: إزاي أضيف الـ Actuator للمشروع؟

### الخطوة الوحيدة: ضيف الـ Dependency

في `pom.xml` بتاعك ضيف:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

خلاص! الـ Actuator اتضاف. 🎉

---

## تالثاً: إزاي أشوف اللي الـ Actuator بيعمله؟

بعد ما تشغّل التطبيق، افتح المتصفح أو Postman واعمل GET request على:

```
http://localhost:8080/actuator
```

هتلاقي response زي ده:

```json
{
  "_links": {
    "self": { "href": "http://localhost:8080/actuator" },
    "health": { "href": "http://localhost:8080/actuator/health" },
    "info": { "href": "http://localhost:8080/actuator/info" }
  }
}
```

ده فهرس الـ Endpoints المتاحة — زي قائمة المحتويات في كتاب.

> ⚠️ **ملاحظة مهمة:** بشكل افتراضي Spring Boot بيظهر Endpoints محدودة فقط (`health` و `info`) لأسباب أمان. هنشوف إزاي نفتح الباقي دلوقتي.

---

## رابعاً: الـ Endpoints المهمة دي إيه؟

فكّر في كل Endpoint إنه "سؤال" بتسأله لتطبيقك:

| الـ Endpoint | السؤال اللي بتسأله |
|---|---|
| `/actuator/health` | التطبيق بخير ولا لأ؟ |
| `/actuator/info` | إيه المعلومات العامة للتطبيق؟ |
| `/actuator/metrics` | الأرقام والإحصائيات إيه؟ |
| `/actuator/env` | الـ Environment Variables إيه؟ |
| `/actuator/beans` | الـ Spring Beans اللي اتعملت إيه؟ |
| `/actuator/mappings` | الـ REST Endpoints الموجودة في التطبيق إيه؟ |
| `/actuator/loggers` | الـ Logging على أي مستوى؟ |
| `/actuator/threaddump` | الـ Threads اللي بتشتغل دلوقتي إيه؟ |
| `/actuator/heapdump` | أعطيني صورة من الـ Memory (Heap) |
| `/actuator/shutdown` | قفّل التطبيق (خطر! مش بيتفعّل افتراضياً) |

---

## خامساً: إزاي أفتح كل الـ Endpoints؟

في ملف `application.properties` بتاعك ضيف:

```properties
# فتح كل الـ endpoints على الـ HTTP
management.endpoints.web.exposure.include=*

# أو لو عايز تحدد بعض منهم بس
management.endpoints.web.exposure.include=health,info,metrics,loggers
```

أو في `application.yml`:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

---

## سادساً: شرح كل Endpoint مهم بالتفصيل

---

### 1. `/actuator/health` — هل التطبيق بخير؟

ده أهم Endpoint على الإطلاق.

**بيقولك إيه:**
- التطبيق `UP` (شغال) ولا `DOWN` (وقع)
- حالة الـ Database
- حالة الـ Disk Space
- حالة أي خدمة تانية متصل بيها

**مثال على الـ Response:**

```json
{
  "status": "UP"
}
```

ده الشكل الافتراضي البسيط. عشان تشوف التفاصيل، ضيف في `application.properties`:

```properties
management.endpoint.health.show-details=always
```

دلوقتي الـ Response هيبقى:

```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "Microsoft SQL Server",
        "validationQuery": "isValid()"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 500107862016,
        "free": 200500000000,
        "threshold": 10485760,
        "exists": true
      }
    }
  }
}
```

**الـ Status ممكن يكون:**
- `UP` — كل حاجة تمام ✅
- `DOWN` — في مشكلة ❌
- `OUT_OF_SERVICE` — الخدمة مش متاحة مؤقتاً
- `UNKNOWN` — مش عارف الحالة

> 💡 **فايدة عملية:** الـ DevOps أو الـ Kubernetes بيضرب على الـ `/health` كل شوية عشان يعرف لو التطبيق محتاج restart أو لأ.

---

### 2. `/actuator/info` — معلومات عامة عن التطبيق

بشكل افتراضي بيرجع response فاضي، بس انت اللي بتحدد إيه اللي يتعرض.

**في `application.properties` ضيف:**

```properties
info.app.name=My Spring Boot App
info.app.version=1.0.0
info.app.description=Enterprise Resource Planning System
info.app.developer=My Company
```

**دلوقتي الـ Response:**

```json
{
  "app": {
    "name": "My Spring Boot App",
    "version": "1.0.0",
    "description": "Enterprise Resource Planning System",
    "developer": "My Company"
  }
}
```

---

### 3. `/actuator/metrics` — الأرقام والإحصائيات

ده كنز! بيديك قياسات دقيقة جداً عن كل حاجة في التطبيق.

**أول ما تفتحه هتلاقي قائمة بكل الـ metrics المتاحة:**

```json
{
  "names": [
    "jvm.memory.used",
    "jvm.memory.max",
    "jvm.gc.pause",
    "http.server.requests",
    "process.cpu.usage",
    "system.cpu.usage",
    "tomcat.threads.busy",
    "tomcat.threads.current"
  ]
}
```

**عشان تشوف قيمة metric معينة:**

```
GET /actuator/metrics/jvm.memory.used
```

```json
{
  "name": "jvm.memory.used",
  "description": "The amount of used memory",
  "baseUnit": "bytes",
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 256000000
    }
  ]
}
```

**أهم الـ Metrics اللي محتاج تعرفها:**

| الـ Metric | بيقيس إيه؟ |
|---|---|
| `jvm.memory.used` | الـ Memory اللي بتتاكل دلوقتي |
| `jvm.memory.max` | أقصى Memory ممكن يستخدمها |
| `process.cpu.usage` | نسبة الـ CPU اللي التطبيق بياخده |
| `http.server.requests` | عدد الـ HTTP Requests ووقت استجابتها |
| `tomcat.threads.busy` | عدد الـ Threads المشغولة في Tomcat |

---

### 4. `/actuator/loggers` — التحكم في الـ Logs

ده الـ Endpoint السحري! مش بس بيقولك الـ Logging Levels، لكن بيخليك **تغيّر الـ Level من غير ما تعيد تشغيل التطبيق!**

**شوف الـ Level الحالي لـ Package معين:**

```
GET /actuator/loggers/com.example.myapp
```

```json
{
  "configuredLevel": "INFO",
  "effectiveLevel": "INFO"
}
```

**غيّر الـ Level وانت التطبيق شغال!**

```
POST /actuator/loggers/com.example.myapp
Content-Type: application/json

{
  "configuredLevel": "DEBUG"
}
```

دلوقتي الـ Package ده هيبدأ يطلع DEBUG logs من غير restart! 🔥

لما تخلص من التشخيص، رجّعه:

```
POST /actuator/loggers/com.example.myapp
Content-Type: application/json

{
  "configuredLevel": "INFO"
}
```

---

### 5. `/actuator/env` — الـ Environment Variables

بيعرض كل الـ Properties اللي التطبيق شايفها من كل مصادرها:

- ملفات الـ `application.properties`
- الـ System Environment Variables
- الـ JVM Arguments
- وغيرها

```json
{
  "activeProfiles": ["production"],
  "propertySources": [
    {
      "name": "application.properties",
      "properties": {
        "server.port": { "value": "8080" },
        "spring.datasource.url": { "value": "******" }
      }
    }
  ]
}
```

> ⚠️ **تنبيه أمان:** قيم زي passwords وsecrets بتتخبّى تلقائياً (`******`). بس احذر من تفعيل الـ `/env` في الـ Production من غير حماية!

---

### 6. `/actuator/mappings` — إيه الـ Endpoints الموجودة؟

بيعرض كل الـ REST Endpoints اللي في تطبيقك مع كل تفاصيلها:

```json
{
  "contexts": {
    "application": {
      "mappings": {
        "dispatcherServlets": {
          "dispatcherServlet": [
            {
              "handler": "com.example.InvoiceController#getInvoice(Long)",
              "predicate": "{GET /api/invoices/{id}}"
            }
          ]
        }
      }
    }
  }
}
```

---

### 7. `/actuator/beans` — الـ Spring Beans اللي اتعملت

بيعرض كل الـ Beans اللي الـ Spring Container عملها. مفيد لما يكون عندك مشكلة في الـ Dependency Injection وعايز تتأكد إن Bean معين اتعمل.

---

## سابعاً: كيف أحمي الـ Actuator Endpoints؟

ده سؤال مهم جداً! الـ Actuator Endpoints فيها معلومات حساسة، فمش المفروض يبقى متاح للكل.

### الحل 1: غيّر الـ Port بتاع الـ Actuator

```properties
# التطبيق على 8080
server.port=8080

# الـ Actuator على port تاني مقفول من الـ Firewall
management.server.port=9090
```

دلوقتي الـ Actuator مش هيتوصله حد من برة إلا من الـ Internal Network.

### الحل 2: غيّر الـ Base Path

```properties
management.endpoints.web.base-path=/internal-monitor
```

بدل `/actuator` هيبقى `/internal-monitor`.

### الحل 3: استخدم Spring Security

ضيف الـ Dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

وبعدين اعمل Security Config يحمي الـ Actuator Endpoints:

```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .requestMatcher(EndpointRequest.toAnyEndpoint())
            .authorizeRequests()
                .anyRequest().hasRole("ADMIN")
            .and()
            .httpBasic();
        return http.build();
    }
}
```

---

## ثامناً: Custom Health Indicator — اعمل Check خاص بيك

مش بس بتشوف الـ Built-in checks، تقدر تضيف Check خاص بيك!

مثلاً عايز تتأكد إن External Service معين شغال:

```java
@Component
public class ExternalServiceHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        boolean serviceIsUp = checkExternalService(); // منطقك هنا

        if (serviceIsUp) {
            return Health.up()
                         .withDetail("service", "ExternalPaymentAPI")
                         .withDetail("status", "reachable")
                         .build();
        } else {
            return Health.down()
                         .withDetail("service", "ExternalPaymentAPI")
                         .withDetail("reason", "Connection timeout")
                         .build();
        }
    }

    private boolean checkExternalService() {
        // هنا بتعمل ping أو HTTP call للسيرفس الخارجي
        return true;
    }
}
```

دلوقتي في `/actuator/health` هتلاقي:

```json
{
  "status": "UP",
  "components": {
    "externalService": {
      "status": "UP",
      "details": {
        "service": "ExternalPaymentAPI",
        "status": "reachable"
      }
    }
  }
}
```

---

## تاسعاً: Custom Metrics — قياسات خاصة بيك

تقدر تضيف Metrics خاصة بالـ Business Logic بتاعتك!

مثلاً عايز تعد كام Order اتعمل:

```java
@Service
public class OrderService {

    private final Counter orderCounter;

    public OrderService(MeterRegistry registry) {
        // سجّل الـ Counter في الـ Actuator
        this.orderCounter = Counter.builder("orders.created.total")
                                   .description("Total number of orders created")
                                   .register(registry);
    }

    public void createOrder(Order order) {
        // منطق إنشاء الأوردر
        saveOrder(order);

        // زوّد الـ Counter بواحد
        orderCounter.increment();
    }
}
```

دلوقتي تقدر تشوف:

```
GET /actuator/metrics/orders.created.total
```

```json
{
  "name": "orders.created.total",
  "description": "Total number of orders created",
  "measurements": [
    { "statistic": "COUNT", "value": 1523.0 }
  ]
}
```

---

## عاشراً: ملف الـ Properties كامل — Settings مقترحة

```properties
# ==========================================
# Actuator Settings
# ==========================================

# فتح الـ Endpoints المهمة
management.endpoints.web.exposure.include=health,info,metrics,loggers,env,mappings,beans

# إظهار تفاصيل الـ Health
management.endpoint.health.show-details=always

# تشغيل الـ Actuator على Port منفصل (للأمان)
management.server.port=9090

# غيّر الـ Base Path
management.endpoints.web.base-path=/actuator

# معلومات التطبيق
info.app.name=My Spring Boot App
info.app.version=1.0.0
```

---

## حادي عشر: الـ Actuator مع Docker و Kubernetes

لو شغّال بيئة Docker أو Kubernetes، الـ Actuator بيبقى أهم من كده!

الـ Kubernetes بيستخدم Endpoints خاصة اسمها **Probes**:

```yaml
# في ملف deployment.yaml الخاص بالـ Kubernetes
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 20
  periodSeconds: 5
```

وفي `application.properties` فعّل الـ Probes دي:

```properties
management.endpoint.health.probes.enabled=true
```

- **Liveness Probe:** لو فشل، الـ Kubernetes بيعمل restart للـ Container.
- **Readiness Probe:** لو فشل، الـ Kubernetes بيوقف إرسال Traffic للـ Container.

---

## خلاصة — الكلام في نقط

| الموضوع | الخلاصة |
|---|---|
| **الـ Actuator إيه؟** | لوحة قيادة جاهزة لتطبيق Spring Boot |
| **إزاي أضيفه؟** | Dependency واحدة في `pom.xml` |
| **أهم Endpoint** | `/health` — دايماً اشغله |
| **الـ Metrics** | قياسات JVM و HTTP و CPU وغيرها |
| **الـ Loggers** | تغيير الـ Log Level بدون restart |
| **الأمان** | لازم تحمي الـ Endpoints دي في Production |
| **التخصيص** | تقدر تضيف Custom Health و Custom Metrics |
| **الـ DevOps** | مهم جداً لـ Docker و Kubernetes |

---

> 💬 **كلمة أخيرة:**
> الـ Actuator مش حاجة "advanced" أو للمحترفين بس — ده جزء أساسي من أي تطبيق Production. أي تطبيق بتشغّله من غير Actuator، إنت بتطير "أعمى" — مش عارف إيه اللي بيحصل جوّاه. ابدأ بـ `/health` و `/metrics` ودي هتكفيك في البداية. 🎯
