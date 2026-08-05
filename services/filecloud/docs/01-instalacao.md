# FileCloud — Instalação

> **Estado: planeado, ainda não executado.** Esqueleto criado em 2026-08-05, quando o FileCloud substituiu o Jellyfin/Navidrome como serviço principal do homeserver (consolidando o que estava planeado como projeto AWS separado).

---

## O que é

FileCloud é uma plataforma self-hosted de partilha e sincronização de ficheiros — equivalente a um Dropbox/ownCloud privado, com portal web para utilizadores e uma consola de administração separada. A **Community Edition** é gratuita e pensada para uso pessoal/pequenas equipas.

---

## Aplicabilidade

Útil para quem quer controlo total sobre onde os ficheiros ficam guardados (em vez de num serviço cloud de terceiros), com acesso via browser ou clientes de sincronização, e partilha de ficheiros/pastas por link.

---

## Pertinência para este projeto

Estava planeado inicialmente como um projeto à parte, a correr na AWS. Decisão de 2026-08-05: consolidar no homeserver em vez de manter duas infraestruturas separadas — o homeserver já tem Tailscale para acesso remoto seguro, Docker para contenção, e o HDD 5TB para armazenamento, o que cobre os requisitos do FileCloud sem custos de cloud.

Esta mudança substitui o Jellyfin e o Navidrome (media server), que saem do âmbito do projeto.

---

## Passos previstos

### 1. Instalar Docker (ver [infra/README.md](../../../infra/README.md))

### 2. Criar a estrutura de armazenamento dedicada

```bash
sudo mkdir -p /mnt/dados/filecloud
sudo chown -R ivan:ivan /mnt/dados/filecloud
```

### 3. Subir o container

```bash
cd services/filecloud/config
docker compose up -d
```

- `--privileged` (já definido no `docker-compose.yml`) — exigido pela imagem oficial da FileCloud, que gere internamente alguns dos seus próprios serviços (Apache, MongoDB, Solr) dentro do container

### 4. Aceder à configuração inicial

- Portal de administração: `http://homeserver/ui/admin/index.html`
- Definir password de admin, licença Community Edition, e configurar o armazenamento apontado para `/filecloud-data` (mapeado para `/mnt/dados/filecloud`)

### 5. Testar o portal de utilizador

- `http://homeserver/ui/core/index.html`

---

## Teste de validação

> Por executar — atualizar esta secção com o resultado real.

- [ ] Container arranca sem erros (`docker compose logs -f`)
- [ ] Portal de administração acessível e configurado
- [ ] Upload/download de um ficheiro de teste via portal de utilizador
- [ ] Acesso confirmado via Tailscale, fora da LAN
- [ ] Avaliar se o desempenho da imagem Docker (pensada para trial/teste, segundo a documentação oficial) é suficiente para uso doméstico contínuo, ou se compensa migrar para instalação nativa

---

## Comandos de referência (troubleshooting)

```bash
# Ver logs do container
docker compose logs -f filecloud

# Reiniciar o serviço
docker compose restart filecloud

# Verificar estado do container
docker ps --filter name=filecloud
```

---

*Faz parte do projeto [Home Server + Tailscale](../../../README.md)*
