---
tags: [proyecto, activo, openclaw, telegram, monitoreo]
fecha: 2026-04-03
estado: activo
solicitado_por: Daniel Banariba
prioridad: alta
---

# OpenClaw Telegram Bot

## Contexto
Necesitábamos un agente de IA accesible desde el celular para consultar y operar Dagster PRD sin necesidad de VPN ni compu. El usuario mencionó OpenClaw como referencia.

## Objetivo
Bot de Telegram que consulta Dagster, ejecuta assets, revisa logs y reporta estado — todo desde el celular.

## Componentes
- [x] OpenClaw instalado (v2026.4.2) como systemd service
- [x] Telegram bot `@dagster_watchdog_farinter_bot` conectado
- [x] WhatsApp conectado (deshabilitado, usando Telegram)
- [x] Modelo: `openai-codex/gpt-5.4` (usa suscripción Codex, $0 extra)
- [x] Skills: `dagster-ops`, `dagster-report`
- [x] MCP servers: `farinter-db`, `dagster-prd`, `obsidian`
- [x] Exec approvals: `security: "off"` (máquina personal)
- [x] DM policy: `allowlist` solo user ID `1019432734`

## Decisiones tomadas
- **gpt-5.4 via Codex** en vez de API key — $0 costo extra
- **Telegram en vez de WhatsApp** — llega como contacto separado
- **exec security: off** — es máquina personal, la seguridad está en el allowlist de Telegram
- **Scripts helper** en vez de curls crudos — el modelo se confundía armando GraphQL

## Archivos clave
- `~/.openclaw/workspace/AGENTS.md` — instrucciones operativas
- `~/.openclaw/workspace/SOUL.md` — personalidad del bot
- `~/.openclaw/workspace/skills/dagster-ops/SKILL.md`
- `~/.openclaw/workspace/skills/dagster-report/SKILL.md`
- `~/.openclaw/openclaw.json` — config principal

## Lo que aprendí
- [[OpenClaw setup-token vs API key]]
- [[gpt-4o-mini no sigue instrucciones complejas]]
- [[OpenClaw exec approvals]]

## Referencias
- OpenClaw docs: https://docs.openclaw.ai
- Telegram bot: @dagster_watchdog_farinter_bot
- Chat ID: 1019432734
