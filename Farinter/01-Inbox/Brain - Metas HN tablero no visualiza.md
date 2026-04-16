---
tags: [inbox, requerimiento, metas, kielsa, brain]
fecha: 2026-04-06
de: Brain Padilla
canal: whatsapp
estado: sin-procesar
---

# Brain - Metas HN tablero no se visualiza

## Conversación original
- Brain reportó que el tablero de metas no se puede visualizar
- El archivo `hnmetas_202604.xlsx` tiene 3009 valores en blanco
- Ruta: `\\10.0.4.157\data_repo\kielsa\metas_venta\HN\hnmetas_202604.xlsx`
- Brain dice: "lo único que tiene que hacer el sensor es actualizar las dependencias hasta BI, el tablero lo toma cada día"
- El correo de notificación de error no le llegó a Brain — revisar método de envío

## Análisis
Dos problemas separados:
1. **Archivo con valores en blanco** — el archivo Excel de metas abril 2026 tiene 3009 valores vacíos, el sensor lo rechaza
2. **Correo no llega** — Brain no recibió la notificación de error

## Tareas extraídas
- [ ] Investigar el sensor de metas — qué asset/job lo procesa
- [ ] Verificar por qué el archivo tiene 3009 blancos — ¿el responsable subió mal el archivo?
- [ ] Brain le va a enviar el archivo al responsable para que lo corrija
- [ ] Revisar el envío de correos — Brain pregunta si se usa el mismo método actual
- [ ] Verificar el CSV de correos de notificación

## Clasificación
Nuevo issue — no tiene proyecto existente aún
