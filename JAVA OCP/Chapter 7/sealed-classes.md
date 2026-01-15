# Sealed Classes - شرح بالمصري 🔒

## يعني إيه Sealed Class؟

تخيل إنك عندك كلاس وعايز تتحكم **مين بالظبط** اللي يقدر يعمل منه extend أو implement. يعني مش أي حد ييجي يورث منك، لأ - إنت بتحدد قايمة معينة من الكلاسات اللي مسموحلها بس.

الموضوع ده اتضاف في **Java 17** كـ feature رسمية.

---

## الفرق بين الأنواع المختلفة

| النوع | الوضع |
|-------|-------|
| `class` عادي | أي حد يقدر يورث منه |
| `final class` | محدش خالص يقدر يورث منه |
| `sealed class` | بس الكلاسات اللي إنت محددها تقدر تورث منه |

---

## Syntax بتاعته

```java
public sealed class Shape 
    permits Circle, Rectangle, Triangle {
    // الكود بتاعك هنا
}
```

الكلمة `permits` دي بتقول: "بس الـ 3 كلاسات دول اللي مسموحلهم يورثوا مني"

---

## الكلاسات اللي بتورث لازم تكون إيه؟

أي كلاس بيورث من sealed class لازم يكون واحد من 3:

### 1. `final` - يعني خلاص كده مفيش حد تاني يورث منه

```java
public final class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    public double getArea() {
        return Math.PI * radius * radius;
    }
}
```

### 2. `sealed` - يعني هو كمان هيحدد مين يورث منه

```java
public sealed class Rectangle extends Shape 
    permits Square, FilledRectangle {
    
    protected double width, height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
}
```

### 3. `non-sealed` - يعني فاتح الباب لأي حد يورث منه

```java
public non-sealed class Triangle extends Shape {
    // أي حد يقدر يورث من Triangle
}
```

---

## مثال عملي كامل - نظام الدفع 💳

تخيل إنك بتعمل نظام دفع في الـ ERP وعايز تتحكم في أنواع الدفع:

### الكلاس الأب

```java
public sealed abstract class PaymentMethod 
    permits CashPayment, CardPayment, BankTransfer, WalletPayment {
    
    protected double amount;
    protected LocalDateTime timestamp;
    
    public PaymentMethod(double amount) {
        this.amount = amount;
        this.timestamp = LocalDateTime.now();
    }
    
    public abstract boolean process();
    public abstract String getReceipt();
}
```

### دفع كاش - final يعني مفيش أنواع تانية من الكاش

```java
public final class CashPayment extends PaymentMethod {
    private double receivedAmount;
    
    public CashPayment(double amount, double receivedAmount) {
        super(amount);
        this.receivedAmount = receivedAmount;
    }
    
    @Override
    public boolean process() {
        return receivedAmount >= amount;
    }
    
    @Override
    public String getReceipt() {
        double change = receivedAmount - amount;
        return String.format("كاش: %.2f - الباقي: %.2f", amount, change);
    }
}
```

### الكارت - sealed لأن فيه أنواع محددة من الكروت

```java
public sealed class CardPayment extends PaymentMethod 
    permits VisaPayment, MastercardPayment {
    
    protected String cardNumber;
    protected String cardHolder;
    
    public CardPayment(double amount, String cardNumber, String cardHolder) {
        super(amount);
        this.cardNumber = cardNumber;
        this.cardHolder = cardHolder;
    }
    
    @Override
    public boolean process() {
        return cardNumber != null && cardNumber.length() == 16;
    }
    
    @Override
    public String getReceipt() {
        String maskedCard = "****" + cardNumber.substring(12);
        return String.format("كارت: %s - المبلغ: %.2f", maskedCard, amount);
    }
}
```

### Visa Payment

```java
public final class VisaPayment extends CardPayment {
    public VisaPayment(double amount, String cardNumber, String cardHolder) {
        super(amount, cardNumber, cardHolder);
    }
    
    @Override
    public boolean process() {
        return super.process() && cardNumber.startsWith("4");
    }
}
```

### Mastercard Payment

```java
public final class MastercardPayment extends CardPayment {
    public MastercardPayment(double amount, String cardNumber, String cardHolder) {
        super(amount, cardNumber, cardHolder);
    }
    
    @Override
    public boolean process() {
        return super.process() && 
               (cardNumber.startsWith("51") || cardNumber.startsWith("52"));
    }
}
```

### تحويل بنكي - non-sealed لأن ممكن يكون فيه أنواع كتير

```java
public non-sealed class BankTransfer extends PaymentMethod {
    private String bankCode;
    private String accountNumber;
    
    public BankTransfer(double amount, String bankCode, String accountNumber) {
        super(amount);
        this.bankCode = bankCode;
        this.accountNumber = accountNumber;
    }
    
    @Override
    public boolean process() {
        return bankCode != null && accountNumber != null;
    }
    
    @Override
    public String getReceipt() {
        return String.format("تحويل بنكي: %s - المبلغ: %.2f", bankCode, amount);
    }
}
```

### المحافظ الإلكترونية

```java
public final class WalletPayment extends PaymentMethod {
    private String walletProvider; // vodafone, etisalat, fawry
    private String phoneNumber;
    
    public WalletPayment(double amount, String provider, String phone) {
        super(amount);
        this.walletProvider = provider;
        this.phoneNumber = phone;
    }
    
    @Override
    public boolean process() {
        return phoneNumber != null && phoneNumber.matches("^01[0125][0-9]{8}$");
    }
    
    @Override
    public String getReceipt() {
        return String.format("%s: %s - المبلغ: %.2f", 
                           walletProvider, phoneNumber, amount);
    }
}
```

---

## الـ Pattern Matching مع Sealed Classes 🎯

أحلى حاجة في sealed classes إنها بتشتغل حلو جداً مع switch expressions:

```java
public class PaymentProcessor {
    
    public String processPayment(PaymentMethod payment) {
        // الcompiler عارف كل الأنواع الممكنة!
        return switch (payment) {
            case CashPayment cash -> {
                if (cash.process()) {
                    yield "✅ " + cash.getReceipt();
                }
                yield "❌ المبلغ المدفوع مش كافي";
            }
            
            case VisaPayment visa -> {
                if (visa.process()) {
                    yield "✅ Visa: " + visa.getReceipt();
                }
                yield "❌ كارت Visa مش صالح";
            }
            
            case MastercardPayment mc -> {
                if (mc.process()) {
                    yield "✅ Mastercard: " + mc.getReceipt();
                }
                yield "❌ كارت Mastercard مش صالح";
            }
            
            case CardPayment card -> {
                // ده هيمسك أي CardPayment تاني لو فيه
                yield "كارت عام: " + card.getReceipt();
            }
            
            case BankTransfer transfer -> {
                if (transfer.process()) {
                    yield "✅ " + transfer.getReceipt();
                }
                yield "❌ بيانات التحويل غلط";
            }
            
            case WalletPayment wallet -> {
                if (wallet.process()) {
                    yield "✅ " + wallet.getReceipt();
                }
                yield "❌ رقم الموبايل غلط";
            }
        };
        // مفيش default لأن الcompiler عارف إن دول كل الحالات!
    }
}
```

---

## مثال تاني - حالات الفاتورة في الـ E-Invoice 📄

### تعريف الـ States

```java
public sealed interface InvoiceState 
    permits Draft, Submitted, Accepted, Rejected, Cancelled {
}

public record Draft(LocalDateTime createdAt, String createdBy) 
    implements InvoiceState {}

public record Submitted(LocalDateTime submittedAt, String submissionId) 
    implements InvoiceState {}

public record Accepted(String uuid, LocalDateTime acceptedAt) 
    implements InvoiceState {}

public record Rejected(String uuid, List<String> errors, LocalDateTime rejectedAt) 
    implements InvoiceState {}

public record Cancelled(String reason, LocalDateTime cancelledAt) 
    implements InvoiceState {}
```

### الاستخدام

```java
public class InvoiceService {
    
    public String getStateDescription(InvoiceState state) {
        return switch (state) {
            case Draft d -> "مسودة - اتعملت في " + d.createdAt();
            case Submitted s -> "مرسلة - رقم الإرسال: " + s.submissionId();
            case Accepted a -> "مقبولة - UUID: " + a.uuid();
            case Rejected r -> "مرفوضة - الأخطاء: " + String.join(", ", r.errors());
            case Cancelled c -> "ملغية - السبب: " + c.reason();
        };
    }
    
    public boolean canEdit(InvoiceState state) {
        return state instanceof Draft;
    }
    
    public boolean canCancel(InvoiceState state) {
        return switch (state) {
            case Draft d -> true;
            case Submitted s -> true;  // ممكن قبل القبول
            case Accepted a -> true;   // بس بإجراءات معينة
            case Rejected r -> false;
            case Cancelled c -> false;
        };
    }
}
```

---

## ليه نستخدم Sealed Classes؟

### 1. Type Safety

الـ compiler بيعرف كل الأنواع الممكنة، فبيقدر يتأكد إنك handle-ت كل الحالات.

### 2. Better Design

بتعبر عن الـ domain بتاعك بشكل أوضح - "أنواع الدفع عندنا بس 4"

### 3. Exhaustive Switch

مش محتاج `default` case لأن الcompiler عارف كل الحالات

### 4. Documentation

الكود نفسه بيوثق إيه الأنواع المسموح بيها

### 5. Maintainability

لو ضفت نوع جديد، الcompiler هيقولك على كل الأماكن اللي محتاج تعدلها

---

## قواعد مهمة لازم تعرفها ⚠️

### القاعدة 1: نفس الـ Package

الكلاسات المسموحة لازم تكون في نفس الـ package أو module

```java
// ❌ ده مش هيشتغل لو Circle في package تاني
```

### القاعدة 2: لازم يورث فعلاً

لازم كل permitted class يورث فعلاً من الـ sealed class

```java
// ❌ لو كتبت permits Circle وCircle مش بيextend منك - error
```

### القاعدة 3: نفس الملف

لو الكلاسات في نفس الملف، مش لازم permits

```java
public sealed class Shape {
    // permits مش لازم لأن الكلاسات تحت في نفس الملف
}

final class Circle extends Shape {}
final class Rectangle extends Shape {}
```

### القاعدة 4: Sealed Interface

```java
public sealed interface Drawable permits Circle, Rectangle {}
```

---

## الخلاصة

Sealed Classes بتديك تحكم كامل في الـ inheritance hierarchy بتاعتك. بدل ما تسيب الكلاس مفتوح لأي حد يورث منه، إنت بتحدد بالظبط مين المسموحله. ده بيخلي الكود أكتر أمان وأسهل في الصيانة، خصوصاً لما تيجي تستخدم pattern matching مع switch.

---

## مراجع مفيدة

- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
- [Oracle Java Documentation](https://docs.oracle.com/en/java/javase/17/language/sealed-classes-and-interfaces.html)
