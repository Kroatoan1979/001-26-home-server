# subscription-manager — Registo da Subscrição RHEL

> **Estado: planeado, ainda não executado.** Este documento é um esqueleto criado antes da reinstalação do servidor (decisão de 2026-08-05) — a preencher com os resultados reais assim que o passo for executado em RHEL.

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

## Comandos previstos

### 1. Registar a máquina na subscrição

```bash
sudo subscription-manager register --username <utilizador-redhat> --auto-attach
```

- `register` — associa esta instalação à conta Red Hat Developer do Ivan
- `--username` — evita o prompt interativo de utilizador (a password é sempre pedida interativamente, nunca em texto simples na linha de comandos)
- `--auto-attach` — a subscrição Developer aplica-se automaticamente, sem ser preciso escolher manualmente um "pool" de subscrição (relevante em contas empresariais com várias subscrições disponíveis)

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

> Por executar — atualizar esta secção com o resultado real após a reinstalação.

- [ ] `subscription-manager status` devolve "Overall Status: Current"
- [ ] `dnf update -y` corre sem erro de "no repositories configured"

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
