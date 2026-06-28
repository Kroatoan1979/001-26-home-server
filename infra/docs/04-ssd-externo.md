# SSD Externo — Montagem e Estrutura de Media

O SSD externo de 1 TB é usado exclusivamente para media (filmes, séries, música) e backups. Deve ser montado automaticamente no arranque.

---

## 1. Identificar o disco

```bash
lsblk
# Procurar o disco de ~1 TB — geralmente /dev/sdb

# Mais detalhe
sudo fdisk -l /dev/sdb
```

---

## 2. Formatar (apenas se necessário)

> Se o SSD já tiver conteúdo que queiras preservar, salta este passo.

```bash
# Criar tabela de partições GPT
sudo parted /dev/sdb mklabel gpt

# Criar partição única
sudo parted /dev/sdb mkpart primary ext4 0% 100%

# Formatar em ext4
sudo mkfs.ext4 /dev/sdb1
```

---

## 3. Criar ponto de montagem

```bash
sudo mkdir -p /mnt/dados
```

---

## 4. Montar e testar

```bash
sudo mount /dev/sdb1 /mnt/dados
df -h /mnt/dados
# Deve mostrar ~1 TB disponível
```

---

## 5. Montagem automática via `/etc/fstab`

Obter o UUID do disco (mais estável que o nome `/dev/sdbX`):

```bash
sudo blkid /dev/sdb1
# Exemplo: UUID="a1b2c3d4-..."
```

Editar `/etc/fstab`:

```bash
sudo vim /etc/fstab
```

Adicionar no final:

```
UUID=a1b2c3d4-xxxx-xxxx-xxxx-xxxxxxxxxxxx  /mnt/dados  ext4  defaults,nofail  0  2
```

> A opção `nofail` evita que o servidor não arranque caso o SSD externo não esteja ligado.

Testar sem reiniciar:

```bash
sudo umount /mnt/dados
sudo mount -a
df -h /mnt/dados
```

---

## 6. Criar estrutura de pastas de media

```bash
sudo mkdir -p /mnt/dados/media/filmes
sudo mkdir -p /mnt/dados/media/series
sudo mkdir -p /mnt/dados/media/musica
sudo mkdir -p /mnt/dados/backups

# Dar ownership ao utilizador ivan
sudo chown -R ivan:ivan /mnt/dados
```

Verificar:

```bash
ls -la /mnt/dados/
```

---

## Estrutura final

```
/mnt/dados/
├── media/
│   ├── filmes/
│   ├── series/
│   └── musica/
└── backups/
```

---

## Próximo passo

→ A infraestrutura base está completa. Prosseguir para a camada de rede:
[network/README.md](../../network/README.md) — instalação do Tailscale
