---
tags: [proyecto, activo, dagster, monitoreo, telegram, openclaw]
fecha_inicio: 2026-04-03
estado: activo
solicitado_por: Edwin Martinez, Brain Padilla
prioridad: alta
---

# Dagster Watchdog

## Contexto
Brain reportó procesos pegados por 22+ horas. Edwin pidió monitoreo. Daniel no sabía cómo identificar procesos pegados.

## Objetivo
Sistema autónomo 24/7 que detecte runs pegados, los cancele, relance, y notifique por Telegram.

## Componentes
- [x] Script `dagster_watchdog.py` — monitoreo basado en baselines aprendidos
- [x] Systemd service — `dagster-watchdog.service` permanente
- [x] OpenClaw + Telegram bot — `@dagster_watchdog_farinter_bot`
- [x] Skills de OpenClaw — `dagster-ops`, `dagster-report`
- [x] Notificaciones FALLO/RESUELTO con error reason

## Baselines aprendidos
| Sensor | Max | Timeout |
|---|---|---|
| acs_hourly | 38 min | 48 min |
| acs_monthly | 80 min | 90 min |
| acs_daily | 0.5 min | 10.5 min |
| manual | 80 min | 90 min |

## Lo que aprendí
- OpenClaw `message send` funciona sin LLM — solo usa el canal
- `gpt-4o-mini` no sigue instrucciones complejas, `gpt-5.4` sí
- Los exec approvals de OpenClaw necesitan `security: "off"` en máquina personal
- El bot puede usar MCP servers (farinter-db, dagster-prd, obsidian)

## Archivos clave
- `scripts/dagster_watchdog.py`
- `~/.config/systemd/user/dagster-watchdog.service`
- `~/.openclaw/workspace/AGENTS.md`
- `~/.openclaw/workspace/skills/dagster-ops/SKILL.md`
- `~/.dagster_watchdog/baselines.json`
