2xx – نجاح (Success)

|الكود|المعنى|
|---|---|
|`200 OK`|الطلب تم بنجاح (الأكثر شيوعًا)|
|`201 Created`|تم إنشاء مورد جديد (زي POST نجح)|
|`204 No Content`|تم التنفيذ لكن مفيش بيانات راجعة|
3xx – إعادة توجيه (Redirection)
- معناها لازم العميل يروح لمكان تاني.

|الكود|المعنى|
|---|---|
|`301 Moved Permanently`|الصفحة اتنقلت نهائيًا|
|`302 Found`|مؤقتًا في مكان تاني|
|`304 Not Modified`|المحتوى لسه هو نفسه، استخدم الكاش|

4xx – خطأ من العميل (Client Error)

- الغلط من الطلب أو البيانات اللي بعتها الـ client.

|الكود|المعنى|
|---|---|
|`400 Bad Request`|فيه خطأ في الطلب (مثلاً JSON بايظ)|
|`401 Unauthorized`|محتاج تسجيل دخول أو Token|
|`403 Forbidden`|مش مسموحلك تدخل|
|`404 Not Found`|العنوان اللي بتطلبه مش موجود|
|`409 Conflict`|فيه تعارض في الطلب (مثلاً سجل موجود بالفعل)|

5xx – خطأ من السيرفر (Server Error)

|الكود|المعنى|
|---|---|
|`500 Internal Server Error`|خطأ عام في السيرفر|
|`502 Bad Gateway`|السيرفر الوسيط خد رد غير صحيح|
|`503 Service Unavailable`|السيرفر مش متاح حاليًا|
|`504 Gateway Timeout`|السيرفر ماردّش في الوقت المناسب|

---
```java
@GetMapping("/user/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    Optional<User> user = userService.findById(id);
    if (user.isPresent()) {
        return ResponseEntity.ok(user.get()); // 200 OK
    } else {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).build(); // 404 Not Found
    }
}

```