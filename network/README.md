# Network — Tailscale VPN Mesh

Camada de rede do home server. O Tailscale cria uma rede privada virtual (mesh) entre todos os dispositivos, permitindo acesso remoto seguro ao servidor sem expor portas ao exterior.

> ✅ **Migração Rocky Linux → RHEL concluída para o Tailscale** (2026-08-07). Node antigo removido, node novo `homeserver` autenticado em RHEL, MagicDNS testado e a funcionar (`ssh ivan@homeserver` operacional). O Avahi continua por repetir — ver estado abaixo.

---

## Estado

| Componente | Estado |
|---|---|
| Avahi (mDNS) no servidor | [x] Concluído — `homeserver.local` (desde 2026-10-13; `deny-interfaces=tailscale0`) |
| Tailscale instalado no servidor | [x] Concluído — 2026-08-07, RHEL 10.2 |
| Servidor autenticado na tailnet | [x] Concluído — node `homeserver` (foi `hometail` até 2026-10-13), IP `100.112.247.104` (tailnet pessoal, 2026-10-13; antes `100.98.31.9`), expiração da chave desativada |
| MagicDNS testado (`ssh ivan@homeserver`) | [x] Concluído — 2026-08-07, confirmado a partir do DAT-WS-IA-01 |
| Acer (`acer`, ex-DAT-WS-IA-01) na tailnet | [x] Concluído — `100.82.47.47` (2026-10-14; na antiga era `100.124.224.123`) |
| ZenBook Duo UX482 na tailnet | [x] Concluído — `100.99.98.113` (2026-10-13) |
| Telemóvel (pessoal) na tailnet | [x] Concluído — `100.108.10.121` (2026-10-13) |
| Tablet (pessoal) na tailnet | [x] Concluído — `100.119.33.58` (2026-10-13); acesso fora de casa (hotspot) testado a 2026-10-21: Cockpit + SSH por chave |

---

## Documentação

| Documento | Descrição |
|---|---|
| [00-avahi-mdns.md](docs/00-avahi-mdns.md) | Avahi (mDNS) — resolução `homeserver.local` independente do Tailscale |
| [01-tailscale-instalacao.md](docs/01-tailscale-instalacao.md) | Instalar e configurar Tailscale no servidor RHEL |
| [02-tailscale-dispositivos.md](docs/02-tailscale-dispositivos.md) | Adicionar todos os dispositivos à tailnet |
| [03-firewalld-restricao-origem.md](docs/03-firewalld-restricao-origem.md) | firewalld: SSH e Cockpit só da LAN e da tailnet (rich rules por origem) — estudo |

---

## Conta Tailscale

- **Plano:** Personal (gratuito, uso não comercial) — conta Google pessoal do Ivan, desde 2026-10-13. A conta anterior (`digitalltogether.com`, Free desde 2026-08-03) ficou bloqueada com "Trial ended"; motivo e alternativas no `CLAUDE.md` (Estado atual)
- **Email:** conta Gmail pessoal (omitida — repositório público)
- **Painel admin:** `https://login.tailscale.com/admin/machines`
- **MagicDNS ativo** — resolução por hostname (`homeserver`, `asus`, `acer`, `redmi-note-14`, `tab-s8-de-ivan`; LAN pelo mesmo nome com `.local`) dentro da tailnet, confirmada mesmo fora da LAN (dados móveis)

---

## Como funciona

```
Dispositivo A  ←──── WireGuard P2P ────→  Servidor
     |                                        |
     └─── Tailscale Coordination Server ──────┘
          (apenas para auth e descoberta)
```

O tráfego entre dispositivos é peer-to-peer via WireGuard quando possível. O Coordination Server do Tailscale nunca vê o tráfego — apenas facilita o handshake inicial.

---

## IPs Tailscale

| Dispositivo | IP Tailscale | MagicDNS |
|---|---|---|
| ASUS X555L (servidor) | `100.112.247.104` | `homeserver` |
| Acer Predator | `100.82.47.47` | `acer` |
| ZenBook Duo UX482 | `100.99.98.113` | `asus` |
| Telemóvel (Redmi Note 14) | `100.108.10.121` | `redmi-note-14` |
| Tablet (Galaxy Tab S8) | `100.119.33.58` | `tab-s8-de-ivan` |

---

## Pré-requisito

→ [infra/README.md](../infra/README.md) — servidor deve ter SSH operacional antes de instalar Tailscale

---

*Faz parte do projeto [Home Server + Tailscale](../README.md)*
