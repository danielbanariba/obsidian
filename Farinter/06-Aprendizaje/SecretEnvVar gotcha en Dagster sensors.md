---
tags: [aprendizaje, dagster, gotcha, python, postgres]
fecha: 2026-04-13
estado: completado
contexto: Discord Bot recovery sensor
---

# SecretEnvVar gotcha en Dagster sensors

## El problema

`DagsterSettings.dagster_postgres_nexus_password` retorna un **objeto `SecretEnvVar`**, no un string. Si lo pasás directamente a un cliente externo como `psycopg2.connect`, Dagster levanta:

```
RuntimeError: Attempted to directly retrieve environment variable
SecretEnvVar("DAGSTER_POSTGRES_NEXUS_PASSWORD"). SecretEnvVar defers
resolution of the environment variable value until run time, and should
only be used as input to Dagster config or resources.

To access the environment variable value, call `get_value` on the
SecretEnvVar, or use os.getenv directly.
```

## Por qué es tan tramposo

`DagsterSettings` es una dataclass cacheada (`@lru_cache` en `get_dagster_config()`). Sus campos usan `default_factory=lambda: os.getenv(...)` que se ejecuta una vez al instanciar.

Para los campos `Optional[str]` el `default_factory` devuelve un string normal o None. Pero el password usa `SecretEnvVar(...)` que es un **wrapper opaco** diseñado para que Dagster lo resuelva a runtime cuando lo inyecta en config schemas o resources.

Visualmente parece que `cfg.dagster_postgres_nexus_password` es un string (el resto de los campos lo son), pero NO LO ES.

## Cómo detectar

Si tu código hace algo como:

```python
cfg = get_dagster_config()
psycopg2.connect(
    host=cfg.dagster_postgres_nexus_host,    # OK, string
    user=cfg.dagster_postgres_nexus_username, # OK, string
    password=cfg.dagster_postgres_nexus_password,  # ❌ SecretEnvVar
)
```

Y está envuelto en `try/except Exception: pass` (best-effort), el bug es **invisible** — falla silenciosamente.

## Fix correcto

Usar `os.getenv` directo en sensores y código que pase credenciales a clientes externos:

```python
import os, psycopg2

def _get_nexus_conn():
    return psycopg2.connect(
        host=os.getenv("DAGSTER_POSTGRES_NEXUS_HOST"),
        port=int(os.getenv("DAGSTER_POSTGRES_NEXUS_PORT", "5432")),
        dbname=os.getenv("DAGSTER_POSTGRES_NEXUS_DB"),
        user=os.getenv("DAGSTER_POSTGRES_NEXUS_USERNAME"),
        password=os.getenv("DAGSTER_POSTGRES_NEXUS_PASSWORD"),
        connect_timeout=5,
    )
```

Alternativa: si querés mantener el wrapper, llamar `cfg.dagster_postgres_nexus_password.get_value()` para extraer el string. Pero para uso ad-hoc en sensores, `os.getenv` es más simple y consistente.

## Regla general

- `DagsterSettings` y `SecretEnvVar` están diseñados para Dagster config schemas y `dg.resource`s, no para uso libre
- En sensores, jobs, ops o helpers que llamen clientes externos: **usar `os.getenv` directo**
- **NUNCA** combinar `try/except Exception: pass` con conexiones a recursos externos sin loguear — oculta bugs por días

## Síntomas que vimos

1. Bug merged en PR #134 (2026-04-09)
2. Sensor `failed_asset_notification_sensor` fired SUCCESS ticks correctamente
3. Pero `_record_failed_assets` swallowed `RuntimeError`
4. Tabla `asset_failure_state` quedó VACÍA siempre
5. Sensor `asset_recovery_notification_sensor` corrió cada 5 min, encontró 0 rows, SKIPPED siempre
6. **Embeds verdes de "Asset Recovered" nunca llegaron en los 2 días post-merge**

## Detectado el

2026-04-13, durante investigación de [[Discord Bot Dagster - Fixes 2026-04-13]] cuando usuario preguntó si las alertas de recovery funcionaban tras dos fallos reales esa madrugada (sensor_acs_daily y dlt_dwh_kielsa_daily_job).

## Referencias

- [[Discord Bot Dagster - Fixes 2026-04-13]]
- Línea original con bug: `dagster-shared-gf/dagster_shared_gf/shared_failed_sensors.py:104` (pre-fix)
- Línea con patrón correcto desde el inicio: `dagster-shared-gf/dagster_shared_gf/shared_failed_sensors.py:752-756`
