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
        WS["DAT-WS-IA-01\nWindows 11 Pro"]
        ZB["ZenBook Duo UX482\nWindows 11 Home"]
        TM1["Telemóvel (pessoal)\nAndroid"]
    end

    Router <-->|WAN| Internet
    Router -->|Ethernet| Server
    Router -->|Ethernet| TV

    Server <-->|"Tailscale VPN"| Internet
    WS <-->|"Tailscale VPN"| Internet
    ZB <-->|"Tailscale VPN"| Internet
    TM1 <-->|"Tailscale VPN"| Internet

    WS -->|"SSH :22"| Server
    ZB -->|"SSH :22"| Server
    TM1 -->|"FileCloud :443"| Server

    classDef linux fill:#2d6a4f,color:#fff,stroke:#1b4332
    classDef windows fill:#0d6efd,color:#fff,stroke:#084298
    classDef mobile fill:#6f42c1,color:#fff,stroke:#432874
    classDef network fill:#495057,color:#fff,stroke:#212529
    classDef tv fill:#6c757d,color:#fff,stroke:#343a40

    class Server linux
    class WS,ZB windows
    class TM1 mobile
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

        subgraph Docker["Docker Engine"]
            FileCloud["FileCloud\n:80 / :443\nPartilha de ficheiros"]
        end
    end

    subgraph Storage["Armazenamento"]
        SSD_int["SSD Interno\nSO + Docker + configs"]
        subgraph HDD_ext["HDD Externo 5TB — /mnt/dados"]
            FCData["/mnt/dados/filecloud"]
            Backups["/mnt/dados/backups"]
            Arquivo["/mnt/dados/arquivo"]
        end
    end

    Docker -.->|"instalado em"| SSD_int

    FileCloud -->|"volume"| FCData

    WinPC["Windows PC\nTailscale SSH"] -->|"SSH"| SSHD
    Phone["Telemóvel\nTailscale"] -->|"HTTPS"| FileCloud
    Browser["Browser\nqualquer dispositivo"] -->|"HTTP/HTTPS"| FileCloud

    classDef service fill:#2d6a4f,color:#fff,stroke:#1b4332
    classDef storage fill:#6c757d,color:#fff,stroke:#343a40
    classDef client fill:#0d6efd,color:#fff,stroke:#084298

    class FileCloud,SSHD,Tailscaled service
    class SSD_int,FCData,Backups,Arquivo storage
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

    D->>S: SSH / RDP / HTTP(S) FileCloud (encriptado)
```

---

*Atualizado em 2026-08-05 — Kody para Ivan / DAT — labels do servidor atualizados de Rocky Linux 10 para RHEL; Jellyfin/Navidrome substituídos por FileCloud; Smart TV marcada como sem função ativa*
