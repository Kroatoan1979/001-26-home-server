# SSH — Configuração e Acesso Remoto

O SSH é o principal método de administração do servidor após a instalação. Objetivos: serviço ativo no arranque, acesso por chave (sem password), e configuração segura básica.

---

## 1. Ativar o serviço SSH

```bash
sudo systemctl enable --now sshd

# Verificar estado
sudo systemctl status sshd
```

---

## 2. Abrir a porta na firewall

```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload

# Verificar
sudo firewall-cmd --list-services
```

---

## 3. Primeiro acesso a partir do Windows

No terminal da workstation `DAT-WS-IA-01`:

```powershell
ssh ivan@192.168.1.136
```

Aceitar a fingerprint do servidor quando solicitado (apenas na primeira ligação).

---

## 4. Autenticação por chave (recomendado)

### Gerar par de chaves na workstation Windows (se ainda não existir)

```powershell
ssh-keygen -t ed25519 -C "ivan@dat-ws-ia-01"
# Guardar em: C:\Users\IvanAlmeida\.ssh\id_ed25519
```

### Copiar a chave pública para o servidor

```powershell
# Método 1 — ssh-copy-id (disponível no Git Bash / WSL)
ssh-copy-id ivan@192.168.1.136

# Método 2 — manual (PowerShell)
$pubkey = Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub"
ssh ivan@192.168.1.136 "mkdir -p ~/.ssh && echo '$pubkey' >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

### Testar autenticação por chave

```powershell
ssh ivan@192.168.1.136
# Deve entrar sem pedir password
```

---

## 5. Hardening básico do SSH

Editar `/etc/ssh/sshd_config` no servidor:

```bash
sudo vim /etc/ssh/sshd_config
```

Ajustar ou adicionar:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

> **Atenção:** só desativar `PasswordAuthentication` depois de confirmar que o acesso por chave funciona.

```bash
sudo systemctl restart sshd
```

---

## 6. Alias SSH no Windows (opcional)

Criar/editar `C:\Users\IvanAlmeida\.ssh\config`:

```
Host homeserver-lan
    HostName 192.168.1.136
    User ivan
    IdentityFile ~/.ssh/id_ed25519

Host homeserver
    HostName 100.x.x.x
    User ivan
    IdentityFile ~/.ssh/id_ed25519
```

Depois basta usar: `ssh homeserver-lan` ou `ssh homeserver`

---

## Próximo passo

→ [04-ssd-externo.md](04-ssd-externo.md) — montar o SSD externo 1 TB
