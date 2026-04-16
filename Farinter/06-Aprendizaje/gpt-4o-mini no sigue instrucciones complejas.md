---
tags: [aprendizaje, openclaw, llm, modelos]
tema: openclaw
fecha: 2026-04-03
---

# gpt-4o-mini no sigue instrucciones complejas

## Concepto
En OpenClaw, `gpt-4o-mini` no siguió las instrucciones del AGENTS.md para usar curl contra la API de Dagster. Buscó archivos locales (`precios.sh`) y usó `crontab` en vez de los scripts configurados. `gpt-4o` lo hizo mejor pero seguía fallando con curls complejos. `gpt-4.1` y `gpt-5.4` sí siguen las instrucciones.

## Por qué es importante
El modelo que usés en OpenClaw importa MUCHO para tareas que requieren seguir instrucciones específicas (como "usa este script, no armes el curl a mano").

## Ranking para OpenClaw
| Modelo | Sigue instrucciones | Costo |
|---|---|---|
| gpt-5.4 (codex) | Excelente | $0 (suscripción) |
| gpt-4.1 | Muy bueno | ~$2/M tokens |
| gpt-4o | Bueno | ~$2.50/M tokens |
| gpt-4o-mini | No sigue | ~$0.15/M tokens |

## Links
- [[OpenClaw Telegram Bot]]
