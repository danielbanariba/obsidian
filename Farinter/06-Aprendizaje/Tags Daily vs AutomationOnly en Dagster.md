---
tags: [aprendizaje, dagster, tags, jobs]
tema: dagster
fecha: 2026-04-08
---

# Tags Daily vs AutomationOnly en Dagster

## Concepto
En el monorepo, `kielsa_daily_downstream_job` selecciona TODOS los assets con tag `Daily`. Si un asset tiene `Daily` Y también corre por automation/sensor, va a correr **dos veces**.

## La regla (de Brain)
> "El tag automation only es lo único que tiene que llevar. Si solo lleva automation de tags no deberías tener problema, el problema es si le pones el de daily que son los que agarra el job."

## Cuándo usar cada uno

| Tag | Cuándo usar | El asset corre por... |
|---|---|---|
| `tags_repo.Daily.tag` | Assets que van en el job daily downstream | Schedule diario (job) |
| `tags_repo.AutomationOnly.tag` | Assets que tienen su propio sensor/automation | Sensor o automation condition |
| `tags_repo.Disparado.tag` | Assets que se disparan manualmente | Solo manual |
| `tags_repo.DetenerCarga.tag` | Assets desactivados temporalmente | No corre |

## Cómo verificar
SIEMPRE antes de hacer PR, verificar en la UI de Dagster que el asset no aparece en jobs donde no debería.

## Lo que pasó
Netflix tenía `Daily` + sensor → corría dos veces. El fix fue cambiar a `AutomationOnly`.

## Links
- [[Brain - Estandarizacion y fixes 2026-04-08]]
