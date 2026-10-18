# X11 Forwarding — Acesso Gráfico Remoto (Windows)

> **Estado: concluído (2026-10-18).** Funciona a partir do Omarchy (`ssh -Y`, 2026-10-14) e do `acer` pelo WSL Kali + WSLg (`ssh -X`, 2026-10-18). O VcXsrv deixou de ser preciso e foi desinstalado do Windows a 2026-10-18. As secções sobre o VcXsrv ficam como histórico. Por testar só o TraderShark (pausado).

---

## O que é

**X11 forwarding** é uma funcionalidade do protocolo SSH que encapsula o tráfego gráfico do **X Window System** (o servidor de janelas usado pelo GNOME/Linux) dentro do túnel SSH já encriptado. Em vez de mover só texto/comandos, o `ssh -X` permite correr uma aplicação gráfica na máquina remota (o servidor) e ver a janela a aparecer no ecrã da máquina local (o cliente), sem exportar todo o ambiente de trabalho (ao contrário do RDP/xrdp, que transmite o ecrã inteiro).

Do lado do cliente é preciso um **X server** — no Linux/macOS já vem incluído; no Windows precisa de ser instalado separadamente (ex: **VcXsrv** ou **Xming**), porque o Windows não tem X Window System nativo.

---

## Aplicabilidade

Útil sempre que se quer correr **uma aplicação gráfica específica** de uma máquina remota Linux, sem precisar de todo o ambiente de trabalho — típico em administração remota de servidores Linux com ferramentas gráficas pontuais (ex: `wireshark`, um instalador gráfico, ou neste caso o TraderShark). Mais leve que um desktop remoto completo, mas só funciona enquanto a ligação SSH estiver ativa.

---

## Pertinência para este projeto

O TraderShark é uma app gráfica nativa Linux, sensível a latência. O uso primário previsto é **local** (tampa aberta, sessão GNOME direta) ou em **LAN por ethernet**. O X11 forwarding cobre o cenário secundário: aceder à interface do TraderShark (ou outra ferramenta gráfica pontual) a partir da workstation Windows (`DAT-WS-IA-01`) sem sair de casa, sem precisar de um desktop remoto completo como o xrdp.

Alternativa avaliada: usar sempre xrdp para tudo o que é gráfico. Rejeitada como única solução porque o xrdp transmite o ambiente de trabalho inteiro (mais overhead), enquanto o X11 forwarding é mais leve para uma app pontual — os dois ficam disponíveis, cada um para o seu cenário (xrdp pensado para o tablet Android, que não tem alternativa nativa ao RDP).

---

## Comandos previstos

### 1. No servidor (RHEL) — instalar o suporte a X11 forwarding

```bash
sudo dnf install -y xorg-x11-xauth
```

- `xorg-x11-xauth` — fornece o `xauth`, necessário para o `sshd` autorizar a ligação X11 encaminhada (sem isto, o `ssh -X` falha com "xauth not found")

### 2. Ativar X11 forwarding no `sshd_config` (se não estiver já ativo por omissão)

```bash
sudo grep -i X11Forwarding /etc/ssh/sshd_config
# Se necessário, definir:
# X11Forwarding yes
sudo systemctl restart sshd
```

### 3. No cliente Windows — instalar um X server

- Descarregar e instalar **VcXsrv** (ou Xming)
- Arrancar o VcXsrv com **XLaunch**, perfil "Multiple windows", **Disable access control** ativado (necessário para aceitar ligações do túnel SSH)

### 4. Ligar com X11 forwarding

```powershell
ssh -X ivan@homeserver
```

- `-X` — ativa o X11 forwarding para esta sessão (existe também `-Y` para "trusted forwarding", com menos restrições de segurança do X — só usar se `-X` falhar por causa de verificações de segurança do X11)

### 5. Testar

```bash
# No servidor, depois de ligado com ssh -X
xclock
```

- Se o VcXsrv estiver a correr no Windows, deve aparecer uma janela de relógio no ecrã do Windows — confirma que o encaminhamento gráfico está a funcionar antes de testar com o TraderShark

---

## Progresso real (2026-08-09)

- [x] **Passo 1 — `xorg-x11-xauth` no servidor:** já vinha instalado por omissão na instalação "Server with GUI" (`xorg-x11-xauth-1.1.2-8.el10.x86_64` confirmado via `rpm -qa`). Nada a fazer.
- [x] **Passo 2 — `X11Forwarding` no `sshd_config`:** já vem `yes` por omissão no RHEL 10 (`sudo grep -i X11Forwarding /etc/ssh/sshd_config` confirma). Nada a fazer, sem necessidade de restart do `sshd`.
- [x] **Passo 3 — VcXsrv no Windows (DAT-WS-IA-01):** instalado via `winget install --id marha.VcXsrv -e`. Em vez do assistente gráfico XLaunch, arrancado diretamente por linha de comando com as flags equivalentes ao perfil pretendido: `vcxsrv.exe :0 -multiwindow -clipboard -wgl -ac` (`-multiwindow` = perfil "Multiple windows"; `-ac` = "Disable access control"). Confirmado a correr via `tasklist`. Não precisa de regra de firewall — o `ssh -X` tunela o tráfego X11 pela ligação SSH já estabelecida, não por ligação direta à rede.
- [ ] **Passo 4/5 — `ssh -X` + `xclock`:** ainda por testar. Dois obstáculos encontrados no caminho, ambos por resolver na próxima sessão:
  1. O pacote `xorg-x11-apps` (que traria o `xclock` para o teste) **não existe nos repositórios do RHEL 10** — foi descontinuado/renomeado nesta versão. Por pesquisar alternativa (`dnf search xclock`/`xcalc`/`xterm`).
  2. **Achado à parte, não específico do X11:** a chave GPG do repositório `tailscale-stable` nunca ficou importada de forma persistente — qualquer `dnf` que toque nesse repo pede confirmação `[y/N]` da chave, o que bloqueia scripts/automação não-interativa. Corrigir com `sudo rpm --import https://pkgs.tailscale.com/stable/rhel/10/repo.gpg` antes de repetir a pesquisa de pacotes.

## Teste de validação

### 2026-10-14 — a partir do Omarchy (`asus`, cliente Linux, sem VcXsrv)

```bash
ssh -Y homeserver gnome-calculator
```

- Resultado: a janela da calculadora do servidor abre no ecrã local (XWayland no Hyprland; confirmado pelo Ivan). O comando termina quando se fecha a janela.
- Os dois obstáculos de 2026-08-09 ficaram resolvidos: não é preciso o `xorg-x11-apps` (a `gnome-calculator` e o `gnome-text-editor` já vêm no "Server with GUI"), e a chave GPG do Tailscale foi importada.
- **`-X` não funciona com apps GTK4:** o modo *untrusted* esconde a extensão XInput2 e a app termina com "XInput2 support not found". Com `-Y` (*trusted*) funciona. Risco do `-Y`: a app remota pode ler o teclado e o ecrã do X local, por isso só se usa com hosts de confiança (este servidor é do Ivan).

### 2026-10-18 — a partir do `acer`, pelo WSL (Kali Linux + WSLg, sem VcXsrv)

O WSLg (servidor gráfico do WSL2, incluído no Windows 11) faz de X server local, por isso o VcXsrv deixou de ser preciso. A Kali tem chave SSH própria (`ivan@acer-kali`, ver [03-ssh.md](03-ssh.md)).

```bash
# 1. Canal X11 (sem app gráfica)
ssh -X homeserver 'echo "homeserver DISPLAY=$DISPLAY"; xauth list'
# 2. Janela real
ssh -X homeserver xterm
```

- Passo 1: `DISPLAY=localhost:10.0` e um cookie `MIT-MAGIC-COOKIE-1`. O aviso `No xauth data; using fake authentication data` é inofensivo: o WSLg não cria `~/.Xauthority` na Kali, e o ssh gera um cookie próprio.
- Passo 2: a janela do `xterm` abriu no Windows e `hostname` respondeu `homeserver` (confirmado pelo Ivan). As letras ficam pequenas (fontes bitmap `misc` num ecrã com escala alta), o que é só estético.
- Instalou-se o `xterm` (`sudo dnf install -y xterm`, AppStream; entra com `libXaw`, `mkfontscale` e `xorg-x11-fonts-misc`, 8,4 MB) por ser X11 puro: funciona com `-X` (*untrusted*), ao contrário das apps GTK4, que precisam de `-Y`. O `xeyes`, o `xclock` e o `xlogo` não existem nos repositórios do RHEL 10.
- O VcXsrv instalado no Windows em 2026-08-09 ficou sem uso e o Ivan desinstalou-o a 2026-10-18 (`winget uninstall --id marha.VcXsrv -e`).

### Por testar

- [x] App de teste aparece no ecrã do `acer` — 2026-10-18, pelo WSL/WSLg (ver acima), não pelo VcXsrv
- [ ] TraderShark corre via X11 forwarding sem erros gráficos relevantes (aceitando alguma latência de renderização) — nota: TraderShark continua pausado, este ponto só se aplica se/quando for retomado

---

## Comandos de referência (troubleshooting)

```bash
# Confirmar que a variável DISPLAY está definida na sessão SSH
echo $DISPLAY

# Ver se o X11Forwarding está ativo na configuração corrente do sshd
sudo sshd -T | grep x11forwarding

# Testar com -Y se -X for bloqueado por restrições de segurança do X
ssh -Y ivan@homeserver
```

---

*Faz parte do projeto [Home Server + Tailscale](../../README.md)*
