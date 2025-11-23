# شرح أنماط إنشاء المعاملات المحاسبية في نظام Nama ERP

## إيه الموضوع ده؟

النظام بيحتاج يعمل قيود محاسبية (من المدين والدائن) لكل العمليات اللي بتحصل زي البيع والشراء والمخازن وكده.

فيه 4 طرق مختلفة عشان نعمل القيود دي، كل واحدة ليها استخدامها.

---

## الطرق الأربعة (بالترتيب من الأفضل للأسوأ):

### 1. الطريقة المفضلة: Utility-Based Pattern 🌟

**متى نستخدمها:** في كل التطوير الجديد (ده الأول اختيار دايماً)

**ليه هي الأفضل؟**

- كود أقل وأبسط
- أخطاء أقل
- سهلة الصيانة
- موحدة في كل النظام

**الخطوات:**

1. تخلي المستند يورث من الواجهات المطلوبة
2. تعمل الـ methods الأساسية (applyEffects, updateEffects, cancelEffects)
3. تستخدم الـ utility اللي جاهز بدل ما تكتب كود كتير

**مثال بسيط:**

```java
// الطريقة السهلة والمختصرة
@Override
public Result applyEffects() {
    Result result = Result.createAccumulatingResult();
    LedgerTransReqCreator.applyEffects(this, result, createLines(), 
        getTermConfig().getDebit(), getTermConfig().getCredit());
    return result;
}
```

---

### 2. الطريقة المبنية على التكلفة: Cost-Based Pattern 📦

**متى نستخدمها:** للمخازن والعمليات اللي ليها حسابات تكلفة

**كيف بتشتغل؟**

1. المستند يعمل حركة مخزون
2. النظام يحسب التكلفة تلقائياً
3. يحول حسابات التكلفة لقيود محاسبية

**مثال:** نقل البضاعة من مخزن لمخزن، النظام يحسب تكلفة النقل ويعمل القيد تلقائي

---

### 3. الطريقة المبنية على الفواتير: Invoice-Based Pattern 📄

**متى نستخدمها:** للفواتير المعقدة اللي فيها ضرايب وخصومات ومصاريف

**مميزاتها:**

- تتعامل مع الضرايب المعقدة
- تحسب الخصومات المتعددة
- تدعم طرق الدفع المختلفة
- تتعامل مع العملات المتعددة

**مثال:** فاتورة بيع فيها 15 صنف، عليها ضريبة 14%، وعليها خصم 5%، ومدفوعة نص كاش ونص شيك

---

### 4. الطريقة القديمة: Direct Implementation ⚠️

**متى نستخدمها:** **مانستخدمهاش خالص في التطوير الجديد!**

**ليه مش كويسة؟**

- كود كتير ومكرر
- أخطاء كتير
- صعبة الصيانة
- مش موحدة

---

## إزاي أختار الطريقة المناسبة؟

### لو عندك مستند جديد:

1. **هل هو بسيط؟** → استخدم Utility-Based Pattern
2. **هل ده متعلق بالمخازن؟** → استخدم Cost-Based Pattern
3. **هل ده فاتورة معقدة؟** → استخدم Invoice-Based Pattern

### إعداد المستند عشان يشتغل مع القيود:

#### 1. في ملف الـ DSL:

```java
@NaMaEntity(classType = ClassType.Common)
public class MyDocument extends BasicDocument {
    @NaMaField(system = true)
    UniqueIDDF ledgerTransReqId;  // ده مهم عشان يحفظ رقم القيد
}
```

#### 2. في كود المستند:

```java
public class MyDocument extends GeneratedMyDocument 
        implements HasLedgerTransReqId, IGeneratesAccountingRequest {
    
    // الـ methods المطلوبة
    @Override
    public Result applyEffects() { /* كود إنشاء القيد */ }
    
    @Override
    public Result updateEffects() { /* كود تعديل القيد */ }
    
    @Override
    public Result cancelEffects() { /* كود إلغاء القيد */ }
}
```

#### 3. تسجيل المستند في النظام:

```java
// في ملف EntitiesWithLedgerEffectsOrWithParents.java
public static String[] entitiesWithLedgerEffects = {
    "MyDocument"  // نضيف اسم المستند هنا
};
```

---

## إيه اللي بيحصل لما المستخدم يغير الإعدادات المحاسبية؟

النظام عنده طريقة ذكية عشان يعيد توليد كل القيود القديمة بالإعدادات الجديدة:

1. **المستخدم يغير إعدادات الحسابات**
2. **النظام يشغل عملية إعادة التوليد**
3. **كل القيود القديمة تتمحي وتتعمل من جديد بالإعدادات الجديدة**

---

## نصايح مهمة:

### ✅ اعمل كده:

- استخدم Utility-Based Pattern دايماً للمستندات الجديدة
- اختبر المستند كويس قبل ما تنشره
- اتأكد إن الحسابات المحاسبية صح

### ❌ ماتعملش كده:

- ماتستخدمش Direct Implementation للمستندات الجديدة
- ماتنساش تسجل المستند في قائمة الكيانات
- ماتتسرعش في الاختبار

---

## ملخص سريع:

1. **للمستندات الجديدة البسيطة:** Utility-Based Pattern
2. **للمخازن والتكلفة:** Cost-Based Pattern
3. **للفواتير المعقدة:** Invoice-Based Pattern
4. **تجنب الطريقة القديمة تماماً**

النظام مصمم يخليك تركز على منطق العمل بتاعك، وهو يتولى إنشاء القيود المحاسبية بطريقة صحيحة وآمنة.

---
# خطوات إضافة التأثير المحاسبي لمستند ipinvestmentStart

## الخطوة الأولى: تحديث ملف الـ DSL 📝

**الملف:** `modules/investment/investmententities/src/main/resources/entities/ipinvestmentStart.dsl`

```java
@NaMaEntity(classType = ClassType.Common)
public class IpinvestmentStart extends BasicDocument {
    
    // إضافة الحقل المطلوب للقيود المحاسبية (لو مش موجود)
    @NaMaField(system = true)
    UniqueIDDF ledgerTransReqId;
    
    // باقي حقول المستند...
    // المبلغ المستثمر
    DecimalDF investmentAmount;
    
    // نوع الاستثمار
    StringDF investmentType;
    
    // المستثمر
    UniqueIDDF investorId;
    
    // الخ...
}
```

---

## الخطوة التانية: تحديث كلاس المستند 🏗️

**الملف:** `modules/investment/investmentdomain/src/main/java/com/namasoft/modules/investment/domain/entities/IpinvestmentStart.java`

```java
public class IpinvestmentStart extends GeneratedIpinvestmentStart 
        implements HasLedgerTransReqId, IGeneratesAccountingRequest {

    // 1. إضافة الـ methods الأساسية للتأثير المحاسبي
    @Override
    public Result applyEffects() {
        Result result = Result.createAccumulatingResult();
        
        // التحقق من وجود إعدادات محاسبية
        IpinvestmentStartTermConfig termConfig = fetchTermConf();
        if (termConfig != null && termConfig.hasAccountingEffects()) {
            LedgerTransReqCreator.applyEffects(this, result, createLines(), 
                termConfig.getDebitAccount(), termConfig.getCreditAccount());
        }
        
        return result;
    }

    @Override
    public Result updateEffects() {
        Result result = Result.createAccumulatingResult();
        
        IpinvestmentStartTermConfig termConfig = fetchTermConf();
        if (termConfig != null && termConfig.hasAccountingEffects()) {
            LedgerTransReqCreator.updateEffects(this, result, createLines(), 
                termConfig.getDebitAccount(), termConfig.getCreditAccount());
        }
        
        return result;
    }

    @Override
    public Result cancelEffects() {
        Result result = Result.createAccumulatingResult();
        LedgerTransReqCreator.cancelEffects(this, result);
        return result;
    }

    @Override
    public void genAccEffect(Result result) {
        // للـ regeneration (إعادة التوليد)
        if (BooleanDF.isFalse(getCommitedBefore()))
            LedgerTransReqCreator.cancelEffects(this, result);
        else {
            IpinvestmentStartTermConfig termConfig = fetchTermConf();
            if (termConfig != null && termConfig.hasAccountingEffects()) {
                LedgerTransReqCreator.updateEffects(this, result, createLines(), 
                    termConfig.getDebitAccount(), termConfig.getCreditAccount());
            }
        }
    }

    // 2. إنشاء خطوط القيد المحاسبي
    private List<? extends AnyToLedgerReqLineConverter> createLines() {
        List<AnyToLedgerReqLineConverter> lines = new ArrayList<>();
        
        // إنشاء خط واحد للاستثمار (يمكن تعديله حسب الحاجة)
        lines.add(new AnyToLedgerReqLineConverter.AbstractAnyToLedgerReqLineConverter() {
            @Override
            public Object line() { 
                return IpinvestmentStart.this; // المستند نفسه
            }
            
            @Override
            public DocumentFile<?> doc() { 
                return IpinvestmentStart.this; 
            }
            
            @Override
            public Currency currency() { 
                // العملة الأساسية للشركة
                return getLegalEntity().getLedger().getMainCurrency(); 
            }
            
            @Override
            public DecimalDF value() { 
                // مبلغ الاستثمار
                return getInvestmentAmount(); 
            }
            
            @Override
            public HasSubsidiaryAccounts sourceCustomer() { 
                // المستثمر (لو موجود)
                return findInvestor(); 
            }
        });
        
        return lines;
    }

    // 3. البحث عن المستثمر (لو محتاج)
    private HasSubsidiaryAccounts findInvestor() {
        if (getInvestorId() != null) {
            // البحث عن المستثمر في قاعدة البيانات
            // return InvestorRepository.findById(getInvestorId());
        }
        return null;
    }

    // 4. الحصول على إعدادات المستند
    private IpinvestmentStartTermConfig fetchTermConf() {
        if (getTerm() != null && getTerm().getTermConfig() != null) {
            return (IpinvestmentStartTermConfig) getTerm().getTermConfig();
        }
        return null;
    }
}
```

---

## الخطوة التالتة: إنشاء إعدادات المحاسبة 🎯

**الملف:** `modules/investment/investmentdomain/src/main/java/com/namasoft/modules/investment/domain/termconfigs/IpinvestmentStartTermConfig.java`

```java
@DocumentTermConfig(documentTypes = InvestmentEntities.IpinvestmentStart)
public class IpinvestmentStartTermConfig extends GeneratedIpinvestmentStartTermConfig {

    // 1. إعدادات الحسابات المحاسبية
    @Override
    public ACCSideConfig getDebitAccount() {
        // الحساب المدين (مثلاً: حساب الاستثمارات)
        return getConfig().getInvestmentDebitAccount();
    }

    @Override
    public ACCSideConfig getCreditAccount() {
        // الحساب الدائن (مثلاً: حساب النقدية أو البنك)
        return getConfig().getInvestmentCreditAccount();
    }

    // 2. التحقق من وجود إعدادات محاسبية
    public boolean hasAccountingEffects() {
        return getConfig() != null && 
               getConfig().getInvestmentDebitAccount() != null && 
               getConfig().getInvestmentCreditAccount() != null;
    }
}
```

---

## الخطوة الرابعة: تحديث ملف الـ DSL للإعدادات 📋

**الملف:** `modules/investment/investmententities/src/main/resources/termconfigs/IpinvestmentStartTermConfig.dsl`

```java
@NaMaTermConfig
public class IpinvestmentStartTermConfig extends BasicTermConfig {
    
    // إعدادات الحسابات المحاسبية
    @NaMaField
    ACCSideConfig investmentDebitAccount;   // الحساب المدين (حساب الاستثمارات)
    
    @NaMaField  
    ACCSideConfig investmentCreditAccount;  // الحساب الدائن (النقدية/البنك)
    
    // إعدادات إضافية (اختيارية)
    @NaMaField
    BooleanDF enableAccountingEffects;      // تفعيل/إلغاء التأثير المحاسبي
    
    @NaMaField
    StringDF investmentAccountingNotes;     // ملاحظات محاسبية
}
```

---

## الخطوة الخامسة: تسجيل المستند في النظام 📝

**الملف:** `infra/nama-common/src/main/java/com/namasoft/common/fieldids/EntitiesWithLedgerEffectsOrWithParents.java`

```java
public class EntitiesWithLedgerEffectsOrWithParents {
    
    public static String[] entitiesWithLedgerEffects = {
        "InterestPaymentDocument", 
        "TreasuryBillSalesDoc", 
        "HRLoanDocument",
        // ... الكيانات الموجودة
        
        "IpinvestmentStart"  // إضافة المستند هنا ← مهم جداً!
    };
    
    // باقي الكود...
}
```

---

## الخطوة السادسة: إنشاء Migration للقاعدة 🗃️

**ملف جديد:** `modules/investment/investmentdomain/src/main/java/com/namasoft/modules/investment/domain/migrations/AddLedgerTransReqIdToIpinvestmentStart.java`

```java
public class AddLedgerTransReqIdToIpinvestmentStart extends MigratorBase {
    
    @Override
    protected Result performMigration() {
        Result result = Result.createAccumulatingResult();
        
        // إضافة العمود الجديد لو مش موجود
        String sql = """
            ALTER TABLE IpinvestmentStart 
            ADD COLUMN ledgerTransReqId CHAR(36) NULL;
            """;
            
        executeSQL(sql, result);
        
        return result;
    }
    
    @Override
    public String getDescription() {
        return "Add ledgerTransReqId field to IpinvestmentStart table";
    }
}
```

---

## الخطوة السابعة: اختبار التنفيذ ✅

### 1. اختبار إنشاء المستند:

```java
// إنشاء مستند استثمار جديد
IpinvestmentStart investment = new IpinvestmentStart();
investment.setInvestmentAmount(DecimalDF.fromDouble(100000.0));
investment.setInvestmentType(StringDF.fromString("Fixed Deposit"));

// حفظ المستند (سيتم إنشاء القيد تلقائياً)
Result saveResult = investment.save();
```

### 2. التحقق من إنشاء القيد:

```java
// التحقق من وجود القيد المحاسبي
if (investment.getLedgerTransReqId() != null) {
    LedgerTransReq ledgerReq = LedgerTransReq.findById(investment.getLedgerTransReqId());
    // فحص خطوط القيد المحاسبي
    System.out.println("Ledger lines count: " + ledgerReq.getLinesList().size());
}
```

---

## ملاحظات مهمة ⚠️

### ✅ تأكد من:

1. **إعداد الحسابات المحاسبية** في Term Config
2. **وجود المبلغ** في حقل investmentAmount
3. **تسجيل المستند** في EntitiesWithLedgerEffectsOrWithParents
4. **تشغيل Migration** لإضافة العمود الجديد

### 🔧 للتصحيح:

- تأكد إن الـ Term Config معرف صح
- تحقق من وجود الحسابات المحاسبية في النظام
- اختبر على بيانات تجريبية أول

### 📊 مثال القيد المحاسبي المتوقع:

```
من / الاستثمارات طويلة الأجل    100,000
إلى / النقدية بالصندوق                     100,000
```

---

## الملخص السريع:

1. **حدث DSL** وأضف ledgerTransReqId
2. **حدث كلاس المستند** وأضف الـ methods المطلوبة
3. **أنشئ Term Config** للإعدادات المحاسبية
4. **سجل المستند** في قائمة الكيانات
5. **أنشئ Migration** للقاعدة
6. **اختبر** التنفيذ

كده المستند هيبقى يعمل قيود محاسبية تلقائياً! 🎉

---
# دليل إنشاء مستند الاستثمار مع القيود المحاسبية

## الهدف من المستند

إنشاء مستند `InvestmentStartDoc` يولد قيود محاسبية تلقائياً عند الاعتماد، مثل:

- ودائع ثابتة 🏦
- صناديق استثمار 📈
- أسهم في الشركات 📊
- سندات حكومية 💰

## المكونات الأساسية

### 1. الحقول الإجبارية للمستند

```java
// الحقول الأساسية (موروثة من DocumentFile)
EntityCodeDF code;           // رقم المستند
DateDF valueDate;           // تاريخ المعاملة
DocumentTerm term;          // ربط بإعدادات المستند
Department department;      // القسم
LegalEntity legalEntity;    // الكيان القانوني
FiscalYear fiscalYear;      // السنة المالية
FiscalPeriod fiscalPeriod;  // الفترة المالية

// الحقل الأهم: معرف المعاملة المحاسبية
@NaMaField(system = true)
UniqueIDDF ledgerTransReqId;  // بدون ده مش هيشتغل!

// الحقول المالية
InvoiceMoney money;         // المبلغ والعملة
DecimalDF amount;           // مبلغ الاستثمار
```

### 2. إنشاء كيان المستند في DSL

```java
@NaMaEntity
public class InvestmentStartDoc extends DocumentFile {

    // الحقل الحاسم للقيود المحاسبية
    @NaMaField(system = true)
    UniqueIDDF ledgerTransReqId;

    // نوع الاستثمار
    @NaMaField(required = true)
    InvestmentType investmentType; // وديعة، صندوق، أسهم

    // المعلومات المالية
    @NaMaField(hasEmptyPrefix = true, required = true)
    InvoiceMoney money;

    // تفاصيل الاستثمار
    @NaMaField(required = true)
    TextDF investmentName;      // اسم الاستثمار
    DecimalDF interestRate;     // معدل الفائدة
    IntegerDF termInMonths;     // مدة الاستثمار بالشهور
    DateDF maturityDate;        // تاريخ الاستحقاق

    // الحسابات المحاسبية
    GenericReference sourceAccount;     // الحساب المدين منه
    GenericReference investmentAccount; // حساب الاستثمار

    // للأسهم والصناديق
    DecimalDF numberOfUnits;    // عدد الوحدات
    DecimalDF unitPrice;        // سعر الوحدة

    // تفاصيل البنك
    Bank bank;
    TextDF accountNumber;

    // المرفقات
    LargeData attachment;

    // مراجع إضافية
    GenericReference subsidiary;
    Employee responsiblePerson;
}
```

### 3. أنواع الاستثمار

```java
@NaMaEnum
public enum InvestmentType {
    CASH_DEPOSIT("وديعة نقدية"),
    INVESTMENT_FUND("صندوق استثمار"),
    SHARES("أسهم"),
    BONDS("سندات"),
    FIXED_DEPOSIT("وديعة ثابتة");
}
```

## التطبيق العملي للمستند

### 1. الكيان الأساسي مع الواجهات

```java
@Entity
public class InvestmentStartDoc extends GeneratedInvestmentStartDoc
    implements IGeneratesAccountingRequest {

    @Override
    public Result applyEffects() {
        Result result = Result.createAccumulatingResult();

        // توليد التأثيرات المحاسبية
        InvestmentTermConfig termConfig = (InvestmentTermConfig) getTerm().getTermConfig();
        generateLedgerTransaction(termConfig).addToAccumulatingResult(result);

        // تحديث تتبع الاستثمار
        createOrUpdateInvestmentAsset().addToAccumulatingResult(result);

        return result;
    }

    @Override
    public void genAccEffect(Result result) {
        // هذه الدالة تستدعى تلقائياً من النظام
        InvestmentTermConfig termConfig = termConfig();

        if (BooleanDF.isTrue(getCommitedBefore())) {
            // تطبيق التأثيرات عند الاعتماد
            applyAccountingEffects(termConfig).addToAccumulatingResult(result);
        } else {
            // إلغاء التأثيرات عند الإلغاء
            cancelAccountingEffects(termConfig).addToAccumulatingResult(result);
        }
    }
}
```

### 2. إنشاء القيود المحاسبية

```java
private Result applyAccountingEffects(InvestmentTermConfig termConfig) {
    Result result = Result.createAccumulatingResult();

    // إنشاء طلب المعاملة المحاسبية
    LedgerTransReq request = new LedgerTransReq();
    request.updateCommonData(this);

    // تعيين أو إعادة استخدام معرف المعاملة
    if (ObjectChecker.isNotEmptyOrNull(getLedgerTransReqId())) {
        request.setId(getLedgerTransReqId());
    } else {
        setLedgerTransReqId(request.getId());
    }

    request.setRequestType(TransactionRequestType.Create());

    // إنشاء أسطر القيد
    List<LedgerTransReqLine> lines = createLedgerLines(termConfig);
    request.setLinesList(new LedgerTransReqLineList(lines));

    // إرسال الطلب للنظام المحاسبي
    BusinessRequestUtils.sendBusinessRequest(request, this, true, result);

    return result;
}
```

### 3. إنشاء أسطر القيد المحاسبي

```java
private List<LedgerTransReqLine> createLedgerLines(InvestmentTermConfig config) {
    List<LedgerTransReqLine> lines = new ArrayList<>();

    // السطر المدين (حساب الاستثمار)
    LedgerTransReqLine debitLine = new LedgerTransReqLine();
    debitLine.setAccount(config.getInvestmentAccount());
    debitLine.setDebit(createTransactionMoney(getMoney().getNetValue()));
    debitLine.setNarration("استثمار: " + getInvestmentName());
    debitLine.setOriginId(getId());
    debitLine.setOriginType(getEntityType());
    debitLine.setOriginCode(getCode());
    lines.add(debitLine);

    // السطر الدائن (حساب المصدر)
    LedgerTransReqLine creditLine = new LedgerTransReqLine();
    creditLine.setAccount(config.getSourceAccount());
    creditLine.setCredit(createTransactionMoney(getMoney().getNetValue()));
    creditLine.setNarration("استثمار: " + getInvestmentName());
    creditLine.setOriginId(getId());
    creditLine.setOriginType(getEntityType());
    creditLine.setOriginCode(getCode());
    lines.add(creditLine);

    return lines;
}
```

## أمثلة عملية للاستخدام

### مثال 1: وديعة ثابتة 🏦

```java
// إنشاء وديعة ثابتة
InvestmentStartDoc doc = new InvestmentStartDoc();
doc.setInvestmentType(InvestmentType.FIXED_DEPOSIT);
doc.setInvestmentName("وديعة ثابتة - بنك ABC");
doc.setBank(abcBank);
doc.setAccountNumber("FD-123456789");

// التفاصيل المالية
doc.getMoney().setNetValue(new DecimalDF(100000)); // 100,000 جنيه
doc.setInterestRate(new DecimalDF(12)); // فائدة 12% سنوياً
doc.setTermInMonths(12); // سنة واحدة
doc.setMaturityDate(DateUtils.addMonths(today, 12));

// عند الاعتماد ينتج:
// مدين: حساب الودائع الثابتة    100,000 جنيه
// دائن: حساب البنك             100,000 جنيه
```

### مثال 2: شراء أسهم 📊

```java
// شراء أسهم شركة
InvestmentStartDoc doc = new InvestmentStartDoc();
doc.setInvestmentType(InvestmentType.SHARES);
doc.setInvestmentName("أسهم شركة المصرية للاتصالات");
doc.setNumberOfUnits(new DecimalDF(1000)); // 1000 سهم
doc.setUnitPrice(new DecimalDF(15)); // 15 جنيه للسهم

// إجمالي الاستثمار = 1000 × 15 = 15,000 جنيه
doc.getMoney().setNetValue(new DecimalDF(15000));

// عند الاعتماد ينتج:
// مدين: استثمارات في الأسهم    15,000 جنيه
// دائن: حساب البنك             15,000 جنيه
```

### مثال 3: صندوق استثمار 📈

```java
// استثمار في صندوق نمو
InvestmentStartDoc doc = new InvestmentStartDoc();
doc.setInvestmentType(InvestmentType.INVESTMENT_FUND);
doc.setInvestmentName("صندوق النمو المتوازن");
doc.setNumberOfUnits(new DecimalDF(500)); // 500 وحدة
doc.setUnitPrice(new DecimalDF(50)); // 50 جنيه للوحدة

// إجمالي الاستثمار = 500 × 50 = 25,000 جنيه
doc.getMoney().setNetValue(new DecimalDF(25000));

// عند الاعتماد ينتج:
// مدين: صناديق الاستثمار       25,000 جنيه
// دائن: حساب البنك             25,000 جنيه
```

## خطوات التنفيذ

### 1. إعداد المستند ✅

- توريث من DocumentFile
- إضافة حقل ledgerTransReqId
- تطبيق واجهة IGeneratesAccountingRequest

### 2. إعداد التكوين المحاسبي ⚙️

- إنشاء InvestmentTermConfig
- تحديد الحسابات المدينة والدائنة
- تعيين قواعد التحقق

### 3. تطبيق التأثيرات المحاسبية 💰

- إنشاء LedgerTransReq
- إضافة أسطر القيد المتوازنة
- استخدام BusinessRequestUtils للإرسال

### 4. الاختبار والتحقق ✔️

- اختبار توليد القيود
- التأكد من توازن المدين والدائن
- مراجعة التقارير المحاسبية

## النقاط الحاسمة للنجاح

### الحقل الإجباري

```java
@NaMaField(system = true)
UniqueIDDF ledgerTransReqId;  // بدونه لن تعمل المحاسبة!
```

### الواجهة الإجبارية

```java
implements IGeneratesAccountingRequest {
    @Override
    public void genAccEffect(Result result) {
        // هنا يتم توليد القيود
    }
}
```

### إرسال الطلب المحاسبي

```java
BusinessRequestUtils.sendBusinessRequest(request, this, true, result);
```

## الأخطاء الشائعة وحلولها

❌ **نسيان ledgerTransReqId**: المستند لن يولد قيود محاسبية ✅ **الحل**: إضافة الحقل مع التعليق الصحيح

❌ **عدم تطبيق IGeneratesAccountingRequest**: لا يوجد ربط محاسبي ✅ **الحل**: تطبيق الواجهة وتعريف genAccEffect

❌ **قيود غير متوازنة**: مجموع المدين ≠ مجموع الدائن ✅ **الحل**: التأكد من تساوي المبالغ في الطرفين

❌ **عدم استخدام BusinessRequestUtils**: القيود لن ترسل للنظام المحاسبي ✅ **الحل**: استخدام الأداة الصحيحة لإرسال الطلبات

## الخلاصة النهائية

لإنشاء مستند استثمار مع قيود محاسبية تلقائية:

1. **ورث من DocumentFile** للوظائف الأساسية
2. **أضف ledgerTransReqId** مع التعليق المطلوب
3. **طبق IGeneratesAccountingRequest** للربط المحاسبي
4. **أنشئ TermConfig** مع إعدادات الحسابات
5. **ولد LedgerTransReq** مع أسطر متوازنة
6. **استخدم BusinessRequestUtils** لإرسال الطلب

هذا النمط يضمن تكامل مستند الاستثمار مع النظام المحاسبي وتوليد القيود المناسبة عند الاعتماد! 🚀