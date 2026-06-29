# Home Server + Tailscale — Contexto do Projeto

## Objetivo

Transformar um portátil antigo (ASUS X555L) num home server headless a correr Rocky Linux 10, acessível remotamente via Tailscale. Serviços de media (Jellyfin, Navidrome) em Docker, com armazenamento em SSD externo 1TB.

---

## Hardware do Servidor

| Componente | Especificação |
|---|---|
| Modelo | ASUS X555L |
| CPU | Intel Core i7 (4ª ou 5ª geração) |
| RAM | 8 GB DDR3 |
| Disco interno | SSD (SO + serviços) |
| Disco externo | SSD 1TB (media + dados) |
| Rede | Ethernet ao router doméstico |
| Avaria | Placa de som — irrelevante para servidor |

---

## Stack Técnico

| Camada | Tecnologia |
|---|---|
| SO | Rocky Linux 10 (Minimal) |
| VPN mesh | Tailscale (plano Personal, gratuito) |
| Contenção | Docker + docker-compose |
| Media (vídeo) | Jellyfin (porta 8096) |
| Media (música) | Navidrome (protocolo Subsonic) |
| Acesso remoto Linux | SSH |
| Gestão de pacotes | `dnf` (não `apt`) |

---

## Topologia de Rede

### Dispositivos na Tailnet

| Dispositivo | OS | Protocolo de acesso |
|---|---|---|
| ASUS X555L (servidor) | Rocky Linux 10 | SSH |
| DAT-WS-IA-01 (workstation Ivan) | Windows 11 Pro | RDP / SSH |
| ZenBook Duo UX482 | Windows 11 Home | — (não aceita RDP de entrada) |
| Telemóvel 1 | Android/iOS | App Tailscale |
| Telemóvel 2 | Android/iOS | App Tailscale |

> PC Kyndryl excluído do projeto — equipamento corporativo, risco de violação de políticas de segurança da Kyndryl.

### IPs e Hostnames

| Dispositivo | IP LAN | IP Tailscale |
|---|---|---|
| Servidor — homeserver (ASUS X555L) | `192.168.1.2` (estático no SO) | *(a preencher)* |
| DAT-WS-IA-01 — Wi-Fi | `192.168.1.100` | *(a preencher)* |
| DAT-WS-IA-01 — Ethernet | `192.168.1.101` | *(a preencher)* |
| ZenBook UX482 — Wi-Fi | `192.168.1.110` | *(a preencher)* |
| ZenBook UX482 — Ethernet USB-C | `192.168.1.111` | *(a preencher)* |
| Smart TV (Xiaomi) | `192.168.1.120` | — (sem Tailscale) |

> Pool DHCP do router (Vodafone): `192.168.1.64–253`. Reservas DHCP dentro deste range. Homeserver com IP estático via `nmcli` fora do pool.

### Acesso à tailnet

- Conta Tailscale: `ivan.almeida1979@gmail.com`
- Painel admin: `https://login.tailscale.com/admin/machines`

---

## Estrutura de Armazenamento

```
SSD interno
├── Rocky Linux 10
├── Docker + serviços (Jellyfin, Navidrome, Tailscale)
└── Configurações e logs

SSD externo 1TB (ponto de montagem: /mnt/dados ou /media)
├── /media/filmes
├── /media/series
├── /media/musica
└── /backups
```

---

## Comandos Frequentes

```bash
# Ligar ao servidor via SSH (Tailscale)
ssh ivan@100.x.x.x

# Ligar ao servidor via SSH (LAN)
ssh ivan@192.168.1.2

# Verificar estado do Tailscale
sudo tailscale status

# Verificar temperatura (headless)
watch -n 2 sensors

# Gerir serviços Docker
docker-compose up -d
docker-compose down
docker ps

# Atualizar sistema
sudo dnf update -y
```

---

## Estado do Projeto (Checklist)

- [x] Descarregar Minimal ISO Rocky Linux 10
- [x] Criar pen USB bootável com Rufus
- [x] Instalar Rocky Linux no ASUS X555L
- [x] Configurar modo headless (tampa fechada)
- [x] Instalar e configurar SSH
- [x] Configurar autenticação por chave SSH (ED25519) — chave instalada no homeserver para o utilizador `ivan`
- [ ] Desativar autenticação por password SSH — adiado para após instalação do Tailscale
- [x] Configurar IP estático no homeserver via `nmcli` (`192.168.1.2/24`) — independente do router
- [x] Configurar DHCP Reservation no router para DAT-WS-IA-01, ZenBook e Smart TV
- [x] Recolher MAC Ethernet USB-C do ZenBook (`00-E0-4C-4D-25-20`) e criar reserva
- [x] Actualizar toda a documentação para referenciar nodes com hostname + IP
- [x] Reiniciar dispositivos para aplicar novos IPs reservados e verificar com `ipconfig` — DAT-WS-IA-01 Wi-Fi `.100` ✅ · ZenBook Wi-Fi `.110` ✅ · Ethernet e telemóveis por verificar (cabos/dispositivos não disponíveis)
- [ ] Instalar Tailscale e integrar todos os dispositivos
- [ ] Configurar Devolutions RDM Free (workspace DAT) — adicionar ligações SSH e RDP para todos os nodes com IPs LAN e Tailscale
- [ ] Instalar Docker
- [ ] Instalar Jellyfin via Docker
- [ ] Instalar Navidrome via Docker
- [ ] Configurar SSD externo e estrutura de pastas
- [ ] Testar acesso da Smart TV ao Jellyfin
- [ ] Testar acesso remoto via Tailscale de todos os dispositivos

### Sub-projeto: Web Server + Cloudflare
- [ ] Sessão de estudo: conceitos de proxy e reverse proxy
- [ ] Decidir qual o web server a usar (Caddy / Nginx / Traefik)
- [ ] Transferir gestão DNS do `digitalltogether.xyz` da Hostinger para a Cloudflare
- [ ] Configurar Cloudflare Tunnel no home server
- [ ] Instalar e configurar o web server escolhido via Docker
- [ ] Configurar reverse proxy para expor o web server via `digitalltogether.xyz`
- [ ] Testar acesso público ao site de estudo

### Sub-projeto: Guacamole (Gestão de Ligações Web)
> Pré-requisitos: Docker operacional + Cloudflare Tunnel configurado
- [ ] Estudar arquitetura do Apache Guacamole (guacd + guacamole-client)
- [ ] Instalar Guacamole via Docker no homeserver
- [ ] Configurar ligações SSH e RDP para todos os nodes da rede
- [ ] Expor via Cloudflare Tunnel (ex: `remoto.digitalltogether.xyz`)
- [ ] Testar acesso via browser de dentro e fora da rede

---

## Notas Importantes

- Servidor é **headless** — após setup inicial, opera com tampa fechada, sem monitor nem teclado
- A avaria na placa de som é **completamente irrelevante** para o funcionamento como servidor
- O acesso remoto é feito **exclusivamente via Tailscale** quando fora da rede local
- Na LAN (em casa), servidor e Smart TV comunicam diretamente — Tailscale não é necessário
- Rocky Linux 10 suporta **sessões SSH simultâneas ilimitadas**
- Windows 11 Pro limita a **1 sessão RDP** simultânea
- Monitorizar temperatura nas primeiras horas com tampa fechada

---

## Convenções de Trabalho

- Língua: **Português de Portugal (PT)**
- Assistente: **Kody**
- Utilizador: **Ivan**
- Toda a documentação deste projeto é escrita em PT
