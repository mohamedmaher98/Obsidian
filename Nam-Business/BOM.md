# شرح Production Order و BOM System

## إيه هو النظام ده؟

ده نظام متكامل لإدارة الإنتاج في ERP، بيتكون من **أوامر الإنتاج** و **قوائم المواد (BOM)**.

## الهيكل العام:

### 1. نظام BOM (Bill of Materials) - قائمة المواد

#### أ) كلاس `BOM` الرئيسي:

**الغرض:** بيحدد إيه المواد والكميات المطلوبة لإنتاج منتج معين

**البيانات الأساسية:**

- `invItem` - المنتج النهائي اللي هنصنعه
- `quantity` - الكمية الأساسية للوصفة
- `routing` - خطوات التصنيع
- `details` - قائمة المواد المطلوبة (BOMMaterialLine)
- `coProducts` - المنتجات المشتركة اللي بتطلع مع المنتج الأساسي

**إعدادات مهمة:**

- `defaultBom` - هل دي الوصفة الافتراضية؟
- `mrp` - هل يستخدم في تخطيط المواد؟
- `batchQty` - حجم الدفعة المطلوبة
- `finishedProductDensity` - كثافة المنتج النهائي

#### ب) كلاس `BOMMaterialLine`:

**الغرض:** بيمثل كل مادة خام في الوصفة

**البيانات المهمة:**

- `finalQty` - الكمية النهائية المطلوبة
- `operationSeq` - رقم العملية اللي المادة تتضاف فيها
- `yield` - نسبة المخرجات (كام % من المادة هيطلع منتج نهائي)
- `issueType` - طريقة صرف المادة (أوتوماتيك، يدوي، إلخ)
- `potency` - قوة/تركيز المادة
- `permittedPercentage` - النسبة المسموح بها للتغيير في الكمية

#### ج) كلاس `BOMCoProductLine`:

**الغرض:** المنتجات الثانوية اللي بتطلع أثناء الإنتاج

**مثال:** لو بتصنع سكر، ممكن يطلع معاه مولاس كمنتج ثانوي

### 2. كلاس Production Order (اللي شرحناه قبل كده)

## الـ Business Logic المفروض يبقى موجود:

### أ) في BOM System:

#### 1. Validation Rules:

- **مش ممكن BOM يبقى فيه circular reference** (منتج يعتمد على نفسه)
- **لازم مجموع yield النسب في BOMMaterialLine ≤ 100%**
- **لازم يبقى في defaultBom واحد بس لكل منتج**
- **operationSeq لازم يبقى متسلسل ومنطقي**

#### 2. Calculation Logic:

```java
// حساب الكمية المطلوبة لكل مادة خام
finalQty = (bomQuantity / bomBaseQuantity) * productionOrderQuantity * (100 / yield)
```

#### 3. Co-Product Management:

- **توزيع التكاليف:** التكلفة الكلية تتوزع على المنتج الأساسي والمنتجات المشتركة
- **حساب costPercentage:** كل co-product له نسبة من التكلفة الكلية

### ب) في Production Order System:

#### 1. BOM Integration:

```java
// لما Production Order يتعمل من BOM:
1. Copy كل BOMMaterialLine إلى OrderComponentLine
2. حساب finalQty لكل component بناء على كمية الأمر
3. Copy كل BOMCoProductLine إلى OrderCoProductLine
4. Set up routing operations
```

#### 2. Material Requirements Planning (MRP):

- **لو BOM.mrp = true:** النظام يحسب المواد المطلوبة أوتوماتيكياً
- **Material explosion:** يحسب كل المواد المطلوبة في كل المستويات
- **Net requirements:** يطرح المخزون المتاح من المطلوب

#### 3. Yield و Waste Management:

- **Expected output = inputQuantity × (yield / 100)**
- **Waste calculation = inputQuantity - expectedOutput**
- **Cost allocation:** تكلفة الـ waste تتوزع على المنتج النهائي

## Business Rules مفروض تبقى موجودة:

### 1. BOM Rules:

- **Unique default BOM:** كل منتج له BOM افتراضي واحد بس
- **Valid routing:** لازم الـ routing يبقى active وصالح للاستخدام
- **Material availability:** قبل إنشاء أمر إنتاج، النظام يتأكد إن المواد متوفرة
- **Yield validation:** مجموع نسب الـ yield مش ممكن يزيد عن 100%

### 2. Production Order Rules:

- **BOM consistency:** لازم الـ BOM المستخدم يبقى نفس إصدار وقت إنشاء الأمر
- **Quantity scaling:** كل الكميات تتحسب بناء على نسبة أمر الإنتاج للـ BOM base quantity
- **Co-product allocation:** التكاليف والكميات تتوزع على كل المنتجات بنسبها المحددة
- **Operation sequence:** العمليات لازم تتنفذ بالترتيب المحدد في الـ routing

### 3. Cost Rules:

- **Material cost = (quantity × unit cost) / yield**
- **Co-product cost distribution:** كل منتج مشترك ياخد نسبته من التكلفة الكلية
- **Waste cost:** تكلفة المفقود تتحمل على المنتج الأساسي

### 4. Inventory Rules:

- **Material issue:** المواد تتصرف بناء على issueType (أوتوماتيك/يدوي)
- **Backflushing:** بعض المواد تتصرف أوتوماتيكياً لما الإنتاج يخلص
- **Lot tracking:** لو المنتج محتاج lot tracking، كل المواد لازم يبقى لها lots محددة

## التحسينات المقترحة:

### 1. BOM Validation Methods:

```java
public class BOM {
    public boolean isValid()
    public List<String> validateStructure()
    public boolean hasCircularReference()
    public decimal calculateTotalYield()
}
```

### 2. BOM Calculation Methods:

```java
public class BOMMaterialLine {
    public RawQuantity calculateRequiredQty(RawQuantity orderQty)
    public decimal calculateWastePercentage()
    public decimal calculateMaterialCost()
}
```

### 3. Production Order Integration:

```java
public class ProductionOrder {
    public void explodeBOM()
    public void calculateMaterialRequirements()
    public void distributeCoProdCosts()
    public boolean canStart()
    public void allocateCoProductCosts()
}
```

### 4. MRP Integration:

```java
public class BOM {
    public List<MaterialRequirement> explodeMaterials(RawQuantity qty)
    public void updateMRPCalculations()
}
```

## مثال عملي:

### BOM للخبز:

- **المنتج النهائي:** رغيف خبز (1 كيلو)
- **المواد:**
    - دقيق: 600 جرام (yield: 95%)
    - ماء: 300 جرام (yield: 80%)
    - خميرة: 10 جرام (yield: 100%)
    - ملح: 12 جرام (yield: 100%)

### لما نعمل Production Order لـ 100 رغيف:

- **دقيق مطلوب:** (600 × 100) ÷ 95% = 63.16 كيلو
- **ماء مطلوب:** (300 × 100) ÷ 80% = 37.5 كيلو
- **خميرة:** 1 كيلو
- **ملح:** 1.2 كيلو

## الخلاصة:

النظام ده متكامل لإدارة الإنتاج، بس محتاج business methods قوية للـ validation والحسابات والتكامل بين BOM والإنتاج. أهم حاجة هي الـ yield management والـ co-product cost allocation.