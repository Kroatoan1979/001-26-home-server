# Home Server + Tailscale

Infraestrutura pessoal baseada num portátil antigo transformado em servidor headless com **RHEL**, com acesso remoto seguro via VPN mesh (Tailscale) e ambiente de prática para a certificação **RHCSA**. O único serviço planeado é o regresso do **TraderScope** em 24/7, quando a app estiver sólida localmente.

> ⚠️ **Migração em curso (decisão de 2026-08-05):** o servidor está a ser recriado do zero, de Rocky Linux 10 para **RHEL (Red Hat Developer Subscription for Individuals)**. Ver detalhe completo em [CLAUDE.md](CLAUDE.md).
>
> ⚠️ **Âmbito reduzido (2026-10-18, decisão do Ivan):** FileCloud, Web Server + Cloudflare, Guacamole, xrdp e TraderShark saíram do plano. Fica só o TraderScope. Detalhe em [CLAUDE.md](CLAUDE.md).

---

## Motivação

Reutilizar hardware existente (ASUS X555L com avaria na placa de som, irrelevante para servidor) para criar uma infraestrutura doméstica robusta e de baixo custo, com:

- Acesso remoto seguro a todos os dispositivos de casa
- Cópia local do arquivo pessoal no HDD externo (`/mnt/dados/backups`)
- Ambiente oficial de estudo/prática para a certificação **RHCSA** (o RHEL Developer é o mesmo SO usado no exame, ao contrário do Rocky Linux que é apenas um rebuild binário)
- Execução do **TraderScope** em 24/7 (planeado, quando estiver sólido localmente)

---

## Hardware do Servidor

| Componente | Especificação |
|---|---|
| Modelo | ASUS X555L |
| CPU | Intel Core i7 (4ª ou 5ª geração) |
| RAM | 8 GB DDR3 |
| Disco interno | SSD (SO + serviços) |
| Disco externo | HDD 5TB WD My Passport (backups) |
| Rede | Ethernet ao router doméstico |
| Modo de operação | Headless — tampa fechada, sem monitor |

---

## Sistema Operativo

**RHEL 10** (Red Hat Developer Subscription for Individuals) — instalação "Server with GUI" (GNOME). Versão confirmada em 2026-08-05 como a base atual do exame RHCSA — continuidade natural com o Rocky Linux 10 anterior.

- Conta gratuita: [developers.redhat.com](https://developers.redhat.com)
- Criação de pen bootável: **Rufus** (Windows) → modo DD Image
- Registo obrigatório pós-instalação: `subscription-manager register`
- Gestão de pacotes: `dnf`
- GUI (GNOME) instalada; escolhida a pensar no TraderShark (fora do plano desde 2026-10-18), mantém-se por agora para as apps gráficas por X11 forwarding — não interfere com o SSH nem com a prática RHCSA (maioritariamente CLI)

---

## Serviços

| Serviço | Função | Porta |
|---|---|---|
| **Tailscale** | VPN mesh — acesso remoto seguro | — |
| **Cockpit** | Administração web (só LAN e tailnet) | 9090 |
| **X11 forwarding** | Acesso gráfico pontual pelo SSH (Omarchy / WSL do acer) | — |
| **TraderScope** | Planeado: app de trading desportivo em 24/7, quando estiver sólida localmente | — |

---

## Topologia de Rede

### Dispositivos na Tailnet

| Dispositivo | OS | Acesso |
|---|---|---|
| ASUS X555L (servidor) | RHEL (Developer Subscription) | SSH, X11 forwarding |
| Acer Predator (`acer`; era DAT-WS-IA-01) | Windows 11 Pro | RDP / SSH |
| ZenBook Duo UX482 (`asus`) | Omarchy (Arch Linux + Hyprland), desde 10/2026 | SSH (cliente) |
| Telemóvel (pessoal, Xiaomi Redmi Note 14) | Android | App Tailscale + Termius |
| Tablet (pessoal, Samsung Galaxy Tab S8) | Android | App Tailscale |

> PC Kyndryl e Telemóvel Kyndryl excluídos do projeto — equipamentos corporativos, risco de violação de políticas de segurança da Kyndryl.

### Protocolos de Acesso Remoto

| Origem → Destino | Protocolo | Ferramenta |
|---|---|---|
| Windows → Linux | SSH | Terminal / VS Code Remote |
| Windows → Windows | RDP | mstsc (nativo) |
| Linux → Linux | SSH | Terminal |
| Telemóvel → Linux | SSH | Termius / JuiceSSH |
| Telemóvel → Windows | RDP | Microsoft RDP app |

### Smart TV

> ⚠️ Desde 2026-08-05, a Smart TV não tem papel definido no projeto — estava na rede apenas para aceder ao Jellyfin, que saiu do âmbito. Mantém-se como dispositivo de inventário de rede (ligação Ethernet ao mesmo router), sem função ativa até se decidir um novo uso.

---

## Estrutura de Armazenamento

```
SSD interno
├── RHEL (Developer Subscription)
├── Serviços (Tailscale; TraderScope quando voltar)
└── Configurações e logs

HDD externo 5TB WD My Passport (ponto de montagem: /mnt/dados)
└── /mnt/dados/backups
```

---

## Guia de Instalação

### 1. Instalar RHEL

```bash
# Arrancar pela pen USB (BIOS: F2 → boot order)
# Selecionar: Server with GUI (GNOME) — não Minimal (escolha feita a pensar no TraderShark, entretanto fora do plano)
# Criar utilizador: ivan
# Ativar rede durante instalação
```

### 2. Pós-instalação imediata

```bash
sudo subscription-manager register
sudo dnf update -y
sudo dnf install openssh-server -y
sudo systemctl enable --now sshd
```

### 3. Modo headless (tampa fechada)

Editar `/etc/systemd/logind.conf`:

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
```

```bash
sudo systemctl restart systemd-logind
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### 4. Instalar Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo systemctl enable --now tailscaled
sudo tailscale up
# Autenticar com conta Google
# Verificar IP atribuído: sudo tailscale ip
```

### 5. Monitorizar temperatura

```bash
sudo dnf install lm_sensors -y
sudo sensors-detect
watch -n 2 sensors
```

---

## Checklist de Progresso

> Migração Rocky Linux → RHEL concluída na parte de base (SO, SSH, headless, Avahi, Tailscale, HDD). Pendentes: prática RHCSA e o regresso do TraderScope.

- [x] Obter subscrição RHEL Developer e descarregar a ISO
- [x] Criar pen USB bootável com Rufus (modo DD Image)
- [x] Instalar RHEL no ASUS X555L ("Server with GUI") — RHEL 10.2
- [x] `subscription-manager register`
- [x] Configurar modo headless — corrigido a 2026-10-12
- [x] Instalar e configurar SSH (autenticação por chave ED25519) — só por chave desde 2026-10-15 (password desligada); Cockpit e SSH limitados à LAN e à tailnet na firewall
- [x] Configurar Avahi (mDNS) para resolução `homeserver.local` (2026-10-13; foi `homelan.local` de 2026-09-28 a 2026-10-13)
- [x] Instalar Tailscale e integrar todos os dispositivos — tailnet pessoal desde 2026-10-13: servidor, Asus, telemóvel, tablet e Acer (2026-10-14) feitos; nós da tailnet antiga apagados a 2026-10-14
- [x] Reestruturar pastas do HDD — 2026-10-11 (fica só `backups/`)
- [x] Configurar HDD externo 5TB (feito em Rocky Linux; reposto em RHEL a 2026-10-11: `fstab` + regra `udev`)
- [x] Testar acesso remoto via Tailscale de todos os dispositivos — Asus e telemóvel (dados móveis) OK a 2026-10-14; Acer OK a 2026-10-14 (`ssh ivan@homeserver` pela tailnet e `ssh ivan@homeserver.local` pela LAN, sem avisos de host key); só por chave desde 2026-10-15
- [x] Configurar X11 forwarding para acesso gráfico remoto — Omarchy OK a 2026-10-14 (`ssh -Y`) e Acer a 2026-10-18 (WSL Kali + WSLg, `ssh -X`)
- [ ] Redeploy do TraderScope em 24/7, quando estiver sólido localmente
- [ ] Praticar RHCSA (CLI: utilizadores, permissões, LVM, systemd, firewalld, SELinux, containers com Podman)

> Checklist detalhado e sempre atualizado: ver `CLAUDE.md`.

---

## Notas

- Tailscale — conta Google pessoal do Ivan, plano **Personal** (gratuito, uso não comercial), desde 2026-10-13. A conta `digitalltogether.com` ficou bloqueada com "Trial ended"; motivo no `CLAUDE.md`
- **Nomes:** `<nome>.local` = LAN (mDNS), `<nome>` = tailnet (MagicDNS) — `homeserver`, `asus` e `acer`
- **MagicDNS ativo** — dispositivos acessíveis pelo hostname (ex: `ssh ivan@homeserver`), sem precisar do IP Tailscale, mesmo fora da LAN
- RHEL suporta sessões SSH **simultâneas ilimitadas**
- Windows 11 Pro limita a **1 sessão RDP** simultânea
- Latência local (mesmo router) < 1ms

---

*Projeto DAT — Ivan Almeida · 2026*
