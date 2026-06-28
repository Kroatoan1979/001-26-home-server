# Infra — Sistema Base

Camada de infraestrutura do servidor: sistema operativo, modo headless, SSH e armazenamento externo. Tudo o que precisa de estar funcional antes de instalar qualquer serviço.

---

## Estado

| Componente | Estado |
|---|---|
| Rocky Linux 10 instalado | [ ] Planeado |
| Modo headless configurado | [ ] Planeado |
| SSH operacional | [ ] Planeado |
| SSD externo montado | [ ] Planeado |

---

## Documentação

| Documento | Descrição |
|---|---|
| [01-instalacao-rocky-linux.md](docs/01-instalacao-rocky-linux.md) | Instalação do SO — Minimal ISO, particionamento, utilizador |
| [02-modo-headless.md](docs/02-modo-headless.md) | Configurar o portátil para operar com tampa fechada |
| [03-ssh.md](docs/03-ssh.md) | Setup SSH, primeiro acesso remoto, autenticação por chave |
| [04-ssd-externo.md](docs/04-ssd-externo.md) | Montar o SSD externo 1 TB e estrutura de pastas de media |

---

## Hardware

- **Modelo:** ASUS X555L
- **CPU:** Intel Core i7 (4ª/5ª geração)
- **RAM:** 8 GB DDR3
- **Disco interno:** SSD (Rocky Linux + serviços Docker)
- **Disco externo:** SSD 1 TB — `/mnt/dados` (media + backups)
- **Rede:** Ethernet ao router doméstico

---

## Pré-requisitos

- Pen USB ≥ 8 GB
- Rufus (Windows) para criar pen bootável
- Acesso físico temporário ao portátil (teclado + monitor) apenas durante a instalação
- Cabo Ethernet ligado ao router

---

*Faz parte do projeto [Home Server + Tailscale](../README.md)*
