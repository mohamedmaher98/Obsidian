# Java Collections – Summary

## 1. List
### الفكرة
- Collection **مرتبة**
- تسمح بـ **التكرار**
- الوصول بالعناصر عن طريق **index**

### إمتى أستخدمها؟
- لما الترتيب يفرق
- لما ينفع العنصر يتكرر
- لما أحتاج أجيب عنصر برقم

### أشهر Implementations
- ArrayList
- LinkedList
- Vector (قديم)

### أمثلة استخدام
- API Responses
- Results من Database
- Pagination
- Playlists

---

## 2. Set
### الفكرة
- Collection **بدون تكرار**
- كل عنصر **unique**
- الترتيب غالبًا مش مهم

### إمتى أستخدمها؟
- لما أمنع التكرار
- لما يهمني uniqueness أكتر من الترتيب

### أشهر Implementations
- HashSet
- LinkedHashSet (يحافظ على الترتيب)
- TreeSet (مرتب)

### أمثلة استخدام
- Roles في Spring Security
- Permissions
- Emails
- IDs

---

## 3. Queue
### الفكرة
- ال Collection مبنية على **الدور**
- ال First In First Out (FIFO)

### إمتى أستخدمها؟
- لما يكون في processing بالترتيب
- لما أول واحد يدخل يطلع الأول

### أشهر Implementations
- LinkedList
- PriorityQueue
- ArrayDeque

### أمثلة استخدام
- Task processing
- Background jobs
- Message Queues
- Async operations

---

## ملخص سريع (Cheat Sheet)

| Collection | ترتيب | تكرار | فكرة |
|----------|------|-------|------|
| List | ✅ | ✅ | ترتيب |
| Set | ❌ | ❌ | تمييز |
| Queue | ✅ | ❌ | دور |

### قاعدة ذهبية
- الList = ترتيب
- الSet = uniqueness
- ال Queue = دور
