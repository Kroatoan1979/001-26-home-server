# Home Server + Tailscale — Contexto do Projeto

## Objetivo

Transformar um portátil antigo (ASUS X555L) num home server headless a correr Rocky Linux 10, acessível remotamente via Tailscale. Serviços de media (Jellyfin, Navidrome) em Docker, com armazenamento em HDD externo 5TB.

---

## Hardware do Servidor

| Componente | Especificação |
|---|---|
| Modelo | ASUS X555L |
| CPU | Intel Core i7 (4ª ou 5ª geração) |
| RAM | 8 GB DDR3 |
| Disco interno | SSD (SO + serviços) |
| Disco externo | HDD 5TB WD My Passport (media + dados — uso 24/7 aceitável para home server de uso familiar) |
| Rede | Ethernet ao router doméstico |
| Avaria | Placa de som — irrelevante para servidor |

---

## Stack Técnico

| Camada | Tecnologia |
|---|---|
| SO | Rocky Linux 10 (Minimal) |
| VPN mesh | Tailscale — conta `digitalltogether.com` em trial (14 dias, a partir de 2026-07-07). Decisão pendente: manter plano pago ou migrar para conta pessoal gratuita antes do fim do trial |
| Contenção | Docker + docker-compose |
| Media (vídeo) | Jellyfin (porta 8096) |
| Media (música) | Navidrome (protocolo Subsonic) |
| Acesso remoto Linux | SSH |
| Gestão de pacotes | `dnf` (não `apt`) |

---

## Topologia de Rede

### Dispositivos na Tailnet

| Dispositivo | OS | Protocolo de acesso |
|---|---|---|
| ASUS X555L (servidor) | Rocky Linux 10 | SSH |
| DAT-WS-IA-01 (workstation Ivan) | Windows 11 Pro | RDP / SSH |
| ZenBook Duo UX482 | Windows 11 Home | — (não aceita RDP de entrada) |
| Telemóvel (pessoal) | Android | App Tailscale |

> PC Kyndryl e Telemóvel Kyndryl excluídos do projeto — equipamentos corporativos, risco de violação de políticas de segurança da Kyndryl.

### IPs e Hostnames

| Dispositivo | IP LAN | IP Tailscale |
|---|---|---|
| Servidor — homeserver (ASUS X555L) | `192.168.1.2` (estático no SO) | `100.97.253.85` |
| DAT-WS-IA-01 — Wi-Fi | `192.168.1.100` | `100.124.224.123` |
| DAT-WS-IA-01 — Ethernet | `192.168.1.101` | `100.124.224.123` |
| ZenBook UX482 — Wi-Fi | `192.168.1.110` | `100.81.2.53` |
| ZenBook UX482 — Ethernet USB-C | `192.168.1.111` | `100.81.2.53` |
| Telemóvel (pessoal, Redmi Note 14) | — | `100.78.250.58` |
| Smart TV (Xiaomi) | `192.168.1.120` | — (sem Tailscale) |

> Pool DHCP do router (Vodafone): `192.168.1.64–253`. Reservas DHCP dentro deste range. Homeserver com IP estático via `nmcli` fora do pool.

### Acesso à tailnet

- Conta Tailscale: `ivan.almeida@digitalltogether.com`
- Painel admin: `https://login.tailscale.com/admin/machines`
- **MagicDNS ativo** — resolução por hostname dentro da tailnet, funciona em qualquer rede (Wi-Fi, dados móveis), sem precisar do IP Tailscale. Confirmado com `ssh ivan@homeserver` a partir do telemóvel via dados móveis.

### Nomes DNS dos nodes (MagicDNS)

| Dispositivo | Hostname MagicDNS |
|---|---|
| Servidor (ASUS X555L) | `homeserver` |
| DAT-WS-IA-01 | `dat-ws-ia-01` |
| ZenBook UX482 | `ux482` |
| Telemóvel (Redmi Note 14) | `redmi-note-14` |

> ⚠️ Nomes atribuídos automaticamente pelo Tailscale — **por rever**: avaliar se vale a pena renomear para consistência com o resto da documentação (ex: `ux482` pouco descritivo). Ver tarefa pendente.

---

## Estrutura de Armazenamento

```
SSD interno
├── Rocky Linux 10
├── Docker + serviços (Jellyfin, Navidrome, Tailscale)
└── Configurações e logs

HDD externo 5TB WD My Passport (ponto de montagem: /mnt/dados)
├── /mnt/dados/filmes
├── /mnt/dados/series
├── /mnt/dados/musica
├── /mnt/dados/backups
└── /mnt/dados/arquivo
```

> **Nota de armazenamento:** SSD M.2 2280 1TB (enclosure USB) usado como disco portátil de alta capacidade para transferência de ficheiros entre máquinas — não integrado no servidor. HDD 5TB WD My Passport escolhido pelo espaço; spin-down pode ser desativado via `hdparm` se causar delays no Jellyfin.

---

## Comandos Frequentes

```bash
# Ligar ao servidor via SSH (Tailscale / MagicDNS)
ssh ivan@homeserver
# ou: ssh ivan@100.97.253.85

# Ligar ao servidor via SSH (LAN)
ssh ivan@192.168.1.2

# Verificar estado do Tailscale
sudo tailscale status

# Verificar temperatura (headless)
watch -n 2 sensors

# Gerir serviços Docker
docker-compose up -d
docker-compose down
docker ps

# Atualizar sistema
sudo dnf update -y
```

---

## Estado do Projeto (Checklist)

- [x] Descarregar Minimal ISO Rocky Linux 10
- [x] Criar pen USB bootável com Rufus
- [x] Instalar Rocky Linux no ASUS X555L
- [x] Configurar modo headless (tampa fechada)
- [x] Instalar e configurar SSH
- [x] Configurar autenticação por chave SSH (ED25519) — chave instalada no homeserver para o utilizador `ivan`
- [x] Desativar autenticação por password SSH (`PasswordAuthentication no`) — chave ED25519 validada
- [x] Configurar IP estático no homeserver via `nmcli` (`192.168.1.2/24`) — independente do router
- [x] Configurar DHCP Reservation no router para DAT-WS-IA-01, ZenBook e Smart TV
- [x] Recolher MAC Ethernet USB-C do ZenBook (`00-E0-4C-4D-25-20`) e criar reserva
- [x] Actualizar toda a documentação para referenciar nodes com hostname + IP
- [x] Reiniciar dispositivos para aplicar novos IPs reservados e verificar com `ipconfig` — DAT-WS-IA-01 Wi-Fi `.100` ✅ · ZenBook Wi-Fi `.110` ✅ · ZenBook Ethernet USB-C `.111` ✅ · DAT-WS-IA-01 Ethernet `.101` ✅ (porta nativa Killer E2600)
- [x] Confirmar IP LAN do homeserver `.2` via SSH (`nmcli`) — confirmado `manual`/`192.168.1.2/24`, todos os IPs LAN da rede confirmados
- [x] Configurar Avahi (mDNS) no homeserver — resolução `homeserver.local` independente do Tailscale. Documentado em [network/docs/00-avahi-mdns.md](network/docs/00-avahi-mdns.md)
- [x] Testar resolução `homeserver.local` a partir do DAT-WS-IA-01 (`ping` + `ssh` confirmados)
- [x] Testar resolução `homeserver.local` a partir da ZenBook Duo UX482 (`ping` confirmado, 3-6ms — falhava com Surfshark VPN ligada, resolvido ao desligar)
- [x] Instalar Tailscale e integrar todos os dispositivos (homeserver, DAT-WS-IA-01, ZenBook, telemóvel — conta `digitalltogether.com` em trial)
- [x] Instalar e ativar OpenSSH Server na ZenBook Duo UX482 (`sshd` a correr, arranque automático configurado, firewall `OpenSSH-Server-In-TCP` ativo, perfis de rede Wi-Fi e Tailscale confirmados como `Private`)
- [ ] Instalar e ativar OpenSSH Server no DAT-WS-IA-01 (mesmo procedimento usado na ZenBook)
- [ ] Configurar autenticação por chave SSH na ZenBook (decisão tomada: chaves sempre, sem password — ver política de autenticação SSH abaixo). Conta é Administrador, por isso o OpenSSH Server do Windows exige a chave pública em `C:\ProgramData\ssh\administrators_authorized_keys`, não no `authorized_keys` normal do utilizador
- [ ] Testar `ssh ivana@localhost` na ZenBook com a chave configurada
- [ ] Testar SSH à ZenBook a partir de outro dispositivo (LAN e/ou Tailscale) — depende do ponto anterior
- [ ] Replicar a configuração de autenticação por chave SSH no DAT-WS-IA-01, quando o OpenSSH Server aí for instalado
- [ ] Rever todos os nodes da tailnet (homeserver, ZenBook, DAT-WS-IA-01) para confirmar que a autenticação SSH por chave está corretamente configurada em todos
- [ ] Avaliar node a node se a autenticação por password deve ser desativada, ou mantida ativa como failsafe (decisão manual por node, não automática)
- [x] Desinstalar Devolutions RDM na ZenBook (confirmado via `winget list` — já não aparece instalada)
- [ ] Verificar se a Devolutions RDM está instalada no DAT-WS-IA-01 e desinstalar se aplicável
- [ ] Instalar Docker
- [ ] Instalar Jellyfin via Docker
- [ ] Instalar Navidrome via Docker
- [x] Ligar HDD externo 5TB ao homeserver e montar via SSH (formatado em XFS, configurado `/etc/fstab` com UUID + `nofail`)
- [x] Configurar estrutura de pastas no HDD (`/mnt/dados/filmes`, `/mnt/dados/series`, `/mnt/dados/musica`, `/mnt/dados/backups`, `/mnt/dados/arquivo`)
- [ ] Desativar spin-down do HDD externo via `hdparm` para evitar delays no Jellyfin
- [ ] Testar acesso da Smart TV ao Jellyfin
- [x] Testar acesso remoto via Tailscale de todos os dispositivos (ZenBook confirmada — ligação direta P2P, ~2-3ms após handshake inicial)

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

- Servidor é **headless** — após setup inicial, opera com tampa fechada, sem monitor nem teclado
- A avaria na placa de som é **completamente irrelevante** para o funcionamento como servidor
- O acesso remoto é feito **exclusivamente via Tailscale** quando fora da rede local
- Na LAN (em casa), servidor e Smart TV comunicam diretamente — Tailscale não é necessário
- Rocky Linux 10 suporta **sessões SSH simultâneas ilimitadas**
- Windows 11 Pro limita a **1 sessão RDP** simultânea
- Monitorizar temperatura nas primeiras horas com tampa fechada
- **Política de autenticação SSH:** preferência do Ivan é sempre chave SSH, nunca password, em todos os nodes da tailnet. Ao configurar SSH num node novo (ou rever um existente), a chave é a opção por defeito — não propor password como alternativa igualmente válida. Password authentication só se mantém ativa como failsafe mediante avaliação manual e explícita por node (não é o default). Homeserver já cumpre (chave ED25519, password desativada); ZenBook e DAT-WS-IA-01 pendentes de configuração/revisão
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

Sempre que se instala/configura um **serviço ou capacidade nova e significativa** no projeto (ex: Avahi, Tailscale, Docker, Jellyfin, Navidrome, montagem de disco, hardening SSH, Cloudflare Tunnel, Guacamole), **perguntar ao Ivan** se quer um documento de estudo dedicado antes de o criar. **Não aplicar a comandos triviais ou correções pontuais** — só a trabalho que introduz algo novo e relevante ao projeto.

**Localização:** `<área>/docs/NN-nome-do-topico.md` (ex: `network/docs/00-avahi-mdns.md`, `infra/docs/03-ssh.md`), seguindo a numeração sequencial já usada em cada pasta. Serviços em `services/<nome>/` recebem também a sua própria subpasta `docs/`.

**Estrutura obrigatória do documento** (modelo validado em `network/docs/00-avahi-mdns.md`):

1. **O que é** — explicação do conceito/protocolo/serviço
2. **Aplicabilidade** — onde e como se usa em geral
3. **Pertinência para este projeto** — porquê foi necessário aqui, alternativas avaliadas e razão da escolha (tabela comparativa quando fizer sentido)
4. **Comandos utilizados** — cada comando em bloco de código, seguido de explicação de **cada flag/switch individualmente** (o que cada opção faz e o que aconteceria sem ela)
5. **Teste de validação** — comandos e resultado que confirmou funcionamento, incluindo o que falta testar
6. **Comandos de referência/troubleshooting** — extras úteis para o futuro

**Objetivo:** material de estudo pessoal para o Ivan (aprendizagem de Linux/redes do zero) e, futuramente, base para uma versão sanitizada (sem IPs/MACs/hostnames reais) para portfolio público. **Este repositório mantém-se privado** com os dados reais da rede doméstica — qualquer versão pública/portfolio será um repositório separado e anonimizado, criado só quando explicitamente pedido.
