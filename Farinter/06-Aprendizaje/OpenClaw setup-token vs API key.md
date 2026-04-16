---
tags: [aprendizaje, openclaw, anthropic, auth]
tema: openclaw
fecha: 2026-04-03
---

# OpenClaw setup-token vs API key

## Concepto
`claude setup-token` genera un OAuth token (`sk-ant-oat01-...`) para autenticar Claude Code CLI. OpenClaw necesita una API key real (`sk-ant-api03-...`) de console.anthropic.com. Son formatos diferentes e incompatibles.

## Por qué es importante
Pasamos 1 hora intentando hacer funcionar el setup-token en OpenClaw. No funciona. Hay que ir directo a la API key o usar otro provider.

## Solución final
Usar `openai-codex/gpt-5.4` que reutiliza la suscripción de Codex via OAuth — $0 costo extra. La API key de Anthropic no tiene créditos.

## Links
- [[OpenClaw Telegram Bot]]
