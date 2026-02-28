# شرح ميكانيزم نقل حالة الدفع من قيد اثبات الاستحقاق لعقد الايجار

## المشكلة كانت ايه؟

تعالى نتخيل السيناريو ده:

1. عندك **عقد ايجار** (RERentContract) فيه دفعات (اقساط) - مثلا 12 قسط شهري
2. لما بتعمل commit للعقد، السيستم بيولد **قيد اثبات استحقاق** (RERentInstallmentLedger) - ده بيبقى زي نسخة من الاقساط بتاعتك
3. لما المستأجر يدفع، بتعمل **سند قبض** (Receipt Voucher) وبتربطه بقيد اثبات الاستحقاق

المشكلة كانت ايه؟ لما سند القبض بيدفع قسط، **قيد اثبات الاستحقاق** بيتحدث وبيقولك "القسط ده اتدفع" .. بس **عقد الايجار نفسه** مش بيعرف! الاقساط فيه فاضلة زي ما هي، مفيش حاجة اتغيرت.

يعني لو فتحت عقد الايجار هتلاقي القسط لسه مكتوب عليه "لم يتم الدفع" مع ان فعلا اتدفع من خلال سند القبض. 

---

## السيستم بيشتغل ازاي اصلا؟

### الميكانيزم الاصلي (قبل التعديل)

لما سند القبض بيدفع قسط من قيد الاستحقاق، السيستم بيعمل كده:

```
سند القبض (Receipt Voucher)
    ↓
بيعمل InstallmentPaymentEntry (سجل دفع)
    - installmentDoc = قيد الاستحقاق
    - paymentDoc = سند القبض
    - paidValue = المبلغ المدفوع
    ↓
InstallmentUtils.updateInstallmentLines()
    ↓
بيحدث اقساط قيد الاستحقاق (systemPaid, remaining, etc.)
```

يعني السيستم بيعمل "سجل دفع" (InstallmentPaymentEntry) بيقول "القسط الفلاني في المستند الفلاني اتدفع منه كذا". وبعدين بيروح يحدث المستند ده.

المشكلة ان السجل ده بيشاور على **قيد الاستحقاق بس**، مش على عقد الايجار.

### ميكانيزم الـ ClonePayReceiptEntry

السيستم عنده ميكانيزم جاهز اسمه **ClonePayReceiptEntry**. الفكرة بتاعته بسيطة:

> لو عندك مستند A اتولد من مستند B، وعايز لما حد يدفع في A، المدفوعات تتنسخ في B كمان .. بتعمل سجل ClonePayReceiptEntry بيقول:
> - `doc` = B (المستند اللي عايز المدفوعات تروحله)
> - `fromDoc` = A (المستند اللي المدفوعات بتيجي منه)
> - `forInstallments` = TRUE

وبعد كده السيستم لوحده (في method اسمها `cloneEntriesIfNeeded`) بيلاقي السجل ده وبينسخ الـ InstallmentPaymentEntry من A لـ B تلقائي.

---

## الحل اللي عملناه

احنا استخدمنا **نفس الميكانيزم الموجود** (ClonePayReceiptEntry) عشان نخلي المدفوعات تتنقل من قيد الاستحقاق لعقد الايجار.

### الفلو بعد التعديل

```
عقد الايجار (RERentContract)
    ↓ generateRentLedger()
قيد اثبات الاستحقاق (RERentInstallmentLedger)
    ↓ بنعمل ClonePayReceiptEntry جديد:
    |   doc = عقد الايجار
    |   fromDoc = قيد الاستحقاق
    |   forInstallments = TRUE
    ↓
سند القبض (Receipt Voucher) بيدفع قسط
    ↓
InstallmentPaymentEntry بيتعمل لقيد الاستحقاق
    ↓
cloneEntriesIfNeeded() بتلاقي الـ ClonePayReceiptEntry
    ↓
بتنسخ InstallmentPaymentEntry لعقد الايجار كمان (cloned = TRUE)
    ↓
updateInstallmentLines() بتحدث الاتنين:
    ✅ قيد الاستحقاق → systemPaid اتحدث
    ✅ عقد الايجار → systemPaid اتحدث
```

يعني ببساطة: لما العقد بيولد قيد استحقاق، بنقول للسيستم "اي دفعات تحصل على قيد الاستحقاق ده، انسخها في العقد كمان".

---

## التعديلات اللي اتعملت بالتفصيل

### الملف الاول: RERentContract.java

#### 1. method اسمها `createOrUpdateReverseCloneEntry`

دي بتتنادى كل ما العقد يولد قيد استحقاق. بتعمل ايه؟

- بتدور الاول لو في ClonePayReceiptEntry قديم (لنفس العقد ونفس القيد)
- لو لقت واحد قديم بتعيد استخدامه، لو ملقتش بتعمل واحد جديد
- بتملاه:
  - `doc` = عقد الايجار (اللي عايز يستقبل المدفوعات)
  - `fromDoc` = قيد الاستحقاق (اللي المدفوعات هتيجي منه)
  - `forInstallments` = TRUE
  - `forPayment` = FALSE
- بتحفظه في الداتابيز

الـ method دي بتتنادى جوه `generateRentLedger()` بعد ما القيد يتحفظ ويعمل flush.

#### 2. method اسمها `deleteAllReverseCloneEntries`

دي بتمسح كل الـ ClonePayReceiptEntry اللي الـ `doc` بتاعها = العقد ده. بتتنادى لما العقد يتلغى (cancelEffects).

ليه؟ عشان لو العقد اتلغى، مفيش داعي الدفعات تتنسخ تاني.. العقد خلاص مش موجود.

#### 3. في `cancelEffects()`

اضفنا `deleteAllReverseCloneEntries(this, result)` في اول الـ method قبل ما يمسح القيود.

---

### الملف التاني: RERentInstallmentLedger.java

#### 1. method اسمها `deleteReverseCloneEntry`

دي بتمسح الـ ClonePayReceiptEntry **الخاص بقيد استحقاق معين**. يعني بتقوله "امسح السجل اللي بيقول انسخ من القيد ده للعقد ده".

بتشتغل ازاي؟
- بتجيب العقد الاصلي من `fromDoc`
- بتمسح الـ ClonePayReceiptEntry اللي `doc.id = العقد` و `fromDoc.id = القيد`

#### 2. في `cancelEffects()` و `deleteDraft()`

اضفنا `deleteReverseCloneEntry(getFromDoc(), this)` في الاتنين.

ليه؟ عشان لو قيد الاستحقاق اتلغى او اتمسح كـ draft، لازم نشيل سجل النسخ بتاعه.. مش هينفع السيستم يفضل يحاول ينسخ مدفوعات لمستند مش موجود.

---

### الملف التالت: REOpeningRentContract.java

ده عقد الايجار الافتتاحي (الارصدة الافتتاحية). هو بيستخدم نفس ميكانيزم توليد القيود بتاع RERentContract.

اضفنا `RERentContract.deleteAllReverseCloneEntries(this, result)` في `cancelEffects()` بنفس الطريقة.

---

## سيناريوهات مهمة

### لما سند القبض يتلغى

مفيش حاجة محتاجين نعملها! السيستم بيعمل كده لوحده:

1. سند القبض بيتلغى
2. الـ InstallmentPaymentEntry بتاعته بتتمسح (الاصلية **والمنسوخة** كمان، لان كلهم `paymentDoc = سند القبض`)
3. `updateInstallmentLines()` بتشتغل على كل المستندات المتأثرة
4. قيد الاستحقاق **وعقد الايجار** الاتنين بيرجعوا زي ما كانوا

### لما العقد يتعدل ويتولد قيود جديدة

`generateRentLedger()` بتشتغل تاني:
- القيود القديمة اللي مبقتش محتاجة بتتمسح (و `cancelEffects` بتاعتها بتنادي `deleteReverseCloneEntry`)
- القيود الجديدة بتتعمل (و `createOrUpdateReverseCloneEntry` بتتنادى لكل واحد)

### لما في اكتر من قيد استحقاق لعقد واحد

عادي خالص. كل قيد بيبقى ليه ClonePayReceiptEntry خاص بيه. يعني لو العقد عنده 3 قيود استحقاق (مثلا قيد لكل شهر)، هيبقى في 3 سجلات ClonePayReceiptEntry، كل واحد `fromDoc` بيشاور على قيد مختلف، وكلهم `doc` بيشاور على نفس العقد.

---

## ملخص سريع

| اللي بيحصل | اللي بنعمله |
|---|---|
| العقد بيولد قيد استحقاق | بنعمل ClonePayReceiptEntry (doc=عقد, fromDoc=قيد) |
| سند قبض بيدفع قسط في القيد | السيستم بينسخ الدفعة للعقد تلقائي |
| سند القبض يتلغى | السيستم بيمسح الدفعات من القيد **والعقد** تلقائي |
| قيد الاستحقاق يتلغى | بنمسح الـ ClonePayReceiptEntry الخاص بيه |
| العقد يتلغى | بنمسح كل الـ ClonePayReceiptEntry بتوعه |
