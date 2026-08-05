# TraderShark — Trading Desportivo (Ladder)

App nativa Linux de trading desportivo em modo ladder. Ao contrário do Jellyfin/Navidrome, **não corre em Docker** — é uma aplicação gráfica instalada diretamente no sistema, pensada para uso local/LAN devido à sensibilidade a latência.

> **Motivo de estar neste projeto:** um dos objetivos da migração para RHEL (decisão de 2026-08-05) é precisamente ter sessão gráfica local (GNOME) para correr esta app.
>
> ⚠️ **Bloqueio identificado em 2026-08-05:** pesquisa não encontrou nenhuma aplicação Linux nativa chamada "TraderShark" com pacote de instalação disponível. O único resultado (`tradershark.com`) é um serviço de formação/sala de trading para futuros (S&P Emini, Euro, Crude, Gold) — sem relação com trading desportivo em modo ladder, e sem download de app. **Antes de avançar com esta fase, o Ivan precisa de fornecer o link/fonte oficial real da aplicação** — pode ser um nome trocado com outra ferramenta (ex: Bet Angel, Gruss Software, Cymatic Trader, comuns em trading de mercados de apostas tipo Betfair).

---

## Estado

| Componente | Estado |
|---|---|
| RHEL com GUI (GNOME) instalado | [ ] Pendente — ver [infra/README.md](../../infra/README.md) |
| Dependências/formato do pacote confirmados | [ ] Planeado |
| TraderShark instalado | [ ] Planeado |
| Teste local (sessão gráfica do servidor) | [ ] Planeado |
| Teste via X11 forwarding (Windows) | [ ] Planeado |

---

## Documentação

| Documento | Descrição |
|---|---|
| [01-instalacao.md](docs/01-instalacao.md) | Confirmar dependências e instalar o TraderShark |

---

## Acesso

| Contexto | Como |
|---|---|
| Local (recomendado) | Sessão gráfica direta no servidor, tampa aberta |
| LAN (ethernet) | X11 forwarding — ver [infra/docs/06-acesso-grafico-x11.md](../../infra/docs/06-acesso-grafico-x11.md) |
| Fora de casa | **Não recomendado** — app sensível a latência (modo ladder); evitar X11/xrdp pela internet durante trading ativo |

---

## Requisitos

- RHEL instalado em modo "Server with GUI" (GNOME) — não Minimal
- Dependências gráficas/formato do pacote a confirmar no site oficial do TraderShark antes da instalação

---

## Pré-requisito

→ [infra/README.md](../../infra/README.md) — infraestrutura base operacional, incluindo GUI

---

*Faz parte do projeto [Home Server + Tailscale](../../README.md)*
