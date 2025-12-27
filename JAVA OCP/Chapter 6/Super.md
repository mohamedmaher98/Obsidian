لما تعمل `new` لابن (subclass)، Java لازم تجهّز “جزء الأب” الأول وبعدين “جزء الابن”، عشان كده أول سطر في أي constructor لازم يكون يا إمّا `this(...)` (ينادي constructor تاني في نفس الكلاس) يا إمّا `super(...)` (ينادي constructor في الأب).  
ولو إنت ماكتبتش `super(...)` ولا `this(...)`، الـ compiler غالبًا بيحط `super()` تلقائي كأول سطر (يعني ينادي no-arg constructor بتاع الأب) بشرط إن الأب فعلًا عنده no-arg constructor.  
وعشان كده لو الأب مفيهوش no-arg constructor وانت ماحددتش `super(args)` صح، الكود يقع compile error لأن الـ compiler حاول يحط `super()` وملاقاهوش.

## Concept breakdown (what, why, how)

- What (إيه اللي بيحصل):
    
    - `الsuper(...)` = “ابدأ بتهيئة الأب” عن طريق استدعاء constructor من الـ direct parent.
        
    - ال`this(...)` = “فوض التهيئة” لconstructor تاني جوه نفس الكلاس، وفي الآخر السلسلة لازم تنتهي بـ `super(...)` (مباشرة أو ضمنيًا).
        
- Why (ليه لازم):
    
    - لأن الـ object الواحد فيه جزء موروث من الأب (fields + invariants) ولازم يتعمله initialization قبل ما الابن يبني عليه.
        
    - الJava بتفرض إن استدعاء constructors يبقى “constructor chain” واضح ومنضبط عشان تمنع حالة إنك تلمس `this` قبل ما الأب يتجهّز.
        
- How (إزاي تكتبها صح):
    
    - ال`super(...)` أو `this(...)` لازم يكونوا أول statement في الـ constructor، وممنوع الاتنين مع بعض في نفس الـ constructor.
        
    - ال`super()` دايمًا بيروح لأقرب parent (direct superclass) مش للـ grandparent.
        

## ERP-based Java examples (realistic, production-like)

تخيل base class لكل مستندات الـ ERP اسمه `ErpDocument` فيه حاجات مشتركة زي companyCode و createdAt:

java

`public class ErpDocument {     private final String companyCode;    private final long createdAtEpochMillis;     public ErpDocument(String companyCode) {        if (companyCode == null || companyCode.isBlank())            throw new IllegalArgumentException("companyCode is required");        this.companyCode = companyCode;        this.createdAtEpochMillis = System.currentTimeMillis();    } }`

وابن منه `SalesInvoice` لازم يضمن إنه يمرر companyCode للأب:

java

`public class SalesInvoice extends ErpDocument {     private final String invoiceNo;     public SalesInvoice(String companyCode, String invoiceNo) {        super(companyCode);        if (invoiceNo == null || invoiceNo.isBlank())            throw new IllegalArgumentException("invoiceNo is required");        this.invoiceNo = invoiceNo;    }     public SalesInvoice(String companyCode) {        this(companyCode, InvoiceNumberGenerator.next());    } }`

هنا `this(companyCode, ...)` بيقلل تكرار، وفي الآخر بيبقى فيه `super(companyCode)` فعليًا كجزء من السلسلة.

نفس الفكرة مفيدة في payroll: `EmployeeTransaction` كـ base، و`SalaryPayment` كـ child لازم يضمن إن رقم الموظف/الشركة اتعمله validation في الأب قبل أي منطق في الابن.

## Common mistakes in enterprise systems

- نسيان إن الأب مفيهوش no-arg constructor:
    
    - تكتب `class Seal extends Mammal {}` فتفتكر الدنيا تمام، لكن الـ compiler يولّد `Seal()` وفيه `super()` ضمنيًا، ولما `Mammal` مفيهوش `Mammal()` الكود يفشل.
        
- محاولة تحط log قبل `super(...)`:
    
    - زي `System.out.println(...)` قبل `super(...)` → غلط لأن `super/this` لازم أول statement.
        
- خلط `super` و `super()`:
    
    - `super.field` ده access لmember في الأب، إنما `super(...)` ده constructor call، ودي قواعدها مختلفة تمامًا.
        
- نداء `super()` أكتر من مرة أو بعد أول سطر:
    
    - ممنوع—مرة واحدة فقط وبداية الـ constructor.
        
- تصميم inheritance غلط في domain:
    
    - ساعات بنعمل inheritance بين documents لمجرد reuse، ثم نكتشف إن constructors بتبقى معقدة جدًا وبتفرض “تهيئة” مش منطقية على الابن.
        

## Best practices for large codebases

- خلي الأب يحمي invariants الأساسية:
    
    - أي data مشتركة وحاسمة (companyCode, tenantId, createdAt) تتعمل في parent constructor عشان أي child يطلع سليم.
        
- استخدم `this(...)` لتوحيد مسارات الإنشاء في الابن:
    
    - خلي عندك constructor “أساسي” واحد في الابن يستقبل كل المطلوب، والباقي convenience constructors بتفوض له.
        
- لو محتاج مرونة أكبر من constructors:
    
    - استخدم static factories أو builders بدل inheritance/constructors متشابكة، خصوصًا في ERP entities المعقدة.
        
- راعي الـ accessibility:
    
    - لازم الـ child يقدر يوصل للـ parent constructor (مش `private`) وإلا مش هتعرف تبني child بشكل طبيعي.
        

## Practical questions to test understanding

- لو parent عنده بس `Parent(int x)` ومفيش `Parent()`: إيه اللي يحصل لو كتبت `class Child extends Parent { public Child() {} }`؟
    
- ليه Java بتجبر `super(...)` يبقى أول statement؟
    
- إيه الفرق بين `super.member` و `super(...)`؟
    
- هل ينفع `this(...)` وبعده `super(...)` في نفس الـ constructor؟
    
- في ERP: إمتى inheritance للـ documents يبقى مفيد، وإمتى يبقى عبء؟
    

## Answers to those questions

- الـ compiler هيحط `super()` ضمنيًا في `Child()`، ولأن `Parent()` مش موجود، يحصل compile-time error ولازم تكتب `super(someInt)` بنفسك.
    
- لأن ده بيضمن إن تهيئة الـ superclass تتم قبل أي code يلمس الـ object، وبيفرض constructor chaining منضبط.
    
- `super.member` للوصول لعضو في الأب، إنما `super(...)` لاستدعاء constructor بتاع الأب.
    
- لا، ممنوع؛ أول statement لازم يكون يا `this(...)` يا `super(...)` فقط.
    
- مفيد لما فعلاً فيه “is-a” واضح وبيشتركوا في invariants قوية (زي `ErpDocument` كأساس لكل المستندات)، وعبء لما يكون الهدف reuse بس ويبدأ يطلع constructors معقدة ومش طبيعية.
    

## Markdown summary (Obsidian)

- أول statement في أي constructor لازم يكون `this(...)` أو `super(...)` (explicit constructor invocation).
    
- لو مفيش `this(...)` ولا `super(...)` مكتوبين، الـ compiler بيضيف `super()` تلقائيًا كأول statement.
    
- لو الأب مفيهوش no-arg constructor، لازم الابن يكتب `super(args)` صراحة وإلا compilation يفشل.
    
- ال`super(...)` بيروح للـ direct parent فقط (مش لأي ancestor أقدم).
    
- `الsuper` (بدون أقواس) غير `super(...)`: الأول للوصول لأعضاء الأب، والتاني لاستدعاء constructor الأب.