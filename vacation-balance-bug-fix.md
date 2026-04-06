# اصلاح مشكلة رصيد الاجازات مع مستند تحديث بيانات الموظف

## المشكلة بالعربي البسيط

لما بتعمل مستند اجازة لموظف عنده مستند "تحديث بيانات موظف" (UpdateEmployeeInfo) متعمد في نفس الفترة، وبتغير تاريخ بداية الاجازة (مثلا من 02-08 لـ 03-08)، **الرصيد المتبقي مش بيتغير** وبيفضل نفس القيمة.

لكن لو جربت نفس الكلام على موظف **مالوش** مستند تحديث بيانات، الرصيد بيتحدث صح وبيضيف اليوم المستحق.

---

## ليه المشكلة بتحصل؟

### الفكرة الاساسية

النظام لما بيحسب رصيد الاجازة، بيدور على **اخر قيد رصيد** قبل تاريخ الاجازة اللي انت بتعمله. يعني بيقول "ايه اخر حاجة اتسجلت قبل التاريخ ده؟" وبيبني عليها.

### اللي بيحصل للموظف العادي (من غير تحديث بيانات)

```
بيدور على اخر قيد ← مش لاقي حاجة (null)
↓
بيحسب الرصيد من اول السنة لحد تاريخ الاجازة ← الحساب بيتغير مع التاريخ ✅
```

### اللي بيحصل للموظف اللي عنده تحديث بيانات

```
بيدور على اخر قيد ← لاقي قيد تحديث البيانات
↓
بياخد الرصيد المتبقي من القيد ده مباشرة (رقم ثابت) ← مش بيتغير ابدا ❌
```

---

## وين المشكلة في الكود؟

### الملف

```
modules/hr/hrdomain/src/main/java/com/namasoft/modules/humanresource/utils/CommonSysVacationBalanceUtils.java
```

### الدالة المسؤولة: `findLastEntryBeforeMe`

الدالة دي بتجيب **اخر قيد رصيد اجازة** قبل تاريخ معين. المشكلة انها كانت بترجع قيد الـ UpdateEmployeeInfo عادي، وده بيخلي الحساب يستخدم الرصيد الثابت بتاعه.

### المسار الكامل للمشكلة

```
المستخدم يغير تاريخ البداية في الشاشة
    ↓
PostActor (postStartDate) بيتنفذ
    ↓
بيستدعي Web Service: calcVacationConsumedRemainderBalance
    ↓
بيروح على VacationsUtils.computeVacationAssignedConsumedRemainderAtDate
    ↓
بيعمل createBalanceInquiryEntry
    ↓
بيستدعي findLastEntryBeforeMe ← هنا بيلاقي قيد الـ UpdateEmployeeInfo
    ↓
بيستدعي inquiryBalance(doc, line, lastEntry, result)
    ↓
بيستدعي calculateCurrentBalance(lastEntry, entry)
    ↓
بيستدعي currentBalanceIsLastEntryBalance(lastEntry, entry) ← بترجع true
    ↓
بيرجع lastEntry.getRemainder() ← رقم ثابت مش بيتغير مع التاريخ ❌
```

### الدالة اللي بتسبب المشكلة: `currentBalanceIsLastEntryBalance`

```java
private static <T extends ISysBalanceEntry> boolean currentBalanceIsLastEntryBalance(T lastEntry, T entry)
{
    if (byDateBalance())
        return false;
    if (ObjectChecker.isFalse(areEntriesInSameYear(lastEntry, entry)))
        return false;
    return isVacationTypeBeginningOfYear(entry.getVacationType());
}
```

الدالة دي بتقول: "لو القيد الاخير في نفس السنة ونوع الاجازة بداية السنة، خد الرصيد من القيد الاخير مباشرة."

المشكلة ان ده بيشتغل مع القيود العادية (زي اجازة سابقة)، لكن مع قيد تحديث البيانات بيعطي رقم ثابت مش بيتغير.

### الدالة اللي بتستخدم النتيجة: `calculateCurrentBalance`

```java
private static <T extends ISysBalanceEntry> DecimalDF calculateCurrentBalance(T lastEntry, T entry)
{
    if (currentBalanceIsLastEntryBalance(lastEntry, entry))
        return fetchMaxVacationBalance(lastEntry.getVacationType(), lastEntry.getRemainder());
        // ↑ هنا المشكلة: بيرجع الرصيد الثابت من القيد

    // الكود ده مش بيتنفذ لان الشرط فوق بيكون true
    DecimalDF balance = entry.getPeriodBalance();
    if (lastEntry != null && shouldAddLastEntryBalance(lastEntry, entry))
        balance = balance.add(lastEntry.getRemainder());
    return fetchMaxVacationBalance(entry.getVacationType(), balance);
}
```

---

## الحل

### الفكرة

بدل ما `findLastEntryBeforeMe` ترجع قيد الـ UpdateEmployeeInfo، **نتخطاه** ونرجع القيد اللي قبله (او null لو مفيش). كده النظام بيحسب الرصيد من الصفر زي ما بيعمل مع الموظف العادي.

### ليه ده شغال؟

لان بيانات الموظف **اتحدثت فعلا** لما مستند تحديث البيانات اتعمد. يعني الـ EmployeeInfoOnDate بتستخدم البيانات الجديدة تلقائي. مش محتاجين قيد الـ UpdateEmployeeInfo عشان نعرف الرصيد الصح.

### الكود اللي اتضاف

في دالة `findLastEntryBeforeMe` (سطر 460)، بعد التعامل مع FiringDocument:

```java
if (lastEntryBeforeMe.getOwner().instanceOf(HREntityTypes.UpdateEmployeeInfo()))
{
    Set<UniqueIDDF> newExceptEntries = new HashSet<>();
    if (exceptEntries != null)
        newExceptEntries.addAll(exceptEntries);
    newExceptEntries.add(lastEntryBeforeMe.getId());
    return findLastEntryBeforeMe(doc, type, employee, startDate, lineId, newExceptEntries);
}
```

### شرح الكود سطر سطر

```java
// لو القيد الاخير تبع مستند تحديث بيانات موظف
if (lastEntryBeforeMe.getOwner().instanceOf(HREntityTypes.UpdateEmployeeInfo()))
{
    // بنعمل مجموعة جديدة للقيود اللي عايزين نتجاهلها
    Set<UniqueIDDF> newExceptEntries = new HashSet<>();

    // لو في قيود متجاهلة من قبل، نضيفها
    if (exceptEntries != null)
        newExceptEntries.addAll(exceptEntries);

    // نضيف قيد الـ UpdateEmployeeInfo للقيود المتجاهلة
    newExceptEntries.add(lastEntryBeforeMe.getId());

    // ندور تاني على اخر قيد، بس المرة دي من غير قيد الـ UpdateEmployeeInfo
    return findLastEntryBeforeMe(doc, type, employee, startDate, lineId, newExceptEntries);
}
```

---

## قبل وبعد الاصلاح

### قبل (المشكلة)

```
تاريخ 02-08 ← findLastEntryBeforeMe ← قيد UEI ← رصيد = 25 (ثابت)
تاريخ 03-08 ← findLastEntryBeforeMe ← قيد UEI ← رصيد = 25 (ثابت) ← نفس القيمة ❌
```

### بعد (الحل)

```
تاريخ 02-08 ← findLastEntryBeforeMe ← تخطى UEI ← null ← يحسب من الاول ← رصيد يتغير ✅
تاريخ 03-08 ← findLastEntryBeforeMe ← تخطى UEI ← null ← يحسب من الاول ← رصيد يتغير ✅
```

---

## ليه الحل ده امان؟

1. **نفس النمط الموجود**: الكود الاصلي بيعمل نفس الحاجة مع `FiringDocument` (بيتخطاه في حالات معينة). الحل بيتبع نفس النمط.

2. **مش بيأثر على الاعتماد**: في مسار الاعتماد (`reCalculateEntriesData`)، قيود الـ UEI موجودة في `exceptEntries` اصلا، فالتغيير مش بيأثر عليها.

3. **مفيش loop لا نهائي**: لو في اكتر من مستند تحديث بيانات، الدالة بتتخطاهم واحد واحد لحد ما تلاقي قيد عادي او ترجع null.

4. **بيانات الموظف محدثة**: بعد اعتماد مستند تحديث البيانات، بيانات الموظف نفسه بتتحدث. فالنظام بيستخدم البيانات الجديدة تلقائي من غير ما يحتاج قيد الـ UEI.
