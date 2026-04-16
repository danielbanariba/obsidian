---
tags: [runbook, discord, bot, dagster, troubleshooting]
sistema: dagster-discord-bot-gf
severidad: media
ultima_ocurrencia: 2026-04-13
---

# Bot Discord no responde

## Síntomas

- Tipeás `!dagster-status`, `!register`, `!list` o cualquier comando del bot y NO hay respuesta
- O el bot postea un error rojo tipo "no existe la relación..."

## Diagnóstico — orden de chequeo

### 1. Container del bot está corriendo

**En prd**:
```bash
ssh analiticasu@172.16.2.220 'docker ps --format "{{.Names}} {{.Status}}" | grep discord'
```

Esperado: `main-dagster-prd-discord-bot-1   Up X minutes`

Si dice `Restarting (1)` → ir a paso 2.

### 2. Logs del bot — ver si crashea

```bash
ssh analiticasu@172.16.2.220 'docker logs --tail 30 main-dagster-prd-discord-bot-1'
```

**Errores comunes**:

| Error | Causa | Fix |
|-------|-------|-----|
| `ModuleNotFoundError: No module named 'discord'` | Dockerfile bug — CMD usa `python` en vez de `uv run` | Ver [[Discord Bot Dagster - Fixes 2026-04-13]] |
| `discord.errors.LoginFailure: Improper token has been passed` | `DISCORD_BOT_TOKEN` mal seteado | Reset Token en Discord Developer Portal y actualizar `.env` |
| `psycopg2.OperationalError: could not connect to server` | Nexus postgres down o env vars mal | Verificar env vars `DAGSTER_POSTGRES_NEXUS_*` |

### 3. Bot está logueado al Gateway

En logs deberías ver:
```
INFO    discord.gateway Shard ID None has connected to Gateway
```

Si no aparece después de varios segundos → token vencido o rate-limit.

### 4. Channel está en allow-list

```bash
ssh analiticasu@172.16.2.220 'grep DISCORD_ALLOWED_CHANNEL_IDS /opt/main-dagster/.env'
```

Si el channel ID donde tipeás no está → agregalo separado por coma:

```bash
ssh analiticasu@172.16.2.220 \
  "sed -i 's/^DISCORD_ALLOWED_CHANNEL_IDS=.*/DISCORD_ALLOWED_CHANNEL_IDS=<ids,separados,por,coma>/' /opt/main-dagster/.env"

ssh analiticasu@172.16.2.220 \
  "cd /opt/main-dagster && docker compose -f docker/docker-compose.base.yaml -f docker/docker-compose.prd.yaml --env-file .env up -d --force-recreate discord-bot"
```

⚠️ Esta fricción debería desaparecer cuando se implemente `!enable-here` (ver [[Discord Bot Dagster - Fixes 2026-04-13]]).

### 5. Bot tiene permisos en el canal de Discord

Verificar via API con el token del bot:

```bash
curl -s -H "Authorization: Bot <TOKEN>" -H 'User-Agent: DiscordBot/1.0' \
  "https://discord.com/api/v10/channels/<CHANNEL_ID>" | python3 -m json.tool
```

- Si devuelve `{"message": "Missing Access", "code": 50001}` → el bot no tiene permiso de "View Channel" en ese canal
- Fix: pedir a un admin del server que vaya a "Edit Channel → Permissions" y agregue el rol `Dagster Register Bot` con View Channel + Send Messages + Read Message History

### 6. Tabla en nexus existe

```bash
ssh analiticasu@172.16.2.220 'docker exec main-dagster-prd-code-location-kielsa-1 \
  /opt/main-dagster/dagster-kielsa-gf/.venv/bin/python -c "
import os, psycopg2
conn = psycopg2.connect(host=os.getenv(\"DAGSTER_POSTGRES_NEXUS_HOST\"), port=5432,
  dbname=os.getenv(\"DAGSTER_POSTGRES_NEXUS_DB\"),
  user=os.getenv(\"DAGSTER_POSTGRES_NEXUS_USERNAME\"),
  password=os.getenv(\"DAGSTER_POSTGRES_NEXUS_PASSWORD\"))
cur=conn.cursor()
cur.execute(\"SELECT table_name FROM information_schema.tables WHERE table_schema=\\\"public\\\"\")
for (t,) in cur.fetchall(): print(t)
"'
```

Debería listar `discord_user_mapping` y `asset_failure_state`. Si faltan, restart del bot las recrea (`_ensure_nexus_schema()` es self-bootstrap).

## Resolución rápida

Si todo lo demás falla, **rebuild + recreate**:

```bash
ssh analiticasu@172.16.2.220 \
  "cd /opt/main-dagster && docker compose -f docker/docker-compose.base.yaml -f docker/docker-compose.prd.yaml --env-file .env up -d --build --force-recreate discord-bot"
```

## Prevención

- Agregar healthcheck al container `discord-bot` en compose para que crash loops sean detectables
- Implementar `!enable-here` para evitar el restart por cambio de allow-list
- Considerar `Container Status` alert via webhook si el bot se cae > 5 min

## Historial

| Fecha | Qué pasó | Quién lo resolvió |
|-------|----------|-------------------|
| 2026-04-13 | Bot crash loop por bug Dockerfile (`python` vs `uv run`) hace 2 días sin que nadie lo notara | Daniel + sesión interna, ver [[Discord Bot Dagster - Fixes 2026-04-13]] |
| 2026-04-13 | `!list` retorna error de tabla inexistente en prd | Crear tabla manual + agregar `_ensure_nexus_schema()` self-bootstrap |
| 2026-04-13 | Bot no responde en `#alerts-dagster` (Farinter) | Axell agregó View Channel al rol global del bot |
| 2026-04-13 | Bot no responde en `#alerts-dagster-qa` (Farinter) | Axell agregó override per-channel + agregar id al allow-list |

## Referencias

- [[Discord Bot Dagster - Fixes 2026-04-13]]
- [[Bot Discord Dagster - Comandos]]
- [[SecretEnvVar gotcha en Dagster sensors]]
