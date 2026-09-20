# KOMPAS-3D en Ubuntu: CAD profesional nativo — instalación de v25 Home

[English](README.md) | [Русский](README.ru.md) | [简体中文](README.zh-CN.md) | [हिन्दी](README.hi.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Français](README.fr.md) | [Português](README.pt.md) | [বাংলা](README.bn.md) | [Bahasa Indonesia](README.id.md)

Ejecuta la versión Linux de KOMPAS-3D directamente en Ubuntu, sin Wine ni máquina virtual. Esta guía comunitaria documenta una instalación satisfactoria en Ubuntu 26.04.1 LTS, amd64. Instala la edición Home del producto CAD profesional.

> ASCON no admite oficialmente Ubuntu. Home es para uso personal no comercial; el título no implica una licencia comercial. ASCON ofrece una prueba de Home de 60 días, no disponible en máquinas virtuales ni servidores de terminales. Consulta las condiciones oficiales al final.

Registro del 20-09-2026: Ubuntu 26.04.1 LTS (resolute), amd64 y paquetes KOMPAS 25.0.1.2738. La simulación inicial de dos paquetes añadía 47 paquetes sin eliminar ni actualizar otros; las dependencias del sistema procedían de Ubuntu. La utilidad de activación se instaló después. El usuario confirmó que todo funcionaba. No se verificaron por separado los grandes ensamblajes, el rendimiento ni la estabilidad a largo plazo.

## 1. Comprobar el sistema y preparar las herramientas

Usa Bash y ejecuta los bloques en orden. La arquitectura debe ser amd64. Detente si falla un comando. Esta guía corresponde a Ubuntu 26.04; otras versiones requieren comprobaciones adicionales.

```bash
. /etc/os-release
printf '%s (%s)\n' "$PRETTY_NAME" "$VERSION_CODENAME"
dpkg --print-architecture
sudo apt update
sudo apt install curl gnupg ca-certificates
mkdir -p ~/Downloads/kompas25
cd ~/Downloads/kompas25
```

## 2. Descargar las claves

Continúa en la misma terminal, en ~/Downloads/kompas25. Las claves se descargan de ASCON mediante HTTPS y se vinculan a cada repositorio con signed-by.

```bash
curl -fSL https://repo.ascon.ru/personal/deb/ascon.gpg -o ascon-personal.gpg &&
curl -fSL https://repo.ascon.ru/stable/deb/ascon.gpg -o ascon-stable.gpg &&
gpg --dearmor --yes --output ascon-personal-keyring.gpg ascon-personal.gpg &&
gpg --dearmor --yes --output ascon-stable-keyring.gpg ascon-stable.gpg &&
sudo install -d -m 0755 /etc/apt/keyrings &&
sudo install -m 0644 ascon-personal-keyring.gpg /etc/apt/keyrings/ &&
sudo install -m 0644 ascon-stable-keyring.gpg /etc/apt/keyrings/
```

## 3. Añadir los repositorios de ASCON

Durante la instalación, ambos repositorios solo ofrecían 1.8_x86-64. Los scripts del proveedor usarían resolute y devolverían HTTP 404. Seleccionamos explícitamente la rama de paquetes ASCON para Astra Linux. No añadas los repositorios del sistema operativo Astra Linux. Estos comandos sobrescriben los dos archivos .list indicados; revísalos y guarda una copia si ya usas repositorios ASCON. Detente ante errores de firma o del repositorio; no desactives la verificación.

```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-personal-keyring.gpg] https://repo.ascon.ru/personal/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon-personal.list
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-stable-keyring.gpg] https://repo.ascon.ru/stable/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon.list
sudo apt update
```

## 4. Simular la instalación

No se modifican paquetes. Revisa el plan completo: no debe eliminar paquetes, bajar versiones ni sustituir bibliotecas de Ubuntu por versiones de otra distribución. La cantidad de paquetes puede variar. Si faltan dependencias, investiga la causa sin forzar la instalación.

```bash
apt-get --simulate install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 5. Instalar KOMPAS y la utilidad de activación

Se incluye la utilidad de activación que faltaba en la instalación mínima inicial. Revisa el plan antes de confirmar. --no-remove detiene APT si necesita eliminar paquetes.

```bash
sudo apt-get --no-remove install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 6. Iniciar como usuario normal

No inicies la aplicación con sudo. La salida se guarda en first-launch.log; comprueba si contiene datos personales antes de compartirla.

```bash
kompas-home-v25 2>&1 | tee ~/Downloads/kompas25/first-launch.log
```

## 7. Activar la prueba

Abre Ayuda → Utilidad de la clave de protección (Справка → Утилита ключа защиты) → Licencias de prueba (Ознакомительные лицензии). Selecciona el modo de prueba, introduce tu correo, lee el aviso de privacidad y marca la casilla si estás de acuerdo. Pulsa Activate. Los nombres dependen del idioma de la interfaz; traducir esta guía no cambia el idioma de la aplicación.

## 8. Solución de problemas

**No se encuentra la utilidad de la clave de protección:** cierra KOMPAS, instala el paquete siguiente y vuelve a abrirlo.

```bash
sudo apt-get --no-remove install ascon-kompas-kactivation-v25
```

**Controles superpuestos o casilla inaccesible:** usa Tab / Shift+Tab para enfocar la casilla y Espacio para cambiarla. Si hace falta, establece temporalmente Configuración de Ubuntu → Pantallas → Escala en 100%, cierra la utilidad y KOMPAS y vuelve a iniciarlos. Son soluciones propuestas: el usuario confirmó el éxito sin precisar cuál funcionó. Si persiste el problema, recoge estos diagnósticos; no se ha identificado una biblioteca gráfica concreta como causa.

```bash
echo "Session: $XDG_SESSION_TYPE"
printenv | sort | grep -E '^(QT_|GDK_|GTK_|.*SCALE|.*DPI)'
dpkg -L ascon-kompas-kactivation-v25 | grep -E '/bin/|\.desktop$|\.sh$'
```

Tras activar, crea una pieza, haz una extrusión sencilla, guarda y vuelve a abrir el archivo para verificar tu instalación. Si no inicia, revisa first-launch.log. Este repositorio contiene instrucciones, no binarios de ASCON ni claves de licencia.

## Referencias oficiales

- [ASCON — Linux installation / Astra Linux (PDF)](https://kompas.ru/source/documents/2026/Home/Install_astralinux.pdf)

- [ASCON — Home trial activation on Linux (PDF)](https://kompas.ru/source/documents/2026/Home/activ_trial_Linux_25.pdf)

- [ASCON — Home trial terms](https://kompas.ru/kompas-3d-home/download/)

- [ASCON — KOMPAS-3D v25](https://kompas.ru/kompas-3d/v25/)
