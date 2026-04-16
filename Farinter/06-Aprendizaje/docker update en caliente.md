---
tags: [aprendizaje, docker, memoria, devops]
tema: docker
fecha: 2026-04-02
---

# docker update en caliente

## Concepto
`docker update --memory 8g --memory-swap 12g <container>` cambia los límites de memoria de un container SIN reiniciarlo. El cambio es inmediato.

## Por qué es importante
Cuando un container se queda sin memoria (OOM), no necesitás recrear el container ni hacer deploy. Cambiás el límite en caliente y relanzás el proceso.

## Ejemplo práctico
```bash
# Ver límites actuales
docker inspect <container> --format '{{.HostConfig.Memory}}'

# Cambiar en caliente
docker update --memory 8g --memory-swap 12g main-dagster-prd-code-location-sap-1

# IMPORTANTE: no es persistente — si el container se recrea, vuelve al valor del compose
# Para persistir: editar docker-compose.prd.yaml
```

## Links
- [[OOM Demanda Limpia]]
- [[OOM en Dagster Container]]
