# KOMPAS-3D على Ubuntu: تصميم بمساعدة الحاسوب احترافي وأصلي — تثبيت v25 Home

[English](README.md) | [Русский](README.ru.md) | [简体中文](README.zh-CN.md) | [हिन्दी](README.hi.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Français](README.fr.md) | [Português](README.pt.md) | [বাংলা](README.bn.md) | [Bahasa Indonesia](README.id.md)

شغّل إصدار Linux من KOMPAS-3D مباشرة على Ubuntu، دون Wine أو آلة افتراضية. يوثّق هذا الدليل المجتمعي تثبيتًا ناجحًا على Ubuntu 26.04.1 LTS بمعمارية amd64. الإصدار المثبّت هو Home من برنامج التصميم الاحترافي هذا.

> لا تدعم ASCON نظام Ubuntu رسميًا. إصدار Home مخصّص للاستخدام الشخصي غير التجاري؛ ولا يعني العنوان الحصول على ترخيص تجاري. تتيح ASCON تجربة Home لمدة 60 يومًا، لا تعمل على الآلات الافتراضية أو خوادم الطرفيات. راجع الشروط الرسمية أدناه.

سُجّلت التجربة في 2026-09-20: Ubuntu 26.04.1 LTS ‏(resolute)، ومعمارية amd64، وحزم KOMPAS بالإصدار 25.0.1.2738. كانت المحاكاة الأولى لحزمتين تقترح إضافة 47 حزمة دون إزالة الحزم الموجودة أو ترقيتها، وكانت اعتماديات النظام تأتي من Ubuntu. ثُبّتت أداة التفعيل بصورة منفصلة، ثم أكّد المستخدم أن كل شيء يعمل. لم تُختبر التجميعات الكبيرة أو الأداء أو الاستقرار طويل الأمد بشكل مستقل.

## 1. فحص النظام وتجهيز الأدوات

استخدم Bash ونفّذ كتل الأوامر بالترتيب. يجب أن تكون المعمارية amd64. توقّف إذا فشل أي أمر. هذه الخطوات مخصّصة لـ Ubuntu 26.04؛ وتحتاج الإصدارات الأخرى إلى تحقق منفصل.

```bash
. /etc/os-release
printf '%s (%s)\n' "$PRETTY_NAME" "$VERSION_CODENAME"
dpkg --print-architecture
sudo apt update
sudo apt install curl gnupg ca-certificates
mkdir -p ~/Downloads/kompas25
cd ~/Downloads/kompas25
```

## 2. تنزيل مفاتيح المستودعات

تابع في الطرفية نفسها داخل ~/Downloads/kompas25. تُنزّل المفاتيح من ASCON عبر HTTPS وتُربط بمستودعاتها باستخدام signed-by.

```bash
curl -fSL https://repo.ascon.ru/personal/deb/ascon.gpg -o ascon-personal.gpg &&
curl -fSL https://repo.ascon.ru/stable/deb/ascon.gpg -o ascon-stable.gpg &&
gpg --dearmor --yes --output ascon-personal-keyring.gpg ascon-personal.gpg &&
gpg --dearmor --yes --output ascon-stable-keyring.gpg ascon-stable.gpg &&
sudo install -d -m 0755 /etc/apt/keyrings &&
sudo install -m 0644 ascon-personal-keyring.gpg /etc/apt/keyrings/ &&
sudo install -m 0644 ascon-stable-keyring.gpg /etc/apt/keyrings/
```

## 3. إضافة مستودعات ASCON

وقت التثبيت، لم يتوفر في المستودعين سوى الفرع 1.8_x86-64. كانت سكربتات المورّد ستستخدم resolute وتؤدي إلى HTTP 404. لذلك نختار صراحةً فرع حزم ASCON المخصّص لـ Astra Linux. لا تضف مستودعات نظام التشغيل Astra Linux نفسه. تستبدل الأوامر ملفَي .list المحددين؛ افحص الملفات الموجودة وانسخها احتياطيًا إذا كنت تستخدم مستودعات ASCON مسبقًا. توقّف عند أخطاء التوقيع أو المستودع، ولا تعطّل التحقق.

```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-personal-keyring.gpg] https://repo.ascon.ru/personal/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon-personal.list
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-stable-keyring.gpg] https://repo.ascon.ru/stable/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon.list
sudo apt update
```

## 4. محاكاة التثبيت

لن تتغير أي حزم. راجع الخطة كاملةً: يجب ألا تتضمن إزالة حزم أو خفض إصدارات أو استبدال مكتبات Ubuntu بنسخ من توزيعة أخرى. قد يختلف عدد الحزم. إذا تعذّر حل الاعتماديات، ابحث عن السبب بدل فرض التثبيت.

```bash
apt-get --simulate install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 5. تثبيت KOMPAS وأداة التفعيل

تتضمن الخطوة أداة التفعيل التي كانت مفقودة من التثبيت الأدنى الأول. راجع خطة APT قبل تأكيدها. يوقف الخيار --no-remove العملية إذا احتاج APT إلى إزالة حزم.

```bash
sudo apt-get --no-remove install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 6. التشغيل بحساب المستخدم العادي

لا تشغّل التطبيق باستخدام sudo. يُحفظ خرج الطرفية في first-launch.log؛ افحصه بحثًا عن معلومات شخصية قبل مشاركته.

```bash
kompas-home-v25 2>&1 | tee ~/Downloads/kompas25/first-launch.log
```

## 7. تفعيل الفترة التجريبية

افتح «مساعدة ← أداة مفتاح الحماية» (Справка → Утилита ключа защиты)، ثم «التراخيص التجريبية» (Ознакомительные лицензии). اختر الوضع التجريبي، وأدخل بريدك الإلكتروني، واقرأ إشعار الخصوصية وحدد خانة الموافقة إن كنت موافقًا، ثم اضغط Activate. قد تختلف أسماء القوائم بحسب لغة الواجهة؛ ترجمة هذا الدليل لا تغيّر لغة التطبيق.

## 8. حل المشكلات

**تعذّر العثور على أداة مفتاح الحماية:** أغلق KOMPAS، وثبّت الحزمة التالية، ثم أعد فتح التطبيق.

```bash
sudo apt-get --no-remove install ascon-kompas-kactivation-v25
```

**تداخل عناصر الواجهة أو تعذّر تحديد خانة الموافقة:** استخدم Tab أو Shift+Tab للانتقال إلى الخانة، ثم مفتاح المسافة لتغيير حالتها. عند الحاجة، اضبط مؤقتًا إعدادات Ubuntu ← الشاشات ← مقياس العرض على 100%، وأغلق الأداة وKOMPAS ثم أعد تشغيلهما. كانت هذه حلولًا مقترحة؛ أكّد المستخدم النجاح دون تحديد أي حل ساعده. إذا استمرت المشكلة، اجمع معلومات التشخيص التالية؛ لم تُحدّد مكتبة رسومية بعينها بوصفها السبب.

```bash
echo "Session: $XDG_SESSION_TYPE"
printenv | sort | grep -E '^(QT_|GDK_|GTK_|.*SCALE|.*DPI)'
dpkg -L ascon-kompas-kactivation-v25 | grep -E '/bin/|\.desktop$|\.sh$'
```

بعد التفعيل، أنشئ جزءًا ونفّذ عملية بثق بسيطة، ثم احفظ الملف وأعد فتحه للتحقق من تثبيتك. إذا فشل التشغيل، راجع first-launch.log. يحتوي هذا المستودع على تعليمات فقط، ولا يوزّع ملفات ASCON التنفيذية أو مفاتيح الترخيص.

## المراجع الرسمية

- [ASCON — Linux installation / Astra Linux (PDF)](https://kompas.ru/source/documents/2026/Home/Install_astralinux.pdf)

- [ASCON — Home trial activation on Linux (PDF)](https://kompas.ru/source/documents/2026/Home/activ_trial_Linux_25.pdf)

- [ASCON — Home trial terms](https://kompas.ru/kompas-3d-home/download/)

- [ASCON — KOMPAS-3D v25](https://kompas.ru/kompas-3d/v25/)
