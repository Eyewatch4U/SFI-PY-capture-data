# SFIN-PY-capture-data

Capturador de snapshots de Meltwater para el **Monitor Sistema Financiero · Paraguay**
(NSGroup / Eyewatch Intelligence Unit). Clon estándar del flujo BROU / Temas PY / Bancos PY.

```
cron-job.org (cada 30 min)
  → GitHub Actions (capture_meltwater.py baja y valida el .gz)
  → webhook de Make (SFIN-PY-DATA)
  → HTTP POST /update-url {"url":"...","sync":true} + x-sync-token
  → Cloudflare Worker (monitorsfinpy) → KV (SFINMONITOR, key latest_sfin_py)
  → Dashboard /monitor-sfin
```

## Secrets del repo (Settings → Secrets and variables → Actions)

| Secret | Valor |
|---|---|
| `MELTWATER_URL` | URL pública del snapshot del reporte "Monitoreo Sistema financiero - FBP" |
| `MELTWATER_PASSWORD` | Contraseña del reporte público (o vacío si no hay gate) |
| `MAKE_WEBHOOK_URL` | Webhook `SFIN-PY-DATA` de Make |

## Disparo

`cron-job.org` hace `POST` cada 30 min a:
`https://api.github.com/repos/Eyewatch4U/SFIN-PY-capture-data/actions/workflows/meltwater.yml/dispatches`
con header `Authorization: Bearer <PAT Actions RW>` y body `{"ref":"main"}`.

El workflow corre en `ubuntu-latest`. Si Meltwater devuelve 502 persistente (bloqueo de IP de
datacenter), pasar a self-hosted runner o setear `PROXY_URL` residencial.
