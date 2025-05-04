hibernate is object-relational mapping - is converting data between type system using object-oriented programing language 

--- how to add new project 
new - maven project 
add  - hibernate.cfg.xml
ده بيقي فيه اعدادات ال  session 
<hibernate-configuration>

<session-factory>

<property name="hibernate.connection.url">

jdbc:sqlserver://localhost:1433;databaseName=user;encrypt=true;trustServerCertificate=true;

</property>

/localhost:1433 - لازم تيضيف البورت ده من Sql server configuration in TCP

 <property name="hibernate.connection.username">sa</property>

<property name="hibernate.connection.password">123</property>

<property name="hibernate.dialect">org.hibernate.dialect.SQLServerDialect

الـ Dialect هو الذي يحدد الـ SQL syntax والـ features التي يدعمها محرك قاعدة البيانات،
<property name="connection.pool_size">10</property>
هي خاصية تحدد عدد **اتصالات قاعدة البيانات** التي يمكن أن يحتفظ بها Hibernate في الـ **Connection Pool** في نفس الوقت.

<property name="show_sql">true</property>

StandardServiceRegistry registry = new StandardServiceRegistryBuilder().configure().build(); 

- **`StandardServiceRegistryBuilder()`**:
    
    - يُستخدم لبناء وتكوين خدمات Hibernate المطلوبة. يقوم هذا الكائن بإعداد مختلف الخدمات التي يحتاجها Hibernate مثل الاتصال بقاعدة البيانات، إعدادات الـ Dialect، إعدادات الـ Connection Pool، وغيرها.
        
- **`configure()`**:
    
    - هذه الدالة تقوم بقراءة الإعدادات من ملف `hibernate.cfg.xml` بشكل افتراضي. يعني إنها بتبحث عن ملف الإعدادات هذا وتقرأ القيم الموجودة فيه لتكوين البيئة الخاصة بـ Hibernate.
        
- **`build()`**:
    
    - يقوم بإنشاء الـ **StandardServiceRegistry** بعد أن يتم تهيئته بالإعدادات.
      
      لـ **MetadataSources** بتخزن معلومات عن الـ Entities اللي هتتعامل معاها Hibernate. زي ما انت شايف، هنا بنمرر الـ registry علشان يستخدم الإعدادات اللي اتعملت في الخطوة اللي فاتت.
      
      الـ **SessionFactory** هي الكائن المسؤول عن إنشاء الجلسات (Sessions) في Hibernate، واللي بتساعدك في تنفيذ العمليات على الكائنات (Entities) اللي عندك.
      
      Session session = sessionFactory.openSession();
      
      session.beginTransaction();
      
      session.createNativeQuery("SELECT 1").getSingleResult()
      
      session.getTransaction().commit();
      
      
      ------
      Session Factory 
      