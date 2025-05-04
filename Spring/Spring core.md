@autowird 
مهمتها انها تعمل object تلقائي بدل ماتعمله بإيدك
فايدة @Autowired في Spring ببساطة هي إنك تسيب مسؤولية إنشاء وربط الكائنات (Objects) لـ Spring بدل ما تعملها manually.

Car car = new Car();
Travel travel = new Travel(car);

@Autowired
private Car car;
تستخدمها علشان Spring يـ **يحقن dependency** بشكل تلقائي في الكلاسات اللي محتاجاها.

ex : 
Interface
public interface Engine {
    void start();
}

import org.springframework.stereotype.Component;

@Component
public class PetrolEngine implements Engine {
    public void start() {
        System.out.println("Petrol engine started.");
    }
}

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Car {

    @Autowired
    private Engine engine;

    public void drive() {
        engine.start();
        System.out.println("Car is driving...");
    }
}

Main :

@Configuration
@ComponentScan("com.maher")
public class App {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(App.class);

        Car car = context.getBean(Car.class);
        car.drive();
    }
}

لو فيه أكتر من Bean من نفس النوع → يحصل **conflict**  
➜ لازم تستخدم `@Qualifier`


