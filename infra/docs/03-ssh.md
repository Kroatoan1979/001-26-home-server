# SSH — Configuração e Acesso Remoto

O SSH é o principal método de administração do servidor após a instalação. Objetivos: serviço ativo no arranque, acesso por chave (sem password), e configuração segura básica.

> **Estado a 2026-10-15:** só por chave (secção 7) e acessível só a partir da LAN e da tailnet (secção 8). As secções 2 e 5 ficam como histórico da instalação; onde houver diferença, prevalecem as secções 7 e 8.

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
ssh ivan@192.168.1.2
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
ssh-copy-id ivan@192.168.1.2

# Método 2 — manual (PowerShell)
$pubkey = Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub"
ssh ivan@192.168.1.2 "mkdir -p ~/.ssh && echo '$pubkey' >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

### Testar autenticação por chave

```powershell
ssh ivan@192.168.1.2
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
    HostName 192.168.1.2
    User ivan
    IdentityFile ~/.ssh/id_ed25519

Host homeserver
    HostName 100.x.x.x
    User ivan
    IdentityFile ~/.ssh/id_ed25519
```

Depois basta usar: `ssh homeserver-lan` ou `ssh homeserver`

---

## 7. Só por chave: password desligada (2026-10-15)

### Chaves autorizadas

Uma chave ED25519 por dispositivo. O comentário no fim de cada linha diz de onde é, para se poder revogar a chave certa se um dispositivo se perder:

| Comentário | Dispositivo | Como foi gerada |
|---|---|---|
| `ivan@acer` | Acer Predator (Windows) | `ssh-keygen` (era `ivan@DAT-WS-IA-01`) |
| `ivan@asus` | ZenBook Duo (Omarchy) | `ssh-keygen` (era `ivan@zenbook`) |
| `ivan@redmi-note-14` | Telemóvel (Termius) | Termius: Keychain → Generate Key (ED25519) → "Export to host" |
| `ivan@acer-kali` | Acer Predator, WSL Kali Linux | `ssh-keygen -t ed25519 -C "ivan@acer-kali"` (2026-10-18; acrescentada a partir do `asus`) |
| `ivan@tab-s8-de-ivan` | Tablet Galaxy Tab S8 (Termius) | Termius: Keychain → Generate Key (ED25519); a chave pública foi copiada da app e acrescentada a partir do `asus` (2026-10-21). O "Export to host" já não serve: precisa da password, que está desligada desde 2026-10-15 |

O "Export to host" do Termius junta a chave ao `authorized_keys` sem comentário; o nome dado na app fica só na app. Os comentários foram acertados à mão a 2026-10-15 (as chaves ficaram iguais, byte a byte; backup em `~/.ssh/authorized_keys.bak-2026-10-15`, apagado a 2026-10-18 depois de confirmar que as três chaves estavam no ficheiro atual).

No Termius, o host tem de usar a chave **e não ter a password guardada**; caso contrário, o teste não prova que a chave funciona.

**Troubleshooting (tablet, 2026-10-21):** o Termius dava "falha de autenticação" com a chave certa no `authorized_keys` e no host. O registo normal só mostrava `Disconnected from authenticating user ivan … [preauth]`. Com `LogLevel VERBOSE` temporário apareceu a causa: `Accepted key ED25519 SHA256:… found at …authorized_keys:5` seguido de `Postponed publickey` e de um `Normal Shutdown` do cliente. Ou seja, o servidor aceitou a chave pública, mas o Termius não chegou a assinar com a chave privada (chave privada estragada dentro da app). **Solução:** apagar a chave no Termius, gerar outra e trocar a linha no `authorized_keys`. Registo detalhado temporário:

```bash
# ligar (o 00- faz com que seja lido primeiro; no sshd ganha o primeiro valor)
echo "LogLevel VERBOSE" | sudo tee /etc/ssh/sshd_config.d/00-debug-temp.conf && sudo sshd -t && sudo systemctl reload sshd
journalctl -u sshd --since "-5 min"
# desligar sempre no fim
sudo rm /etc/ssh/sshd_config.d/00-debug-temp.conf && sudo sshd -t && sudo systemctl reload sshd
```

### Desligar a password

Só depois de as três chaves estarem testadas (o telemóvel por Wi-Fi e só com dados móveis) e **com uma sessão SSH aberta** como rede de segurança:

```bash
sudo tee /etc/ssh/sshd_config.d/10-sem-password.conf <<'EOF2'
# 2026-10-15: só autenticação por chave (acer, asus, redmi-note-14)
PasswordAuthentication no
KbdInteractiveAuthentication no
EOF2
sudo sshd -t && sudo systemctl reload sshd
```

- Drop-in em `sshd_config.d` em vez de editar o `sshd_config`: as atualizações do pacote não o tocam. No `sshd`, o **primeiro** valor lido ganha, e os ficheiros do diretório são lidos por ordem alfabética, daí o prefixo `10-`.
- `KbdInteractiveAuthentication no` também é preciso: sem ele, o PAM pode continuar a pedir a password por outro método.
- `sshd -t` valida a configuração e o `&&` só deixa recarregar se não houver erros.

### Validação

```bash
# No servidor: os dois a "no"
sudo sshd -T | grep -iE '^(passwordauthentication|kbdinteractiveauthentication)'

# Num cliente: a chave entra, a password forçada é recusada
ssh homeserver 'echo OK'
ssh -o PubkeyAuthentication=no -o BatchMode=yes homeserver true
# → Permission denied (publickey,gssapi-keyex,gssapi-with-mic)
```

Resultado a 2026-10-15: `sshd -T` com os dois a `no`; a partir do Asus, a chave entra e a password é recusada tanto por `homeserver` (tailnet) como por `homeserver.local` (LAN). O `gssapi-*` é Kerberos, ligado de origem no RHEL; sem um KDC não dá acesso a ninguém.

**Recuperação se se perderem todas as chaves:** consola física do servidor ou terminal do Cockpit (que autentica pelo PAM e continua a aceitar password, só a partir da LAN e da tailnet).

---

## 8. Firewall: SSH e Cockpit só a partir da LAN e da tailnet (2026-10-15)

O servidor tem um endereço **IPv6 global** em `enp2s0`. Em IPv6 não há NAT, por isso os serviços abertos na zona `public` ficavam alcançáveis da Internet se a firewall do router deixasse passar. O Cockpit era o caso grave: continua a aceitar password.

Os serviços `ssh` e `cockpit` saíram da zona e passaram a entrar só por *rich rules* com origem:

```bash
for svc in cockpit ssh; do
  sudo firewall-cmd --permanent --zone=public --add-rich-rule="rule family=\"ipv4\" source address=\"192.168.1.0/24\" service name=\"$svc\" accept"
  sudo firewall-cmd --permanent --zone=public --add-rich-rule="rule family=\"ipv4\" source address=\"100.64.0.0/10\" service name=\"$svc\" accept"
  sudo firewall-cmd --permanent --zone=public --add-rich-rule="rule family=\"ipv6\" source address=\"fd7a:115c:a1e0::/48\" service name=\"$svc\" accept"
  sudo firewall-cmd --permanent --zone=public --remove-service=$svc
done
sudo firewall-cmd --reload
sudo firewall-cmd --list-all --zone=public
```

| Origem | Rede |
|---|---|
| LAN IPv4 | `192.168.1.0/24` |
| Tailnet IPv4 (CGNAT da Tailscale) | `100.64.0.0/10` |
| Tailnet IPv6 (ULA da Tailscale) | `fd7a:115c:a1e0::/48` |

- O IPv6 global da LAN **não** entra, de propósito: o prefixo é do ISP e pode mudar, e uma regra com ele deixaria de valer sem aviso. O `homeserver.local` também resolve para esse endereço, mas o alvo `default` da zona responde com uma recusa imediata e os clientes voltam ao IPv4 sem se notar.
- A `tailscale0` não tem zona (`firewall-cmd --get-zone-of-interface=tailscale0` → `no zone`); aplica-se a `public`, a zona por omissão.
- Ficam abertos na zona só `dhcpv6-client` e `mdns`.
- Documento de estudo (cada opção explicada, alternativas, troubleshooting): [network/docs/03-firewalld-restricao-origem.md](../../network/docs/03-firewalld-restricao-origem.md).

**Validado a 2026-10-15 a partir do Asus:** SSH e Cockpit (`https://…:9090`) funcionam por `homeserver.local`, `192.168.1.2`, `homeserver`, IP `100.x` e IP `fd7a:…` da tailnet; pelo IPv6 global, o Cockpit dá *connection refused* e o SSH *Permission denied* na ligação. Teste a partir da Internet (dados móveis, sem Tailscale): abandonado pelo Ivan a 2026-10-18; revalidação a partir do Asus no mesmo dia com o mesmo resultado.

---

## Próximo passo

→ [04-storage-hdd-5tb.md](04-storage-hdd-5tb.md) — montar o HDD externo 5TB
