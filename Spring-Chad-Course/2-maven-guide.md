 # 📦 دليلك الشامل لـ Maven

> شرح مبسط من الصفر — لو دي أول مرة تسمع عن Maven، أنت في المكان الصح.

---

## 🤔 أولاً: ما هو Maven؟

تخيل إنك بتبني بيت. محتاج طوب، إسمنت، حديد، نجار، سباك... كل ده لازم تجيبه وترتبه وتتأكد إن كل حاجة متوافقة مع بعض.

**Maven** بالظبط زي مقاول البيت ده — بس في عالم Java.

هو **أداة لإدارة المشاريع وبناءها** (Build Tool & Project Management Tool). مهمته إنه:

- **يجيبلك الـ Libraries** اللي محتاجها (Dependencies) من الإنترنت أوتوماتيك
- **يبني (Build) المشروع** بترتيب صح
- **يشغّل الـ Tests** أوتوماتيك
- **يعمل Package** للمشروع (JAR / WAR)
- **يرفع المشروع (Deploy)** على السيرفر

بدل Maven كنت هتعمل كل ده إيدين! 😩

---

## 📁 ثانياً: هيكل مشروع Maven (Project Structure)

لما تعمل مشروع Maven، هيكون ليه شكل موحد ثابت — وده من أجمل حاجة في Maven، لأن أي حد يشتغل بيه هيفهم المشروع فوراً.

```
my-project/
│
├── 📄 pom.xml                         ← ده قلب المشروع (هنشرحه بالتفصيل)
│
├── 📁 src/
│   ├── 📁 main/
│   │   ├── 📁 java/                   ← كود Java بتاعك هنا
│   │   │   └── com/namasoft/myapp/
│   │   │       └── Main.java
│   │   │
│   │   └── 📁 resources/             ← ملفات الإعدادات (application.properties, XML...)
│   │       └── application.properties
│   │
│   └── 📁 test/
│       ├── 📁 java/                   ← كود الـ Tests بتاعك هنا
│       │   └── com/namasoft/myapp/
│       │       └── MainTest.java
│       │
│       └── 📁 resources/             ← ملفات إعدادات خاصة بالـ Tests
│
└── 📁 target/                        ← Maven بيحطّ هنا كل اللي يبنيه (مش بتلمسه أنت)
    ├── classes/
    ├── test-classes/
    └── my-project-1.0.jar
```

### شرح كل مجلد:

| المجلد | الغرض |
|--------|--------|
| `src/main/java` | كود الـ Production بتاعك — اللي هيتشغّل فعلاً |
| `src/main/resources` | ملفات الإعدادات (مش كود Java) |
| `src/test/java` | كود الـ Unit Tests |
| `src/test/resources` | إعدادات خاصة بالـ Tests |
| `target/` | Maven بيعملها أوتوماتيك لما تعمل Build |
| `pom.xml` | ملف إعداد Maven — الأهم على الإطلاق |

> 💡 **نصيحة:** الـ `target/` مش بتتحطش على Git (بتحطها في .gitignore) لأن Maven يعيد إنشاءها في أي وقت.

---

## 🧠 ثالثاً: المفاهيم الأساسية في Maven (Key Concepts)

### 1️⃣ pom.xml — قلب المشروع

**POM = Project Object Model**

ده ملف XML واحد بيقول لـ Maven كل حاجة عن مشروعك. خليك تفكر فيه كـ "بطاقة هوية + قائمة مشتريات + خطة عمل" للمشروع كله.

```xml
<project>
    
    <!-- 🪪 هوية المشروع -->
    <groupId>com.namasoft</groupId>       <!-- اسم شركتك / مؤسستك -->
    <artifactId>nama-erp</artifactId>     <!-- اسم المشروع نفسه -->
    <version>1.0.0</version>              <!-- رقم الإصدار -->
    <packaging>jar</packaging>            <!-- نوع الـ Output: jar أو war -->

    <!-- 📦 المكتبات اللي محتاجها (Dependencies) -->
    <dependencies>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.27</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>   <!-- ده بس للـ Tests مش هيتحط في الـ JAR النهائي -->
        </dependency>

    </dependencies>

    <!-- ⚙️ إعدادات البناء -->
    <build>
        <plugins>
            <!-- Plugins إضافية لو احتجت -->
        </plugins>
    </build>

</project>
```

---

### 2️⃣ Coordinates — عنوان كل مكتبة

كل مكتبة في عالم Maven ليها "عنوان" فريد مكوّن من 3 حاجات:

```
groupId : artifactId : version
```

مثال:
```
org.springframework : spring-core : 5.3.27
```

ده بيضمن إنك دايماً تجيب المكتبة الصح بالإصدار الصح.

---

### 3️⃣ Dependencies — إدارة المكتبات

ده أهم ميزة في Maven. بدله كنت:
- تدور على الـ JAR على النت
- تحمّله يدوي
- تحطه في المشروع
- تتأكد إنه متوافق مع باقي المكتبات

**مع Maven:** بتكتب 4 أسطر XML وبس — وهو بيجيب كل حاجة أوتوماتيك من الـ Maven Central Repository (زي متجر للمكتبات على الإنترنت).

#### الـ Scope في الـ Dependencies:

| Scope | معناه |
|-------|--------|
| `compile` | (الافتراضي) — متاح في كل مراحل البناء والتشغيل |
| `test` | بس في وقت الـ Tests — مش بيتحط في الـ JAR النهائي |
| `provided` | موجود في السيرفر (زي Servlet API في Tomcat) — مش بتحطه في الـ JAR |
| `runtime` | محتاجه وقت التشغيل بس مش وقت الـ Compile |

---

### 4️⃣ Repositories — مخازن المكتبات

Maven بيدور على المكتبات في ترتيب معين:

```
1. Local Repository   ← على جهازك: ~/.m2/repository
         ↓ (مش لاقيه؟)
2. Remote Repository  ← Maven Central على الإنترنت
         ↓ (أو)
3. Private Repository ← سيرفر Nexus / Artifactory جوّا شركتك
```

> 💡 أول ما Maven يحمّل مكتبة، بيحطها في الـ Local Repository — فالمرة الجاية بيلاقيها على جهازك فوراً بدون إنترنت.

---

### 5️⃣ Build Lifecycle — رحلة البناء

Maven عنده 3 Lifecycles رئيسية. الأهم هو **default lifecycle** وبيمر بالمراحل دي بالترتيب:

```
validate → compile → test → package → verify → install → deploy
```

| المرحلة | بتعمل إيه؟ |
|---------|------------|
| `validate` | بيتحقق إن الـ pom.xml صح |
| `compile` | بيحوّل كود Java لـ .class |
| `test` | بيشغّل الـ Unit Tests |
| `package` | بيلمّ كل حاجة في JAR أو WAR |
| `install` | بيحط الـ JAR في الـ Local Repository |
| `deploy` | بيرفعه على الـ Remote Repository |

> 🔑 **قاعدة مهمة:** لما بتشغّل مرحلة، Maven بيشغّل كل المراحل اللي قبلها أوتوماتيك.
> يعني `mvn package` = validate + compile + test + package كلهم.

#### أشهر الأوامر:

```bash
mvn compile          # بناء الكود بس
mvn test             # تشغيل الـ Tests
mvn package          # عمل JAR/WAR
mvn install          # package + حطّه في Local Repository
mvn clean            # مسح مجلد target (بداية نظيفة)
mvn clean install    # أشهر أمر — نظّف وابدأ من الأول
mvn clean install -DskipTests  # نفسه بس تجاهل الـ Tests (اسرع)
```

---

### 6️⃣ Plugins — قدرات إضافية

Maven نفسه خفيف، بس بيتوسّع بـ Plugins. كل حاجة Maven بيعملها هي في الأصل Plugin.

```xml
<build>
    <plugins>

        <!-- Plugin عشان تحدد إصدار Java -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>17</source>
                <target>17</target>
            </configuration>
        </plugin>

        <!-- Plugin عشان تعمل Executable JAR (Spring Boot) -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>

    </plugins>
</build>
```

---

### 7️⃣ Inheritance & Multi-Module — المشاريع الكبيرة

لو مشروعك كبير (زي Nama ERP 😉) ممكن تقسمه لـ Modules:

```
nama-erp/                    ← Parent Project
├── pom.xml                  ← Parent POM (الإعدادات المشتركة هنا)
├── nama-core/               ← Module 1
│   └── pom.xml
├── nama-hr/                 ← Module 2
│   └── pom.xml
├── nama-accounting/         ← Module 3
│   └── pom.xml
└── nama-web/                ← Module 4 (WAR)
    └── pom.xml
```

الـ Parent POM بيعرّف الـ Dependencies والإصدارات المشتركة مرة واحدة — والـ Modules بترثها. ده بيمنع تكرار الإصدارات في كل مكان.

---

## ⚖️ رابعاً: Maven vs Gradle — الفرق بالتفصيل

### نظرة سريعة

| | Maven | Gradle |
|-|-------|--------|
| **اللغة** | XML (pom.xml) | Groovy أو Kotlin DSL (build.gradle) |
| **عمره** | 2004 | 2012 |
| **السرعة** | متوسطة | **أسرع بكتير** |
| **المرونة** | محدودة | **مرن جداً** |
| **الصعوبة** | **سهل وواضح** | أصعب شوية في البداية |
| **الانتشار** | واسع جداً (Enterprise) | واسع (Android إجباري) |

---

### الفرق في طريقة الكتابة

**Maven (XML) — واضح لكن كتير:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.1.0</version>
</dependency>
```

**Gradle (Groovy DSL) — أقصر وأسرع:**
```groovy
implementation 'org.springframework.boot:spring-boot-starter-web:3.1.0'
```

نفس الحاجة تماماً — بس Gradle أقل كلاماً.

---

### الفرق في السرعة

Gradle **أسرع بكتير** من Maven وده بسبب:

| الميزة | Maven | Gradle |
|--------|-------|--------|
| **Incremental Build** | محدود | ✅ بيبني بس اللي اتغيّر |
| **Build Cache** | ❌ | ✅ بيحفظ نتايج قديمة ويعيد استخدامها |
| **Parallel Execution** | محدود | ✅ بيشغّل Tasks بالتوازي |
| **Daemon** | ❌ | ✅ Process في الخلفية بيسرّع كل run |

في مشاريع كبيرة، Gradle ممكن يكون **أسرع 2-10 أضعاف** من Maven.

---

### الفرق في المرونة

**Maven** بيتبع **Convention over Configuration** — يعني في طريقة واحدة "صح" للأمور. ده ميزة لأنه بيوحّد المشاريع، لكنه قيد لو عايز تعمل حاجة غير تقليدية.

**Gradle** بيديك **حرية كاملة** — تقدر تكتب منطق Build معقد، تعمل Custom Tasks، تتحكم في كل تفصيلة. لكن ده بيجي بثمن — ممكن يبقى الـ build.gradle معقد ومش مفهوم.

---

### متى تختار Maven؟ ومتى تختار Gradle?

#### ✅ اختار Maven لو:
- المشروع Enterprise ومحتاج استقرار (زي Nama ERP ✅)
- الفريق مختلط والناس محتاجة تفهم الـ Build بسهولة
- بتشتغل مع Spring Boot (بيدعم الاتنين بس Maven الأكثر شيوعاً فيه)
- محتاج Reproducible Builds (نفس النتيجة دايماً)
- الـ CI/CD Pipeline محتاج يكون بسيط وواضح

#### ✅ اختار Gradle لو:
- بتعمل Android Development (إجباري تقريباً)
- المشروع ضخم ومحتاج بناء سريع
- عندك منطق Build معقد ومحتاج مرونة عالية
- الفريق متمرس ومش هيتوه في الـ DSL
- بتشتغل مع Kotlin (Gradle's Kotlin DSL ممتاز)

---

### مقارنة الـ Multi-Module

**Maven:**
```xml
<!-- في الـ Parent pom.xml -->
<modules>
    <module>nama-core</module>
    <module>nama-hr</module>
    <module>nama-web</module>
</modules>
```

**Gradle:**
```groovy
// في settings.gradle
include 'nama-core', 'nama-hr', 'nama-web'
```

---

## 🎯 خلاصة سريعة

```
Maven  = مقاول منظم ومتوقع — بيعمل نفس الحاجة كل مرة بنفس الطريقة
Gradle = مقاول موهوب وسريع — بيتكيف مع كل حاجة بس محتاج خبرة أكتر
```

| السؤال | الإجابة |
|--------|---------|
| Maven بييجي منين؟ | Apache Software Foundation |
| الملف الأساسي؟ | `pom.xml` |
| فين بيحمّل المكتبات؟ | `~/.m2/repository` (Local) ثم Maven Central |
| أشهر أمر؟ | `mvn clean install` |
| مقابله؟ | Gradle (أسرع وأمرن) أو Ant (قديم) |
| مين بيستخدم Maven؟ | معظم مشاريع Java Enterprise وSpring |
| مين بيستخدم Gradle؟ | Android + مشاريع كبيرة محتاجة سرعة |

---

> 📌 **نصيحة أخيرة:** لو شغّال على Nama ERP (Java + Spring + Tomcat)، Maven هو الاختيار الطبيعي والأكثر استقراراً لمشاريع Enterprise زيّه. الـ Convention اللي Maven بيفرضه ده مش قيد — ده نعمة لفريق كبير. 😄
