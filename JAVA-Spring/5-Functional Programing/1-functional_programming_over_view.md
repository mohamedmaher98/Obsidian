# Functional Programming in Java --- Study Notes

## 1. Functional Programming

Functional Programming is a programming style where **functions are
treated like data**.

This means we can: - Store functions in variables - Pass functions as
parameters - Return functions from methods

Example:

``` java
Function<Integer, Integer> f = x -> x * 2;
```

------------------------------------------------------------------------

# 2. Functional Interface

A Functional Interface is: 

> An interface that contains **only one abstract method**.

Example:

``` java
interface Operation {
    int apply(int a, int b);
}
```

Functional interfaces allow us to use **Lambda Expressions**.

------------------------------------------------------------------------

# 3. Lambda Expression

A Lambda is a short way to implement a method of a functional interface.

Example:

``` java
(a, b) -> a + b
```

Instead of writing a full class implementation.

Example:

``` java
Operation add = (a, b) -> a + b;
```

------------------------------------------------------------------------

# 4. Important Functional Interfaces in Java

## Function\<T, R\>

Represents a function that:

-   takes an input of type T
-   returns a result of type R

Example:

``` java
Function<Integer, Integer> square = x -> x * x;
```

------------------------------------------------------------------------

## Predicate`<T>`{=html}

Represents a function that:

-   takes an input
-   returns **true or false**

Example:

``` java
Predicate<Integer> isEven = x -> x % 2 == 0;
```

------------------------------------------------------------------------

## Consumer`<T>`{=html}

Represents a function that:

-   takes an input
-   performs an action
-   returns nothing (void)

Example:

``` java
Consumer<String> printer = s -> System.out.println(s);
```

------------------------------------------------------------------------

# 5. Main Concepts of Functional Programming

## First-Class Functions

Functions can be treated like data.

You can: - store them in variables - pass them as parameters - return
them from methods

Example:

``` java
Function<Integer,Integer> f = x -> x * 2;
```

------------------------------------------------------------------------

## Pure Functions

A pure function:

1.  Depends only on its input
2.  Has no side effects

Example:

``` java
x -> x * x
```

Calling it multiple times with the same input always produces the same
result.

------------------------------------------------------------------------

## Immutability

Immutability means:

> Data should not change after it is created.

Example:

Instead of modifying:

``` java
x = x + 5;
```

We create a new value:

``` java
int y = x + 5;
```

------------------------------------------------------------------------

## Recursion

Recursion is when a function calls itself.

Example: factorial

``` java
int factorial(int n){
    if(n == 1)
        return 1;
    return n * factorial(n-1);
}
```

Example:

    factorial(4)
    = 4 * factorial(3)
    = 3 * factorial(2)
    = 2 * factorial(1)

------------------------------------------------------------------------

## Referential Transparency

A function is referentially transparent if:

> You can replace the function call with its result without changing the
> program behavior.

Example:

    square(5) = 25

You can replace:

    square(5)

with

    25

------------------------------------------------------------------------

## Lazy Evaluation

Lazy evaluation means:

> Code executes **only when the result is needed**.

Example using Streams:

``` java
list.stream()
    .filter(x -> x > 5)
    .map(x -> x * 2)
```

Nothing executes until a **terminal operation** is added:

``` java
.collect(Collectors.toList())
```

------------------------------------------------------------------------

# 6. Imperative vs Declarative Programming

## Imperative Programming

You describe **how to perform the task step by step**.

Example:

``` java
for(int x : list){
    System.out.println(x);
}
```

------------------------------------------------------------------------

## Declarative Programming

You describe **what you want**, not how to do it.

Example:

``` java
list.forEach(System.out::println);
```

------------------------------------------------------------------------

# 7. Function vs Method

## Function

A general programming concept:

-   takes input
-   returns output

Example:

    f(x) = x * 2

------------------------------------------------------------------------

## Method

A function that exists **inside a class**.

Example:

``` java
class Calculator{

    int square(int x){
        return x * x;
    }

}
```

In Java:

> All functions are methods because they must exist inside classes.

------------------------------------------------------------------------

# Summary

Today we studied:

-   Functional Programming
-   Functional Interfaces
-   Lambda Expressions
-   Function / Predicate / Consumer
-   First-Class Functions
-   Pure Functions
-   Immutability
-   Recursion
-   Referential Transparency
-   Lazy Evaluation
-   Imperative vs Declarative Programming
-   Function vs Method

These are the **core foundations of Functional Programming in Java**.
