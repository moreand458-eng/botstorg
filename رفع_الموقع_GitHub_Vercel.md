# رفع Bot Forge على GitHub ثم نشره على Vercel

---

## ⚠️ حاجة مهمة قبل ما تبدأ (قاعدة البيانات)

بما إن الموقع هيشتغل بجد للناس، الملف المحدّث بقى متظبط على **Postgres** من الأول (مش SQLite)،
لأن Vercel سيرفرات serverless بتمسح أي ملفات محلية (زي SQLite) كل مرة، وهتفقد بيانات اليوزرز.

اعمل قاعدة بيانات مجانية (يكفي مشروع صغير/متوسط) من واحد من دول:
- **Neon** (الأسهل والأسرع): https://neon.tech
- **Vercel Postgres**: من داخل داشبورد Vercel نفسه بعد ربط المشروع
- **Supabase**: https://supabase.com

خطوات Neon (مثال سريع):
1. سجل دخول بحساب GitHub
2. Create Project → اختار اسم ومنطقة قريبة منك
3. هتلاقي Connection String جاهز، انسخه (يبدأ بـ `postgresql://...`)
4. حطه في `DATABASE_URL` محليًا وفي Vercel (قسم 3 تحت)

---

## 1) رفع المشروع على GitHub

```bash
cd bot-forge
git init
git add .
git commit -m "Initial commit - Bot Forge"
```

اعمل ريبو جديد فاضي على GitHub (من غير README) من هنا:
https://github.com/new

بعدين اربطه وارفع:

```bash
git branch -M main
git remote add origin https://github.com/USERNAME/REPO_NAME.git
git push -u origin main
```

> غيّر `USERNAME/REPO_NAME` باسمك واسم الريبو.

### تأكد إن دول متسيبوش يترفعوا (موجودين في `.gitignore` بالفعل):
- `node_modules/`
- `.next/`
- `dev.db`
- `.env.local` ← **مهم جدًا متسيبهوش يترفع، فيه أسرارك (Client Secret, NEXTAUTH_SECRET)**

---

## 2) استيراد المشروع في Vercel

1. روح https://vercel.com وسجل دخول بحساب GitHub بتاعك.
2. من الداشبورد: **Add New → Project**
3. اختار الريبو اللي رفعته دلوقتي.
4. Framework Preset هيتحدد أوتوماتيك: **Next.js**.
5. **متضغطش Deploy لسه** — روح لقسم Environment Variables الأول (تحت).

---

## 3) ضبط Environment Variables في Vercel

في نفس صفحة الإعداد (أو Project Settings → Environment Variables) ضيف:

| Key | Value |
|---|---|
| `GOOGLE_CLIENT_ID` | نفس القيمة من Google Cloud Console |
| `GOOGLE_CLIENT_SECRET` | نفس القيمة |
| `NEXTAUTH_URL` | `https://your-project.vercel.app` (أو دومينك الخاص) |
| `NEXTAUTH_SECRET` | ولّده بالأمر: `openssl rand -base64 32` |
| `ADMIN_EMAIL` | إيميلك اللي هتسجل بيه عشان تبقى أدمن |
| `DATABASE_URL` | الكونكشن سترينج اللي هتاخده من Neon/Vercel Postgres/Supabase (شكله `postgresql://user:pass@host/db?sslmode=require`) |

> بعد أول Deploy هتاخد الدومين الحقيقي بتاع Vercel، وقتها ارجع حدّث `NEXTAUTH_URL` بيه بالظبط وأعد الـ Deploy.

---

## 4) تحديث Google OAuth Redirect URI

روح Google Cloud Console → APIs & Services → Credentials → افتح الـ OAuth Client بتاعك، وضيف:

```
https://your-project.vercel.app/api/auth/callback/google
```

(بدّل الدومين بدومينك الحقيقي بعد أول Deploy). لو هتستخدم دومين خاص لاحقًا، ضيفه هنا كمان.

---

## 5) تجهيز قاعدة البيانات (Postgres)

السكيما (`prisma/schema.prisma`) أصلًا متظبطة على `postgresql`، محتاج بس تربطها بقاعدة حقيقية:

1. اعمل قاعدة بيانات من Neon/Vercel Postgres/Supabase (خطوات فوق).
2. حط الكونكشن سترينج في `DATABASE_URL` جوه `.env.local` عندك محليًا.
3. شغّل محليًا:
```bash
npx prisma db push
```
ده هيبني الجداول (User, Account, Session, Bundle...) على القاعدة الحقيقية.

4. نفس الكونكشن سترينج بالظبط حطه في Vercel Environment Variables (قسم 3).

---

## 6) الـ Deploy

بعد ما تضبط الـ Environment Variables، دوس **Deploy**.

Vercel هيعمل تلقائيًا:
```
npm install
prisma generate   (بيحصل من postinstall)
npm run build
```

لو فيه Build Error يبان في الـ Logs مباشرة — أكتر خطأ متوقع هو نسيان متغير بيئة أو دومين OAuth غلط.

---

## 7) بعد أول Deploy — خطوات لازمة

1. افتح الدومين اللي Vercel داهولك (`https://xxx.vercel.app`)
2. سجل دخول بإيميل الأدمن (`ADMIN_EMAIL`) — هيتحول أدمن أوتوماتيك.
3. جرب فورم التخصيص وحمل ZIP تجريبي تتأكد إن التوليد شغال.
4. روح `/admin` وتأكد إن لوحة التحكم شغالة (عدد اليوزرز، الحظر...).
5. لو غيّرت `NEXTAUTH_URL` بعد أول deploy، اعمل **Redeploy** يدوي من Vercel (زرار الـ ⋯ جنب آخر Deployment → Redeploy).

---

## 8) دومين خاص (اختياري)

Project Settings → Domains → ضيف الدومين بتاعك واتبع تعليمات الـ DNS (CNAME أو A record حسب مزود الدومين).
متنساش بعدها تحدّث:
- `NEXTAUTH_URL` في Vercel
- Redirect URI في Google Cloud Console

---

## مشاكل شائعة

| المشكلة | الحل |
|---|---|
| `Error: PrismaClient is unable to run in this browser environment` | تأكد إن الاستدعاء لـ `prisma` من كود سيرفر (`route.ts`) مش من `"use client"` component |
| بعد تسجيل الدخول بيرجعلك تاني لصفحة الدخول (Redirect Loop) | `NEXTAUTH_URL` مش مطابق للدومين الحالي بالظبط، أو الـ Redirect URI في Google غلط |
| `Can't reach database server` وقت الـ Deploy | تأكد إن `DATABASE_URL` نفسه بالظبط (متطابق) محطوط في Vercel Environment Variables، ومفيهوش مسافات زيادة |
| ZIP التوليد بيرجع 500 | تأكد إن فولدر `bot-template/` اترفع كامل مع الريبو ومفيهوش أخطاء JS جوه `config.js` |
