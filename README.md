# رفع ونشر Bot Forge من موبايل أندرويد (Termux)

بما إنك شغال من التليفون، أسهل وأقوى طريقة هي **Termux** — تطبيق terminal حقيقي على أندرويد،
هتقدر بيه تعمل git + node زي أي كمبيوتر تقريبًا.

---

## 1) تثبيت Termux

⚠️ **متنزلش Termux من Google Play** (نسخة قديمة ومهجورة).
نزّله من **F-Droid** بدل كده:

https://f-droid.org/packages/com.termux/

(لو معندكش F-Droid: نزّل F-Droid الأول من f-droid.org، بعدين دور فيه على Termux ونزله)

---

## 2) تجهيز Termux

افتح Termux واكتب:

```bash
pkg update -y && pkg upgrade -y
pkg install git nodejs unzip -y
```

اديله شوية وقت أول مرة (بيثبت حاجات كتير).

اسمح لـ Termux يشوف ملفاتك:
```bash
termux-setup-storage
```
(هيطلعلك إذن، اضغط Allow)

---

## 3) نقل ملف المشروع لجوه Termux

1. حمّل ملف `bot-forge.zip` اللي بعتهولك (لو مش محمل، افتحه من شات Claude واحفظه في Downloads).
2. في Termux:

```bash
cd ~
mkdir work && cd work
cp /sdcard/Download/bot-forge.zip .
unzip bot-forge.zip
cd bot-forge
```

---

## 4) عمل حساب GitHub Token (بدل الباسورد)

GitHub بقى مش بيقبل باسورد عادي وقت الـ push من terminal. لازم **Personal Access Token**:

1. من متصفح التليفون افتح: https://github.com/settings/tokens
2. **Generate new token → Generate new token (classic)**
3. اديله اسم زي `termux-phone`
4. حدد صلاحية **repo** بس (تيك عليها)
5. Generate token واحفظ الكود اللي هيطلعلك (بيتشاف مرة واحدة بس!) — انسخه في مكان آمن (Google Keep مثلاً)

---

## 5) عمل الريبو على GitHub

من المتصفح: https://github.com/new
- اسم الريبو (مثلاً `bot-forge`)
- خليه **Private** (أفضل، عشان جوه بوتك)
- متعملش تيك على README ولا .gitignore (سيبها فاضية)
- Create repository

---

## 6) رفع المشروع من Termux

في Termux (لسه جوه فولدر bot-forge):

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git config --global user.email "بريدك@example.com"
git config --global user.name "اسمك"
git remote add origin https://github.com/USERNAME/bot-forge.git
git push -u origin main
```

- بدّل `USERNAME` باسمك في GitHub، و`bot-forge` باسم الريبو اللي عملته.
- وقت الـ push هيطلبلك **Username** (اسمك في GitHub) و**Password** — هنا حط الـ **Token** اللي عملته في الخطوة 4 بدل الباسورد.

لو الـ push نجح، الملفات هتظهر على صفحة الريبو على GitHub فورًا.

---

## 7) قاعدة البيانات (Neon) — من المتصفح

1. افتح https://neon.tech وسجل دخول بحساب GitHub
2. Create Project → اختار اسم
3. انسخ الـ Connection String (بيبدأ بـ `postgresql://...`)
4. احفظه، هتحتاجه في الخطوة الجاية وفي Vercel

### تجهيز الجداول من Termux (اختياري بس مهم):
```bash
cd ~/work/bot-forge
echo 'DATABASE_URL="الكونكشن سترينج بتاعك هنا"' > .env
npm install
npx prisma db push
```
ده بيعمل الجداول (User, Account, Session, Bundle) على قاعدة Neon مباشرة.
لو `npm install` بطيء أو فيه مشاكل، قدر تتخطى الخطوة دي وتخلي Vercel يعملها وقت أول Deploy — بس أضمن تعمل db push بنفسك.

---

## 8) Google OAuth — من المتصفح

1. https://console.cloud.google.com → أنشئ مشروع
2. APIs & Services → Credentials → Create Credentials → OAuth Client ID → Web Application
3. Authorized redirect URIs: هتضيفها بعد ما تاخد دومين Vercel (خطوة 10)
4. احفظ **Client ID** و **Client Secret**

---

## 9) استيراد المشروع في Vercel — من المتصفح

1. https://vercel.com → سجل دخول بحساب GitHub
2. Add New → Project → اختار ريبو `bot-forge`
3. **قبل الـ Deploy**، ضيف الـ Environment Variables:

| Key | Value |
|---|---|
| `GOOGLE_CLIENT_ID` | من خطوة 8 |
| `GOOGLE_CLIENT_SECRET` | من خطوة 8 |
| `NEXTAUTH_URL` | هتحطها بعد أول deploy (سيبها فاضية دلوقتي أو حط أي حاجة مؤقتة) |
| `NEXTAUTH_SECRET` | أي نص عشوائي طويل (32+ حرف) |
| `ADMIN_EMAIL` | إيميلك اللي هتسجل بيه |
| `DATABASE_URL` | كونكشن سترينج Neon من خطوة 7 |

4. اضغط **Deploy**

---

## 10) بعد أول Deploy

1. هتاخد دومين زي `https://bot-forge-xxxx.vercel.app`
2. ارجع Google Cloud Console → ضيف Redirect URI:
   ```
   https://bot-forge-xxxx.vercel.app/api/auth/callback/google
   ```
3. ارجع Vercel → Settings → Environment Variables → عدّل `NEXTAUTH_URL` بنفس الدومين بالظبط
4. اعمل **Redeploy** (من صفحة الـ Deployments → ⋯ → Redeploy)

---

## 11) اختبار

- افتح الدومين، سجل دخول بإيميل الأدمن
- جرب الفورم وحمّل ZIP تجريبي
- افتح `/admin` وتأكد إنك أدمن

---

## نصايح خاصة بـ Termux

- لو النت اتقطع وسط `npm install`، شغّل نفس الأمر تاني، بيكمل من غير ما يعيد كل حاجة.
- لو عايز ترجع تعدل ملف بعدين: `pkg install nano` وبعدين `nano اسم_الملف`
- عشان متكتبش التوكن كل مرة:
```bash
git config --global credential.helper store
```
(هيحفظ التوكن بعد أول مرة تكتبه)

---

## تذكير: بوت الواتساب بتاعك

فولدر `bot-template/` جوه المشروع لسه محتاج ملفات **Escanor Bot** الحقيقية بتاعتك.
ابعتهاليّ في شات تاني وهظبطها جوه القالب بالـ placeholders الصح، وبعدين تعمل:
```bash
git add .
git commit -m "Add real bot files"
git push
```
وVercel هيعمل Deploy جديد أوتوماتيك بمجرد الـ push.