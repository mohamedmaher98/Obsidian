## 🧭 يعني إيه Mapper في Spring / Java؟

### ✅ تعريف بسيط:

**Mapper** هو كود بيحوّل البيانات من شكل معين لشكل تاني.

- من قاعدة بيانات → Object
    
- من Entity → DTO
    
- من DTO → Entity
    

---

## 🔥 الأنواع المشهورة من الـ Mappers:

### 🟢 1. RowMapper (في JdbcTemplate)

بيحوّل كل صف جاي من قاعدة البيانات إلى Object:

```java
public class EmployeeRowMapper implements RowMapper<Employee> {
    @Override
    public Employee mapRow(ResultSet rs, int rowNum) throws SQLException {
        Employee emp = new Employee();
        emp.setId(rs.getInt("id"));
        emp.setName(rs.getString("name"));
        emp.setSalary(rs.getDouble("salary"));
        return emp;
    }
}
```

الاستخدام:

```java
List<Employee> list = jdbcTemplate.query("SELECT * FROM employees", new EmployeeRowMapper());
```

---

### 🟢 2. ModelMapper (Object ↔ Object)

أداة جاهزة من مكتبة بتعمل mapping تلقائي بين Objects زي Entity و DTO:

```java
ModelMapper modelMapper = new ModelMapper();
EmployeeDTO dto = modelMapper.map(employee, EmployeeDTO.class);
```

---

### 🟢 3. MapStruct (أداء أعلى)

بتولد الكود وقت الـ Compile، وبتكون أسرع من ModelMapper.

```java
@Mapper
public interface EmployeeMapper {
    EmployeeDTO toDto(Employee emp);
    Employee toEntity(EmployeeDTO dto);
}
```

---

## 🤔 ليه نستخدم الـ Mapper؟

- علشان نفصل بين طبقات المشروع (Separation of Concerns)
    
- نحافظ على الكود منظم ونظيف
    
- نحضّر البيانات قبل العرض أو التخزين
    

---

## 📌 خلاصة:

|النوع|بيشتغل على إيه؟|مميزاته|
|---|---|---|
|RowMapper|DB → Java Object|سهل ومباشر في JdbcTemplate|
|ModelMapper|Object → Object|تلقائي وسهل|
|MapStruct|Object → Object|أسرع وأكتر كفاءة|