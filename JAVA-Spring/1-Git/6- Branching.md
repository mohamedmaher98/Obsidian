# دليل Git Branches الشامل

## يعني إيه Branch أصلاً؟

تخيل إنك بتشتغل على مشروع، وعايز تجرب فكرة جديدة من غير ما تأثر على الكود الأساسي اللي شغال كويس. هنا بالظبط بييجي دور الـ Branch.

الـ Branch هو ببساطة **نسخة موازية** من الكود بتاعك. زي ما تكون فاتح طريق جانبي من الطريق الرئيسي، تشتغل فيه براحتك، ولما تخلص وكل حاجة تمام، ترجع تدمجه في الطريق الرئيسي.

---

## الـ Branch الرئيسي (main / master)

لما بتعمل `git init` لأول مرة، Git بيعملك branch افتراضي اسمه `master` (أو `main` في الريبوهات الأحدث). ده بيكون الخط الرئيسي للكود بتاعك — النسخة المستقرة اللي المفروض تكون شغالة دايماً.

---

## إزاي تشوف الـ Branches الموجودة؟

### عرض الـ Branches المحلية

```bash
git branch
```

هيطلعلك قائمة بكل الـ branches اللي عندك محلياً، والـ branch اللي واقف عليه دلوقتي هيكون عليه علامة `*`.

### عرض الـ Branches كلها (محلية + Remote)

```bash
git branch -a
```

ده هيوريك كل حاجة — المحلي واللي على السيرفر (remote).

### عرض الـ Remote Branches بس

```bash
git branch -r
```

---

## إنشاء Branch جديد

### إنشاء بس من غير ما تنتقل ليه

```bash
git branch feature/login-page
```

كده عملت branch جديد اسمه `feature/login-page` بس لسه واقف على الـ branch القديم.

### إنشاء والانتقال في خطوة واحدة

```bash
git checkout -b feature/login-page
```

أو بالطريقة الأحدث:

```bash
git switch -c feature/login-page
```

### إنشاء branch من branch معين أو commit معين

```bash
# من branch معين
git checkout -b feature/new-thing origin/develop

# من commit معين
git checkout -b hotfix/urgent-fix abc1234
```

---

## التنقل بين الـ Branches

### الانتقال لـ branch موجود

```bash
git checkout feature/login-page
```

أو:

```bash
git switch feature/login-page
```

### الرجوع للـ branch اللي كنت عليه قبل كده

```bash
git checkout -
```

ده shortcut مفيد جداً — بينقلك لآخر branch كنت واقف عليه.

---

## تسمية الـ Branches — الطريقة الصح

في conventions متعارف عليها بتخلي الشغل منظم:

| البادئة | الاستخدام | مثال |
|---------|-----------|------|
| `feature/` | ميزة جديدة | `feature/user-auth` |
| `bugfix/` | إصلاح bug | `bugfix/login-crash` |
| `hotfix/` | إصلاح عاجل في Production | `hotfix/payment-error` |
| `release/` | تجهيز نسخة للنشر | `release/v2.1.0` |
| `chore/` | مهام صيانة | `chore/update-deps` |
| `refactor/` | إعادة هيكلة | `refactor/api-layer` |

### نصائح للتسمية

- استخدم `-` أو `/` للفصل بين الكلمات
- خلي الاسم وصفي وقصير
- اتجنب المسافات والحروف الخاصة
- لو شغال بنظام تذاكر (Jira مثلاً)، حط رقم التذكرة: `feature/PROJ-123-add-login`

---

## دمج الـ Branches (Merge)

لما تخلص شغلك على branch وعايز تدمجه في الـ branch الرئيسي:

### الـ Merge العادي (Merge Commit)

```bash
# 1. روح على الـ branch اللي عايز تدمج فيه
git checkout master

# 2. ادمج الـ branch
git merge feature/login-page
```

ده بيعمل "merge commit" — يعني commit جديد بيجمع تاريخ الاتنين مع بعض.

### الـ Fast-Forward Merge

لو الـ `master` ما اتغيرش من وقت ما عملت الـ branch، Git هيعمل fast-forward — يعني هيحرك مؤشر الـ master لقدام بدل ما يعمل merge commit:

```bash
git merge feature/login-page
# Fast-forward merge (لو مفيش تغييرات على master)
```

لو عايز تمنع الـ fast-forward وتعمل merge commit دايماً:

```bash
git merge --no-ff feature/login-page
```

### الـ Squash Merge

لو عملت 20 commit على الـ branch وعايز تدمجهم كلهم في commit واحد:

```bash
git merge --squash feature/login-page
git commit -m "Add login page feature"
```

---

## الـ Rebase — البديل للـ Merge

الـ Rebase بياخد الـ commits بتاعتك ويحطها فوق آخر commit في الـ branch التاني، كأنك كنت شغال من هناك من الأول.

```bash
# وانت واقف على الـ feature branch
git rebase master
```

### الفرق بين Merge و Rebase

| | Merge | Rebase |
|--|-------|--------|
| **التاريخ** | بيحافظ على التاريخ الأصلي | بيعيد كتابة التاريخ |
| **الشكل** | بيعمل merge commit | تاريخ خطي ونضيف |
| **الأمان** | آمن على shared branches | خطر لو الـ branch متشير مع حد تاني |
| **الاستخدام** | للدمج النهائي | لتحديث الـ feature branch |

### القاعدة الذهبية للـ Rebase

> **ما تعملش rebase أبداً على branch اتعمله push وناس تانية شغالة عليه.** الـ Rebase بيغير تاريخ الـ commits، ولو حد تاني شغال على نفس الـ branch، هتحصل مشاكل كبيرة.

---

## حل الـ Merge Conflicts

أحياناً لما بتدمج branchين، Git مش بيعرف يدمج التغييرات لوحده لأن نفس السطر اتعدل في الاتنين. هنا بيحصل **conflict**.

### شكل الـ Conflict في الملف

```
<<<<<<< HEAD
const title = "Welcome";
=======
const title = "Hello World";
>>>>>>> feature/new-title
```

- اللي بين `<<<<<<< HEAD` و `=======` → الكود في الـ branch الحالي
- اللي بين `=======` و `>>>>>>> feature/new-title` → الكود في الـ branch اللي بتدمجه

### خطوات الحل

```bash
# 1. افتح الملفات اللي فيها conflict وعدلها يدوي
# 2. بعد ما تحل، ضيف الملفات
git add .

# 3. كمل الـ merge
git merge --continue

# أو لو عايز تلغي الـ merge خالص
git merge --abort
```

---

## مسح الـ Branches

### مسح branch محلي

```bash
# مسح عادي (لو الـ branch اتدمج)
git branch -d feature/login-page

# مسح بالقوة (حتى لو ما اتدمجش)
git branch -D feature/login-page
```

### مسح branch من الـ Remote

```bash
git push origin --delete feature/login-page
```

### تنضيف الـ Remote Branches اللي اتمسحت

```bash
git fetch --prune
```

أو:

```bash
git remote prune origin
```

---

## رفع Branch على الـ Remote

### أول مرة ترفع branch

```bash
git push -u origin feature/login-page
```

الـ `-u` (أو `--set-upstream`) بتربط الـ branch المحلي بالـ remote، فبعد كده تقدر تعمل `git push` و `git pull` من غير ما تحدد الاسم.

### رفع عادي بعد كده

```bash
git push
```

---

## تتبع الـ Remote Branches

### جلب branch من الـ Remote

```bash
# الأول حدث قائمة الـ remote branches
git fetch

# بعدين اعمل checkout
git checkout feature/someone-elses-branch
```

Git هيعمل لوحده tracking للـ remote branch.

### إنشاء branch محلي من remote branch

```bash
git checkout -b my-local-name origin/remote-branch-name
```

---

## الـ Stash — لما تحتاج تنقل branch وعندك تغييرات

لو عندك تغييرات مش مستعد تعملها commit وعايز تنقل branch:

```bash
# احفظ التغييرات مؤقتاً
git stash

# نقل للـ branch التاني
git checkout other-branch

# لما ترجع، استرجع التغييرات
git checkout original-branch
git stash pop
```

### أوامر Stash مفيدة

```bash
# حفظ مع رسالة وصفية
git stash save "work in progress on login"

# عرض قائمة الـ stashes
git stash list

# استرجاع stash معين
git stash apply stash@{2}

# مسح كل الـ stashes
git stash clear
```

---

## استراتيجيات Branching الشائعة

### 1. Git Flow

النظام ده فيه branches ثابتة ومنظمة:

```
master ─────────────────────────────────────
  │                                    ▲
  └── develop ──────────────────────── │
        │          ▲         │         │
        └── feature/x ──────┘         │
                                       │
        └── release/1.0 ──────────────┘
```

- **master**: النسخة اللي في Production
- **develop**: الكود اللي بيتطور
- **feature/**: ميزات جديدة بتتفرع من develop
- **release/**: تجهيز نسخة للنشر
- **hotfix/**: إصلاحات عاجلة بتتفرع من master

### 2. GitHub Flow

أبسط بكتير:

1. اعمل branch من `main`
2. اشتغل وعدل و commit
3. افتح Pull Request
4. ناقش واعمل review
5. ادمج في `main`

### 3. Trunk-Based Development

الكل بيشتغل على `main` مباشرة مع branches قصيرة العمر (يوم أو اتنين بالكتير).

---

## أوامر متقدمة ومفيدة

### شوف تاريخ الـ Branches بشكل مرئي

```bash
git log --oneline --graph --all
```

### شوف آخر commit في كل branch

```bash
git branch -v
```

### شوف الـ Branches اللي اتدمجت في الـ Branch الحالي

```bash
# الـ branches اللي اتدمجت (ممكن تتمسح)
git branch --merged

# الـ branches اللي لسه ما اتدمجتش
git branch --no-merged
```

### إعادة تسمية branch

```bash
# وانت واقف على الـ branch
git branch -m new-name

# وانت على branch تاني
git branch -m old-name new-name
```

### Cherry-Pick — نقل commit معين من branch لـ branch

```bash
# انقل commit معين للـ branch الحالي
git cherry-pick abc1234
```

مفيد لما تحتاج commit واحد بس من branch تاني من غير ما تدمج كل حاجة.

---

## أخطاء شائعة وإزاي تتجنبها

### 1. الشغل على الـ master مباشرة
دايماً اعمل branch جديد قبل ما تبدأ أي شغل.

### 2. عدم تحديث الـ Branch قبل الدمج
قبل ما تعمل merge، حدث الـ branch بتاعك:
```bash
git checkout feature/my-feature
git pull origin master
# أو
git rebase master
```

### 3. نسيان مسح الـ Branches القديمة
كل فترة نضف الـ branches اللي خلصت:
```bash
git branch --merged | grep -v "master\|main" | xargs git branch -d
```

### 4. عمل Force Push على Shared Branches
`git push --force` ممكن يمسح شغل ناس تانية. لو محتاج force push، استخدم:
```bash
git push --force-with-lease
```

ده بيتأكد إن محدش تاني عمل push قبلك.

---

## ملخص سريع لأهم الأوامر

| الأمر | الوصف |
|-------|-------|
| `git branch` | عرض الـ branches |
| `git branch name` | إنشاء branch |
| `git checkout -b name` | إنشاء + انتقال |
| `git switch name` | انتقال لـ branch |
| `git merge name` | دمج branch |
| `git rebase name` | إعادة تأسيس |
| `git branch -d name` | مسح branch |
| `git push -u origin name` | رفع branch |
| `git stash` | حفظ مؤقت |
| `git cherry-pick hash` | نقل commit معين |
