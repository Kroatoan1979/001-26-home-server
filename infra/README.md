# Infra — Sistema Base

Camada de infraestrutura do servidor: sistema operativo, modo headless, SSH e armazenamento externo. Tudo o que precisa de estar funcional antes de instalar qualquer serviço.

---

## Estado

| Componente | Estado |
|---|---|
| Rocky Linux 10 instalado | [x] Concluído |
| Modo headless configurado | [x] Concluído |
| SSH operacional | [x] Concluído (autenticação por chave ED25519) |
| HDD externo 5TB montado | [x] Concluído (`/mnt/dados`, XFS, fstab, hdparm+udev) |

---

## Documentação

| Documento | Descrição |
|---|---|
| [01-instalacao-rocky-linux.md](docs/01-instalacao-rocky-linux.md) | Instalação do SO — Minimal ISO, particionamento, utilizador |
| [02-modo-headless.md](docs/02-modo-headless.md) | Configurar o portátil para operar com tampa fechada |
| [03-ssh.md](docs/03-ssh.md) | Setup SSH, primeiro acesso remoto, autenticação por chave |
| [04-storage-hdd-5tb.md](docs/04-storage-hdd-5tb.md) | Montar o HDD externo 5TB, estrutura de pastas de media e gestão de energia (hdparm+udev) |

---

## Hardware

- **Modelo:** ASUS X555L
- **CPU:** Intel Core i7 (4ª/5ª geração)
- **RAM:** 8 GB DDR3
- **Disco interno:** SSD (Rocky Linux + serviços Docker)
- **Disco externo:** HDD 5TB WD My Passport — `/mnt/dados` (media + backups)
- **Rede:** Ethernet ao router doméstico

---

## Pré-requisitos

- Pen USB ≥ 8 GB
- Rufus (Windows) para criar pen bootável
- Acesso físico temporário ao portátil (teclado + monitor) apenas durante a instalação
- Cabo Ethernet ligado ao router

---

*Faz parte do projeto [Home Server + Tailscale](../README.md)*
