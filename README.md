# KOMPAS-3D on Ubuntu: professional native CAD — v25 Home installation guide

[English](README.md) | [Русский](README.ru.md) | [简体中文](README.zh-CN.md) | [हिन्दी](README.hi.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Français](README.fr.md) | [Português](README.pt.md) | [বাংলা](README.bn.md) | [Bahasa Indonesia](README.id.md)

Run the Linux build of KOMPAS-3D directly on Ubuntu, without Wine or a virtual machine. This community guide documents a successful installation on Ubuntu 26.04.1 LTS, amd64. It installs the Home edition of the professional CAD product.

> Ubuntu is not officially supported by ASCON. Home is for personal, non-commercial use; the title does not imply a commercial license. ASCON advertises a 60-day Home trial, unavailable in virtual machines or on terminal servers. See the official terms below.

Recorded on 2026-09-20: Ubuntu 26.04.1 LTS (resolute), amd64, KOMPAS packages 25.0.1.2738. The initial two-package simulation added 47 packages, removed/upgraded none, and obtained system dependencies from Ubuntu. The activation utility was added separately. The user subsequently reported that everything worked. Large assemblies, performance and long-term stability were not independently tested.

## 1. Check the system and prepare tools

Use Bash and execute the blocks in order. The architecture must be amd64. Stop if a command fails. This procedure targets Ubuntu 26.04; other versions need separate validation.

```bash
. /etc/os-release
printf '%s (%s)\n' "$PRETTY_NAME" "$VERSION_CODENAME"
dpkg --print-architecture
sudo apt update
sudo apt install curl gnupg ca-certificates
mkdir -p ~/Downloads/kompas25
cd ~/Downloads/kompas25
```

## 2. Download the repository keys

Keep this terminal open in ~/Downloads/kompas25. Keys are downloaded from ASCON over HTTPS and scoped to their repositories with signed-by.

```bash
curl -fSL https://repo.ascon.ru/personal/deb/ascon.gpg -o ascon-personal.gpg &&
curl -fSL https://repo.ascon.ru/stable/deb/ascon.gpg -o ascon-stable.gpg &&
gpg --dearmor --yes --output ascon-personal-keyring.gpg ascon-personal.gpg &&
gpg --dearmor --yes --output ascon-stable-keyring.gpg ascon-stable.gpg &&
sudo install -d -m 0755 /etc/apt/keyrings &&
sudo install -m 0644 ascon-personal-keyring.gpg /etc/apt/keyrings/ &&
sudo install -m 0644 ascon-stable-keyring.gpg /etc/apt/keyrings/
```

## 3. Add the ASCON repositories

At the time of installation, both ASCON repositories offered only 1.8_x86-64. The vendor scripts would substitute Ubuntu's resolute and produce HTTP 404. We explicitly select the ASCON package branch for Astra Linux. Do not add Astra Linux's operating-system repositories. These commands replace the two named ASCON .list files; inspect/back up existing files if you already use ASCON repositories. Stop on signature or repository errors; do not disable verification.

```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-personal-keyring.gpg] https://repo.ascon.ru/personal/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon-personal.list
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-stable-keyring.gpg] https://repo.ascon.ru/stable/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon.list
sudo apt update
```

## 4. Simulate installation

This makes no package changes. Review the entire plan: no removals, downgrades or replacement of Ubuntu system libraries with another distribution's versions. Package counts may differ. If dependencies cannot be satisfied, stop and investigate rather than forcing installation.

```bash
apt-get --simulate install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 5. Install KOMPAS and the activation utility

Includes the activation utility that was missing from the first minimal installation. Review APT's proposed changes before confirming. --no-remove stops APT if package removal is required.

```bash
sudo apt-get --no-remove install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 6. Launch as your normal user

Do not use sudo to start the application. The terminal output is saved in first-launch.log; review it for personal details before sharing it.

```bash
kompas-home-v25 2>&1 | tee ~/Downloads/kompas25/first-launch.log
```

## 7. Activate the trial

Open Help → Protection key utility (Справка → Утилита ключа защиты) → Trial licenses (Ознакомительные лицензии). Select trial mode, enter your email, read the privacy notice and check consent if you agree, then click Activate. Menu language may differ; these translations do not change the application's interface language.

## 8. Troubleshooting

**“Protection key utility not found”:** close KOMPAS, install the package below, then reopen the application.

```bash
sudo apt-get --no-remove install ascon-kompas-kactivation-v25
```

**Overlapping controls or an inaccessible consent checkbox:** try Tab / Shift+Tab to focus the checkbox and Space to toggle it. If necessary, temporarily set Ubuntu Settings → Displays → Scale to 100%, close both applications and relaunch. These were suggested workarounds; the user confirmed success without identifying which one resolved the issue. If the problem remains, collect the following diagnostics; do not assume a particular graphics toolkit is responsible.

```bash
echo "Session: $XDG_SESSION_TYPE"
printenv | sort | grep -E '^(QT_|GDK_|GTK_|.*SCALE|.*DPI)'
dpkg -L ascon-kompas-kactivation-v25 | grep -E '/bin/|\.desktop$|\.sh$'
```

After activation, create a part, make a simple extrusion, save it and reopen it to check your own installation. If startup fails, inspect first-launch.log. This repository distributes instructions only, not ASCON binaries or license keys.

## Official references

- [ASCON — Linux installation / Astra Linux (PDF)](https://kompas.ru/source/documents/2026/Home/Install_astralinux.pdf)

- [ASCON — Home trial activation on Linux (PDF)](https://kompas.ru/source/documents/2026/Home/activ_trial_Linux_25.pdf)

- [ASCON — Home trial terms](https://kompas.ru/kompas-3d-home/download/)

- [ASCON — KOMPAS-3D v25](https://kompas.ru/kompas-3d/v25/)
