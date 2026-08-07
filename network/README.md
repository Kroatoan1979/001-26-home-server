# Network — Tailscale VPN Mesh

Camada de rede do home server. O Tailscale cria uma rede privada virtual (mesh) entre todos os dispositivos, permitindo acesso remoto seguro ao servidor sem expor portas ao exterior.

> ✅ **Migração Rocky Linux → RHEL concluída para o Tailscale** (2026-08-07). Node antigo removido, node novo `homeserver` autenticado em RHEL, MagicDNS testado e a funcionar (`ssh ivan@homeserver` operacional). O Avahi continua por repetir — ver estado abaixo.

---

## Estado

| Componente | Estado |
|---|---|
| Avahi (mDNS) no servidor | [ ] Pendente — a repetir em RHEL (era `homeserver.local` em Rocky Linux) |
| Tailscale instalado no servidor | [x] Concluído — 2026-08-07, RHEL 10.2 |
| Servidor autenticado na tailnet | [x] Concluído — novo node `homeserver`, IP `100.98.31.9` (substitui `100.97.253.85` do node Rocky Linux, removido) |
| MagicDNS testado (`ssh ivan@homeserver`) | [x] Concluído — 2026-08-07, confirmado a partir do DAT-WS-IA-01 |
| DAT-WS-IA-01 na tailnet | [x] Concluído — `100.124.224.123` |
| ZenBook Duo UX482 na tailnet | [x] Concluído — `100.81.2.53` |
| Telemóvel (pessoal) na tailnet | [x] Concluído — `100.78.250.58` |

---

## Documentação

| Documento | Descrição |
|---|---|
| [00-avahi-mdns.md](docs/00-avahi-mdns.md) | Avahi (mDNS) — resolução `homeserver.local` independente do Tailscale |
| [01-tailscale-instalacao.md](docs/01-tailscale-instalacao.md) | Instalar e configurar Tailscale no servidor RHEL |
| [02-tailscale-dispositivos.md](docs/02-tailscale-dispositivos.md) | Adicionar todos os dispositivos à tailnet |

---

## Conta Tailscale

- **Plano:** Free — conta `digitalltogether.com`. Trial de 14 dias (desde 2026-07-07) terminado; decisão tomada em 2026-08-03 de manter esta conta no plano Free, sem necessidade de plano pago nem de migrar para conta pessoal
- **Email:** `ivan.almeida@digitalltogether.com`
- **Painel admin:** `https://login.tailscale.com/admin/machines`
- **MagicDNS ativo** — resolução por hostname (`homeserver`, `dat-ws-ia-01`, `ux482`, `redmi-note-14`) dentro da tailnet, confirmada mesmo fora da LAN (dados móveis)

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
| ASUS X555L (servidor) | `100.98.31.9` | `homeserver` |
| DAT-WS-IA-01 | `100.124.224.123` | `dat-ws-ia-01` |
| ZenBook Duo UX482 | `100.81.2.53` | `ux482` |
| Telemóvel (Redmi Note 14) | `100.78.250.58` | `redmi-note-14` |

---

## Pré-requisito

→ [infra/README.md](../infra/README.md) — servidor deve ter SSH operacional antes de instalar Tailscale

---

*Faz parte do projeto [Home Server + Tailscale](../README.md)*
