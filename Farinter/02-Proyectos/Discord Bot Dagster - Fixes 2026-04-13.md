---
tags: [proyecto, discord, dagster, bot, prd, completado]
fecha: 2026-04-13
estado: completado
solicitado_por: Daniel Banariba (sesión interna)
prioridad: alta
pr: "https://github.com/grupo-farinter/main-dagster/pull/136"
---

# Discord Bot Dagster — Fixes 2026-04-13

## Contexto

Trigger inicial: usuario pidió verificar si el comando `!dagster-status` funcionaba. Investigación reveló que el bot estaba en crash loop hace 2 días en prd, y descubrimos en cascada 3 bugs reales del PR #134 ([[Discord Status Recovery PR134]]) que nadie había detectado:

1. Bug en Dockerfile del bot (CMD usaba `python` del sistema en vez del venv)
2. Bug silencioso en `_get_nexus_conn()` (pasaba SecretEnvVar a psycopg2)
3. Tablas de nexus nunca creadas en prd (solo en dev y local)

## Objetivo

Dejar el bot operativo en prd con `!dagster-status`, `!register`, etc., y arreglar el ciclo completo de failure → recovery alerts (embed verde "Asset Recovered").

## Tareas

- [x] Diagnóstico bot prd crash loop
- [x] Fix Dockerfile (`uv run --no-sync python` en CMD)
- [x] Diagnóstico recovery sensor — root cause SecretEnvVar
- [x] Fix `_get_nexus_conn` con `os.getenv` directo
- [x] Reemplazar `except Exception: pass` por `logger.warning`
- [x] Crear tablas `discord_user_mapping` y `asset_failure_state` en prd nexus
- [x] Self-bootstrap `_ensure_nexus_schema()` en bot y sensor
- [x] Configurar `DISCORD_BOT_TOKEN` y `DISCORD_ALLOWED_CHANNEL_IDS` en prd `.env`
- [x] Deploy directo en prd via scp + rebuild (PR aún no merged)
- [x] PR #136 abierto a `dev`
- [x] Verificación end-to-end: bot responde en `#general`, `#alerts-dagster`, `#alerts-dagster-qa`, testing
- [x] Implementar self-service `!enable-here` / `!disable-here` (in-band, sin restart)
- [x] Seed automático de canales del env var a la tabla nueva
- [ ] Merge PR #136 → main → CI deploy
- [ ] `git checkout main && git pull` en prd para limpiar dirty state

## Decisiones tomadas

### Por qué NO usar framework de migrations versionadas

Al inicio implementé un sistema de migrations (`scripts/sql/nexus/`, script aplicador, target Makefile). **Axell hizo bien en señalar que era over-engineering** para 2 tablas de un servicio chico.

**Decisión final**: self-bootstrap idempotente con `CREATE TABLE IF NOT EXISTS` al startup del bot/sensor. Beneficios:

- Cero pasos manuales de deploy
- Self-healing si las tablas se borran
- Menos código que mantener
- Acorde al tamaño del servicio (KISS / YAGNI)

### Por qué patch directo en prd (no esperar al PR merge)

Usuario quería demostrar funcionalidad el mismo día a sus compañeros (Edwin, Axell, agchavez). Trade-off: dirty state temporal en `/opt/main-dagster` hasta merge oficial. Compensado con commit del fix en branch `fix/discord-bot-dockerfile-nexus-conn` para que cuando merge a main, `git pull` en prd limpie el state.

### Por qué `os.getenv` directo en vez de `DagsterSettings.dagster_postgres_nexus_password`

`SecretEnvVar` (tipo retornado por `DagsterSettings`) **no se puede pasar a `psycopg2.connect`** — Dagster tira `RuntimeError("SecretEnvVar defers resolution... use os.getenv directly")`. El recovery sensor (líneas 752-756) ya tenía el patrón correcto, solo había que aplicarlo a `_get_nexus_conn`. Ver [[SecretEnvVar gotcha en Dagster sensors]].

## Archivos modificados

- `dagster-discord-bot-gf/Dockerfile` — CMD usa venv via `uv run --no-sync`
- `dagster-discord-bot-gf/dagster_discord_bot_gf/bot.py` — `ensure_nexus_schema()` + call en `main()`
- `dagster-shared-gf/dagster_shared_gf/shared_failed_sensors.py` — fix `_get_nexus_conn`, `_ensure_nexus_schema`, logger en lugar de swallow

## Lo que aprendí

Ver notas dedicadas:
- [[SecretEnvVar gotcha en Dagster sensors]]
- [[Discord Bot - Permisos por canal vs rol global]]

## Referencias

- PR #136: https://github.com/grupo-farinter/main-dagster/pull/136
- PR #134 original (origen de los bugs): https://github.com/grupo-farinter/main-dagster/pull/134
- [[Bot Discord Dagster - Comandos]] (referencia rápida de comandos)
- [[Bot Discord no responde]] (runbook)
- Conversación Discord en `#alerts-dagster` (Farinter server)

## Estado de prd post-deploy

| Componente                                                         | Estado                                       |
| ------------------------------------------------------------------ | -------------------------------------------- |
| Bot logueado al Gateway                                            | ✅                                            |
| Bot operativo en `#general` (Farinter)                             | ✅                                            |
| Bot operativo en `#alerts-dagster` (Farinter)                      | ✅                                            |
| Bot operativo en `#alerts-dagster-qa` (Farinter)                   | ⚠️ Funciona pero muestra stats de prd, no qa |
| Bot operativo en testing server                                    | ✅                                            |
| Tablas `discord_user_mapping` + `asset_failure_state` en prd nexus | ✅                                            |
| Recovery sensor escribe en nexus correctamente                     | ✅                                            |
| Embed verde de recovery (próxima validación E2E)                   | ⏳ Esperando próximo failure+recovery real    |
