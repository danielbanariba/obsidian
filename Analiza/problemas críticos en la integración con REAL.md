## PROBLEMAS CRÍTICOS IDENTIFICADOS

### 2. **DISEÑO DEFICIENTE DEL CIM (Punto mejora)** 

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


ordene tubo y ordenes pruebas