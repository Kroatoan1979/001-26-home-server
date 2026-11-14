# HDD1TB: disco de backups do servidor

> Criado a 2026-11-12. Decisão do Ivan: o WD de 5 TB fica só para media e tudo o que é backup crítico passa para este disco. Registo da decisão em [CLAUDE.md](../../CLAUDE.md) (secção do estado atual).

---

## O que é

O **HDD1TB** é um disco rígido de 2,5" e 1 TB (Samsung/Seagate HN-M101MBB, um disco de portátil reaproveitado) metido numa caixa USB e ligado ao homeserver. Tem uma única partição **ext4** com a etiqueta `HDD1TB`, montada em **`/mnt/hdd1tb`**.

Guarda:

| Pasta | Conteúdo | Quem escreve |
|---|---|---|
| `05_Backups/restic-asus/` | Repositório do **rest-server**: o backup restic diário do asus (`/home/ivan` + `/etc`, cifrado, com versões) | O asus, pela tailnet (append-only) |
| `05_Backups/traderscope/` | Repositório restic e `ultima/traderscope.db` da TraderScope | `traderscope-backup.timer` (04:00) |
| `01_…` a `99_…` (raiz) | Cópia do arquivo pessoal do SSD externo | À mão |

---

## Aplicabilidade

- Um disco "de backups" separado do disco de dados é o padrão mais simples para seguir a regra **3-2-1**: 3 cópias, em 2 suportes diferentes, 1 delas fora de casa. Aqui, o disco do asus e o HDD1TB são as duas cópias locais; a cópia fora de casa (Drive) é a peça que falta.
- Discos USB em servidores caseiros são comuns, mas trazem dois riscos clássicos: podem **não montar no arranque** (e o servidor não deve ficar preso por causa disso) e, se não montarem, os programas podem **escrever na pasta vazia do ponto de montagem**, ou seja, no SSD do sistema, sem ninguém dar por isso. As opções `nofail` e `RequiresMountsFor` (mais abaixo) tratam destes dois casos.
- O **`badblocks -w`** antes de pôr um disco usado ao serviço é a forma mais segura de saber se ele aguenta: escreve e lê todos os setores e obriga o disco a remapear os setores fracos.

---

## Pertinência para este projeto

**Porquê agora.** O WD de 5 TB começou a dar setores ilegíveis a 2026-11-07: 30 setores pendentes (estáveis desde então) e o teste longo SMART falhou a 2026-11-09. Os dois backups mais importantes do servidor estavam lá: o restic do asus e a BD da TraderScope. O Ivan decidiu (2026-11-10/12) que o WD fica **só para media**, sem RMA, e que os backups críticos passam para um disco são.

**Alternativas avaliadas:**

| Alternativa | Porque não |
|---|---|
| Deixar os backups no WD | O disco já tem setores ilegíveis; um backup num disco a falhar dá uma falsa segurança |
| Deixar uma 2.ª cópia "degradada" no WD | Contraria a decisão (WD só media) e continua a ser um disco em que não se pode confiar |
| Comprar um disco novo | Custo e tempo; o HDD1TB estava parado e passou nos testes |
| Ir já só para a nuvem | A nuvem é a perna fora de casa, não substitui uma cópia local rápida de restaurar |

**Porquê ext4.** Lê-se no asus (Arch) e no homeserver (RHEL) sem nada instalado. O XFS não encolhe e repara pior em discos com problemas; o btrfs não é suportado no RHEL 10. O NTFS original foi apagado.

**Risco aceite.** O HDD1TB é um disco de portátil com ~12 450 horas, ligado por USB. Passa a ser a **única cópia local** dos backups. Por isso a cópia fora de casa (Drive, fase 3 do backup da TraderScope) deixa de ser opcional.

---

## Comandos utilizados

### 1. Teste de superfície (no asus, 2026-11-09/10)

```bash
systemd-inhibit --what=sleep:idle:handle-lid-switch \
  badblocks -b 4096 -t 0 -wsv -o ~/badblocks-HDD1TB.txt /dev/disk/by-id/<disco>
```

| Flag | O que faz | Sem ela |
|---|---|---|
| `systemd-inhibit --what=sleep:idle:handle-lid-switch` | Impede o asus de suspender (por inatividade ou por fechar a tampa) enquanto o comando corre | Ao fim de uns minutos o portátil suspendia e o teste de ~6 h morria a meio |
| `-b 4096` | Testa em blocos de 4096 bytes, o tamanho real dos setores do disco | Usaria 1024 bytes: mais lento e com números de bloco que não batem com o ext4 |
| `-t 0` | Padrão de teste: só zeros | Faz 4 passagens com padrões diferentes (0xaa, 0x55, 0xff, 0x00): 4× mais tempo (~24 h) |
| `-w` | Modo **escrita destrutivo**: escreve o padrão em todo o disco e volta a ler | Só lia (modo por omissão), o que não obriga o disco a remapear setores fracos. **Apaga tudo**, por isso só num disco vazio |
| `-s` | Mostra o progresso | Ficava em silêncio durante horas |
| `-v` | Modo detalhado (conta erros de leitura/escrita/comparação) | Só via o resultado final |
| `-o ficheiro` | Grava a lista de blocos maus num ficheiro | A lista só aparecia no ecrã e perdia-se |
| `/dev/disk/by-id/…` | Aponta para o disco pelo modelo e número de série | Com `/dev/sdX` o nome pode mudar entre ligações e o `-w` apagava o disco errado |

Resultado: **0 blocos maus** em 6 h 39 min. O único setor pendente que o SMART mostrava antes (Pending 1) desapareceu com a escrita (Pending 0, Realloc 0).

### 2. Partição e sistema de ficheiros (no asus, 2026-11-10)

```bash
sudo wipefs -a /dev/sdX
sudo parted -s /dev/sdX mklabel gpt mkpart HDD1TB ext4 1MiB 100%
sudo mkfs.ext4 -L HDD1TB -m 0 -E root_owner=1000:1000 /dev/sdX1
```

| Comando / flag | O que faz | Sem ela |
|---|---|---|
| `wipefs -a` | Apaga todas as assinaturas antigas (tabela de partições, NTFS) | Restos das 5 partições antigas podiam confundir o `blkid` e a montagem automática |
| `mklabel gpt` | Tabela de partições GPT | Com MBR funcionava, mas a GPT tem cópia de segurança no fim do disco |
| `mkpart … 1MiB 100%` | Uma partição a começar em 1 MiB, até ao fim | Um início desalinhado com os setores de 4K torna as escritas mais lentas |
| `-L HDD1TB` | Etiqueta do sistema de ficheiros | O disco aparecia só pelo UUID, difícil de reconhecer |
| `-m 0` | 0% de blocos reservados para o root | O ext4 reservava 5% (~46 GB) para o root, útil num disco de sistema e desperdiçado num disco de dados |
| `-E root_owner=1000:1000` | A raiz do sistema de ficheiros pertence ao UID/GID 1000 (o `ivan`, igual no asus e no homeserver) | A raiz ficava do `root` e era preciso `sudo chown` antes de escrever |

### 3. Montagem persistente (no homeserver, 2026-11-12)

Linha no `/etc/fstab`:

```
UUID=<uuid> /mnt/hdd1tb ext4 defaults,nofail,x-systemd.device-timeout=10s 0 2
```

| Campo / opção | O que faz | Sem ela |
|---|---|---|
| `UUID=…` | Identifica a partição pelo UUID | Com `/dev/sdc1`, se o WD e o HDD1TB trocarem de ordem no arranque, montava o disco errado |
| `nofail` | Se o disco não estiver ligado, o arranque continua | O servidor parava no modo de emergência à espera do disco, sem SSH |
| `x-systemd.device-timeout=10s` | Espera no máximo 10 s pelo disco | Esperava 90 s (o valor por omissão) em cada arranque sem disco |
| `0` (5.º campo) | Sem `dump` | Irrelevante hoje |
| `2` (6.º campo) | `fsck` no arranque, depois do disco do sistema | Com `0` nunca se verificava o sistema de ficheiros |

### 4. Mudança do repositório do rest-server (no homeserver, 2026-11-12)

```bash
sudo systemctl stop rest-server && \
sudo install -d -o restic -g restic -m 700 /mnt/hdd1tb/05_Backups/restic-asus && \
sudo rsync -aHAX --info=stats1 /mnt/dados/backups/restic/ /mnt/hdd1tb/05_Backups/restic-asus/ && \
sudo sed -i 's#/mnt/dados/backups/restic#/mnt/hdd1tb/05_Backups/restic-asus#g' /etc/systemd/system/rest-server.service && \
sudo systemctl daemon-reload && sudo systemctl start rest-server
```

| Comando / flag | O que faz | Sem ela |
|---|---|---|
| `systemctl stop rest-server` | Pára o servidor antes da cópia | Um backup do asus a meio da cópia deixava o repositório novo incompleto |
| `install -d -o restic -g restic -m 700` | Cria a pasta já com o dono `restic` e só ele lá entra | Ficava do `root` (o rest-server não conseguia escrever) ou legível por todos |
| `rsync -a` | Modo arquivo: recursivo, mantém permissões, donos, datas e links | Perdia o dono `restic` e as permissões 700/600 |
| `-H` | Mantém os *hard links* | Duplicava ficheiros ligados (o restic não usa, mas é o seguro) |
| `-A` | Mantém as ACL | Perdia ACL, se existissem |
| `-X` | Mantém os atributos estendidos (incluindo o contexto SELinux) | O SELinux podia ficar com contextos diferentes |
| `--info=stats1` | Mostra só as estatísticas no fim | Sem output nenhum (ou, com `-v`, uma linha por ficheiro) |
| `/origem/` com barra final | Copia o **conteúdo** da pasta | Criava `restic-asus/restic/…`, um nível a mais |
| `&&` entre os passos | Cada passo só corre se o anterior correu bem | Se o rsync falhasse (setor ilegível no WD), a unidade apontava na mesma para uma cópia incompleta |
| `sed -i 's#…#…#g'` | Troca o caminho nas 3 linhas da unidade (`RequiresMountsFor`, `--path`, `ReadWritePaths`) | Era preciso editar à mão e arriscar esquecer uma |
| `daemon-reload` | O systemd relê a unidade alterada | Arrancava com a configuração antiga, já em memória |

**Porque é que `RequiresMountsFor=` é obrigatório.** Com `RequiresMountsFor=/mnt/hdd1tb/05_Backups/restic-asus`, o systemd só arranca o rest-server depois de `/mnt/hdd1tb` estar montado. Se o disco não montar, o serviço não arranca, o backup do asus falha e o alerta dispara (`OnFailure=` global). Sem esta linha, o rest-server arrancava na pasta vazia do SSD do sistema, criava ali um repositório novo e o asus fazia backups para o sítio errado sem ninguém dar por nada. `ReadWritePaths=` é a outra metade: com `ProtectSystem=strict`, o serviço só consegue escrever nesse caminho.

A pasta antiga ficou intacta em `/mnt/dados/backups/restic` até haver backups automáticos bons no HDD1TB.

**Limpeza do WD (2026-11-14, com OK do Ivan).** Antes de apagar: restic do asus com 3 snapshots no HDD1TB (12d90cc2, 5ee9417e de 13/11 e be4c734b de 14/11, os dois últimos automáticos) e TraderScope com 3 backups automáticos às 04:00 (12, 13 e 14/11). Contagem: WD 1156 ficheiros (igual ao rsync) e HDD1TB 1223 (+67 dos snapshots novos). Apagados pelo nome exato:

```bash
rm -rf -- /mnt/dados/backups/traderscope.antigo-20261112      # dono ivan
ssh -t homeserver 'sudo rm -rf -- /mnt/dados/backups/restic'   # dono restic, precisa de sudo
```

| Flag | O que faz | Sem ela |
|---|---|---|
| `-r` | Apaga a pasta e tudo o que tem dentro | O `rm` recusa apagar uma pasta |
| `-f` | Não pede confirmação ficheiro a ficheiro (os ficheiros do restic são só de leitura) | Uma pergunta por cada um dos 1156 ficheiros |
| `--` | Fim das opções: o que vem a seguir é sempre um caminho | Um caminho começado por `-` seria lido como opção |
| `ssh -t` | Abre um terminal remoto para o `sudo` poder pedir a password | O `sudo` falha com "a terminal is required" |

Resultado: o WD passou de 270 G para 253 G ocupados e em `/mnt/dados/backups` ficou só a `Ivan_SSD`.

---

## Teste de validação

| Teste | Resultado (2026-11-12) |
|---|---|
| `badblocks` completo | 0 blocos maus (6 h 39 min) |
| SMART depois do badblocks | PASSED, Pending 0, Realloc 0, CRC 0 |
| Montagem pelo fstab | `/mnt/hdd1tb` montado às 18:01 |
| Cópia do repositório | 18,85 GB, sem erros de leitura no WD; **1156 = 1156** ficheiros |
| rest-server com o caminho novo | `active (running)` às 18:25 |
| Backup de teste a partir do asus | Snapshot `12d90cc2`, `Result=success` |
| `restic-check` do asus no repositório novo | 1.ª tentativa falhou por um lock abandonado de 2026-11-03 00:24 (backup interrompido; veio do WD na cópia, não é da mudança). Depois de `restic unlock`: 22/22 snapshots, 5% dos packs lidos (54/54), **no errors were found** |
| Backup da TraderScope | Destino `/mnt/hdd1tb/05_Backups/traderscope` (validado na sessão do projeto 002) |

**Por testar:**
- [x] Primeiro backup automático do asus e da TraderScope (04:00) no HDD1TB (13/11 e 14/11); cópias antigas do WD apagadas a 2026-11-14
- [ ] Reinício do servidor: o HDD1TB monta sozinho e o rest-server arranca depois dele
- [ ] Arranque **sem** o HDD1TB ligado: o servidor arranca, o rest-server fica parado e chega o alerta

---

## Comandos de referência e troubleshooting

```bash
# Está montado? Qual é o disco?
findmnt /mnt/hdd1tb
lsblk -f -o NAME,SIZE,MODEL,LABEL,MOUNTPOINTS

# Espaço
df -h /mnt/hdd1tb

# Saúde (o by-id evita trocar discos)
sudo smartctl -H -A /dev/disk/by-id/<HDD1TB> | grep -Ei 'result|Reallocated|Pending|Uncorrect|CRC|Power_On'

# rest-server: a apontar para onde?
systemctl cat rest-server | grep -E 'path|RequiresMounts|ReadWrite'
systemctl status rest-server --no-pager

# Último backup do asus (no asus)
systemctl show restic-backup -p Result -p ExecMainExitTimestamp
journalctl -u restic-backup -n 5 --no-pager -o cat

# Sistema de ficheiros com erros (desmontar primeiro; o rest-server pára sozinho por causa do RequiresMountsFor)
sudo systemctl stop rest-server
sudo umount /mnt/hdd1tb
sudo e2fsck -f /dev/disk/by-label/HDD1TB
sudo mount /mnt/hdd1tb && sudo systemctl start rest-server
```

| Sintoma | Causa provável | O que fazer |
|---|---|---|
| Backup do asus falha com "connection refused" | rest-server parado (disco não montou) | `findmnt /mnt/hdd1tb`; ver o cabo/caixa USB; `sudo mount /mnt/hdd1tb` e `sudo systemctl start rest-server` |
| `/mnt/hdd1tb` vazio mas sem erro | Disco não montado; a pasta é só o ponto de montagem no SSD | **Não escrever lá.** Montar primeiro |
| SMART com Pending/Realloc a subir | Disco a degradar-se | Avisar o Ivan; antecipar a cópia fora de casa e um disco novo |
| Erros de I/O no `dmesg` em `sdX` | Caixa USB ou cabo | Trocar o cabo/porta antes de culpar o disco |
| `restic-check` falha com "repository is already locked … lock was created at" (data antiga) | Lock abandonado por um backup interrompido (asus desligou a meio). O backup diário continua a funcionar (lock partilhado), mas o check e o prune precisam de lock exclusivo | No asus: `sudo bash -c 'set -a; . /etc/restic/env; set +a; export RESTIC_REST_PASSWORD=$(< /etc/restic/rest-password); restic list locks --no-lock; restic unlock'`. O `unlock` só remove locks abandonados (sem `--remove-all`); o rest-server append-only deixa apagar locks, mas não dados |
