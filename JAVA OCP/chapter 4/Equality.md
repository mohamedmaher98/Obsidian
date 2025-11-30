```java
class Person {
    String name;
    int age;
}

Person p1 = new Person("Ali", 20);
Person p2 = new Person("Ali", 20);


System.out.println(p1.equals(p2)); // false !!
```
ليه؟  
لأن Person **ما عرّفتش equals()** بنفسك.# الحل: Override equals() و hashCode()

لو عايز تقارن محتوى object:

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;  
    if (obj == null || getClass() != obj.getClass()) return false;

    Person p = (Person) obj;
    return age == p.age &&
           Objects.equals(name, p.name);
}

```
- **== يقارن العناوين** (Reference)
    
- **equals() يقارن المحتوى** (Value)
    
- لو عامل كلاس بنفسك → لازم تعمل Override لـ equals + hashCode
    
- معظم الإنترفيو بيوقع في حتة:
    
    > "String a == String b"  
    > والسؤال ده فيه مصايب بسبب String Pool.