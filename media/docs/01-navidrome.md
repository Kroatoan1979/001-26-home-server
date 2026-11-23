# Navidrome: servidor de música pessoal (Podman sem root + Quadlet)

> Instalado a 2026-11-23. Primeiro serviço do **media center**, o segundo projeto do homeserver (reaberto pelo Ivan a 2026-10-21; o MVP da TraderScope, que era a condição, ficou concluído a 2026-11-22). Decisões registadas em [CLAUDE.md](../../CLAUDE.md).

---

## O que é

O **Navidrome** é um servidor de música pessoal, de código aberto, escrito em Go: uma espécie de "Spotify caseiro" que toca a **nossa** coleção. Lê uma pasta de ficheiros de áudio, organiza-os pelas etiquetas (artista, álbum, ano, faixa), mostra as capas e serve a música por uma interface web e pelo protocolo **Subsonic**, um padrão antigo que dezenas de apps conhecem (no Android: Symfonium, Tempo, Substreamer, DSub).

Faz **transcodificação em tempo real** com o `ffmpeg`: um FLAC de ~1000 kbps pode chegar ao telemóvel em Opus de 128 kbps, cerca de 8 vezes menos dados, sem tocar no ficheiro original.

Aqui corre num **contentor Podman sem root**, gerido pelo systemd através de um ficheiro **Quadlet**.

| Peça | O que faz |
|---|---|
| **Podman** | Motor de contentores da Red Hat, compatível com Docker, sem daemon. "Sem root" (*rootless*): o contentor corre com um utilizador normal e o root dentro do contentor é esse utilizador cá fora. |
| **Quadlet** | Gerador do systemd incluído no Podman. Lê um ficheiro `.container` (formato parecido com uma unidade systemd) e gera a unidade `navidrome.service`. Substitui o antigo `podman generate systemd`. |
| **Linger** | Faz o systemd arrancar o "gestor de utilizador" de um utilizador no arranque da máquina, sem ele ter sessão aberta. Sem linger, os serviços de utilizador só correm enquanto há login. |
| **tailscale serve** | Proxy da Tailscale: publica um serviço local em HTTPS, com certificado válido, **só para a tailnet**. |

---

## Aplicabilidade

- **Servidores de média caseiros:** Navidrome (só música), Jellyfin (música e vídeo), Plex (proprietário). O Navidrome é o mais leve e o melhor dos três para música.
- **Contentores sem root com systemd** são a forma recomendada pela Red Hat de correr serviços em contentores no RHEL, e um **objetivo do exame RHCSA** ("configurar um contentor para arrancar automaticamente como serviço systemd", "contentores sem root", "armazenamento persistente com volumes", "SELinux para volumes").
- O padrão **"serviço só em localhost + proxy da tailnet"** serve para qualquer aplicação web caseira: não abre portas na firewall nem na LAN e dá HTTPS sem gerir certificados.

---

## Pertinência para este projeto

**Objetivo do Ivan:** ouvir a coleção de música (FLAC, discografias completas) fora de casa, no telemóvel (Redmi Note 14), sem gastar muitos dados móveis.

**Alternativas avaliadas (2026-11-23):**

| Opção | Decisão | Motivo |
|---|---|---|
| **Navidrome** | ✅ escolhido | Só música, leve (~100 MB de RAM), muitas apps, transcodificação |
| Jellyfin | ❌ | Mais pesado; para música é inferior. Fica como hipótese se um dia houver vídeo |
| Plex | ❌ | Proprietário, conta na cloud, funções pagas |
| VLC por SFTP | ❌ | Sem biblioteca, sem capas, sem transcodificação |

**Forma de instalação:**

| Opção | Decisão | Motivo |
|---|---|---|
| **Podman sem root + Quadlet, imagem oficial `deluan/navidrome`** | ✅ | A imagem **traz o `ffmpeg`**, que os repositórios do RHEL não têm. Isolamento num utilizador próprio. Matéria do RHCSA |
| RPM oficial do Navidrome | ❌ | Sem `ffmpeg` não há transcodificação; seria preciso juntar o EPEL ao RHEL só para isso |

**Outras decisões:**

- **Utilizador próprio `navidrome`** (UID 1002, sem shell, com linger), como o `kody`: isolado do `ivan` e da TraderScope.
- **Música só de leitura**, a partir do espelho `/mnt/dados/media/Music` (WD de 5 TB). O original está no asus (`~/Music`).
- **Acesso:** o Navidrome só escuta em `127.0.0.1:4533`; a `tailscale serve` publica `https://homeserver.taildc92c5.ts.net:4533` na tailnet. Nada fica aberto à LAN nem à internet. Também se evita o problema do arranque que o `rest-server` teve a 2026-11-23 (escutar no IP da tailnet antes de ele existir).
- **Versão fixada** (`0.64.2`), atualizada à mão.
- **Telemetria anónima desligada** (`ND_ENABLEINSIGHTSCOLLECTOR=false`; vem ligada por omissão).
- **O espelho da música passou a usar `--delete`** (aprovado pelo Ivan a 2026-11-23): sem isso, o Navidrome mostrava duplicados e álbuns que já não existiam no asus. Os ficheiros apagados vão para `.substituidos/<data>/Music` e ficam 90 dias.
- **Riscos aceites:** a música está no WD com 30 setores pendentes (se as faixas começarem a falhar, plano B: HDD1TB, decisão do Ivan); o Podman sem root está avariado para os outros utilizadores (ver troubleshooting).

---

## Comandos utilizados (RHEL, homeserver)

Os passos 1 a 6 estão nos scripts `~/navidrome-instalar.sh` e `~/navidrome-continuar.sh` do `ivan` no homeserver, corridos com `ssh -t homeserver sudo bash <script>`.

### 1. Criar o utilizador

```bash
sudo useradd -m -s /sbin/nologin -c "Navidrome (Podman sem root)" navidrome
```

| Flag | O que faz | Sem ela |
|---|---|---|
| `-m` | Cria a pasta pessoal `/home/navidrome` | Sem pasta pessoal, o Podman não tem onde guardar imagens nem o Quadlet |
| `-s /sbin/nologin` | Shell que recusa login interativo | O utilizador poderia entrar por SSH ou `su` |
| `-c "…"` | Comentário (campo GECOS), aparece no `/etc/passwd` | Utilizador sem descrição |

Um `useradd` normal (sem `-r`) também reserva **intervalos de UID/GID secundários** em `/etc/subuid` e `/etc/subgid` (aqui `655360:65536`). O Podman sem root precisa deles para mapear os utilizadores de dentro do contentor. Com `-r` (utilizador de sistema) esses intervalos não são criados.

### 2. Linger

```bash
sudo loginctl enable-linger navidrome
sudo systemctl start user@1002.service
```

- `enable-linger` faz o gestor de utilizador do `navidrome` arrancar no boot e ficar a correr sem sessão. Sem isto, o Navidrome parava quando não houvesse ninguém ligado e não arrancava depois de um reinício.
- O `systemctl start user@1002.service` arranca já esse gestor, sem esperar pelo próximo reinício.

### 3. SELinux: música legível pelo contentor

```bash
sudo semanage fcontext -a -t container_ro_file_t "/mnt/dados/media/Music(/.*)?"
sudo restorecon -R /mnt/dados/media/Music
```

| Parte | O que faz |
|---|---|
| `semanage fcontext` | Gere as regras **permanentes** de etiquetas SELinux por caminho |
| `-a` | Acrescenta uma regra (`-m` altera, `-d` apaga) |
| `-t container_ro_file_t` | Tipo SELinux "ficheiro que os contentores podem **ler**" (mas não escrever) |
| `"/mnt/dados/media/Music(/.*)?"` | Expressão regular: a pasta e tudo o que está dentro |
| `restorecon -R` | Aplica as regras aos ficheiros que já existem; `-R` = recursivo. Sem ele, a regra só valeria para ficheiros novos ou depois de um `restorecon` |

**Porquê assim e não com `:z` no volume:** o `:z` pede ao Podman para reetiquetar os ficheiros, mas um contentor sem root do `navidrome` não pode mudar etiquetas de ficheiros do `ivan`. Além disso, os ficheiros estavam como `unlabeled_t`. Ficheiros novos criados pelo espelho herdam o tipo da pasta-mãe, por isso ficam também `container_ro_file_t`.

### 4. storage.conf do utilizador (correção)

```ini
# /home/navidrome/.config/containers/storage.conf
[storage]
driver = "overlay"
graphroot = "/home/navidrome/.local/share/containers/storage"
runroot = "/run/user/1002/containers"
```

- **Problema encontrado:** o `dnf-automatic` atualizou o `podman` para a **6.1.4** a 2026-11-20, mas o `containers-common` (que traz `/usr/share/containers/storage.conf`) ficou na **5.8**. Com esta combinação, o Podman sem root tenta usar o armazenamento do root (`/var/lib/containers/storage`) e falha com `mkdir /var/lib/containers/storage/libpod: permission denied`, **para qualquer utilizador** (confirmado também com o `ivan`).
- `graphroot` = onde ficam as imagens e as camadas dos contentores; `runroot` = estado temporário (em `/run/user/<UID>`, apagado a cada arranque); `driver = "overlay"` = sistema de camadas.
- Um `storage.conf` por utilizador tem prioridade sobre o do sistema e só afeta esse utilizador.

### 5. Quadlet

```bash
sudo install -d -o navidrome -g navidrome -m 750 /home/navidrome/data
sudo install -d -o navidrome -g navidrome -m 755 /home/navidrome/.config/containers/systemd
sudo install -o navidrome -g navidrome -m 644 ~/navidrome.container \
  /home/navidrome/.config/containers/systemd/navidrome.container
```

`install -d` cria pastas; `-o`/`-g` define dono e grupo; `-m` define permissões, tudo num passo (sem `mkdir` + `chown` + `chmod`).

Conteúdo de `navidrome.container` (cópia em `media/navidrome.container` neste repositório):

| Linha | O que faz |
|---|---|
| `Image=docker.io/deluan/navidrome:0.64.2` | Imagem oficial, versão fixada. Com `:latest`, cada reinício podia trazer outra versão |
| `ContainerName=navidrome` | Nome fixo (`podman ps`, `podman logs navidrome`) |
| `PublishPort=127.0.0.1:4533:4533` | Porta só no loopback do servidor. Sem o `127.0.0.1`, ficava em todas as interfaces |
| `Volume=%h/data:/data:Z` | BD, cache e config em `/home/navidrome/data`. `%h` = pasta pessoal; `:Z` reetiqueta para uso exclusivo deste contentor (pode, porque a pasta é do `navidrome`) |
| `Volume=/mnt/dados/media/Music:/music:ro` | Música só de leitura (`:ro`); o Navidrome nunca escreve nos ficheiros |
| `Environment=ND_ENABLEINSIGHTSCOLLECTOR=false` | Desliga a telemetria anónima |
| `Environment=ND_SCANNER_SCHEDULE=24h` | Varrimento completo diário; no dia a dia o vigilante de ficheiros apanha as alterações |
| `ExecStartPre=/usr/bin/mountpoint -q /mnt/dados` | Não arranca sem o WD montado. Numa unidade de utilizador não se pode usar `RequiresMountsFor` |
| `Restart=on-failure` + `RestartSec=30` | Recupera sozinho de falhas |
| `TimeoutStartSec=600` | Dá tempo para descarregar a imagem na 1.ª vez |
| `WantedBy=default.target` | Arranca com o gestor do utilizador, ou seja, no boot, graças ao linger |

```bash
sudo systemctl --user -M navidrome@ daemon-reload
sudo systemctl --user -M navidrome@ start navidrome.service
```

- `--user -M navidrome@` = falar com o gestor de utilizador do `navidrome` a partir do root, sem entrar como ele (útil porque tem `nologin`).
- `daemon-reload` corre o gerador Quadlet, que cria `navidrome.service`. Unidades geradas não se ativam com `enable`: o `[Install]` do `.container` já trata disso.

### 6. Publicar na tailnet

```bash
sudo tailscale serve --bg --https=4533 http://127.0.0.1:4533
```

| Flag | O que faz | Sem ela |
|---|---|---|
| `--bg` | Fica em segundo plano e **persiste a reinícios** | Só funcionava enquanto o comando estivesse a correr |
| `--https=4533` | Escuta em HTTPS na porta 4533 da interface da tailnet, com certificado `*.ts.net` válido | Por omissão usaria a 443, que já serve a TraderScope |
| `http://127.0.0.1:4533` | Destino do proxy | |

Para desligar: `sudo tailscale serve --https=4533 off`.

### 7. Espelho da música com `--delete` (no asus)

No script `~/.local/bin/espelho-media` do asus (repositório `omarchy-config`):

```bash
rsync -a --partial --delete --backup --backup-dir="$sub/Music" \
  --exclude='.Trash*' --exclude='.directory' \
  ~/Music/ homeserver.local:/mnt/dados/media/Music/
```

| Flag | O que faz |
|---|---|
| `--delete` | Apaga no destino o que já não existe na origem |
| `--backup --backup-dir=…` | Em vez de apagar ou sobrepor de vez, move o ficheiro antigo para `.substituidos/<data>/Music`. O script apaga estas pastas ao fim de 90 dias |
| barras no fim (`Music/`) | Sincroniza o **conteúdo** de Music para dentro de Music, por isso o `--delete` nunca sai dessa pasta |
| guarda no script | Se `~/Music` tiver menos de 100 faixas (disco por montar, pasta apagada por engano), o `--delete` não corre e o espelho falha, com alerta pelo healthchecks |

---

## Teste de validação (2026-11-23)

| Teste | Resultado |
|---|---|
| `curl https://homeserver.taildc92c5.ts.net:4533/app/` a partir do asus | ✅ 200, certificado válido |
| `curl http://192.168.1.2:4533/` (LAN) | ✅ recusado |
| `ss -ltn` no servidor | ✅ 4533 só em `127.0.0.1` e na interface da tailnet |
| Varrimento | ✅ importou as 9 bandas à medida que o espelho as copiava, com as capas `cover.jpg` |
| Contas | ✅ administrador + utilizador sem privilégios para o telemóvel (passwords no Bitwarden) |
| Telemóvel (Symfonium, pela Tailscale) | ✅ toca. 1.º teste em FLAC original (985 kbps, `transcoding=false`) |
| Espelho com `--delete` | ✅ asus e espelho com 2429 faixas cada; 105 ficheiros (duplicado do Black Album + lixo antigo) foram para `.substituidos/` |

**Falta testar:**
- [ ] Transcodificação em dados móveis (Opus 128 no Symfonium em rede móvel): procurar `transcoding=true` no registo
- [ ] Reinício do servidor: o Navidrome arranca sozinho (linger + `WantedBy`)
- [ ] Os 13 álbuns sem capa (Tool ×7, Iron Maiden ×5, S&M CD 2): ver se o `external` (internet) as resolve
- [ ] Alerta ntfy se o serviço falhar; backup de `/home/navidrome/data`

---

## Comandos de referência e troubleshooting

```bash
# Estado e registo (como root; o navidrome não tem shell)
sudo systemctl --user -M navidrome@ status navidrome
journalctl _UID=1002 -f                       # registo do serviço e do contentor (o ivan também pode ler)
journalctl _UID=1002 -o cat | grep -i -E "level=(error|warning)"

# Reiniciar / parar
sudo systemctl --user -M navidrome@ restart navidrome
sudo systemctl --user -M navidrome@ stop navidrome

# Ver o contentor
sudo machinectl shell navidrome@ /usr/bin/podman ps
sudo machinectl shell navidrome@ /usr/bin/podman logs --tail 50 navidrome

# Atualizar a versão: mudar a tag Image= no .container, depois
sudo systemctl --user -M navidrome@ daemon-reload
sudo systemctl --user -M navidrome@ restart navidrome

# Ver o Quadlet gerado (útil quando o .container tem erros)
sudo machinectl shell navidrome@ /usr/libexec/podman/quadlet -dryrun -user

# SELinux: confirmar a etiqueta da música
ls -Zd /mnt/dados/media/Music
sudo ausearch -m avc -ts recent | grep navidrome   # recusas do SELinux

# tailscale serve
tailscale serve status
```

**Problemas conhecidos:**

| Sintoma | Causa | Solução |
|---|---|---|
| `mkdir /var/lib/containers/storage/libpod: permission denied` | podman 6.x com containers-common 5.8 (atualização automática de 2026-11-20) | `storage.conf` por utilizador (passo 4). Solução geral, por decidir: `rootless_storage_path = "$HOME/.local/share/containers/storage"` em `/etc/containers/storage.conf`, ou esperar pela atualização do `containers-common` |
| Álbuns duplicados ou que já não existem | Espelho sem `--delete` | Resolvido a 2026-11-23 (passo 7). Faixas ausentes ficam marcadas como "missing" no Navidrome (só o administrador vê; `ND_SCANNER_PURGEMISSING` por omissão = nunca) |
| `Artist not found in deezer` no registo | Nomes de músicos individuais nas etiquetas (ex.: "Brad J. Wilk") | Inofensivo: a imagem da banda vem do artista principal |
| Faixas a saltar ou a falhar | Setores ilegíveis no WD | `journalctl -k | grep -i "medium error"`; plano B: música no HDD1TB (decisão do Ivan) |
