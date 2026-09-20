# Ubuntu 上的专业原生 CAD：KOMPAS-3D v25 Home 安装指南

[English](README.md) | [Русский](README.ru.md) | [简体中文](README.zh-CN.md) | [हिन्दी](README.hi.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Français](README.fr.md) | [Português](README.pt.md) | [বাংলা](README.bn.md) | [Bahasa Indonesia](README.id.md)

在 Ubuntu 中直接运行 KOMPAS-3D 的 Linux 版本，无需 Wine 或虚拟机。本社区指南记录了在 Ubuntu 26.04.1 LTS、amd64 平台上的成功安装过程，安装的是这款专业 CAD 产品的 Home 家庭版。

> ASCON 并未正式支持 Ubuntu。Home 仅供个人非商业用途；标题不代表提供商业许可证。ASCON 提供 60 天 Home 试用，试用许可证不适用于虚拟机或终端服务器。请参阅文末的官方条款。

记录日期：2026-09-20。系统为 Ubuntu 26.04.1 LTS（resolute）、amd64，KOMPAS 软件包版本为 25.0.1.2738。最初针对两个软件包的模拟安装计划新增 47 个包，不删除或升级现有包；系统依赖来自 Ubuntu。激活工具后来单独安装。用户随后确认一切正常。大型装配体、性能和长期稳定性未经过独立验证。

## 1. 检查系统并准备工具

使用 Bash，按顺序执行各代码块。架构必须是 amd64。若命令失败，请停止操作。本流程针对 Ubuntu 26.04，其他版本需要另行验证。

```bash
. /etc/os-release
printf '%s (%s)\n' "$PRETTY_NAME" "$VERSION_CODENAME"
dpkg --print-architecture
sudo apt update
sudo apt install curl gnupg ca-certificates
mkdir -p ~/Downloads/kompas25
cd ~/Downloads/kompas25
```

## 2. 下载软件源密钥

继续使用同一个终端，工作目录为 ~/Downloads/kompas25。密钥通过 HTTPS 从 ASCON 下载，并通过 signed-by 限定到对应软件源。

```bash
curl -fSL https://repo.ascon.ru/personal/deb/ascon.gpg -o ascon-personal.gpg &&
curl -fSL https://repo.ascon.ru/stable/deb/ascon.gpg -o ascon-stable.gpg &&
gpg --dearmor --yes --output ascon-personal-keyring.gpg ascon-personal.gpg &&
gpg --dearmor --yes --output ascon-stable-keyring.gpg ascon-stable.gpg &&
sudo install -d -m 0755 /etc/apt/keyrings &&
sudo install -m 0644 ascon-personal-keyring.gpg /etc/apt/keyrings/ &&
sudo install -m 0644 ascon-stable-keyring.gpg /etc/apt/keyrings/
```

## 3. 添加 ASCON 软件源

安装时两个 ASCON 软件源都只有 1.8_x86-64 分支。官方脚本会代入 Ubuntu 的 resolute，从而返回 HTTP 404。因此这里明确选择面向 Astra Linux 的 ASCON 软件包分支。不要添加 Astra Linux 操作系统本身的软件源。以下命令会覆盖指定的两个 ASCON .list 文件；如果已经使用这些软件源，请先检查并备份现有文件。如遇签名或软件源错误，请停止，不要禁用验证。

```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-personal-keyring.gpg] https://repo.ascon.ru/personal/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon-personal.list
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-stable-keyring.gpg] https://repo.ascon.ru/stable/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon.list
sudo apt update
```

## 4. 模拟安装

此命令不会修改软件包。检查完整计划：不应删除软件包、降级版本，或用其他发行版的库替换 Ubuntu 系统库。软件包数量可能不同。若依赖无法满足，请排查原因，不要强制安装。

```bash
apt-get --simulate install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 5. 安装 KOMPAS 和激活工具

这里包含了最初最小安装中遗漏的激活工具。确认前请检查 APT 的操作计划。若需要删除软件包，--no-remove 会使 APT 停止。

```bash
sudo apt-get --no-remove install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 6. 以普通用户启动

不要用 sudo 启动应用。终端输出保存到 first-launch.log；分享日志前请检查是否含有个人信息。

```bash
kompas-home-v25 2>&1 | tee ~/Downloads/kompas25/first-launch.log
```

## 7. 激活试用许可证

打开“帮助 → 保护密钥工具”（Справка → Утилита ключа защиты）→“试用许可证”（Ознакомительные лицензии）。选择试用模式，输入邮箱，阅读隐私说明；若同意，再勾选同意框并点击 Activate。菜单名称可能因界面语言而异；本指南的翻译不会改变程序的界面语言。

## 8. 故障排查

**提示找不到保护密钥工具：**关闭 KOMPAS，安装以下软件包，然后重新打开程序。

```bash
sudo apt-get --no-remove install ascon-kompas-kactivation-v25
```

**控件重叠或无法点击同意框：**用 Tab / Shift+Tab 将焦点移到复选框，再按空格切换。必要时，临时把“Ubuntu 设置 → 显示器 → 缩放”改为 100%，关闭激活工具和 KOMPAS 后重新启动。这些是当时提出的解决办法；用户确认问题已解决，但未说明具体是哪一种生效。若问题仍存在，请收集以下诊断信息；目前无法确定是哪一种图形库导致问题。

```bash
echo "Session: $XDG_SESSION_TYPE"
printenv | sort | grep -E '^(QT_|GDK_|GTK_|.*SCALE|.*DPI)'
dpkg -L ascon-kompas-kactivation-v25 | grep -E '/bin/|\.desktop$|\.sh$'
```

激活后，新建一个零件，完成简单拉伸，保存并重新打开文件，以检查自己的安装。若启动失败，请查看 first-launch.log。本仓库仅提供说明，不分发 ASCON 程序文件或许可证密钥。

## 官方参考资料

- [ASCON — Linux installation / Astra Linux (PDF)](https://kompas.ru/source/documents/2026/Home/Install_astralinux.pdf)

- [ASCON — Home trial activation on Linux (PDF)](https://kompas.ru/source/documents/2026/Home/activ_trial_Linux_25.pdf)

- [ASCON — Home trial terms](https://kompas.ru/kompas-3d-home/download/)

- [ASCON — KOMPAS-3D v25](https://kompas.ru/kompas-3d/v25/)
