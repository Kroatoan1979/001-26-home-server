# Jellyfin — Media Server (Vídeo)

Servidor de media open-source para filmes e séries. Corre em Docker, expõe a interface web na porta 8096, e é acessível na LAN e via Tailscale.

---

## Estado

| Componente | Estado |
|---|---|
| Docker instalado no servidor | [ ] Planeado |
| `docker-compose.yml` criado | [ ] Planeado |
| Jellyfin a correr | [ ] Planeado |
| Biblioteca de media configurada | [ ] Planeado |
| Acesso via Smart TV (LAN) | [ ] Planeado |
| Acesso via Tailscale (remoto) | [ ] Planeado |

---

## Documentação

| Documento | Descrição |
|---|---|
| [01-instalacao.md](docs/01-instalacao.md) | Instalar Docker e subir o container Jellyfin |
| [02-configuracao.md](docs/02-configuracao.md) | Configuração inicial, bibliotecas e clientes |

## Configuração

| Ficheiro | Descrição |
|---|---|
| [config/docker-compose.yml](config/docker-compose.yml) | Definição do container Docker |

---

## Acesso

| Contexto | URL |
|---|---|
| LAN (em casa) | `http://192.168.1.2:8096` ou `http://homeserver.local:8096` |
| Remoto via Tailscale | `http://100.x.x.x:8096` |

---

## Requisitos

- Docker Engine instalado no servidor
- HDD externo montado em `/mnt/dados` (concluído — ver [infra/docs/04-storage-hdd-5tb.md](../../infra/docs/04-storage-hdd-5tb.md))
- Media organizada em `/mnt/dados/filmes` e `/mnt/dados/series`

---

## Pré-requisito

→ [infra/README.md](../../infra/README.md) — infraestrutura base operacional
→ [network/README.md](../../network/README.md) — Tailscale instalado (para acesso remoto)

---

*Faz parte do projeto [Home Server + Tailscale](../../README.md)*
