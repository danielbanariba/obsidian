---
tags: [proyecto, activo, rapibac, dagster, katherine]
fecha: 2026-04-07
estado: en-progreso
solicitado_por: Katherine Lopez
prioridad: media
---

# Rapibac ETL

## Contexto
Katherine pidió programar la carga de archivos Rapibac. El archivo llega a `\\192.168.0.211\Servicios Externos\RAPIBAC` con formato Excel: una columna de sucursal y otra de monto de transacciones. Katherine dice que "es lo mismito que TENGO".

## Tareas
- [x] Crear módulo `rapibac_etl_dwh` siguiendo patrón TENGO
- [x] Asset `DL_Kielsa_Rapibac_Stg` con procesamiento Polars
- [x] Sensor `kielsa_rapibac_file_sensor` para detectar archivos nuevos
- [x] Job `rapibac_daily_job`
- [x] Probar localmente: SMB → proceso → SQL Server DEV (231 filas OK)
- [x] Verificar tabla en DEV con MCP
- [x] PR #122 creado
- [ ] Esperar aprobación de Brain
- [ ] Deployar a DEV y probar en Dagster UI
- [ ] Katherine hace el tablero de Power BI

## Decisiones tomadas
- Usar `smb_resource_analitica_srvkielsa211` (ya existe para 192.168.0.211)
- Prefijos: `"RAPIBAC"` y `"Farmacia Kielsa"` (por si el archivo no tiene prefijo RAPIBAC)
- `swap_table=True` — full refresh cada vez

## Archivos creados
- `dagster-kielsa-gf/defs/rapibac_etl_dwh/assets.py`
- `dagster-kielsa-gf/defs/rapibac_etl_dwh/defs.yaml`
- `dagster-kielsa-gf/defs/rapibac_etl_dwh/__init__.py`

## Archivos modificados
- `dagster-kielsa-gf/defs/jobs_schedules/jobs.py` — `rapibac_daily_job`
- `dagster-kielsa-gf/defs/sensors/sensors.py` — `kielsa_rapibac_file_sensor`

## Referencias
- PR: grupo-farinter/main-dagster#122
- Carpeta SMB: `\\192.168.0.211\Servicios Externos\RAPIBAC`
- Archivo ejemplo: `Farmacia Kielsa 06 al 08 Feb 2026.xlsx`
- [[Katherine - Carpeta Rapibac]]
