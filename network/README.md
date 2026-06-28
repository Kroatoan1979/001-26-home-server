# Network — Tailscale VPN Mesh

Camada de rede do home server. O Tailscale cria uma rede privada virtual (mesh) entre todos os dispositivos, permitindo acesso remoto seguro ao servidor sem expor portas ao exterior.

---

## Estado

| Componente | Estado |
|---|---|
| Tailscale instalado no servidor | [ ] Planeado |
| Servidor autenticado na tailnet | [ ] Planeado |
| DAT-WS-IA-01 na tailnet | [ ] Planeado |
| ZenBook Duo UX482 na tailnet | [ ] Planeado |
| PC Windows 3 na tailnet | [ ] Planeado |
| Telemóvel 1 na tailnet | [ ] Planeado |
| Telemóvel 2 na tailnet | [ ] Planeado |

---

## Documentação

| Documento | Descrição |
|---|---|
| [01-tailscale-instalacao.md](docs/01-tailscale-instalacao.md) | Instalar e configurar Tailscale no servidor Rocky Linux |
| [02-tailscale-dispositivos.md](docs/02-tailscale-dispositivos.md) | Adicionar todos os dispositivos à tailnet |

---

## Conta Tailscale

- **Plano:** Personal (gratuito)
- **Email:** `ivan.almeida1979@gmail.com`
- **Painel admin:** `https://login.tailscale.com/admin/machines`
- **Dispositivos ilimitados** no plano Personal

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

## IPs Tailscale (preencher após instalação)

| Dispositivo | IP Tailscale |
|---|---|
| ASUS X555L (servidor) | `100.x.x.x` |
| DAT-WS-IA-01 | `100.x.x.x` |
| ZenBook Duo UX482 | `100.x.x.x` |

---

## Pré-requisito

→ [infra/README.md](../infra/README.md) — servidor deve ter SSH operacional antes de instalar Tailscale

---

*Faz parte do projeto [Home Server + Tailscale](../README.md)*
