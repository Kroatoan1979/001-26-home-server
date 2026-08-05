# FileCloud — Partilha e Sincronização de Ficheiros

Plataforma self-hosted de partilha/sincronização de ficheiros (alternativa tipo Dropbox/ownCloud). Corre em Docker, expõe portal de utilizador e de administração via HTTP/HTTPS.

> **Consolidação de projeto (2026-08-05):** este serviço estava planeado como projeto separado ("FileCloud na AWS"). Decisão tomada de o centralizar aqui no homeserver em vez de numa instância AWS à parte — substitui o Jellyfin e o Navidrome, que saem do âmbito do projeto.

---

## Estado

| Componente | Estado |
|---|---|
| Docker instalado no servidor | [ ] Planeado |
| `docker-compose.yml` criado | [ ] Planeado |
| FileCloud a correr | [ ] Planeado |
| Portal admin configurado | [ ] Planeado |
| Estrutura de pastas/utilizadores definida | [ ] Planeado |
| Acesso via Tailscale (remoto) | [ ] Planeado |

---

## Documentação

| Documento | Descrição |
|---|---|
| [01-instalacao.md](docs/01-instalacao.md) | Instalar Docker e subir o container FileCloud (Community Edition) |

## Configuração

| Ficheiro | Descrição |
|---|---|
| [config/docker-compose.yml](config/docker-compose.yml) | Definição do container Docker |

---

## Acesso

| Contexto | URL |
|---|---|
| Portal utilizador | `http://homeserver/ui/core/index.html` (LAN) ou via Tailscale |
| Portal administração | `http://homeserver/ui/admin/index.html` |

> ⚠️ A documentação oficial do FileCloud nota que a imagem Docker é pensada sobretudo para avaliação/teste, não otimizada para produção — a validar se é suficiente para uso doméstico 24/7, ou se compensa avaliar a instalação nativa (não containerizada) mais tarde.

---

## Requisitos

- Docker Engine instalado no servidor
- HDD externo montado em `/mnt/dados` (concluído — ver [infra/docs/04-storage-hdd-5tb.md](../../infra/docs/04-storage-hdd-5tb.md))
- Estrutura de armazenamento dedicada em `/mnt/dados/filecloud`

---

## Pré-requisito

→ [infra/README.md](../../infra/README.md) — infraestrutura base operacional
→ [network/README.md](../../network/README.md) — Tailscale instalado (para acesso remoto)

---

*Faz parte do projeto [Home Server + Tailscale](../../README.md)*
