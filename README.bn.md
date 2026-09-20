# Ubuntu-তে পেশাদার নেটিভ CAD: KOMPAS-3D v25 Home ইনস্টলেশন নির্দেশিকা

[English](README.md) | [Русский](README.ru.md) | [简体中文](README.zh-CN.md) | [हिन्दी](README.hi.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Français](README.fr.md) | [Português](README.pt.md) | [বাংলা](README.bn.md) | [Bahasa Indonesia](README.id.md)

Wine বা ভার্চুয়াল মেশিন ছাড়াই Ubuntu-তে সরাসরি KOMPAS-3D-এর Linux সংস্করণ চালান। এই কমিউনিটি নির্দেশিকায় Ubuntu 26.04.1 LTS, amd64-এ সফল ইনস্টলেশনের বিবরণ রয়েছে। এখানে এই পেশাদার CAD সফটওয়্যারের Home সংস্করণ ইনস্টল করা হয়।

> ASCON আনুষ্ঠানিকভাবে Ubuntu সমর্থন করে না। Home শুধুমাত্র ব্যক্তিগত, অ-বাণিজ্যিক ব্যবহারের জন্য; শিরোনামটি বাণিজ্যিক লাইসেন্স পাওয়ার দাবি করে না। ASCON, Home-এর জন্য ৬০ দিনের ট্রায়াল দেয়, যা ভার্চুয়াল মেশিন বা টার্মিনাল সার্ভারে চলে না। নিচে আনুষ্ঠানিক শর্তগুলো দেখুন।

2026-09-20 তারিখে নথিবদ্ধ: Ubuntu 26.04.1 LTS (resolute), amd64, KOMPAS প্যাকেজ 25.0.1.2738। প্রথমে দুটি প্যাকেজের সিমুলেশনে ৪৭টি নতুন প্যাকেজ যোগ করার পরিকল্পনা ছিল; কোনো পুরোনো প্যাকেজ অপসারণ বা আপগ্রেড হতো না এবং সিস্টেমের নির্ভরতাগুলো Ubuntu থেকে আসছিল। অ্যাক্টিভেশন ইউটিলিটি পরে আলাদাভাবে ইনস্টল করা হয়। এরপর ব্যবহারকারী জানান যে সবকিছু কাজ করছে। বড় অ্যাসেম্বলি, কর্মক্ষমতা ও দীর্ঘমেয়াদি স্থিতিশীলতা আলাদাভাবে পরীক্ষা করা হয়নি।

## 1. সিস্টেম যাচাই ও টুল প্রস্তুত করুন

Bash ব্যবহার করুন এবং কোড ব্লকগুলো ক্রমানুসারে চালান। আর্কিটেকচার অবশ্যই amd64 হতে হবে। কোনো কমান্ড ব্যর্থ হলে থামুন। এই পদ্ধতি Ubuntu 26.04-এর জন্য; অন্য সংস্করণ আলাদাভাবে যাচাই করতে হবে।

```bash
. /etc/os-release
printf '%s (%s)\n' "$PRETTY_NAME" "$VERSION_CODENAME"
dpkg --print-architecture
sudo apt update
sudo apt install curl gnupg ca-certificates
mkdir -p ~/Downloads/kompas25
cd ~/Downloads/kompas25
```

## 2. রিপোজিটরির কী ডাউনলোড করুন

একই টার্মিনালে ~/Downloads/kompas25 ডিরেক্টরি থেকে কাজ চালিয়ে যান। ASCON থেকে HTTPS-এর মাধ্যমে কী ডাউনলোড হয় এবং signed-by দিয়ে সংশ্লিষ্ট রিপোজিটরিতে সীমাবদ্ধ থাকে।

```bash
curl -fSL https://repo.ascon.ru/personal/deb/ascon.gpg -o ascon-personal.gpg &&
curl -fSL https://repo.ascon.ru/stable/deb/ascon.gpg -o ascon-stable.gpg &&
gpg --dearmor --yes --output ascon-personal-keyring.gpg ascon-personal.gpg &&
gpg --dearmor --yes --output ascon-stable-keyring.gpg ascon-stable.gpg &&
sudo install -d -m 0755 /etc/apt/keyrings &&
sudo install -m 0644 ascon-personal-keyring.gpg /etc/apt/keyrings/ &&
sudo install -m 0644 ascon-stable-keyring.gpg /etc/apt/keyrings/
```

## 3. ASCON রিপোজিটরি যোগ করুন

ইনস্টলেশনের সময় দুটি রিপোজিটরিতেই শুধু 1.8_x86-64 শাখা ছিল। সরবরাহকারীর স্ক্রিপ্ট resolute ব্যবহার করত এবং HTTP 404 পাওয়া যেত। তাই আমরা Astra Linux-এর জন্য তৈরি ASCON প্যাকেজের শাখা সরাসরি বেছে নিচ্ছি। Astra Linux অপারেটিং সিস্টেমের নিজস্ব রিপোজিটরি যোগ করবেন না। কমান্ডগুলো উল্লেখিত দুটি .list ফাইল প্রতিস্থাপন করবে; আগে থেকে ASCON রিপোজিটরি ব্যবহার করলে পুরোনো ফাইল যাচাই ও ব্যাকআপ করুন। স্বাক্ষর বা রিপোজিটরির ত্রুটি হলে থামুন; যাচাইকরণ বন্ধ করবেন না।

```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-personal-keyring.gpg] https://repo.ascon.ru/personal/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon-personal.list
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-stable-keyring.gpg] https://repo.ascon.ru/stable/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon.list
sudo apt update
```

## 4. ইনস্টলেশন সিমুলেট করুন

এতে কোনো প্যাকেজ বদলাবে না। পুরো পরিকল্পনা দেখুন: প্যাকেজ অপসারণ, সংস্করণ নামানো বা Ubuntu-এর সিস্টেম লাইব্রেরি অন্য ডিস্ট্রিবিউশনের সংস্করণ দিয়ে প্রতিস্থাপন করা উচিত নয়। প্যাকেজের সংখ্যা ভিন্ন হতে পারে। নির্ভরতা পূরণ না হলে কারণ খুঁজুন, জোর করে ইনস্টল করবেন না।

```bash
apt-get --simulate install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 5. KOMPAS ও অ্যাক্টিভেশন ইউটিলিটি ইনস্টল করুন

প্রথম ন্যূনতম ইনস্টলেশনে বাদ পড়া অ্যাক্টিভেশন ইউটিলিটি এখানে অন্তর্ভুক্ত আছে। নিশ্চিত করার আগে APT-এর পরিকল্পনা দেখুন। প্যাকেজ অপসারণের প্রয়োজন হলে --no-remove, APT-কে থামিয়ে দেয়।

```bash
sudo apt-get --no-remove install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 6. সাধারণ ব্যবহারকারী হিসেবে চালু করুন

sudo দিয়ে অ্যাপ চালাবেন না। টার্মিনালের আউটপুট first-launch.log-এ সংরক্ষিত হবে; শেয়ার করার আগে ব্যক্তিগত তথ্য আছে কি না দেখুন।

```bash
kompas-home-v25 2>&1 | tee ~/Downloads/kompas25/first-launch.log
```

## 7. ট্রায়াল সক্রিয় করুন

Help → Protection key utility (Справка → Утилита ключа защиты) → Trial licenses (Ознакомительные лицензии) খুলুন। ট্রায়াল মোড বেছে নিন, ইমেইল লিখুন, গোপনীয়তার বিজ্ঞপ্তি পড়ুন এবং সম্মত হলে সম্মতির ঘরে টিক দিন, তারপর Activate চাপুন। ইন্টারফেসের ভাষা অনুযায়ী মেনুর নাম ভিন্ন হতে পারে; এই নির্দেশিকার অনুবাদ অ্যাপের ভাষা বদলায় না।

## 8. সমস্যার সমাধান

**Protection key utility পাওয়া যাচ্ছে না:** KOMPAS বন্ধ করুন, নিচের প্যাকেজটি ইনস্টল করুন এবং অ্যাপ আবার খুলুন।

```bash
sudo apt-get --no-remove install ascon-kompas-kactivation-v25
```

**কন্ট্রোল একটির ওপর আরেকটি উঠে গেলে বা চেকবক্সে ক্লিক করা না গেলে:** Tab / Shift+Tab দিয়ে চেকবক্সে ফোকাস আনুন এবং Space দিয়ে অবস্থা বদলান। প্রয়োজনে সাময়িকভাবে Ubuntu Settings → Displays → Scale-এ 100% দিন, ইউটিলিটি ও KOMPAS বন্ধ করে আবার চালু করুন। এগুলো প্রস্তাবিত সমাধান ছিল; ব্যবহারকারী সফল হওয়ার কথা জানালেও কোনটি কাজে লেগেছে বলেননি। সমস্যা থাকলে নিচের ডায়াগনস্টিক তথ্য সংগ্রহ করুন; নির্দিষ্ট কোনো গ্রাফিক্স লাইব্রেরিকে কারণ হিসেবে চিহ্নিত করা হয়নি।

```bash
echo "Session: $XDG_SESSION_TYPE"
printenv | sort | grep -E '^(QT_|GDK_|GTK_|.*SCALE|.*DPI)'
dpkg -L ascon-kompas-kactivation-v25 | grep -E '/bin/|\.desktop$|\.sh$'
```

অ্যাক্টিভেশনের পরে একটি পার্ট তৈরি করে সাধারণ এক্সট্রুশন করুন, ফাইল সংরক্ষণ করুন এবং আবার খুলে নিজের ইনস্টলেশন যাচাই করুন। চালু না হলে first-launch.log দেখুন। এই রিপোজিটরিতে শুধু নির্দেশনা রয়েছে, ASCON-এর প্রোগ্রাম বা লাইসেন্স কী নেই।

## আনুষ্ঠানিক তথ্যসূত্র

- [ASCON — Linux installation / Astra Linux (PDF)](https://kompas.ru/source/documents/2026/Home/Install_astralinux.pdf)

- [ASCON — Home trial activation on Linux (PDF)](https://kompas.ru/source/documents/2026/Home/activ_trial_Linux_25.pdf)

- [ASCON — Home trial terms](https://kompas.ru/kompas-3d-home/download/)

- [ASCON — KOMPAS-3D v25](https://kompas.ru/kompas-3d/v25/)
