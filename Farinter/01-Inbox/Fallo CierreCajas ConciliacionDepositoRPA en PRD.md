---
tags: [inbox, incidente, cierre-cajas, rpa, prd]
fecha: 2026-04-06
de: Watchdog Telegram Bot
canal: telegram
estado: completado
---

# Fallo CierreCajas ConciliacionDepositoRPA en PRD

## Diagnóstico
- **Asset**: `DL_FARINTER/dbo/DL_Kielsa_CierreCajas_ConciliacionDepositoRPA_Stg`
- **Error**: `Invalid object name 'dbo.Conciliacion_ResultadosCierreFinalRPA'`
- **Causa**: La tabla fuente no existe en PRD — la crea el RPA de Katherine
- **En DEV**: la tabla SÍ existe y tiene datos
- **Run fallido**: `6cec9d53` (PRD)
- **Reproducible**: sí, falló 2 veces seguidas

## Código
`dagster-kielsa-gf/defs/cierre_cajas_etl_dwh/assets.py:2003`
```sql
SELECT * FROM dbo.Conciliacion_ResultadosCierreFinalRPA WITH (NOLOCK)
```

## Acción requerida
Katherine o Brain deben crear/ejecutar el RPA en PRD para que la tabla exista.

## Tareas
- [x] Tabla copiada de DEV → PRD (27,492 filas)
- [x] Asset relanzado y corrió exitosamente (run ef0162af, 58s)
- [x] Verificar que Katherine corra el RPA en PRD para mantener la tabla actualizada
