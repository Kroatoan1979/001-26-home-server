# Diagramas de Rede — Home Server + Tailscale

---

## 1. Topologia de Rede

Visão geral da infraestrutura: LAN doméstica, Tailscale VPN mesh e protocolos de acesso por dispositivo.

```mermaid
graph TB
    Internet(("Internet"))

    subgraph LAN["LAN Doméstica — 192.168.1.0/24"]
        Router["Router\n192.168.1.1"]
        Server["ASUS X555L\nRHEL\n192.168.1.2"]
        TV["Smart TV\n(sem função ativa)"]
    end

    subgraph Tailnet["Tailscale Mesh — 100.x.x.x/8"]
        WS["acer (Acer Predator)\nWindows 11 Pro"]
        ZB["asus (ZenBook Duo UX482)\nOmarchy"]
        TM1["Telemóvel (pessoal)\nAndroid"]
        TB1["Tablet (Galaxy Tab S8)\nAndroid"]
    end

    Router <-->|WAN| Internet
    Router -->|Ethernet| Server
    Router -->|Ethernet| TV

    Server <-->|"Tailscale VPN"| Internet
    WS <-->|"Tailscale VPN"| Internet
    ZB <-->|"Tailscale VPN"| Internet
    TM1 <-->|"Tailscale VPN"| Internet
    TB1 <-->|"Tailscale VPN"| Internet

    WS -->|"SSH :22"| Server
    ZB -->|"SSH :22"| Server
    TM1 -->|"SSH :22 (Termius)"| Server
    TB1 -->|"SSH :22 (Termius)"| Server

    classDef linux fill:#2d6a4f,color:#fff,stroke:#1b4332
    classDef windows fill:#0d6efd,color:#fff,stroke:#084298
    classDef mobile fill:#6f42c1,color:#fff,stroke:#432874
    classDef network fill:#495057,color:#fff,stroke:#212529
    classDef tv fill:#6c757d,color:#fff,stroke:#343a40

    class Server linux
    class WS,ZB windows
    class TM1,TB1 mobile
    class Router,Internet network
    class TV tv
```

> **Nota:** A comunicação via Tailscale é peer-to-peer sempre que possível (sem passar pelos servidores DERP). A Internet aparece no diagrama apenas como representação simplificada do caminho VPN.

---

## 2. Arquitetura de Serviços do Servidor

Serviços a correr no ASUS X555L e mapeamento de volumes para o armazenamento externo.

```mermaid
graph TB
    subgraph Server["ASUS X555L — RHEL"]
        Tailscaled["tailscaled\nVPN mesh"]
        SSHD["sshd\n:22"]

        Cockpit["Cockpit\n:9090\nsó LAN + tailnet"]
        TraderScope["TraderScope\n(planeado, 24/7)"]
    end

    subgraph Storage["Armazenamento"]
        SSD_int["SSD Interno\nSO + serviços + configs"]
        subgraph HDD_ext["HDD Externo 5TB — /mnt/dados"]
            Backups["/mnt/dados/backups"]
        end
    end

    TraderScope -.->|"instalado em"| SSD_int

    WinPC["acer / asus\nTailscale ou LAN"] -->|"SSH / X11"| SSHD
    Phone["Telemóvel / tablet\nTailscale"] -->|"SSH"| SSHD
    Browser["Browser\nLAN ou tailnet"] -->|"HTTPS"| Cockpit

    classDef service fill:#2d6a4f,color:#fff,stroke:#1b4332
    classDef storage fill:#6c757d,color:#fff,stroke:#343a40
    classDef client fill:#0d6efd,color:#fff,stroke:#084298

    class Cockpit,SSHD,Tailscaled,TraderScope service
    class SSD_int,Backups storage
    class WinPC,Phone,Browser client
```

---

## 3. Fluxo de Acesso — Tailscale

Como um dispositivo externo acede ao servidor via Tailscale.

```mermaid
sequenceDiagram
    participant D as Dispositivo<br/>(PC / Telemóvel)
    participant TS as Tailscale<br/>Coordination Server
    participant S as Servidor<br/>ASUS X555L

    D->>TS: Autenticação (login.tailscale.com)
    TS-->>D: IP Tailscale atribuído (100.x.x.x)
    TS-->>S: IP Tailscale atribuído (100.x.x.x)

    D->>S: Tentativa de ligação direta (peer-to-peer WireGuard)
    alt Ligação direta possível
        S-->>D: Túnel WireGuard direto estabelecido
    else Necessário relay
        D->>TS: Reencaminhamento via DERP relay
        TS->>S: Pacotes encaminhados
        S-->>D: Resposta via DERP
    end

    D->>S: SSH / X11 / HTTPS Cockpit (encriptado)
```

---

*Atualizado em 2026-10-18 — Kody para Ivan / DAT — FileCloud, xrdp e restantes projetos retirados; o único serviço planeado é o TraderScope (antes: 2026-08-05, Rocky Linux 10 → RHEL, Jellyfin/Navidrome → FileCloud)*
