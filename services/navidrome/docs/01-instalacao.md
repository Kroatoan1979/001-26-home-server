# Navidrome — Instalação

---

## 1. Pré-requisito: Docker instalado

Se o Docker ainda não estiver instalado, seguir o passo 1 de [../../jellyfin/docs/01-instalacao.md](../../jellyfin/docs/01-instalacao.md).

---

## 2. Criar estrutura de pastas no servidor

```bash
mkdir -p ~/docker/navidrome/data
```

---

## 3. Criar `docker-compose.yml`

O ficheiro de configuração está em [`../config/docker-compose.yml`](../config/docker-compose.yml).

Criar no servidor:

```bash
mkdir -p ~/docker/navidrome
vim ~/docker/navidrome/docker-compose.yml
# Colar o conteúdo do ficheiro config/docker-compose.yml
```

---

## 4. Iniciar o Navidrome

```bash
cd ~/docker/navidrome
docker compose up -d

# Verificar
docker ps
docker compose logs -f navidrome
```

---

## 5. Primeiro acesso e utilizador admin

Abrir no browser:

```
http://192.168.1.136:4533
```

Na primeira visita, o Navidrome pede para criar um utilizador administrador:

- **Username:** `ivan`
- **Password:** (usar gestor de passwords)
- **Nome:** Ivan

---

## 6. Indexação da biblioteca

Após o login, o Navidrome começa automaticamente a indexar a pasta `/mnt/dados/media/musica`. Dependendo do tamanho da coleção, pode demorar alguns minutos.

Progresso visível em: **Painel → Indexação**

---

## 7. Configurar app mobile (Subsonic)

Exemplo com **Symfonium** (Android):

1. Adicionar servidor
2. Tipo: **Navidrome** (ou Subsonic)
3. URL: `http://100.x.x.x:4533` (via Tailscale) ou `http://192.168.1.136:4533` (LAN)
4. Username e password do passo 5

---

## Organização recomendada da música

```
/mnt/dados/media/musica/
└── Artista/
    └── Album (ano)/
        ├── 01 - Titulo.flac
        └── 02 - Titulo.flac
```

O Navidrome lê tags ID3/Vorbis automaticamente. A estrutura de pastas serve apenas de fallback.
