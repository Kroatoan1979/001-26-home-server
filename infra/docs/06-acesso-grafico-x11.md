# X11 Forwarding — Acesso Gráfico Remoto (Windows)

> **Estado: planeado, ainda não executado.** Este documento é um esqueleto criado antes da reinstalação do servidor (decisão de 2026-08-05) — a preencher com os resultados reais assim que o passo for executado.

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

## Teste de validação

> Por executar — atualizar esta secção com o resultado real.

- [ ] `xclock` (ou app equivalente) aparece no ecrã do Windows via VcXsrv
- [ ] TraderShark corre via X11 forwarding sem erros gráficos relevantes (aceitando alguma latência de renderização)

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
