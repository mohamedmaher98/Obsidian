

---

## 📦 إيه هي الـ **Persistence Layer**؟

هي **الطبقة اللي مسؤولة عن حفظ واسترجاع البيانات** من وإلى قاعدة البيانات (Database).  
يعني أي كود بيتعامل مع الـ DB مباشرة (زي SELECT, INSERT, UPDATE, DELETE) بيبقى في الطبقة دي.

---

## 🧱 الطبقة دي بتحتوي على إيه؟

1. **Repositories**  
    → الكلاسات أو الـ interfaces اللي بتتعامل مع DB (زي DAO أو JPA Repositories)
    
2. **Entities**  
    → الكائنات اللي بتمثل الجداول في قاعدة البيانات.
    
3. **ORM Tools** (زي Hibernate أو JPA)  
    → علشان نربط الجداول بالكائنات بطريقة أوتوماتيك.
    

---

## 🧪 مثال عملي (JPA):

### 1. كيان (Entity):



```java
@Entity
public class Employee {  
@Id
@GeneratedValue
private Long id;
private String name;
private double salary;
// getters & setters }`
```

---

### 2. Repository:

java

Copy code

```java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
List<Employee> findByName(String name);
}`
```

---

## 💡 فوائد الـ Persistence Layer:

|الفايدة|الشرح|
|---|---|
|فصل الكود|بتخلي التعامل مع البيانات منفصل عن الـ Business Logic|
|سهولة التست|تقدر تعمل Mock للـ Repository بسهولة|
|إعادة استخدام|الكود بيبقى reusable في كذا مكان|
|استخدام ORM|بتقدر تستخدم أدوات زي Hibernate/JPA بدل SQL مباشر|

---

## 🔁 العلاقة بين الـ Layers:

nginx

Copy code

`Controller → Service Layer → Persistence Layer → Database`

![[Pasted image 20250510210259.png]]