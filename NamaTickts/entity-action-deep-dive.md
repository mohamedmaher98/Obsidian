# شرح تفصيلي: كيف تبني EntityAction في NaMa ERP

هنا هنفهم كل سطر في كلاس `EADeleteNotificationsByDuration` - وده هيخليك تقدر تبني أي EntityAction تاني بنفس النمط.

---

## الفكرة العامة: إيه هو EntityAction؟

الـ EntityAction هو كلاس بيعمل "عملية" معينة على entity. في حالتنا، الـ entity هو `TaskSchedule` - يعني الأكشن ده بيتشغل أوتوماتيك من الـ Task Scheduler بتاع السيستم.

فكر فيه كإنه "وظيفة مجدولة" - بتقولها: كل يوم الساعة 12 بالليل، امسحلي التنبيهات القديمة.

---

## السطر بالسطر

### الـ Package والـ Imports

```java
package com.namasoft.infor.domainbase.util.actions;
```
ده المكان اللي كل الـ Entity Actions العامة بتتحط فيه. لو فتحت الفولدر ده هتلاقي فيه:
- `EADeleteOldFiles` - لحذف الملفات القديمة
- `EARunManualNotification` - لتشغيل تنبيه يدوي
- وغيرهم...

لو الأكشن بتاعك متعلق بموديول معين (مثلاً HR أو Accounting)، حطه في فولدر الموديول. لكن لو عام زي ده، يبقى مكانه هنا.

```java
import com.namasoft.bus.TransactionProvider;
```
ده اللي بيديك القدرة تعمل `doInNewTransaction()` - يعني تفتح transaction جديدة في الداتابيز. مهم جداً لما بتحذف كميات كبيرة عشان مش تقفل الداتابيز.

```java
import com.namasoft.common.Pair;
```
`Pair` هو كلاس بسيط بيشيل قيمتين (X و Y). هنا بنستخدمه كـ "flag" نقدر نغيره من جوه الـ lambda. ليه مش boolean عادي؟ لأن الـ lambda في Java مش بتقبل تغيير متغيرات محلية - لازم يكون object تقدر تغير قيمته من جواها.

```java
import com.namasoft.common.fieldids.newids.basic.IdsOfUserNotification;
```
ده كلاس مُولّد أوتوماتيك فيه كل أسماء الـ fields بتاعة `UserNotification`. بدل ما تكتب `"submittedOn"` كـ string (وممكن تغلط فيه)، بتكتب `IdsOfUserNotification.submittedOn` وده ثابت (constant) مضمون إنه صح.

```java
import com.namasoft.infra.domainbase.common.NaMaContext;
```
ده الـ Context بتاع الـ request الحالي. منه بتجيب الـ user الحالي، الشركة، اللغة... أي حاجة متعلقة بالـ session الحالية.

```java
import com.namasoft.infra.domainbase.common.PageData;
```
بيحدد حجم الصفحة لما بتجيب بيانات من الداتابيز. `PageData.records(100)` يعني "هاتلي 100 record بس في كل مرة".

```java
import com.namasoft.infra.domainbase.common.criteria.Criteria;
import com.namasoft.infra.domainbase.common.criteria.CriteriaBuilder;
```
ده الـ query builder بتاع السيستم. بدل ما تكتب SQL مباشرة، بتبني الـ query بكود Java:
```java
CriteriaBuilder.create().field("اسم الحقل").equal("القيمة").build()
```
- `CriteriaBuilder` - بيبني الـ query
- `Criteria` - الناتج النهائي اللي بتباسيه للـ Persister

::: warning ملاحظة مهمة
`CriteriaBuilder.create()` بيرجع `CriteriaBuilder`، لكن لما تنادي `.field(...)` بيرجع `ExpressionBuilder` (كلاس داخلي). فمتقدرش تخزن الناتج في متغير من نوع `CriteriaBuilder` - لازم تبني الـ chain كاملة وتنادي `.build()` في الآخر عشان تحصل على `Criteria`.
:::

```java
import com.namasoft.infra.domainbase.datafields.BooleanDF;
```
في السيستم ده مفيش `boolean` عادي. كل حاجة ليها DataField خاص بيها. `BooleanDF` عنده methods زي `isFalse()` و `isTrue()` اللي بتتعامل مع null بأمان.

```java
import com.namasoft.infra.domainbase.datafields.DateDF;
import com.namasoft.infra.domainbase.datafields.DateTimeDF;
```
- `DateDF` = تاريخ بس (يوم/شهر/سنة)
- `DateTimeDF` = تاريخ + وقت

```java
import com.namasoft.infra.domainbase.datafields.LongTextDF;
```
ده نوع الـ parameters اللي بتيجي للـ EntityAction. كل parameter هو `LongTextDF` (نص طويل). أنت بتقرأه كـ string وبتحوله للنوع اللي محتاجه.

```java
import com.namasoft.infra.domainbase.entity.base.EntityAction;
```
ده الـ interface اللي لازم كل Entity Action يعمله implement. فيه الـ methods اللي لازم تعملها override:
- `doAction()` - التنفيذ الفعلي
- `describe()` - وصف الأكشن
- `columnNames()` - أسماء الـ parameters
- `validateParameters()` - التحقق من المدخلات

```java
import com.namasoft.infra.domainbase.persistence.repos.FilterOnDimensions;
```
الـ Dimensions في السيستم هي (فرع، قسم، مشروع...). `FilterOnDimensions.No` يعني "متفلترش على الـ dimensions - هاتلي كل حاجة بغض النظر عن الفرع أو القسم".

```java
import com.namasoft.infra.domainbase.persistence.repos.ListPageMatchingParameters;
```
ده الـ object اللي بتبنيه عشان تقول للـ Persister: "عايز أجيب records بالشروط دي، والصفحة دي، والفلتر ده".

```java
import com.namasoft.infra.domainbase.persistence.repos.Persister;
```
**ده أهم كلاس في السيستم كله.** الـ Persister هو اللي بيتعامل مع الداتابيز:
- `Persister.findByID()` - جيب record بالـ ID
- `Persister.listPageMatching()` - جيب records بشروط معينة
- `Persister.deleteAll()` - امسح مجموعة records
- `Persister.findByBusinessCode()` - جيب record بالكود

```java
import com.namasoft.infra.domainbase.security.NaMaUser;
```
ده كائن الـ user الحالي. منه بتجيب الـ Security Profile والصلاحيات.

```java
import com.namasoft.infra.domainbase.util.Result;
```
**ده ثاني أهم كلاس.** كل عملية في السيستم بترجع `Result`:
- `Result.createSuccessResult()` - نجحت
- `Result.createFailureResult("رسالة الخطأ")` - فشلت
- `Result.createAccumulatingResult()` - result بيجمع أخطاء متعددة

```java
import com.namasoft.modules.basic.domain.entities.TaskSchedule;
```
ده الـ entity اللي بيمثل "المهمة المجدولة". لما بتعمل `EntityAction<TaskSchedule>` يعني الأكشن ده بيشتغل على TaskSchedule.

```java
import com.namasoft.modules.basic.domain.primitives.EntityTargetAction;
```
بيحمل معلومات عن الـ action نفسه (اسمه، الـ entity اللي بيشتغل عليه). بيتباسي في `validateParameters`.

```java
import com.namasoft.modules.basic.domain.primitives.NotificationStatus;
```
Enum بيمثل حالة التنبيه:
- `NotificationStatus.Viewed()` = مقروء
- `NotificationStatus.Pending()` = لسه جديد

```java
import com.namasoft.modules.basic.domain.valueobjects.UserNotification;
```
ده الـ entity بتاع التنبيه نفسه - الـ record اللي في الداتابيز.

---

### تعريف الكلاس

```java
public class EADeleteNotificationsByDuration implements EntityAction<TaskSchedule>
```

- **التسمية**: كل Entity Action بيبدأ بـ `EA` (اختصار Entity Action). ده convention مهم عشان لما حد يشوف `EA` في اسم كلاس يعرف إنه Entity Action.
- **`implements EntityAction<TaskSchedule>`**: بنقول للسيستم إن الأكشن ده بيشتغل على `TaskSchedule`.
- **التسجيل أوتوماتيك**: مش محتاج تسجل الكلاس في أي مكان. السيستم عنده `EntityActionsReflectionUtil` بيعمل scan لكل الكلاسات اللي بتعمل implement لـ `EntityAction` وبيسجلها أوتوماتيك.

---

### ثوابت الـ Parameters

```java
private static final int DURATION_DAYS = 1;
private static final int DELETE_TYPE = 2;
```

الـ parameters بتيجي كـ array (`LongTextDF... parameters`). بدل ما تكتب `parameters[1]` في كل مكان (وتنسى إيه اللي في index 1)، بتعمل constants بأسماء واضحة.

- **Index 0**: فاضي (محجوز)
- **Index 1 = `DURATION_DAYS`**: عدد الأيام
- **Index 2 = `DELETE_TYPE`**: نوع الحذف (readonly أو all)

::: tip
دايماً اعمل constants للـ parameter indices. ده بيخلي الكود أوضح وأسهل في التعديل.
:::

---

### Method: `doAction` - التنفيذ الفعلي

ده قلب الأكشن. خلينا نفصله جزء جزء:

#### 1. التحقق من الصلاحيات

```java
NaMaUser currentUser = NaMaContext.getCurrentContext().getCurrentUser();
if (currentUser.getSecurityProfile() == null || BooleanDF.isFalse(currentUser.getSecurityProfile().getFullAuthority()))
    return Result.createFailureResult("You do not have the authority...");
```

ده نمط (pattern) ثابت في السيستم لما تعمل عملية حساسة:
1. جيب الـ user الحالي من الـ Context
2. تأكد إن عنده Security Profile
3. تأكد إن عنده Full Authority
4. لو مش عنده، ارجع failure result

::: warning
ده بالظبط نفس الكود الموجود في `BasicUtilityWSImpl.deleteAllNotifications()`. لازم تنسخ النمط بالظبط - نفس الشرط، نفس الرسالة تقريباً.
:::

#### 2. قراءة الـ Parameters

```java
int days = Integer.parseInt(parameters[DURATION_DAYS].getPrimitiveValue().trim());
```
- `parameters[DURATION_DAYS]` = جيب الـ parameter من الـ array (index 1)
- `.getPrimitiveValue()` = حوله لـ String عادي
- `.trim()` = شيل المسافات
- `Integer.parseInt()` = حوله لرقم

```java
String deleteType = parameters[DELETE_TYPE].getPrimitiveValue().trim().toLowerCase();
```
نفس الفكرة، بس هنا بنخليه lowercase عشان المقارنة.

#### 3. حساب تاريخ القطع (Cutoff Date)

```java
DateTimeDF cutoffDateTime = DateTimeDF.fromLocalDateTime(
    DateDF.today().minusDays(days).getPrimitiveValue().atStartOfDay()
);
```

خلينا نفككها:
1. `DateDF.today()` → تاريخ النهارده (مثلاً 2026-03-02)
2. `.minusDays(days)` → اطرح عدد الأيام (لو days=30، يبقى 2026-01-31)
3. `.getPrimitiveValue()` → حول من `DateDF` لـ `LocalDate` (Java standard)
4. `.atStartOfDay()` → حول لـ `LocalDateTime` في أول لحظة من اليوم (00:00:00)
5. `DateTimeDF.fromLocalDateTime(...)` → غلفه في `DateTimeDF` عشان السيستم يفهمه

يعني لو قلتله "30 يوم"، هيحسب التاريخ اللي قبل 30 يوم وهيحذف كل التنبيهات الأقدم من التاريخ ده.

#### 4. تحديد نوع الحذف

```java
boolean readonlyOnly = deleteType.equals("readonly");
```

بسيطة - لو الـ parameter بيقول "readonly" يبقى هنحذف المقروءة بس.

#### 5. حلقة الحذف بالـ Batch

```java
Pair<Boolean, Boolean> hasMoreNotifications = new Pair<>(true, true);
```

هنا الحتة المهمة. ليه مش بنحذف كل حاجة مرة واحدة؟

**لأن لو عندك مليون تنبيه وحذفتهم في transaction واحدة:**
- الداتابيز هتتقفل (lock) لفترة طويلة
- ممكن تحصل OutOfMemoryError
- لو حصل خطأ في النص، كل حاجة هترجع (rollback)

فبدل كده، بنحذف 100 في كل مرة في transaction مستقلة.

**ليه `Pair<Boolean, Boolean>` مش `boolean`؟**
لأن في Java، الـ lambda (الكود اللي جوه `doInNewTransaction`) مش بيقدر يغير قيمة متغير محلي (locally effective final). فبنستخدم `Pair` كـ wrapper object - الـ object نفسه مش بيتغير، بس القيمة اللي جواه بتتغير.

```java
do
{
    TransactionProvider.get().doInNewTransaction(() -> {
```

ال`doInNewTransaction` بيفتح transaction جديدة في الداتابيز، ينفذ الكود، ولو نجح يعمل commit. لو حصل exception يعمل rollback. كل iteration من الحلقة = transaction مستقلة.

#### 6. بناء الـ Criteria (الشروط)

```java
Criteria criteria;
if (readonlyOnly)
    criteria = CriteriaBuilder.create()
        .field(IdsOfUserNotification.submittedOn).lessThanOrEqual(cutoffDateTime)
        .and()
        .field(IdsOfUserNotification.status).equal(NotificationStatus.Viewed())
        .build();
else
    criteria = CriteriaBuilder.create()
        .field(IdsOfUserNotification.submittedOn).lessThanOrEqual(cutoffDateTime)
        .build();
```

ده الـ query builder. خلينا نترجمه لـ SQL:

**حالة readonly:**
```sql
WHERE submittedOn <= '2026-01-31 00:00:00' AND status = 'Viewed'
```

**حالة all:**
```sql
WHERE submittedOn <= '2026-01-31 00:00:00'
```

ملاحظات:
- `CriteriaBuilder.create()` → ابدأ query جديد
- `.field(IdsOfUserNotification.submittedOn)` → حدد الحقل
- `.lessThanOrEqual(cutoffDateTime)` → الشرط (أقل من أو يساوي)
- `.and()` → و كمان
- `.build()` → خلص وأرجعلي الـ Criteria

::: warning مهم جداً
لازم تبني الـ chain كاملة في سطر واحد (أو أسطر متصلة) وتنادي `.build()` في الآخر. متحاولش تخزن نتيجة وسطية في `CriteriaBuilder` لأن `.field()` بيرجع `ExpressionBuilder` مش `CriteriaBuilder`.
:::

#### 7. جلب وحذف الـ Batch

```java
List<UserNotification> userNotifications = Persister.listPageMatching(UserNotification.class,
    ListPageMatchingParameters.create()
        .pageData(PageData.records(100))
        .criteria(criteria)
        .filterOnDimensions(FilterOnDimensions.No));
```

- `Persister.listPageMatching()` → جيب records من الداتابيز بالشروط دي
- `UserNotification.class` → نوع الـ entity
- `PageData.records(100)` → 100 record بس في كل مرة
- `.criteria(criteria)` → الشروط اللي بنيناها
- `.filterOnDimensions(FilterOnDimensions.No)` → متفلترش على الأبعاد (فروع/أقسام)

```java
Persister.deleteAll(userNotifications);
```
امسح الـ 100 record دول.

```java
if (userNotifications.size() < 100)
    hasMoreNotifications.setX(false);
```
لو جابلنا أقل من 100، يعني خلصنا - مفيش تانبيهات تانية. فبنقفل الحلقة.

```java
        return null;
    });
}
while (hasMoreNotifications.getX());
```

الحلقة بتفضل شغالة طالما لسه فيه records للحذف.

#### 8. الـ Return

```java
return Result.createSuccessResult();
```

كل حاجة تمام، رجع نجاح.

---

### Method: `describe` - وصف الأكشن

```java
@Override
public String describe()
{
    return "Deletes notifications older than specified number of days...";
}
```

ده الوصف اللي بيظهر في UI لما حد يختار الأكشن ده من قائمة الـ Entity Actions. اكتب وصف واضح بالإنجليزي.

---

### Method: `columnNames` - أسماء الـ Parameters

```java
@Override
public List<String> columnNames()
{
    return Arrays.asList("", "Duration Days", "Delete Type (readonly, all)");
}
```

دي أسماء الأعمدة اللي بتظهر في UI لما حد يعمل setup للـ TaskSchedule:

| (فاضي) | Duration Days | Delete Type (readonly, all) |
|---------|--------------|----------------------------|
|         | 30           | readonly                   |

كل entry في الـ List بتقابل parameter index:
- Index 0 → `""` (فاضي/محجوز)
- Index 1 → `"Duration Days"`
- Index 2 → `"Delete Type (readonly, all)"`

::: tip
حط القيم المسموحة في الاسم زي `(readonly, all)` عشان اللي بيملي الـ parameters يعرف يكتب إيه.
:::

---

### Method: `validateParameters` - التحقق من المدخلات

```java
@Override
public Result validateParameters(ParamValidationInfo info, EntityTargetAction action, LongTextDF... parameters)
{
    Result result = Result.createAccumulatingResult();
    EntityAction.ensureParamIsNotEmpty(info, result, DURATION_DAYS, parameters);
    EntityAction.ensureParamIsIntegerOrEmpty(info, result, DURATION_DAYS, parameters);
    EntityAction.ensureParamIsNotEmpty(info, result, DELETE_TYPE, parameters);
    EntityAction.ensureParamIsOneOfOrEmpty(info, result, DELETE_TYPE, parameters, "readonly", "all");
    return result;
}
```

ده بيتنادى قبل ما الأكشن يشتغل. بيتأكد إن الـ parameters صح.

- `Result.createAccumulatingResult()` → result بيجمع كل الأخطاء (مش بيوقف عند أول غلطة)
- `ensureParamIsNotEmpty()` → تأكد إن الـ parameter مش فاضي
- `ensureParamIsIntegerOrEmpty()` → تأكد إنه رقم صحيح
- `ensureParamIsOneOfOrEmpty()` → تأكد إنه واحد من القيم المسموحة

**الـ validation helpers المتاحة في `EntityAction`:**

| Helper | الغرض |
|--------|-------|
| `ensureParamIsNotEmpty` | الـ parameter لازم يكون مليان |
| `ensureParamIsIntegerOrEmpty` | لازم يكون رقم صحيح أو فاضي |
| `ensureParamIsDecimalOrEmpty` | لازم يكون رقم عشري أو فاضي |
| `ensureParamIsTrueOrFalseOrEmpty` | لازم يكون true/false أو فاضي |
| `ensureParamIsOneOfOrEmpty` | لازم يكون واحد من قائمة قيم أو فاضي |
| `makeSureAtLeastOneParameterIsFilled` | على الأقل parameter واحد مليان |

---

## ملخص: لو جاتلك تيكت تعمل EntityAction جديد

### الخطوات:

1. **أنشئ كلاس** في `infor/domainbase/util/actions/` اسمه يبدأ بـ `EA`
2. **عمل `implements EntityAction<T>`** - حدد الـ entity type (غالباً `TaskSchedule` أو `BaseEntity`)
3. **حدد الـ parameter indices** كـ `private static final int`
4. **عمل override لأربع methods:**
   - `doAction()` → المنطق
   - `describe()` → الوصف
   - `columnNames()` → أسماء الـ parameters
   - `validateParameters()` → التحقق
5. **مش محتاج تسجل حاجة** - السيستم بيكتشفه أوتوماتيك

### الأنماط المهمة اللي لازم تحفظها:

- **صلاحيات**: `NaMaContext.getCurrentContext().getCurrentUser()` + check `getFullAuthority()`
- **حذف batch**: `do/while` + `doInNewTransaction` + `PageData.records(100)` + check `size() < 100`
- **بناء queries**: `CriteriaBuilder.create().field(...).operator(...).build()`
- **Pair كـ flag**: `Pair<Boolean, Boolean>` عشان تقدر تغيره من جوه الـ lambda
- **الـ Result**: دايماً ارجع `Result` - إما success أو failure

### Template سريع:

```java
public class EAYourAction implements EntityAction<TaskSchedule>
{
    private static final int PARAM_1 = 0;
    private static final int PARAM_2 = 1;

    @Override
    public Result doAction(TaskSchedule object, LongTextDF... parameters)
    {
        // 1. صلاحيات (لو محتاج)
        // 2. اقرأ الـ parameters
        // 3. نفذ المنطق
        // 4. ارجع Result
        return Result.createSuccessResult();
    }

    @Override
    public String describe()
    {
        return "وصف الأكشن بالإنجليزي";
    }

    @Override
    public List<String> columnNames()
    {
        return Arrays.asList("Param 1 Name", "Param 2 Name");
    }

    @Override
    public Result validateParameters(ParamValidationInfo info, EntityTargetAction action, LongTextDF... parameters)
    {
        Result result = Result.createAccumulatingResult();
        EntityAction.ensureParamIsNotEmpty(info, result, PARAM_1, parameters);
        return result;
    }
}
```
