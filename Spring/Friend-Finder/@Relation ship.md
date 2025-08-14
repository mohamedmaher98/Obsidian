

| العلاقة      | الطرف اللي عنده المفتاح                  | تعمل إيه؟                          | الطرف العكسي               | تعمل إيه؟     |
|--------------|------------------------------------------|--------------------------------------|----------------------------|---------------|
| OneToOne     | أي طرف تختاره                            | `@JoinColumn`                        | الطرف التاني               | `mappedBy`    |
| OneToMany    | الطرف Many                               | `@JoinColumn`                        | الطرف One                  | `mappedBy`    |
| ManyToOne    | الطرف Many                               | `@JoinColumn`                        | —                          | —             |
| ManyToMany   | الطرف اللي ماسك الجدول الوسيط (owning)   | `@JoinTable`                         | الطرف التاني               | `mappedBy`    |

|القاعدة|الشرح بالبلدي|مثال سريع|
|---|---|---|
|**1- لازم تحدد الطرف المالك (Owning Side)**|الطرف المالك هو اللي بيحمل الـ **foreign key** في جدول قاعدة البيانات.|لو عندك `Book` فيه `author_id` → الـ Book هو المالك.|
|**2- الطرف المالك هو اللي بيحط الـ @JoinColumn**|لأن عنده العمود اللي فيه المفتاح الخارجي.|`@JoinColumn(name = "author_id")` في `Book`.|
|**3- الطرف التاني بيحط `mappedBy`**|علشان تقول له: "أنا مش المالك، روح بص على العلاقة عند الطرف المالك".|`@OneToMany(mappedBy = "author")` في `Author`.|
|**4- لو العلاقة OneToMany بدون ManyToOne**|هتطلع جدول وسيط أو أخطاء، فالأفضل تعمل ManyToOne في الطرف "Many".|`Order` → `Customer`|
|**5- لو العلاقة ManyToMany**|JPA بيعمل جدول وسيط أوتوماتيك، أو تحدد أنت بـ @JoinTable.|`Student` ↔ `Course`.|
|**6- استخدم `cascade` بحذر**|عشان لو مسحت أب أو عملت حفظ، يتطبق على الأولاد. لكن خلي بالك من `REMOVE` لأنه ممكن يمسح بيانات غلط.|`cascade = CascadeType.ALL` مع الحذر.|
|**7- استخدم `fetch = LAZY` في معظم الحالات**|علشان متجبش بيانات زيادة من غير ما تحتاجها.|`@OneToMany(fetch = FetchType.LAZY)`|
|**8- UUID مفيد للـ ID لو المشروع موزع**|أما لو بسيط وكله في DB واحدة، Long Auto Increment أسرع وأسهل.|`@GeneratedValue(strategy = GenerationType.IDENTITY)`|
|**9- دايمًا اربط العلاقات بالـ Objects مش IDs**|يعني خليك تمسك كائن `Author` جوه `Book` مش `authorId`.|`private Author author;`|
|**10- افصل بين الـ Entity والـ DTO**|علشان ما تبعتش الـ Entity مباشرة للـ API وتتجنب مشاكل Lazy Loading أو الحلقات.|استخدم MapStruct أو ModelMapper.|

أ. **OneToOne** (واحد لواحد)
- يعني كل سجل في الجدول الأول مرتبط بسجل واحد في الجدول التاني.
    
- **لو أنت الطرف اللي هيمسك الـ foreign key** → تحط `@JoinColumn`.
    
- **لو أنت الطرف العكسي** → تحط `mappedBy`
@OneToOne
@JoinColumn(name = "passport_id") // المفتاح هنا
private Passport passport;
في الطرف التاني 
@OneToOne(mappedBy = "passport") // أنا العكسي
private User user;

---
### ب - **OneToMany** (واحد لكثير)

- واحد من الجدول الأول ليه أكتر من سجل في الجدول التاني.
    
- **الطرف اللي فيه العمود بتاع المفتاح الأجنبي** = Many side → يحط `@JoinColumn`.
    
- **الطرف الواحد (One side)** → يحط `mappedBy`.
  
  
  // الجانب الواحد
@OneToMany(mappedBy = "user")
private List<Post> posts;

// الجانب الكثير
@ManyToOne
@JoinColumn(name = "user_id")
private User user;
----
### ج -  **ManyToOne** (كثير لواحد)

- العكس من OneToMany.
    
- **دايمًا الـ Many side هو اللي فيه الـ foreign key** → تحط `@JoinColumn`.
    
- مفيش `mappedBy` هنا لأن العلاقة بتبقى من ناحية واحدة.
  
  @ManyToOne
@JoinColumn(name = "user_id")
private User user;


---

### 4️⃣ **ManyToMany** (كثير لكثير)

- كل سجل في جدول مرتبط بأكتر من سجل في الجدول التاني، والعكس.
    
- لازم يبقى فيه جدول وسيط (Join Table).
    
- **في الطرف اللي هيدير العلاقة** → تحط `@JoinTable`.
    
- **الطرف العكسي** → تحط `mappedBy`.

// الطرف اللي ماسك العلاقة
@ManyToMany
@JoinTable(
    name = "student_course",
    joinColumns = @JoinColumn(name = "student_id"),
    inverseJoinColumns = @JoinColumn(name = "course_id")
)
private List<Course> courses;

// الطرف التاني
@ManyToMany(mappedBy = "courses")
private List<Student> students;
----
| العلاقة                 | الطرف اللي عنده المفتاح                   | تعمل إيه؟               | الطرف العكسي            | تعمل إيه؟  |
| ----------                | -----------------------------       | -------------                | ------------         | ---------- |
| اOneToOne           | أي طرف تختاره                        |`@JoinColum            | الطرف التاني        |  `mappedBy` |
|  ااOneToManyا |       Many side  |         @JoinColumn  |                           One side     |     mappedBy
| اManyToOne  |                           Many side |     @JoinColumn`               | —                     | —          |
| اManyToMany     | الطرف اللي ماسك الجدول الوسيط | `@JoinTable`         | الطرف التاني         | `mappedBy` |





