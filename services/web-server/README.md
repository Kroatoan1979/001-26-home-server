# Web Server

Subprojeto em planeamento. A tecnologia ainda não está definida.

---

## Opções em consideração

| Opção | Caso de uso |
|---|---|
| **Caddy** | Reverse proxy com HTTPS automático (Let's Encrypt); configuração simples |
| **Nginx** | Reverse proxy clássico; mais configuração, mais controlo |
| **Traefik** | Reverse proxy dinâmico integrado com Docker; ideal se o número de serviços crescer |

---

## Objetivos possíveis

- Reverse proxy para o FileCloud (URL amigável em vez de porta)
- Servir uma página pessoal ou dashboard
- HTTPS para acesso via domínio (requer domínio ou solução como Tailscale Funnel)

---

## Estado

Em análise — decidir tecnologia antes de avançar.

---

*Faz parte do projeto [Home Server + Tailscale](../../README.md)*
