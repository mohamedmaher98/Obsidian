# Class structure

in java classes are the basic blocks of the building

object : is a run time instance of a class in memory

# ==Comments==

types of comments:

1-// single line comment

2-/*

*Multiple

Comment

*/

3-javadoc comment : this is special syntax tell Javadoc to pay attention to the comment

it look like multi line comment except it starts with /**

# ==Main method==

java program start execute with it’s main() method

it’s managed by JVM (java virtual machine)
--
## ==JVM==

JVM : transfer editor code → .class file

java application —> JVM —> OS (windows , mac , Linux)

it’s 1- load the code 2- verifies the code 3-Execute the code

java language is independent : it’s mean it’s not matter it run on mac or windows or any OS

so we send the bite code file to the JVM to execute it not the CPU

## ==JRE==

---

JRE: java Runtime Environment

contains the libraries and software needed by your java programs to run

contains the JVM for the specified OS

no need to be developer to install it on your machine sometimes it install by other software

## ==JDK==

JDK : java development kit

developer need to install it

contains the libraries and software needed by developer to develop an application

contains the JRE for specified OS

---
# ==Import Package==

-name conflicts : class names don’t have to be unique across all of java
----------------------------------------------------------------------------------------------------

# ==Constructor==

Constructor : Special type of method that creates a new object

- the name of the constructor should match the name of the class
- the are no return type
- The purpose of a constructor is to initialize fields,
---
# ==Instance initializer Blocks==

- code between two {}

## order of initialization

1- fields and instance initializer blocks in the order in which they appear

2- constructor runs after all fields and instance initializer blocks have run

---
# ==_**java Data Types==**_

## 1 - primitive type

java has built in data types

|key word|type|example|
|---|---|---|
|boolean|true or false|true|
|byte|8-bit integral value|123|
|short|16 -bit|123|
|int|32- bit|123|
|long|64-bit|123|
|float|32 bit floating point value|123.45f|
|double|64 bit floating point value|123.456|
|char|16- bit|‘a’|

you can have underscores in numbers to make them easy to read

int m= 1_000_000;

---

## ==2- reference types==

- A reference type refers to an object (an instance of a class).
- Unlike primitive types that hold their values in the memory where the variable is allocated
- references do not hold the value of the object they refer to. Instead, a reference “points” to an object by storing the memory address where the object is located
- reference types can be assigned null
---
the default for the variables is

|boolean|false|
|---|---|
|byte -short- int - long|0|
|float - double|0.0|
|char|/uooo|
|all object reference|null|

---
# ==Declaring multiple variables==

string s1 , s2 ;

string s1=”yes” string s2=”no”;

The name must begin with a letter or the symbol $ or _.

Local variable —> in methods and it should be initialize

local variables are declared within a method or they are in method parameter .

instance and classes variable —> static and if not initialize it take the default variable

---
# Destroying objects

Java automatically takes care of destroying the objects that aren’t need any more

by garbage collector

all objects store in heap

Garbage collection refers to the process of automatically freeing memory on the heap by deleting objects that are no longer reachable in your program.

An object is no longer reachable when one of two situations occurs:

- The object no longer has any references pointing to it.
- All references to the object have gone out of scope.

## finalize()

This method gets called if the garbage collector tries to collect the object.

it might not get called and that it definitely won’t be called twice.

finalize() is only run when the object is eligible for garbage collection

---

# Benefits of Java

1. object oriented :

which means all code is defined in classes and most of those classes can be instantiated into objects

1. Encapsulation: java

supports access modifiers to protect data from unintended access and modification

1. Platform Independent :

“write once, run everywhere.”

1. Robust :

One of the major advantages of Java over C++ is that it prevents memory leaks. Java manages memory on its own and does garbage collection automatically

1. simple:

it’s more simple than c++

1. secure:

Java code runs inside the JVM. This creates a sandbox that makes it hard for Java code to do evil things to the computer it is running on.

---
