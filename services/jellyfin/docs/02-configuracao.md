# Jellyfin — Configuração Inicial

Após a instalação, o assistente web guia pela configuração básica. Este documento regista as escolhas feitas e como adicionar bibliotecas e clientes.

---

## 1. Assistente de configuração inicial

Aceder a `http://192.168.1.2:8096` pela primeira vez.

### Passos do assistente

1. **Idioma:** Português (ou English)
2. **Criar utilizador admin**
   - Username: `ivan`
   - Password: (usar gestor de passwords)
3. **Adicionar biblioteca de media** — ver secção abaixo
4. **Preferências de metadados:** manter padrão (The Movie Database + TheTVDB)
5. **Confirmar configuração**

---

## 2. Adicionar bibliotecas

### Filmes

| Campo | Valor |
|---|---|
| Tipo de conteúdo | Filmes |
| Nome da biblioteca | Filmes |
| Pasta | `/media/filmes` (dentro do container = `/mnt/dados/filmes` no host) |
| Idioma preferido | Português |

### Séries

| Campo | Valor |
|---|---|
| Tipo de conteúdo | Séries |
| Nome da biblioteca | Séries |
| Pasta | `/media/series` |

---

## 3. Organização dos ficheiros de media

O Jellyfin segue convenções específicas para identificar automaticamente metadados.

### Filmes

```
/mnt/dados/filmes/
└── Titulo do Filme (ano)/
    └── Titulo do Filme (ano).mkv
```

Exemplo:
```
/mnt/dados/filmes/
├── Inception (2010)/
│   └── Inception (2010).mkv
└── The Dark Knight (2008)/
    └── The Dark Knight (2008).mkv
```

### Séries

```
/mnt/dados/series/
└── Nome da Serie/
    └── Season 01/
        ├── Nome da Serie - S01E01.mkv
        └── Nome da Serie - S01E02.mkv
```

---

## 4. Clientes recomendados

| Plataforma | App |
|---|---|
| Smart TV | App Jellyfin nativa (se disponível) ou browser |
| Android | Jellyfin for Android (Google Play) |
| iOS | Infuse ou Jellyfin for iOS |
| Windows | Jellyfin Media Player ou browser |
| Browser | Qualquer — `http://IP:8096` |

### Configurar cliente

- **Servidor:** `http://192.168.1.2:8096` (LAN) ou `http://100.x.x.x:8096` (Tailscale)
- **Username:** `ivan`
- **Password:** (a definida no passo de configuração)

---

## 5. Transcodificação de hardware (opcional — fase 2)

O i7 de 4ª/5ª geração suporta Intel Quick Sync para transcodificação acelerada. Configurar em:

`Dashboard → Reprodução → Transcodificação → Aceleração de Hardware → Intel QuickSync`

> Pode requerer configuração adicional do container Docker para acesso ao `/dev/dri`.
