# Infra — Sistema Base

Camada de infraestrutura do servidor: sistema operativo, modo headless, SSH, acesso gráfico remoto e armazenamento externo. Tudo o que precisa de estar funcional antes de instalar qualquer serviço.

> ⚠️ **Migração Rocky Linux → RHEL em curso** (decisão de 2026-08-05). Instalação, modo headless e SSH ficam repostos a pendente — serão refeitos do zero em RHEL. O HDD externo 5TB mantém-se inalterado (mesma formatação, mesmo UUID).

---

## Estado

| Componente | Estado |
|---|---|
| RHEL instalado | [x] RHEL 10.2 (substitui o Rocky Linux 10) |
| Registo `subscription-manager` | [x] Registado (Simple Content Access) |
| Modo headless configurado | [x] Corrigido a 2026-10-12 (drop-in do `logind`) |
| SSH operacional | [x] Só por chave ED25519 desde 2026-10-15 (acer, asus, redmi-note-14; acer-kali desde 2026-10-18); só LAN e tailnet |
| Acesso gráfico remoto (X11) | [x] asus 2026-10-14, acer pelo WSL 2026-10-18 (o xrdp saiu do plano a 2026-10-18) |
| HDD externo 5TB montado | [x] Reposto em RHEL a 2026-10-11 (`/mnt/dados`, XFS, fstab, regra `udev` do hdparm recriada) |

---

## Documentação

| Documento | Descrição |
|---|---|
| [01-instalacao-rhel.md](docs/01-instalacao-rhel.md) | Instalação do SO — RHEL Developer Subscription, "Server with GUI", particionamento, utilizador |
| [02-modo-headless.md](docs/02-modo-headless.md) | Configurar o portátil para operar com tampa fechada |
| [03-ssh.md](docs/03-ssh.md) | Setup SSH, primeiro acesso remoto, autenticação por chave |
| [04-storage-hdd-5tb.md](docs/04-storage-hdd-5tb.md) | Montar o HDD externo 5TB, estrutura de pastas, gestão de energia (hdparm+udev), SMART e reposição em RHEL |
| [05-subscription-manager.md](docs/05-subscription-manager.md) | Registo da subscrição RHEL Developer (`subscription-manager`) |
| [06-acesso-grafico-x11.md](docs/06-acesso-grafico-x11.md) | X11 forwarding — sessão gráfica remota a partir do Windows |
| [08-backup-arquivo-ssd.md](docs/08-backup-arquivo-ssd.md) | Backup do arquivo pessoal (SSD externo NTFS) para `/mnt/dados/backups` com `rsync` por SSH e verificação SHA-256 |
| [09-logind-journald.md](docs/09-logind-journald.md) | Tampa fechada sem loops (`logind` por drop-in) e journal persistente |
| [10-alertas-ntfy-healthchecks.md](docs/10-alertas-ntfy-healthchecks.md) | Alertas para o telemóvel: ntfy, healthchecks.io, smartd, `OnFailure=` global, disco e corrente |

---

## Hardware

- **Modelo:** ASUS X555L
- **CPU:** Intel Core i7 (4ª/5ª geração)
- **RAM:** 8 GB DDR3
- **Disco interno:** SSD (RHEL + serviços)
- **Disco externo:** HDD 5TB WD My Passport — `/mnt/dados` (backups)
- **Rede:** Ethernet ao router doméstico

---

## Pré-requisitos

- Pen USB ≥ 8 GB
- Rufus (Windows) para criar pen bootável
- Acesso físico temporário ao portátil (teclado + monitor) apenas durante a instalação
- Cabo Ethernet ligado ao router

---

*Faz parte do projeto [Home Server + Tailscale](../README.md)*
