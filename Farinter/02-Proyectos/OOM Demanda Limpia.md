---
tags: [proyecto, completado, dagster, oom, memoria]
fecha_inicio: 2026-04-02
fecha_fin: 2026-04-03
estado: completado
solicitado_por: Edwin Martinez
prioridad: alta
---

# OOM Demanda Limpia

## Contexto
Edwin reportó que `BI_SAP_Hecho_SocAlmArtGpoCli_Demanda_Limpia` no se actualizaba. La tabla llevaba ~30 días sin materializar. Brain reportó que había procesos pegados por 22 horas.

## Objetivo
Materializar la tabla y evitar que vuelva a fallar por OOM.

## Tareas
- [x] Diagnosticar por qué falló
- [x] Materializar dependencias DBT (Demanda_Plan: 25min, Stock_Plan: 96s)
- [x] Subir memoria del container de 4GB a 8GB via `docker update`
- [x] Materializar Demanda Limpia (80 min, 1,134,382 filas)
- [x] Optimizar código para reducir pico de memoria ~50%
- [ ] Persistir `docker update` en docker-compose.prd.yaml
- [ ] Crear PR con optimización para Brian

## Decisiones tomadas
- **docker update en caliente** en vez de esperar PR — Edwin necesitaba la tabla HOY
- **8GB** para code-location-sap (servidor tiene 47GB, estaba desperdiciado con 4GB)
- **Eliminar clone()** en `clean_dataframe_for_sql` — Polars `with_columns` ya usa COW

## Archivos modificados
- `dagster-shared-gf/dagster_shared_gf/shared_helpers.py` — eliminados 2x clone()
- `dagster-sap-gf/defs/control_demanda/limpiar_demanda.py` — del demanda_procesada
- `dagster-kielsa-gf/defs/control_demanda/limpiar_demanda.py` — mismo fix

## Lo que aprendí
- El OOM era en `save_demanda_procesada`, NO en MDDME — el MDDME sí terminaba
- `docker update --memory 8g` cambia límites sin restart
- PRD tiene 47GB RAM pero el container solo usaba 4GB
- El pico de memoria era ~4x el DataFrame por los clones innecesarios

## Referencias
- PR: grupo-farinter/main-dagster#116
- Run exitoso: `69f1cd41` (PRD)
- SSH PRD: `ssh analiticasu@172.16.2.220`
