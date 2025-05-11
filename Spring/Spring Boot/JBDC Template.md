## ✅ Spring Boot + JdbcTemplate (بالمصري)

### 🧠 يعني إيه JdbcTemplate؟

ال`JdbcTemplate` هو كلاس من Spring بيخليك تتعامل مع قاعدة البيانات بشكل سهل وآمن، وبتقدر تستخدمه بدل الـ JDBC التقليدي اللي بيحتاج تكتب كود كتير.

---

### 🔧 1. إعداد الـ Dependencies:

في ملف `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

---

### ⚙️ 2. إعداد الاتصال بقاعدة البيانات:

في ملف `application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

---

### 🧱 3. استخدام JdbcTemplate في الكود:

```java
@Repository
public class EmployeeRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public List<Employee> findAll() {
        String sql = "SELECT * FROM employees";
        return jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(Employee.class));
    }

    public void save(Employee emp) {
        String sql = "INSERT INTO employees (name, salary) VALUES (?, ?)";
        jdbcTemplate.update(sql, emp.getName(), emp.getSalary());
    }

    public Employee getById(int id) {
        String sql = "SELECT * FROM employees WHERE id = ?";
        return jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(Employee.class), id);
    }
}
```

---

### ✅ مميزات JdbcTemplate:

- بيقلل الكود اللي بتكتبه.
    
- بيحميك من SQL Injection.
    
- بيشتغل مباشرة مع قواعد البيانات.
    
- بيساعدك ترجّع البيانات كـ Java Objects بسهولة.
    

---

### 🛠️ أشهر Methods فيه:

|Method|وظيفة|
|---|---|
|`query()`|ترجّع List من النتائج|
|`update()`|تستخدمها للـ INSERT و UPDATE و DELETE|
|`queryForObject()`|ترجّع Row واحد كـ Object|
|`batchUpdate()`|تنفذ أكتر من Query مرة واحدة (Batch Mode)|

---

ده كده كل اللي محتاجه تبدأ بـ JdbcTemplate في Spring Boot.