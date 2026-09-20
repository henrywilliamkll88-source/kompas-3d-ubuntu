# KOMPAS-3D no Ubuntu: CAD profissional nativo — instalação da v25 Home

[English](README.md) | [Русский](README.ru.md) | [简体中文](README.zh-CN.md) | [हिन्दी](README.hi.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Français](README.fr.md) | [Português](README.pt.md) | [বাংলা](README.bn.md) | [Bahasa Indonesia](README.id.md)

Execute a versão Linux do KOMPAS-3D diretamente no Ubuntu, sem Wine nem máquina virtual. Este guia da comunidade documenta uma instalação bem-sucedida no Ubuntu 26.04.1 LTS, amd64. Ele instala a edição Home do software CAD profissional.

> O Ubuntu não é oficialmente suportado pela ASCON. A edição Home destina-se ao uso pessoal não comercial; o título não implica uma licença comercial. A ASCON oferece uma avaliação Home de 60 dias, indisponível em máquinas virtuais e servidores de terminais. Consulte os termos oficiais abaixo.

Registro de 20/09/2026: Ubuntu 26.04.1 LTS (resolute), amd64, pacotes KOMPAS 25.0.1.2738. A simulação inicial de dois pacotes adicionava 47 pacotes, sem remoções nem atualizações; as dependências do sistema vinham do Ubuntu. O utilitário de ativação foi adicionado separadamente. Depois, o usuário confirmou que tudo funcionava. Grandes montagens, desempenho e estabilidade a longo prazo não foram verificados separadamente.

## 1. Verificar o sistema e preparar as ferramentas

Use Bash e execute os blocos na ordem. A arquitetura deve ser amd64. Pare se algum comando falhar. Este procedimento destina-se ao Ubuntu 26.04; outras versões exigem validação própria.

```bash
. /etc/os-release
printf '%s (%s)\n' "$PRETTY_NAME" "$VERSION_CODENAME"
dpkg --print-architecture
sudo apt update
sudo apt install curl gnupg ca-certificates
mkdir -p ~/Downloads/kompas25
cd ~/Downloads/kompas25
```

## 2. Baixar as chaves

Continue no mesmo terminal, em ~/Downloads/kompas25. As chaves são baixadas da ASCON por HTTPS e vinculadas aos respectivos repositórios com signed-by.

```bash
curl -fSL https://repo.ascon.ru/personal/deb/ascon.gpg -o ascon-personal.gpg &&
curl -fSL https://repo.ascon.ru/stable/deb/ascon.gpg -o ascon-stable.gpg &&
gpg --dearmor --yes --output ascon-personal-keyring.gpg ascon-personal.gpg &&
gpg --dearmor --yes --output ascon-stable-keyring.gpg ascon-stable.gpg &&
sudo install -d -m 0755 /etc/apt/keyrings &&
sudo install -m 0644 ascon-personal-keyring.gpg /etc/apt/keyrings/ &&
sudo install -m 0644 ascon-stable-keyring.gpg /etc/apt/keyrings/
```

## 3. Adicionar os repositórios ASCON

Durante a instalação, os dois repositórios ofereciam apenas 1.8_x86-64. Os scripts do fornecedor usariam resolute e retornariam HTTP 404. Selecionamos explicitamente a ramificação de pacotes ASCON para Astra Linux. Não adicione os repositórios do sistema operacional Astra Linux. Estes comandos substituem os dois arquivos .list indicados; confira e faça cópias se já usa repositórios ASCON. Pare diante de erros de assinatura ou repositório; não desative a verificação.

```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-personal-keyring.gpg] https://repo.ascon.ru/personal/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon-personal.list
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-stable-keyring.gpg] https://repo.ascon.ru/stable/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon.list
sudo apt update
```

## 4. Simular a instalação

Nenhum pacote será alterado. Revise todo o plano: não deve haver remoções, redução de versões ou substituição de bibliotecas Ubuntu pelas de outra distribuição. A quantidade de pacotes pode variar. Se as dependências não forem resolvidas, investigue sem forçar a instalação.

```bash
apt-get --simulate install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 5. Instalar o KOMPAS e o utilitário de ativação

Inclui o utilitário de ativação que faltava na primeira instalação mínima. Revise o plano antes de confirmar. --no-remove interrompe o APT se for necessário remover pacotes.

```bash
sudo apt-get --no-remove install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 6. Iniciar como usuário comum

Não inicie o aplicativo com sudo. A saída é salva em first-launch.log; verifique se há dados pessoais antes de compartilhá-la.

```bash
kompas-home-v25 2>&1 | tee ~/Downloads/kompas25/first-launch.log
```

## 7. Ativar a avaliação

Abra Ajuda → Utilitário da chave de proteção (Справка → Утилита ключа защиты) → Licenças de avaliação (Ознакомительные лицензии). Selecione o modo de avaliação, informe seu e-mail, leia o aviso de privacidade e marque o consentimento se concordar. Clique em Activate. Os nomes dependem do idioma da interface; estas traduções não alteram o idioma do aplicativo.

## 8. Solução de problemas

**Utilitário da chave de proteção não encontrado:** feche o KOMPAS, instale o pacote abaixo e abra novamente.

```bash
sudo apt-get --no-remove install ascon-kompas-kactivation-v25
```

**Controles sobrepostos ou caixa de seleção inacessível:** use Tab / Shift+Tab para focar a caixa e Espaço para alterná-la. Se necessário, ajuste temporariamente Configurações do Ubuntu → Monitores → Escala para 100%, feche o utilitário e o KOMPAS e abra novamente. Essas soluções foram sugeridas; o usuário confirmou o sucesso sem indicar qual resolveu o problema. Se persistir, colete os diagnósticos abaixo; nenhuma biblioteca gráfica específica foi identificada como causa.

```bash
echo "Session: $XDG_SESSION_TYPE"
printenv | sort | grep -E '^(QT_|GDK_|GTK_|.*SCALE|.*DPI)'
dpkg -L ascon-kompas-kactivation-v25 | grep -E '/bin/|\.desktop$|\.sh$'
```

Após ativar, crie uma peça, faça uma extrusão simples, salve e reabra o arquivo para verificar sua instalação. Se não iniciar, consulte first-launch.log. Este repositório distribui apenas instruções, sem binários ASCON ou chaves de licença.

## Referências oficiais

- [ASCON — Linux installation / Astra Linux (PDF)](https://kompas.ru/source/documents/2026/Home/Install_astralinux.pdf)

- [ASCON — Home trial activation on Linux (PDF)](https://kompas.ru/source/documents/2026/Home/activ_trial_Linux_25.pdf)

- [ASCON — Home trial terms](https://kompas.ru/kompas-3d-home/download/)

- [ASCON — KOMPAS-3D v25](https://kompas.ru/kompas-3d/v25/)
