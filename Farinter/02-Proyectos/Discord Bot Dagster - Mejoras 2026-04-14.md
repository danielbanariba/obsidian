---
tags: [proyecto, discord, dagster, bot, sensor, prd, en-progreso]
fecha: 2026-04-14
estado: en-progreso
solicitado_por: Edwin Martínez + Axell Padilla (conversación Discord)
prioridad: alta
pr: "https://github.com/grupo-farinter/main-dagster/pull/138"
proyecto_previo: "[[Discord Bot Dagster - Fixes 2026-04-13]]"
---

# Discord Bot Dagster — Mejoras 2026-04-14

Continuación de [[Discord Bot Dagster - Fixes 2026-04-13]]. Ese día cerramos los 3 bugs del PR #134. Hoy, feedback del equipo en `#alerts-dagster` generó nuevas mejoras.

## Contexto

### Feedback de Edwin

> "Como le señalo un proceso en particular? Quiero saber como va el cubo de ventas"
> "No es muy descriptivo ese último fallo, asaber que asset es"
> "para que no me llegue el mensaje a esta seccion de alerta es porque si lo intento y no pudo correr…ajusten eso porfa"

### Feedback de Axell

> "mira si podés usar el api de búsqueda, poniendo búsqueda aproximada, si no nos servirá mucho, te devolverá una lista de parecidos"
> "la política de freshness se cumpla significa que no me interesa el error a menos que también esté incumpliendo la política"

## Objetivo

Expandir el bot con un comando de query por asset (fuzzy) y refactorizar el sensor de fallos para que use freshness-aware alerts (no mandar alertas por fallos transitorios con retry automático exitoso).

## Tareas

- [x] Comando `!asset <nombre>` con fuzzy search + multi-palabra + CamelCase/snake_case tolerance
- [x] Reemplazar `__ASSET_JOB` genérico por nombres reales de assets en `!dagster-status`
- [x] Tabla `pending_failure_alerts` con cola de alertas (auto-bootstrap)
- [x] Columna `alerted_at` en `asset_failure_state` (auto-bootstrap via ALTER IF NOT EXISTS)
- [x] `failed_asset_notification_sensor` reescrito: encola en vez de mandar
- [x] Nuevo `late_failure_alert_sensor` que evalúa deadline + recovery
- [x] `asset_recovery_notification_sensor` respeta `alerted_at` (solo manda verde si ya se había alertado)
- [x] Tests actualizados: 20/20 pasan verificando el nuevo flow
- [x] E2E test manual con 2 escenarios (transitorio + real) — ambos OK
- [x] PR #138 abierto con scope expandido
- [ ] Merge PR #138 → `dev` → `main` → CI deploy
- [ ] Limpiar `pending_failure_alerts` en prd nexus post-deploy si quedaron artefactos
- [ ] Siguiente iteración: integrar freshness policy real via GraphQL

## Arquitectura nueva (failure → alert flow)

```
           ┌─────────────────────────────────────────────────────────┐
           │ Asset corre                                             │
           └──────────┬──────────────────────────────┬───────────────┘
                      │                              │
                   FAILURE                       SUCCESS
                      │                              │
                      ▼                              ▼
  ┌───────────────────────────────┐  ┌───────────────────────────────┐
  │ failed_asset_notification_    │  │ asset_recovery_notification_  │
  │ sensor                        │  │ sensor (cada 5 min)           │
  │                               │  │                               │
  │ Encola en                     │  │ Si asset en                   │
  │ pending_failure_alerts        │  │ asset_failure_state:          │
  │ (NO manda nada)               │  │  - alerted_at NOT NULL        │
  │                               │  │    → manda embed VERDE        │
  │ Marca asset en                │  │  - alerted_at IS NULL         │
  │ asset_failure_state           │  │    → cleanup silencioso       │
  └───────────────────────────────┘  │  (transient sin alertar)      │
                      │              └───────────────────────────────┘
                      │
                      ▼
  ┌───────────────────────────────────────────────────┐
  │ late_failure_alert_sensor (cada 5 min)            │
  │                                                   │
  │ Lee pending_failure_alerts WHERE alerted_at NULL  │
  │                                                   │
  │ Para cada row:                                    │
  │  deadline = failing_since + 120min (default)      │
  │                                                   │
  │  Si NOW < deadline:                               │
  │    skip (todavía hay tiempo para recovery)        │
  │                                                   │
  │  Si NOW ≥ deadline:                               │
  │    ¿Todos los assets ya se recuperaron?           │
  │      YES → DELETE row (cleanup silencioso)        │
  │      NO  → dispara email + embed ROJO             │
  │            UPDATE alerted_at = NOW                │
  └───────────────────────────────────────────────────┘
```

## Decisiones tomadas

### Default window = 120 min (no freshness policy real todavía)

Axell propuso usar `FreshnessPolicy` de cada asset para calcular el deadline. Implementar eso correctamente requiere acceso al asset graph desde el sensor, lo cual cross-code-location es complicado. Empezamos con un fallback fijo de 120 min (configurable via `LATE_ALERT_DEFAULT_MINUTES`) y dejamos el TODO para integrar freshness real en otro PR.

### Sensor registrado solo en global

Registrar `late_failure_alert_sensor` en las 3 code locations causaría race condition (3 sensores procesando los mismos rows). Lo puse solo en `dagster-global-gf` para que haya un solo procesador.

### Tests refactorizados con patch de `_queue_pending_failure_alert`

Los tests originales verificaban `send_email_mock.call_count`. Ahora la función que se llama es `_queue_pending_failure_alert`, así que los tests usan `unittest.mock.patch` para capturar las llamadas. Helper: `_intercept_queued_alerts()` en el test file.

## Archivos modificados

- `dagster-discord-bot-gf/dagster_discord_bot_gf/bot.py` (bot commands)
- `dagster-shared-gf/dagster_shared_gf/shared_failed_sensors.py` (sensor refactor completo)
- `dagster-shared-gf/dagster_shared_gf_tests/test_shared_failed_sensors.py` (tests actualizados)
- `dagster-global-gf/dagster_global_gf/definitions.py` (registro del nuevo sensor)

## Lo que aprendí

- Freshness-aware alerting es un patrón típico en data platforms: no alertar por flakes, solo por violaciones reales del SLA.
- Dagster GraphQL `assetSelection` es útil para saber qué assets corren en un run genérico tipo `__ASSET_JOB`.
- `difflib.get_close_matches` para fuzzy search es suficiente para matcheo aproximado de asset keys sin dependencias extra.
- Normalizar `_`, `-`, `/`, espacios antes de matchear hace que snake_case matchee CamelCase naturalmente.
- En sensors compartidos entre code locations, considerar race conditions — registrar en UNA sola location para evitar concurrent processing.

## Referencias

- PR #138: https://github.com/grupo-farinter/main-dagster/pull/138
- Proyecto previo: [[Discord Bot Dagster - Fixes 2026-04-13]]
- Docs de comandos: [[Bot Discord Dagster - Comandos]]
- Troubleshooting: [[Bot Discord no responde]]
- Gotcha de SecretEnvVar: [[SecretEnvVar gotcha en Dagster sensors]]

## Estado de validación

| Item | Estado |
|---|---|
| Lint + type check | ✅ |
| Unit tests (20/20) | ✅ |
| E2E scenario transitorio | ✅ cleanup silencioso confirmado |
| E2E scenario fallo real | ✅ email + alerted_at confirmado |
| Load en 3 code locations | ✅ |
| Merge + deploy | ⏳ Pendiente |
