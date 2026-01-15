# دليل شامل لـ Enum في Java

## جدول المحتويات
- [ما هو Enum؟](#ما-هو-enum)
- [لماذا نستخدم Enum؟](#لماذا-نستخدم-enum)
- [الصيغة الأساسية](#الصيغة-الأساسية)
- [Enum مع Properties و Methods](#enum-مع-properties-و-methods)
- [Enum مع Constructor](#enum-مع-constructor)
- [Methods مهمة في Enum](#methods-مهمة-في-enum)
- [Enum مع Switch Statement](#enum-مع-switch-statement)
- [Enum يُنفذ Interface](#enum-ينفذ-interface)
- [Abstract Methods في Enum](#abstract-methods-في-enum)
- [EnumSet و EnumMap](#enumset-و-enummap)
- [Enum في قواعد البيانات (JPA/Hibernate)](#enum-في-قواعد-البيانات)
- [أمثلة عملية من عالم ERP](#أمثلة-عملية-من-عالم-erp)
- [أفضل الممارسات](#أفضل-الممارسات)
- [أخطاء شائعة يجب تجنبها](#أخطاء-شائعة-يجب-تجنبها)

---

## ما هو Enum؟

ال**Enum** (اختصار لـ Enumeration) هو نوع بيانات خاص في Java يُستخدم لتعريف مجموعة ثابتة من القيم المعروفة مسبقاً.

فكر فيه كقائمة محددة من الخيارات لا يمكن تغييرها أثناء تشغيل البرنامج.

```java
// مثال بسيط: أيام الأسبوع
public enum Day {
    SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY
}
```

### الفرق بين Enum والثوابت العادية

```java
// ❌ الطريقة القديمة (غير آمنة)
public class OrderStatus {
    public static final int PENDING = 0;
    public static final int PROCESSING = 1;
    public static final int SHIPPED = 2;
    public static final int DELIVERED = 3;
}

// يمكن تمرير أي رقم حتى لو كان غير صحيح!
public void updateOrder(int status) {
    // ماذا لو مررنا status = 99 ؟ 💥
}
```

```java
// ✅ الطريقة الصحيحة باستخدام Enum
public enum OrderStatus {
    PENDING, PROCESSING, SHIPPED, DELIVERED
}

// الآن المترجم يضمن أن القيمة صحيحة
public void updateOrder(OrderStatus status) {
    // يستحيل تمرير قيمة غير موجودة في الـ enum ✨
}
```

---

## لماذا نستخدم Enum؟

### 1. Type Safety (أمان النوع)
```java
// لا يمكن تمرير قيمة خاطئة
public void setPaymentMethod(PaymentMethod method) {
    // PaymentMethod.CASH أو CREDIT_CARD فقط
}
```

### 2. قابلية القراءة
```java
// ❌ ماذا يعني 2؟
if (order.getStatus() == 2) { ... }

// ✅ واضح تماماً
if (order.getStatus() == OrderStatus.SHIPPED) { ... }
```

### 3. دعم IDE
- Auto-complete للقيم المتاحة
- تحذيرات عند نسيان حالة في switch

### 4. إمكانية إضافة سلوك
```java
// Enum يمكن أن يحتوي على methods و properties
public enum Currency {
    EGP("جنيه مصري", "£"),
    SAR("ريال سعودي", "﷼"),
    USD("دولار أمريكي", "$");
    
    private final String arabicName;
    private final String symbol;
    // ...
}
```

---

## الصيغة الأساسية

### تعريف Enum بسيط

```java
public enum Priority {
    LOW,
    MEDIUM,
    HIGH,
    URGENT
}
```

### استخدام Enum

```java
public class Task {
    private String title;
    private Priority priority;
    
    public Task(String title, Priority priority) {
        this.title = title;
        this.priority = priority;
    }
    
    public Priority getPriority() {
        return priority;
    }
}

// الاستخدام
Task task = new Task("إصلاح Bug", Priority.HIGH);

// المقارنة
if (task.getPriority() == Priority.URGENT) {
    sendNotification();
}
```

---

## Enum مع Properties و Methods

### إضافة خصائص للـ Enum

```java
public enum HttpStatus {
    OK(200, "Success"),
    CREATED(201, "Created"),
    BAD_REQUEST(400, "Bad Request"),
    UNAUTHORIZED(401, "Unauthorized"),
    NOT_FOUND(404, "Not Found"),
    INTERNAL_ERROR(500, "Internal Server Error");
    
    // الخصائص (Fields)
    private final int code;
    private final String message;
    
    // Constructor (يجب أن يكون private أو package-private)
    HttpStatus(int code, String message) {
        this.code = code;
        this.message = message;
    }
    
    // Getters
    public int getCode() {
        return code;
    }
    
    public String getMessage() {
        return message;
    }
    
    // Method إضافية
    public boolean isError() {
        return code >= 400;
    }
    
    // Static method للبحث بالكود
    public static HttpStatus fromCode(int code) {
        for (HttpStatus status : values()) {
            if (status.code == code) {
                return status;
            }
        }
        throw new IllegalArgumentException("Unknown HTTP status code: " + code);
    }
}
```

### الاستخدام

```java
HttpStatus status = HttpStatus.NOT_FOUND;

System.out.println(status.getCode());      // 404
System.out.println(status.getMessage());   // Not Found
System.out.println(status.isError());      // true

// البحث بالكود
HttpStatus found = HttpStatus.fromCode(200);
System.out.println(found);  // OK
```

---

## Enum مع Constructor

### قواعد مهمة للـ Constructor

1. **يجب أن يكون `private` أو بدون modifier** (لا يمكن أن يكون `public` أو `protected`)
2. **يُستدعى تلقائياً** عند تحميل الـ Enum
3. **لا يمكن استدعاؤه يدوياً** - لا يمكنك عمل `new`

```java
public enum Season {
    // القيم مع معاملات الـ constructor
    SPRING("الربيع", 15, 25),
    SUMMER("الصيف", 30, 45),
    AUTUMN("الخريف", 15, 25),
    WINTER("الشتاء", 5, 15);
    
    private final String arabicName;
    private final int minTemp;
    private final int maxTemp;
    
    // Constructor - يُستدعى مرة واحدة لكل قيمة
    Season(String arabicName, int minTemp, int maxTemp) {
        this.arabicName = arabicName;
        this.minTemp = minTemp;
        this.maxTemp = maxTemp;
        System.out.println("Creating: " + this.name()); // للتوضيح فقط
    }
    
    public String getArabicName() {
        return arabicName;
    }
    
    public String getTemperatureRange() {
        return minTemp + "°C - " + maxTemp + "°C";
    }
}
```

```java
// عند أول استخدام للـ Enum، يُطبع:
// Creating: SPRING
// Creating: SUMMER
// Creating: AUTUMN
// Creating: WINTER

Season current = Season.SUMMER;
System.out.println(current.getArabicName());        // الصيف
System.out.println(current.getTemperatureRange()); // 30°C - 45°C
```

---

## Methods مهمة في Enum

كل Enum يرث من `java.lang.Enum` ويحصل على هذه الـ methods تلقائياً:

### 1. `name()` - اسم الثابت كـ String

```java
OrderStatus status = OrderStatus.PENDING;
String name = status.name();  // "PENDING"
```

### 2. `ordinal()` - الترتيب (يبدأ من 0)

```java
public enum Size { SMALL, MEDIUM, LARGE, XLARGE }

Size.SMALL.ordinal();   // 0
Size.MEDIUM.ordinal();  // 1
Size.LARGE.ordinal();   // 2
Size.XLARGE.ordinal();  // 3
```

> ⚠️ **تحذير:** لا تعتمد على `ordinal()` في المنطق البرمجي أو قواعد البيانات!
> إذا أضفت قيمة جديدة في المنتصف، ستتغير كل الأرقام.

### 3. `values()` - مصفوفة بكل القيم

```java
// الحصول على كل القيم
for (Day day : Day.values()) {
    System.out.println(day);
}

// تحويل إلى List
List<Day> days = Arrays.asList(Day.values());
```

### 4. `valueOf(String)` - التحويل من String

```java
// التحويل من نص
OrderStatus status = OrderStatus.valueOf("PENDING");

// ⚠️ يرمي IllegalArgumentException إذا كانت القيمة غير موجودة
OrderStatus invalid = OrderStatus.valueOf("UNKNOWN"); // 💥 Exception!
```

### 5. `toString()` - يمكن تخصيصه

```java
public enum PaymentMethod {
    CASH("نقدي"),
    CREDIT_CARD("بطاقة ائتمان"),
    BANK_TRANSFER("تحويل بنكي");
    
    private final String displayName;
    
    PaymentMethod(String displayName) {
        this.displayName = displayName;
    }
    
    @Override
    public String toString() {
        return displayName;
    }
}

// الاستخدام
System.out.println(PaymentMethod.CASH);           // نقدي
System.out.println(PaymentMethod.CASH.name());    // CASH
```

### 6. `compareTo()` - المقارنة (حسب الترتيب)

```java
Priority p1 = Priority.LOW;
Priority p2 = Priority.HIGH;

p1.compareTo(p2);  // رقم سالب (p1 قبل p2)
p2.compareTo(p1);  // رقم موجب (p2 بعد p1)
p1.compareTo(p1);  // 0 (متساويان)
```

---

## Enum مع Switch Statement

### Switch التقليدي

```java
public String getStatusMessage(OrderStatus status) {
    switch (status) {
        case PENDING:
            return "الطلب في انتظار المراجعة";
        case PROCESSING:
            return "جاري تجهيز الطلب";
        case SHIPPED:
            return "تم شحن الطلب";
        case DELIVERED:
            return "تم التوصيل";
        default:
            return "حالة غير معروفة";
    }
}
```

### Switch Expression (Java 14+)

```java
public String getStatusMessage(OrderStatus status) {
    return switch (status) {
        case PENDING -> "الطلب في انتظار المراجعة";
        case PROCESSING -> "جاري تجهيز الطلب";
        case SHIPPED -> "تم شحن الطلب";
        case DELIVERED -> "تم التوصيل";
    };
    // لا حاجة لـ default إذا غطيت كل الحالات!
}
```

### Switch مع كتل متعددة

```java
public int getShippingDays(ShippingMethod method) {
    return switch (method) {
        case EXPRESS -> 1;
        case STANDARD -> {
            System.out.println("الشحن العادي");
            yield 5;  // yield بدلاً من return داخل block
        }
        case ECONOMY -> 10;
    };
}
```

---

## Enum يُنفذ Interface

Enum يمكنه تنفيذ Interface واحد أو أكثر:

```java
// تعريف Interface
public interface Discountable {
    double getDiscountPercentage();
    String getDiscountDescription();
}

// Enum ينفذ الـ Interface
public enum CustomerType implements Discountable {
    REGULAR {
        @Override
        public double getDiscountPercentage() {
            return 0;
        }
        
        @Override
        public String getDiscountDescription() {
            return "لا يوجد خصم";
        }
    },
    SILVER {
        @Override
        public double getDiscountPercentage() {
            return 5;
        }
        
        @Override
        public String getDiscountDescription() {
            return "خصم 5% للعملاء الفضيين";
        }
    },
    GOLD {
        @Override
        public double getDiscountPercentage() {
            return 10;
        }
        
        @Override
        public String getDiscountDescription() {
            return "خصم 10% للعملاء الذهبيين";
        }
    },
    PLATINUM {
        @Override
        public double getDiscountPercentage() {
            return 15;
        }
        
        @Override
        public String getDiscountDescription() {
            return "خصم 15% للعملاء البلاتينيين";
        }
    }
}
```

### الاستخدام مع Polymorphism

```java
public double calculateTotal(double amount, Discountable discountable) {
    double discount = amount * (discountable.getDiscountPercentage() / 100);
    return amount - discount;
}

// الاستخدام
double total = calculateTotal(1000, CustomerType.GOLD);
System.out.println(total);  // 900.0
```

---

## Abstract Methods في Enum

يمكن تعريف abstract method في Enum وإجبار كل قيمة على تنفيذها:

```java
public enum Operation {
    ADD("+") {
        @Override
        public double apply(double a, double b) {
            return a + b;
        }
    },
    SUBTRACT("-") {
        @Override
        public double apply(double a, double b) {
            return a - b;
        }
    },
    MULTIPLY("×") {
        @Override
        public double apply(double a, double b) {
            return a * b;
        }
    },
    DIVIDE("÷") {
        @Override
        public double apply(double a, double b) {
            if (b == 0) throw new ArithmeticException("القسمة على صفر!");
            return a / b;
        }
    };
    
    private final String symbol;
    
    Operation(String symbol) {
        this.symbol = symbol;
    }
    
    public String getSymbol() {
        return symbol;
    }
    
    // Abstract method - كل قيمة يجب أن تنفذها
    public abstract double apply(double a, double b);
}
```

### الاستخدام

```java
double result = Operation.ADD.apply(10, 5);      // 15.0
double result2 = Operation.MULTIPLY.apply(4, 3); // 12.0

// استخدام في حاسبة
public double calculate(double a, double b, Operation op) {
    System.out.println(a + " " + op.getSymbol() + " " + b);
    return op.apply(a, b);
}

calculate(10, 5, Operation.ADD);  // 10 + 5 = 15.0
```

---

## EnumSet و EnumMap

Java توفر تطبيقات محسّنة للـ Set و Map خصيصاً للـ Enums:

### EnumSet - أداء عالي جداً

```java
import java.util.EnumSet;

public enum Permission {
    READ, WRITE, DELETE, EXECUTE, ADMIN
}

// إنشاء EnumSet
EnumSet<Permission> noPermissions = EnumSet.noneOf(Permission.class);
EnumSet<Permission> allPermissions = EnumSet.allOf(Permission.class);
EnumSet<Permission> somePermissions = EnumSet.of(Permission.READ, Permission.WRITE);

// نطاق من القيم
EnumSet<Permission> range = EnumSet.range(Permission.READ, Permission.DELETE);
// يحتوي: READ, WRITE, DELETE

// العكس (complement)
EnumSet<Permission> readOnly = EnumSet.of(Permission.READ);
EnumSet<Permission> notReadOnly = EnumSet.complementOf(readOnly);
// يحتوي: WRITE, DELETE, EXECUTE, ADMIN
```

### استخدام EnumSet للصلاحيات

```java
public class User {
    private String name;
    private EnumSet<Permission> permissions;
    
    public User(String name) {
        this.name = name;
        this.permissions = EnumSet.noneOf(Permission.class);
    }
    
    public void grantPermission(Permission permission) {
        permissions.add(permission);
    }
    
    public void revokePermission(Permission permission) {
        permissions.remove(permission);
    }
    
    public boolean hasPermission(Permission permission) {
        return permissions.contains(permission);
    }
    
    public boolean hasAllPermissions(EnumSet<Permission> required) {
        return permissions.containsAll(required);
    }
}

// الاستخدام
User admin = new User("أحمد");
admin.grantPermission(Permission.READ);
admin.grantPermission(Permission.WRITE);
admin.grantPermission(Permission.ADMIN);

if (admin.hasPermission(Permission.ADMIN)) {
    System.out.println("مستخدم مسؤول");
}
```

### EnumMap - خريطة بمفاتيح Enum

```java
import java.util.EnumMap;

public enum Day {
    SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY
}

// إنشاء EnumMap
EnumMap<Day, String> schedule = new EnumMap<>(Day.class);

schedule.put(Day.SUNDAY, "إجازة");
schedule.put(Day.MONDAY, "اجتماع الفريق");
schedule.put(Day.TUESDAY, "تطوير");
schedule.put(Day.WEDNESDAY, "مراجعة الكود");
schedule.put(Day.THURSDAY, "اختبار");
schedule.put(Day.FRIDAY, "إجازة");
schedule.put(Day.SATURDAY, "صيانة");

// الاستخدام
String todaySchedule = schedule.get(Day.MONDAY);
System.out.println(todaySchedule);  // اجتماع الفريق

// التكرار
for (Map.Entry<Day, String> entry : schedule.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

### مثال عملي: أسعار الشحن

```java
public enum Region {
    CAIRO, GIZA, ALEXANDRIA, DELTA, UPPER_EGYPT, SINAI
}

EnumMap<Region, Double> shippingCosts = new EnumMap<>(Region.class);
shippingCosts.put(Region.CAIRO, 25.0);
shippingCosts.put(Region.GIZA, 30.0);
shippingCosts.put(Region.ALEXANDRIA, 45.0);
shippingCosts.put(Region.DELTA, 50.0);
shippingCosts.put(Region.UPPER_EGYPT, 65.0);
shippingCosts.put(Region.SINAI, 80.0);

public double getShippingCost(Region region) {
    return shippingCosts.getOrDefault(region, 100.0);
}
```

---

## Enum في قواعد البيانات

### مع JPA/Hibernate

#### الطريقة الأولى: @Enumerated(EnumType.STRING) ✅ (مُفضّلة)

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String customerName;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "status", length = 20)
    private OrderStatus status;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "payment_method", length = 30)
    private PaymentMethod paymentMethod;
}
```

```sql
-- في قاعدة البيانات
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    customer_name VARCHAR(100),
    status VARCHAR(20),          -- 'PENDING', 'PROCESSING', etc.
    payment_method VARCHAR(30)   -- 'CASH', 'CREDIT_CARD', etc.
);
```

#### الطريقة الثانية: @Enumerated(EnumType.ORDINAL) ❌ (تجنبها!)

```java
@Enumerated(EnumType.ORDINAL)  // ❌ خطير!
private OrderStatus status;
```

```sql
-- في قاعدة البيانات
status INTEGER  -- 0, 1, 2, 3...
```

> ⚠️ **لماذا ORDINAL خطير؟**
> إذا أضفت قيمة جديدة في المنتصف، ستتغير كل الأرقام وستفسد البيانات!

#### الطريقة الثالثة: AttributeConverter (للتحكم الكامل)

```java
public enum DocumentType {
    INVOICE("INV"),
    CREDIT_NOTE("CN"),
    DEBIT_NOTE("DN"),
    RECEIPT("RCP");
    
    private final String code;
    
    DocumentType(String code) {
        this.code = code;
    }
    
    public String getCode() {
        return code;
    }
    
    public static DocumentType fromCode(String code) {
        for (DocumentType type : values()) {
            if (type.code.equals(code)) {
                return type;
            }
        }
        throw new IllegalArgumentException("Unknown code: " + code);
    }
}

// Converter
@Converter(autoApply = true)
public class DocumentTypeConverter implements AttributeConverter<DocumentType, String> {
    
    @Override
    public String convertToDatabaseColumn(DocumentType attribute) {
        return attribute == null ? null : attribute.getCode();
    }
    
    @Override
    public DocumentType convertToEntityAttribute(String dbData) {
        return dbData == null ? null : DocumentType.fromCode(dbData);
    }
}
```

```java
@Entity
public class Document {
    @Id
    private Long id;
    
    // سيستخدم الـ Converter تلقائياً
    @Column(name = "doc_type", length = 5)
    private DocumentType type;  // يُخزن كـ "INV", "CN", etc.
}
```

---

## أمثلة عملية من عالم ERP

### 1. حالة الفاتورة الإلكترونية (مصر)

```java
public enum EInvoiceStatus {
    DRAFT("مسودة", false),
    PENDING_SUBMISSION("في انتظار الإرسال", false),
    SUBMITTED("تم الإرسال", true),
    VALID("صالحة", true),
    INVALID("غير صالحة", true),
    CANCELLED("ملغاة", true),
    REJECTED("مرفوضة", true);
    
    private final String arabicName;
    private final boolean sentToTaxAuthority;
    
    EInvoiceStatus(String arabicName, boolean sentToTaxAuthority) {
        this.arabicName = arabicName;
        this.sentToTaxAuthority = sentToTaxAuthority;
    }
    
    public String getArabicName() {
        return arabicName;
    }
    
    public boolean isSentToTaxAuthority() {
        return sentToTaxAuthority;
    }
    
    public boolean canBeEdited() {
        return this == DRAFT;
    }
    
    public boolean canBeCancelled() {
        return this == VALID;
    }
    
    // الحالات التالية المسموحة
    public EnumSet<EInvoiceStatus> getAllowedTransitions() {
        return switch (this) {
            case DRAFT -> EnumSet.of(PENDING_SUBMISSION);
            case PENDING_SUBMISSION -> EnumSet.of(SUBMITTED, DRAFT);
            case SUBMITTED -> EnumSet.of(VALID, INVALID, REJECTED);
            case VALID -> EnumSet.of(CANCELLED);
            case INVALID, CANCELLED, REJECTED -> EnumSet.noneOf(EInvoiceStatus.class);
        };
    }
    
    public boolean canTransitionTo(EInvoiceStatus newStatus) {
        return getAllowedTransitions().contains(newStatus);
    }
}
```

### 2. نوع المخزون

```java
public enum InventoryTransactionType {
    // الإدخال
    PURCHASE("شراء", true, "PUR"),
    PURCHASE_RETURN("مرتجع شراء", false, "PURRET"),
    SALES_RETURN("مرتجع مبيعات", true, "SALRET"),
    TRANSFER_IN("تحويل وارد", true, "TRIN"),
    ADJUSTMENT_IN("تسوية إضافة", true, "ADJIN"),
    OPENING_BALANCE("رصيد افتتاحي", true, "OPEN"),
    
    // الإخراج
    SALES("مبيعات", false, "SAL"),
    TRANSFER_OUT("تحويل صادر", false, "TROUT"),
    ADJUSTMENT_OUT("تسوية خصم", false, "ADJOUT"),
    DAMAGE("تالف", false, "DMG"),
    CONSUMPTION("استهلاك", false, "CONS");
    
    private final String arabicName;
    private final boolean increasesStock;
    private final String code;
    
    InventoryTransactionType(String arabicName, boolean increasesStock, String code) {
        this.arabicName = arabicName;
        this.increasesStock = increasesStock;
        this.code = code;
    }
    
    public String getArabicName() {
        return arabicName;
    }
    
    public boolean increasesStock() {
        return increasesStock;
    }
    
    public String getCode() {
        return code;
    }
    
    public int getStockMultiplier() {
        return increasesStock ? 1 : -1;
    }
    
    public static InventoryTransactionType fromCode(String code) {
        for (InventoryTransactionType type : values()) {
            if (type.code.equals(code)) {
                return type;
            }
        }
        throw new IllegalArgumentException("Unknown transaction code: " + code);
    }
}
```

### استخدام في حساب المخزون

```java
public class InventoryService {
    
    public void processTransaction(InventoryTransaction transaction) {
        int quantityChange = transaction.getQuantity() 
                           * transaction.getType().getStockMultiplier();
        
        updateStock(transaction.getProduct(), quantityChange);
        
        // Log
        System.out.printf("تم %s %d وحدة من %s%n",
            transaction.getType().getArabicName(),
            transaction.getQuantity(),
            transaction.getProduct().getName()
        );
    }
}
```

### 3. طريقة الدفع مع التكامل

```java
public enum PaymentMethod {
    CASH("نقدي", "CASH", true, false) {
        @Override
        public PaymentProcessor getProcessor() {
            return new CashPaymentProcessor();
        }
    },
    CREDIT_CARD("بطاقة ائتمان", "CC", false, true) {
        @Override
        public PaymentProcessor getProcessor() {
            return new CreditCardProcessor();
        }
    },
    BANK_TRANSFER("تحويل بنكي", "BT", false, true) {
        @Override
        public PaymentProcessor getProcessor() {
            return new BankTransferProcessor();
        }
    },
    INSTAPAY("انستاباي", "INSTA", false, true) {
        @Override
        public PaymentProcessor getProcessor() {
            return new InstapayProcessor();
        }
    },
    CHEQUE("شيك", "CHQ", false, false) {
        @Override
        public PaymentProcessor getProcessor() {
            return new ChequeProcessor();
        }
    };
    
    private final String arabicName;
    private final String code;
    private final boolean requiresNoVerification;
    private final boolean electronic;
    
    PaymentMethod(String arabicName, String code, 
                  boolean requiresNoVerification, boolean electronic) {
        this.arabicName = arabicName;
        this.code = code;
        this.requiresNoVerification = requiresNoVerification;
        this.electronic = electronic;
    }
    
    public abstract PaymentProcessor getProcessor();
    
    public String getArabicName() { return arabicName; }
    public String getCode() { return code; }
    public boolean requiresNoVerification() { return requiresNoVerification; }
    public boolean isElectronic() { return electronic; }
    
    // الحصول على طرق الدفع الإلكترونية فقط
    public static EnumSet<PaymentMethod> getElectronicMethods() {
        return EnumSet.of(CREDIT_CARD, BANK_TRANSFER, INSTAPAY);
    }
}
```

### 4. نوع ضريبة القيمة المضافة

```java
public enum VATType {
    STANDARD("قيمة مضافة عادية", new BigDecimal("14.00"), "T1"),
    ZERO_RATED("نسبة صفر", BigDecimal.ZERO, "T2"),
    EXEMPT("معفى", BigDecimal.ZERO, "T3"),
    NOT_SUBJECT("غير خاضع", BigDecimal.ZERO, "T4"),
    EXPORT("تصدير", BigDecimal.ZERO, "T5");
    
    private final String arabicName;
    private final BigDecimal rate;
    private final String taxAuthorityCode;
    
    VATType(String arabicName, BigDecimal rate, String taxAuthorityCode) {
        this.arabicName = arabicName;
        this.rate = rate;
        this.taxAuthorityCode = taxAuthorityCode;
    }
    
    public BigDecimal getRate() {
        return rate;
    }
    
    public BigDecimal calculateTax(BigDecimal amount) {
        return amount.multiply(rate).divide(new BigDecimal("100"), 2, RoundingMode.HALF_UP);
    }
    
    public String getArabicName() {
        return arabicName;
    }
    
    public String getTaxAuthorityCode() {
        return taxAuthorityCode;
    }
}
```

---

## أفضل الممارسات

### 1. استخدم أسماء واضحة ومعبّرة

```java
// ❌ سيء
public enum S { P, C, S, D }

// ✅ جيد
public enum OrderStatus { PENDING, CONFIRMED, SHIPPED, DELIVERED }
```

### 2. استخدم SCREAMING_SNAKE_CASE للقيم

```java
// ❌ سيء
public enum Color { red, Blue, DARK_green }

// ✅ جيد
public enum Color { RED, BLUE, DARK_GREEN }
```

### 3. لا تعتمد على ordinal()

```java
// ❌ سيء - سينكسر إذا أضفت قيمة جديدة
if (status.ordinal() == 2) { ... }

// ✅ جيد
if (status == OrderStatus.SHIPPED) { ... }
```

### 4. استخدم @Enumerated(EnumType.STRING) مع JPA

```java
// ❌ سيء - سينكسر البيانات
@Enumerated(EnumType.ORDINAL)
private Status status;

// ✅ جيد - آمن ومقروء
@Enumerated(EnumType.STRING)
private Status status;
```

### 5. أضف method للتحويل من String بأمان

```java
public enum Status {
    ACTIVE, INACTIVE;
    
    // ✅ تحويل آمن
    public static Optional<Status> fromString(String value) {
        try {
            return Optional.of(Status.valueOf(value.toUpperCase()));
        } catch (IllegalArgumentException e) {
            return Optional.empty();
        }
    }
}

// الاستخدام
Status.fromString("active").ifPresent(status -> {
    // استخدم الـ status
});
```

### 6. استخدم EnumSet بدلاً من List للمجموعات

```java
// ❌ أبطأ وأقل كفاءة
List<Permission> permissions = new ArrayList<>();
permissions.add(Permission.READ);

// ✅ محسّن للـ Enum
EnumSet<Permission> permissions = EnumSet.of(Permission.READ, Permission.WRITE);
```

### 7. وثّق القيم بـ Javadoc

```java
public enum InvoiceType {
    /**
     * فاتورة مبيعات عادية للعملاء
     */
    SALES,
    
    /**
     * إشعار خصم للعميل
     */
    CREDIT_NOTE,
    
    /**
     * إشعار إضافة على العميل
     */
    DEBIT_NOTE
}
```

---

## أخطاء شائعة يجب تجنبها

### 1. ❌ محاولة إنشاء Enum بـ new

```java
// ❌ خطأ في الترجمة!
OrderStatus status = new OrderStatus();

// ✅ صحيح
OrderStatus status = OrderStatus.PENDING;
```

### 2. ❌ مقارنة Enum بـ equals() بدون داعي

```java
// ❌ غير ضروري (يعمل لكن...)
if (status.equals(OrderStatus.PENDING)) { ... }

// ✅ أبسط وأسرع
if (status == OrderStatus.PENDING) { ... }
```

### 3. ❌ استخدام valueOf() بدون التحقق

```java
// ❌ قد يرمي Exception
OrderStatus status = OrderStatus.valueOf(userInput);

// ✅ آمن
try {
    OrderStatus status = OrderStatus.valueOf(userInput.toUpperCase());
} catch (IllegalArgumentException e) {
    // قيمة غير صالحة
}

// ✅ أو استخدم Optional
public static Optional<OrderStatus> safeValueOf(String name) {
    try {
        return Optional.of(OrderStatus.valueOf(name));
    } catch (IllegalArgumentException e) {
        return Optional.empty();
    }
}
```

### 4. ❌ تغيير قيم الـ Fields بعد الإنشاء

```java
// ❌ سيء - Enum يجب أن يكون immutable
public enum BadEnum {
    VALUE;
    private String mutableField;  // ❌
    public void setField(String value) { this.mutableField = value; }  // ❌
}

// ✅ جيد - جميع الـ fields نهائية
public enum GoodEnum {
    VALUE("fixed");
    private final String immutableField;  // ✅
    GoodEnum(String field) { this.immutableField = field; }
}
```

### 5. ❌ نسيان حالة في Switch بدون default

```java
// ❌ خطير - إذا أضفت قيمة جديدة لن يحذرك المترجم
switch (status) {
    case PENDING: return "...";
    case PROCESSING: return "...";
    // نسيت SHIPPED و DELIVERED!
}

// ✅ Java 14+ - المترجم يفرض تغطية كل الحالات
return switch (status) {
    case PENDING -> "...";
    case PROCESSING -> "...";
    case SHIPPED -> "...";
    case DELIVERED -> "...";
};
```

---

## ملخص سريع

| الميزة | الوصف |
|--------|-------|
| `name()` | اسم الثابت كـ String |
| `ordinal()` | ترتيب الثابت (لا تستخدمه!) |
| `values()` | مصفوفة بكل القيم |
| `valueOf(String)` | تحويل String إلى Enum |
| `EnumSet` | مجموعة محسّنة للـ Enum |
| `EnumMap` | خريطة بمفاتيح Enum |
| `@Enumerated(STRING)` | حفظ في قاعدة البيانات كنص |
| `AttributeConverter` | تحكم كامل في التخزين |

---

## مراجع إضافية

- [Java Enum Documentation](https://docs.oracle.com/en/java/javase/17/language/enum-types.html)
- [Effective Java - Item 34: Use enums instead of int constants](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Baeldung - Java Enum](https://www.baeldung.com/a-guide-to-java-enums)
