الSpring Cloud Config Server هو سيرفيس بيخزن **ملفات الإعدادات (configurations)** الخاصة بكل الميكروسيرفيسز في مكان مركزي، بحيث كل سيرفيس يقدر يقرأ إعداداته من هناك بدل ما تخزنها محليًا.

غالبًا بنخزن الإعدادات دي في **مستودع Git** علشان:

- نقدر نعمل version control
    
- نستفيد من الـ branching
    
- نرجع لإصدار قديم بسهولة


## تخيل معايا:

عندك 5 ميكروسيرفيس (A, B, C, D, E)، وكل واحد فيهم محتاج إعدادات (زي قواعد البيانات، روابط الـ APIs، الـ ports...).

لو كل ميكروسيرفيس حاطط الإعدادات في ملفه الخاص، هيبقى صعب تغيرها بعدين، وهتضطر تعدل كل واحد على حدة.

### الحل:

نعمل **مخزن إعدادات مركزي** في مكان واحد، وكل الميكروسيرفيسز تقرأ منه الإعدادات.  
المكان ده هو **Spring Cloud Config Server**.


### طيب الإعدادات نفسها تتحط فين؟

الإعدادات (ملفات `application.yml` أو `application.properties`) بنحطها في **Repo على Git**.  
ليه Git؟

- بيحتفظ بتاريخ كل التعديلات
    
- نقدر نرجع لأي إصدار قديم
    
- نقدر نشتغل بفروع مختلفة (Branches) للإعدادات
  
  
  ### هنا بقى بيجي دور:

`spring.cloud.config.server.git.uri`  
دي بتقول لـ **Config Server** مكان الـ Git Repo اللي فيه ملفات الإعدادات.

### الخلاصة:
- **Config Server** = بواب يوزع الإعدادات على الميكروسيرفيسز.
    
- **`spring.cloud.config.server.git.uri`** = العنوان اللي بيقول للبواب يجيب الإعدادات منين.

![[Pasted image 20250814074406.png]]


---
لو عايز **تشغل Config Server** في Spring Boot، لازم تعمل 3 خطوات أساسية:


## 1️⃣ إضافة الـ Dependency الخاصة بـ Config Server


```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>

```

## 2️⃣ تفعيل Config Server

في الـ **Main Application Class**، أضف الأنوتيشن:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer  // ← دي اللي بتشغل Config Server
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

```

## 3️⃣ تحديد مكان الـ Git Repo اللي فيه ملفات الإعدادات

`properties
server.port=8888
spring.cloud.config.server.git.uri=https://github.com/username/config-repo
spring.cloud.config.server.git.default-label=main


---
بعد كده، لما تشغل المشروع، **Config Server** هيبقى شغال على البورت اللي حددته (افتراضي 8888) ويقرأ الإعدادات من Git Repo.