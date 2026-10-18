# Home Server + Tailscale — Contexto do Projeto

## 🔄 Estado atual (verificado a 2026-10-10; HDD atualizado a 2026-10-11; logind/journal e Wi-Fi a 2026-10-12; Tailscale a 2026-10-13; Acer a 2026-10-14; SSH e firewall a 2026-10-15; âmbito a 2026-10-18)

> Prevalece sobre o texto histórico abaixo, quando houver diferença.
>
> - **Convenção de nomes (decisão do Ivan, 2026-10-13; substitui `homelan`/`hometail` de 2026-09-28):** o mesmo nome nos dois caminhos e o sufixo diz qual se usa. **`<nome>.local` = LAN** (mDNS) e **`<nome>` sem sufixo = tailnet** (MagicDNS; FQDN completo omitido porque o repositório é público). Nomes: `homeserver` (servidor), `asus` (ZenBook, Omarchy) e `acer` (Acer Predator, Windows 11; era `DAT-WS-IA-01`. **Decidido a 2026-10-13, feito a 2026-10-14:** o nome do computador Windows também passou a `acer`, para o `acer.local` funcionar na LAN; o nome `DAT-WS-IA-01` deixou de ser usado na rede). Telemóvel e tablet mantêm os nomes. **Alternativa recusada:** sufixos `.lan`/`.tail`. O Tailscale não aceita pontos no nome do nó, e o `.lan` exigiria um DNS local (`dnsmasq` + split DNS) com o servidor como ponto único de falha. **Avahi:** `deny-interfaces=tailscale0` no servidor e no Asus, porque sem isto o nome `.local` era anunciado também com o IP `100.x`. **SSH no Asus:** o alias `homeserver` do `~/.ssh/config` já não força a LAN (`HostKeyAlias homeserver` comum a todos os nomes). Validado a 2026-10-13: `ssh homeserver` (tailnet), `ssh homeserver.local` (LAN) e `asus.local` → `192.168.1.110` visto do servidor.
> - **Tailnet migrada para a conta pessoal (decisão do Ivan, 2026-10-13).** A conta `digitalltogether.com` ficou bloqueada ("Trial ended": sem aprovar nem adicionar dispositivos). **Motivo:** a Tailscale trata um domínio próprio como uso empresarial; o homelab é pessoal, e o plano **Personal** (gratuito, só para uso não comercial) exige isso mesmo. Passou a usar a conta Google pessoal do Ivan (Gmail; endereço omitido porque o repositório é público). Alternativas recusadas: passar a tailnet da DAT para o Personal (ambíguo face aos termos, por ser um domínio da empresa), pagar o plano Starter (sem necessidade) e Headscale (obrigaria a expor o servidor; fica como possível estudo futuro). **Efeitos:** IPs `100.x` e sufixo MagicDNS novos; os nomes dos nós mantêm-se. Nós na tailnet nova: `homeserver`, `asus`, `tab-s8-de-ivan`, `redmi-note-14` (todos testados a 2026-10-13) e `acer` (2026-10-14: `tailscale ping acer` direto pela LAN, `acer.local` → `192.168.1.100`); `homeserver` com expiração da chave desativada. Nós da tailnet antiga apagados pelo Ivan a 2026-10-14: a migração está concluída.
> - **ZenBook Duo UX482:** desde outubro de 2026 corre **Omarchy (Arch Linux + Hyprland)** e já não Windows 11 Home. Tailscale instalado e na tailnet nova desde 2026-10-13. Hostname `asus` (era `omarchy`), nó `asus` (`100.99.98.113`).
> - **Avahi/mDNS:** ativo; `homeserver.local` resolve na LAN (até 2026-10-13 era `homelan.local`). O serviço `mdns` foi aberto no firewalld a 2026-09-28.
> - **Chave GPG do repositório Tailscale:** importada (`gpg-pubkey` presente). O `sudo dnf makecache` de 2026-10-12 passou sem erros nos 3 repos (BaseOS, AppStream, Tailscale); o erro de 2026-10-11 era só por correr sem `sudo`.
> - **Wi-Fi do servidor (`wlp3s0`): recurso abandonado (decisão do Ivan a 2026-10-12).** Substitui a decisão de 2026-08-07. Investigado a 2026-10-12: o Wi-Fi estava em baixo desde 2026-09-01 00:18 **sem ser intencional** (o NetworkManager esgotou as 4 tentativas de religação; a causa exata perdeu-se porque o journal era volátil). **Motivo:** o recurso só cobria falhas do cabo ou da porta Ethernet (usa o mesmo router), esteve morto 6 semanas sem ninguém dar por isso, e o acesso alternativo já está garantido pelo Tailscale e pelo mDNS. **Ficou reativável:** o perfil `Vodafone-C27A91 2.4G` mantém-se com a password guardada, só com `connection.autoconnect no`; o rádio continua ligado. Para reativar: `sudo nmcli connection modify "Vodafone-C27A91 2.4G" connection.autoconnect yes connection.autoconnect-retries 0 && sudo nmcli connection up "Vodafone-C27A91 2.4G"` (o `autoconnect-retries 0` evita repetir a falha de setembro) e criar a reserva DHCP para o MAC `80:a5:89:93:a6:fb`.
> - **Loop de suspensão do `systemd-logind` corrigido a 2026-10-12.** O `/etc/systemd/logind.conf` criado em agosto não tinha `[Login]`, por isso a tampa fechada pedia `suspend` sem parar e o `suspend.target` mascarado recusava. Durante 66 dias o `logind` gastou ~82% de um core, gerou ~1,8 GB/semana de `/var/log/messages` e deixou o journal com só ~6 min de histórico. Corrigido com o drop-in `/etc/systemd/logind.conf.d/10-homeserver.conf` (o ficheiro antigo ficou como `logind.conf.bak-2026-10-12`); a máscara dos alvos de suspensão mantém-se. **Journal agora persistente** (`/etc/systemd/journald.conf.d/10-persistent.conf`, `SystemMaxUse=1G`). Os `/var/log/messages-2026*` antigos (~7 GB de lixo do loop) ficam a cargo do `logrotate` (decisão do Ivan a 2026-10-12: não apagar à mão; o último sai por volta de 2026-11-08). Ver [infra/docs/09-logind-journald.md](infra/docs/09-logind-journald.md).
> - **HDD externo 5 TB** (`/dev/sdb1`, XFS): **montado em `/mnt/dados` pelo `/etc/fstab`** (UUID + `nofail`) desde 2026-10-11. Regra `udev` do `hdparm` recriada (APM 254 confirmado). Pastas: só `backups/` (o Ivan apagou `filecloud/` e `arquivo/`, que estavam vazias, a 2026-10-11; confirmado a 2026-10-12 que não fazem falta: a `filecloud/` deixou de ter uso com a saída do FileCloud do plano a 2026-10-18 e a `arquivo/` ficou redundante com `backups/Ivan_SSD/Arquivo`). SMART saudável a 2026-10-11 (0 setores realocados/pendentes, teste curto sem erros); teste longo em curso a 2026-10-12; vigiado pelo `smartd` desde 2026-10-12. Ver [infra/docs/04-storage-hdd-5tb.md](infra/docs/04-storage-hdd-5tb.md).
> - **Backup do arquivo pessoal (decisão do Ivan, 2026-10-11):** este disco guarda a cópia local do `Arquivo` do SSD externo de 1 TB (`Ivan_SSD`) em `/mnt/dados/backups/Ivan_SSD/`. Motivo: o SSD era a única cópia local das fotos; o OneDrive é a cópia fora de casa. Alternativas recusadas: ligar o SSD diretamente ao servidor (o RHEL não traz suporte NTFS) e um disco novo (este tem 4,5 TB livres e SMART saudável).
> - **TraderScope removida do servidor a 2026-10-12** (decisão do Ivan): serviço, `/opt/traderscope`, porta `8765/tcp` no firewalld, `tailscale serve`, operador do Tailscale (`ivan`) e deploy key. A app passa a ser desenvolvida só localmente (Acer + ZenBook) e volta ao servidor 24/7 quando estiver sólida. Ver `PLANO.md` (M7) no repositório `002-26-traderscope`. Nota: sem operador, os comandos `tailscale serve`/`set` voltam a precisar de `sudo`.
> - **Âmbito reduzido ao TraderScope (decisão do Ivan, 2026-10-18).** O único projeto planeado para o servidor passa a ser o **regresso do TraderScope em 24/7**, quando a app estiver sólida localmente (tarefa 13(e); antes disso, autenticação no `live_server`, que estava aberto à LAN). **Saem do plano, com tudo o que deles dependia:** FileCloud (e a decisão Docker vs Podman, que só existia por causa dele), sub-projeto Web Server + Cloudflare Tunnel (incluindo a passagem do DNS do `digitalltogether.xyz` para a Cloudflare), sub-projeto Guacamole, xrdp (tablet Android) e TraderShark. Nada disto chegou a ser instalado no servidor (verificado a 2026-10-18: `xrdp` e `docker-ce` ausentes, `/mnt/dados` só com `backups/`). Foram apagados `services/filecloud/`, `services/tradershark/`, `services/web-server/` e `infra/docs/07-acesso-grafico-xrdp.md` (ficam no histórico do git). O Podman continua instalado por vir com o RHEL; serve para a prática de contentores do RHCSA. **Por reavaliar:** a instalação "Server with GUI" foi escolhida por causa do TraderShark; o GNOME fica por agora (o X11 forwarding usa as apps gráficas do servidor).
> - **SSH: só por chave desde 2026-10-15** (decisão do Ivan). Três chaves ED25519 no `authorized_keys`, com o comentário a dizer o dispositivo: `ivan@acer`, `ivan@asus` e `ivan@redmi-note-14` (Termius, gerada na app e enviada com "Export to host"; testada por Wi-Fi e só com dados móveis). **2026-10-18:** quarta chave, `ivan@acer-kali` (WSL Kali Linux do acer, que substituiu o Ubuntu), para o X11 forwarding pelo WSLg. Os comentários antigos `ivan@DAT-WS-IA-01` e `ivan@zenbook` foram renomeados no mesmo dia (as chaves ficaram iguais). A password foi desligada com o drop-in `/etc/ssh/sshd_config.d/10-sem-password.conf` (`PasswordAuthentication no`, `KbdInteractiveAuthentication no`); o `sshd -T` confirma os dois a `no` e, a partir do Asus, um login forçado por password é recusado pela LAN e pela tailnet. Nos portáteis não há SSH de entrada (decisão de 2026-10-14). **Incoerência corrigida a 2026-10-15:** o OpenSSH Server do `acer` estava `Running`/`Automatic`; foi parado, passou a `Disabled` e a regra `OpenSSH-Server-In-TCP` da firewall do Windows foi desligada (desativado, não desinstalado: volta com `Set-Service sshd -StartupType Automatic; Start-Service sshd` e `Enable-NetFirewallRule`). A porta 22 do `acer` não responde, vista do Asus por `acer` e `acer.local`. No Asus (Omarchy), o `sshd` está inativo e desativado, sem nada à escuta na porta 22 (confirmado a 2026-10-15). Ver [infra/docs/03-ssh.md](infra/docs/03-ssh.md).
> - **Firewall: Cockpit e SSH só a partir da LAN e da tailnet (2026-10-15).** **Motivo:** o servidor tem um endereço **IPv6 global** em `enp2s0` (prefixo do ISP, omitido porque o repositório é público). Em IPv6 não há NAT, e os serviços `cockpit` e `ssh` estavam abertos a qualquer origem na zona `public`, o que deixava a proteção só a cargo da firewall do router. O Cockpit era o único acesso que ainda aceitava password. **Agora:** os serviços `cockpit` e `ssh` saíram da zona `public` e entram só por *rich rules* de origem: `192.168.1.0/24` (LAN), `100.64.0.0/10` (tailnet IPv4) e `fd7a:115c:a1e0::/48` (tailnet IPv6). O IPv6 global da LAN fica recusado de propósito, porque o prefixo do ISP pode mudar; o `homeserver.local` também resolve para esse endereço, mas a recusa é imediata e os clientes voltam ao IPv4. **Validado a partir do Asus:** Cockpit e SSH funcionam por `homeserver.local`, `192.168.1.2`, `homeserver` e pelos IPs da tailnet (v4 e v6); pelo IPv6 global são recusados. A `tailscale0` não tem zona atribuída (aplica-se a `public`, a zona por omissão). Por testar (opcional): se o router bloqueia IPv6 vindo da Internet (telemóvel só com dados móveis, sem Tailscale).
> - **Repositório público** (decisão do Ivan a 2026-10-08).
> - **Memória do Claude:** sincronizada entre o Omarchy e o Windows pelo repositório privado `claude-memoria` (concluído a 2026-10-10).

## ⚠️ Migração em curso: Rocky Linux → RHEL Developer

> **Nota de 2026-10-18:** o texto desta secção é histórico. O TraderShark, o FileCloud e o xrdp nela referidos saíram do plano (ver "Âmbito reduzido ao TraderScope" no Estado atual).

> **Decisão tomada em 2026-08-05.** O servidor vai ser **recriado do zero** com **RHEL (Red Hat Developer Subscription for Individuals)**, substituindo o Rocky Linux 10 atualmente instalado. Motivo principal: usar o SO oficial da certificação **RHCSA** para estudo/prática, e não um rebuild binário. Aproveita-se também para instalar o **TraderShark** (app nativa de trading desportivo, sensível a latência).
>
> **Mudança adicional (mesma data):** o **Jellyfin e o Navidrome saem do projeto** — o serviço principal do servidor passa a ser o **FileCloud** (partilha/sincronização de ficheiros), que consolida aqui o que estava planeado como projeto separado ("FileCloud na AWS"). A estrutura de pastas do HDD 5TB é reorganizada de acordo (deixa de ser filmes/séries/música).
>
> **Reinstalação em curso — rede e SSH operacionais, X11 em configuração (2026-08-09).** ISO DVD x86_64 verificada por checksum, pen USB criada com Rufus (modo DD Image), instalação "Server with GUI" concluída no ASUS X555L, subscrição registada via `subscription-manager register` (sem `--auto-attach` — ver [infra/docs/05-subscription-manager.md](infra/docs/05-subscription-manager.md)), `dnf update -y` aplicado (versão atual: RHEL 10.2 "Coughlan"). SSH ativo e acessível, modo headless configurado (passos 1–2), Tailscale reinstalado e autenticado como novo node `homeserver` (IP `100.98.31.9`, MagicDNS testado com sucesso). Servidor já colocado na rack definitiva, ligado por Ethernet + Wi-Fi (Wi-Fi mantido propositadamente como via de acesso permanente de recurso, decisão do Ivan). **Os dois bugs de rede/SSH foram resolvidos em 2026-08-09:** IP estático `192.168.1.2` a aplicar-se corretamente em `enp2s0` (o perfil `nmcli` ativo estava trocado — corrigido), autenticação SSH por chave a funcionar de forma consistente (causa real: `~/.ssh` nem existia no servidor — recriado com permissões e contexto SELinux corretos). Confirmado também que `subscription-manager status` mostrar `Registered` (não `Current`) é o comportamento esperado em Simple Content Access, não um problema. **TraderShark continua pausado** (decisão do Ivan em 2026-08-07 — problemas na própria aplicação, projeto secundário), mas **o X11 forwarding foi retomado por decisão do Ivan em 2026-08-09**, independente do TraderShark — em configuração, ver checklist. xrdp mantém-se pausado (caso de uso futuro independente, tablet Android). **Decisões que não dependem do SO** (armazenamento HDD 5TB, conta Tailscale `digitalltogether.com`, IPs/DHCP reservations, topologia de rede) **mantêm-se inalteradas**.
>
> **Vai mudar:**
> - SO: Rocky Linux 10 → RHEL 10 Developer Subscription, instalação "Server with GUI" (GNOME) — deixa de ser Minimal, porque o TraderShark precisa de sessão gráfica local
> - Identidade Tailscale do servidor (novo node — remover o node `homeserver` antigo no admin console após a reinstalação)
> - Chave de host SSH (nova — clientes vão pedir para aceitar a fingerprint nova)
> - Modo headless deixa de ser "sempre" — passa a "headless por padrão, exceto durante o uso do TraderShark" (sessão local, tampa aberta)
>
> **Não muda:**
> - Hardware (ASUS X555L)
> - HDD externo 5TB WD My Passport, estrutura `/mnt/dados`, `hdparm`
> - Conta Tailscale `digitalltogether.com`, plano Free *(mudou a 2026-10-13 — ver Estado atual)*
> - IPs LAN, reservas DHCP, topologia de rede
> - Política de SSH por chave (sem password)

---

## Objetivo

Transformar um portátil antigo (ASUS X555L) num home server headless a correr **RHEL (Red Hat Developer Subscription for Individuals)**, acessível remotamente via Tailscale. Além do papel de servidor doméstico, o projeto serve estes objetivos (âmbito revisto a 2026-10-18):

- **Estudo e prática para a certificação RHCSA** (Red Hat Certified System Administrator) — maioritariamente via CLI (utilizadores/grupos, permissões, LVM/storage, systemd, firewalld, SELinux, containers com Podman)
- **TraderScope em 24/7** — único serviço planeado; volta ao servidor quando estiver sólido localmente (repositório `002-26-traderscope`, `PLANO.md` M7)
- **Backup local do arquivo pessoal** no HDD externo 5TB (`/mnt/dados/backups`)
- Acesso gráfico pontual por X11 forwarding (Omarchy e WSL do acer)

---

## Hardware do Servidor

| Componente | Especificação |
|---|---|
| Modelo | ASUS X555L |
| CPU | Intel Core i7 (4ª ou 5ª geração) |
| RAM | 8 GB DDR3 |
| Disco interno | SSD (SO + serviços) |
| Disco externo | HDD 5TB WD My Passport (backups — uso 24/7 aceitável para home server de uso familiar) |
| Rede | Ethernet ao router doméstico |
| Avaria | Placa de som — irrelevante para servidor |

---

## Stack Técnico

| Camada | Tecnologia |
|---|---|
| SO | **RHEL 10** (Red Hat Developer Subscription for Individuals), instalação "Server with GUI" (GNOME) — versão confirmada em 2026-08-05 como a base atual do exame RHCSA |
| Subscrição | `subscription-manager register` — conta gratuita developers.redhat.com, uso de desenvolvimento individual (não produção em equipa) |
| VPN mesh | Tailscale — conta Google pessoal do Ivan (Gmail), plano **Personal** gratuito — migrada a 2026-10-13 a partir da conta `digitalltogether.com` (ver Estado atual). Antes: conta `digitalltogether.com`, plano Free (decisão de 2026-08-03, revertida a 2026-10-13) |
| Contentores | Podman (vem com o RHEL; só prática RHCSA, nenhum serviço em contentor) |
| Serviço planeado | TraderScope (24/7, quando estiver sólido localmente) |
| Acesso remoto Linux (terminal) | SSH |
| Acesso remoto Linux (gráfico) | X11 forwarding (`ssh -Y` a partir do Omarchy; `ssh -X` a partir do WSL do acer, via WSLg) |
| Gestão de pacotes | `dnf` (não `apt`) |

---

## Topologia de Rede

### Dispositivos na Tailnet

| Dispositivo | OS | Protocolo de acesso |
|---|---|---|
| ASUS X555L (servidor) | RHEL (Developer Subscription) | SSH, X11 forwarding |
| Acer Predator — `acer` (workstation Ivan; era DAT-WS-IA-01) | Windows 11 Pro | RDP / SSH |
| ZenBook Duo UX482 | Omarchy (Arch Linux + Hyprland), desde 10/2026 (antes: Windows 11 Home) | SSH |
| Telemóvel (pessoal, Xiaomi Redmi Note 14) | Android | App Tailscale |
| Tablet (pessoal, Samsung Galaxy Tab S8) | Android | App Tailscale |

> PC Kyndryl e Telemóvel Kyndryl excluídos do projeto — equipamentos corporativos, risco de violação de políticas de segurança da Kyndryl.

### IPs e Hostnames

| Dispositivo | IP LAN | IP Tailscale |
|---|---|---|
| Servidor — homeserver (ASUS X555L) | `192.168.1.2` (estático, Ethernet `enp2s0`, reconfirmado a 2026-10-10) · Wi-Fi `wlp3s0` em baixo desde 2026-09-01, recurso abandonado a 2026-10-12, reativável (antes `192.168.1.146`, dinâmico) | `100.112.247.104` (tailnet pessoal, desde 2026-10-13; antes `100.98.31.9`) |
| acer (Acer Predator) — Wi-Fi | `192.168.1.100` | `100.82.47.47` (tailnet pessoal, desde 2026-10-14; antes `100.124.224.123`) |
| acer (Acer Predator) — Ethernet | `192.168.1.101` | `100.82.47.47` |
| ZenBook UX482 — Wi-Fi | `192.168.1.110` | `100.99.98.113` |
| ZenBook UX482 — Ethernet USB-C | `192.168.1.111` | `100.99.98.113` |
| Telemóvel (pessoal, Redmi Note 14) | — | `100.108.10.121` |
| Tablet (pessoal, Galaxy Tab S8) | — | `100.119.33.58` |
| Smart TV (Xiaomi) | `192.168.1.120` | — (sem Tailscale) |

> Pool DHCP do router (Vodafone): `192.168.1.64–253`. Reservas DHCP dentro deste range. Homeserver com IP estático via `nmcli` fora do pool. **Decisão do Ivan (reconfirmada a 2026-10-12):** o servidor fica com IP estático no SO, de propósito; é a exceção à regra de reservas DHCP, que continua a valer para os outros dispositivos. Vantagem técnica (nota do Kody, não é motivo dado pelo Ivan): o IP não depende da configuração do router.

### Acesso à tailnet

- Conta Tailscale: conta Google pessoal do Ivan (Gmail), plano Personal — desde 2026-10-13 (antes `ivan.almeida@digitalltogether.com`)
- Painel admin: `https://login.tailscale.com/admin/machines`
- **MagicDNS ativo** — resolução por hostname dentro da tailnet, funciona em qualquer rede (Wi-Fi, dados móveis), sem precisar do IP Tailscale. Confirmado com `ssh ivan@homeserver` a partir do telemóvel via dados móveis (nome da altura; voltou a ser `ssh ivan@homeserver` a 2026-10-13).

### Nomes DNS dos nodes (MagicDNS)

| Dispositivo | Hostname MagicDNS |
|---|---|
| Servidor (ASUS X555L) | `homeserver` (foi `hometail` de 2026-09-28 a 2026-10-13) |
| Acer Predator | `acer` (era `dat-ws-ia-01` até 2026-10-14) |
| ZenBook UX482 | `asus` (era `ux482` até 2026-10-13) |
| Telemóvel (Redmi Note 14) | `redmi-note-14` |
| Tablet (Galaxy Tab S8) | `tab-s8-de-ivan` |

> Nomes revistos a 2026-10-13 (ver "Convenção de nomes" no Estado atual). O `dat-ws-ia-01` passou a `acer` a 2026-10-14.

---

## Estrutura de Armazenamento

```
SSD interno
├── RHEL (Developer Subscription)
├── Serviços (Tailscale; TraderScope quando voltar)
└── Configurações e logs

HDD externo 5TB WD My Passport (ponto de montagem: /mnt/dados)
└── /mnt/dados/backups
    └── Ivan_SSD/Arquivo   (cópia do arquivo pessoal do SSD externo, 2026-10-11 — ver infra/docs/08-backup-arquivo-ssd.md)
```

> **Nota de armazenamento:** SSD M.2 2280 1TB (enclosure USB) usado como disco portátil de alta capacidade para transferência de ficheiros entre máquinas — não integrado no servidor. HDD 5TB WD My Passport escolhido pelo espaço; spin-down pode ser desativado via `hdparm` se causar delays de acesso.
>
> ⚠️ **Reestruturação de 2026-08-05:** as pastas `filmes`/`series`/`musica` (pensadas para Jellyfin/Navidrome) saem do plano. Passam a existir fisicamente no disco de uma reinstalação anterior — a limpar/renomear ao recriar a estrutura em RHEL, ver [infra/docs/04-storage-hdd-5tb.md](infra/docs/04-storage-hdd-5tb.md).

---

## Comandos Frequentes

```bash
# Ligar ao servidor via SSH (Tailscale / MagicDNS)
ssh homeserver
# ou: ssh ivan@100.112.247.104

# Ligar ao servidor via SSH (LAN, mDNS — o .local é obrigatório)
ssh homeserver.local

# Ligar ao servidor via SSH (LAN, Ethernet, IP estático confirmado)
ssh ivan@192.168.1.2

# Verificar estado do Tailscale
sudo tailscale status

# Verificar temperatura (headless)
watch -n 2 sensors

# Atualizar sistema
sudo dnf update -y

# Verificar estado da subscrição RHEL
sudo subscription-manager status

# Ligação gráfica X11 forwarding (a partir do WSL do acer, via WSLg; apps GTK4 precisam de -Y)
ssh -X homeserver xterm
```

---

## Estado do Projeto (Checklist)

> **Próximas tarefas prioritárias (por ordem) — continuar por aqui:**
> - [x] **Importar a chave GPG do repositório Tailscale de forma persistente** (confirmado a 2026-10-10): `sudo rpm --import https://pkgs.tailscale.com/stable/rhel/10/repo.gpg` — sem isto, qualquer `dnf` que toque no repo `tailscale-stable` fica preso a pedir confirmação `[y/N]`
> - [x] **Alternativa ao `xorg-x11-apps` para testar o X11 forwarding** — 2026-10-14: não é preciso instalar nada; a `gnome-calculator` e o `gnome-text-editor` já estão no servidor e servem de app de teste
> - [x] **Testar o X11 forwarding** — 2026-10-14, a partir do Omarchy (`asus`): `ssh -Y homeserver gnome-calculator` abre a janela no ecrã local (confirmado pelo Ivan). O `-X` (*untrusted*) **não funciona** com apps GTK4 ("XInput2 support not found"), por isso usa-se `-Y`, só para hosts de confiança. Do Acer: **testado a 2026-10-18 pelo WSL Kali + WSLg** (`ssh -X homeserver xterm`), sem VcXsrv
> - [x] Decidir se avança já a desativação de `PasswordAuthentication` no SSH do homeserver — **decidido e feito a 2026-10-15** (ver "Estado atual")
> - [x] Atualização de memória geral Claude no UX482: concluída a 2026-10-10. Os protocolos DAT estão no `CLAUDE.md` global e a memória sincroniza entre os PCs pelo repositório privado `claude-memoria`.

### Migração Rocky Linux → RHEL (reinstalação completa do servidor)

> Os pontos abaixo estavam concluídos em Rocky Linux e foram repostos a pendente porque serão refeitos do zero em RHEL. Ver [infra/docs/01-instalacao-rhel.md](infra/docs/01-instalacao-rhel.md).

- [x] Criar conta gratuita em developers.redhat.com e obter a subscrição Developer
- [x] Descarregar a ISO do RHEL — **DVD iso x86_64 (7.88 GB)**, não a Boot iso — ver [infra/docs/01-instalacao-rhel.md](infra/docs/01-instalacao-rhel.md)
- [x] Criar pen USB bootável com Rufus, modo DD Image
- [x] Instalar RHEL no ASUS X555L, modo "Server with GUI" (GNOME)
- [x] `subscription-manager register` — **sem `--auto-attach`** (opção removida no RHEL 10, usa Simple Content Access) — ver [infra/docs/05-subscription-manager.md](infra/docs/05-subscription-manager.md)
- [x] `dnf update -y` — confirmado indiretamente (sistema já em RHEL 10.2 "Coughlan", kernel atualizado)
- [x] Definir hostname `homeserver` (foi `homelan`/`hometail` de 2026-09-28 a 2026-10-13; voltou a `homeserver` nos dois)
- [x] Configurar IP estático no homeserver via `nmcli` (`192.168.1.2/24`) — **resolvido em 2026-08-09**: existiam dois perfis para `enp2s0` (`Wired connection 1`, automático/DHCP, e `enp2s0`, manual/estático, já configurado corretamente mas com `autoconnect: no`). Corrigido com `nmcli connection modify "Wired connection 1" connection.autoconnect no`, `nmcli connection modify enp2s0 connection.autoconnect yes` e `nmcli connection up enp2s0`. Confirmado via `ip -4 a show enp2s0` e `ip route show default`
- [x] Criar utilizador `ivan`, não-root, com sudo
- [x] Configurar modo headless (tampa fechada): passos 1 e 2 concluídos. ⚠️ **O passo 1 estava partido** (`logind.conf` sem `[Login]`); refeito a 2026-10-12 com um drop-in, e `HandleLidSwitch = ignore` confirmado com `busctl`. Ver [infra/docs/02-modo-headless.md](infra/docs/02-modo-headless.md) e [infra/docs/09-logind-journald.md](infra/docs/09-logind-journald.md)
- [x] Instalar e configurar SSH — `sshd` ativo, porta aberta na firewall, ligação testada — ver [infra/docs/03-ssh.md](infra/docs/03-ssh.md)
- [x] Configurar autenticação por chave SSH (ED25519) — **resolvido em 2026-08-09**: a causa real não era inconsistência, era ausência total — `~/.ssh` nem existia no servidor. Criado `~/.ssh` (`chmod 700`), `authorized_keys` (`chmod 600`) com a chave pública `id_ed25519` da DAT-WS-IA-01, e `restorecon -Rv ~/.ssh` para repor o contexto SELinux (`ssh_home_t`). Confirmado com ligação `ssh -o BatchMode=yes` bem-sucedida
- [x] Migrar o `dat-ws-ia-01` para a tailnet pessoal, renomear o computador Windows e o nó para `acer`, confirmar `acer.local` na LAN e atualizar o IP Tailscale nesta documentação — **2026-10-14** (`Rename-Computer` num PowerShell elevado; o primeiro login no Tailscale não ficou na conta pessoal e teve de ser repetido)
- [x] Apagar os nós da tailnet antiga (`digitalltogether.com`) — 2026-10-14
- [x] Desativar autenticação por password SSH (`PasswordAuthentication no`) — **feito a 2026-10-15**: primeiro juntou-se a chave do telemóvel (Termius, `ivan@redmi-note-14`) e testou-se por Wi-Fi e dados móveis; depois o drop-in `10-sem-password.conf`, com uma sessão SSH aberta como rede de segurança. Ver [infra/docs/03-ssh.md](infra/docs/03-ssh.md)
- [x] Limitar Cockpit e SSH à LAN e à tailnet na `firewalld` (o servidor tem IPv6 global, sem NAT) — **feito a 2026-10-15**, ver "Estado atual"
- [x] Configurar Avahi (mDNS) no homeserver: ativo desde 2026-09-28, resolve `homeserver.local` (era `homelan.local` até 2026-10-13; `deny-interfaces=tailscale0` desde essa data) (firewalld `mdns` aberto). Ver [network/docs/00-avahi-mdns.md](network/docs/00-avahi-mdns.md)
- [x] Remover o node antigo "homeserver" (Rocky Linux) no admin console do Tailscale
- [x] Instalar Tailscale no RHEL e autenticar (`tailscale up --hostname=homeserver`) — novo IP `100.98.31.9` (hoje `100.112.247.104`, tailnet pessoal desde 2026-10-13). Ver [network/docs/01-tailscale-instalacao.md](network/docs/01-tailscale-instalacao.md)
- [x] Testar acesso remoto via Tailscale/MagicDNS — confirmado a partir do DAT-WS-IA-01 (`ssh ivan@homeserver`); **restantes dispositivos testados a 2026-10-15**: `asus` e `acer` (LAN e tailnet) e telemóvel (Termius, Wi-Fi e só dados móveis), todos por chave
- [x] X11 forwarding funcional a partir do Omarchy — **2026-10-14** (`ssh -Y`; o `-X` falha com GTK4). Do Acer: **OK a 2026-10-18 pelo WSL Kali + WSLg** (`ssh -X homeserver xterm`; o VcXsrv deixou de ser preciso). Ver [infra/docs/06-acesso-grafico-x11.md](infra/docs/06-acesso-grafico-x11.md)
- [x] Ligar o HDD externo 5TB e montar via `fstab` em `/mnt/dados` — **feito a 2026-10-11** (UUID inalterado, sem reformatar). Ver [infra/docs/04-storage-hdd-5tb.md](infra/docs/04-storage-hdd-5tb.md), secção "Reposição em RHEL"
- [x] Reestruturar as pastas do HDD — **feito a 2026-10-11**: `filmes`/`series`/`musica` removidas (estavam vazias), `/mnt/dados/filecloud` criada; `/backups` e `/arquivo` mantidas. **Atualização 2026-10-12:** `filecloud/` e `arquivo/` apagadas pelo Ivan (vazias); fica só `backups/`
- [x] Regra `udev` do `hdparm` (`99-hdd-power.rules`) — **recriada a 2026-10-11** (não existia após a reinstalação), `APM_level = 254` confirmado
- [x] Confirmar com `sudo dnf makecache` se o repo `tailscale-stable` ainda falha a verificação GPG: **passou a 2026-10-12** (os 3 repos sem erros). O erro de 2026-10-11 era só por correr sem `sudo`
- [x] Testar reboot do servidor: **feito a 2026-10-12.** O HDD montou sozinho em `/mnt/dados`, a regra `udev` aplicou `APM_level = 254`, o `HandleLidSwitch` ficou `ignore` (tampa fechada, 0 mensagens de suspensão, logind a 0% de CPU), o `journalctl --list-boots` mostra 2 boots (journal persistente), o `traderscope.service` arrancou sozinho, o Wi-Fi ficou desligado (`autoconnect no`) e não há units `failed`
- [x] Decidir o recurso Wi-Fi (`wlp3s0`): **abandonado, mas reativável** (2026-10-12, `autoconnect no` aplicado e confirmado). Ver "Estado atual"
- [ ] Teste SMART longo ao HDD: **em curso**, arrancado a 2026-10-12 às 14:20, fim previsto às 17:46. Ver o resultado com `journalctl -u smartd | grep -i self-test` (sem sudo) ou `sudo smartctl -l selftest /dev/sdb`
- [ ] Por volta de 2026-11-08: confirmar com `ls -lh /var/log/messages*` que o `logrotate` já removeu os `messages-2026*` antigos (~7 GB do loop do `logind`; decisão do Ivan: não apagar à mão). Ver [infra/docs/09-logind-journald.md](infra/docs/09-logind-journald.md)
- [ ] Por volta de 2026-10-19: ver o crescimento real com `journalctl --disk-usage` e `ls -l /var/log/messages*` (deve ser KB/MB por dia, não GB)
- [x] HDD vigiado pelo `smartd` (2026-10-12): linha própria no `smartd.conf` (o `DEVICESCAN` não apanhava o disco USB) e teste longo automático no dia 1 de cada mês às 03h. Ver [infra/docs/04-storage-hdd-5tb.md](infra/docs/04-storage-hdd-5tb.md)
- [ ] Redeploy do TraderScope em 24/7, só quando estiver sólido localmente (tarefa 13(e); primeiro, autenticação no `live_server`). Ver `PLANO.md` (M7) no repositório `002-26-traderscope`
- [ ] Praticar RHCSA: utilizadores/grupos, permissões, LVM/storage, systemd, firewalld, SELinux, containers (Podman) — maioritariamente via CLI
- [x] Checklist de validação final: SSH, Tailscale e X11 a partir de todos os dispositivos. **Feita a 2026-10-15:** SSH só por chave a partir de `asus`, `acer` e telemóvel (LAN e tailnet; password e IPv6 global recusados); `tailscale ping` direto pela LAN; X11 com `ssh -Y` a partir do `asus` e com `ssh -X` a partir do `acer` (WSL Kali + WSLg, 2026-10-18). xrdp, FileCloud e TraderShark saíram do plano a 2026-10-18; o TraderScope terá validação própria quando voltar

### Independente do SO (não afetado pela reinstalação; tem itens concluídos e pendentes)

- [x] Configurar DHCP Reservation no router para DAT-WS-IA-01, ZenBook e Smart TV
- [x] Recolher MAC Ethernet USB-C do ZenBook (`00-E0-4C-4D-25-20`) e criar reserva
- [x] Actualizar toda a documentação para referenciar nodes com hostname + IP
- [x] Reiniciar dispositivos para aplicar novos IPs reservados e verificar com `ipconfig` — DAT-WS-IA-01 Wi-Fi `.100` ✅ · ZenBook Wi-Fi `.110` ✅ · ZenBook Ethernet USB-C `.111` ✅ · DAT-WS-IA-01 Ethernet `.101` ✅ (porta nativa Killer E2600)
- [x] Ligar HDD externo 5TB ao homeserver e montar via SSH (formatado em XFS, configurado `/etc/fstab` com UUID + `nofail`) — feito em Rocky Linux, disco e formatação mantêm-se após a reinstalação
- [x] Configurar estrutura de pastas no HDD (`/mnt/dados/filmes`, `/mnt/dados/series`, `/mnt/dados/musica`, `/mnt/dados/backups`, `/mnt/dados/arquivo`) — ⚠️ estrutura superada em 2026-08-05: `filmes`/`series`/`musica` saem do plano (Jellyfin/Navidrome removidos), ver tarefa de reestruturação para `/mnt/dados/filecloud` na secção de migração acima
- [x] Desativar spin-down do HDD externo via `hdparm` para evitar delays de acesso — `hdparm -B 254 -S 0` aplicado e tornado persistente via regra `udev` (`99-hdd-power.rules`), validado em [infra/docs/04-storage-hdd-5tb.md](infra/docs/04-storage-hdd-5tb.md) — regra `udev` precisa de ser recriada após reinstalação do SO (fica em `/etc`, não sobrevive)
- [x] Instalar e ativar OpenSSH Server na ZenBook Duo UX482 (`sshd` a correr, arranque automático configurado, firewall `OpenSSH-Server-In-TCP` ativo, perfis de rede Wi-Fi e Tailscale confirmados como `Private`)
- [x] Instalar e ativar OpenSSH Server no DAT-WS-IA-01 (`sshd` a correr, arranque automático configurado, firewall `OpenSSH-Server-In-TCP` ativo — só se aplica ao perfil `Private`; perfil de rede Wi-Fi corrigido de `Public` para `Private` para paridade com a ZenBook e para o SSH ficar de facto acessível por essa rede)
- [~] ~~Configurar autenticação por chave SSH na ZenBook (decisão tomada: chaves sempre, sem password — ver política de autenticação SSH abaixo). Conta é Administrador, por isso o OpenSSH Server do Windows exige a chave pública em `C:\ProgramData\ssh\administrators_authorized_keys`, não no `authorized_keys` normal do utilizador~~ **Obsoleto (2026-10-10): a ZenBook deixou de ter Windows (agora Omarchy).**
- [~] ~~Testar `ssh ivana@localhost` na ZenBook com a chave configurada~~ **Obsoleto (2026-10-10): a ZenBook deixou de ter Windows (agora Omarchy).**
- [~] ~~Testar SSH à ZenBook a partir de outro dispositivo (LAN e/ou Tailscale) — depende do ponto anterior~~ **Obsoleto (2026-10-10): a ZenBook deixou de ter Windows (agora Omarchy).**
- [x] Replicar a configuração de `administrators_authorized_keys` no DAT-WS-IA-01 (chave própria ED25519 adicionada, ACLs corrigidas — `SYSTEM:F` + `BUILTIN\Administrators:F`) — **mas autenticação por chave ainda não funciona, ver bloqueio abaixo**
- [ ] ⚠️ **Bloqueio descoberto:** a conta Windows da DAT-WS-IA-01 é Azure AD cloud-only (`azuread\ivanalmeida`, SID `S-1-12-1-...`), e o `sshd` do Windows não consegue resolver esta identidade para login (`LookupAccountName`/`LogonUser` falham) — testados username simples, `AzureAD\user` e o SID direto, todos rejeitados como `Invalid user` no log do Event Viewer (`OpenSSH/Operational`), mesmo com a chave pública corretamente instalada. É uma limitação conhecida da plataforma (contas Azure AD Joined "cloud-only", sem AD on-premises por trás), não um erro de configuração. Contorno recomendado: criar conta local dedicada ao SSH. **Decisão pendente** até se confirmar se a ZenBook tem o mesmo problema
- [~] ~~Verificar se a ZenBook tem o mesmo problema de conta Azure AD cloud-only (`whoami /user` — SID a começar por `S-1-12-1-` confirma; `dsregcmd /status` — `AzureAdJoined : YES` confirma) antes de decidir a estratégia de contorno para autenticação por chave SSH nas duas máquinas Windows~~ **Obsoleto (2026-10-10): a ZenBook deixou de ter Windows (agora Omarchy).**
- [ ] Rever todos os nodes da tailnet quanto à autenticação SSH por chave. **Âmbito decidido pelo Ivan a 2026-10-14:** o que conta é o homeserver aceitar só chave; **não se abre SSH de entrada nos portáteis** (`asus` tem o `sshd` desligado; no `acer` o servidor OpenSSH chegou a ser configurado em 2026-08 e falta confirmar que está desligado), porque ninguém precisa dele e seria só superfície de ataque. Estado: `asus` e `acer` entram no homeserver só com chave (tailnet e LAN) ✅; falta o telemóvel (Termius ainda entra por password; resolve-se com a desativação de `PasswordAuthentication` acima)
- [ ] Avaliar node a node se a autenticação por password deve ser desativada, ou mantida ativa como failsafe (decisão manual por node, não automática)
- [x] Desinstalar Devolutions RDM na ZenBook (confirmado via `winget list` — já não aparece instalada)
- [ ] Verificar se a Devolutions RDM está instalada no `acer` (ex-DAT-WS-IA-01) e desinstalar se aplicável
- [ ] Adicionar tablet à tailnet e à documentação — dispositivo já adicionado ao Tailscale (2026-08-03), acesso remoto por design (mesmo padrão do telemóvel pessoal, sem IP fixo necessário). Falta: confirmar acesso fora da rede local (teste do Ivan pendente) e depois preencher IP Tailscale/hostname MagicDNS/modelo em `CLAUDE.md`, `README.md`, `network/README.md`, `docs/network-inventory.md` e `docs/diagrama-rede.md`

> Nota: o teste de acesso remoto via Tailscale ficou reposto a pendente na secção "Migração Rocky Linux → RHEL" acima, por depender do SO do servidor. A Smart TV deixa de ter um papel definido no projeto — estava só ligada ao Jellyfin, que saiu do âmbito (ver Notas Importantes). Os sub-projetos Web Server + Cloudflare e Guacamole foram retirados a 2026-10-18 (ver Estado atual).

---

## Notas Importantes

- Servidor é **headless** — tampa fechada, sem monitor nem teclado (a exceção para o TraderShark caiu com a saída dele do plano, a 2026-10-18)
- ~~**Servidor com duas interfaces de rede ativas em permanência** (decisão de 2026-08-07): Ethernet (principal, ligado à rack, IP estático `192.168.1.2` confirmado) + Wi-Fi (IP dinâmico, atualmente `192.168.1.146`, fallback de acesso, decisão explícita do Ivan de não desligar — sem reserva DHCP por agora, decisão reconfirmada em 2026-08-09)~~ **Superado a 2026-10-12:** o recurso Wi-Fi foi abandonado (reativável), ver "Estado atual"
- A avaria na placa de som é **completamente irrelevante** para o funcionamento como servidor
- O acesso remoto (terminal ou gráfico) é feito **exclusivamente via Tailscale** quando fora da rede local
- **Smart TV sem papel definido no projeto** (desde 2026-08-05) — estava na tailnet/inventário só para aceder ao Jellyfin; com o Jellyfin removido, mantém-se como dispositivo de inventário de rede, mas sem função ativa até se decidir um novo uso
- RHEL suporta **sessões SSH simultâneas ilimitadas** (mesmo comportamento que o Rocky Linux, ambos baseados em OpenSSH/systemd)
- Windows 11 Pro limita a **1 sessão RDP** simultânea
- Monitorizar temperatura nas primeiras horas com tampa fechada
- A GUI instalada (GNOME, modo "Server with GUI") **não interfere** com o SSH — são serviços independentes
- A subscrição RHEL Developer cobre uso de **desenvolvimento individual**, não produção em equipa
- Para o RHCSA, a prática deve manter-se maioritariamente em CLI, mesmo com o GNOME instalado
- **Política de autenticação SSH:** preferência do Ivan é sempre chave SSH, nunca password, em todos os nodes da tailnet. Ao configurar SSH num node novo (ou rever um existente), a chave é a opção por defeito — não propor password como alternativa igualmente válida. Password authentication só se mantém ativa como failsafe mediante avaliação manual e explícita por node (não é o default). Homeserver cumpria isto em Rocky Linux (chave ED25519, password desativada) — **a repetir do zero após a reinstalação em RHEL**; nos portáteis (`asus`, `acer`) **não há SSH de entrada** (decisão de 2026-10-14): só são clientes
- **Limitação Azure AD + OpenSSH (Windows):** contas Windows "Azure AD Joined" cloud-only (sem AD on-premises por trás — SID `S-1-12-1-...`) não são resolvíveis pelo `sshd.exe` do Windows para autenticação (nenhum formato de username funciona: simples, `AzureAD\user`, ou SID direto). Confirmado na DAT-WS-IA-01. Antes de configurar chaves SSH num node Windows, confirmar primeiro o tipo de conta (`whoami /user` + `dsregcmd /status`) — se for Azure AD cloud-only, a chave SSH só funciona associada a uma conta local dedicada, não à conta principal do Ivan
- **`subscription-manager status` mostrar "Registered" (não "Current") é o esperado** — RHEL 10 Developer usa Simple Content Access (SCA), que elimina o conceito de entitlements "Current"/"Expired"; basta estar registado numa org com SCA para os repositórios funcionarem (confirmado com `dnf repolist` a mostrar `appstream`/`baseos` populados). Não é um problema a corrigir
- **Surfshark não instalado** — decisão de 2026-08-09; só fazia falta ao TraderShark, que saiu do plano a 2026-10-18
- **Reserva DHCP do Wi-Fi do servidor não criada** (o recurso Wi-Fi foi abandonado a 2026-10-12; criar a reserva só se for reativado) — decisão de 2026-08-09: manter IP dinâmico por agora (MAC confirmado: `80:a5:89:93:a6:fb`, para quando fizer sentido criar a reserva)
- **Devolutions RDM abandonado** — decidido não usar como gestor central de ligações. SSH fica no Warp (já usado, com IA), RDP fica no cliente nativo do Windows (`mstsc`, mesmo motor RDP que o RDM usaria por baixo). A centralização de ligações chegou a estar prevista no Guacamole, retirado do plano a 2026-10-18

---

## Convenções de Trabalho

- Língua: **Português de Portugal (PT)**
- Assistente: **Kody**
- Utilizador: **Ivan**
- Toda a documentação deste projeto é escrita em PT

---

## Convenção: Documentos de Estudo por Serviço/Integração

> Segue o protocolo global "Documentos de Estudo por Serviço/Integração" (memória geral do Ivan) — **perguntar sempre antes de criar**, nunca criar automaticamente. Este projeto não é exceção a essa regra.

Sempre que se instala/configura um **serviço ou capacidade nova e significativa** no projeto (ex: Avahi, Tailscale, montagem de disco, hardening SSH, firewalld, TraderScope em 24/7), **perguntar ao Ivan** se quer um documento de estudo dedicado antes de o criar. **Não aplicar a comandos triviais ou correções pontuais** — só a trabalho que introduz algo novo e relevante ao projeto.

**Localização:** `<área>/docs/NN-nome-do-topico.md` (ex: `network/docs/00-avahi-mdns.md`, `infra/docs/03-ssh.md`), seguindo a numeração sequencial já usada em cada pasta. Serviços em `services/<nome>/` recebem também a sua própria subpasta `docs/`.

**Estrutura obrigatória do documento** (modelo validado em `network/docs/00-avahi-mdns.md`):

1. **O que é** — explicação do conceito/protocolo/serviço
2. **Aplicabilidade** — onde e como se usa em geral
3. **Pertinência para este projeto** — porquê foi necessário aqui, alternativas avaliadas e razão da escolha (tabela comparativa quando fizer sentido)
4. **Comandos utilizados** — cada comando em bloco de código, seguido de explicação de **cada flag/switch individualmente** (o que cada opção faz e o que aconteceria sem ela)
5. **Teste de validação** — comandos e resultado que confirmou funcionamento, incluindo o que falta testar
6. **Comandos de referência/troubleshooting** — extras úteis para o futuro

**Objetivo:** material de estudo pessoal para o Ivan (aprendizagem de Linux/redes do zero) e, futuramente, base para uma versão sanitizada (sem IPs/MACs/hostnames reais) para portfolio público. **Este repositório é público** (decisão do Ivan a 2026-10-08), apesar de conter dados reais da rede doméstica. Uma versão de portfólio anonimizada continua a ser um repositório separado, criado só quando o Ivan o pedir explicitamente.
