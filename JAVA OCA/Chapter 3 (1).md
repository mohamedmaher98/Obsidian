### ==String== 

reference types are created using the new keyword
String name = "Fluffy";
String name = new String("Fluffy");
but String class is special and doesn’t need to be instantiated with new.

---
### ==Concatenation==

System.out.println(1 + 2); // 3
System.out.println("a" + "b"); // ab
System.out.println("a" + "b" + 3); // ab3
System.out.println(1 + 2 + "c"); // 3c

---
### ==Immutability==
- [ ] -Once a String object is created, it is not allowed to change. It cannot be made larger or
	smaller, and you cannot change one of the characters inside it.
- [ ] -Mutable is another word for changeable
- [ ] can’t be changed once it’s created.
- [ ] Immutable only has a getter. There's no way to change the value of s once it's set.
- [ ] You can even make the instance variable final so the compiler reminds
	you if you accidentally change s. 
- [ ] immutable classes in Java are final
---
### ==another way to concatenation in java==
String s1 = "1";
String s2 = s1.concat("2");

---
### ==The String Pool==

- [ ] Since strings are everywhere in Java, they use up a lot of memory.
- [ ] string pool, also known as the intern pool, is a location in the Java virtual machine (JVM)
	that collects all these strings.
- [ ] Remember back when we said these two lines are subtly different?
	String name = "Fluffy";
	String name = new String("Fluffy");
- [ ] The former says to use the string pool normally. The second says “No, JVM. I really
	don’t want you to use the string pool. Please create a new object for me even though it is
	less efficient.” When you write programs, you wouldn’t want to do this. For the exam, you
	need to know that it is allowed.
---
### ==Important Methods For String== 
- [ ] length 
- [ ] charAt(index)
- [ ] indexOf(char ch)
- [ ] subString()
- [ ] toLowerCase () and toUpperCase
- [ ] equals() and equalsIgnoreCase()
- [ ] startsWith() and endsWith()
- [ ] contains()
- [ ] replace() --> 
	-System.out.println("abcabc".replace('a', 'A')); // AbcAbc
    -System.out.println("abcabc".replace("a", "A")); // AbcAbc
- [ ] trim()
---
### ==Using the StringBuilder Class==
- `StringBuilder` هو **كائن قابل للتغيير (mutable)**.
    
- يستخدم لإنشاء نصوص (Strings) يمكن تعديلها بدون إنشاء كائن جديد كل مرة.
    
- **مش متزامن (Not synchronized)** ➔ يعني مش آمن في حالة تعدد الخيوط (multi-threading).
    
- عشان مش متزامن، فهو أسرع من `StringBuffer` لما تكون شغال في برنامج عادي مش فيه تعدد خيوط.
    

### أهم مميزاته:

- سريع جدًا مقارنة بالـ `String` أو `StringBuffer` لما بتشتغل في بيئة خيط واحد (single-thread).
    
- يسمحلك تضيف (`append`)، تحذف (`delete`)، تدخل (`insert`)، تعكس (`reverse`) النص بسهولة.
---
### - ==Mutability and Chaining==

- **Mutability** ➔ الكائن كأنه سبورة، تقدر تمسح وتكتب فيها برحتك.
    
- **Chaining** ➔ كأنك بتدي أوامر متتالية لشخص من غير ما توقفه كل شوية.
---
### ==Size vs. Capacity==

- **Size** ➔ هو **اللي فعليًا متخزن** أو متسجل جوا الكائن.
    
- **Capacity** ➔ هو **المساحة الفاضية كمان** اللي الكائن محجزها عشان لما تضيف بيانات تانية، ميضطرش يكبر المساحة كل شوية.
---
StringBuilder one = new StringBuilder();
StringBuilder two = new StringBuilder();
StringBuilder three = one.append("a");
System.out.println(one == two); // false
System.out.println(one == three); // true

---
### ==Java Arrays==
An array is an area of memory on the heap with space for a designated number of elements
array can be of any other Java type.
array can be of any other Java type.an array is an ordered list. It can contain duplicates
int[] numbers1 = new int[3];
![[Pasted image 20250429180334.png]]
Another way to create an array is to specify all the elements it should start out with:
int[] numbers2 = new int[] {42, 55, 99};
int[] numbers2 = {42, 55, 99};
This approach is called an anonymous array. It is anonymous because you don’t specify
the type and size.
Finally, you can type the [] before or after the name
int[] numAnimals;
int [] numAnimals2;
int numAnimals3[];
int numAnimals4 [];

---
Creating an Array with Reference Variables
You can choose any Java type to be the type of the array. This includes classes you create
yourself. Let’s take a look at a built-in type with String:
public class ArrayType {
public static void main(String args[]) {
String [] bugs = { "cricket", "beetle", "ladybug" };
String [] alias = bugs;
System.out.println(bugs.equals(alias)); // true
System.out.println(bugs.toString()); // [Ljava.lang.String;@160bc7c0
} }

4: String[] mammals = {"monkey", "chimp", "donkey"};
5: System.out.println(mammals.length); // 3
6: System.out.println(mammals[0]); // monkey
7: System.out.println(mammals[1]); // chimp
8: System.out.println(mammals[2]); // donkey

---
### <mark style="background: #BBFABBA6;">Searching</mark>

Target element found in sorted array-->Index of match

Target element not found in sorted array --> Negative value showing one smaller than the
negative of index, where a match needs to be inserted to preserve sorted order

Unsorted array --> A surprise—this result isn’t predictable
3: int[] numbers = {2,4,6,8};
4: System.out.println(Arrays.binarySearch(numbers, 2)); // 0
5: System.out.println(Arrays.binarySearch(numbers, 4)); // 1
6: System.out.println(Arrays.binarySearch(numbers, 1)); // -1
7: System.out.println(Arrays.binarySearch(numbers, 3)); // -2
8: System.out.println(Arrays.binarySearch(numbers, 9)); // -5

### <mark style="background: #BBFABBA6;">Sorting</mark>

Java makes it easy to sort an array by providing a sort method
int[] numbers = { 6, 9, 1 };
Arrays.sort(numbers);
1  6  9

---
### <mark style="background: #BBFABBA6;">Vargas </mark>

كلمة **Varargs** هي اختصار لـ "**Variable Arguments**"، يعني إنك تقدر تمرر **عدد غير محدد من القيم** كـ **arguments** لدالة في Java.
بدل ما تعمل overload أو تبعت مصفوفة، تقدر تستخدم Varargs.

شكلها
public void printNames(String... names) {
    for (String name : names) {
        System.out.println(name);
    }
}

printNames("Ahmed", "Ali", "Sara");
// أو حتى ممكن تبعتها فاضية
printNames();
**لازم تكون آخر باراميتر** في الدالة:
public void example(int id, String... names) {} // صح
public void example(String... names, int id) {}  // غلط

public static void main(String[] args)
public static void main(String args[])
public static void main(String... args) // varargs

---
### <mark style="background: #BBFABBA6;">Multidimensional Arrays</mark>
![[Pasted image 20250430120042.png]]

---
### <mark style="background: #BBFABBA6;">Array List</mark>

هي class في Java بتسمحلك تخزن مجموعة من العناصر زي المصفوفة، لكن بمرونة أكبر:
![[Pasted image 20250430121332.png]]

ArrayList<String> names = new ArrayList<>(); 


ArrayList implements an interface called List.
just know that you can store an ArrayList in a List reference variable but not vice
versa.
The reason is that List is an interface and interfaces can’t be instantiated.
List<String> list6 = new ArrayList<>();
ArrayList<String> list7 = new List<>(); // DOES NOT COMPILE

-- add

ArrayList list = new ArrayList();
list.add("hawk"); // [hawk]
list.add(Boolean.TRUE); // [hawk, true]
System.out.println(list); // [hawk, true]
This is okay because we didn’t
specify a type for ArrayList; therefore, the type is Object, which includes everything
except primitives.

4: List<String> birds = new ArrayList<>();
5: birds.add("hawk"); // [hawk]
6: birds.add(1, "robin"); // [hawk, robin]
7: birds.add(0, "blue jay"); // [blue jay, hawk, robin]
8: birds.add(1, "cardinal"); // [blue jay, cardinal, hawk, robin]
9: System.out.println(birds); // [blue jay, cardinal, hawk, robin]

--move()
The remove() methods remove the fi rst matching value in the ArrayList or remove the
element at a specifi ed index.

3: List<String> birds = new ArrayList<>();
4: birds.add("hawk"); // [hawk]
5: birds.add("hawk"); // [hawk, hawk]
6: System.out.println(birds.remove("cardinal")); // prints false
7: System.out.println(birds.remove("hawk")); // prints true
System.out.println(birds.remove(0)); // prints hawk
9: System.out.println(birds); // []

--Set()

The set() method changes one of the elements of the ArrayList without changing the size.

15: List<String> birds = new ArrayList<>();
16: birds.add("hawk"); // [hawk]
17: System.out.println(birds.size()); // 1
18: birds.set(0, "robin"); // [robin]
19: System.out.println(birds.size()); // 1
20: birds.set(1, "robin"); // IndexOutOfBoundsException

--isEmpty and Size()
The isEmpty() and size() methods look at how many of the slots are in System.out.println(birds.isEmpty()); // true
System.out.println(birds.size()); // 0
birds.add("hawk"); // [hawk]
birds.add("hawk"); // [hawk, hawk]
System.out.println(birds.isEmpty()); // false
System.out.println(birds.size()); // 2

--clear()

The clear() method provides an easy way to discard all elements of the ArrayList

List<String> birds = new ArrayList<>();
birds.add("hawk"); // [hawk]
birds.add("hawk"); // [hawk, hawk]
System.out.println(birds.isEmpty()); // false
System.out.println(birds.size()); // 2
birds.clear(); // []
System.out.println(birds.isEmpty()); // true
System.out.println(birds.size()); // 0

--contains()

The contains() method checks whether a certain value is in the ArrayList.

List<String> birds = new ArrayList<>();
birds.add("hawk"); // [hawk]
System.out.println(birds.contains("hawk")); // true
System.out.println(birds.contains("robin")); // false


--equals()

Finally, ArrayList has a custom implementation of equals() so you can compare two lists
to see if they contain the same elements in the same order

List<String> one = new ArrayList<>();
32: List<String> two = new ArrayList<>();
33: System.out.println(one.equals(two)); // true
34: one.add("a"); // [a]
35: System.out.println(one.equals(two)); // false
36: two.add("a"); // [a]
37: System.out.println(one.equals(two)); // true
38: one.add("b"); // [a,b]
39: two.add(0, "b"); // [b,a]
40: System.out.println(one.equals(two)); // false

 ----------
ArrayList مش **Thread Safe** (يعني مش آمنة في بيئة multi-thread).
- `ArrayList` بيحتفظ بالترتيب اللي ضفت بيه العناصر.

 من Array لـ ArrayList:
 String[] arr = {"A", "B", "C"};
ArrayList<String> list = new ArrayList<>(Arrays.asList(arr));
العكس 
String[] array = list.toArray(new String[0]);