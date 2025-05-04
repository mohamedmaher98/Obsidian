
public class App {

public static void main(String[] args) {

  

SessionFactory factory = new Configuration().configure("hibernate.cfg.xml").addAnnotatedClass(Student.class)

.buildSessionFactory();

  

Session session = factory.getCurrentSession();

  

try {

// create object of entity class

Student student = new Student("maher", "66203773", "mohamed", "farghaly");

// start session

session.beginTransaction();

// perform operation

session.save(student);

// commit transaction

session.getTransaction().commit();

} finally {

session.close();

factory.close();

}

  

}

}