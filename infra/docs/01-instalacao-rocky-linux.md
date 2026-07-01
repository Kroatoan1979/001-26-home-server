# Instalação — Rocky Linux 10

## 1. Descarregar a ISO

- Página oficial: [rockylinux.org/download](https://rockylinux.org/download)
- Versão: **Rocky Linux 10 — Minimal ISO (x86_64)**
- Verificar o checksum SHA256 após download

---

## 2. Criar pen USB bootável (Windows)

1. Abrir **Rufus**
2. Selecionar a pen USB (≥ 8 GB)
3. Selecionar a ISO descarregada
4. Esquema de partição: **GPT**
5. Sistema de destino: **UEFI (non-CSM)**
6. Sistema de ficheiros: **FAT32**
7. Clicar em **START** e aguardar

---

## 3. Arrancar pela pen USB

1. Ligar o ASUS X555L com monitor e teclado ligados
2. Na BIOS (tecla `F2` no arranque): alterar boot order para USB primeiro
3. Guardar e reiniciar

---

## 4. Instalação do Rocky Linux

No instalador Anaconda, configurar:

### Idioma e teclado
- Idioma do instalador: English (ou Português se disponível)
- Layout de teclado: **Portuguese** (verificar depois com `localectl`)

### Software
- Base Environment: **Minimal Install**
- Não adicionar pacotes adicionais nesta fase

### Destino de instalação
- Selecionar o **SSD interno**
- Particionamento automático é suficiente para esta instalação

### Rede
- Ativar a interface Ethernet durante a instalação
- Hostname: `homeserver` (ou `asus-x555l`)

### Utilizador root e utilizador normal
- Definir password de root (guardar em gestor de passwords)
- Criar utilizador: **ivan**
- Marcar como **administrador** (para uso de `sudo`)

### Iniciar instalação

Aguardar o processo. No final, remover a pen USB e reiniciar.

---

## 5. Pós-instalação imediata

Entrar como `ivan` e executar:

```bash
# Atualizar todo o sistema
sudo dnf update -y

# Instalar utilitários essenciais
sudo dnf install -y vim curl wget net-tools lm_sensors

# Verificar conectividade
ping -c 3 google.com

# Verificar IP local
ip addr show
```

Anotar o IP local atribuído (neste projeto, configurado como estático em `192.168.1.2`) — necessário para SSH até o Tailscale estar configurado.

---

## 6. Verificação final

```bash
# Versão do sistema
cat /etc/rocky-release

# Uptime
uptime

# Espaço em disco
df -h
```

---

## Próximo passo

→ [02-modo-headless.md](02-modo-headless.md) — configurar o portátil para operar sem monitor com tampa fechada
