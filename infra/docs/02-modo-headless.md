# Modo Headless — Tampa Fechada

Por omissão, o Linux suspende o sistema quando a tampa do portátil é fechada. Para operar como servidor headless, é necessário desativar esse comportamento.

---

## 1. Configurar `systemd-logind`

```bash
sudo vim /etc/systemd/logind.conf
```

Adicionar ou descomentar estas linhas:

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Aplicar sem reiniciar:

```bash
sudo systemctl restart systemd-logind
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
