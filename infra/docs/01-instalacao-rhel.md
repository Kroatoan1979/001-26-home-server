# Instalação — RHEL 10 (Red Hat Developer Subscription for Individuals)

> Substitui o Rocky Linux 10 anteriormente instalado — decisão de 2026-08-05, motivada pela certificação RHCSA (que usa o RHEL oficial, não um rebuild binário) e pela necessidade de sessão gráfica local para o TraderShark.
>
> **Versão:** RHEL 10, confirmado em 2026-08-05 como a versão atual e base do exame RHCSA — antes de agendar o exame, confirmar sempre a versão ativa na página oficial de agendamento da Red Hat, já que pode variar por região/data.

## 1. Obter a subscrição e a ISO

- Criar conta gratuita em [developers.redhat.com](https://developers.redhat.com)
- Ativar a **Red Hat Developer Subscription for Individuals** (gratuita, uso de desenvolvimento individual)
- Descarregar a ISO do **RHEL 10** (arquitetura x86_64) a partir do portal de downloads do Red Hat Developer
- Verificar o checksum SHA256 após download

---

## 2. Criar pen USB bootável (Windows)

1. Abrir **Rufus**
2. Selecionar a pen USB (≥ 8 GB)
3. Selecionar a ISO descarregada
4. Modo de imagem: **DD Image** (confirmado: desde o RHEL 8.3 que o modo ISO Image deixou de ser reconhecido corretamente pelo instalador — usar sempre DD Image para ISOs RHEL)
5. Clicar em **START** e aguardar

---

## 3. Arrancar pela pen USB

1. Ligar o ASUS X555L com monitor e teclado ligados
2. Na BIOS (tecla `F2` no arranque): alterar boot order para USB primeiro
3. Guardar e reiniciar

---

## 4. Instalação do RHEL

No instalador Anaconda, configurar:

### Idioma e teclado
- Idioma do instalador: English (ou Português se disponível)
- Layout de teclado: **Portuguese** (verificar depois com `localectl`)

### Software
- Base Environment: **Server with GUI** (GNOME) — não Minimal, porque o TraderShark é uma app gráfica nativa que precisa de correr localmente
- Não adicionar pacotes adicionais nesta fase (o resto é instalado depois via `dnf`/Docker)

### Destino de instalação
- Selecionar o **SSD interno**
- Particionamento automático é suficiente para esta instalação (LVM por defeito — útil também para a prática RHCSA)

### Rede
- Ativar a interface Ethernet durante a instalação
- Hostname: `homeserver`

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
# Registar a subscrição — obrigatório no RHEL, ao contrário do Rocky Linux
# (ver detalhe em 05-subscription-manager.md)
sudo subscription-manager register

# Atualizar todo o sistema
sudo dnf update -y

# Instalar utilitários essenciais
sudo dnf install -y vim curl wget net-tools lm_sensors

# Verificar conectividade
ping -c 3 google.com

# Verificar IP local
ip addr show
```

Configurar o IP estático via `nmcli` (`192.168.1.2/24`) — mesma configuração já usada em Rocky Linux, independente do router.

---

## 6. Verificação final

```bash
# Versão do sistema
cat /etc/redhat-release

# Uptime
uptime

# Espaço em disco
df -h

# Estado da subscrição
sudo subscription-manager status
```

---

## Próximo passo

→ [05-subscription-manager.md](05-subscription-manager.md) — detalhe do registo da subscrição, se ainda não tiver sido feito
→ [02-modo-headless.md](02-modo-headless.md) — configurar o portátil para operar sem monitor com tampa fechada
