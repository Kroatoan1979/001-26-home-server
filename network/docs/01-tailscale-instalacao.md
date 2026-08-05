# Tailscale — Instalação no Servidor (RHEL)

> ⚠️ **Migração Rocky Linux → RHEL (2026-08-05):** este servidor vai gerar um **node Tailscale novo** ao reinstalar. Antes de autenticar aqui, remover o node antigo "homeserver" (Rocky Linux) no admin console (`https://login.tailscale.com/admin/machines`), para não ficar com dois registos do mesmo servidor.

---

## 1. Instalar Tailscale

O instalador oficial suporta RHEL diretamente:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Alternativamente, via repositório manual (o repositório já era o mesmo usado em Rocky Linux, por ser um rebuild binário do RHEL):

```bash
sudo dnf config-manager --add-repo https://pkgs.tailscale.com/stable/rhel/10/tailscale.repo
sudo dnf install -y tailscale
```

---

## 2. Ativar o serviço

```bash
sudo systemctl enable --now tailscaled

# Verificar
sudo systemctl status tailscaled
```

---

## 3. Autenticar na tailnet

```bash
sudo tailscale up
```

O comando devolve um URL. Abrir esse URL num browser e autenticar com a conta `ivan.almeida@digitalltogether.com` (a mesma conta usada em todos os dispositivos da tailnet).

---

## 4. Verificar o IP atribuído

```bash
sudo tailscale ip
# Exemplo: 100.x.x.x

sudo tailscale status
# Lista todos os dispositivos na tailnet e o seu estado
```

Anotar o IP Tailscale do servidor e atualizar a tabela em [../README.md](../README.md).

---

## 5. Testar acesso SSH via Tailscale

A partir da workstation Windows (que também já deve ter Tailscale instalado):

```powershell
ssh ivan@100.x.x.x
```

Se funcionar, o acesso remoto está operacional independentemente da rede local.

---

## 6. Firewall — verificar que o Tailscale está permitido

O instalador normalmente configura automaticamente, mas verificar:

```bash
sudo firewall-cmd --list-all
# Deve incluir a interface tailscale0
```

Se necessário:

```bash
sudo firewall-cmd --zone=trusted --add-interface=tailscale0 --permanent
sudo firewall-cmd --reload
```

---

## Próximo passo

→ [02-tailscale-dispositivos.md](02-tailscale-dispositivos.md) — adicionar os restantes dispositivos à tailnet
