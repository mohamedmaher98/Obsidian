
public class App {

public static void main(String[] args) {

  

SessionFactory factory = new Configuration().configure("hibernate.cfg.xml").addAnnotatedClass(Student.class)

.buildSessionFactory();

  

Session session = factory.getCurrentSession();

  

try {

// create object of entity class

Student student = new Student();

// start session

session.beginTransaction();

// perform operation

student = session.get(Student.class,1);

// commit transaction

session.getTransaction().commit();

System.out.println(student.toString());

} finally {

session.close();

factory.close();

}

  

}

}