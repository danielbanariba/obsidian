---
tags: [tarea, crm, nilo, power-bi]
fecha: 2026-04-01
estado: completado
proyecto: 
asignado_por: Nilo
---

# Verificar tablero Nilo - Productividad CRM

## Requerimiento
Nilo reportó que el tablero de Productividad de Llamadas CRM no mostraba datos del 31/03 y 01/04. La tabla base `client_to_call.call_day` estaba atrasada.

## Conversación original
Nilo (01/04): "la tabla DL_FARINTER.mongo_db_crm_hn.client_to_call no está generando datos del 31 y 1"

## Plan
- [x] Identificar la fuente: asset `clientToCall` en `dagster_kielsa_gf`
- [x] Parchear vista `VW_LlamadasRecetasHist` con COALESCE para usar fecha real de llamada
- [x] Parchear `client_to_call` directo en PRD (145 filas del 31/03)
- [x] Verificar data actual — tiene registros hasta 2026-04-05 (1,151 hoy)
- [x] Confirmar con Nilo que el tablero se ve bien después de refresh (06/04 - confirmado por WhatsApp)

## Resultado
- Vista parcheada y deployada en PRD via DBT
- Data verificada: `call_day` tiene registros diarios hasta hoy
- Vista `VW_LlamadasRecetasHist` muestra 148 registros del 31/03 (antes: 0)
- Nilo dijo "sigue lo mismo" el 01/04 — probablemente cache de Power BI

## Notas
- La vista depende de `call_day` para clientes NO llamados
- Si `call_day` se atrasa de nuevo, los clientes sin llamada no aparecen
- El asset `clientToCall` se materializa cada hora — si CRM no actualiza `call_day`, el problema es upstream
