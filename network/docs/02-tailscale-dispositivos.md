# Tailscale — Adicionar Dispositivos à Tailnet

Cada dispositivo precisa de instalar o cliente Tailscale e autenticar com a mesma conta (`ivan.almeida@digitalltogether.com`).

---

## Windows (DAT-WS-IA-01, ZenBook)

1. Descarregar o cliente: [tailscale.com/download/windows](https://tailscale.com/download/windows)
2. Instalar e iniciar
3. Clicar em **Sign in** → autenticar com Google
4. O dispositivo aparece automaticamente no painel admin

Verificar no painel: `https://login.tailscale.com/admin/machines`

---

## Android (Telemóvel pessoal)

- **Android:** [Google Play — Tailscale](https://play.google.com/store/apps/details?id=com.tailscale.ipn.android)

1. Instalar a app
2. Tocar em **Sign in** → autenticar com Google
3. Ativar a VPN quando solicitado

---

## Verificar a tailnet completa

No servidor:

```bash
sudo tailscale status
```

Exemplo de output esperado:

```
100.x.x.x  homeserver     linux   -
100.x.x.x  dat-ws-ia-01   windows active; relay "..."
100.x.x.x  zenbook-ux482  windows active; relay "..."
100.x.x.x  telemovel      android active; relay "..."
```

---

## Testar conectividade entre dispositivos

### Windows → Servidor (SSH)
```powershell
ssh ivan@100.x.x.x
```

### Telemóvel → Servidor (SSH)
- App recomendada: **Termius** ou **JuiceSSH**
- Host: IP Tailscale do servidor
- Porta: 22
- Utilizador: ivan

### Telemóvel → FileCloud (após instalação do serviço)
- Browser (portal de utilizador FileCloud)
- URL: `http://100.x.x.x` (porta 80) ou `https://100.x.x.x` (porta 443)

---

## Painel de administração

Gerir dispositivos, revogar acessos, ver IPs e estado de ligação:

`https://login.tailscale.com/admin/machines`

---

## Atualizar IPs no repositório

Após todos os dispositivos estarem na tailnet, atualizar a tabela de IPs em:
- [../README.md](../README.md) — secção "IPs Tailscale"
- [../../README.md](../../README.md) — secção "Topologia de Rede"
