# Tailscale — Adicionar Dispositivos à Tailnet

Cada dispositivo precisa de instalar o cliente Tailscale e autenticar com a mesma conta (a conta Google pessoal do Ivan) — até 2026-10-13 era `ivan.almeida@digitalltogether.com`.

---

## Windows (Acer — DAT-WS-IA-01, a renomear para `acer`)

1. Descarregar o cliente: [tailscale.com/download/windows](https://tailscale.com/download/windows)
2. Instalar e iniciar
3. Clicar em **Sign in** → autenticar com Google
4. O dispositivo aparece automaticamente no painel admin

Verificar no painel: `https://login.tailscale.com/admin/machines`

---

## Linux (Asus — Omarchy)

```bash
sudo pacman -S tailscale
sudo systemctl enable --now tailscaled
sudo tailscale up --hostname=asus
```

O `up` mostra um URL: abrir no browser e autenticar com a conta Google pessoal.

---

## Android (telemóvel Xiaomi e tablet Samsung)

- **Android:** [Google Play — Tailscale](https://play.google.com/store/apps/details?id=com.tailscale.ipn.android)

1. Instalar a app
2. Tocar em **Sign in** → autenticar com Google
3. Ativar a VPN quando solicitado

---

## Verificar a tailnet completa

No servidor:

```bash
sudo tailscale status
```

Exemplo de output esperado:

```
100.x.x.x  homeserver      linux    -
100.x.x.x  asus            linux    -
100.x.x.x  acer            windows  -
100.x.x.x  redmi-note-14   android  -
100.x.x.x  tab-s8-de-ivan  android  -
```

---

## Testar conectividade entre dispositivos

### Windows → Servidor (SSH)
```powershell
ssh ivan@homeserver          # tailnet (MagicDNS)
ssh ivan@homeserver.local    # LAN (mDNS)
```

### Telemóvel → Servidor (SSH)
- App recomendada: **Termius** ou **JuiceSSH**
- Host: `homeserver` (MagicDNS; funciona em dados móveis)
- Porta: 22
- Utilizador: ivan

### Telemóvel → FileCloud (após instalação do serviço)
- Browser (portal de utilizador FileCloud)
- URL: `http://100.x.x.x` (porta 80) ou `https://100.x.x.x` (porta 443)

---

## Migração de tailnet e convenção de nomes (2026-10-13)

### O que mudou

A tailnet passou da conta `digitalltogether.com` para a **conta Google pessoal do Ivan** (plano **Personal**, gratuito, uso não comercial). A conta antiga ficou bloqueada com *"Trial ended"*: deixou de ser possível aprovar ou adicionar dispositivos. A Tailscale trata um domínio próprio como uso empresarial, e o homelab é pessoal. Motivo completo e alternativas recusadas: `CLAUDE.md`, secção *Estado atual*.

**Não é possível mudar a conta de uma tailnet.** A migração é criar uma tailnet nova e voltar a autenticar cada nó. Consequências:

| Muda | Mantém-se |
|---|---|
| IPs `100.x` (novos) | Instalação do cliente (não se reinstala nada) |
| Sufixo MagicDNS (`tailXXXX.ts.net` novo) | Host keys SSH (são do servidor, não da tailnet) |
| ACL, definições de DNS, expiração das chaves (refazer na consola nova) | Nomes dos nós, se se usar `--hostname` |
| Config de `tailscale serve` (guarda o FQDN literal; fazer `tailscale serve reset`) | |

### Convenção de nomes

O mesmo nome nos dois caminhos; **o sufixo diz qual se usa**. É a estratégia de dois níveis do [00-avahi-mdns.md](00-avahi-mdns.md), retomada depois do período `homelan`/`hometail` (2026-09-28 a 2026-10-13).

| Nó | LAN (mDNS) | Tailnet (MagicDNS) |
|---|---|---|
| Servidor | `homeserver.local` | `homeserver` |
| Asus (Omarchy) | `asus.local` | `asus` |
| Acer (Windows) | `acer.local` (por fazer) | `acer` (por fazer) |
| Telemóvel / tablet | — (só clientes) | `redmi-note-14` / `tab-s8-de-ivan` |

Em casa, mesmo pelo nome da tailnet, o tráfego segue **direto pela LAN** (`tailscale ping` mostra `via 192.168.1.x:41641`). Por isso, o nome sem sufixo serve em qualquer lado; o `.local` é o recurso quando o Tailscale está em baixo.

**Recusado:** sufixos `.lan` / `.tail`. O Tailscale não aceita pontos no nome de um nó, e o `.lan` exigiria um DNS local (`dnsmasq` + *split DNS*), com o servidor como ponto único de falha.

### Comandos utilizados

```bash
sudo tailscale logout
```
Sai da tailnet atual e apaga a identidade do nó (chave de máquina). Sem isto, o `up` seguinte voltava a ligar à tailnet antiga.

```bash
sudo tailscale up --hostname=homeserver
```
- `up` — liga o nó; sem identidade, mostra um URL de autenticação (num servidor sem browser, abre-se o URL noutro PC).
- `--hostname=homeserver` — nome do nó na tailnet (e no MagicDNS). Sem esta flag, usa o hostname do sistema.

```bash
sudo hostnamectl set-hostname homeserver
```
Muda o hostname do sistema de forma persistente (`/etc/hostname`). É este o nome que o Avahi anuncia como `homeserver.local`. Sem `sudo`, falha por falta de permissão.

```bash
sudo tailscale set --hostname=homeserver
```
Muda só o nome do nó, sem voltar a autenticar (o `up` exigiria repetir as outras flags). Na consola, o nome atualiza sozinho, exceto se tiver sido editado à mão antes.

```bash
sudo sed -i 's/^#deny-interfaces=eth1/deny-interfaces=tailscale0/' /etc/avahi/avahi-daemon.conf
sudo systemctl restart avahi-daemon
```
- `-i` — edita o ficheiro no próprio sítio (sem a flag, só mostrava o resultado no ecrã).
- `deny-interfaces=tailscale0` — o Avahi deixa de anunciar nessa interface. **Sem isto, `asus.local` era anunciado com o IP `100.x`**, que não funciona para quem está só na LAN. Usou-se `deny` e não `allow` para não ter de listar Wi-Fi, Ethernet USB-C, etc.
- `restart` — o Avahi só lê o ficheiro ao arrancar.

**Expiração da chave:** desativada só no servidor (consola → Machines → `⋯` → *Disable key expiry*). Sem isto, ao fim de 180 dias o servidor saía da tailnet sem aviso. Nos portáteis e telemóveis fica ativa (a do Asus termina a 2027-04-11).

### Teste de validação (2026-10-13 e 2026-10-14)

| Teste | Resultado |
|---|---|
| `tailscale ping homeserver` (do Asus, em casa) | direto, `via 192.168.1.2:41641`, 1 ms |
| `ssh homeserver` / `ssh homeserver.local` / FQDN | OK (tailnet / LAN / tailnet) |
| Host key pela tailnet = pela LAN | igual (ED25519 `SHA256:qcGX2F…NWa4`) |
| `asus.local` visto do servidor | `192.168.1.110` (IP LAN, não `100.x`) |
| `homelan.local` / `omarchy.local` | já não resolvem (esperado) |
| Ping aos dois Android | OK (ligações diretas) |
| Termius no telemóvel (dados móveis) → `homeserver` | OK, entrada a partir de `100.108.10.121` (2026-10-14). Autenticação **por password**: o telemóvel não tem chave em `authorized_keys` |
| Surfshark + Tailscale ligados no Asus | OK: regra do Tailscale (5270, tabela 52) antes da do Surfshark (31317); `.ts.net` resolvido por `100.100.100.100`; internet sai pelo Surfshark; ligação ao servidor passa uns segundos pelo DERP depois de o Surfshark ligar e volta a ser direta |

**Falta testar:** o Acer depois da migração (`acer`, `acer.local`).

### Referência e troubleshooting

```bash
tailscale status                      # nós, IPs, estado
tailscale ping <nó>                   # mostra se o caminho é direto ou por DERP (relay)
tailscale netcheck                    # UDP, NAT, DERP mais próximo
resolvectl query homeserver           # que link resolveu o nome (tailscale0 = MagicDNS)
avahi-resolve -4 -n homeserver.local  # resolução mDNS
ip rule; ip route show table 52       # ordem das regras e rotas do Tailscale
```

- **SSH pede para aceitar uma fingerprint nova depois de mudar de nome:** comparar com a da LAN (`ssh-keyscan -t ed25519 192.168.1.2 | ssh-keygen -lf -`) antes de aceitar. No Asus, o `HostKeyAlias homeserver` do `~/.ssh/config` faz todos os nomes partilharem a mesma entrada.
- **Ligação por DERP em vez de direta, dentro de casa:** normal nos primeiros segundos ou depois de mudar a rede (ligar uma VPN, por exemplo). Se continuar, ver `tailscale netcheck` e a firewall (UDP 41641).
- **`<nome>.local` resolve para `100.x`:** falta o `deny-interfaces=tailscale0` no Avahi dessa máquina.
- **Android:** só aceita uma VPN ativa; ligar o Surfshark desliga o Tailscale.

---

## Painel de administração

Gerir dispositivos, revogar acessos, ver IPs e estado de ligação:

`https://login.tailscale.com/admin/machines`

---

## Atualizar IPs no repositório

Após todos os dispositivos estarem na tailnet, atualizar a tabela de IPs em:
- [../README.md](../README.md) — secção "IPs Tailscale"
- [../../README.md](../../README.md) — secção "Topologia de Rede"
