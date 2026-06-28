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
| ZenBook Duo UX482 | Windows 11 | RDP / SSH |
| PC Windows 3 | Windows | RDP |
| Telemóvel 1 | Android/iOS | App Tailscale |
| Telemóvel 2 | Android/iOS | App Tailscale |

### IPs e Hostnames

| Dispositivo | IP LAN | IP Tailscale |
|---|---|---|
| Servidor (ASUS X555L) | `192.168.1.136` | `100.x.x.x` (a confirmar após Tailscale) |

> Preencher após instalação e integração na tailnet.

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
ssh ivan@192.168.1.136

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
- [ ] Configurar autenticação por chave SSH (ED25519) e desativar autenticação por password
- [ ] Configurar DHCP Reservation no router para todos os nodes da rede (homeserver, DAT-WS-IA-01, ZenBook, etc.)
- [ ] Actualizar toda a documentação para referenciar nodes com hostname + IP (ex: `homeserver (192.168.1.136)`)
- [ ] Instalar Tailscale e integrar todos os dispositivos
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
