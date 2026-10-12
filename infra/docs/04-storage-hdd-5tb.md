# Storage — HDD Externo 5TB (WD My Passport)

---

## O que é

Este documento cobre todo o percurso de um disco externo USB, desde a deteção pelo Linux até estar pronto a servir o Jellyfin/Navidrome: verificação da capacidade USB, identificação do disco, particionamento GPT, sistema de ficheiros XFS, montagem persistente via `/etc/fstab`, e gestão de energia (impedir que o disco "adormeça") via `hdparm` + `udev`.

Cada uma destas peças é uma camada independente do stack de armazenamento em Linux:

- **USB (xHCI/UAS)** — o barramento físico que liga o disco ao servidor
- **Bloco/Partição (GPT)** — a tabela que organiza o disco em partições
- **Sistema de ficheiros (XFS)** — a estrutura que organiza os dados dentro da partição
- **`/etc/fstab`** — a instrução que diz ao Linux para montar automaticamente no arranque
- **`hdparm` + `udev`** — a gestão de energia do disco e a automação que a torna persistente

---

## Aplicabilidade

Este fluxo aplica-se a qualquer disco externo USB (HDD ou SSD) ligado a um servidor Linux para expansão de armazenamento — casos comuns: NAS caseiro, servidor de media, backups externos, discos de arquivo. Os mesmos princípios servem tanto para discos permanentemente ligados como para discos amovíveis ocasionais (daí o uso de `nofail` no `fstab`).

---

## Pertinência para este projeto

O SSD interno do homeserver (223,6 GB) chega para o sistema operativo e para os containers Docker, mas não para armazenamento de dados em volume. Este HDD externo de 5TB (WD My Passport) é o armazenamento dedicado ao FileCloud e a backups.

> ⚠️ **Atualização de 2026-08-05:** este disco foi montado e formatado originalmente a pensar no Jellyfin/Navidrome (media server). Esses serviços saíram do projeto — o disco e a formatação (XFS, UUID, `hdparm`) mantêm-se inalterados, mas a estrutura de pastas é reorganizada (ver secção 8 abaixo) para servir o FileCloud em vez de bibliotecas de media.

**Duas decisões relevantes tomadas durante esta tarefa, que já foram corrigidas na documentação do projeto:**

1. **Capacidade real:** o disco identificou-se como 5TB (`5.00 TB` / `4.55 TiB`), não 4TB como estava inicialmente documentado no `CLAUDE.md`/`README.md`. Documentação corrigida.
2. **Sistema de ficheiros:** o plano original (`infra/docs/03-ssh.md` e um rascunho anterior) previa `ext4`. Optámos por **XFS** — sistema de ficheiros nativo e por defeito no Rocky Linux/RHEL, com bom histórico para ficheiros grandes (típico de media — filmes em `.mkv` de várias dezenas de GB) e suporte robusto a *extended attributes*, sem desvantagem relevante face ao `ext4` neste caso de uso.

**Porquê a gestão de energia (`hdparm`):** discos externos WD são conhecidos por "estacionar as cabeças" muito agressivamente após poucos segundos de inatividade, para poupar energia. Num servidor 24/7 com acessos espaçados ao longo do dia (ex: sincronização de ficheiros no FileCloud), isto causa dois problemas: atraso percetível ao aceder a um ficheiro depois do disco estar parado, e desgaste acumulado do contador `Load_Cycle_Count` do disco (tem limite de vida útil).

---

## Comandos utilizados

### 1. Verificação da capacidade USB (antes de ligar o disco)

```bash
sudo dnf install -y pciutils
lspci | grep -i usb
```

- `pciutils` — pacote que fornece o `lspci`; não vem instalado por defeito no Rocky Linux Minimal
- `lspci | grep -i usb` — lista os controladores USB da placa-mãe; procura-se por **`xHCI`** no nome, que indica suporte a USB 3.x (o homeserver tem um `Intel Wildcat Point-LP USB xHCI Controller`)

### 2. Confirmar a velocidade real da ligação (depois de ligar o disco)

```bash
sudo dmesg | grep -i "speed"
```

- `dmesg` — mostra o buffer de mensagens do kernel; requer `sudo` porque o acesso é restrito por segurança (`kernel.dmesg_restrict`)
- Procura-se a linha da deteção do disco: `new SuperSpeed USB device` confirma USB 3.0 real; `new high-speed USB device` indicaria USB 2.0 (mais lento)

### 3. Identificar o disco

```bash
lsblk
```

- Lista todos os discos e partições do sistema. Neste caso, `sda` é o disco interno (SO, **nunca tocar**) e `sdb` (4,5T) é o HDD externo recém-ligado, com 2 partições antigas do Windows (a apagar).

### 4. Limpar o disco e criar a tabela de partições

```bash
sudo wipefs -a /dev/sdb
sudo parted /dev/sdb -- mklabel gpt
sudo parted -a optimal /dev/sdb -- mkpart primary 0% 100%
```

- `wipefs -a` — apaga assinaturas de sistemas de ficheiros/tabelas de partição existentes no disco inteiro (aqui, resíduos do NTFS/GPT antigo do Windows)
- `parted /dev/sdb -- mklabel gpt` — cria uma tabela de partições **GPT** (obrigatório para discos acima de 2 TiB; o MBR antigo não suporta isso)
- `parted -a optimal /dev/sdb -- mkpart primary 0% 100%` — cria **uma única partição primária** ocupando 100% do disco; `-a optimal` alinha a partição da forma mais eficiente para o hardware (evita perdas de performance por desalinhamento)
- O `--` em ambos os comandos separa as opções do `parted` dos argumentos que começam por `%` ou `-`, evitando ambiguidade de parsing

### 5. Formatar em XFS

```bash
sudo mkfs.xfs -f /dev/sdb1
```

- `mkfs.xfs` — cria um sistema de ficheiros XFS na partição
- `-f` — força a formatação mesmo que o `mkfs` detete uma assinatura de sistema de ficheiros antiga (neste caso, restos do NTFS que o `wipefs` no disco inteiro não limpou dentro da nova partição)

### 6. Ponto de montagem e teste manual

```bash
sudo mkdir -p /mnt/dados
sudo mount /dev/sdb1 /mnt/dados
df -h /mnt/dados
```

- `mkdir -p` — cria a pasta que serve de ponto de montagem (o `-p` evita erro se já existir)
- `mount` — monta a partição manualmente, só para esta sessão (não sobrevive a reboot)
- `df -h` — confirma o espaço disponível (`-h` = *human-readable*, mostra em GB/TB em vez de blocos)

### 7. Montagem automática via `/etc/fstab`

```bash
sudo blkid /dev/sdb1
echo "UUID=aca21e44-94be-42fd-817d-633ab4be3568 /mnt/dados xfs defaults,nofail 0 2" | sudo tee -a /etc/fstab
sudo systemctl daemon-reload
sudo umount /mnt/dados
sudo mount -a
df -h /mnt/dados
```

- `blkid` — obtém o **UUID** da partição (identificador único e estável — o nome `/dev/sdb1` pode mudar se ligares outro disco USB antes deste, o UUID nunca muda)
- A linha do `fstab`: `UUID=... /mnt/dados xfs defaults,nofail 0 2`
  - `defaults` — opções de montagem standard (leitura/escrita, execução permitida, montagem automática)
  - `nofail` — **essencial para discos externos**: se o disco não estiver ligado no arranque, o boot não fica bloqueado à espera dele
  - `0` — desativa o `dump` (mecanismo de backup antigo, não usado)
  - `2` — ordem de verificação pelo `fsck` no arranque (`1` seria o disco de sistema; `2` = verificado depois)
- `systemctl daemon-reload` — o `systemd` gera automaticamente unidades de montagem a partir do `fstab`; isto força a releitura do ficheiro alterado
- `umount` + `mount -a` — testa a entrada do `fstab` sem precisar de reiniciar o servidor (`mount -a` monta tudo o que está no `fstab` e ainda não montado)

### 8. Estrutura de pastas

> ⚠️ Estrutura original (Jellyfin/Navidrome, 2026-08): `{filmes,series,musica,backups,arquivo}`. Reestruturada em 2026-08-05 para o FileCloud — comandos abaixo já refletem a estrutura nova; ao reinstalar o servidor (ver [01-instalacao-rhel.md](01-instalacao-rhel.md)), as pastas antigas de media não são recriadas.

```bash
sudo mkdir -p /mnt/dados/{filecloud,backups,arquivo}
sudo chown -R ivan:ivan /mnt/dados
```

- A sintaxe `{a,b,c}` do bash expande para múltiplos argumentos — evita repetir `mkdir -p` três vezes
- `chown -R ivan:ivan` — muda o dono das pastas para o utilizador `ivan` (por defeito, pastas criadas com `sudo mkdir` pertencem ao `root`, o que impediria o FileCloud de escrever/ler ficheiros mais tarde); `-R` aplica recursivamente a todas as subpastas

### 9. Gestão de energia — impedir o disco de "adormecer"

```bash
sudo dnf install -y hdparm
sudo hdparm -I /dev/sdb
sudo hdparm -B 254 -S 0 /dev/sdb
```

- `hdparm -I` — interroga o disco sobre as suas capacidades (confirma que a caixa USB permite passagem de comandos ATA via SAT — nem todas permitem)
- `-B 254` — define o *Advanced Power Management* para 254, o valor da norma ATA que significa "máximo desempenho, sem gestão de energia" (a escala 1-127 permite standby/spin-down; 128 = mínimo consumo sem standby; 254 = sem gestão de energia)
- `-S 0` — desativa por completo o temporizador de standby (nunca entra em suspensão por inatividade)
- Os avisos `SG_IO: bad/missing sense data` que aparecem são normais em pontes USB-SATA — a caixa não responde exatamente no formato esperado à confirmação, mas o valor fica aplicado (confirmado a seguir com `hdparm -B /dev/sdb`)

### 10. Tornar a gestão de energia persistente (via `udev`)

Os valores do `hdparm` aplicados no passo anterior são **voláteis** — perdem-se ao desligar o disco ou reiniciar o servidor. Para reaplicar automaticamente sempre que o disco for detetado, criámos uma regra `udev`:

```bash
udevadm info --query=all --name=/dev/sdb | grep -i serial
echo 'ACTION=="add", SUBSYSTEM=="block", KERNEL=="sd?", ENV{ID_SERIAL_SHORT}=="WD-WXA2D333EEPF", RUN+="/usr/sbin/hdparm -B 254 -S 0 /dev/%k"' | sudo tee /etc/udev/rules.d/99-hdd-power.rules
sudo udevadm control --reload
```

- `udevadm info --query=all --name=/dev/sdb` — mostra todas as propriedades que o `udev` conhece sobre o dispositivo; usámos para obter o `ID_SERIAL_SHORT`, o número de série do disco
- A regra em `/etc/udev/rules.d/99-hdd-power.rules`:
  - `ACTION=="add"` — só dispara quando o disco é ligado/detetado, não em qualquer outro evento
  - `SUBSYSTEM=="block"` — restringe a dispositivos de bloco (discos), não outro hardware
  - `KERNEL=="sd?"` — o `?` corresponde a exatamente um caracter, por isso apanha `sdb` (o disco inteiro) mas não `sdb1` (a partição) — evita correr o comando duas vezes
  - `ENV{ID_SERIAL_SHORT}=="WD-WXA2D333EEPF"` — **crucial**: garante que a regra só se aplica a este disco específico, nunca a outro dispositivo USB ligado no futuro
  - `RUN+="/usr/sbin/hdparm -B 254 -S 0 /dev/%k"` — comando executado quando a regra corresponde; `%k` é substituído pelo nome do dispositivo (`sdb`)
- `udevadm control --reload` — recarrega as regras `udev` a partir do disco (necessário depois de criar/alterar um ficheiro de regras)

---

## Teste de validação

**Montagem persistente:**
```bash
sudo umount /mnt/dados && sudo mount -a && df -h /mnt/dados
```
Resultado: `/dev/sdb1 4,6T 90G 4,5T 2% /mnt/dados` — montagem via `fstab` confirmada.

**Regra udev de gestão de energia** — prova de que funciona sem intervenção manual:
```bash
sudo hdparm -B 128 /dev/sdb          # repõe o valor "de fábrica" para simular reconexão
sudo udevadm trigger --action=add --name-match=/dev/sdb   # simula o disco a ser "ligado"
sleep 1
sudo hdparm -B /dev/sdb              # deve voltar a 254 sozinho
```
Resultado: `APM_level = 254` — confirmado que a regra reaplica a definição automaticamente, sem qualquer comando manual.

---

## Reposição em RHEL (2026-10-11)

A reinstalação Rocky Linux → RHEL apagou tudo o que vivia em `/etc` (a entrada do `fstab` e a regra `udev`). O disco, a partição, o XFS e o UUID ficaram intactos, por isso **não foi preciso reformatar**. Até aqui o disco estava a ser montado pelo udisks em `/run/media/ivan/<uuid>`, só porque havia uma sessão de utilizador. Num servidor isso é frágil: depois de um reboot sem login gráfico, o disco não monta, e um backup automático escreveria em `/run/media/...`, que está em RAM.

### 1. Saúde do disco antes de lhe confiar dados (SMART)

```bash
sudo smartctl -a /dev/sdb
sudo smartctl -t short /dev/sdb
sleep 150; sudo smartctl -l selftest /dev/sdb
```

- `smartctl` — lê o SMART, o autodiagnóstico do próprio disco (pacote `smartmontools`, já instalado no RHEL). Precisa de `sudo` porque fala diretamente com o dispositivo
- `-a` — mostra toda a informação SMART: identificação, avaliação geral e atributos. Sem flag, o `smartctl` não mostra nada de útil
- `-t short` — arranca o teste curto (~2 min) **dentro do disco**; o comando volta logo e o teste corre em segundo plano. Sem o tipo (`-t` sozinho) o comando fica incompleto
- `-l selftest` — mostra o registo dos testes já corridos e o resultado de cada um
- `sleep 150` — espera 2,5 min, para o teste curto ter tempo de acabar antes de pedir o registo

**Atributos que importam** (os restantes são informativos):

| ID | Atributo | Valor a 2026-10-11 | O que indica |
|---|---|---|---|
| 5 | `Reallocated_Sector_Ct` | 0 | setores estragados que o disco já substituiu |
| 197 | `Current_Pending_Sector` | 0 | setores que falharam a leitura e esperam decisão |
| 198 | `Offline_Uncorrectable` | 0 | setores que não se conseguiram ler de todo |
| 199 | `UDMA_CRC_Error_Count` | 0 | erros no caminho dos dados (cabo / ponte USB) |
| 9 | `Power_On_Hours` | 3.619 | idade em horas ligado |
| 193 | `Load_Cycle_Count` | 8.803 | estacionamentos das cabeças; sobe depressa sem a regra `udev` do passo 4 |

Qualquer valor acima de 0 nos IDs 5, 197 ou 198 é motivo para **não** usar o disco como única cópia de nada. Resultado: `PASSED`, todos a 0, teste curto `Completed without error`. **Por fazer:** o teste longo (`-t long`, ~194 min), que lê a superfície inteira.

### 2. Montagem persistente em `/mnt/dados`

```bash
sudo cp /etc/fstab /etc/fstab.bak-2026-10-11
sudo mkdir -p /mnt/dados
echo "UUID=aca21e44-94be-42fd-817d-633ab4be3568 /mnt/dados xfs defaults,nofail 0 2" | sudo tee -a /etc/fstab
sudo findmnt --verify
sudo systemctl daemon-reload
sudo umount /dev/sdb1
sudo mount -a
findmnt /mnt/dados
```

- `cp /etc/fstab /etc/fstab.bak-…` — cópia de segurança antes de mexer. Um `fstab` partido pode deixar o servidor em modo de emergência no próximo boot
- A linha do `fstab` é a mesma da secção 7 (já validada em Rocky Linux)
- `findmnt --verify` — valida o `fstab` (sintaxe, UUID existente, tipo de sistema de ficheiros) **antes** de o usar. O aviso `fstab has been modified, but systemd still uses the old version` é esperado neste ponto e desaparece com o `daemon-reload`
- `umount /dev/sdb1` — desmonta a montagem do udisks em `/run/media/...` (pelo dispositivo, porque o caminho era longo)
- `mount -a` — monta tudo o que está no `fstab` e ainda não está montado; testa a entrada sem reiniciar

> ⚠️ **Lição aprendida (2026-10-11):** por engano foram corridos dois blocos de comandos, um com `/srv/dados` e outro com `/mnt/dados`. Resultado: duas linhas no `fstab` para o mesmo UUID, o disco montado em dois sítios e o backup do `fstab` reescrito pelo segundo `cp` (já com a linha errada). Correção:
>
> ```bash
> sudo umount /srv/dados
> sudo sed -i '\|/srv/dados|d' /etc/fstab
> sudo rmdir /srv/dados
> grep aca21e44 /etc/fstab; findmnt -S /dev/sdb1
> ```
>
> - `sed -i '\|padrão|d'` — apaga do ficheiro (`-i`, edita no próprio ficheiro) as linhas que contêm o padrão; `\|…|` usa `|` como delimitador para não escapar as `/` do caminho
> - `findmnt -S /dev/sdb1` — lista **todos** os pontos de montagem desse dispositivo; tem de aparecer um só
>
> Regras a reter: antes de acrescentar ao `fstab`, ver se já existe linha para o UUID (`grep <uuid> /etc/fstab`), e dar nomes únicos aos backups (ex.: com hora, `fstab.bak-$(date +%F-%H%M)`).

### 3. Estrutura de pastas

```bash
cd /mnt/dados && rmdir filmes series musica && mkdir -p filecloud backups/Ivan_SSD
```

- `rmdir` — só apaga pastas **vazias**; se alguma tivesse ficheiros, falhava em vez de apagar dados (por isso é preferível a `rm -r` aqui)
- Sem `sudo`: a raiz do XFS pertence ao `ivan` (secção 8)
- `backups/Ivan_SSD` — destino da cópia do arquivo pessoal (ver o `CLAUDE.md`, "Estado atual")

### 4. Regra `udev` do `hdparm`

Os mesmos comandos da secção 10, mais a prova imediata:

```bash
sudo udevadm trigger --action=add --name-match=/dev/sdb
sleep 1; sudo hdparm -B /dev/sdb
```

Resultado: `APM_level = 254`.

### Validação da reposição

| Teste | Comando | Resultado |
|---|---|---|
| Uma só entrada no `fstab` | `grep aca21e44 /etc/fstab` | 1 linha, `/mnt/dados` ✅ |
| Um só ponto de montagem | `findmnt -S /dev/sdb1` | `/mnt/dados` ✅ |
| Backup do `fstab` = original | `diff <(grep -v aca21e44 /etc/fstab) /etc/fstab.bak-2026-10-11` | sem diferenças ✅ |
| Regra `udev` | `hdparm -B /dev/sdb` | `APM_level = 254` ✅ |
| SMART | `smartctl -l selftest` | teste curto sem erros ✅ |

**Reboot testado a 2026-10-12:** montou sozinho em `/mnt/dados` e a regra `udev` aplicou `APM_level = 254` no arranque ✅. **Teste SMART longo** arrancado a 2026-10-12 às 14:20, fim previsto às 17:46 (206 min). Resultado: _por registar_.

### Monitorização contínua com `smartd` (2026-10-12)

**Problema encontrado:** o `smartd` (ativo por omissão no RHEL) só vigiava o SSD interno `/dev/sda`. A linha `DEVICESCAN` **não apanha discos USB**, por isso o HDD com o backup do `Arquivo` nunca tinha sido monitorizado.

**Correção:** uma linha explícita no `/etc/smartmontools/smartd.conf`, **antes** do `DEVICESCAN` (o `smartd` ignora as linhas que vêm depois do `DEVICESCAN`):

```
/dev/disk/by-id/ata-WDC_WD50NDZW-11BCSS0_WD-WXA2D333EEPF -d sat -a -n standby,10,q -s L/../01/./03 -m root -M exec /usr/libexec/smartmontools/smartdnotify
```

- `/dev/disk/by-id/ata-…`: nome estável, ligado ao número de série. Sem isto (`/dev/sdb`), a letra podia mudar se outro disco USB fosse ligado primeiro
- `-d sat`: fala com o disco por SCSI-ATA Translation, através da bridge USB. Sem isto, o `smartd` tentava SCSI puro e não lia os atributos ATA
- `-a`: monitoriza tudo (saúde, atributos, setores realocados e pendentes, registo de erros e de testes). Sem isto, só faria o que estivesse explicitamente indicado
- `-n standby,10,q`: não acorda o disco se estiver em standby, até 10 verificações seguidas; `q` não regista no log quando salta uma verificação. Sem isto, o `smartd` acordava o disco a cada verificação (com o spin-down desligado pelo `hdparm -S 0` isto conta pouco, mas mantém a regra coerente com o `DEVICESCAN`)
- `-s L/../01/./03`: agenda um teste **longo** (`L`) em qualquer mês (`..`), no dia `01`, em qualquer dia da semana (`.`), às `03`h. Sem isto, os testes só correm quando alguém se lembra
- `-m root -M exec …/smartdnotify`: em caso de problema, avisa o `root` pelo script de notificação do RHEL. Sem isto, os problemas só ficavam no journal

```bash
sudo systemctl restart smartd
journalctl -u smartd --since -5min
```

- `restart`: o `smartd` relê o `smartd.conf`
- Validação: o journal mostrou `Adding to "monitor" list` para o WD50NDZW e `Monitoring 2 ATA/SATA … devices` (antes era 1) ✅. O `/dev/sdb` aparece como `same as /dev/disk/by-id/… ignored`, o que é correto: não fica duplicado

**Vantagem prática:** o resultado dos testes fica no journal, que o utilizador `ivan` (grupo `wheel`) lê **sem sudo**: `journalctl -u smartd | grep -i "self-test"`.

**Por melhorar:** o `-m root` entrega na caixa de correio local do `root`, que ninguém lê. Um aviso que chegue mesmo ao Ivan (email ou notificação) fica para quando se decidir um canal de alertas para o servidor.

---

## Comandos de referência / troubleshooting

```bash
# Saúde do disco: resumo, teste longo e registo de testes
sudo smartctl -H /dev/sdb
sudo smartctl -t long /dev/sdb
sudo smartctl -l selftest /dev/sdb

# Progresso de um teste em curso ("xx% of test remaining")
sudo smartctl -c /dev/sdb | grep -A1 "Self-test execution"

# O que o smartd registou (sem sudo)
journalctl -u smartd --since today

# Ver o estado de energia atual do disco (active/idle/standby)
sudo hdparm -C /dev/sdb

# Ver informação do sistema de ficheiros XFS montado
sudo xfs_info /mnt/dados

# Ver todos os discos com sistema de ficheiros e ponto de montagem
lsblk -f

# Testar uma regra udev sem a aplicar (modo simulação)
udevadm test $(udevadm info --query=path --name=/dev/sdb)

# Desmontar em segurança antes de desligar fisicamente o disco
sudo umount /mnt/dados
```

---

## Próximo passo

→ [network/README.md](../../network/README.md) — instalação do Tailscale
