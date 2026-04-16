---
tags: [proyecto, completado, mcp, sql-server]
fecha: 2026-04-05
estado: completado
solicitado_por: Daniel Banariba
prioridad: media
---

# MCP Improvements

## Contexto
Los MCP servers tenían limitaciones que bloqueaban el trabajo: no soportaban `datetimeoffset`, no permitían filtros WHERE complejos, y no podían acceder a schemas como `mongo_db_crm_hn`.

## Tareas
- [x] Agregar soporte `datetimeoffset` en farinter-db (SQL type -155)
- [x] Agregar parámetro `where` a `read_table` para filtros SQL complejos
- [x] Habilitar schemas `mongo_db_crm_hn` y `db_planning`
- [x] Fix bug en dagster MCP: `assetSelection` va dentro de `selector`, no en `ExecutionParams`
- [x] Agregar MCP de Obsidian a Claude Code, Codex, OpenCode y OpenClaw

## Archivos modificados
- `/home/banar/Desktop/grupo-farinter/mcp-db/server.py` — datetimeoffset converter + where param
- `/home/banar/Desktop/grupo-farinter/main-dagster/.mcp.json` — schemas + obsidian
- `~/.codex/config.toml` — obsidian MCP
- `~/.config/opencode/opencode.json` — obsidian MCP

## Lo que aprendí
- [[datetimeoffset en pyodbc]]
- [[MCP read_table con WHERE]]
