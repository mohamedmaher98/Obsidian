# نظام التكاليف المعيارية وتحليل الانحرافات

## 1. مستند السعر القياسي الجديد

### أ) الهيدر:

```java
@NaMaEntity
public class StandardPrice extends MasterFile {
    @NaMaField(required = true)
    Routing routing;           // التوجيه
    
    @NaMaField(required = true)
    DateDF fromDate;           // من تاريخ
    
    @NaMaField(required = true)
    DateDF toDate;             // إلى تاريخ
    
    @NaMaField(required = true)
    IntegerDF priority;        // الأولوية
    
    List<StandardPriceLine> details;
}
```

### ب) التفاصيل:

```java
@NaMaDetailLine
public class StandardPriceLine {
    @NaMaField(required = true)
    InvItem item;              // الصنف
    
    @NaMaField(required = true)
    DecimalDF standardPrice;   // السعر القياسي
}
```

## 2. تطوير مستند إغلاق أمر الإنتاج

### أ) الحقول الجديدة في الهيدر:

```java
// الحقول موجودة بالفعل في الكود اللي عندك:
@TranslateField(ar = "تكلفة الوحدة القياسية", en = "Standard Unit Cost")
DecimalDF productUnitStandardCost;

@TranslateField(ar = "إجمالي تكلفة المنتج الفعلية", en = "Actual Product Cost")
DecimalDF actualProductCost;

@TranslateField(ar = "انحراف التكلفة القياسية", en = "Standard Cost Deviation")
DecimalDF standardCostDeviation;

// مطلوب إضافة الجانبين المحاسبيين:
AccountRef debitAccount;    // حساب مدين لانحراف التكلفة
AccountRef creditAccount;   // حساب دائن لانحراف التكلفة
```

### ب) جريد انحراف الخامات (موجود بالفعل):

```java
// الكلاس موجود، بس مطلوب التأكد من كل الحقول:
@NaMaDetailLine
public class OrderCloseMaterialDeviationLine {
    InvItemRef item;                           // الصنف
    DecimalDF actualQty;                       // الكمية الفعلية
    DecimalDF standardQty;                     // الكمية المعيارية
    DecimalDF avgActualUnitCost;               // متوسط تكلفة الوحدة الفعلي
    DecimalDF standardUnitPrice;               // سعر الوحدة القياسي
    DecimalDF totalActualCost;                 // إجمالي التكلفة الفعلية
    DecimalDF totalStandardCostAvgActual;      // إجمالي التكلفة المعيارية بالمتوسط الفعلي
    DecimalDF costDeviation;                   // الانحراف في التكلفة
    DecimalDF quantityDeviation;               // الانحراف في الكمية
    DecimalDF standardQtyCostByStdPrice;       // تكلفة الكمية المعيارية بالسعر القياسي
    DecimalDF actualQtyCostByStdPrice;         // تكلفة الكمية الفعلية بالسعر القياسي
}
```

### ج) جريد انحراف الموارد (موجود بالفعل مع تعديل):

```java
@NaMaDetailLine
public class OrderCloseResourceDeviationLine {
    Resource resource;                         // المورد
    DecimalDF actualHours;                     // ساعات التشغيل الفعلية
    DecimalDF standardHours;                   // ساعات التشغيل المعيارية
    DecimalDF avgActualHourRate;               // متوسط سعر الساعة الفعلي
    DecimalDF standardHourRate;                // سعر الساعة المعياري
    DecimalDF totalActualCost;                 // إجمالي التكلفة الفعلية
    DecimalDF totalStandardCost;               // إجمالي التكلفة المعيارية
    DecimalDF costDeviation;                   // الانحراف في التكلفة
    DecimalDF hoursDeviation;                  // الانحراف في الساعات
    Activity activity;
}
```

## 3. تطوير مستند التكاليف الغير مباشرة الفعلية

```java
// مطلوب إضافة في جريد التكاليف الغير مباشرة:
@NaMaDetailLine
public class ActualIndirectCostLine {
    // الحقول الموجودة...
    
    // الحقل الجديد المطلوب:
    @TranslateField(ar = "انحراف التكلفة الغير مباشرة", en = "Indirect Cost Deviation")
    DecimalDF indirectCostDeviation;
}
```

---

## المعادلات المطلوبة:

### أ) معادلات الهيدر:

#### 1. تكلفة الوحدة القياسية:

```
productUnitStandardCost = إجمالي التكلفة القياسية ÷ الكمية المنتجة
```

#### 2. تكلفة المنتج الفعلية:

```
actualProductCost = (إجمالي تكلفة المواد الفعلية + إجمالي تكلفة الموارد الفعلية + إجمالي التكاليف الغير مباشرة الفعلية)
```

#### 3. انحراف التكلفة القياسية:

```
standardCostDeviation = actualProductCost - (productUnitStandardCost × الكمية المنتجة)
```

### ب) معادلات انحراف الخامات:

#### 1. متوسط تكلفة الوحدة الفعلي:

```
avgActualUnitCost = totalActualCost ÷ actualQty
```

#### 2. إجمالي التكلفة المعيارية بالمتوسط الفعلي:

```
totalStandardCostAvgActual = standardQty × avgActualUnitCost
```

#### 3. الانحراف في التكلفة:

```
costDeviation = totalActualCost - (actualQty × standardUnitPrice)
```

#### 4. الانحراف في الكمية:

```
quantityDeviation = (actualQty - standardQty) × standardUnitPrice
```

#### 5. تكلفة الكمية المعيارية بالسعر القياسي:

```
standardQtyCostByStdPrice = standardQty × standardUnitPrice
```

#### 6. تكلفة الكمية الفعلية بالسعر القياسي:

```
actualQtyCostByStdPrice = actualQty × standardUnitPrice
```

### ج) معادلات انحراف الموارد:

#### 1. متوسط سعر الساعة الفعلي:

```
avgActualHourRate = totalActualCost ÷ actualHours
```

#### 2. إجمالي التكلفة الفعلية:

```
totalActualCost = actualHours × avgActualHourRate
```

#### 3. إجمالي التكلفة المعيارية:

```
totalStandardCost = standardHours × standardHourRate
```

#### 4. الانحراف في التكلفة:

```
costDeviation = totalActualCost - totalStandardCost
```

#### 5. الانحراف في الساعات:

```
hoursDeviation = (actualHours - standardHours) × standardHourRate
```

### د) معادلة التكاليف الغير مباشرة:

#### انحراف التكلفة الغير مباشرة:

```
indirectCostDeviation = التكاليف الغير مباشرة المقدرة - التكاليف الغير مباشرة الفعلية
```

---

## ملاحظات مهمة:

### 1. الحصول على السعر القياسي:

- البحث في مستند السعر القياسي بناء على:
    - الصنف
    - التاريخ (تاريخ الإغلاق يقع بين fromDate و toDate)
    - أعلى أولوية في حالة وجود أكثر من سعر

### 2. تحليل الانحرافات:

- **انحراف السعر:** الفرق بين السعر الفعلي والقياسي
- **انحراف الكمية:** الفرق بين الكمية الفعلية والمعيارية
- **انحراف الكفاءة:** في الموارد (الفرق في ساعات العمل)

### 3. المعالجة المحاسبية:

- الانحرافات الموجبة (فعلي > قياسي) = تكلفة إضافية
- الانحرافات السالبة (فعلي < قياسي) = توفير في التكلفة