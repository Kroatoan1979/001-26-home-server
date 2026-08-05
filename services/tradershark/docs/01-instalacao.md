# TraderShark — Instalação

> **Estado: planeado, ainda não executado.** Este documento é um esqueleto criado antes da reinstalação do servidor (decisão de 2026-08-05) — a preencher com os passos e comandos reais assim que o pacote/formato de instalação do TraderShark for confirmado no site oficial.
>
> ⚠️ **Bloqueio (2026-08-05):** pesquisa na web não confirmou a existência de uma app Linux nativa chamada "TraderShark" para trading desportivo em modo ladder — ver nota em [../README.md](../README.md). **Este passo não pode avançar sem o link/fonte oficial fornecido pelo Ivan.**

---

## O que é

TraderShark é uma aplicação nativa Linux para trading desportivo em modo **ladder** (interface de ordens rápidas, tipicamente usada em apostas em tempo real durante eventos desportivos). É sensível a latência de renderização e de rede — o objetivo de a correr localmente no servidor é evitar a variabilidade de latência de soluções gráficas remotas pela internet.

---

## Aplicabilidade

Relevante apenas para quem faz trading desportivo ativo e precisa de uma interface ladder nativa Linux. Não é um serviço de infraestrutura do projeto (como o Jellyfin/Navidrome) — é uma aplicação de uso pessoal que aproveita o facto de o servidor já ter uma sessão gráfica GNOME disponível após a escolha de "Server with GUI" na instalação do RHEL.

---

## Pertinência para este projeto

A escolha de instalar o RHEL em modo "Server with GUI" (em vez de Minimal, que seria suficiente só para FileCloud/Tailscale) foi motivada em parte por este requisito — o TraderShark precisa de um ambiente gráfico local. Correr esta app é um dos objetivos que justificam a migração de Rocky Linux para RHEL, a par do RHCSA e do FileCloud.

---

## Passos previstos

### 1. Confirmar dependências e formato do pacote

- Consultar o site oficial do TraderShark para o formato de distribuição Linux (`.deb`, `.rpm`, AppImage, ou instalador próprio)
- Confirmar dependências gráficas necessárias (ex: bibliotecas Qt/GTK, se aplicável)

### 2. Instalar

```bash
# A preencher — depende do formato real do pacote confirmado no passo anterior
# Exemplos possíveis, a validar:
# sudo dnf install ./tradershark-x.y.z.rpm
# ou, se for AppImage:
# chmod +x TraderShark-x.y.z.AppImage
```

### 3. Testar localmente

- Abrir sessão gráfica local no servidor (tampa aberta, GNOME)
- Correr o TraderShark e confirmar que a interface responde sem lag percetível

---

## Teste de validação

> Por executar — atualizar esta secção com o resultado real.

- [ ] TraderShark instala sem erros de dependências
- [ ] App abre e responde localmente sem lag percetível
- [ ] App testada via X11 forwarding a partir do Windows (ver [infra/docs/06-acesso-grafico-x11.md](../../../infra/docs/06-acesso-grafico-x11.md)) — avaliar se a latência adicional é aceitável

---

## Próximo passo

→ [infra/docs/06-acesso-grafico-x11.md](../../../infra/docs/06-acesso-grafico-x11.md) — se for preciso aceder à app remotamente a partir do Windows

---

*Faz parte do projeto [Home Server + Tailscale](../../../README.md)*
