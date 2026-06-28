# Network Inventory — Home Server + Tailscale

> Documento de referência rápida para todos os dispositivos da rede doméstica.
> **Manter sempre atualizado** — qualquer alteração de IP, MAC, hostname ou serviços deve ser refletida aqui imediatamente.

---

## Infraestrutura de Rede

| Campo | Valor |
|---|---|
| Subnet LAN | `192.168.1.0/24` |
| Gateway / Router | `192.168.1.1` |
| Tailscale subnet | `100.x.x.x/8` |
| DNS local | `192.168.1.1` (router) |

---

## Inventário de Dispositivos

### Router

| Campo | Valor |
|---|---|
| Hostname | `router` |
| Tipo / Papel | Router ISP / Gateway |
| Fabricante / Modelo | Huawei OptiXstar HG8247X6-8N-10 |
| Fornecedor | Vodafone (Vodafone Smart Router) |
| Firmware | V5R020C10S230 |
| HW Version | 2EB0.E |
| Número de série | 485754431EF9B7AF |
| IP LAN | `192.168.1.1` |
| IP WAN | `100.88.200.19` |
| IP Tailscale | — (não aplicável) |
| MAC Address LAN | `28:FB:AE:C2:7A:91` |
| MAC Address Wi-Fi 2.4GHz | `28:FB:AE:C2:7A:98` |
| MAC Address Wi-Fi 5GHz | `28:FB:AE:C2:7A:9C` |
| DHCP Reservation | — (IP estático por defeito) |
| Ligação | — |
| Interface de gestão | `http://192.168.1.1` |
| Serviços expostos | DHCP, DNS, NAT, Wi-Fi 6 |
| Acesso remoto | Interface web local (`vodafone` / ver etiqueta) |
| Portas LAN | LAN1: 1000Mbps · LAN2: 1000Mbps · LAN3: 100Mbps · LAN4: 100Mbps |
| Notas | SSID 2.4GHz: `Vodafone-C27A91 2.4G` · SSID 5GHz: `Vodafone-C27A91 5G` |

---

### Servidor — ASUS X555L

| Campo | Valor |
|---|---|
| Hostname | `homeserver` |
| Tipo / Papel | Home Server (media + serviços) |
| Hardware | ASUS X555L |
| OS | Rocky Linux 10 (Minimal) |
| IP LAN | `192.168.1.136` |
| IP Tailscale | — *(a preencher após instalação Tailscale)* |
| MAC Address | `9c:5c:8e:1e:d8:7e` (Ethernet — `enp2s0`) |
| DHCP Reservation | Pendente |
| Ligação | Ethernet |
| Serviços expostos | SSH `:22`, Jellyfin `:8096`, Navidrome `:4533` |
| Acesso remoto | SSH |
| Notas | Headless (tampa fechada). SSD externo 1TB em `/mnt/dados`. |

---

### Workstation — DAT-WS-IA-01

| Campo | Valor |
|---|---|
| Hostname | `DAT-WS-IA-01` |
| Tipo / Papel | Workstation principal (Ivan) |
| OS | Windows 11 Pro |
| IP LAN | `192.168.1.10` *(reservado — pendente aplicação)* |
| IP Tailscale | — *(a preencher)* |
| MAC Address Wi-Fi | `A0-02-A5-A1-A7-B6` (Killer Wi-Fi 6 AX1650i) |
| MAC Address Ethernet | `74-D4-DD-55-D8-C1` (Killer E2600 Gigabit) |
| DHCP Reservation | Pendente (2 reservas — Wi-Fi + Ethernet → mesmo IP) |
| Ligação | Wi-Fi (Ethernet disponível mas desligada) |
| Serviços expostos | RDP `:3389` |
| Acesso remoto | RDP, SSH |
| Notas | VirtualBox, Hyper-V (WSL2), Surfshark VPN instalados. |

---

### Portátil — ZenBook Duo UX482

| Campo | Valor |
|---|---|
| Hostname | `UX482` |
| Tipo / Papel | Portátil secundário |
| OS | Windows 11 |
| IP LAN | `192.168.1.11` *(reservado — pendente aplicação)* |
| IP Tailscale | — *(a preencher)* |
| MAC Address Wi-Fi | `38-FC-98-85-6F-88` (Intel Wi-Fi 6 AX200) |
| MAC Address Ethernet (USB-C) | — *(a recolher quando adaptador ligado)* |
| DHCP Reservation | Pendente Wi-Fi · Pendente Ethernet USB-C (a recolher) |
| Ligação | Wi-Fi (Ethernet via adaptador USB-C — 50% do tempo) |
| Serviços expostos | RDP `:3389` |
| Acesso remoto | RDP, SSH |
| Notas | Sem porta Ethernet física. Adaptador USB-C → Ethernet utilizado. Surfshark VPN instalado. |

---

### PC Kyndryl

| Campo | Valor |
|---|---|
| Hostname | `pc-kyndryl` |
| Tipo / Papel | PC de secretária (Kyndryl) |
| OS | Windows |
| IP LAN | `192.168.1.12` *(reservado — pendente aplicação)* |
| IP Tailscale | — *(a preencher)* |
| MAC Address Wi-Fi | `80-C0-1E-32-6A-95` (Intel Wi-Fi 7 BE200) |
| MAC Address Ethernet (USB-C) | — *(a recolher quando adaptador ligado)* |
| DHCP Reservation | Pendente Wi-Fi · Pendente Ethernet USB-C (a recolher) |
| Ligação | Wi-Fi (Ethernet via adaptador USB-C — 50% do tempo) |
| Serviços expostos | RDP `:3389` |
| Acesso remoto | RDP |
| Notas | VPN corporativa Kyndryl instalada (GlobalProtect + FortiClient). |

---

### Telemóvel 1

| Campo | Valor |
|---|---|
| Hostname | — *(a definir)* |
| Tipo / Papel | Telemóvel pessoal |
| OS | — *(Android / iOS — a confirmar)* |
| IP LAN | — *(dinâmico Wi-Fi)* |
| IP Tailscale | — *(a preencher)* |
| MAC Address | — *(a preencher)* |
| DHCP Reservation | Pendente |
| Ligação | Wi-Fi |
| Serviços expostos | — |
| Acesso remoto | App Tailscale |
| Notas | — |

---

### Telemóvel 2

| Campo | Valor |
|---|---|
| Hostname | — *(a definir)* |
| Tipo / Papel | Telemóvel |
| OS | — *(Android / iOS — a confirmar)* |
| IP LAN | — *(dinâmico Wi-Fi)* |
| IP Tailscale | — *(a preencher)* |
| MAC Address | — *(a preencher)* |
| DHCP Reservation | Pendente |
| Ligação | Wi-Fi |
| Serviços expostos | — |
| Acesso remoto | App Tailscale |
| Notas | — |

---

### Smart TV

| Campo | Valor |
|---|---|
| Hostname | `smarttv` |
| Tipo / Papel | Smart TV (cliente Jellyfin) |
| OS | Android TV (Xiaomi) |
| IP LAN | `192.168.1.50` *(reservado — pendente aplicação)* |
| IP Tailscale | — (acesso direto na LAN — sem Tailscale) |
| MAC Address Ethernet | `04:7a:0b:fb:92:02` |
| MAC Address Wi-Fi | `40:24:b2:b0:5a:a8` |
| Número de série | `35979C48100002281` |
| DHCP Reservation | Pendente (Ethernet) |
| Ligação | Ethernet |
| Serviços expostos | — |
| Acesso remoto | — |
| Notas | Acede ao Jellyfin diretamente na LAN (`:8096`). Não precisa de Tailscale. |

---

## Comandos de Referência

```bash
# Obter MAC address (Linux)
ip link show

# Obter IP e MAC (Windows — PowerShell)
Get-NetAdapter | Select-Object Name, MacAddress, Status
ipconfig /all

# Verificar estado Tailscale
sudo tailscale status

# Ping ao servidor na LAN
ping homeserver
ping 192.168.1.136
```

---

*Atualizado em 2026-06-28 — Kody para Ivan / DAT*
