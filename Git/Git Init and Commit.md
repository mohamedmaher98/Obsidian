| الأمر                      | وظيفته                                                                  | المنطقة اللي بيشتغل فيها             |
| -------------------------- | ----------------------------------------------------------------------- | ------------------------------------ |
| `git status`               | بيوريك حالة الملفات: إيه متعدل، إيه جاهز للـ commit                     | Working Directory + Staging Area     |
| `git add <file>`           | يضيف الملفات للـ **staging area** عشان تكون جاهزة للـ commit            | Staging Area                         |
| `git commit -m "رسالة"`    | يحفظ snapshot لكل الملفات الموجودة في staging area                      | Local Repository                     |
| `git push origin <branch>` | يبعت كل الـ commits الموجودة عندك للـ remote repository (GitHub/GitLab) | Local Repository → Remote Repository |
| `git pull`                 | يجلب أي تغييرات جديدة من الـ remote ويحدث الـ local repository عندك     | Remote Repository → Local Repository |
| `git checkout -b <branch>` | ينشئ فرع جديد وتشتغل عليه                                               | Local Repository                     |
| `git merge <branch>`       | يدمج تغييرات فرع معين في الفرع الحالي                                   | Local Repository                     |
