الJava فيها 7 مجموعات أساسية من الـ Operators. هقولّك كل مجموعة بتعمل إيه، وأمثلة بسيطة، وملاحظات مهمة كمبرمج شغال في بيئة ERP أو أنظمة كبيرة زينا.


1) **Arithmetic Operators — العمليات الحسابية**  -+*/
2) 2) **Unary Operators — عمليات على قيمة واحدة** + - ++ -- !
3) 3) **Relational (Comparison) Operators — المقارنات** == !=  >= < =  > < 
4) **Logical Operators — العمليات المنطقية** 
			`&&` و `||` بيعملوا "Short-Circuit"  
يعني ممكن اليمين ما يتنفّذش خالص لو الشمال كافي يحدد النتيجة.

5) **Bitwise Operators — عمليات على البِتّات**

|`&`|AND على البتّات|
|`|`|
|`^`|XOR|
|`~`|NOT|
|`<<`|Shift left|
|`>>`|Shift right|
|`>>>`|Shift right بدون sign|

6) **Assignment Operators — الإسناد**
= =+ =- =/ =*

7) **Ternary Operator (?:) — الشرط المختصر**
replacement للـ if السريع.
int age = 20;
String type = (age >= 18) ? "Adult" : "Child";
8) **Type Comparison Operator — instanceof**
if(obj instanceof ProductionOrder){
    // ...
}

9) **Lambda Operators — السهم (->)**
list.forEach(item -> System.out.println(item));


# 10) **Operator Precedence — أولوية التنفيذ**

العمليات مش بتتنفّذ حسب ترتيب كتابتها… ليها أولوية.
### أعلى أولوية → أدنى أولوية

1. l`()`
    
2. unary: `++`, `--`, `!`, `+`, `-`
    
3. `* / %`
    
4. `+ -`
    
5. `< > <= >=`
    
6. == !=
    
7. `&&`
    
8. `||`
    
9. `?:`
    
10. = و أخواته

ملخص 
- عندك **7 أنواع أساسية** من الـ operators.
    
- استخدم **.equals** بدل == في الـ Objects.
    
- خليك فاكر **short-circuit** في `&&` و `||`.
    
- ما تستخدمش البتّات إلا لو بتتعامل مع flags.
    
- استخدم `?:` للعمليات البسيطة، مش حاجات معقدة.
    
- الأولوية بتغيّر معنى الكود… استخدم أقواس لتوضيح النية.