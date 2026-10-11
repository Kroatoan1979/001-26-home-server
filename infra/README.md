# Infra — Sistema Base

Camada de infraestrutura do servidor: sistema operativo, modo headless, SSH, acesso gráfico remoto e armazenamento externo. Tudo o que precisa de estar funcional antes de instalar qualquer serviço.

> ⚠️ **Migração Rocky Linux → RHEL em curso** (decisão de 2026-08-05). Instalação, modo headless e SSH ficam repostos a pendente — serão refeitos do zero em RHEL. O HDD externo 5TB mantém-se inalterado (mesma formatação, mesmo UUID).

---

## Estado

| Componente | Estado |
|---|---|
| RHEL instalado | [ ] Pendente (substitui o Rocky Linux 10) |
| Registo `subscription-manager` | [ ] Pendente |
| Modo headless configurado | [ ] Pendente (exceto durante uso do TraderShark) |
| SSH operacional | [ ] Pendente (autenticação por chave ED25519 — nova chave de host) |
| Acesso gráfico remoto (X11 / xrdp) | [ ] Pendente |
| HDD externo 5TB montado | [x] Reposto em RHEL a 2026-10-11 (`/mnt/dados`, XFS, fstab, regra `udev` do hdparm recriada) |

---

## Documentação

| Documento | Descrição |
|---|---|
| [01-instalacao-rhel.md](docs/01-instalacao-rhel.md) | Instalação do SO — RHEL Developer Subscription, "Server with GUI", particionamento, utilizador |
| [02-modo-headless.md](docs/02-modo-headless.md) | Configurar o portátil para operar com tampa fechada (exceto uso do TraderShark) |
| [03-ssh.md](docs/03-ssh.md) | Setup SSH, primeiro acesso remoto, autenticação por chave |
| [04-storage-hdd-5tb.md](docs/04-storage-hdd-5tb.md) | Montar o HDD externo 5TB, estrutura de pastas para o FileCloud, gestão de energia (hdparm+udev), SMART e reposição em RHEL |
| [05-subscription-manager.md](docs/05-subscription-manager.md) | Registo da subscrição RHEL Developer (`subscription-manager`) |
| [06-acesso-grafico-x11.md](docs/06-acesso-grafico-x11.md) | X11 forwarding — sessão gráfica remota a partir do Windows |
| [07-acesso-grafico-xrdp.md](docs/07-acesso-grafico-xrdp.md) | xrdp — acesso gráfico remoto a partir do tablet Android |
| [08-backup-arquivo-ssd.md](docs/08-backup-arquivo-ssd.md) | Backup do arquivo pessoal (SSD externo NTFS) para `/mnt/dados/backups` com `rsync` por SSH e verificação SHA-256 |

---

## Hardware

- **Modelo:** ASUS X555L
- **CPU:** Intel Core i7 (4ª/5ª geração)
- **RAM:** 8 GB DDR3
- **Disco interno:** SSD (RHEL + serviços Docker)
- **Disco externo:** HDD 5TB WD My Passport — `/mnt/dados` (FileCloud + backups)
- **Rede:** Ethernet ao router doméstico

---

## Pré-requisitos

- Pen USB ≥ 8 GB
- Rufus (Windows) para criar pen bootável
- Acesso físico temporário ao portátil (teclado + monitor) apenas durante a instalação
- Cabo Ethernet ligado ao router

---

*Faz parte do projeto [Home Server + Tailscale](../README.md)*
