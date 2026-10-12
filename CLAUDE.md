# Home Server + Tailscale — Contexto do Projeto

## 🔄 Estado atual (verificado a 2026-10-10; HDD atualizado a 2026-10-11; logind/journal e Wi-Fi a 2026-10-12)

> Prevalece sobre o texto histórico abaixo, quando houver diferença.
>
> - **Nomes do servidor** (renomeado a 2026-09-28): o hostname é `homelan` (LAN, mDNS `homelan.local`) e o nó Tailscale é `hometail` (FQDN MagicDNS completo omitido de propósito, porque o repositório é público). O nome `homeserver` deixou de existir na rede. Ficou apenas como alias SSH local (`~/.ssh/config`) no Omarchy.
> - **ZenBook Duo UX482:** desde outubro de 2026 corre **Omarchy (Arch Linux + Hyprland)** e já não Windows 11 Home. O Tailscale **ainda não está instalado** no Omarchy, por isso o nó `ux482` (`100.81.2.53`) está parado na tailnet.
> - **Avahi/mDNS:** ativo; `homelan.local` resolve na LAN. O serviço `mdns` foi aberto no firewalld a 2026-09-28.
> - **Chave GPG do repositório Tailscale:** importada (`gpg-pubkey` presente). O `sudo dnf makecache` de 2026-10-12 passou sem erros nos 3 repos (BaseOS, AppStream, Tailscale); o erro de 2026-10-11 era só por correr sem `sudo`.
> - **Wi-Fi do servidor (`wlp3s0`): recurso abandonado (decisão do Ivan a 2026-10-12).** Substitui a decisão de 2026-08-07. Investigado a 2026-10-12: o Wi-Fi estava em baixo desde 2026-09-01 00:18 **sem ser intencional** (o NetworkManager esgotou as 4 tentativas de religação; a causa exata perdeu-se porque o journal era volátil). **Motivo:** o recurso só cobria falhas do cabo ou da porta Ethernet (usa o mesmo router), esteve morto 6 semanas sem ninguém dar por isso, e o acesso alternativo já está garantido pelo Tailscale e pelo mDNS. **Ficou reativável:** o perfil `Vodafone-C27A91 2.4G` mantém-se com a password guardada, só com `connection.autoconnect no`; o rádio continua ligado. Para reativar: `sudo nmcli connection modify "Vodafone-C27A91 2.4G" connection.autoconnect yes connection.autoconnect-retries 0 && sudo nmcli connection up "Vodafone-C27A91 2.4G"` (o `autoconnect-retries 0` evita repetir a falha de setembro) e criar a reserva DHCP para o MAC `80:a5:89:93:a6:fb`.
> - **Loop de suspensão do `systemd-logind` corrigido a 2026-10-12.** O `/etc/systemd/logind.conf` criado em agosto não tinha `[Login]`, por isso a tampa fechada pedia `suspend` sem parar e o `suspend.target` mascarado recusava. Durante 66 dias o `logind` gastou ~82% de um core, gerou ~1,8 GB/semana de `/var/log/messages` e deixou o journal com só ~6 min de histórico. Corrigido com o drop-in `/etc/systemd/logind.conf.d/10-homeserver.conf` (o ficheiro antigo ficou como `logind.conf.bak-2026-10-12`); a máscara dos alvos de suspensão mantém-se. **Journal agora persistente** (`/etc/systemd/journald.conf.d/10-persistent.conf`, `SystemMaxUse=1G`). Os `/var/log/messages-2026*` antigos (~7 GB de lixo do loop) ficam a cargo do `logrotate` (decisão do Ivan a 2026-10-12: não apagar à mão; o último sai por volta de 2026-11-08). Ver [infra/docs/09-logind-journald.md](infra/docs/09-logind-journald.md).
> - **HDD externo 5 TB** (`/dev/sdb1`, XFS): **montado em `/mnt/dados` pelo `/etc/fstab`** (UUID + `nofail`) desde 2026-10-11. Regra `udev` do `hdparm` recriada (APM 254 confirmado). Pastas: só `backups/` (o Ivan apagou `filecloud/` e `arquivo/`, que estavam vazias, a 2026-10-11; confirmado a 2026-10-12 que não fazem falta: a `filecloud/` cria-se com o próprio serviço FileCloud e a `arquivo/` ficou redundante com `backups/Ivan_SSD/Arquivo`). SMART saudável a 2026-10-11 (0 setores realocados/pendentes, teste curto sem erros); teste longo em curso a 2026-10-12; vigiado pelo `smartd` desde 2026-10-12. Ver [infra/docs/04-storage-hdd-5tb.md](infra/docs/04-storage-hdd-5tb.md).
> - **Backup do arquivo pessoal (decisão do Ivan, 2026-10-11):** este disco guarda a cópia local do `Arquivo` do SSD externo de 1 TB (`Ivan_SSD`) em `/mnt/dados/backups/Ivan_SSD/`. Motivo: o SSD era a única cópia local das fotos; o OneDrive é a cópia fora de casa. Alternativas recusadas: ligar o SSD diretamente ao servidor (o RHEL não traz suporte NTFS) e um disco novo (este tem 4,5 TB livres e SMART saudável).
> - **Contentores:** o Podman está instalado (vem com o RHEL) e o Docker não. Falta decidir entre Docker e Podman para o FileCloud.
> - **SSH:** a chave ED25519 funciona a partir do DAT-WS-IA-01 e do Omarchy. `PasswordAuthentication` continua ativo (valor por omissão); falta a decisão do Ivan.
> - **Repositório público** (decisão do Ivan a 2026-10-08).
> - **Memória do Claude:** sincronizada entre o Omarchy e o Windows pelo repositório privado `claude-memoria` (concluído a 2026-10-10).

## ⚠️ Migração em curso: Rocky Linux → RHEL Developer

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
> - Conta Tailscale `digitalltogether.com`, plano Free
> - IPs LAN, reservas DHCP, topologia de rede
> - Política de SSH por chave (sem password)

---

## Objetivo

Transformar um portátil antigo (ASUS X555L) num home server headless a correr **RHEL (Red Hat Developer Subscription for Individuals)**, acessível remotamente via Tailscale. Além do papel de servidor doméstico, o projeto serve quatro objetivos adicionais:

- **Estudo e prática para a certificação RHCSA** (Red Hat Certified System Administrator) — maioritariamente via CLI (utilizadores/grupos, permissões, LVM/storage, systemd, firewalld, SELinux, containers)
- **TraderShark** — app gráfica nativa Linux de trading desportivo em modo ladder, sensível a latência, para uso local/LAN
- **FileCloud** — partilha e sincronização de ficheiros (self-hosted, tipo Dropbox/ownCloud) em Docker, com armazenamento em HDD externo 5TB. Consolida aqui o que estava planeado como projeto separado na AWS
- Acesso gráfico remoto: X11 forwarding (Windows 11) e xrdp (tablet Android)

---

## Hardware do Servidor

| Componente | Especificação |
|---|---|
| Modelo | ASUS X555L |
| CPU | Intel Core i7 (4ª ou 5ª geração) |
| RAM | 8 GB DDR3 |
| Disco interno | SSD (SO + serviços) |
| Disco externo | HDD 5TB WD My Passport (armazenamento FileCloud + backups — uso 24/7 aceitável para home server de uso familiar) |
| Rede | Ethernet ao router doméstico |
| Avaria | Placa de som — irrelevante para servidor |

---

## Stack Técnico

| Camada | Tecnologia |
|---|---|
| SO | **RHEL 10** (Red Hat Developer Subscription for Individuals), instalação "Server with GUI" (GNOME) — versão confirmada em 2026-08-05 como a base atual do exame RHCSA |
| Subscrição | `subscription-manager register` — conta gratuita developers.redhat.com, uso de desenvolvimento individual (não produção em equipa) |
| VPN mesh | Tailscale — conta `digitalltogether.com`, plano **Free** (decisão tomada em 2026-08-03, reconfirmada em 2026-08-05 para o servidor RHEL: mantém-se esta conta, sem necessidade de plano pago nem de migrar para conta pessoal — cobre bem o caso de uso) |
| Contenção | Docker + docker-compose |
| Partilha/sincronização de ficheiros | FileCloud Community Edition (portas 80/443) — consolida o projeto que estava planeado na AWS |
| Trading (app gráfica) | TraderShark — nativo Linux, sensível a latência, uso local/LAN |
| Acesso remoto Linux (terminal) | SSH |
| Acesso remoto Linux (gráfico) | X11 forwarding (`ssh -X` + VcXsrv/Xming no Windows), xrdp (tablet Android + Microsoft Remote Desktop) |
| Gestão de pacotes | `dnf` (não `apt`) |

---

## Topologia de Rede

### Dispositivos na Tailnet

| Dispositivo | OS | Protocolo de acesso |
|---|---|---|
| ASUS X555L (servidor) | RHEL (Developer Subscription) | SSH, X11 forwarding, xrdp |
| DAT-WS-IA-01 (workstation Ivan) | Windows 11 Pro | RDP / SSH |
| ZenBook Duo UX482 | Omarchy (Arch Linux + Hyprland), desde 10/2026 (antes: Windows 11 Home) | SSH; Tailscale ainda por instalar no Omarchy |
| Telemóvel (pessoal) | Android | App Tailscale |

> PC Kyndryl e Telemóvel Kyndryl excluídos do projeto — equipamentos corporativos, risco de violação de políticas de segurança da Kyndryl.

### IPs e Hostnames

| Dispositivo | IP LAN | IP Tailscale |
|---|---|---|
| Servidor — homelan / hometail (ASUS X555L) | `192.168.1.2` (estático, Ethernet `enp2s0`, reconfirmado a 2026-10-10) · Wi-Fi `wlp3s0` em baixo desde 2026-09-01, recurso abandonado a 2026-10-12, reativável (antes `192.168.1.146`, dinâmico) | `100.98.31.9` (novo node RHEL, confirmado a funcionar) |
| DAT-WS-IA-01 — Wi-Fi | `192.168.1.100` | `100.124.224.123` |
| DAT-WS-IA-01 — Ethernet | `192.168.1.101` | `100.124.224.123` |
| ZenBook UX482 — Wi-Fi | `192.168.1.110` | `100.81.2.53` |
| ZenBook UX482 — Ethernet USB-C | `192.168.1.111` | `100.81.2.53` |
| Telemóvel (pessoal, Redmi Note 14) | — | `100.78.250.58` |
| Smart TV (Xiaomi) | `192.168.1.120` | — (sem Tailscale) |

> Pool DHCP do router (Vodafone): `192.168.1.64–253`. Reservas DHCP dentro deste range. Homeserver com IP estático via `nmcli` fora do pool. **Decisão do Ivan (reconfirmada a 2026-10-12):** o servidor fica com IP estático no SO, de propósito; é a exceção à regra de reservas DHCP, que continua a valer para os outros dispositivos. Motivo: o servidor deve ter o mesmo IP mesmo que o router seja trocado ou reposto.

### Acesso à tailnet

- Conta Tailscale: `ivan.almeida@digitalltogether.com`
- Painel admin: `https://login.tailscale.com/admin/machines`
- **MagicDNS ativo** — resolução por hostname dentro da tailnet, funciona em qualquer rede (Wi-Fi, dados móveis), sem precisar do IP Tailscale. Confirmado com `ssh ivan@homeserver` a partir do telemóvel via dados móveis (nome da altura; hoje é `ssh ivan@hometail`).

### Nomes DNS dos nodes (MagicDNS)

| Dispositivo | Hostname MagicDNS |
|---|---|
| Servidor (ASUS X555L) | `hometail` (era `homeserver` até 2026-09-28) |
| DAT-WS-IA-01 | `dat-ws-ia-01` |
| ZenBook UX482 | `ux482` (parado; o Tailscale ainda não está instalado no Omarchy) |
| Telemóvel (Redmi Note 14) | `redmi-note-14` |

> ⚠️ Nomes atribuídos automaticamente pelo Tailscale — **por rever**: avaliar se vale a pena renomear para consistência com o resto da documentação (ex: `ux482` pouco descritivo). Ver tarefa pendente.

---

## Estrutura de Armazenamento

```
SSD interno
├── RHEL (Developer Subscription)
├── Docker + serviços (FileCloud, Tailscale)
└── Configurações e logs

HDD externo 5TB WD My Passport (ponto de montagem: /mnt/dados)
├── /mnt/dados/backups
│   └── Ivan_SSD/Arquivo   (cópia do arquivo pessoal do SSD externo, 2026-10-11 — ver infra/docs/08-backup-arquivo-ssd.md)
└── /mnt/dados/filecloud   (planeada: criar no deploy do FileCloud, com dono/SELinux do serviço)
```

> **Nota de armazenamento:** SSD M.2 2280 1TB (enclosure USB) usado como disco portátil de alta capacidade para transferência de ficheiros entre máquinas — não integrado no servidor. HDD 5TB WD My Passport escolhido pelo espaço; spin-down pode ser desativado via `hdparm` se causar delays de acesso.
>
> ⚠️ **Reestruturação de 2026-08-05:** as pastas `filmes`/`series`/`musica` (pensadas para Jellyfin/Navidrome) saem do plano. Passam a existir fisicamente no disco de uma reinstalação anterior — a limpar/renomear ao recriar a estrutura em RHEL, ver [infra/docs/04-storage-hdd-5tb.md](infra/docs/04-storage-hdd-5tb.md).

---

## Comandos Frequentes

```bash
# Ligar ao servidor via SSH (Tailscale / MagicDNS)
ssh ivan@hometail
# ou: ssh ivan@100.98.31.9

# Ligar ao servidor via SSH (LAN, mDNS — o .local é obrigatório)
ssh ivan@homelan.local

# Ligar ao servidor via SSH (LAN, Ethernet, IP estático confirmado)
ssh ivan@192.168.1.2

# Verificar estado do Tailscale
sudo tailscale status

# Verificar temperatura (headless)
watch -n 2 sensors

# Gerir contentores (hoje só existe Podman; Docker por decidir)
podman ps
docker-compose up -d
docker-compose down
docker ps

# Atualizar sistema
sudo dnf update -y

# Verificar estado da subscrição RHEL
sudo subscription-manager status

# Ligação gráfica X11 forwarding (a partir do Windows, com VcXsrv a correr)
ssh -X ivan@hometail
```

---

## Estado do Projeto (Checklist)

> **Próximas tarefas prioritárias (por ordem) — continuar por aqui:**
> - [x] **Importar a chave GPG do repositório Tailscale de forma persistente** (confirmado a 2026-10-10): `sudo rpm --import https://pkgs.tailscale.com/stable/rhel/10/repo.gpg` — sem isto, qualquer `dnf` que toque no repo `tailscale-stable` fica preso a pedir confirmação `[y/N]`
> - [ ] **Encontrar alternativa ao `xorg-x11-apps` para testar o X11 forwarding:** o pacote não existe nos repositórios do RHEL 10 (descontinuado/renomeado) — pesquisar `dnf search xclock`/`xcalc`/`xterm` (com a chave GPG já importada) e instalar o que estiver disponível como app de teste leve
> - [ ] **Testar `ssh -X ivan@hometail`** a partir da DAT-WS-IA-01 (ou, mais simples, do Omarchy, que é Linux e não precisa de VcXsrv) e confirmar que uma app gráfica (xclock ou equivalente) aparece no ecrã do Windows via VcXsrv — depois avançar para o TraderShark se fizer sentido
> - [ ] Decidir se avança já a desativação de `PasswordAuthentication` no SSH do homeserver — o bloqueio original (chave inconsistente) já está resolvido, por isso deixou de haver motivo técnico para adiar; falta só a decisão do Ivan
> - [x] Atualização de memória geral Claude no UX482: concluída a 2026-10-10. Os protocolos DAT estão no `CLAUDE.md` global e a memória sincroniza entre os PCs pelo repositório privado `claude-memoria`.

### Migração Rocky Linux → RHEL (reinstalação completa do servidor)

> Os pontos abaixo estavam concluídos em Rocky Linux e foram repostos a pendente porque serão refeitos do zero em RHEL. Ver [infra/docs/01-instalacao-rhel.md](infra/docs/01-instalacao-rhel.md).

- [x] Criar conta gratuita em developers.redhat.com e obter a subscrição Developer
- [x] Descarregar a ISO do RHEL — **DVD iso x86_64 (7.88 GB)**, não a Boot iso — ver [infra/docs/01-instalacao-rhel.md](infra/docs/01-instalacao-rhel.md)
- [x] Criar pen USB bootável com Rufus, modo DD Image
- [x] Instalar RHEL no ASUS X555L, modo "Server with GUI" (GNOME)
- [x] `subscription-manager register` — **sem `--auto-attach`** (opção removida no RHEL 10, usa Simple Content Access) — ver [infra/docs/05-subscription-manager.md](infra/docs/05-subscription-manager.md)
- [x] `dnf update -y` — confirmado indiretamente (sistema já em RHEL 10.2 "Coughlan", kernel atualizado)
- [x] Definir hostname `homeserver` (renomeado a 2026-09-28 para `homelan`; Tailscale `hometail`)
- [x] Configurar IP estático no homeserver via `nmcli` (`192.168.1.2/24`) — **resolvido em 2026-08-09**: existiam dois perfis para `enp2s0` (`Wired connection 1`, automático/DHCP, e `enp2s0`, manual/estático, já configurado corretamente mas com `autoconnect: no`). Corrigido com `nmcli connection modify "Wired connection 1" connection.autoconnect no`, `nmcli connection modify enp2s0 connection.autoconnect yes` e `nmcli connection up enp2s0`. Confirmado via `ip -4 a show enp2s0` e `ip route show default`
- [x] Criar utilizador `ivan`, não-root, com sudo
- [x] Configurar modo headless (tampa fechada): passos 1 e 2 concluídos. ⚠️ **O passo 1 estava partido** (`logind.conf` sem `[Login]`); refeito a 2026-10-12 com um drop-in, e `HandleLidSwitch = ignore` confirmado com `busctl`. Ver [infra/docs/02-modo-headless.md](infra/docs/02-modo-headless.md) e [infra/docs/09-logind-journald.md](infra/docs/09-logind-journald.md)
- [x] Instalar e configurar SSH — `sshd` ativo, porta aberta na firewall, ligação testada — ver [infra/docs/03-ssh.md](infra/docs/03-ssh.md)
- [x] Configurar autenticação por chave SSH (ED25519) — **resolvido em 2026-08-09**: a causa real não era inconsistência, era ausência total — `~/.ssh` nem existia no servidor. Criado `~/.ssh` (`chmod 700`), `authorized_keys` (`chmod 600`) com a chave pública `id_ed25519` da DAT-WS-IA-01, e `restorecon -Rv ~/.ssh` para repor o contexto SELinux (`ssh_home_t`). Confirmado com ligação `ssh -o BatchMode=yes` bem-sucedida
- [ ] Desativar autenticação por password SSH (`PasswordAuthentication no`) — bloqueio técnico resolvido (chave já 100% consistente); **falta só a decisão do Ivan** sobre avançar
- [x] Configurar Avahi (mDNS) no homeserver: ativo, resolve `homelan.local` desde 2026-09-28 (firewalld `mdns` aberto). Ver [network/docs/00-avahi-mdns.md](network/docs/00-avahi-mdns.md)
- [x] Remover o node antigo "homeserver" (Rocky Linux) no admin console do Tailscale
- [x] Instalar Tailscale no RHEL e autenticar (`tailscale up --hostname=homeserver`) — novo IP `100.98.31.9`. Ver [network/docs/01-tailscale-instalacao.md](network/docs/01-tailscale-instalacao.md)
- [x] Testar acesso remoto via Tailscale/MagicDNS — confirmado a partir do DAT-WS-IA-01 (`ssh ivan@homeserver`); falta testar a partir dos restantes dispositivos
- [ ] **Em configuração** (retomado 2026-08-09, decisão do Ivan — independente do TraderShark): X11 forwarding. Feito: `xorg-x11-xauth` já vinha instalado no servidor, `X11Forwarding yes` já ativo por omissão no `sshd_config`, VcXsrv instalado e a correr na DAT-WS-IA-01 (`vcxsrv.exe :0 -multiwindow -clipboard -wgl -ac`). A chave GPG do repo Tailscale já não bloqueia o `dnf` (`sudo dnf makecache` sem erros a 2026-10-12). Por fazer: encontrar alternativa ao `xorg-x11-apps` (não existe no RHEL 10) para testar com uma app gráfica leve, testar `ssh -X` de facto — ver [infra/docs/06-acesso-grafico-x11.md](infra/docs/06-acesso-grafico-x11.md)
- [ ] ⏸️ **Pausado** (2026-08-07, decisão do Ivan): Instalar e configurar xrdp, abrir porta 3389 — ver [infra/docs/07-acesso-grafico-xrdp.md](infra/docs/07-acesso-grafico-xrdp.md). Nota: mantém-se um caso de uso futuro independente (tablet Android), a retomar quando fizer sentido
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
- [ ] Instalar Docker ou Podman. Estado a 2026-10-10: o Podman já está instalado (vem com o RHEL) e o Docker não; falta decidir
- [ ] Deploy FileCloud via Docker (apontado a `/mnt/dados/filecloud`, pasta a criar neste passo com o dono e o contexto SELinux que o serviço precisar) — ver [services/filecloud/docs/01-instalacao.md](services/filecloud/docs/01-instalacao.md)
- [ ] Configurar portal de administração e primeira conta de utilizador no FileCloud
- [ ] ⏸️ **Pausado** (2026-08-07, decisão do Ivan — problemas na própria aplicação, projeto secundário sem urgência): Instalar e testar TraderShark. Ver [services/tradershark/README.md](services/tradershark/README.md)
- [ ] Praticar RHCSA: utilizadores/grupos, permissões, LVM/storage, systemd, firewalld, SELinux, containers — maioritariamente via CLI
- [ ] Checklist de validação final: SSH, Tailscale, X11, xrdp, FileCloud, TraderShark, a partir de todos os dispositivos

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
- [ ] Rever todos os nodes da tailnet (homeserver, ZenBook, DAT-WS-IA-01; nota de 2026-10-12: o ZenBook corre Omarchy sem Tailscale, por isso está offline na tailnet até se decidir instalá-lo) para confirmar que a autenticação SSH por chave está corretamente configurada em todos
- [ ] Avaliar node a node se a autenticação por password deve ser desativada, ou mantida ativa como failsafe (decisão manual por node, não automática)
- [x] Desinstalar Devolutions RDM na ZenBook (confirmado via `winget list` — já não aparece instalada)
- [ ] Verificar se a Devolutions RDM está instalada no DAT-WS-IA-01 e desinstalar se aplicável
- [ ] Adicionar tablet à tailnet e à documentação — dispositivo já adicionado ao Tailscale (2026-08-03), acesso remoto por design (mesmo padrão do telemóvel pessoal, sem IP fixo necessário). Falta: confirmar acesso fora da rede local (teste do Ivan pendente) e depois preencher IP Tailscale/hostname MagicDNS/modelo em `CLAUDE.md`, `README.md`, `network/README.md`, `docs/network-inventory.md` e `docs/diagrama-rede.md`

> Nota: instalação de Docker/FileCloud e teste de acesso remoto via Tailscale ficam repostos a pendente na secção "Migração Rocky Linux → RHEL" acima, por dependerem do SO do servidor. A Smart TV deixa de ter um papel definido no projeto — estava só ligada ao Jellyfin, que saiu do âmbito (ver Notas Importantes).

### Sub-projeto: Web Server + Cloudflare
- [ ] Sessão de estudo: conceitos de proxy e reverse proxy
- [ ] Decidir qual o web server a usar (Caddy / Nginx / Traefik)
- [ ] Transferir gestão DNS do `digitalltogether.xyz` da Hostinger para a Cloudflare
- [ ] Configurar Cloudflare Tunnel no home server
- [ ] Instalar e configurar o web server escolhido via Docker
- [ ] Configurar reverse proxy para expor o web server via `digitalltogether.xyz`
- [ ] Testar acesso público ao site de estudo

### Sub-projeto: Guacamole (Gestão de Ligações Web)
> Pré-requisitos: Docker operacional + Cloudflare Tunnel configurado
- [ ] Estudar arquitetura do Apache Guacamole (guacd + guacamole-client)
- [ ] Instalar Guacamole via Docker no homeserver
- [ ] Configurar ligações SSH e RDP para todos os nodes da rede
- [ ] Expor via Cloudflare Tunnel (ex: `remoto.digitalltogether.xyz`)
- [ ] Testar acesso via browser de dentro e fora da rede

---

## Notas Importantes

- Servidor é **headless por padrão** — tampa fechada, sem monitor nem teclado, **exceto durante o uso do TraderShark** (sessão gráfica local, sensível a latência) — ⚠️ nota 2026-08-07: TraderShark pausado, ver checklist; o modo headless mantém-se como configuração base do servidor de qualquer forma
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
- O **TraderShark é sensível a latência** (modo ladder) — recomendado correr localmente ou em LAN por ethernet, evitando X11/xrdp pela internet durante trading ativo
- **Política de autenticação SSH:** preferência do Ivan é sempre chave SSH, nunca password, em todos os nodes da tailnet. Ao configurar SSH num node novo (ou rever um existente), a chave é a opção por defeito — não propor password como alternativa igualmente válida. Password authentication só se mantém ativa como failsafe mediante avaliação manual e explícita por node (não é o default). Homeserver cumpria isto em Rocky Linux (chave ED25519, password desativada) — **a repetir do zero após a reinstalação em RHEL**; ZenBook e DAT-WS-IA-01 continuam pendentes de configuração/revisão, independentemente da migração do servidor
- **Limitação Azure AD + OpenSSH (Windows):** contas Windows "Azure AD Joined" cloud-only (sem AD on-premises por trás — SID `S-1-12-1-...`) não são resolvíveis pelo `sshd.exe` do Windows para autenticação (nenhum formato de username funciona: simples, `AzureAD\user`, ou SID direto). Confirmado na DAT-WS-IA-01. Antes de configurar chaves SSH num node Windows, confirmar primeiro o tipo de conta (`whoami /user` + `dsregcmd /status`) — se for Azure AD cloud-only, a chave SSH só funciona associada a uma conta local dedicada, não à conta principal do Ivan
- **`subscription-manager status` mostrar "Registered" (não "Current") é o esperado** — RHEL 10 Developer usa Simple Content Access (SCA), que elimina o conceito de entitlements "Current"/"Expired"; basta estar registado numa org com SCA para os repositórios funcionarem (confirmado com `dnf repolist` a mostrar `appstream`/`baseos` populados). Não é um problema a corrigir
- **Surfshark não instalado** — decisão de 2026-08-09: deixar por instalar por agora, já que estava ligado ao TraderShark (pausado). Reavaliar se/quando o TraderShark for retomado
- **Reserva DHCP do Wi-Fi do servidor não criada** (o recurso Wi-Fi foi abandonado a 2026-10-12; criar a reserva só se for reativado) — decisão de 2026-08-09: manter IP dinâmico por agora (MAC confirmado: `80:a5:89:93:a6:fb`, para quando fizer sentido criar a reserva)
- **Devolutions RDM abandonado** — decidido não usar como gestor central de ligações. SSH fica no Warp (já usado, com IA), RDP fica no cliente nativo do Windows (`mstsc`, mesmo motor RDP que o RDM usaria por baixo). A centralização de ligações fica para o Guacamole (sub-projeto já planeado), que cobre o mesmo objetivo via browser e com acesso remoto pelo Cloudflare Tunnel

---

## Convenções de Trabalho

- Língua: **Português de Portugal (PT)**
- Assistente: **Kody**
- Utilizador: **Ivan**
- Toda a documentação deste projeto é escrita em PT

---

## Convenção: Documentos de Estudo por Serviço/Integração

> Segue o protocolo global "Documentos de Estudo por Serviço/Integração" (memória geral do Ivan) — **perguntar sempre antes de criar**, nunca criar automaticamente. Este projeto não é exceção a essa regra.

Sempre que se instala/configura um **serviço ou capacidade nova e significativa** no projeto (ex: Avahi, Tailscale, Docker, FileCloud, montagem de disco, hardening SSH, Cloudflare Tunnel, Guacamole), **perguntar ao Ivan** se quer um documento de estudo dedicado antes de o criar. **Não aplicar a comandos triviais ou correções pontuais** — só a trabalho que introduz algo novo e relevante ao projeto.

**Localização:** `<área>/docs/NN-nome-do-topico.md` (ex: `network/docs/00-avahi-mdns.md`, `infra/docs/03-ssh.md`), seguindo a numeração sequencial já usada em cada pasta. Serviços em `services/<nome>/` recebem também a sua própria subpasta `docs/`.

**Estrutura obrigatória do documento** (modelo validado em `network/docs/00-avahi-mdns.md`):

1. **O que é** — explicação do conceito/protocolo/serviço
2. **Aplicabilidade** — onde e como se usa em geral
3. **Pertinência para este projeto** — porquê foi necessário aqui, alternativas avaliadas e razão da escolha (tabela comparativa quando fizer sentido)
4. **Comandos utilizados** — cada comando em bloco de código, seguido de explicação de **cada flag/switch individualmente** (o que cada opção faz e o que aconteceria sem ela)
5. **Teste de validação** — comandos e resultado que confirmou funcionamento, incluindo o que falta testar
6. **Comandos de referência/troubleshooting** — extras úteis para o futuro

**Objetivo:** material de estudo pessoal para o Ivan (aprendizagem de Linux/redes do zero) e, futuramente, base para uma versão sanitizada (sem IPs/MACs/hostnames reais) para portfolio público. **Este repositório é público** (decisão do Ivan a 2026-10-08), apesar de conter dados reais da rede doméstica. Uma versão de portfólio anonimizada continua a ser um repositório separado, criado só quando o Ivan o pedir explicitamente.
