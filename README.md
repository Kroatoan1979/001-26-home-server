# Home Server + Tailscale

Infraestrutura pessoal baseada num portátil antigo transformado em servidor headless, com acesso remoto seguro via VPN mesh (Tailscale) e serviços de media em Docker.

---

## Motivação

Reutilizar hardware existente (ASUS X555L com avaria na placa de som, irrelevante para servidor) para criar uma infraestrutura doméstica robusta e de baixo custo, com:

- Acesso remoto seguro a todos os dispositivos de casa
- Streaming de media (filmes, séries, música) para qualquer dispositivo
- Controlo total dos dados — sem dependência de serviços cloud de terceiros

---

## Hardware do Servidor

| Componente | Especificação |
|---|---|
| Modelo | ASUS X555L |
| CPU | Intel Core i7 (4ª ou 5ª geração) |
| RAM | 8 GB DDR3 |
| Disco interno | SSD (SO + serviços) |
| Disco externo | HDD 5TB WD My Passport (media + dados) |
| Rede | Ethernet ao router doméstico |
| Modo de operação | Headless — tampa fechada, sem monitor |

---

## Sistema Operativo

**Rocky Linux 10** — Minimal ISO (x86_64)

- Download: [rockylinux.org/download](https://rockylinux.org/download)
- Criação de pen bootável: **Rufus** (Windows) → partição GPT, FAT32
- Gestão de pacotes: `dnf`

---

## Serviços

| Serviço | Função | Porta |
|---|---|---|
| **Tailscale** | VPN mesh — acesso remoto seguro | — |
| **Docker** | Plataforma de contenção para os serviços | — |
| **Jellyfin** | Media server — filmes e séries | 8096 |
| **Navidrome** | Media server — música (protocolo Subsonic) | 4533 |

---

## Topologia de Rede

### Dispositivos na Tailnet

| Dispositivo | OS | Acesso |
|---|---|---|
| ASUS X555L (servidor) | Rocky Linux 10 | SSH |
| DAT-WS-IA-01 | Windows 11 Pro | RDP / SSH |
| ZenBook Duo UX482 | Windows 11 Home | — (não aceita RDP de entrada) |
| Telemóvel (pessoal) | Android | App Tailscale |

> PC Kyndryl e Telemóvel Kyndryl excluídos do projeto — equipamentos corporativos, risco de violação de políticas de segurança da Kyndryl.

### Protocolos de Acesso Remoto

| Origem → Destino | Protocolo | Ferramenta |
|---|---|---|
| Windows → Linux | SSH | Terminal / VS Code Remote |
| Windows → Windows | RDP | mstsc (nativo) |
| Linux → Linux | SSH | Terminal |
| Telemóvel → Linux | SSH | Termius / JuiceSSH |
| Telemóvel → Windows | RDP | Microsoft RDP app |

### Smart TV (acesso ao Jellyfin)

- Smart TV e servidor ligados por **ethernet ao mesmo router** → comunicação local direta (< 1ms)
- Tailscale **não é necessário** para acesso em casa
- Endereço Jellyfin na LAN (quando instalado): `http://192.168.1.2:8096` ou `http://homeserver.local:8096`

---

## Estrutura de Armazenamento

```
SSD interno
├── Rocky Linux 10
├── Docker + serviços (Jellyfin, Navidrome)
└── Configurações e logs

HDD externo 5TB WD My Passport (ponto de montagem: /mnt/dados)
├── /mnt/dados/filmes
├── /mnt/dados/series
├── /mnt/dados/musica
├── /mnt/dados/backups
└── /mnt/dados/arquivo
```

---

## Guia de Instalação

### 1. Instalar Rocky Linux

```bash
# Arrancar pela pen USB (BIOS: F2 → boot order)
# Selecionar: Minimal Install
# Criar utilizador: ivan
# Ativar rede durante instalação
```

### 2. Pós-instalação imediata

```bash
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

### 6. Instalar Jellyfin

```bash
mkdir -p ~/docker/jellyfin
# Criar docker-compose.yml com mapeamento para /mnt/dados no HDD externo
docker compose up -d
```

### 7. Instalar Navidrome

```bash
mkdir -p ~/docker/navidrome
# Criar docker-compose.yml com mapeamento para /mnt/dados/musica
docker compose up -d
```

### 8. Monitorizar temperatura

```bash
sudo dnf install lm_sensors -y
sudo sensors-detect
watch -n 2 sensors
```

---

## Checklist de Progresso

- [x] Descarregar Minimal ISO Rocky Linux 10
- [x] Criar pen USB bootável com Rufus
- [x] Instalar Rocky Linux no ASUS X555L
- [x] Configurar modo headless
- [x] Instalar e configurar SSH (autenticação por chave ED25519)
- [x] Configurar Avahi (mDNS) para resolução `homeserver.local`
- [ ] Instalar Tailscale e integrar todos os dispositivos
- [ ] Instalar Docker
- [ ] Instalar Jellyfin via Docker
- [ ] Instalar Navidrome via Docker
- [x] Configurar HDD externo 5TB e estrutura de pastas
- [ ] Testar acesso da Smart TV ao Jellyfin
- [ ] Testar acesso remoto via Tailscale de todos os dispositivos

> Checklist detalhado e sempre atualizado: ver `CLAUDE.md`.

---

## Notas

- Tailscale plano **Personal (gratuito)** — 1 utilizador, dispositivos ilimitados
- Jellyfin e Navidrome **não incluem conteúdo** — organizam e fazem streaming do que está no HDD externo
- Rocky Linux suporta sessões SSH **simultâneas ilimitadas**
- Windows 11 Pro limita a **1 sessão RDP** simultânea
- Latência local (mesmo router) < 1ms

---

*Projeto DAT — Ivan Almeida · 2026*
