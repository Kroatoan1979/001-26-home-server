# Home Server + Tailscale

Infraestrutura pessoal baseada num portátil antigo transformado em servidor headless com **RHEL**, com acesso remoto seguro via VPN mesh (Tailscale), partilha/sincronização de ficheiros via **FileCloud**, e ambiente de prática para a certificação **RHCSA**.

> ⚠️ **Migração em curso (decisão de 2026-08-05):** o servidor está a ser recriado do zero, de Rocky Linux 10 para **RHEL (Red Hat Developer Subscription for Individuals)**. Ver detalhe completo em [CLAUDE.md](CLAUDE.md).
>
> ⚠️ **Mudança de serviços (mesma data):** Jellyfin e Navidrome saíram do projeto. O serviço principal passa a ser o **FileCloud**, que consolida aqui o que estava planeado como projeto separado na AWS.

---

## Motivação

Reutilizar hardware existente (ASUS X555L com avaria na placa de som, irrelevante para servidor) para criar uma infraestrutura doméstica robusta e de baixo custo, com:

- Acesso remoto seguro a todos os dispositivos de casa
- Partilha e sincronização de ficheiros própria, via **FileCloud** — sem depender de Dropbox/Google Drive
- Controlo total dos dados — sem dependência de serviços cloud de terceiros
- Ambiente oficial de estudo/prática para a certificação **RHCSA** (o RHEL Developer é o mesmo SO usado no exame, ao contrário do Rocky Linux que é apenas um rebuild binário)
- Execução do **TraderShark** (app nativa Linux de trading desportivo em modo ladder)

---

## Hardware do Servidor

| Componente | Especificação |
|---|---|
| Modelo | ASUS X555L |
| CPU | Intel Core i7 (4ª ou 5ª geração) |
| RAM | 8 GB DDR3 |
| Disco interno | SSD (SO + serviços) |
| Disco externo | HDD 5TB WD My Passport (FileCloud + backups) |
| Rede | Ethernet ao router doméstico |
| Modo de operação | Headless por padrão — tampa fechada, sem monitor, exceto durante o uso do TraderShark |

---

## Sistema Operativo

**RHEL 10** (Red Hat Developer Subscription for Individuals) — instalação "Server with GUI" (GNOME). Versão confirmada em 2026-08-05 como a base atual do exame RHCSA — continuidade natural com o Rocky Linux 10 anterior.

- Conta gratuita: [developers.redhat.com](https://developers.redhat.com)
- Criação de pen bootável: **Rufus** (Windows) → modo DD Image
- Registo obrigatório pós-instalação: `subscription-manager register`
- Gestão de pacotes: `dnf`
- GUI (GNOME) necessária para correr o TraderShark localmente — não interfere com o SSH nem com a prática RHCSA (maioritariamente CLI)

---

## Serviços

| Serviço | Função | Porta |
|---|---|---|
| **Tailscale** | VPN mesh — acesso remoto seguro | — |
| **Docker** | Plataforma de contenção para os serviços | — |
| **FileCloud** | Partilha e sincronização de ficheiros (self-hosted) | 80 / 443 |
| **TraderShark** | App nativa de trading desportivo (modo ladder), sensível a latência | — |
| **X11 forwarding / xrdp** | Acesso gráfico remoto (Windows / tablet Android) | 3389 (xrdp) |

---

## Topologia de Rede

### Dispositivos na Tailnet

| Dispositivo | OS | Acesso |
|---|---|---|
| ASUS X555L (servidor) | RHEL (Developer Subscription) | SSH, X11 forwarding, xrdp |
| DAT-WS-IA-01 | Windows 11 Pro | RDP / SSH |
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
├── Docker + serviços (FileCloud, Tailscale)
└── Configurações e logs

HDD externo 5TB WD My Passport (ponto de montagem: /mnt/dados)
├── /mnt/dados/filecloud
├── /mnt/dados/backups
└── /mnt/dados/arquivo
```

---

## Guia de Instalação

### 1. Instalar RHEL

```bash
# Arrancar pela pen USB (BIOS: F2 → boot order)
# Selecionar: Server with GUI (GNOME) — não Minimal, por causa do TraderShark
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

### 5. Instalar Docker

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
sudo systemctl enable --now docker
sudo usermod -aG docker ivan
```

### 6. Instalar FileCloud

```bash
mkdir -p /mnt/dados/filecloud
# Ver services/filecloud/config/docker-compose.yml
cd services/filecloud/config
docker compose up -d
```

### 7. Monitorizar temperatura

```bash
sudo dnf install lm_sensors -y
sudo sensors-detect
watch -n 2 sensors
```

---

## Checklist de Progresso

> Migração Rocky Linux → RHEL em curso — instalação, SSH, headless, Avahi e Tailscale do servidor repostos a pendente porque serão refeitos do zero. O HDD externo mantém-se inalterado.

- [ ] Obter subscrição RHEL Developer e descarregar a ISO
- [ ] Criar pen USB bootável com Rufus (modo DD Image)
- [x] Instalar RHEL no ASUS X555L ("Server with GUI") — RHEL 10.2
- [x] `subscription-manager register`
- [x] Configurar modo headless (exceto uso do TraderShark) — corrigido a 2026-10-12
- [x] Instalar e configurar SSH (autenticação por chave ED25519) — falta decidir desativar a password
- [x] Configurar Avahi (mDNS) para resolução `homeserver.local` (2026-10-13; foi `homelan.local` de 2026-09-28 a 2026-10-13)
- [ ] Instalar Tailscale e integrar todos os dispositivos — tailnet pessoal desde 2026-10-13: servidor, Asus, telemóvel e tablet feitos; **falta o Acer**
- [ ] Instalar Docker
- [x] Reestruturar pastas do HDD (`filmes`/`series`/`musica` → `/mnt/dados/filecloud`) — 2026-10-11
- [ ] Instalar FileCloud via Docker e configurar portal de administração
- [x] Configurar HDD externo 5TB (feito em Rocky Linux; reposto em RHEL a 2026-10-11: `fstab` + regra `udev`)
- [ ] Testar acesso remoto via Tailscale de todos os dispositivos — Asus e telemóvel (dados móveis) OK a 2026-10-14; falta o Acer
- [ ] Configurar X11 forwarding e xrdp para acesso gráfico remoto
- [ ] Instalar e testar o TraderShark localmente
- [ ] Praticar RHCSA (CLI: utilizadores, permissões, LVM, systemd, firewalld, SELinux, containers)

> Checklist detalhado e sempre atualizado: ver `CLAUDE.md`.

---

## Notas

- Tailscale — conta Google pessoal do Ivan, plano **Personal** (gratuito, uso não comercial), desde 2026-10-13. A conta `digitalltogether.com` ficou bloqueada com "Trial ended"; motivo no `CLAUDE.md`
- **Nomes:** `<nome>.local` = LAN (mDNS), `<nome>` = tailnet (MagicDNS) — `homeserver`, `asus` e, quando migrado, `acer`
- **MagicDNS ativo** — dispositivos acessíveis pelo hostname (ex: `ssh ivan@homeserver`), sem precisar do IP Tailscale, mesmo fora da LAN
- FileCloud **consolida o projeto que estava planeado na AWS** — deixou de ser uma instância separada
- RHEL suporta sessões SSH **simultâneas ilimitadas**
- Windows 11 Pro limita a **1 sessão RDP** simultânea
- Latência local (mesmo router) < 1ms
- O **TraderShark é sensível a latência** — recomendado uso local/LAN, não X11/xrdp pela internet durante trading ativo

---

*Projeto DAT — Ivan Almeida · 2026*
