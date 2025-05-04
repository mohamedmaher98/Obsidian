-- إنشاء قاعدة بيانات جديدة
CREATE DATABASE TrainingDB;
-- استخدام قاعدة البيانات
USE TrainingDB;

==create table ==
CREATE TABLE -Table_name-
![[Pasted image 20250425183855.png]]

==DataTypes==

1- text Data Types

|   |   |   |
|---|---|---|
|`CHAR(n)`|نص بطول ثابت|مثال: `CHAR(10)` يعني لازم 10 حروف حتى لو الكلمة أقصر|

|   |   |   |
|---|---|---|
|`VARCHAR(n)`|نص بطول متغير|أفضل في المساحة لو الكلمة قصيرة، مثال: `VARCHAR(100)`|

|   |   |   |
|---|---|---|
|`NCHAR(n)`|نص ثابت يدعم العربي|زي `CHAR` لكن يدعم Unicode|

|   |   |   |
|---|---|---|
|`NVARCHAR(n)`|نص متغير يدعم العربي|الأفضل لما تتعامل مع اللغة العربية أو أي لغات فيها رموز|

==Numeric==

`DECIMAL(10,2)` = رقم مكون من 10 خانات، منهم 2 بعد الفاصلة (زي: 12345678.90)|   |

**==Date & Time==**

Date -- > تاريخ فقط (yyyy-mm-dd)
Time  --> وقت فقط (hh:mm:ss)
DateTime --> تاريخ ووقت معًا

`DATETIME` = `2024-04-25 14:30:00'

----------
### **==DDL== --> Date Definition Language**

إنشاء جدول جديد
|   |
|---|
|`CREATE TABLE Students (id INT, name VARCHAR(50));`|

تعديل جدول موجود 

ALTER TABLE Students ADD birthdate DATE;

حذف جدول بالكامل 

DROP TABLE Students;

مسح كل البيانات فالجدول بدون حذف الجدول 

TRUNCATE TABLE Students;

إعادة تسمية جدول

EXEC sp_rename 'OldName', 'NewName';

---------------
### ==DML== --> Data Manipulation Language

أوامر التعامل مع البيانات
|   |
|---|
INSERT INTO Students (id, name) 
VALUES (1, 'Ali');
تعديل بيانات موجودة
UPDATE Students SET name = 'Omar' WHERE id = 1;

حذف بيانات موجوده 
DELETE FROM Students WHERE id = 1;

-------
### DQL - Data Query Language

| الأمر      | الوظيفة           | مثال                                                                    |
| ---------- | ----------------- | ----------------------------------------------------------------------- |
| `SELECT`   | استعلام بيانات    | `SELECT * FROM Students;`                                               |
| `WHERE`    | شرط               | `SELECT * FROM Students WHERE name = 'Ali';`                            |
| `ORDER BY` | ترتيب النتائج     | `SELECT * FROM Students ORDER BY name ASC;`                             |
| `GROUP BY` | تجميع النتائج     | `SELECT department_id, COUNT(*) FROM Employees GROUP BY department_id;` |
| `HAVING`   | شرط بعد الـ Group | `HAVING COUNT(*) > 5`                                                   |
| `DISTINCT` | إزالة التكرار     | `SELECT DISTINCT department_id FROM Employees;`                         |
| `TOP`      | تحديد عدد النتائج | `SELECT TOP 5 * FROM Employees;`                                        |


-------
### **TCL -> Transaction Control Language**
| الأمر               | الوظيفة        | مثال                 |
| ------------------- | -------------- | -------------------- |
| `BEGIN TRANSACTION` | بدء معاملة     | `BEGIN TRANSACTION;` |
| `COMMIT`            | تأكيد المعاملة | `COMMIT;`            |
| `ROLLBACK`          | إلغاء المعاملة | `ROLLBACK;`          |


---

### دوال مهمة بتستخدم مع `SELECT`

| الدالة    | الاستخدام | مثال                                 |
| --------- | --------- | ------------------------------------ |
| `COUNT()` | عد الصفوف | `SELECT COUNT(*) FROM Students;`     |
| `SUM()`   | مجموع     | `SELECT SUM(salary) FROM Employees;` |
| `AVG()`   | المتوسط   | `SELECT AVG(age) FROM Students;`     |
| `MIN()`   | أقل قيمة  | `SELECT MIN(price) FROM Products;`   |
| `MAX()`   | أعلى قيمة | `SELECT MAX(price) FROM Products;`   |

----

--100
 
 