## إيه هو Spring Data JDBC؟

هو مشروع من Spring بيسهّل التعامل مع قواعد البيانات (زي MySQL, Postgres...) باستخدام JDBC لكن:
 **من غير ORM زي Hibernate**  
يعني بتشتغل على الـ Database مباشرة، لكن من غير تعقيدات الـ JPA/Hibernate.
أي الفرق مابينه ومابين JBA

|العنصر|Spring Data JDBC|Spring Data JPA (Hibernate)|
|---|---|---|
|ORM|❌ مفيش ORM|✅ بيستخدم Hibernate|
|الأداء|أسرع وأبسط|أبطأ شوية بسبب الكاش والعلاقات|
|التعقيد|بسيط وسهل|معقد شوية بسبب الـ mapping والعلاقات|
|العلاقات (Relations)|دعم بسيط|دعم كامل (OneToMany - ManyToOne...)|

---
## 🛠️ إزاي أبدأ أشتغل بيه؟

### 1. أضف Dependency في `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>

```

---
فال  ENTITY
```java
@Table("employees")
public class Employee {
    @Id
    private Long id;

    private String name;
    private double salary;

    // getters/setters
}
✅ مفيش `@Entity` هنا — بدلها بنستخدم `@Table`.


in repo 
@Repository
public interface EmployeeRepository extends CrudRepository<Employee, Long> {
    List<Employee> findByName(String name);
}

```


## ✅ مميزات Spring Data JDBC:

- أبسط من JPA، مفيهوش كاش أو Lazy Loading
    
- سريع ومباشر للتعامل مع الجداول
    
- مناسب للمشاريع الصغيرة والمتوسطة أو اللي مش محتاجة علاقات معقدة

---
## ❗ عيوبه:

- مفيهوش دعم للعلاقات المعقدة (زي JPA)
    
- كل علاقة (Relation) لازم تتعالج يدويًا