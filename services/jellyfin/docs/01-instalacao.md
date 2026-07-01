# Jellyfin — Instalação

---

## 1. Instalar Docker Engine

```bash
# Adicionar repositório Docker
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

# Instalar Docker
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Ativar e iniciar
sudo systemctl enable --now docker

# Adicionar utilizador ao grupo docker (evita sudo em cada comando)
sudo usermod -aG docker ivan

# Aplicar o grupo sem logout (ou fazer logout/login)
newgrp docker

# Verificar
docker --version
docker compose version
```

---

## 2. Criar estrutura de pastas no servidor

```bash
mkdir -p ~/docker/jellyfin/config
mkdir -p ~/docker/jellyfin/cache
```

---

## 3. Criar `docker-compose.yml`

O ficheiro de configuração está em [`../config/docker-compose.yml`](../config/docker-compose.yml).

Copiar para o servidor:

```bash
# Criar a pasta e o ficheiro diretamente no servidor
mkdir -p ~/docker/jellyfin
vim ~/docker/jellyfin/docker-compose.yml
# Colar o conteúdo do ficheiro config/docker-compose.yml
```

---

## 4. Iniciar o Jellyfin

```bash
cd ~/docker/jellyfin
docker compose up -d

# Verificar que está a correr
docker ps
docker compose logs -f jellyfin
```

---

## 5. Primeiro acesso

Abrir no browser (de qualquer dispositivo na mesma rede):

```
http://192.168.1.2:8096
```

O assistente de configuração inicial irá arrancar.

---

## Próximo passo

→ [02-configuracao.md](02-configuracao.md) — configurar bibliotecas e clientes
