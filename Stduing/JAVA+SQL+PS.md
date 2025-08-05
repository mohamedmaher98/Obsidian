

---

## 📚 خطة Java - بعد الأساسيات والـ OOP

### الأسبوع الأول: Collections Framework & Exception Handling

**اليوم 1-2: Collections Framework**

- ArrayList vs LinkedList - متى نستخدم كل واحد
- HashMap, TreeMap, LinkedHashMap - الفروقات والاستخدامات
- HashSet, TreeSet, LinkedHashSet
- عمل مشاريع صغيرة: نظام إدارة طلاب، مخزن منتجات

**اليوم 3-4: Exception Handling**

- try-catch-finally blocks
- Custom Exceptions
- throw vs throws
- Best practices في التعامل مع الأخطاء
- مشروع: نظام بنكي بسيط مع exception handling

**اليوم 5: Generics**

- Generic Classes and Methods
- Wildcards (? extends, ? super)
- Type Erasure مفهوم أساسي
- مشروع: Generic Stack and Queue implementation

### الأسبوع الثاني: Multithreading & Concurrency

**اليوم 1-2: Thread Basics**

- Thread class vs Runnable interface
- Thread lifecycle and states
- synchronized keyword
- wait(), notify(), notifyAll()
- مشروع: Producer-Consumer problem

**اليوم 3-4: Advanced Concurrency**

- ExecutorService and Thread Pools
- Callable and Future
- CountDownLatch, Semaphore
- مشروع: Multi-threaded file downloader

**اليوم 5: Java 8+ Features**

- Lambda Expressions
- Functional Interfaces
- Method References
- مشروع: تحويل كود قديم لاستخدام lambdas

### الأسبوع الثالث: Stream API & File I/O

**اليوم 1-2: Stream API**

- Stream creation and intermediate operations
- Terminal operations (collect, reduce, forEach)
- Parallel Streams
- مشروع: تحليل بيانات CSV باستخدام streams

**اليوم 3-4: File I/O & NIO**

- FileInputStream, FileOutputStream
- BufferedReader, BufferedWriter
- Files class and Path API
- مشروع: File manager application

**اليوم 5: Serialization**

- Object Serialization/Deserialization
- transient keyword
- مشروع: Save and load game state

### الأسبوع الرابع: Design Patterns & Advanced Topics

**اليوم 1-2: Essential Design Patterns**

- Singleton Pattern
- Factory Pattern
- Observer Pattern
- مشروع تطبيقي لكل pattern

**اليوم 3-4: More Design Patterns**

- Strategy Pattern
- Decorator Pattern
- Command Pattern
- مشروع: Text Editor مع multiple patterns

**اليوم 5: Reflection & Annotations**

- Class, Method, Field objects
- Custom Annotations
- مشروع: Simple dependency injection framework

### الأسبوع الخامس: Database Integration

**اليوم 1-2: JDBC Fundamentals**

- Connection, Statement, PreparedStatement
- ResultSet handling
- Connection pooling basics
- مشروع: Student Management System

**اليوم 3-4: Advanced JDBC**

- Batch processing
- Transaction management
- Stored procedure calls
- مشروع: Banking system مع transactions

**اليوم 5: JPA/Hibernate Basics**

- Entity mapping
- Basic CRUD operations
- مشروع: Convert JDBC project to JPA

### الأسبوع السادس: Web Development & Final Project

**اليوم 1-2: Servlets & JSP**

- HttpServlet basics
- Request/Response handling
- Session management
- مشروع: Simple web application

**اليوم 3-5: Spring Boot Introduction**

- Spring Boot setup
- REST Controllers
- Dependency Injection
- مشروع نهائي: REST API مع database integration

---

## 🗄️ خطة SQL - من الأساسيات

### الأسبوع الأول: SQL Fundamentals

**اليوم 1: Database Basics & Setup**

- فهم Databases, Tables, Rows, Columns
- تنصيب MySQL/PostgreSQL
- SQL syntax basics
- CREATE DATABASE, USE database

**اليوم 2: Creating Tables & Data Types**

- CREATE TABLE statement
- Data types: INT, VARCHAR, DATE, BOOLEAN, etc.
- Primary Keys and Auto Increment
- تمرين: إنشاء قاعدة بيانات متجر

**اليوم 3: INSERT, SELECT Basics**

- INSERT INTO statements
- SELECT * and specific columns
- WHERE clause basics
- ORDER BY and LIMIT
- تمرين: إدخال وعرض بيانات المنتجات

**اليوم 4: UPDATE & DELETE**

- UPDATE statements with WHERE
- DELETE statements
- Safe updates and backups
- تمرين: إدارة المخزون

**اليوم 5: Data Filtering**

- WHERE with multiple conditions (AND, OR)
- IN, NOT IN operators
- BETWEEN operator
- LIKE and wildcards (%, _)
- IS NULL, IS NOT NULL
- تمرين شامل: تصفية بيانات العملاء

### الأسبوع الثاني: Intermediate Queries

**اليوم 1: Aggregate Functions**

- COUNT, SUM, AVG, MAX, MIN
- GROUP BY clause
- HAVING clause
- تمرين: تقارير المبيعات

**اليوم 2: JOINs - Part 1**

- INNER JOIN
- LEFT JOIN (LEFT OUTER JOIN)
- فهم العلاقات بين الجداول
- تمرين: ربط جداول العملاء والطلبات

**اليوم 3: JOINs - Part 2**

- RIGHT JOIN
- FULL OUTER JOIN
- CROSS JOIN
- Self JOIN
- تمرين: تحليل بيانات معقدة

**اليوم 4: Subqueries**

- Subqueries في WHERE clause
- Subqueries في FROM clause
- EXISTS and NOT EXISTS
- تمرين: العثور على أفضل العملاء

**اليوم 5: UNION & Advanced Filtering**

- UNION vs UNION ALL
- CASE statements
- تمرين شامل: تقرير مبيعات متقدم

### الأسبوع الثالث: Database Design & Normalization

**اليوم 1: Database Design Principles**

- Entity Relationship Diagrams (ERD)
- Primary vs Foreign Keys
- One-to-One, One-to-Many, Many-to-Many relationships
- تمرين: تصميم قاعدة بيانات مكتبة

**اليوم 2: Normalization**

- 1st, 2nd, 3rd Normal Forms
- Denormalization متى نحتاجها
- تمرين: تطبيق normalization على مشروع

**اليوم 3: Constraints & Indexes**

- FOREIGN KEY constraints
- UNIQUE constraints
- CHECK constraints
- CREATE INDEX
- تمرين: تحسين أداء الاستعلامات

**اليوم 4: Views**

- CREATE VIEW
- Updatable vs Non-updatable views
- استخدامات Views في الأمان
- تمرين: إنشاء views للتقارير

**اليوم 5: مراجعة وتطبيق**

- حل مسائل معقدة
- تصميم قاعدة بيانات من الصفر

### الأسبوع الرابع: Advanced SQL

**اليوم 1: Window Functions**

- ROW_NUMBER(), RANK(), DENSE_RANK()
- PARTITION BY
- Running totals مع SUM() OVER()
- تمرين: تحليل المبيعات الشهرية

**اليوم 2: Common Table Expressions (CTEs)**

- WITH clause
- Recursive CTEs
- تمرين: الهيكل التنظيمي للشركة

**اليوم 3: Stored Procedures & Functions**

- CREATE PROCEDURE
- Parameters (IN, OUT, INOUT)
- User-defined functions
- تمرين: stored procedures للعمليات المتكررة

**اليوم 4: Triggers**

- BEFORE and AFTER triggers
- INSERT, UPDATE, DELETE triggers
- تمرين: audit trail system

**اليوم 5: Performance & Optimization**

- Query execution plans
- Index optimization
- Query optimization techniques

### الأسبوع الخامس-السادس: Real-World Applications

**الأسبوع 5: مشاريع تطبيقية**

- مشروع 1: نظام إدارة المكتبة
- مشروع 2: نظام نقاط البيع
- مشروع 3: تحليل بيانات المبيعات

**الأسبوع 6: Integration & NoSQL Basics**

- Integration مع Java (JDBC)
- MongoDB basics للمقارنة
- مراجعة شاملة وحل مسائل معقدة

---

## 🧩 خطة Problem Solving

### المرحلة الأولى (الأسبوع 1-2): Easy Problems

**Data Structures الأساسية:**

- Arrays and Strings manipulation
- Two pointers technique
- Sliding window problems
- Basic sorting and searching

**مسائل يومية مقترحة:**

- يوم 1-3: Array problems (2-3 مسائل/يوم)
    - Find maximum/minimum element
    - Rotate array
    - Remove duplicates
    - Two sum problem
- يوم 4-6: String problems (2-3 مسائل/يوم)
    - Reverse string
    - Check palindrome
    - Anagram detection
    - String compression
- يوم 7-10: Mixed easy problems (2-3 مسائل/يوم)
    - Valid parentheses
    - Merge sorted arrays
    - Missing number
    - Single number
- يوم 11-14: Pattern recognition (2-3 مسائل/يوم)
    - Frequency counting
    - Prefix sum problems
    - Simple recursion

**المنصات المقترحة:**

- LeetCode Easy problems
- HackerRank Problem Solving
- CodeForces Div 2 A problems

### المرحلة الثانية (الأسبوع 3-4): Medium Problems

**Data Structures متوسطة:**

- LinkedList operations
- Stack and Queue applications
- Hash Maps and Sets
- Basic Tree operations

**مسائل أسبوعية:**

- الأسبوع 3:
    - LinkedList: Reverse, cycle detection, intersection
    - Stack: Valid parentheses variations, next greater element
    - Queue: Sliding window maximum
    - Hash Map: Group anagrams, longest substring
- الأسبوع 4:
    - Tree: Level order traversal, validate BST
    - Binary Search: Search in rotated array
    - Dynamic Programming: Fibonacci, climbing stairs
    - Graph: BFS/DFS basics

**الهدف:** 1-2 مسائل متوسطة يومياً + مراجعة المفاهيم

### المرحلة الثالثة (الأسبوع 5-6): Advanced & Mixed

**مواضيع متقدمة:**

- Dynamic Programming patterns
- Graph algorithms (DFS, BFS, shortest path)
- Backtracking
- Advanced Tree problems

**استراتيجية الحل:**

- الأسبوع 5: Focus على DP patterns
    - 0/1 Knapsack variations
    - Longest common subsequence
    - House robber problems
    - Coin change problems
- الأسبوع 6: Mixed challenging problems
    - Graph problems: Number of islands, course schedule
    - Backtracking: N-Queens, permutations
    - Tree: Lowest common ancestor, serialize tree
    - Array: 3Sum, trapping rain water

**التقييم والمراجعة:**

- Weekly reviews لحل المسائل القديمة بطرق مختلفة
- Time complexity analysis لكل حل
- إنتاج solutions مع شرح مفصل

### استراتيجية الحل العامة:

**قبل البدء:**

1. افهم المشكلة جيداً
2. اكتب أمثلة test cases
3. فكر في الحل البسيط الأول (brute force)
4. حسن الحل تدريجياً

**أثناء الحل:**

1. اكتب الكود step by step
2. اختبر مع الأمثلة
3. احسب time & space complexity
4. فكر في edge cases

**بعد الحل:**

1. راجع solutions أخرى
2. تعلم طرق أفضل إن وجدت
3. اكتب notes للمراجعة

**مصادر إضافية:**

- "Cracking the Coding Interview" book
- AlgoExpert platform
- YouTube channels: Back to Back SWE, Abdul Bari