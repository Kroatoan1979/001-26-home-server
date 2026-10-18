# Modo Headless — Tampa Fechada

Por omissão, o Linux suspende o sistema quando a tampa do portátil é fechada. Para operar como servidor headless, é necessário desativar esse comportamento.

> **Exceção — TraderShark (caiu a 2026-10-18, quando o TraderShark saiu do plano; texto histórico):** o servidor passa a ser "headless por padrão, exceto durante o uso do TraderShark". Quando for preciso usar a app (sensível a latência, recomendado local/LAN), a tampa é aberta e usa-se a sessão gráfica local (GNOME) normalmente — desativar os targets de suspensão abaixo não impede o uso normal do ecrã com a tampa aberta, só impede a suspensão automática ao fechá-la.

---

## 1. Configurar `systemd-logind`

> ⚠️ **Corrigido a 2026-10-12.** A versão anterior deste passo mandava editar `/etc/systemd/logind.conf` e "adicionar ou descomentar" as linhas. No RHEL 10 esse ficheiro já não vem instalado, por isso o `vim` criou um ficheiro novo **sem a secção `[Login]`**, e o systemd ignorou as diretivas. Resultado: o `logind` passou dois meses em loop a tentar suspender. Diagnóstico e explicação completa em [09-logind-journald.md](09-logind-journald.md).

Criar um drop-in (com a secção `[Login]`, obrigatória):

```bash
sudo mkdir -p /etc/systemd/logind.conf.d
sudo vim /etc/systemd/logind.conf.d/10-homeserver.conf
```

```ini
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
HandleSuspendKey=ignore
HandleHibernateKey=ignore
```

Aplicar sem reiniciar e **confirmar o valor real em vigor** (não basta testar a tampa: a máscara do passo 2 esconde um `logind.conf` partido):

```bash
sudo systemctl reload systemd-logind
busctl get-property org.freedesktop.login1 /org/freedesktop/login1 org.freedesktop.login1.Manager HandleLidSwitch
# Deve mostrar: s "ignore"
```

---

## 2. Desativar targets de suspensão

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

Verificar:

```bash
systemctl status sleep.target
# Deve mostrar: masked
```

---

## 3. Fechar a tampa e testar

1. Fechar a tampa do ASUS X555L
2. Aguardar 30 segundos
3. Tentar SSH a partir de outro dispositivo na mesma rede:

```bash
ssh ivan@192.168.1.2
```

Se a ligação for bem sucedida, o modo headless está operacional.

---

## 4. Monitorizar temperatura (primeiras horas)

Com a tampa fechada, a ventilação pode ficar comprometida dependendo da posição. Monitorizar nas primeiras horas:

```bash
# Instalar lm_sensors se não instalado
sudo dnf install -y lm_sensors
sudo sensors-detect --auto

# Monitorização em tempo real (atualiza a cada 2s)
watch -n 2 sensors
```

Valores de referência normais para o i7 em idle:
- **CPU:** 35–55 °C
- **Alerta:** acima de 80 °C consistentemente

---

## Próximo passo

→ [03-ssh.md](03-ssh.md) — configurar acesso SSH seguro e autenticação por chave
