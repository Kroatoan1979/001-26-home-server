# Diagramas de Rede — Home Server + Tailscale

---

## 1. Topologia de Rede

Visão geral da infraestrutura: LAN doméstica, Tailscale VPN mesh e protocolos de acesso por dispositivo.

```mermaid
graph TB
    Internet(("Internet"))

    subgraph LAN["LAN Doméstica — 192.168.1.0/24"]
        Router["Router\n192.168.1.1"]
        Server["ASUS X555L\nRocky Linux 10\n192.168.1.136"]
        TV["Smart TV\nJellyfin App"]
    end

    subgraph Tailnet["Tailscale Mesh — 100.x.x.x/8"]
        WS["DAT-WS-IA-01\nWindows 11 Pro"]
        ZB["ZenBook Duo UX482\nWindows 11"]
        PC3["PC Windows 3"]
        TM1["Telemóvel 1\nAndroid/iOS"]
        TM2["Telemóvel 2\nAndroid/iOS"]
    end

    Router <-->|WAN| Internet
    Router -->|Ethernet| Server
    Router -->|Ethernet| TV

    TV -->|"Jellyfin HTTP :8096\ndireto na LAN"| Server

    Server <-->|"Tailscale VPN"| Internet
    WS <-->|"Tailscale VPN"| Internet
    ZB <-->|"Tailscale VPN"| Internet
    PC3 <-->|"Tailscale VPN"| Internet
    TM1 <-->|"Tailscale VPN"| Internet
    TM2 <-->|"Tailscale VPN"| Internet

    WS -->|"SSH :22"| Server
    ZB -->|"SSH :22"| Server
    PC3 -->|"RDP :3389"| Server
    TM1 -->|"SSH :22"| Server
    TM2 -->|"Jellyfin :8096"| Server

    classDef linux fill:#2d6a4f,color:#fff,stroke:#1b4332
    classDef windows fill:#0d6efd,color:#fff,stroke:#084298
    classDef mobile fill:#6f42c1,color:#fff,stroke:#432874
    classDef network fill:#495057,color:#fff,stroke:#212529
    classDef tv fill:#e76f51,color:#fff,stroke:#c1440e

    class Server linux
    class WS,ZB,PC3 windows
    class TM1,TM2 mobile
    class Router,Internet network
    class TV tv
```

> **Nota:** A comunicação via Tailscale é peer-to-peer sempre que possível (sem passar pelos servidores DERP). A Internet aparece no diagrama apenas como representação simplificada do caminho VPN.

---

## 2. Arquitetura de Serviços do Servidor

Serviços a correr no ASUS X555L e mapeamento de volumes para o armazenamento externo.

```mermaid
graph TB
    subgraph Server["ASUS X555L — Rocky Linux 10"]
        Tailscaled["tailscaled\nVPN mesh"]
        SSHD["sshd\n:22"]

        subgraph Docker["Docker Engine"]
            Jellyfin["Jellyfin\n:8096\nMedia server"]
            Navidrome["Navidrome\n:4533\nMúsica / Subsonic"]
        end
    end

    subgraph Storage["Armazenamento"]
        SSD_int["SSD Interno\nSO + Docker + configs"]
        subgraph SSD_ext["SSD Externo 1 TB"]
            Filmes["/media/filmes"]
            Series["/media/series"]
            Musica["/media/musica"]
            Backups["/backups"]
        end
    end

    Docker -.->|"instalado em"| SSD_int

    Jellyfin -->|"volume"| Filmes
    Jellyfin -->|"volume"| Series
    Navidrome -->|"volume"| Musica

    TV["Smart TV\nLAN :8096"] -->|"HTTP streaming"| Jellyfin
    WinPC["Windows PC\nTailscale SSH"] -->|"SSH"| SSHD
    Phone["Telemóvel\nTailscale"] -->|"Subsonic API"| Navidrome
    Browser["Browser\nqualquer dispositivo"] -->|"HTTP :8096"| Jellyfin

    classDef service fill:#2d6a4f,color:#fff,stroke:#1b4332
    classDef storage fill:#6c757d,color:#fff,stroke:#343a40
    classDef client fill:#0d6efd,color:#fff,stroke:#084298
    classDef media fill:#495057,color:#fff,stroke:#212529

    class Jellyfin,Navidrome,SSHD,Tailscaled service
    class SSD_int,Filmes,Series,Musica,Backups,Backups storage
    class TV,WinPC,Phone,Browser client
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

    D->>S: SSH / RDP / HTTP :8096 (encriptado)
```

---

*Atualizado em 2026-06-26 — Kody para Ivan / DAT*
