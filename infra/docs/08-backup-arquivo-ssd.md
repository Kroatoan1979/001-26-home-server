# Backup do Arquivo Pessoal — SSD externo → HDD do homeserver

---

## O que é

Cópia do arquivo pessoal do Ivan (fotos, vídeos, documentos e software), que vive num SSD externo de 1 TB (`Ivan_SSD`, NTFS), para o HDD de 5 TB do homeserver (`/mnt/dados/backups/Ivan_SSD/`). A cópia é feita com **`rsync` sobre SSH** a partir do portátil (ZenBook, Omarchy) onde o SSD está ligado. A seguir verifica-se a integridade com **checksums SHA-256** calculados nos dois lados.

Peças envolvidas:

- **`rsync`** — copia árvores de ficheiros e, numa repetição, só transfere o que mudou
- **SSH** — transporte cifrado e autenticado por chave entre o portátil e o servidor (ver [03-ssh.md](03-ssh.md))
- **`sha256sum`** — calcula uma "impressão digital" de cada ficheiro; se os valores coincidem, o conteúdo é igual byte a byte

---

## Aplicabilidade

Este padrão (rsync por SSH + verificação por hash) é a base de muitos backups em Linux: cópias para um NAS ou servidor caseiro, migração de dados entre máquinas, réplicas para outro site. Ferramentas como `rsnapshot`, `borg` ou `restic` acrescentam versões e deduplicação por cima, mas a ideia de transportar e depois verificar é a mesma.

---

## Pertinência para este projeto

**Problema:** depois da reorganização do SSD (2026-10-11), as fotos e documentos organizados (`Arquivo/`, 115 GiB, 11.086 ficheiros) só existiam nesse SSD. O OneDrive tem uma cópia do conteúdo **anterior** à reorganização, mas não do `Arquivo`.

**Estratégia 3-2-1** (3 cópias, 2 suportes diferentes, 1 fora de casa):

| Cópia | Onde | Conteúdo |
|---|---|---|
| 1 | SSD externo `Ivan_SSD` | original (pastas antigas + `Arquivo/`) |
| 2 | HDD do homeserver | `Arquivo/` (este documento) |
| 3 (fora de casa) | OneDrive | pastas antigas; **falta enviar o `Arquivo/`** |

**Alternativas avaliadas:**

- **Ligar o SSD diretamente ao homeserver** — seria mais rápido (USB 3 local), mas o RHEL não traz suporte para NTFS (nem o módulo `ntfs3` nem o `ntfs-3g`, que só existe no EPEL). Instalar pacotes de terceiros para uma cópia pontual não compensa.
- **Disco novo dedicado** — desnecessário: o HDD de 5 TB tem 4,5 TB livres e o SMART está saudável (ver [04-storage-hdd-5tb.md](04-storage-hdd-5tb.md), "Reposição em RHEL").
- **`cp` / `scp`** — não retomam uma cópia interrompida nem copiam só as diferenças; o `rsync` faz as duas coisas.

**Pré-requisitos cumpridos antes da cópia:** SMART do HDD verificado; montagem persistente em `/mnt/dados` (sem isto, a cópia podia ir parar a um caminho em RAM); pasta de destino criada.

---

## Comandos utilizados

### 1. Proteger o original: SSD montado só em leitura

```bash
udisksctl unmount -b /dev/sda2
udisksctl mount -b /dev/sda2 -o ro
findmnt -no OPTIONS /run/media/ivan/Ivan_SSD
```

- `udisksctl` — gere montagens como utilizador normal, sem `sudo` (o mesmo serviço que monta discos USB automaticamente)
- `-b /dev/sda2` — o dispositivo de bloco (a partição NTFS do SSD)
- `-o ro` — monta **só em leitura**: nenhum comando, por engano, consegue alterar o original. Sem isto, um `rsync` com a origem e o destino trocados podia apagar ou alterar ficheiros do SSD
- `findmnt -no OPTIONS` — confirma que as opções começam por `ro`

### 2. Cópia

```bash
rsync -rt --info=progress2 --info=stats1 \
  /run/media/ivan/Ivan_SSD/Arquivo homeserver:/mnt/dados/backups/Ivan_SSD/
```

- `-r` — recursivo: copia a pasta e todas as subpastas. Sem isto, o `rsync` ignora diretórios
- `-t` — preserva as datas de modificação. É importante por duas razões: a data original das fotos não se perde, e numa repetição o `rsync` usa a data + o tamanho para saber o que já está copiado (sem `-t`, voltava a copiar tudo)
- **Porque não `-a`** (o habitual "modo arquivo"): o `-a` inclui também permissões, dono e grupo (`-pgo`), que no NTFS são simulados e não têm significado. Copiá-los criaria permissões estranhas no XFS
- `--info=progress2` — mostra o progresso global (bytes, %, velocidade), não ficheiro a ficheiro
- `--info=stats1` — resumo no fim (bytes enviados, velocidade média)
- `Arquivo` **sem `/` no fim** — copia a pasta em si, que fica em `.../Ivan_SSD/Arquivo/`. Com `Arquivo/`, copiava só o conteúdo, diretamente para dentro de `Ivan_SSD/`
- `homeserver:` — alias do `~/.ssh/config` do portátil; o `rsync` abre uma ligação SSH e corre um `rsync` do outro lado

Resultado: 122,7 GB (115 GiB) em 29,5 min, **~66 MB/s** pelo Wi-Fi 5 GHz do portátil (o servidor está por cabo gigabit). Por cabo, o limite seria o próprio HDD (~100–130 MB/s).

### 3. Verificação por checksum (nos dois lados, em paralelo)

```bash
( cd /run/media/ivan/Ivan_SSD/Arquivo && find . -type f -print0 | sort -z | xargs -0 sha256sum > ssd.sha256 ) &
ssh homeserver 'cd /mnt/dados/backups/Ivan_SSD/Arquivo && find . -type f -print0 | sort -z | xargs -0 sha256sum' > hs.sha256 &
wait
cmp ssd.sha256 hs.sha256 && echo "CHECKSUMS IGUAIS"
```

- `find . -type f` — lista só ficheiros (não pastas), com caminhos relativos (`./…`), para os dois lados serem comparáveis
- `-print0` / `sort -z` / `xargs -0` — separam os nomes com o carácter nulo em vez de quebra de linha; sem isto, nomes com espaços, acentos ou quebras de linha partem o comando
- `sort -z` — ordena a lista para os dois ficheiros de resultado ficarem pela mesma ordem e se poderem comparar linha a linha
- `sha256sum` — calcula o hash SHA-256 de cada ficheiro
- `&` e `wait` — correm as duas leituras ao mesmo tempo (uma no portátil, outra no servidor) e esperam pelas duas; demora o tempo da mais lenta, não a soma
- `cmp` — compara os dois ficheiros de hashes; sem diferenças, todos os ficheiros têm o mesmo caminho e o mesmo conteúdo
- **Porque não confiar só no `rsync`:** o `rsync` verifica cada bloco durante a transferência, mas não volta a ler do disco de destino o que lá escreveu. Ler outra vez os dois discos apanha erros de escrita e problemas no próprio suporte

---

## Teste de validação

| Teste | Resultado (2026-10-11) |
|---|---|
| `rsync` terminou sem erros | `EXIT 0`, 122.722.341.045 bytes enviados ✅ |
| Número de ficheiros | 11.086 no SSD = 11.086 no servidor ✅ |
| Checksums SHA-256 | `cmp` sem diferenças: **CHECKSUMS IGUAIS** ✅ |
| Espaço ocupado no HDD | 115 GiB; o disco fica com 4,4 TB livres |
| Destino é o HDD | `/mnt/dados` em `/dev/sdb1` ✅ |

A verificação levou ~25 min: o SSD é lido em poucos minutos, o HDD de 4800 rpm é o gargalo.

**Lista de hashes guardada** no servidor, em `/mnt/dados/backups/Ivan_SSD/Arquivo-2026-10-11.sha256`, para verificar o backup no futuro **sem precisar do SSD**:

```bash
ssh homeserver 'cd /mnt/dados/backups/Ivan_SSD/Arquivo && sha256sum -c --quiet ../Arquivo-2026-10-11.sha256 && echo OK'
```

- `sha256sum -c` — em vez de calcular hashes novos, lê a lista e confirma cada ficheiro contra ela
- `--quiet` — só mostra os ficheiros que **falham**; sem ela, imprime uma linha `OK` por ficheiro (11 mil linhas)
- Corre a partir de `Arquivo/` porque a lista tem caminhos relativos (`./…`)
- Útil, por exemplo, uma vez por ano ou depois de uma falha de energia, para detetar ficheiros corrompidos no HDD

**Por testar:**
- Uma **reposição** a sério: copiar uma pasta do backup para outro sítio e abrir os ficheiros (um backup só vale o que valer o restauro)
- Repetir o `rsync` depois de alterações no `Arquivo` e confirmar que só transfere as diferenças

---

## Comandos de referência e troubleshooting

```bash
# Simular a próxima atualização (não copia nada; mostra o que mudaria)
rsync -rtn -i /run/media/ivan/Ivan_SSD/Arquivo homeserver:/mnt/dados/backups/Ivan_SSD/

# Atualizar o backup com o que mudou (sem --delete: ficheiros apagados no SSD ficam no backup)
rsync -rt --info=progress2 /run/media/ivan/Ivan_SSD/Arquivo homeserver:/mnt/dados/backups/Ivan_SSD/

# Comparar conteúdo sem copiar (lê tudo nos dois lados; lento mas exaustivo)
rsync -rcn -i /run/media/ivan/Ivan_SSD/Arquivo homeserver:/mnt/dados/backups/Ivan_SSD/

# Espaço ocupado e número de ficheiros no servidor
ssh homeserver 'du -sh /mnt/dados/backups/Ivan_SSD; find /mnt/dados/backups/Ivan_SSD -type f | wc -l'

# Confirmar que o destino é o HDD e não uma pasta no disco de sistema
ssh homeserver 'findmnt -T /mnt/dados/backups'
```

- `-n` (`--dry-run`) — simula, não escreve nada
- `-i` (`--itemize-changes`) — uma linha por ficheiro que seria alterado, com o motivo (tamanho, data, conteúdo)
- `-c` (`--checksum`) — compara pelo conteúdo e não pela data + tamanho
- `--delete` — **não usado de propósito**: apagaria no backup o que fosse apagado no SSD, e um engano no SSD propagava-se ao backup
- `findmnt -T` — mostra em que sistema de ficheiros está um caminho; tem de aparecer `/dev/sdb1` em `/mnt/dados`
