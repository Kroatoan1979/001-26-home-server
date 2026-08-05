# xrdp — Acesso Gráfico Remoto (Tablet Android)

> **Estado: planeado, ainda não executado.** Este documento é um esqueleto criado antes da reinstalação do servidor (decisão de 2026-08-05) — a preencher com os resultados reais assim que o passo for executado.

---

## O que é

**xrdp** é uma implementação open-source do protocolo **RDP** (Remote Desktop Protocol, criado pela Microsoft) para Linux. Ao contrário do X11 forwarding (que encaminha só uma aplicação), o xrdp expõe uma **sessão de ambiente de trabalho completa** do servidor — o cliente vê o ecrã inteiro do GNOME, como se estivesse fisicamente à frente do portátil.

---

## Aplicabilidade

Útil quando o cliente não tem X server nem suporte nativo a X11 forwarding, mas tem um cliente RDP disponível — é exatamente o caso do Android, que não tem X server, mas tem a app oficial **Microsoft Remote Desktop**. Também é a opção mais simples quando se quer o ambiente de trabalho inteiro, não só uma janela de app.

---

## Pertinência para este projeto

O tablet Android não consegue correr um X server (ao contrário do Windows, que tem o VcXsrv como opção). O xrdp resolve o acesso gráfico remoto a partir do tablet usando um protocolo que o Android já suporta nativamente via app — não é preciso nenhum software adicional do lado do cliente além da app Microsoft Remote Desktop, já usada pelo Ivan para RDP a máquinas Windows.

Alternativa avaliada: VNC (ex: TigerVNC). Rejeitada porque o RDP já é o protocolo usado no resto do projeto para acesso gráfico a Windows (`DAT-WS-IA-01`), e o cliente Android já está instalado e configurado — reutilizar o mesmo protocolo e app evita introduzir uma ferramenta nova só para este caso.

---

## Comandos previstos

### 1. Instalar o xrdp

```bash
sudo dnf install -y xrdp
sudo systemctl enable --now xrdp
```

- `xrdp` — pacote com o serviço RDP para Linux (disponível nos repositórios EPEL — pode ser necessário ativar o repositório EPEL primeiro, ver comandos de referência)
- `enable --now` — ativa o arranque automático e arranca o serviço já

### 2. Abrir a porta na firewall

```bash
sudo firewall-cmd --permanent --add-port=3389/tcp
sudo firewall-cmd --reload
```

- `--add-port=3389/tcp` — o xrdp não tem um serviço pré-definido no `firewalld` como o SSH ou o mDNS, por isso a porta é aberta diretamente pelo número (3389 é a porta padrão do RDP)

### 3. Verificar o estado do serviço

```bash
sudo systemctl status xrdp --no-pager
```

### 4. Ligar a partir do tablet Android

- Abrir a app **Microsoft Remote Desktop**
- Adicionar PC: `homeserver` (via MagicDNS, com Tailscale ativo) ou o IP Tailscale direto
- Utilizador: `ivan` / password da conta local

---

## Teste de validação

> Por executar — atualizar esta secção com o resultado real.

- [ ] Ligação RDP bem-sucedida a partir do tablet Android via Tailscale
- [ ] Sessão gráfica GNOME completa visível e utilizável no tablet
- [ ] Confirmar se a sessão xrdp interfere com uma sessão gráfica local já aberta (GNOME normalmente só permite uma sessão ativa por utilizador — avaliar se é preciso um utilizador dedicado para xrdp)

---

## Comandos de referência (troubleshooting)

```bash
# Ativar o repositório EPEL, se o xrdp não estiver disponível nos repositórios base
sudo dnf install -y epel-release

# Ver logs do xrdp em caso de falha de ligação
sudo journalctl -u xrdp -e

# Confirmar que a porta está mesmo a aceitar ligações
ss -tlnp | grep 3389
```

---

*Faz parte do projeto [Home Server + Tailscale](../../README.md)*
