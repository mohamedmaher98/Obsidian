# شرح الـ Enum في السيستم بتاعنا  
  
## يعني إيه Enum أصلاً؟  
  
الـ Enum ده اختصار لكلمة **Enumeration** يعني "تعداد". تخيل إنك عايز تحدد قيم معينة بس ومفيش غيرها.  
  
مثلاً لو عندك حقل "الحالة" وعايزه يبقى:  
- "نشط" أو "غير نشط" بس  
- مش أي كلام تاني  
  
ده بالظبط الـ Enum - قايمة محددة من القيم المسموح بيها ومفيش غيرها.  
  
## طيب ليه منستخدمش String عادي؟  
  
لو استخدمت String عادي، أي حد ممكن يكتب أي حاجة:  
- "نشط"  
- "Active"  
- "active"  
- "شغال"  
- "ماشي الحال"  
  
وده هيعمل مشاكل كتير! الـ Enum بيمنع الكلام ده لأنه بيقولك: "لأ يا عم، القيم دي بس المسموح بيها، غير كده مش هينفع."  
  
---  
  
## إزاي السيستم بتاعنا بيتعامل مع الـ Enum؟  
  
في السيستم بتاعنا، الـ Enum مش Java Enum عادي. إحنا عاملين **pattern خاص** عشان نحل مشاكل كتير. خلينا نشوف المثال بتاع `RevisionType`:  
  
### 1. الهيكل الأساسي  
  
```java  
public class RevisionType extends EnumDF  
{  
    // القيم المسموح بيها  
    public enum RevisionTypeEnum    {        Revise, UnRevise    }}  
```  
  
هنا عندنا:  
- ال**Class خارجي** (`RevisionType`) - ده الي بنستخدمه في الكود  
- ال**Enum داخلي** (`RevisionTypeEnum`) - ده فيه القيم الفعلية  
  
### 2. ليه الطريقة دي؟  
  
#### المشكلة مع Java Enum العادي:  
```java  
// Java Enum عادي  
public enum Status {  
    Active, Inactive}  
```  
  
المشكلة إن الـ Java Enum:  
- مبيتخزنش في الداتابيز بسهولة  
- صعب تحوله لـ XML أو JSON  
- مفيش validation سهل  
  
#### الحل بتاعنا:  
إحنا بنـ wrap الـ Enum جوه Class بيـ extend من `EnumDF` وده بيدينا:  
- تخزين في الداتابيز كـ String  
- تحويل سهل لـ XML/JSON  
- Validation تلقائي  
  
---  
  
## شرح الكود سطر سطر  
  
### الجزء الأول: تسجيل القيم المسموحة  
  
```java  
private static ArrayList<String> allowedValues = new ArrayList<String>();  
static  
{  
    for (RevisionTypeEnum type : RevisionTypeEnum.values())    {        allowedValues.add(type.toString());    }}  
```  
  
ده بيحصل مرة واحدة بس لما الـ Class يتحمل:  
1. بيعمل ArrayList فاضية  
2. بيلف على كل قيم الـ Enum (`Revise`, `UnRevise`)  
3. بيضيفهم للـ ArrayList  
  
كده السيستم عارف إن القيم المسموحة هي: `["Revise", "UnRevise"]`  
  
### الجزء التاني: الـ Factory Methods  
  
```java  
public static RevisionType Revise()  
{  
    return fromRevisionTypeEnum(RevisionTypeEnum.Revise);}  
  
public static RevisionType UnRevise()  
{  
    return fromRevisionTypeEnum(RevisionTypeEnum.UnRevise);}  
```  
  
دي methods ثابتة بترجعلك قيمة جاهزة. بدل ما تكتب:  
```java  
RevisionType type = RevisionType.fromString("Revise");  
```  
  
تقدر تكتب:  
```java  
RevisionType type = RevisionType.Revise();  
```  
  
أسهل وأوضح ومفيش غلطات إملائية!  
  
### الجزء التالت: الـ Check Methods  
  
```java  
@Transient  
public boolean isRevise()  
{  
    return Revise().equals(this);}  
  
@Transient  
public boolean isUnRevise()  
{  
    return UnRevise().equals(this);}  
```  
  
دي methods بتسهل عليك الفحص. بدل:  
```java  
if (type.getPrimitiveValue().equals("Revise"))  
```  
  
تقدر تكتب:  
```java  
if (type.isRevise())  
```  
  
::: tip ملاحظة  
الـ `@Transient` معناها إن الحقل ده مش هيتخزن في الداتابيز - ده method بس للاستخدام في الكود.  
:::  
  
### الجزء الرابع: الـ Constructors  
  
```java  
private RevisionType()  
{  
    super("");}  
  
private RevisionType(String value)  
{  
    super(value);}  
```  
  
لاحظ إنهم `private`! ده معناه إنك **مش هتقدر** تعمل:  
```java  
RevisionType type = new RevisionType("Revise"); // ❌ مش هينفع  
```  
  
لازم تستخدم الـ Factory Methods:  
```java  
RevisionType type = RevisionType.Revise(); // ✅ صح  
RevisionType type = RevisionType.fromString("Revise"); // ✅ صح  
```  
  
### الجزء الخامس: الـ Conversion Methods  
  
```java  
public static RevisionType fromRevisionTypeEnum(RevisionTypeEnum value)  
{  
    return new RevisionType(value.toString());}  
  
public static RevisionType fromString(String value)  
{  
    return new RevisionType(value);}  
  
@Transient  
public RevisionTypeEnum getRevisionType()  
{  
    if(isEmpty())        return null;    return RevisionTypeEnum.valueOf(getPrimitiveValue());}  
```  
  
دول بيحولوا بين الأشكال المختلفة:  
- `fromRevisionTypeEnum`: من Enum لـ RevisionType  
- `fromString`: من String لـ RevisionType  
- `getRevisionType`: من RevisionType للـ Enum الداخلي  
  
---  
  
## الـ EnumDF الأساسي  
  
كل الـ Enums بتاعتنا بتـ extend من `EnumDF`. ده فيه الـ validation:  
  
```java  
@Override  
public void setPrimitiveValue(String value)  
{  
    if (strict() && ObjectChecker.isNotEmptyOrNull(value) && !getAllowedValues().contains(value))        throw new NaMaFailureResultException(            "Invalid value for enum:{0} in:{1}, allowed values are ({2})",            value,            this.getClass().getName(),            StringUtils.toCSVLine(getAllowedValues())        );    this.value = value;}  
```  
  
لو حاولت تحط قيمة مش موجودة في `allowedValues`، السيستم هيرمي Exception ويقولك:  
> "Invalid value for enum: بلبل in: RevisionType, allowed values are (Revise, UnRevise)"  
  
---  
  
## الـ Adapter - للتحويل من وإلى XML  
  
```java  
public class RevisionTypeAdapter extends NaMaStringFieldBasicAdapter<RevisionType>  
{  
    @Override    public RevisionType unmarshal(String v) throws Exception    {        return RevisionType.fromString(v);    }}  
```  
  
ده بيستخدمه الـ Web Services لما:  
- **Marshal**: يحول الـ Object لـ XML/JSON (بيبعت للـ client)  
- **Unmarshal**: يحول من XML/JSON لـ Object (بيستقبل من الـ client)  
  
---  
  
## الرحلة الكاملة للـ Enum  
  
```  
┌─────────────────────────────────────────────────────────────────┐  
│                         الـ UI (Vue)                            ││                    بيبعت: "Revise"                              │  
└─────────────────────────────────────────────────────────────────┘  
                              │                              ▼┌─────────────────────────────────────────────────────────────────┐  
│                      Web Service                                │  
│         RevisionTypeAdapter.unmarshal("Revise")                 │  
│              ↓                                                  │  
│         RevisionType.fromString("Revise")                       │  
└─────────────────────────────────────────────────────────────────┘  
                              │                              ▼┌─────────────────────────────────────────────────────────────────┐  
│                    Business Logic                               │  
│                                                                 │  
│    if (doc.getRevisionType().isRevise()) {                     │  
│        // اعمل حاجة                                             ││    }                                                            │  
└─────────────────────────────────────────────────────────────────┘  
                              │                              ▼┌─────────────────────────────────────────────────────────────────┐  
│                      Database                                   │  
│           RevisionType column = "Revise"                        │  
│              (متخزن كـ VARCHAR)                                  │└─────────────────────────────────────────────────────────────────┘  
```  
  
---  
  
## إزاي تعمل Enum جديد؟  
  
### الخطوة 1: اعمل الـ Class  
  
```java  
package com.namasoft.modules.yourmodule.domain.primitives;  
  
import com.namasoft.infra.domainbase.datafields.EnumDF;  
import com.namasoft.modules.yourmodule.adapters.YourEnumAdapter;  
import jakarta.persistence.Transient;  
import jakarta.xml.bind.annotation.adapters.XmlJavaTypeAdapter;  
import java.util.ArrayList;  
  
@XmlJavaTypeAdapter(YourEnumAdapter.class)  
@SuppressWarnings("serial")  
public class YourEnum extends EnumDF  
{  
    private static ArrayList<String> allowedValues = new ArrayList<String>();    static    {        for (YourEnumEnum type : YourEnumEnum.values())        {            allowedValues.add(type.toString());        }    }  
    // 1. الـ Enum الداخلي بالقيم  
    public enum YourEnumEnum    {        Value1, Value2, Value3    }  
    // 2. Factory Methods لكل قيمة  
    public static YourEnum Value1()    {        return fromYourEnumEnum(YourEnumEnum.Value1);    }  
    public static YourEnum Value2()    {        return fromYourEnumEnum(YourEnumEnum.Value2);    }  
    public static YourEnum Value3()    {        return fromYourEnumEnum(YourEnumEnum.Value3);    }  
    // 3. Check Methods لكل قيمة  
    @Transient    public boolean isValue1()    {        return Value1().equals(this);    }  
    @Transient    public boolean isValue2()    {        return Value2().equals(this);    }  
    @Transient    public boolean isValue3()    {        return Value3().equals(this);    }  
    // 4. Private Constructors    private YourEnum()    {        super("");    }  
    private YourEnum(String value)    {        super(value);    }  
    // 5. Static Factory Methods    public static YourEnum createEmpty()    {        return new YourEnum();    }  
    public static YourEnum fromYourEnumEnum(YourEnumEnum value)    {        return new YourEnum(value.toString());    }  
    public static YourEnum fromString(String value)    {        return new YourEnum(value);    }  
    // 6. Getter للـ Enum الداخلي  
    @Transient    public YourEnumEnum getYourEnum()    {        if(isEmpty())            return null;        return YourEnumEnum.valueOf(getPrimitiveValue());    }  
    // 7. Allowed Values    @Transient    public ArrayList<String> getAllowedValues()    {        return allowedValues;    }}  
```  
  
### الخطوة 2: اعمل الـ Adapter  
  
```java  
package com.namasoft.modules.yourmodule.adapters;  
  
import com.namasoft.modules.yourmodule.domain.primitives.YourEnum;  
import com.namasoft.infra.domainbase.util.xml.adapters.NaMaStringFieldBasicAdapter;  
  
public class YourEnumAdapter extends NaMaStringFieldBasicAdapter<YourEnum>  
{  
    @Override    public YourEnum unmarshal(String v) throws Exception    {        return YourEnum.fromString(v);    }}  
```  
  
---  
  
## ملخص سريع  
  
| الحاجة | الوظيفة |  
|--------|---------|  
| `YourEnumEnum` | الـ Enum الداخلي - فيه القيم الفعلية |  
| `allowedValues` | List بالقيم المسموحة للـ validation |  
| `Value1()` | Factory method يرجعلك قيمة جاهزة |  
| `isValue1()` | للفحص هل القيمة الحالية هي Value1 |  
| `fromString()` | تحويل من String للـ Type بتاعنا |  
| `fromYourEnumEnum()` | تحويل من الـ Enum الداخلي للـ Type |  
| `getYourEnum()` | يرجعلك الـ Enum الداخلي |  
| `getAllowedValues()` | يرجعلك كل القيم المسموحة |  
| `YourEnumAdapter` | للتحويل في الـ Web Services |  
  
---  
  
## أمثلة من السيستم  
  
السيستم فيه Enums كتير زي:  
- `RevisionType` - (Revise, UnRevise) لنوع المراجعة  
- `FreightChargesType` - لأنواع مصاريف الشحن  
- `POSSecurityCapability` - لصلاحيات نقاط البيع  
- `PurchaseQuotationDeliveryMethod` - لطرق التسليم في عروض الأسعار  
  
كلهم بيتبعوا نفس الـ Pattern!