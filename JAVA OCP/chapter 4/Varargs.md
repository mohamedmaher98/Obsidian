# Using Methods with Varargs – Java Notes

## ✔️ What are Varargs?
Varargs = *Variable Arguments*  
Allows a method to accept **zero, one, or many** arguments of the same type.

```java
void printNames(String... names);
```

---

## ✔️ How it works?
Varargs are compiled as an **array internally**.

Example:
```java
printNames("Ali", "Omar", "Mai");
```
Inside the method:
```
names = ["Ali", "Omar", "Mai"]
```

---

## ✔️ Example
```java
public static void printNumbers(int... nums) {
    for (int n : nums) {
        System.out.println(n);
    }
}
```

Usage:
```java
printNumbers();
printNumbers(5);
printNumbers(1, 2, 3, 4);
```

---

## ✔️ Rules for Varargs
### 1. Must be the **last parameter**
```java
void test(int x, String... names); // OK
```

### 2. Only **one *varargs*** is allowed
```java
void test(String... a, int... b); // ❌ Not allowed
```

---

## ✔️ Why use Varargs?
Replaces multiple overloaded methods.

Instead of:
```java
void sum(int a);
void sum(int a, int b);
void sum(int a, int b, int c);
```

Use:
```java
void sum(int... nums);
```

---

## ✔️ Key Idea
Varargs = syntactic sugar for arrays  
Compiler automatically creates the array for you.
