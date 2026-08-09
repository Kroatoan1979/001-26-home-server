# Network Inventory — Home Server + Tailscale

> Documento de referência rápida para todos os dispositivos da rede doméstica.
> **Manter sempre atualizado** — qualquer alteração de IP, MAC, hostname ou serviços deve ser refletida aqui imediatamente.

---

## Infraestrutura de Rede

| Campo | Valor |
|---|---|
| Subnet LAN | `192.168.1.0/24` (`255.255.255.0`) |
| Gateway / Router | `192.168.1.1` |
| Pool DHCP | `192.168.1.64 – 192.168.1.253` |
| Tailscale subnet | `100.x.x.x/8` |
| DNS local | `192.168.1.1` (router) |
| Conta Tailscale | `ivan.almeida@digitalltogether.com` (plano Free — trial de 14 dias desde 2026-07-07 terminado, decisão tomada em 2026-08-03 de manter esta conta no plano Free) |
| Painel admin Tailscale | `https://login.tailscale.com/admin/machines` |

---

## Plano de Endereçamento IP

```
192.168.1.0/24  (255.255.255.0 — 254 endereços utilizáveis)
│
├── .0          → endereço de rede (reservado — não utilizável)
├── .1          → router Vodafone (fixo — atribuído pelo próprio router)
├── .2          → homeserver ASUS X555L (IP estático no OS via nmcli — fora do pool DHCP)
│
├── .3 – .63   → zona de infraestrutura (livre — fora do pool DHCP)
│                 reservada para futuros equipamentos com IP estático
│                 (servidores, switches, APs, etc.)
│
├── .64 – .253  → pool DHCP do router Vodafone
│   ├── .100    → DAT-WS-IA-01 Wi-Fi        (reserva DHCP por MAC)
│   ├── .101    → DAT-WS-IA-01 Ethernet     (reserva DHCP por MAC)
│   ├── .110    → ZenBook UX482 Wi-Fi       (reserva DHCP por MAC)
│   ├── .111    → ZenBook UX482 Ethernet USB-C (reserva DHCP por MAC)
│   ├── .120    → Smart TV Xiaomi           (reserva DHCP por MAC — Ethernet)
│   └── .121 – .253 → livres (telemóvel pessoal, visitas, dispositivos sem reserva)
│
└── .255        → broadcast (reservado — não utilizável)
```

> **Nota:** O homeserver tem IP estático configurado diretamente no OS (via `nmcli`), **fora do pool DHCP** do router. O router desconhece este IP — a proteção contra conflito é garantida pelo facto de `.2` estar fora do range `.64–.253`.

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
| Tipo / Papel | Home Server (FileCloud + serviços) |
| Hardware | ASUS X555L |
| OS | **RHEL 10.2** (Coughlan) — reinstalação concluída em 2026-08-06/07, substitui o Rocky Linux 10 |
| IP LAN (Ethernet, `enp2s0`) | `192.168.1.2` fixo via `nmcli`, **confirmado ativo em 2026-08-09** — causa do bug anterior: perfil `nmcli` errado (`Wired connection 1`, DHCP) estava ativo em vez do perfil `enp2s0` (manual), corrigido |
| IP LAN (Wi-Fi, `wlp3s0`) | `192.168.1.146` (dinâmico, sem reserva DHCP — decisão de 2026-08-09: manter dinâmico por agora) — mantido ligado permanentemente por decisão do Ivan (2026-08-07), como via de acesso de recurso além do Ethernet |
| IP Tailscale | `100.98.31.9` (MagicDNS: `homeserver`) — novo node, confirmado a funcionar (`ssh ivan@homeserver` testado com sucesso em 2026-08-07) |
| MAC Address Ethernet | `9c:5c:8e:1e:d8:7e` (`enp2s0`) |
| MAC Address Wi-Fi | `80:a5:89:93:a6:fb` (`wlp3s0`) |
| DHCP Reservation | Ethernet: não aplicável (IP estático via SO, confirmado a funcionar) · Wi-Fi: nenhuma — decisão de 2026-08-09 de manter dinâmico por agora |
| Ligação | Ethernet (principal, ligado à rack em 2026-08-07) + Wi-Fi (permanente, fallback) |
| Serviços expostos | SSH `:22` |
| Acesso remoto | SSH por chave, **confirmado consistente em 2026-08-09** |
| Notas | Modo headless configurado (`logind.conf` + `systemctl mask` dos targets de suspensão) — passos 1–2 concluídos, falta o teste final de fechar a tampa com o servidor já na posição definitiva. Autenticação SSH por chave corrigida em 2026-08-09 (causa real: `~/.ssh` não existia no servidor, não uma inconsistência — recriado com permissões e SELinux corretos). `subscription-manager status` mostrar "Registered" é o esperado em Simple Content Access (SCA), repositórios confirmados a funcionar via `dnf repolist`. Surfshark não instalado (decisão: deixar por agora). TraderShark: **projeto pausado** (decisão do Ivan em 2026-08-07, problemas na própria aplicação) — X11 forwarding **retomado em 2026-08-09** (independente do TraderShark, em configuração), xrdp continua pausado. HDD externo 5TB WD My Passport em `/mnt/dados` (FileCloud + backups) — disco mantém-se inalterado pela migração de SO, montagem ainda por confirmar em RHEL. |

---

### Workstation — DAT-WS-IA-01

| Campo | Valor |
|---|---|
| Hostname | `DAT-WS-IA-01` |
| Tipo / Papel | Workstation principal (Ivan) |
| OS | Windows 11 Pro |
| IP LAN Wi-Fi | `192.168.1.100` |
| IP LAN Ethernet | `192.168.1.101` |
| IP Tailscale | `100.124.224.123` (MagicDNS: `dat-ws-ia-01`) |
| MAC Address Wi-Fi | `A0-02-A5-A1-A7-B6` (Killer Wi-Fi 6 AX1650i) |
| MAC Address Ethernet | `74-D4-DD-55-D8-C1` (Killer E2600 Gigabit) |
| DHCP Reservation | Concluída (Wi-Fi → `.100` · Ethernet → `.101`) |
| Ligação | Wi-Fi (Ethernet disponível) |
| Serviços expostos | RDP `:3389`, SSH `:22` |
| Acesso remoto | RDP, SSH (servidor instalado 2026-07-08; autenticação por chave ainda bloqueada — ver nota abaixo) |
| Notas | VirtualBox, Hyper-V (WSL2), Surfshark VPN instalados. IPs diferentes por adaptador (limitação do router). Perfil de rede Wi-Fi corrigido de `Public` para `Private` (2026-07-08) para a regra de firewall do OpenSSH se aplicar. ⚠️ Conta Windows é Azure AD cloud-only (`azuread\ivanalmeida`) — `sshd` não resolve esta identidade para login, chave SSH não autentica ainda (ver `CLAUDE.md`, secção "Limitação Azure AD + OpenSSH"). |

---

### Portátil — ZenBook Duo UX482

| Campo | Valor |
|---|---|
| Hostname | `UX482` |
| Tipo / Papel | Portátil secundário |
| OS | Windows 11 Home |
| IP LAN Wi-Fi | `192.168.1.110` |
| IP LAN Ethernet (USB-C) | `192.168.1.111` |
| IP Tailscale | `100.81.2.53` (MagicDNS: `ux482`) |
| MAC Address Wi-Fi | `38-FC-98-85-6F-88` (Intel Wi-Fi 6 AX200) |
| MAC Address Ethernet (USB-C) | `00-E0-4C-4D-25-20` |
| DHCP Reservation | Concluída (Wi-Fi → `.110` · Ethernet USB-C → `.111`) |
| Ligação | Wi-Fi (Ethernet via adaptador USB-C — 50% do tempo) |
| Serviços expostos | — (Windows 11 Home — não suporta RDP server) |
| Acesso remoto | — |
| Notas | Sem porta Ethernet física. Adaptador USB-C → Ethernet utilizado. Surfshark VPN instalado. Windows Home não aceita ligações RDP de entrada. |

---

### PC Kyndryl

> **Excluído do projeto** — equipamento corporativo da Kyndryl. Integração na rede doméstica excluída por risco de violação das políticas de segurança da empresa. Mantido como registo de inventário apenas.

| Campo | Valor |
|---|---|
| Hostname | `pc-kyndryl` |
| Tipo / Papel | PC de secretária (Kyndryl) — corporativo |
| OS | Windows |
| IP LAN | — (excluído do projeto) |
| IP Tailscale | — (excluído do projeto) |
| MAC Address Wi-Fi | `80-C0-1E-32-6A-95` (Intel Wi-Fi 7 BE200) |
| MAC Address Ethernet (USB-C) | — |
| DHCP Reservation | — (excluído do projeto) |
| Ligação | Wi-Fi |
| Serviços expostos | — |
| Acesso remoto | — |
| Notas | VPN corporativa Kyndryl instalada (GlobalProtect + FortiClient). Excluído por políticas corporativas. |

---

### Telemóvel (pessoal)

| Campo | Valor |
|---|---|
| Hostname | `redmi-note-14` |
| Tipo / Papel | Telemóvel pessoal |
| OS | Android 16 |
| IP LAN | Dinâmico (sem reserva) |
| IP Tailscale | `100.78.250.58` (MagicDNS: `redmi-note-14`) |
| MAC Address | — *(a preencher)* |
| DHCP Reservation | N/A — dispositivo portátil, cliente puro; IP LAN irrelevante |
| Ligação | Wi-Fi / dados móveis |
| Serviços expostos | — |
| Acesso remoto | App Tailscale, Termius (SSH via MagicDNS/IP Tailscale) |
| Notas | Acede aos serviços (ex: FileCloud), não os serve. Conectividade Tailscale confirmada também via dados móveis (fora da LAN). |

---

### Telemóvel Kyndryl

> **Excluído do projeto** — equipamento corporativo da Kyndryl. Integração na rede doméstica excluída por risco de violação das políticas de segurança da empresa. Mantido como registo de inventário apenas.

| Campo | Valor |
|---|---|
| Hostname | — |
| Tipo / Papel | Telemóvel corporativo (Kyndryl) |
| OS | — |
| IP LAN | — (excluído do projeto) |
| IP Tailscale | — (excluído do projeto) |
| MAC Address | — |
| DHCP Reservation | — (excluído do projeto) |
| Ligação | Wi-Fi |
| Serviços expostos | — |
| Acesso remoto | — |
| Notas | Excluído por políticas corporativas da Kyndryl. |

---

### Smart TV

> ⚠️ Sem papel definido no projeto desde 2026-08-05 — estava na rede apenas como cliente Jellyfin, serviço que saiu do âmbito. Mantido como registo de inventário.

| Campo | Valor |
|---|---|
| Hostname | `smarttv` |
| Tipo / Papel | Smart TV (sem função ativa no projeto — ex-cliente Jellyfin) |
| OS | Android TV (Xiaomi) |
| IP LAN | `192.168.1.120` |
| IP Tailscale | — (acesso direto na LAN — sem Tailscale) |
| MAC Address Ethernet | `04:7a:0b:fb:92:02` |
| MAC Address Wi-Fi | `40:24:b2:b0:5a:a8` |
| Número de série | `35979C48100002281` |
| DHCP Reservation | Concluída (Ethernet → `.120`) |
| Ligação | Ethernet |
| Serviços expostos | — |
| Acesso remoto | — |
| Notas | Sem função ativa desde a remoção do Jellyfin (2026-08-05). Anteriormente acedia ao Jellyfin diretamente na LAN (`:8096`). |

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
ping 192.168.1.2

# Verificar IP estático no homeserver (RHEL)
nmcli -f ipv4.method,IP4.ADDRESS,IP4.GATEWAY,IP4.DNS connection show enp2s0
```

---

*Atualizado em 2026-08-09 — Kody para Ivan / DAT — bugs de IP estático e SSH por chave resolvidos, subscrição RHEL confirmada (SCA), X11 forwarding retomado e em configuração*
