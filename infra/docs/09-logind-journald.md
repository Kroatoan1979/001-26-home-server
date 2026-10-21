# systemd-logind e journald — Tampa fechada sem loops e logs persistentes

> Criado a 2026-10-12, depois de diagnosticar um loop de suspensão que tinha o `systemd-logind` a gastar um core inteiro desde 2026-08-07. Corrige e substitui o passo 1 de [02-modo-headless.md](02-modo-headless.md).

---

## O que é

**`systemd-logind`** é o serviço do systemd que gere sessões de utilizador, *seats* (teclado/ecrã/rato) e **eventos de energia**: tampa do portátil, botão de power, teclas de suspender/hibernar e inatividade. Para cada evento há uma diretiva `Handle*=` que diz o que fazer (`ignore`, `poweroff`, `suspend`, `hibernate`, `lock`, …).

**`systemd-journald`** é o serviço que recolhe os logs de todo o sistema (kernel, serviços, `stdout`/`stderr` das units) num formato binário indexado, lido com `journalctl`. A diretiva `Storage=` decide onde ficam:

| `Storage=` | Onde guarda | Sobrevive a reboot? |
|---|---|---|
| `volatile` | `/run/log/journal` (RAM) | Não |
| `persistent` | `/var/log/journal` (disco), cria a pasta se faltar | Sim |
| `auto` (por omissão) | Disco **só se `/var/log/journal` já existir**; senão RAM | Depende |
| `none` | Não guarda (só reencaminha, ex. para o rsyslog) | — |

**Ficheiros de configuração no systemd moderno (RHEL 10, systemd 257):**
- Os valores por omissão do fornecedor ficam em `/usr/lib/systemd/logind.conf` e `/usr/lib/systemd/journald.conf`. **Não se editam** (são substituídos nas atualizações).
- O administrador acrescenta **drop-ins** em `/etc/systemd/logind.conf.d/*.conf` e `/etc/systemd/journald.conf.d/*.conf`. São lidos por ordem alfabética, e o último a definir uma diretiva ganha.
- Cada ficheiro **tem de ter o cabeçalho da secção** (`[Login]` ou `[Journal]`). Uma diretiva fora de secção é **ignorada**, só com um aviso no log.

---

## Aplicabilidade

- **Portáteis usados como servidor** (o caso deste projeto): tampa fechada sem suspender.
- **Servidores e VMs em geral:** desativar as teclas de suspender, decidir o que faz o botão de power, `IdleAction`.
- **Journal persistente:** é o que permite investigar um problema que aconteceu *antes* do último reboot (`journalctl -b -1`), ou há várias semanas. Em RHEL é tema clássico do **RHCSA** ("configure persistent journal storage").
- **Drop-ins em vez de editar o ficheiro principal:** padrão geral do systemd (units, `logind`, `journald`, `resolved`, …). Sobrevive a atualizações de pacotes e mostra numa só pasta tudo o que foi alterado.

---

## Pertinência para este projeto

### O incidente (diagnosticado a 2026-10-12)

Ao investigar porque é que o Wi-Fi do servidor (`wlp3s0`) estava desligado, o `journalctl` não tinha nada de setembro. Na origem estavam dois problemas encadeados:

1. **O `logind` estava em loop desde o boot de 2026-08-07.** O `/etc/systemd/logind.conf` tinha só 3 linhas `HandleLidSwitch*=ignore`, **sem `[Login]`**, por isso o systemd ignorava-as. O valor real em vigor (`busctl`) era `HandleLidSwitch = "suspend"`. Com a tampa fechada (`LidClosed = true`), o `logind` tentava suspender, o `suspend.target` (mascarado no passo 2 do modo headless) recusava, e o `logind` voltava a tentar. Medido:
   - **~3500 mensagens em 3 segundos** (`Suspending...` / `Unit suspend.target is masked, refusing operation.` / `Failed to execute suspend operation: Permission denied`);
   - **~82% de CPU** constante no `systemd-logind`, com **7 semanas e 4 dias de CPU acumulado** (um core do i7-5500U desperdiçado durante 66 dias);
   - **~1,8 GB por semana** de `/var/log/messages` (o rsyslog copia tudo do journal).
2. **O journal era volátil:** `/var/log/journal` não existia, por isso o `Storage=auto` guardava tudo em RAM. Com o loop, o limite de tamanho em RAM só dava para **~6 minutos** de histórico. A causa da queda do Wi-Fi (última ligação a 2026-09-01 00:18) **perdeu-se**.

**Porque é que o ficheiro ficou sem `[Login]`:** no RHEL 10 (systemd 257) o `/etc/systemd/logind.conf` **já não vem instalado**; os valores por omissão estão em `/usr/lib/systemd/logind.conf`. O doc 02 mandava fazer `sudo vim /etc/systemd/logind.conf` e "adicionar ou descomentar" as linhas, o que criou um ficheiro novo, vazio, só com essas 3 linhas.

**Porque é que ninguém deu por isso:** o teste do modo headless (fechar a tampa e confirmar SSH) **passou**. O `systemctl mask` dos alvos de suspensão impedia mesmo a suspensão, por isso o servidor ficava acessível. A máscara **escondeu** que o `logind.conf` não estava a funcionar. Lição: testar o **valor real em vigor** (`busctl get-property`), não só o efeito final.

### Alternativas avaliadas

| Alternativa | Avaliação |
|---|---|
| Só acrescentar `[Login]` ao `/etc/systemd/logind.conf` | Funcionava, mas o ficheiro em `/etc` passa a sombrear o do fornecedor inteiro. Os drop-ins são o padrão recomendado |
| Mudar o GNOME (`gsettings ... sleep-inactive-ac-type 'nothing'`) | **Testado e não resolveu**: o pedido não vinha do GNOME (`CODE_FUNC=handle_action_execute` é o tratamento da tampa pelo próprio `logind`). Ficou aplicado como proteção extra |
| Passar o servidor para `multi-user.target` (sem GNOME) | Não resolve este problema (o `logind` trata a tampa com ou sem interface gráfica). Decidido à parte: feito a 2026-10-21 (ver checklist do `CLAUDE.md`) |
| Tirar a máscara dos alvos de suspensão | Não: a máscara é a segunda barreira, e foi ela que manteve o servidor acessível estes dois meses |
| **Drop-in `logind.conf.d/10-homeserver.conf` + máscara mantida** (escolhido) | Duas barreiras independentes, sobrevive a atualizações e fica visível numa só pasta |
| Journal: `SystemMaxUse` sem limite | Não: com um loop destes encheria o disco. **1G** chega para semanas de histórico normal |

---

## Comandos utilizados

Os comandos foram corridos por um script único (`sudo bash /tmp/fix-logind-journal.sh`), apagado depois. Aqui ficam um a um.

### 1. Diagnóstico (sem sudo)

```bash
journalctl -t systemd-logind -o cat --no-pager | sort | uniq -c | sort -rn | head
```

- `-t systemd-logind`: filtra pelo *syslog identifier* (o nome do programa que escreveu a mensagem). Sem isto, mostra o journal inteiro.
- `-o cat`: só o texto da mensagem, sem data, host e PID. Sem isto, cada linha seria única por causa da data e o `uniq -c` não agrupava nada.
- `--no-pager`: escreve direto para a pipe. Sem isto, abria o `less`.
- `sort | uniq -c | sort -rn | head`: conta as mensagens repetidas e ordena da mais frequente para a menos.

```bash
journalctl -t systemd-logind -n 3 -o verbose --no-pager | grep -E "MESSAGE=|CODE_FUNC"
```

- `-n 3`: só as 3 últimas entradas. Sem isto, imprimia milhares em formato verbose.
- `-o verbose`: mostra **todos os campos** da entrada, incluindo `CODE_FUNC`, a função do código-fonte que gerou a mensagem. Foi isto que mostrou `handle_action_execute` (o tratamento de um evento `Handle*`) em vez de um pedido externo por D-Bus.

```bash
busctl get-property org.freedesktop.login1 /org/freedesktop/login1 \
  org.freedesktop.login1.Manager HandleLidSwitch
```

- `get-property`: lê uma propriedade de um objeto D-Bus.
- `org.freedesktop.login1`: nome do serviço no bus (o `logind`).
- `/org/freedesktop/login1`: caminho do objeto.
- `org.freedesktop.login1.Manager`: interface que tem a propriedade.
- `HandleLidSwitch`: a propriedade. Devolve o valor **realmente em vigor**, e não o que está escrito nos ficheiros. Outras úteis: `LidClosed`, `Docked`, `HandlePowerKey`, `HandleSuspendKey`, `IdleAction`.

```bash
systemd-inhibit --list --no-pager
```

- `--list`: lista os *inhibitors* ativos (programas que bloqueiam ou atrasam suspensão, power e teclas). Serviu para confirmar que a tampa **não** estava inibida pelo GNOME (só as teclas de power/suspend estavam).

### 2. Correção do logind (com sudo)

```bash
sudo mv -v /etc/systemd/logind.conf /etc/systemd/logind.conf.bak-2026-10-12
```

- `mv`: tira o ficheiro partido do caminho. Fica com outro nome, para referência.
- `-v`: *verbose*, mostra o que foi renomeado. Sem isto, o `mv` não diz nada.

```bash
sudo mkdir -p /etc/systemd/logind.conf.d
```

- `-p`: cria os diretórios em falta e não dá erro se já existirem. Sem isto, falhava se a pasta já existisse.

Conteúdo de `/etc/systemd/logind.conf.d/10-homeserver.conf`:

```ini
# homelan: servidor headless de tampa fechada (2026-10-12)
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
HandleSuspendKey=ignore
HandleHibernateKey=ignore
```

- `[Login]`: **obrigatório**. Sem isto, todas as linhas abaixo são ignoradas (foi exatamente o bug).
- `HandleLidSwitch=ignore`: tampa fechada a bateria. Sem isto, o valor é `suspend`.
- `HandleLidSwitchExternalPower=ignore`: tampa fechada ligada à corrente. Sem isto, herda o valor de `HandleLidSwitch`.
- `HandleLidSwitchDocked=ignore`: tampa fechada com monitor externo ou dock (já é `ignore` por omissão; fica explícito).
- `HandleSuspendKey=ignore`, `HandleHibernateKey=ignore`: teclas Fn de suspender e hibernar. Sem isto, uma tecla carregada por engano (ou presa) disparava o mesmo loop.
- `HandlePowerKey` **não** foi alterado: continua `poweroff`. Um toque no botão desliga o servidor de forma limpa, o que é útil sem ecrã.
- Prefixo `10-`: dá ordem de leitura. Um drop-in futuro `20-*.conf` sobrepõe-se a este.

```bash
sudo systemctl reload systemd-logind
```

- `reload`: o `logind` relê a configuração (sinal `SIGHUP`, suportado desde o systemd 250) **sem reiniciar**. As sessões abertas (GNOME, SSH) não são afetadas. Um `restart` também funcionaria, mas é mais intrusivo.

### 3. Journal persistente (com sudo)

```bash
sudo mkdir -p /etc/systemd/journald.conf.d /var/log/journal
```

- Cria a pasta do drop-in e a pasta de destino dos logs. Com `Storage=persistent` o `journald` criava `/var/log/journal` sozinho; criá-la já deixa as permissões e o SELinux tratados nos passos seguintes.

Conteúdo de `/etc/systemd/journald.conf.d/10-persistent.conf`:

```ini
# homelan: logs sobrevivem a reboots (2026-10-12)
[Journal]
Storage=persistent
SystemMaxUse=1G
```

- `[Journal]`: secção obrigatória (a mesma lição do `logind`).
- `Storage=persistent`: guarda em `/var/log/journal`. Sem isto (`auto` sem a pasta), fica em RAM e perde-se no reboot.
- `SystemMaxUse=1G`: teto do espaço em disco do journal; o `journald` apaga os ficheiros mais antigos ao chegar lá. Sem isto, o limite por omissão é 10% do sistema de ficheiros (até 4 GB), o que com um loop como este enchia muito disco.

```bash
sudo systemd-tmpfiles --create --prefix /var/log/journal
```

- `--create`: aplica as regras do `tmpfiles.d` (dono `root:systemd-journal`, bit setgid, ACLs para os grupos `adm` e `wheel` lerem sem sudo).
- `--prefix /var/log/journal`: só para este caminho. Sem isto, aplicava todas as regras do sistema.

```bash
sudo restorecon -R /var/log/journal
```

- `restorecon`: repõe o contexto SELinux certo (`var_log_t`). Uma pasta criada à mão pode ficar com um contexto errado e o `journald` não consegue escrever.
- `-R`: recursivo.

```bash
sudo systemctl restart systemd-journald
sudo journalctl --flush
```

- `restart`: o `journald` relê a configuração. Ao contrário do `logind`, aqui o restart é o caminho normal e é seguro.
- `--flush`: passa o que estava em RAM (`/run/log/journal`) para o disco, já.

### 4. Proteção extra no GNOME (sem sudo, na sessão do ivan)

```bash
export DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type nothing
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type nothing
gsettings set org.gnome.settings-daemon.plugins.power power-button-action nothing
```

- `DBUS_SESSION_BUS_ADDRESS`: numa sessão SSH não há bus de sessão; isto aponta o `gsettings` para o bus da sessão GNOME do utilizador 1000. Sem isto, a alteração não chegava ao GNOME a correr.
- `sleep-inactive-*-type nothing`: o GNOME não pede suspensão por inatividade (à corrente e a bateria).
- `power-button-action nothing`: com a sessão GNOME aberta, quem trata o botão de power é o GNOME (tem um *inhibitor* sobre ele). Sem isto, um toque no botão pedia suspensão (que seria recusada).

---

## Teste de validação

Corrido a 2026-10-12, logo após o script:

| Verificação | Antes | Depois |
|---|---|---|
| `busctl get-property ... HandleLidSwitch` | `s "suspend"` | `s "ignore"` |
| Mensagens do `logind` | ~3500 em 3 s | **0** em 5 s; **1** num minuto |
| CPU do `systemd-logind` (`top`) | ~82% | **0,0%** |
| Load average | ~1,08 | a descer (0,37 no primeiro minuto) |
| `/var/log/journal/<machine-id>/` | não existia | criado (`root:systemd-journal`, setgid, ACL) |
| `journalctl --disk-usage` | 146,6 MB em RAM | 133,4 MB, já em disco |

**Falta testar:**
- [x] **Reboot** (feito a 2026-10-12): `HandleLidSwitch` continuou `ignore`, 0 mensagens de suspensão com a tampa fechada, `logind` a 0% de CPU, e `journalctl --list-boots` mostra **dois** boots (o journal sobreviveu) ✅
- [ ] Daqui a uma semana: `journalctl --disk-usage` para ver o ritmo de crescimento real, e `ls -l /var/log/messages*` para confirmar que o ficheiro cresce ao ritmo normal (KB/MB por dia, não GB).

---

## Comandos de referência e troubleshooting

```bash
# Valor real de todas as diretivas Handle* e estado da tampa
busctl introspect org.freedesktop.login1 /org/freedesktop/login1 | grep -E "Handle|Lid|Docked|IdleAction"

# Configuração efetiva do logind/journald, com todos os drop-ins (mostra a ordem)
systemd-analyze cat-config systemd/logind.conf
systemd-analyze cat-config systemd/journald.conf

# Avisos de sintaxe (ex.: "Assignment outside of section. Ignoring.")
journalctl -b -u systemd-logind -p warning

# Quem está a bloquear ou atrasar a suspensão e as teclas
systemd-inhibit --list

# Alvos de suspensão (devem estar masked)
systemctl is-enabled sleep.target suspend.target hibernate.target hybrid-sleep.target

# Journal: boots guardados, espaço, logs do boot anterior
journalctl --list-boots
journalctl --disk-usage
journalctl -b -1 -p err

# Logs de um serviço numa janela de tempo (o que se queria para o Wi-Fi)
journalctl -u NetworkManager --since "2026-09-01" --until "2026-09-02"

# Limpar o journal manualmente, se for preciso
sudo journalctl --vacuum-size=500M
sudo journalctl --vacuum-time=4weeks

# Detetar "spam" de logs: identificadores mais ruidosos na última hora
journalctl --since -1h -o json | grep -o '"SYSLOG_IDENTIFIER":"[^"]*"' | sort | uniq -c | sort -rn | head
```

**Reverter**, se for preciso: apagar `/etc/systemd/logind.conf.d/10-homeserver.conf` e fazer `reload` ao `logind`; apagar `/etc/systemd/journald.conf.d/10-persistent.conf`, apagar `/var/log/journal` e fazer `restart` ao `journald`.

**Pendente relacionado:** os `/var/log/messages-2026*` antigos (~7 GB de mensagens deste loop). **Decisão do Ivan (2026-10-12):** deixar o `logrotate` removê-los sozinho (4 rotações semanais), sem apagar à mão. O último deve sair por volta de 2026-11-08; confirmar então com `ls -lh /var/log/messages*`.
