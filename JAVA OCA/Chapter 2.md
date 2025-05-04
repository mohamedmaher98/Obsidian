
### **==order of operator precedence==**

1- Post-unary operators   expression ++  , expression- -

2- pre unary operators    ++expression ,   - - expression

# ==Numeric promotion Rules==

if two values have different data types java automatically promote one of the values to the larger of the two data types

if one is floating and two is integer java will automatically promote the integral value to the floating value

smaller data types like byte ,short and char promoted to int

short x =10 ;

short y = 10 ;

promoted to int

double x =49.5;

float y = 5.88f;

promoted to double;

int x =10 ;

long y =5;

promoted to long;

short x = 14 ;

float y = 13;

double z =30;

what is the result of x_y_z;

in this case short to int and int , float promoted to float , float , double promoted to double

---
Here are some tips to help remember this table:
■ AND is only true if both operands are true. x && y
■ Inclusive OR is only false if both operands are false. x || y
■ Exclusive OR is only true if the operands are different. x^y

---
Single Ampersand (`&`)

boolean a = false;
boolean b = true;

if (a & b) { // This will evaluate both 'a' and 'b'
System.out.println("Both are true"); }

|Operator|Usage|Short-circuiting|Example Behavior|
|---|---|---|---|
|`&`|Bitwise AND or Logical AND|No|Always evaluates both operands|
|`&&`|Logical AND|Yes (short-circuit)|Stops evaluating if the first operand is false|
