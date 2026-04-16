---
tags: [bot, discord, dagster, comandos, guia, equipo]
fecha: 2026-04-14
estado: completado
audiencia: equipo de desarrollo Farinter
---

# Bot Discord Dagster — Guía para el equipo

## 🟢 Comandos para todos

### `/dagster-status`
Resumen rápido de los últimos 50 runs: cuántos exitosos, fallidos, corriendo, en cola, y los últimos fallos con nombre del asset.

```
/dagster-status
```

### `/asset nombre:<parte>`
Busca un asset por nombre con búsqueda aproximada. Devuelve última materialización, owners, link directo al asset en Dagster UI.

**Ejemplos**:
```
/asset nombre:cliente_general
/asset nombre:ventas
/asset nombre:hecho ventas        ← multi-palabra
/asset nombre:tabular kielsa      ← multi-palabra
```

Tips:
- Acepta snake_case y CamelCase (busca `cliente_general` y matchea `ClienteGeneral`)
- Si pones varias palabras, busca assets que contengan TODAS
- Si encuentra varios, te lista los 5 más cercanos para que refines

### `/register email:<email>`
Vincula tu email corporativo con tu Discord. Esto es lo que permite que cuando un asset que sos owner falla, te taggee automáticamente con `@vos`.

**Ejemplo**:
```
/register email:tu.nombre@farinter.com
```

Solo acepta dominios `farinter.com`, `farinter.hn`, `kielsa.hn`.

### `/whoami`
Muestra a qué email está vinculado tu Discord actualmente.

### `/list`
Lista todos los usuarios registrados en el bot.

### `/unregister`
Elimina tu registro.

### `/help`
Muestra esta lista de comandos.

---

## 🛠️ Comandos para admins (requiere "Manage Channels")

### `/enable-here`
Habilita el bot en el canal donde tipées el comando. Antes esto requería SSH al server, ahora es self-service.

### `/disable-here`
Inversa del anterior. Quita el bot del canal.

### `/silence asset:<key> duracion:<tiempo>`
Silencia las alertas rojas de un asset por un tiempo determinado. Útil cuando ya sabés que algo está roto y estás trabajando en el fix.

**Ejemplos**:
```
/silence asset:DL_FARINTER/dbo/mi_asset duracion:2h
/silence asset:BI_FARINTER/dbo/X duracion:30m
/silence asset:TEST/foo/bar duracion:1d
```

Formatos válidos: `30m` (minutos), `2h` (horas), `1d` (días).

### `/unsilence asset:<key>`
Remueve el silence de un asset.

```
/unsilence asset:DL_FARINTER/dbo/mi_asset
```

---
## ❓ FAQ

**¿Por qué el bot no responde en mi canal?**
Porque el canal no está habilitado. Pedile a un admin que tipee `/enable-here` ahí.

**¿Por qué no me llega notificación cuando falla "mi" asset?**
Probablemente:
1. El asset no tiene `owners` definido → agregar tu email en el código del asset
2. No estás registrado → tipear `/register email:tu.nombre@farinter.com`

**¿Por qué se demoran las alertas?**
Por diseño: ahora el bot espera la `freshness_policy` del asset antes de alertar. Si el asset es daily con freshness de 27hs, el bot espera 27hs antes de mandar la alerta — porque hasta entonces el retry automático puede haberlo recuperado. Esto elimina ruido de fallos transitorios.

**¿Cómo silencio temporalmente alertas que no me importan ahora?**
`/silence asset:<key> duracion:2h` (admin only).

---
