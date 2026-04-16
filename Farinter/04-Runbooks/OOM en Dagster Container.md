---
tags: [runbook, incidente, dagster, oom, docker]
sistema: dagster
severidad: alta
ultima_ocurrencia: 2026-04-02
---

# OOM en Dagster Container

## Sintomas
- Watchdog reporta: "FALLO Dagster - Run xxx (sensor: acs_monthly). Error: SIGKILL signal 9"
- En logs: `OOMPriorityExecutor: proceso hijo was terminated by signal 9`
- El asset no se materializa

## Causa raíz
Los containers de Docker tienen `mem_limit` bajo. El proceso necesita más RAM de la asignada, el kernel lo mata con SIGKILL.

## Resolución
1. Verificar qué container falló:
   ```bash
   ssh analiticasu@172.16.2.220
   docker stats --no-stream | grep dagster
   ```
2. Subir memoria en caliente (sin restart):
   ```bash
   docker update --memory 8g --memory-swap 12g main-dagster-prd-code-location-sap-1
   ```
3. Relanzar la materialización desde Dagster UI o via GraphQL
4. Persistir el cambio en `docker-compose.prd.yaml`

## Prevención
- Optimizar código para usar menos memoria (eliminar clone() innecesarios)
- Monitorear con `docker stats` los picos de memoria
- El watchdog detecta y notifica automáticamente

## Historial
| Fecha | Qué pasó | Quién lo resolvió |
|-------|----------|-------------------|
| 2026-04-01 | OOM en Demanda Limpia (4GB limit) | Edwin detectó, Daniel investigó |
| 2026-04-02 | Fix: docker update 4GB→8GB + optimización código | Daniel + Claude |
