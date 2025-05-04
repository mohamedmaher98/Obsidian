git init
-يحوّل أي فولدر عادي لفولدر git project.يعني بيبدأ تتبع التعديلات اللي بتحصل فيه.

git clone
لو عايز تنزل مشروع من GitHub أو GitLab:
`git clone https://github.com/user/repo.git`
ينزل نسخة من المشروع عندك.

git add

git add file.txt      # تضيف ملف معين
git add .             # تضيف كل الملفات المعدّلة

git commit
git commit -m "رسالة التعديل"
يسجل نسخة من الكود في تاريخ معين برسالة توضح إنت عملت إيه:

git push origin main

يبعت التعديلات اللي عملتها على السيرفر (GitHub):

git pull origin main
يسحب أحدث نسخة من الكود من الريبو (GitHub مثلاً):

---
عشان ارفع اي فولدر علي جيت

1- git init 
2- git add .
3-