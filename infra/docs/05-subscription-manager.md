# subscription-manager — Registo da Subscrição RHEL

> **Estado: em execução.** Registo testado em RHEL 10 real — `--auto-attach` não existe nesta versão (ver comando corrigido abaixo). Falta confirmar o resultado do `status`/`repolist` para fechar a secção de teste de validação.

---

## O que é

O `subscription-manager` é a ferramenta de linha de comandos que liga uma instalação RHEL a uma **subscrição Red Hat** — o mecanismo que dá acesso aos repositórios oficiais de pacotes (`dnf`), atualizações de segurança e suporte. Sem registo, o `dnf` não tem repositórios configurados e não instala nem atualiza pacotes.

---

## Aplicabilidade

Obrigatório em qualquer instalação RHEL, seja um servidor de produção empresarial (subscrição paga, por socket/instância) ou uma máquina de desenvolvimento/estudo individual (subscrição gratuita **Developer**). O comportamento da ferramenta é o mesmo — muda apenas o tipo de subscrição associada à conta.

---

## Pertinência para este projeto

O Rocky Linux (usado até 2026-08-05) é um *rebuild* binário do RHEL — usa os mesmos pacotes, mas sem depender de subscrição, porque republica os fontes do RHEL sem o mecanismo de registo da Red Hat. Ao migrar para o RHEL oficial (motivado pelo estudo RHCSA, que usa este SO no exame), o `subscription-manager register` passa a ser um passo **obrigatório** logo após a instalação — sem ele, nem o `dnf update` funciona.

Alternativa avaliada: manter o Rocky Linux e só usar RHEL numa VM separada para a prática RHCSA. Rejeitada porque o objetivo é ter o mesmo SO a correr o servidor real 24/7, não uma VM de estudo isolada — reforça o hábito de operar RHEL no dia a dia.

---

## Comandos utilizados

### 1. Registar a máquina na subscrição

```bash
sudo subscription-manager register --username <utilizador-redhat>
```

- `register` — associa esta instalação à conta Red Hat Developer do Ivan
- `--username` — evita o prompt interativo de utilizador (a password é sempre pedida interativamente, nunca em texto simples na linha de comandos)
- **Sem `--auto-attach`** — testado em RHEL 10, dá erro `no such option: --auto-attach`. O RHEL 10 usa **Simple Content Access (SCA)**: a conta Red Hat Developer já não precisa de "anexar" uma subscrição/pool manualmente — o acesso aos repositórios fica disponível assim que a máquina é registada. A flag `--auto-attach` era necessária no modelo antigo (entitlement-based), já não existe como opção no `subscription-manager` atual

### 2. Verificar o estado

```bash
sudo subscription-manager status
sudo subscription-manager list --consumed
```

- `status` — resumo geral (registado/não registado, subscrição válida/expirada)
- `list --consumed` — mostra a(s) subscrição(ões) atualmente aplicadas a esta máquina

### 3. Confirmar que os repositórios ficaram ativos

```bash
sudo dnf repolist
```

- Deve mostrar os repositórios `rhel-9-for-x86_64-baseos-rpms` / `appstream-rpms` (ou equivalente da versão instalada) como ativos

---

## Teste de validação

- [x] `subscription-manager register --username <user>` (sem `--auto-attach`) executado com sucesso em RHEL 10 real
- [ ] `subscription-manager status` devolve "Overall Status: Current" — a confirmar na próxima sessão
- [ ] `dnf update -y` corre sem erro de "no repositories configured" — a confirmar na próxima sessão

---

## Comandos de referência (troubleshooting)

```bash
# Anular o registo (ex: antes de reinstalar o SO, para libertar a subscrição)
sudo subscription-manager unregister

# Ver todas as subscrições disponíveis na conta (mesmo as não aplicadas)
sudo subscription-manager list --available

# Forçar novo pedido de certificados de subscrição
sudo subscription-manager refresh
```

---

*Faz parte do projeto [Home Server + Tailscale](../../README.md)*
