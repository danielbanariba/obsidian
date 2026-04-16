---
tags: [inbox, requerimiento, brain, discord, kuma, alertas]
fecha: 2026-04-08
de: Brain Padilla
canal: whatsapp
estado: sin-procesar
---

# Brain - Discord, Kuma y alertas Dagster

## Contexto
Brain dice que OpenClaw NO va en la empresa (vulnerabilidades). Hay 3 temas separados:

1. **Discord**: Ya hay integración con Dagster — solo se necesita una variable de entorno con una URL webhook. Hay ejemplos en el repo.
2. **Kuma**: Para alertas de servicios (si Dagster está caído, etc). Contactar a Gabriel.
3. **Procesos pegados**: Dagster tiene `dagster/max_runtime` para límites de ejecución. También se puede usar la API para alertas.

## Tareas
- [x] Unirse al Discord del equipo (ya hecho: https://discord.gg/xy6TfGT9K)
- [x] Investigar los ejemplos de Discord webhook en el repo
- [ ] Contactar a Gabriel para lo de Kuma
- [x] Investigar `dagster/max_runtime` como alternativa al watchdog
- [ ] Ver si se pueden combinar: Discord para alertas + max_runtime para procesos pegados

## Notas de Brain
- "para la parte de fallos de procesos, ya te caen correos si sos owner"
- "Es un tag, me parece que solo se podía usar con Jobs"
- "jamas meteriamos eso (openclaw) en una empresa, tiene una de vulnerabilidades"
