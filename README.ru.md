# КОМПАС-3D на Ubuntu: профессиональная нативная САПР — установка v25 Home

[English](README.md) | [Русский](README.ru.md) | [简体中文](README.zh-CN.md) | [हिन्दी](README.hi.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Français](README.fr.md) | [Português](README.pt.md) | [বাংলা](README.bn.md) | [Bahasa Indonesia](README.id.md)

Запускаем Linux-версию КОМПАС-3D непосредственно в Ubuntu, без Wine и виртуальной машины. Эта инструкция сообщества основана на успешной установке в Ubuntu 26.04.1 LTS, amd64. Устанавливается домашняя редакция Home профессиональной САПР.

> Ubuntu официально не поддерживается АСКОН. Home предназначена для личного некоммерческого использования; заголовок не подразумевает коммерческую лицензию. АСКОН предлагает пробный период Home на 60 дней, недоступный в виртуальных машинах и на терминальных серверах. Официальные условия приведены по ссылкам ниже.

Зафиксировано 20.09.2026: Ubuntu 26.04.1 LTS (resolute), amd64, пакеты КОМПАС 25.0.1.2738. Первоначальная симуляция с двумя пакетами добавляла 47 пакетов без удаления и обновления существующих; системные зависимости брались из Ubuntu. Утилиту активации установили отдельно. Затем пользователь подтвердил, что всё работает. Большие сборки, производительность и длительная стабильность отдельно не проверялись.

## 1. Проверить систему и подготовить инструменты

Используйте Bash и выполняйте блоки по порядку. Архитектура должна быть amd64. При ошибке остановитесь. Инструкция рассчитана на Ubuntu 26.04; другие версии требуют отдельной проверки.

```bash
. /etc/os-release
printf '%s (%s)\n' "$PRETTY_NAME" "$VERSION_CODENAME"
dpkg --print-architecture
sudo apt update
sudo apt install curl gnupg ca-certificates
mkdir -p ~/Downloads/kompas25
cd ~/Downloads/kompas25
```

## 2. Скачать ключи репозиториев

Продолжайте в том же терминале в ~/Downloads/kompas25. Ключи скачиваются по HTTPS с сервера АСКОН и привязываются к своим репозиториям через signed-by.

```bash
curl -fSL https://repo.ascon.ru/personal/deb/ascon.gpg -o ascon-personal.gpg &&
curl -fSL https://repo.ascon.ru/stable/deb/ascon.gpg -o ascon-stable.gpg &&
gpg --dearmor --yes --output ascon-personal-keyring.gpg ascon-personal.gpg &&
gpg --dearmor --yes --output ascon-stable-keyring.gpg ascon-stable.gpg &&
sudo install -d -m 0755 /etc/apt/keyrings &&
sudo install -m 0644 ascon-personal-keyring.gpg /etc/apt/keyrings/ &&
sudo install -m 0644 ascon-stable-keyring.gpg /etc/apt/keyrings/
```

## 3. Подключить репозитории АСКОН

На момент установки оба репозитория АСКОН содержали только ветку 1.8_x86-64. Фирменные скрипты подставляли бы resolute и получали HTTP 404. Поэтому явно выбираем ветку пакетов АСКОН для Astra Linux. Репозитории самой ОС Astra Linux добавлять не нужно. Команды перезаписывают два указанных файла АСКОН .list: если вы уже используете эти репозитории, сначала проверьте и сохраните существующие файлы. При ошибках подписи или репозитория остановитесь, не отключайте проверку.

```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-personal-keyring.gpg] https://repo.ascon.ru/personal/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon-personal.list
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-stable-keyring.gpg] https://repo.ascon.ru/stable/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon.list
sudo apt update
```

## 4. Выполнить симуляцию установки

Команда не изменяет пакеты. Проверьте весь план: в нём не должно быть удаления, понижения версий или замены системных библиотек Ubuntu версиями из другой ОС. Число пакетов может отличаться. Если зависимости не разрешаются, выясните причину, не форсируйте установку.

```bash
apt-get --simulate install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 5. Установить КОМПАС и утилиту активации

Включаем утилиту активации, которой не хватило при первоначальной минимальной установке. Перед подтверждением проверьте план APT. Параметр --no-remove остановит APT, если потребуется удалять пакеты.

```bash
sudo apt-get --no-remove install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 6. Запустить от обычного пользователя

Не запускайте приложение через sudo. Вывод терминала сохраняется в first-launch.log; перед публикацией проверьте, нет ли в нём личных данных.

```bash
kompas-home-v25 2>&1 | tee ~/Downloads/kompas25/first-launch.log
```

## 7. Активировать пробную лицензию

Откройте «Справка → Утилита ключа защиты → Ознакомительные лицензии». Выберите ознакомительный режим, введите почту, прочитайте условия обработки данных и поставьте галочку, если согласны, затем нажмите «Активировать». Названия могут отличаться в зависимости от языка интерфейса; переводы этой инструкции не меняют язык программы.

## 8. Решение проблем

**«Утилита ключа защиты не найдена»:** закройте КОМПАС, установите пакет командой ниже и снова откройте программу.

```bash
sudo apt-get --no-remove install ascon-kompas-kactivation-v25
```

**Элементы перекрываются или галочка недоступна:** попробуйте Tab / Shift+Tab для перехода к галочке и Пробел для переключения. При необходимости временно установите «Настройки Ubuntu → Дисплеи → Масштаб → 100%», закройте утилиту и КОМПАС, затем запустите снова. Это предложенные способы: пользователь подтвердил успех, но не уточнил, какой помог. Если проблема сохраняется, соберите диагностику командами ниже; конкретная графическая библиотека как причина не установлена.

```bash
echo "Session: $XDG_SESSION_TYPE"
printenv | sort | grep -E '^(QT_|GDK_|GTK_|.*SCALE|.*DPI)'
dpkg -L ascon-kompas-kactivation-v25 | grep -E '/bin/|\.desktop$|\.sh$'
```

После активации создайте деталь, выполните простое выдавливание, сохраните и откройте файл повторно — так вы проверите свою установку. При сбое запуска посмотрите first-launch.log. Репозиторий содержит только инструкцию, без дистрибутивов АСКОН и лицензионных ключей.

## Официальные источники

- [ASCON — Linux installation / Astra Linux (PDF)](https://kompas.ru/source/documents/2026/Home/Install_astralinux.pdf)

- [ASCON — Home trial activation on Linux (PDF)](https://kompas.ru/source/documents/2026/Home/activ_trial_Linux_25.pdf)

- [ASCON — Home trial terms](https://kompas.ru/kompas-3d-home/download/)

- [ASCON — KOMPAS-3D v25](https://kompas.ru/kompas-3d/v25/)
