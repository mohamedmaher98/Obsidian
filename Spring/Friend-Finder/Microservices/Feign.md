### 🔹 يعني إيه Feign؟

لما يبقى عندك **Microservices** (خدمات كتيرة كل واحدة مستقلة)، طبيعي الخدمات دي محتاجة تكلم بعض.

- ممكن تكلمها بـ **RestTemplate** أو **WebClient** (تكتب request بإيدك: URL, headers, body…).
    
- بس ده مرهق وبيعمل كود كتير.
    

هنا بييجي دور **Feign Client** → مكتبة بتخلّي استدعاء REST Service شبه إنك بتنده على **method عادية في Java**.

### 🔹 ازاي بيشتغل؟

- أنت بتكتب **Interface** عادي (كأنك عامل Service عادي).
    
- تحط عليه Annotation من Feign زي `@FeignClient`.
    
- جوا الـ Interface بتكتب الميثودز اللي هتستدعي API تانية.
    
- Feign من ورا الكواليس بيحوّل الميثود دي لـ **HTTP Request** ويروح يجيبلك النتيجة.
- ### 🔹 مثال عملي:

عندنا سيرفس اسمه **Football-Service** بيجيب بيانات اللاعبين من `/players`.  
وفي سيرفس تاني اسمه **Order-Service** عايز يجيب البيانات دي.

#### 1. نضيف dependency بتاعة Feign:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### 2. نفعّل Feign في المشروع:

في الـ **main class**

```java
@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

```

#### 3. نعمل Interface للـ Feign Client:
```java
@FeignClient(name = "football-service", url = "http://localhost:8081")
public interface FootballClient {

    @GetMapping("/players")
    List<Player> getAllPlayers();

    @GetMapping("/players/{id}")
    Player getPlayerById(@PathVariable("id") Long id);
}

```
#### 4. نستخدمه في Service عندنا:

```java
@Service
public class OrderService {
    private final FootballClient footballClient;

    public OrderService(FootballClient footballClient) {
        this.footballClient = footballClient;
    }

    public void makeOrder(Long playerId) {
        Player player = footballClient.getPlayerById(playerId);
        System.out.println("Buying player: " + player.getName());
    }
}

```

### 🔹 بالبلدي:

- **بدل ما تعك وتكتب RestTemplate وتزبط request/response**  
    → بتكتب Interface عادي، وFeign بيكلم المايكروسيرفس التاني بالنيابة عنك.
    
- **كأنك بتكلم method محلية**، لكن الحقيقة إنها **API Call**.