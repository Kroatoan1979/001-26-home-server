# Navidrome — Media Server (Música)

Servidor de música open-source com suporte ao protocolo Subsonic/OpenSubsonic. Corre em Docker, expõe a interface web na porta 4533, e é compatível com dezenas de apps mobile (DSub, Symfonium, Finamp, etc.).

---

## Estado

| Componente | Estado |
|---|---|
| Docker instalado no servidor | [ ] Planeado |
| `docker-compose.yml` criado | [ ] Planeado |
| Navidrome a correr | [ ] Planeado |
| Biblioteca de música indexada | [ ] Planeado |
| Acesso via app mobile | [ ] Planeado |

---

## Documentação

| Documento | Descrição |
|---|---|
| [01-instalacao.md](docs/01-instalacao.md) | Instalar Docker e subir o container Navidrome |

## Configuração

| Ficheiro | Descrição |
|---|---|
| [config/docker-compose.yml](config/docker-compose.yml) | Definição do container Docker |

---

## Acesso

| Contexto | URL |
|---|---|
| LAN (em casa) | `http://192.168.1.136:4533` |
| Remoto via Tailscale | `http://100.x.x.x:4533` |

### Apps compatíveis (protocolo Subsonic)

| App | Plataforma |
|---|---|
| Symfonium | Android |
| DSub | Android |
| Substreamer | iOS |
| Finamp | Android / iOS |
| Ultrasonic | Android |

---

## Requisitos

- Docker Engine instalado no servidor
- SSD externo montado em `/mnt/dados`
- Música organizada em `/mnt/dados/media/musica`

---

## Pré-requisito

→ [infra/README.md](../../infra/README.md) — infraestrutura base operacional

---

*Faz parte do projeto [Home Server + Tailscale](../../README.md)*
