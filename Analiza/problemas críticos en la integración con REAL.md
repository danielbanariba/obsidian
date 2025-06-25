## PROBLEMAS CRÍTICOS IDENTIFICADOS

### 1. **INCOMPATIBILIDAD DE TIPOS DE DATOS**

El JSON envía:
```json
"tubebarcode": "000103263997201" // String de 15 caracteres
```

Pero la tabla `bacteria_resultado_encabezado` tiene:
```sql
tubo_id: bigint //NO PUEDE almacenar el código completo
```
Aunque el código existe en `core_ordentubos.TuboCodigoBarra`, necesitas almacenar la referencia completa en la tabla de resultados para mantener trazabilidad.

### 2. **DISEÑO DEFICIENTE DEL CIM**

Actualmente guardas todo concatenado:
```sql
cmi: text -- "0 0,12 μg/mL"
```

El JSON ya viene estructurado correctamente:
```json
"value": {
  "value": "0 0,12",
  "unit": "μg/mL"
}
```

**DEBERÍAS tener**:
```sql
cmi_valor: DECIMAL(10,4) -- Para queries analíticos
cmi_unidad: VARCHAR(10)  -- Para la unidad
```

### 3. **INCONSISTENCIAS EN NOMENCLATURA**

- JSON envía: `"BLA": true`
- Tabla tiene: `blee: text`

¿Son lo mismo? ¿Por qué uno es boolean y otro text? **Esto es confuso y propenso a errores**.

## Recomendaciones

### 1. **Modificar la estructura de tablas**:

```sql
ALTER TABLE bacteria_resultado_encabezado 
ADD COLUMN codigo_barras_tubo VARCHAR(20);

ALTER TABLE bacteria_resultado_detalle
ADD COLUMN cmi_valor DECIMAL(10,4),
ADD COLUMN cmi_unidad VARCHAR(10);

-- Renombrar para consistencia
ALTER TABLE bacteria_resultado_encabezado 
RENAME COLUMN blee TO bla;
```

### 2. **Crear documentación de mapeo EXPLÍCITA**:

|Campo JSON|Tabla|Columna|Tipo|Notas|
|---|---|---|---|---|
|tubebarcode|bacteria_resultado_encabezado|codigo_barras_tubo|VARCHAR(20)|Nuevo campo|
|receptiondatetime|bacteria_resultado_encabezado|fecha_recepcion|TIMESTAMP||
|microorganismid|bacteria_resultado_encabezado|organismo_seleccionado_id|INTEGER||
|BLA|bacteria_resultado_encabezado|bla|BOOLEAN|Cambiar de TEXT|
