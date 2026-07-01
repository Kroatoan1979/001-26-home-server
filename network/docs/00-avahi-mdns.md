# Avahi (mDNS) — Resolução de Nomes na LAN

---

## O que é

**Avahi** é a implementação open-source (usada por distribuições Linux) do protocolo **mDNS** (*Multicast DNS*, RFC 6762) e **DNS-SD** (*DNS Service Discovery*, RFC 6763) — o mesmo protocolo que a Apple chama de **Bonjour**.

Em vez de depender de um servidor DNS central (como o router), cada dispositivo com Avahi ativo **anuncia o seu próprio nome** (`<hostname>.local`) para toda a rede local via multicast UDP, na porta **5353**. Qualquer outro dispositivo na mesma rede que também suporte mDNS (nativamente ou via cliente) consegue perguntar "quem é `homeserver.local`?" e receber resposta diretamente do próprio homeserver — sem passar por nenhum servidor intermediário.

---

## Aplicabilidade

- **Zero configuração central** — não depende do router nem de nenhum servidor DNS; funciona só com Avahi ativo na máquina anunciada e suporte mDNS no cliente
- **Só funciona dentro do mesmo segmento de rede local** — o multicast não atravessa routers/VLANs (isto é uma limitação, não um bug — mDNS foi desenhado para descoberta *local*)
- Windows 10/11, macOS e a maioria das distribuições Linux já trazem suporte a cliente mDNS nativo — normalmente não é preciso instalar nada do lado de quem pergunta o nome
- Standard amplamente usado (impressoras de rede, Chromecast, AirPlay, etc. usam mDNS/Bonjour para se anunciarem)

---

## Pertinência para este projeto

O homeserver tem **IP estático configurado diretamente no OS** (via `nmcli`), fora do pool DHCP do router. Isto significa que o router **nunca fica a saber** que o hostname `homeserver` existe — ele só regista nomes de dispositivos que pediram IP por DHCP. Resultado: `ssh ivan@homeserver` falhava com "Could not resolve hostname", obrigando a usar sempre o IP (`192.168.1.2`).

Avaliámos 3 alternativas para resolver isto e escolhemos o Avahi por ser a única sem desvantagens relevantes:

| Alternativa | Problema |
|---|---|
| Hosts file manual (`C:\Windows\System32\drivers\etc\hosts`) | Estático, manual por máquina, e **entra em conflito com o Tailscale MagicDNS** — se usar o mesmo nome (`homeserver`) para as duas coisas, o hosts file tem sempre prioridade e passa a resolver sempre para o IP LAN, mesmo quando estás fora de casa e precisavas do IP Tailscale |
| DNS local no router | O router não regista hosts com IP estático fora do DHCP — não resolve o problema de raiz |
| Tailscale MagicDNS | Só funciona quando o Tailscale está **ativo** na máquina cliente — sem Tailscale a correr, não há resolução |
| **Avahi/mDNS** (escolhido) | Nenhuma — funciona independentemente do Tailscale, não conflita com o MagicDNS porque usa um **nome diferente** (`homeserver.local` em vez de `homeserver`) |

**A estratégia final do projeto tem dois níveis de resolução, sem sobreposição:**

| Cenário | Nome a usar | Mecanismo |
|---|---|---|
| Fora de casa, ou em casa com Tailscale ligado | `ssh ivan@homeserver` | Tailscale MagicDNS |
| Em casa, sem Tailscale ativo | `ssh ivan@homeserver.local` | Avahi (mDNS) |

Como os dois usam nomes diferentes (`homeserver` vs `homeserver.local`), nunca competem entre si — cada mecanismo resolve o seu próprio namespace.

---

## Comandos utilizados (Rocky Linux 10 — homeserver)

### 1. Atualizar o sistema antes de instalar software novo

```bash
sudo dnf update -y
```

- `dnf` — gestor de pacotes do Rocky Linux (sucessor do `yum`)
- `update` — atualiza todos os pacotes instalados para a versão mais recente disponível nos repositórios configurados
- `-y` — responde "yes" automaticamente a qualquer prompt de confirmação (sem isto, o `dnf` pararia a pedir confirmação antes de aplicar as atualizações)

> Boa prática antes de instalar qualquer pacote novo: garante que a *metadata* dos repositórios está atualizada e evita instalar contra versões desatualizadas do sistema.

### 2. Instalar o Avahi

```bash
sudo dnf install -y avahi
```

- `install` — instala o pacote indicado e todas as suas dependências
- `avahi` — nome do pacote no repositório Rocky/RHEL; inclui o binário `avahi-daemon` e o serviço `systemd` correspondente
- `-y` — mesma função que acima, evita o prompt de confirmação "Is this ok [y/N]"

> Nota: este pacote só permite que o homeserver **anuncie** o seu próprio nome. Se no futuro quisermos que o *próprio Linux* resolva nomes `.local` de **outros** dispositivos (o inverso do que fizemos aqui), seria preciso instalar também `nss-mdns` e ajustar `/etc/nsswitch.conf` — não é necessário para o nosso caso, porque quem precisa de resolver é o Windows (DAT/ZenBook), que já tem suporte mDNS nativo.

### 3. Ativar o serviço como daemon persistente

```bash
sudo systemctl enable --now avahi-daemon
```

- `systemctl` — ferramenta de controlo de serviços do `systemd` (o gestor de processos/serviços do Rocky Linux)
- `enable` — cria o *symlink* necessário para que o serviço arranque automaticamente em todos os boots futuros
- `--now` — flag que combina `enable` + `start` num único comando: além de registar o arranque automático, arranca o serviço **imediatamente**, sem esperar por um reboot
- `avahi-daemon` — nome da unit (serviço) `systemd` instalada pelo pacote `avahi`

> Sem `--now`, o serviço ficaria só "agendado" para o próximo arranque — teríamos de correr `sudo systemctl start avahi-daemon` manualmente a seguir para ativar já.

### 4. Abrir a porta mDNS na firewall

```bash
sudo firewall-cmd --add-service=mdns --permanent
sudo firewall-cmd --reload
```

- `firewall-cmd` — ferramenta de gestão do `firewalld`, o serviço de firewall por defeito no Rocky Linux
- `--add-service=mdns` — adiciona a regra pré-definida `mdns` à zona ativa da firewall (o `firewalld` já vem com definições de serviços conhecidos; `mdns` = UDP porta `5353`, sem ser preciso indicar a porta manualmente)
- `--permanent` — grava a regra na configuração persistente em disco (sem esta flag, a regra aplicava-se já mas desaparecia no próximo reload/reboot)
- `--reload` — recarrega a configuração do `firewalld` a partir do disco, aplicando as regras `--permanent` à firewall ativa **sem** cortar ligações já estabelecidas (diferente de reiniciar o serviço)

> Truque: regras `--permanent` só ficam ativas depois de um `--reload` (ou reboot). Sem o `--reload`, a regra ficava gravada mas não em vigor.

### 5. Verificar que o serviço está ativo

```bash
systemctl status avahi-daemon --no-pager
```

- `status` — mostra o estado atual do serviço (ativo/inativo, PID, últimas linhas de log)
- `--no-pager` — desliga a paginação (`less`) do output, para sair tudo direto no terminal (útil para copiar/colar ou correr em scripts)

**Resultado obtido:** serviço `active (running)`, log confirma `Server startup complete. Host name is homeserver.local.` e registo do endereço `192.168.1.2` na interface `enp2s0`.

---

## Teste de validação (a partir do Windows — DAT-WS-IA-01)

```powershell
ping homeserver
ssh ivan@homeserver
```

O Windows tentou resolver `homeserver` por DNS tradicional, falhou, e caiu automaticamente para mDNS — o `ping` mostrou a resolução como `homeserver.local`. O `ssh` entrou diretamente (autenticação por chave ED25519 já configurada, sem pedir password).

> **Ainda por fazer:** repetir o mesmo teste a partir da ZenBook Duo UX482, para confirmar que o mDNS funciona em ambos os clientes Windows da rede.

---

## Comandos de referência (troubleshooting)

```bash
# Ver hostname mDNS anunciado
avahi-resolve-address 192.168.1.2

# Ver todos os serviços mDNS anunciados na rede
avahi-browse -a

# Confirmar que a firewall tem o serviço mdns ativo
sudo firewall-cmd --list-services
```
