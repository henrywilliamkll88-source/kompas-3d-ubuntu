# Ubuntu पर पेशेवर नेटिव CAD: KOMPAS-3D v25 Home इंस्टॉलेशन गाइड

[English](README.md) | [Русский](README.ru.md) | [简体中文](README.zh-CN.md) | [हिन्दी](README.hi.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Français](README.fr.md) | [Português](README.pt.md) | [বাংলা](README.bn.md) | [Bahasa Indonesia](README.id.md)

KOMPAS-3D का Linux संस्करण सीधे Ubuntu पर चलाएँ, बिना Wine या वर्चुअल मशीन के। यह सामुदायिक गाइड Ubuntu 26.04.1 LTS, amd64 पर सफल इंस्टॉलेशन का विवरण देती है। इसमें इस पेशेवर CAD उत्पाद का Home संस्करण स्थापित किया जाता है।

> ASCON आधिकारिक तौर पर Ubuntu का समर्थन नहीं करता। Home केवल व्यक्तिगत, गैर-व्यावसायिक उपयोग के लिए है; शीर्षक का अर्थ व्यावसायिक लाइसेंस मिलना नहीं है। ASCON, Home के लिए 60 दिन का ट्रायल देता है, जो वर्चुअल मशीन या टर्मिनल सर्वर पर उपलब्ध नहीं है। नीचे आधिकारिक शर्तें देखें।

2026-09-20 को दर्ज: Ubuntu 26.04.1 LTS (resolute), amd64, KOMPAS पैकेज 25.0.1.2738। दो पैकेजों के शुरुआती सिमुलेशन में 47 नए पैकेज जुड़ने थे; कोई मौजूदा पैकेज हटना या अपग्रेड होना नहीं था और सिस्टम निर्भरताएँ Ubuntu से आ रही थीं। एक्टिवेशन यूटिलिटी अलग से स्थापित की गई। बाद में उपयोगकर्ता ने बताया कि सब काम कर रहा है। बड़ी असेंबली, प्रदर्शन और लंबे समय की स्थिरता की अलग से जाँच नहीं हुई।

## 1. सिस्टम जाँचें और टूल तैयार करें

Bash का उपयोग करें और कोड ब्लॉक क्रम से चलाएँ। आर्किटेक्चर amd64 होना चाहिए। किसी कमांड में त्रुटि हो तो रुकें। यह प्रक्रिया Ubuntu 26.04 के लिए है; अन्य संस्करणों की अलग जाँच ज़रूरी है।

```bash
. /etc/os-release
printf '%s (%s)\n' "$PRETTY_NAME" "$VERSION_CODENAME"
dpkg --print-architecture
sudo apt update
sudo apt install curl gnupg ca-certificates
mkdir -p ~/Downloads/kompas25
cd ~/Downloads/kompas25
```

## 2. रिपॉजिटरी की कुंजियाँ डाउनलोड करें

उसी टर्मिनल में ~/Downloads/kompas25 डायरेक्टरी से आगे बढ़ें। कुंजियाँ ASCON से HTTPS पर डाउनलोड होती हैं और signed-by के माध्यम से संबंधित रिपॉजिटरी तक सीमित रहती हैं।

```bash
curl -fSL https://repo.ascon.ru/personal/deb/ascon.gpg -o ascon-personal.gpg &&
curl -fSL https://repo.ascon.ru/stable/deb/ascon.gpg -o ascon-stable.gpg &&
gpg --dearmor --yes --output ascon-personal-keyring.gpg ascon-personal.gpg &&
gpg --dearmor --yes --output ascon-stable-keyring.gpg ascon-stable.gpg &&
sudo install -d -m 0755 /etc/apt/keyrings &&
sudo install -m 0644 ascon-personal-keyring.gpg /etc/apt/keyrings/ &&
sudo install -m 0644 ascon-stable-keyring.gpg /etc/apt/keyrings/
```

## 3. ASCON रिपॉजिटरी जोड़ें

इंस्टॉलेशन के समय दोनों रिपॉजिटरी में केवल 1.8_x86-64 शाखा थी। विक्रेता की स्क्रिप्ट resolute का उपयोग करती और HTTP 404 मिलता। इसलिए हम Astra Linux के लिए बने ASCON पैकेजों की शाखा स्पष्ट रूप से चुनते हैं। Astra Linux ऑपरेटिंग सिस्टम की रिपॉजिटरी न जोड़ें। ये कमांड बताए गए दो .list फ़ाइलों को बदल देती हैं; यदि पहले से ASCON रिपॉजिटरी उपयोग करते हैं तो फ़ाइलों की जाँच और बैकअप लें। हस्ताक्षर या रिपॉजिटरी संबंधी त्रुटि पर रुकें; सत्यापन बंद न करें।

```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-personal-keyring.gpg] https://repo.ascon.ru/personal/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon-personal.list
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-stable-keyring.gpg] https://repo.ascon.ru/stable/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon.list
sudo apt update
```

## 4. इंस्टॉलेशन का सिमुलेशन करें

इससे पैकेज नहीं बदलते। पूरी योजना देखें: कोई पैकेज हटना, संस्करण डाउनग्रेड होना या Ubuntu की सिस्टम लाइब्रेरी का दूसरी डिस्ट्रिब्यूशन की लाइब्रेरी से बदलना नहीं चाहिए। पैकेजों की संख्या अलग हो सकती है। निर्भरताएँ पूरी न हों तो कारण जाँचें, जबरन इंस्टॉल न करें।

```bash
apt-get --simulate install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 5. KOMPAS और एक्टिवेशन यूटिलिटी स्थापित करें

इसमें एक्टिवेशन यूटिलिटी शामिल है, जो पहले न्यूनतम इंस्टॉलेशन में छूट गई थी। पुष्टि करने से पहले APT की योजना देखें। पैकेज हटाने की ज़रूरत होने पर --no-remove, APT को रोक देता है।

```bash
sudo apt-get --no-remove install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 6. सामान्य उपयोगकर्ता के रूप में चलाएँ

ऐप को sudo से न चलाएँ। टर्मिनल आउटपुट first-launch.log में सहेजा जाता है; साझा करने से पहले व्यक्तिगत जानकारी की जाँच करें।

```bash
kompas-home-v25 2>&1 | tee ~/Downloads/kompas25/first-launch.log
```

## 7. ट्रायल सक्रिय करें

Help → Protection key utility (Справка → Утилита ключа защиты) → Trial licenses (Ознакомительные лицензии) खोलें। ट्रायल मोड चुनें, ईमेल डालें, गोपनीयता सूचना पढ़ें और सहमत हों तो सहमति का बॉक्स चुनें, फिर Activate दबाएँ। मेनू के नाम इंटरफ़ेस की भाषा के अनुसार बदल सकते हैं; इस गाइड का अनुवाद ऐप की भाषा नहीं बदलता।

## 8. समस्याओं का समाधान

**Protection key utility नहीं मिली:** KOMPAS बंद करें, नीचे दिया पैकेज स्थापित करें और ऐप फिर खोलें।

```bash
sudo apt-get --no-remove install ascon-kompas-kactivation-v25
```

**कंट्रोल एक-दूसरे पर चढ़ रहे हों या चेकबॉक्स न दबे:** Tab / Shift+Tab से चेकबॉक्स पर फ़ोकस लाएँ और Space से उसे बदलें। ज़रूरत हो तो Ubuntu Settings → Displays → Scale को अस्थायी रूप से 100% करें, यूटिलिटी और KOMPAS बंद करके दोबारा चलाएँ। ये सुझाए गए उपाय थे; उपयोगकर्ता ने सफलता की पुष्टि की, लेकिन नहीं बताया कि किस उपाय से समस्या ठीक हुई। समस्या बनी रहे तो नीचे की जानकारी इकट्ठा करें; किसी विशेष ग्राफ़िक्स लाइब्रेरी को कारण नहीं माना गया है।

```bash
echo "Session: $XDG_SESSION_TYPE"
printenv | sort | grep -E '^(QT_|GDK_|GTK_|.*SCALE|.*DPI)'
dpkg -L ascon-kompas-kactivation-v25 | grep -E '/bin/|\.desktop$|\.sh$'
```

एक्टिवेशन के बाद एक पार्ट बनाएँ, साधारण एक्सट्रूज़न करें, फ़ाइल सहेजें और फिर खोलकर अपना इंस्टॉलेशन जाँचें। ऐप न चले तो first-launch.log देखें। इस रिपॉजिटरी में केवल निर्देश हैं, ASCON के प्रोग्राम या लाइसेंस कुंजियाँ नहीं।

## आधिकारिक संदर्भ

- [ASCON — Linux installation / Astra Linux (PDF)](https://kompas.ru/source/documents/2026/Home/Install_astralinux.pdf)

- [ASCON — Home trial activation on Linux (PDF)](https://kompas.ru/source/documents/2026/Home/activ_trial_Linux_25.pdf)

- [ASCON — Home trial terms](https://kompas.ru/kompas-3d-home/download/)

- [ASCON — KOMPAS-3D v25](https://kompas.ru/kompas-3d/v25/)
