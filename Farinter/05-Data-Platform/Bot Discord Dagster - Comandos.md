---
tags: [bot, discord, dagster, comandos, referencia]
fecha: 2026-04-13
estado: completado
servicio: dagster-discord-bot-gf
---

# Bot Discord Dagster — Comandos

Referencia rápida de TODOS los comandos disponibles del `Dagster Register Bot`.

## Tabla resumen

| Comando | Descripción | Permisos | Ejemplo |
|---------|-------------|----------|---------|
| `!help` | Lista todos los comandos | Cualquiera | `!help` |
| `!dagster-status` | Resumen de runs Dagster (últimos 50) | Cualquiera | `!dagster-status` |
| `!asset <nombre>` | Busca asset por nombre (fuzzy) con estado y link | Cualquiera | `!asset cliente_general` |
| `!register <email>` | Vincula tu Discord ID con tu email corporativo | Cualquiera | `!register tu.email@farinter.com` |
| `!unregister` | Elimina tu registro | Cualquiera | `!unregister` |
| `!whoami` | Muestra tu registro actual | Cualquiera | `!whoami` |
| `!list` | Lista todos los usuarios registrados | Cualquiera | `!list` |
| `!enable-here` | Habilitar el bot en el canal actual | Admin (Manage Channels) | `!enable-here` |
| `!disable-here` | Deshabilitar el bot en el canal actual | Admin (Manage Channels) | `!disable-here` |

---

## Detalle por comando

### `!help`

Muestra lista de comandos disponibles.

**Output**:
```
Comandos disponibles:
!register email@farinter.com — Registrar tu email para menciones en alertas Dagster
!unregister                  — Eliminar tu registro
!whoami                      — Ver tu registro actual
!list                        — Ver todos los registrados
```

---

### `!asset <nombre>`

Busca un asset por nombre aproximado. Admite:
- **Snake_case ↔ CamelCase**: `cliente_general` matchea `ClienteGeneral`
- **Múltiples palabras (AND)**: `ventas kielsa` → paths que contienen ambas
- **Fuzzy fallback**: typos tolerables via `difflib`

Si encuentra UN solo asset, devuelve detalle completo (última materialización, owners, link a la UI). Si encuentra varios (hasta 5), los lista y sugiere refinar. Si no encuentra nada, avisa.

**Ejemplos**:
```
!asset crm_sms
→ 📦 DL_FARINTER/mongo_db_crm_hn/crm_sms
  🟢 Última materialización: 2026-04-13 11:47:30 UTC
  🔗 Run: a5ddeada
  🔗 https://dagster.farinter.com/assets/DL_FARINTER/mongo_db_crm_hn/crm_sms

!asset ventas kielsa
→ 📦 5 assets encontrados:
  1. BI_FARINTER/dbo/BI_Hecho_ExpressVentasHist_Kielsa
  2. BI_FARINTER/dbo/BI_Hecho_ProyeccionVentas_Kielsa
  3. BI_FARINTER/dbo/BI_Hecho_ProyeccionVentaseCommerce_Kielsa
  4. BI_FARINTER/dbo/BI_Hecho_Ventas4MesesHist_Kielsa
  5. BI_FARINTER/dbo/BI_Hecho_VentasHist_Kielsa
  Refiná la búsqueda para obtener detalles.
```

Requiere `DAGSTER_UI_URL` env var para mostrar links directos a la UI (opcional).

---

### `!dagster-status`

Consulta el GraphQL de Dagster (`http://webserver:3000/graphql` internamente) y devuelve un resumen de los últimos 50 runs.

**Output ejemplo**:
```
📊 Dagster Status
🟢 Exitosos: 47 (últimos 50 runs)
🔴 Fallidos: 0
🔵 Corriendo: 3
⏳ En cola: 0

❌ Últimos fallos:
  __ASSET_JOB — run 95d05b13
  __ASSET_JOB — run cc7c2919
  ...
```

**Nota importante**: el bot apunta al GraphQL de **prd** (`http://webserver:3000/graphql`). Si lo agregás a un canal de qa, va a mostrar stats de prd igual.

---

### `!register <email>`

Registra tu Discord ID asociado a tu email corporativo en `discord_user_mapping` (nexus). Esto permite que cuando un asset del que sos owner falle, recibas mención en el embed rojo.

**Validaciones**:
- Email debe ser de dominio `farinter.com`, `farinter.hn` o `kielsa.hn`
- Si ya estabas registrado, sobreescribe (`ON CONFLICT DO UPDATE`)

**Ejemplo**:
```
Daniel: !register dbarrientos@kielsa.hn
Bot: ✅ Registrado: dbarrientos@kielsa.hn → @Daniel Banariba
```

**Validación fallida**:
```
Daniel: !register banariba@gmail.com
Bot: ❌ Solo se permiten correos corporativos (farinter.com, farinter.hn, kielsa.hn).
```

---

### `!unregister`

Elimina tu registro de la tabla.

**Output**:
```
✅ Eliminado registro de tu_email@farinter.com
```

---

### `!whoami`

Muestra a qué email está vinculado tu Discord ID actualmente.

**Output ejemplo**:
```
👤 Tu registro: dbarrientos@kielsa.hn
```

---

### `!list`

Lista todos los usuarios registrados en el server actual.

**Output ejemplo**:
```
Usuarios registrados (2):
- @Daniel Banariba → dbarrientos@kielsa.hn
- @Edwin Martínez → emmartinez@farinter.hn
```

---

### `!enable-here` (admin)

Habilita el canal actual para que el bot responda a comandos. Persiste en `bot_allowed_channels` (nexus). **Cero restart del container**.

**Requiere**: el usuario debe tener permiso `Manage Channels` en el server.

**Output**:
```
✅ Canal habilitado para comandos del bot. Probá !dagster-status.
```

Si el canal ya estaba habilitado:
```
ℹ️ Este canal ya estaba habilitado.
```

Si el usuario no tiene permisos:
```
❌ Solo usuarios con permiso 'Manage Channels' pueden habilitar canales.
```

---

### `!disable-here` (admin)

Inversa de `!enable-here`. Quita el canal actual de la allow-list.

**Requiere**: permiso `Manage Channels`.

**Output**:
```
✅ Canal deshabilitado. El bot ya no responderá acá.
```

---

## Funcionalidad NO accesible vía comando (automática)

### Embed rojo de fallo (`Run Failure`)

Cuando un run de Dagster falla, el sensor `failed_asset_notification_sensor` envía un embed **rojo** al canal de alertas (vía webhook `DAGSTER_DISCORD_WEBHOOK_URL`). Incluye:

- Job name + run ID
- Inicio / Fin del run
- Assets fallidos
- Downstream afectados
- **Owners** (con menciones `@usuario` si están registrados via `!register`)
- Error message

### Embed verde de recovery (`Asset Recovered`)

Cuando un asset que estaba en estado de fallo se materializa exitosamente, el sensor `asset_recovery_notification_sensor` (corre cada 5 min) detecta el recovery y envía embed **verde**. Bug fixeado en [[Discord Bot Dagster - Fixes 2026-04-13]]; antes nunca disparaba por bug silencioso de SecretEnvVar.

**Post-[[Discord Bot Dagster - Mejoras 2026-04-14]]**: el embed verde solo llega si el asset **previamente recibió el embed rojo**. Si se recuperó antes del deadline (2hs por default), no llega ni rojo ni verde (fallo transitorio).

### Late alert flow (freshness-aware)

Desde el merge del PR #138, el sensor de fallos NO manda embed inmediato. Encola en la tabla `pending_failure_alerts`, y un nuevo `late_failure_alert_sensor` evalúa cada 5 min:

- Si el asset se recuperó antes del deadline (default 120 min, configurable via `LATE_ALERT_DEFAULT_MINUTES`) → cleanup silencioso
- Si el deadline pasó y no se recuperó → manda email + embed rojo

Esto elimina el ruido de fallos transitorios que se resuelven solos con auto-retry.

**Para ajustar el deadline** en un entorno específico: editar `.env` del server con `LATE_ALERT_DEFAULT_MINUTES=60` (o el valor que corresponda).

---

## Configuración del bot

### Variables de entorno requeridas (en `.env` del server)

```bash
DISCORD_BOT_TOKEN=<token del bot, de Discord Developer Portal>
DISCORD_ALLOWED_CHANNEL_IDS=<ids separados por coma>
DAGSTER_GRAPHQL_URL=http://webserver:3000/graphql
DAGSTER_POSTGRES_NEXUS_HOST=...
DAGSTER_POSTGRES_NEXUS_PORT=5432
DAGSTER_POSTGRES_NEXUS_DB=dwh_nexus
DAGSTER_POSTGRES_NEXUS_USERNAME=...
DAGSTER_POSTGRES_NEXUS_PASSWORD=...
DAGSTER_DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
```

### Allow-list de canales — DB-driven (self-service)

A partir de la implementación de `!enable-here` / `!disable-here`, **la fuente de verdad es la tabla `bot_allowed_channels` en nexus**, NO el env var. Para agregar un canal:

1. Ir al canal en Discord (debe ser admin con permiso Manage Channels)
2. Tipear `!enable-here`
3. Listo — sin restart, sin SSH, sin tocar env vars

El env var `DISCORD_ALLOWED_CHANNEL_IDS` se mantiene como **seed inicial**: si la tabla está vacía al arrancar el bot y el env var tiene valores, se hace una migración one-shot.

### Permisos Discord requeridos por canal

El bot necesita en cada canal donde queremos que responda:
- ✅ View Channel
- ✅ Send Messages
- ✅ Read Message History

Si el canal tiene override per-channel restrictivo (como `#alerts-dagster` o `#alerts-dagster-qa` en Farinter), no alcanza con el rol global — hay que agregar el override del bot en ese canal específico.

---

## Server / canales actuales

| Server | Canal | ID | Estado |
|--------|-------|----|----|
| Farinter (prod) | `#general` | 1063461413244379299 | ✅ Bot responde |
| Farinter (prod) | `#alerts-dagster` | 1491551583278993529 | ✅ Bot responde |
| Farinter (prod) | `#alerts-dagster-qa` | 1491895486309728488 | ⚠️ Bot responde pero stats de prd, no qa |
| Dagster Dev Testing | `#general` | 1491846356354994429 | ✅ Bot responde (también con stats de prd) |

---

## Troubleshooting rápido

| Síntoma | Causa probable | Fix |
|---------|---------------|-----|
| Bot no responde en canal X | Canal no está en allow-list | Agregar id a `DISCORD_ALLOWED_CHANNEL_IDS` y recreate container |
| Bot no responde en canal X (allow-list ok) | Falta permiso "View Channel" en ese canal | Pedirle a admin que agregue rol del bot en Edit Channel → Permissions |
| `❌ Error: no existe la relación «public.discord_user_mapping»` | Tabla no creada en ese nexus | Reiniciar el bot: `_ensure_nexus_schema()` la crea automáticamente |
| Embed verde de recovery nunca llega | Tabla no existe O bug SecretEnvVar | Ver [[SecretEnvVar gotcha en Dagster sensors]] |
| Bot crash loop con `ModuleNotFoundError: discord` | Dockerfile usa `python` en vez del venv | Verificar que CMD use `uv run --no-sync python` |
| Login: `Improper token has been passed` | `DISCORD_BOT_TOKEN` mal seteado o vencido | Reset Token en Developer Portal y actualizar `.env` |

---

## Referencias

- [[Discord Bot Dagster - Fixes 2026-04-13]] — historia completa de los bugs y fixes
- [[Bot Discord no responde]] — runbook de diagnóstico
- [[SecretEnvVar gotcha en Dagster sensors]] — aprendizaje técnico
- Código bot: `dagster-discord-bot-gf/dagster_discord_bot_gf/bot.py`
- Código sensores: `dagster-shared-gf/dagster_shared_gf/shared_failed_sensors.py`
