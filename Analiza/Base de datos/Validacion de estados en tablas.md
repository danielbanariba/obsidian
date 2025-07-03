# Validación de Estados en Tablas - Integración Sistema REAL

## Contexto

Asegurar que en la inserción de resultados de bacteriología se actualicen correctamente los estados en las tablas relacionadas, específicamente `core_ordentubos` y `core_tuboprueba`.

### Estado Actual

-  **core_tuboprueba**: Se actualiza correctamente (100% sincronización)
-  **core_ordentubos.TuboEstatus**: NO se actualiza (permanece en valor "1")

## 2. Análisis

#### bacteria_resultado_encabezado
```sql
- orden_numero: text
- tubo_id: bigint  -- ipo diferente a core_tuboprueba.TuboId (integer)
- prueba_id: bigint
- resultado_validado: boolean
```

#### core_tuboprueba
```sql
- TuboId: integer
- PruebaId: integer
- PruebaFinalizada: boolean
- PruebaFechaHoraEnviaDato: timestamp with time zone
```

#### core_ordentubos
```sql
- TuboId: integer
- TuboEstatus: character varying(2)
- OrdenId: character varying(14)
```

### 2.3 Evidencia del Problema

| Métrica                            | Valor    | Estado           |
| ---------------------------------- | -------- | ---------------- |
| Resultados bacteriología validados | 25       | -                |
| Con PruebaFinalizada actualizada   | 25       | 100%             |
| Órdenes con TuboEstatus = 1        | 18       | Sin variación    |
| Valores únicos de TuboEstatus      | Solo "1" |  No se actualiza |

## 3. Solución

### 3.1 Definir Estados Válidos para TuboEstatus

Es necesario establecer los valores válidos para `TuboEstatus`:

```sql
-- '1' = Tubo registrado/pendiente
-- '2' = Muestra tomada
-- '3' = En proceso de análisis
-- '4' = Resultado parcial
-- '5' = Resultado completo
-- 'RC' = Resultado de Cultivo (bacteriología)
```

### 3.2 Implementar Trigger para Actualización de TuboEstatus

```sql
CREATE OR REPLACE FUNCTION update_tubo_estatus_bacteriologia()
RETURNS TRIGGER AS $$
DECLARE
    v_tubo_id INTEGER;
    v_orden_id VARCHAR(14);
BEGIN
    IF NEW.resultado_validado = TRUE AND NEW.tubo_id IS NOT NULL THEN
        
        -- Convertir bigint a integer de forma segura
        v_tubo_id := NEW.tubo_id::INTEGER;
        
        SELECT "OrdenId" INTO v_orden_id
        FROM core_ordentubos
        WHERE "TuboId" = v_tubo_id;
        
        UPDATE core_ordentubos 
        SET "TuboEstatus" = 'RC'  
        WHERE "TuboId" = v_tubo_id;
    END IF;
    
    RETURN NEW;
EXCEPTION
    WHEN numeric_value_out_of_range THEN
        RAISE WARNING 'Error convirtiendo tubo_id % a integer', NEW.tubo_id;
        RETURN NEW;
    WHEN OTHERS THEN
        RAISE WARNING 'Error actualizando TuboEstatus: %', SQLERRM;
        RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Crear el trigger
CREATE TRIGGER trg_actualizar_tubo_estatus_bacteria
    AFTER INSERT OR UPDATE OF resultado_validado 
    ON bacteria_resultado_encabezado
    FOR EACH ROW
    WHEN (NEW.resultado_validado = TRUE)
    EXECUTE FUNCTION update_tubo_estatus_bacteriologia();
```

### 3.3 Script de Actualización Retroactiva

Para corregir los registros existentes:

```sql
-- Actualizar TuboEstatus para resultados ya validados
UPDATE core_ordentubos ot
SET "TuboEstatus" = 'RC'
FROM bacteria_resultado_encabezado bre
WHERE ot."TuboId" = bre.tubo_id::INTEGER
  AND bre.resultado_validado = TRUE
  AND ot."TuboEstatus" = '1';

-- Verificar la actualización
SELECT 
    ot."TuboEstatus",
    COUNT(DISTINCT bre.orden_numero) as ordenes_actualizadas
FROM bacteria_resultado_encabezado bre
INNER JOIN core_ordentubos ot ON ot."TuboId" = bre.tubo_id::INTEGER
WHERE bre.resultado_validado = TRUE
GROUP BY ot."TuboEstatus";
```

## 4. Validación
```sql
-- 1. Verificar sincronización completa
WITH estado_sincronizacion AS (
    SELECT 
        bre.id,
        bre.orden_numero,
        bre.resultado_validado,
        tp."PruebaFinalizada",
        ot."TuboEstatus",
        CASE 
            WHEN bre.resultado_validado = TRUE 
                AND tp."PruebaFinalizada" = TRUE 
                AND ot."TuboEstatus" = 'RC' 
            THEN 'OK'
            ELSE 'PENDIENTE'
        END as estado_sync
    FROM bacteria_resultado_encabezado bre
    LEFT JOIN core_tuboprueba tp 
        ON tp."TuboId" = bre.tubo_id::INTEGER 
        AND tp."PruebaId" = bre.prueba_id::INTEGER
    LEFT JOIN core_ordentubos ot 
        ON ot."TuboId" = bre.tubo_id::INTEGER
    WHERE bre.tubo_id IS NOT NULL
)
SELECT 
    estado_sync,
    COUNT(*) as cantidad
FROM estado_sincronizacion
GROUP BY estado_sync;

-- 2. Dashboard de monitoreo
SELECT 
    'Total Resultados Bacteriología' as metrica,
    COUNT(*) as valor
FROM bacteria_resultado_encabezado
WHERE tubo_id IS NOT NULL

UNION ALL

SELECT 
    'Resultados Validados',
    COUNT(*)
FROM bacteria_resultado_encabezado
WHERE resultado_validado = TRUE
AND tubo_id IS NOT NULL

UNION ALL

SELECT 
    'Con PruebaFinalizada Actualizada',
    COUNT(*)
FROM bacteria_resultado_encabezado bre
INNER JOIN core_tuboprueba tp 
    ON tp."TuboId" = bre.tubo_id::INTEGER 
    AND tp."PruebaId" = bre.prueba_id::INTEGER
WHERE bre.resultado_validado = TRUE
AND tp."PruebaFinalizada" = TRUE

UNION ALL

SELECT 
    'Con TuboEstatus Actualizado',
    COUNT(DISTINCT bre.orden_numero)
FROM bacteria_resultado_encabezado bre
INNER JOIN core_ordentubos ot 
    ON ot."TuboId" = bre.tubo_id::INTEGER
WHERE bre.resultado_validado = TRUE
AND ot."TuboEstatus" != '1';
```

## 5. Consideraciones

### Compatibilidad de Tipos

- **Problema**: `bacteria_resultado_encabezado.tubo_id` es BIGINT, pero `core_ordentubos.TuboId` es INTEGER
- **Solución**: Conversión explícita con manejo de errores en el trigger

### 5.3 Rollback

Si se necesita revertir los cambios:
```sql
-- Eliminar trigger
DROP TRIGGER IF EXISTS trg_actualizar_tubo_estatus_bacteria 
ON bacteria_resultado_encabezado;

-- Eliminar función
DROP FUNCTION IF EXISTS update_tubo_estatus_bacteriologia();

-- Restaurar estados anteriores (si se guardó backup)
UPDATE core_ordentubos SET "TuboEstatus" = '1';
```