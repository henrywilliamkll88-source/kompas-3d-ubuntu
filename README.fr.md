# KOMPAS-3D sur Ubuntu : CAO professionnelle native — installer v25 Home

[English](README.md) | [Русский](README.ru.md) | [简体中文](README.zh-CN.md) | [हिन्दी](README.hi.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Français](README.fr.md) | [Português](README.pt.md) | [বাংলা](README.bn.md) | [Bahasa Indonesia](README.id.md)

Exécutez la version Linux de KOMPAS-3D directement sur Ubuntu, sans Wine ni machine virtuelle. Ce guide communautaire décrit une installation réussie sur Ubuntu 26.04.1 LTS, amd64. Il installe l’édition Home de ce logiciel de CAO professionnel.

> Ubuntu n’est pas officiellement pris en charge par ASCON. Home est réservé à un usage personnel non commercial ; le titre ne signifie pas qu’une licence commerciale est fournie. ASCON propose un essai Home de 60 jours, indisponible dans les machines virtuelles et sur les serveurs de terminaux. Consultez les conditions officielles ci-dessous.

Constat du 20/09/2026 : Ubuntu 26.04.1 LTS (resolute), amd64, paquets KOMPAS 25.0.1.2738. La simulation initiale avec deux paquets ajoutait 47 paquets, sans suppression ni mise à niveau ; les dépendances système provenaient d’Ubuntu. L’utilitaire d’activation a été ajouté séparément. L’utilisateur a ensuite confirmé que tout fonctionnait. Les grands assemblages, les performances et la stabilité à long terme n’ont pas été vérifiés séparément.

## 1. Vérifier le système et préparer les outils

Utilisez Bash et exécutez les blocs dans l’ordre. L’architecture doit être amd64. Arrêtez-vous en cas d’erreur. Cette procédure vise Ubuntu 26.04 ; les autres versions nécessitent une validation distincte.

```bash
. /etc/os-release
printf '%s (%s)\n' "$PRETTY_NAME" "$VERSION_CODENAME"
dpkg --print-architecture
sudo apt update
sudo apt install curl gnupg ca-certificates
mkdir -p ~/Downloads/kompas25
cd ~/Downloads/kompas25
```

## 2. Télécharger les clés

Restez dans le même terminal, dans ~/Downloads/kompas25. Les clés sont téléchargées depuis ASCON en HTTPS et associées à leurs dépôts avec signed-by.

```bash
curl -fSL https://repo.ascon.ru/personal/deb/ascon.gpg -o ascon-personal.gpg &&
curl -fSL https://repo.ascon.ru/stable/deb/ascon.gpg -o ascon-stable.gpg &&
gpg --dearmor --yes --output ascon-personal-keyring.gpg ascon-personal.gpg &&
gpg --dearmor --yes --output ascon-stable-keyring.gpg ascon-stable.gpg &&
sudo install -d -m 0755 /etc/apt/keyrings &&
sudo install -m 0644 ascon-personal-keyring.gpg /etc/apt/keyrings/ &&
sudo install -m 0644 ascon-stable-keyring.gpg /etc/apt/keyrings/
```

## 3. Ajouter les dépôts ASCON

Lors de l’installation, les deux dépôts proposaient seulement 1.8_x86-64. Les scripts du fournisseur utiliseraient resolute et produiraient une erreur HTTP 404. Nous choisissons explicitement la branche des paquets ASCON pour Astra Linux. N’ajoutez pas les dépôts du système d’exploitation Astra Linux. Ces commandes remplacent les deux fichiers .list indiqués : vérifiez-les et sauvegardez-les si vous utilisez déjà des dépôts ASCON. Arrêtez-vous en cas d’erreur de signature ou de dépôt ; ne désactivez pas la vérification.

```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-personal-keyring.gpg] https://repo.ascon.ru/personal/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon-personal.list
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-stable-keyring.gpg] https://repo.ascon.ru/stable/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon.list
sudo apt update
```

## 4. Simuler l’installation

Aucun paquet n’est modifié. Vérifiez le plan complet : aucune suppression, rétrogradation ou substitution de bibliothèques Ubuntu par celles d’une autre distribution. Le nombre de paquets peut varier. Si des dépendances manquent, cherchez la cause sans forcer l’installation.

```bash
apt-get --simulate install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 5. Installer KOMPAS et l’utilitaire d’activation

L’utilitaire d’activation absent de la première installation minimale est inclus. Vérifiez le plan avant de confirmer. --no-remove arrête APT si une suppression est nécessaire.

```bash
sudo apt-get --no-remove install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 6. Lancer avec votre compte habituel

Ne lancez pas l’application avec sudo. La sortie est enregistrée dans first-launch.log ; vérifiez l’absence de données personnelles avant de la partager.

```bash
kompas-home-v25 2>&1 | tee ~/Downloads/kompas25/first-launch.log
```

## 7. Activer l’essai

Ouvrez Aide → Utilitaire de clé de protection (Справка → Утилита ключа защиты) → Licences d’essai (Ознакомительные лицензии). Choisissez le mode d’essai, saisissez votre adresse électronique, lisez l’avis de confidentialité et cochez le consentement si vous l’acceptez, puis cliquez sur Activate. Les noms dépendent de la langue de l’interface ; traduire ce guide ne change pas celle du logiciel.

## 8. Résolution des problèmes

**Utilitaire de clé de protection introuvable :** fermez KOMPAS, installez le paquet ci-dessous et relancez le logiciel.

```bash
sudo apt-get --no-remove install ascon-kompas-kactivation-v25
```

**Éléments superposés ou case inaccessible :** utilisez Tab / Maj+Tab pour atteindre la case et Espace pour la basculer. Au besoin, réglez temporairement Paramètres Ubuntu → Écrans → Échelle sur 100 %, fermez l’utilitaire et KOMPAS, puis relancez-les. Ce sont des solutions proposées : l’utilisateur a confirmé le succès sans préciser laquelle avait aidé. Si le problème persiste, recueillez les informations suivantes ; aucune bibliothèque graphique précise n’a été identifiée comme cause.

```bash
echo "Session: $XDG_SESSION_TYPE"
printenv | sort | grep -E '^(QT_|GDK_|GTK_|.*SCALE|.*DPI)'
dpkg -L ascon-kompas-kactivation-v25 | grep -E '/bin/|\.desktop$|\.sh$'
```

Après activation, créez une pièce, réalisez une extrusion simple, enregistrez puis rouvrez le fichier pour vérifier votre installation. Si le lancement échoue, consultez first-launch.log. Ce dépôt contient uniquement des instructions, sans binaires ASCON ni clés de licence.

## Références officielles

- [ASCON — Linux installation / Astra Linux (PDF)](https://kompas.ru/source/documents/2026/Home/Install_astralinux.pdf)

- [ASCON — Home trial activation on Linux (PDF)](https://kompas.ru/source/documents/2026/Home/activ_trial_Linux_25.pdf)

- [ASCON — Home trial terms](https://kompas.ru/kompas-3d-home/download/)

- [ASCON — KOMPAS-3D v25](https://kompas.ru/kompas-3d/v25/)
