# Scripts DDL - Integración REAL Bacteriología

## Orden de Ejecución

1. `01_crear_tabla_aux_examenes_real.sql`
2. `02_modificar_bacteria_resultado_encabezado.sql`
3. `03_crear_tabla_bacteria_resultado_organismos.sql`
4. `04_crear_tabla_bacteria_mensajes_real.sql`
5. `05_crear_indices_optimizacion.sql`
6. `06_insertar_datos_iniciales.sql`
7. `07_validaciones_finales.sql`

---

## Script 01: aux_examenes_real

```sql
CREATE TABLE aux_examenes_real (
    id SERIAL PRIMARY KEY,
    examen_analiza_id VARCHAR(20) NOT NULL,
    examen_real_codigo VARCHAR(50) NOT NULL,
    examen_real_nombre VARCHAR(200) NOT NULL,
    tipo_muestra_real INTEGER NOT NULL,
    activo BOOLEAN DEFAULT TRUE,
    fecha_mapeo TIMESTAMP DEFAULT NOW(),
    usuario_mapeo VARCHAR(100) DEFAULT 'SISTEMA',
    fecha_ultima_actualiza TIMESTAMP DEFAULT NOW(),
    observaciones TEXT,
    
    CONSTRAINT fk_aux_examenes_analiza 
        FOREIGN KEY (examen_analiza_id) 
        REFERENCES core_examenes(ExamenId)
        ON DELETE RESTRICT,
    CONSTRAINT fk_aux_tipo_muestra 
        FOREIGN KEY (tipo_muestra_real) 
        REFERENCES bacteria_catalogo_tipo_muestra(id)
        ON DELETE RESTRICT,
    CONSTRAINT uk_aux_examenes_mapeo 
        UNIQUE (examen_analiza_id, examen_real_codigo),
    CONSTRAINT chk_examen_real_codigo_valido 
        CHECK (LENGTH(TRIM(examen_real_codigo)) > 0),
    CONSTRAINT chk_examen_real_nombre_valido 
        CHECK (LENGTH(TRIM(examen_real_nombre)) > 0),
    CONSTRAINT chk_tipo_muestra_valido 
        CHECK (tipo_muestra_real BETWEEN 1 AND 4)
);
```

```sql
CREATE INDEX idx_aux_examenes_analiza_id ON aux_examenes_real(examen_analiza_id);
CREATE INDEX idx_aux_examenes_real_codigo ON aux_examenes_real(examen_real_codigo);
CREATE INDEX idx_aux_examenes_activo ON aux_examenes_real(activo) WHERE activo = TRUE;
```

```sql
CREATE OR REPLACE FUNCTION update_aux_examenes_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.fecha_ultima_actualiza = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_aux_examenes_update_timestamp
    BEFORE UPDATE ON aux_examenes_real
    FOR EACH ROW
    EXECUTE FUNCTION update_aux_examenes_timestamp();
```

```sql
COMMENT ON TABLE aux_examenes_real IS 'Mapeo entre exámenes Analiza y códigos sistema REAL';
COMMENT ON COLUMN aux_examenes_real.examen_analiza_id IS 'ID del examen en sistema Analiza';
COMMENT ON COLUMN aux_examenes_real.examen_real_codigo IS 'Código del examen en sistema REAL';
COMMENT ON COLUMN aux_examenes_real.tipo_muestra_real IS 'Tipo de muestra requerida por REAL (1-4)';
```

---

## Script 02: bacteria_resultado_encabezado

```sql
ALTER TABLE bacteria_resultado_encabezado 
ADD COLUMN IF NOT EXISTS equipo_vitek_id VARCHAR(50),
ADD COLUMN IF NOT EXISTS real_session_id VARCHAR(100),
ADD COLUMN IF NOT EXISTS astm_message_id VARCHAR(100),
ADD COLUMN IF NOT EXISTS hl7_message_id VARCHAR(100),
ADD COLUMN IF NOT EXISTS codigo_barra_muestra VARCHAR(15),
ADD COLUMN IF NOT EXISTS metodo_identificacion VARCHAR(100),
ADD COLUMN IF NOT EXISTS multiple_organismos BOOLEAN DEFAULT FALSE,
ADD COLUMN IF NOT EXISTS organismo_principal_id INTEGER,
ADD COLUMN IF NOT EXISTS organismos_secundarios JSONB,
ADD COLUMN IF NOT EXISTS real_raw_data JSONB,
ADD COLUMN IF NOT EXISTS estado_sincronizacion VARCHAR(20) DEFAULT 'PENDIENTE',
ADD COLUMN IF NOT EXISTS fecha_envio_real TIMESTAMP,
ADD COLUMN IF NOT EXISTS fecha_respuesta_real TIMESTAMP,
ADD COLUMN IF NOT EXISTS error_real TEXT,
ADD COLUMN IF NOT EXISTS version_real VARCHAR(20),
ADD COLUMN IF NOT EXISTS operador_equipo VARCHAR(100),
ADD COLUMN IF NOT EXISTS numero_corrida VARCHAR(50),
ADD COLUMN IF NOT EXISTS control_calidad JSONB,
ADD COLUMN IF NOT EXISTS comentarios_tecnicos TEXT,
ADD COLUMN IF NOT EXISTS requiere_confirmacion BOOLEAN DEFAULT FALSE,
ADD COLUMN IF NOT EXISTS confirmado_por VARCHAR(100),
ADD COLUMN IF NOT EXISTS fecha_confirmacion TIMESTAMP,
ADD COLUMN IF NOT EXISTS fecha_ultima_actualiza TIMESTAMP DEFAULT NOW(),
ADD COLUMN IF NOT EXISTS telemedicina_enviado BOOLEAN DEFAULT FALSE,
ADD COLUMN IF NOT EXISTS telemedicina_fecha_envio TIMESTAMP,
ADD COLUMN IF NOT EXISTS telemedicina_response JSONB,
ADD COLUMN IF NOT EXISTS telemedicina_error TEXT,
ADD COLUMN IF NOT EXISTS telemedicina_reintentos INTEGER DEFAULT 0;
```

```sql
ALTER TABLE bacteria_resultado_encabezado 
ADD CONSTRAINT chk_estado_sincronizacion 
    CHECK (estado_sincronizacion IN ('PENDIENTE', 'ENVIADO', 'CONFIRMADO', 'ERROR')),
ADD CONSTRAINT chk_probabilidad_valida 
    CHECK (probabilidad_seleccion >= 0 AND probabilidad_seleccion <= 100),
ADD CONSTRAINT chk_codigo_barra_formato 
    CHECK (LENGTH(codigo_barra_muestra) <= 15),
ADD CONSTRAINT chk_blee_valores 
    CHECK (blee IN ('POSITIVO', 'NEGATIVO') OR blee IS NULL),
ADD CONSTRAINT chk_carbapenemasa_valores 
    CHECK (carbapenemasas IN ('POSITIVO', 'NEGATIVO') OR carbapenemasas IS NULL);
```

```sql
CREATE INDEX idx_bacteria_resultado_codigo_barra ON bacteria_resultado_encabezado(codigo_barra_muestra);
CREATE INDEX idx_bacteria_resultado_estado_sync ON bacteria_resultado_encabezado(estado_sincronizacion);
CREATE INDEX idx_bacteria_resultado_real_session ON bacteria_resultado_encabezado(real_session_id);
CREATE INDEX idx_bacteria_resultado_equipo_vitek ON bacteria_resultado_encabezado(equipo_vitek_id);
```

```sql
CREATE OR REPLACE FUNCTION update_bacteria_resultado_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.fecha_ultima_actualiza = NOW();
    
    IF NEW.resultado_validado = TRUE AND OLD.resultado_validado = FALSE THEN
        NEW.fecha_confirmacion = NOW();
        NEW.confirmado_por = COALESCE(NEW.usuario_validacion, NEW.usuario_registro);
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_bacteria_resultado_update
    BEFORE UPDATE ON bacteria_resultado_encabezado
    FOR EACH ROW
    EXECUTE FUNCTION update_bacteria_resultado_timestamp();
```

```sql
CREATE OR REPLACE FUNCTION update_tubo_prueba_finalizada()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.resultado_validado = TRUE AND NEW.tubo_id IS NOT NULL AND NEW.prueba_id IS NOT NULL THEN
        UPDATE core_tuboprueba 
        SET PruebaFinalizada = TRUE,
            PruebaFechaHoraEnviaDato = NOW()
        WHERE TuboId = NEW.tubo_id 
        AND PruebaId = NEW.prueba_id;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_actualizar_tubo_prueba
    AFTER UPDATE OF resultado_validado ON bacteria_resultado_encabezado
    FOR EACH ROW
    WHEN (NEW.resultado_validado = TRUE)
    EXECUTE FUNCTION update_tubo_prueba_finalizada();
```

```sql
COMMENT ON COLUMN bacteria_resultado_encabezado.equipo_vitek_id IS 'ID del equipo VITEK que procesó la muestra';
COMMENT ON COLUMN bacteria_resultado_encabezado.real_session_id IS 'ID de sesión en sistema REAL';
COMMENT ON COLUMN bacteria_resultado_encabezado.codigo_barra_muestra IS 'Código de barras de la muestra (15 caracteres)';
COMMENT ON COLUMN bacteria_resultado_encabezado.estado_sincronizacion IS 'Estado de sincronización con REAL';
```

---

## Script 03: bacteria_resultado_organismos

```sql
CREATE TABLE bacteria_resultado_organismos (
    id SERIAL PRIMARY KEY,
    encabezado_id INTEGER NOT NULL,
    organismo_id INTEGER NOT NULL,
    probabilidad_identificacion DOUBLE PRECISION,
    es_organismo_principal BOOLEAN DEFAULT FALSE,
    bionumero VARCHAR(50),
    metodo_identificacion VARCHAR(100),
    tiempo_identificacion_horas DOUBLE PRECISION,
    comentarios_identificacion TEXT,
    fecha_identificacion TIMESTAMP DEFAULT NOW(),
    activo BOOLEAN DEFAULT TRUE,
    
    CONSTRAINT fk_bacteria_organismos_encabezado 
        FOREIGN KEY (encabezado_id) 
        REFERENCES bacteria_resultado_encabezado(id) 
        ON DELETE CASCADE,
    CONSTRAINT fk_bacteria_organismos_bacteria 
        FOREIGN KEY (organismo_id) 
        REFERENCES bacteria_catalogo_bacterias(id)
        ON DELETE RESTRICT,
    CONSTRAINT chk_probabilidad_rango 
        CHECK (probabilidad_identificacion >= 0 AND probabilidad_identificacion <= 100)
);
```

```sql
CREATE INDEX idx_bacteria_organismos_encabezado ON bacteria_resultado_organismos(encabezado_id);
CREATE INDEX idx_bacteria_organismos_bacteria ON bacteria_resultado_organismos(organismo_id);
CREATE INDEX idx_bacteria_organismos_principal ON bacteria_resultado_organismos(es_organismo_principal);
CREATE INDEX idx_bacteria_organismos_activo ON bacteria_resultado_organismos(activo);

CREATE UNIQUE INDEX uk_bacteria_organismos_unico 
    ON bacteria_resultado_organismos(encabezado_id, organismo_id) 
    WHERE activo = TRUE;
```

```sql
COMMENT ON TABLE bacteria_resultado_organismos IS 'Múltiples microorganismos identificados en un mismo cultivo';
COMMENT ON COLUMN bacteria_resultado_organismos.probabilidad_identificacion IS 'Porcentaje de confianza en la identificación (0-100)';
COMMENT ON COLUMN bacteria_resultado_organismos.bionumero IS 'Código bioquímico de identificación del VITEK';
```

---

## Script 04: bacteria_mensajes_real

```sql
CREATE TABLE bacteria_mensajes_real (
    id SERIAL PRIMARY KEY,
    orden_numero VARCHAR(20) NOT NULL,
    tubo_codigo_barra VARCHAR(15),
    tipo_mensaje VARCHAR(10) NOT NULL,
    direccion VARCHAR(10) NOT NULL,
    protocolo VARCHAR(10) NOT NULL DEFAULT 'ASTM',
    mensaje_completo TEXT NOT NULL,
    mensaje_procesado JSONB,
    timestamp_mensaje TIMESTAMP DEFAULT NOW(),
    timestamp_procesamiento TIMESTAMP,
    estado_procesamiento VARCHAR(20) DEFAULT 'PENDIENTE',
    error_procesamiento TEXT,
    error_codigo VARCHAR(20),
    reintentos INTEGER DEFAULT 0,
    max_reintentos INTEGER DEFAULT 3,
    procesado_por VARCHAR(100),
    equipo_origen VARCHAR(50),
    session_id VARCHAR(100),
    fecha_ultima_actualiza TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT chk_tipo_mensaje_valido 
        CHECK (tipo_mensaje IN ('REQUEST', 'RESPONSE', 'ACK', 'NACK')),
    CONSTRAINT chk_direccion_valida 
        CHECK (direccion IN ('ENVIO', 'RECIBO')),
    CONSTRAINT chk_protocolo_valido 
        CHECK (protocolo IN ('ASTM', 'HL7')),
    CONSTRAINT chk_estado_procesamiento_valido 
        CHECK (estado_procesamiento IN ('PENDIENTE', 'PROCESADO', 'ERROR', 'DESCARTADO')),
    CONSTRAINT chk_reintentos_validos 
        CHECK (reintentos >= 0 AND reintentos <= max_reintentos),
    CONSTRAINT chk_mensaje_no_vacio 
        CHECK (LENGTH(TRIM(mensaje_completo)) > 0)
);
```

```sql
CREATE INDEX idx_bacteria_mensajes_orden ON bacteria_mensajes_real(orden_numero);
CREATE INDEX idx_bacteria_mensajes_tubo ON bacteria_mensajes_real(tubo_codigo_barra);
CREATE INDEX idx_bacteria_mensajes_timestamp ON bacteria_mensajes_real(timestamp_mensaje);
CREATE INDEX idx_bacteria_mensajes_estado ON bacteria_mensajes_real(estado_procesamiento);
CREATE INDEX idx_bacteria_mensajes_session ON bacteria_mensajes_real(session_id);
```

```sql
CREATE OR REPLACE FUNCTION update_mensaje_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.fecha_ultima_actualiza = NOW();
    
    IF NEW.estado_procesamiento = 'PROCESADO' AND OLD.estado_procesamiento != 'PROCESADO' THEN
        NEW.timestamp_procesamiento = NOW();
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_mensaje_update_timestamp
    BEFORE UPDATE ON bacteria_mensajes_real
    FOR EACH ROW
    EXECUTE FUNCTION update_mensaje_timestamp();
```

```sql
CREATE OR REPLACE FUNCTION incrementar_reintento_mensaje()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.estado_procesamiento = 'ERROR' AND NEW.reintentos < NEW.max_reintentos THEN
        NEW.reintentos = NEW.reintentos + 1;
        NEW.estado_procesamiento = 'PENDIENTE';
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_auto_retry_mensaje
    BEFORE UPDATE OF estado_procesamiento ON bacteria_mensajes_real
    FOR EACH ROW
    WHEN (NEW.estado_procesamiento = 'ERROR')
    EXECUTE FUNCTION incrementar_reintento_mensaje();
```

```sql
COMMENT ON TABLE bacteria_mensajes_real IS 'Trazabilidad de mensajes ASTM/HL7 con sistema REAL';
COMMENT ON COLUMN bacteria_mensajes_real.direccion IS 'ENVIO (hacia REAL) o RECIBO (desde REAL)';
COMMENT ON COLUMN bacteria_mensajes_real.mensaje_procesado IS 'Mensaje parseado en formato JSON';
```

---

## Script 05: Índices Optimización

```sql
CREATE INDEX idx_bacteria_resultado_orden_fecha_compuesto
    ON bacteria_resultado_encabezado(orden_numero, fecha_registro_resultado DESC, resultado_validado);

CREATE INDEX idx_bacteria_resultado_real_session_estado
    ON bacteria_resultado_encabezado(real_session_id, estado_sincronizacion)
    WHERE real_session_id IS NOT NULL;

CREATE INDEX idx_bacteria_pendientes_gobierno
    ON bacteria_resultado_encabezado(fecha_registro_resultado DESC, orden_numero)
    WHERE resultado_validado = TRUE 
    AND enviado_telemedicina = FALSE;

CREATE INDEX idx_bacteria_real_errores
    ON bacteria_resultado_encabezado(fecha_registro_resultado DESC, error_real)
    WHERE estado_sincronizacion = 'ERROR';

CREATE INDEX idx_mensajes_orden_timestamp
    ON bacteria_mensajes_real(orden_numero, timestamp_mensaje DESC);

CREATE INDEX idx_mensajes_pendientes_retry
    ON bacteria_mensajes_real(reintentos, timestamp_mensaje ASC)
    WHERE estado_procesamiento = 'PENDIENTE' AND reintentos < max_reintentos;

CREATE UNIQUE INDEX uk_bacteria_session_message
    ON bacteria_resultado_encabezado(real_session_id, astm_message_id)
    WHERE real_session_id IS NOT NULL AND astm_message_id IS NOT NULL;
```

```sql
ANALYZE bacteria_resultado_encabezado;
ANALYZE bacteria_resultado_detalle;
ANALYZE aux_examenes_real;
ANALYZE bacteria_resultado_organismos;
ANALYZE bacteria_mensajes_real;
```

---

## Script 06: Datos Iniciales

```sql
INSERT INTO aux_examenes_real (examen_analiza_id, examen_real_codigo, examen_real_nombre, tipo_muestra_real, observaciones) VALUES
('UROCU001', 'UROCULTURE', 'CULTIVO DE ORINA', 3, 'Mapeo urocultivo estándar'),
('HEMCU001', 'BLOODCULTURE', 'CULTIVO DE SANGRE', 2, 'Mapeo hemocultivo estándar'),
('COPCU001', 'STOOLCULTURE', 'CULTIVO DE HECES', 1, 'Mapeo coprocultivo estándar'),
('SECCU001', 'WOUNDCULTURE', 'CULTIVO DE HERIDA', 4, 'Mapeo cultivo de secreciones')
ON CONFLICT (examen_analiza_id, examen_real_codigo) DO NOTHING;
```

```sql
INSERT INTO bacteria_tipo_resultado (codigo, nombre) VALUES
('POSITIVO', 'Cultivo Positivo - Microorganismo Identificado'),
('NEGATIVO', 'Cultivo Negativo - Sin Crecimiento'),
('CONTAMINADO', 'Muestra Contaminada'),
('INSUFICIENTE', 'Muestra Insuficiente'),
('PENDIENTE', 'Análisis en Proceso')
ON CONFLICT (codigo) DO NOTHING;
```

```sql
SELECT setval('bacteria_resultado_encabezado_id_seq', 
    COALESCE((SELECT MAX(id) FROM bacteria_resultado_encabezado), 1));
    
SELECT setval('bacteria_resultado_detalle_id_seq', 
    COALESCE((SELECT MAX(id) FROM bacteria_resultado_detalle), 1));
```

---

## Script 07: Validaciones Finales

```sql
CREATE OR REPLACE FUNCTION validar_integracion_real()
RETURNS TABLE(
    componente TEXT,
    estado TEXT,
    detalles TEXT
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        'aux_examenes_real'::TEXT,
        CASE WHEN EXISTS (SELECT 1 FROM aux_examenes_real WHERE activo = TRUE)
             THEN 'OK'::TEXT 
             ELSE 'ERROR'::TEXT 
        END,
        'Mapeos activos: ' || COALESCE((SELECT COUNT(*)::TEXT FROM aux_examenes_real WHERE activo = TRUE), '0');
    
    RETURN QUERY
    SELECT 
        'campos_real_encabezado'::TEXT,
        CASE WHEN EXISTS (
            SELECT 1 FROM information_schema.columns 
            WHERE table_name = 'bacteria_resultado_encabezado' 
            AND column_name = 'real_session_id'
        ) THEN 'OK'::TEXT ELSE 'ERROR'::TEXT END,
        'Campos REAL agregados correctamente';
    
    RETURN QUERY
    SELECT 
        'triggers_activos'::TEXT,
        CASE WHEN EXISTS (
            SELECT 1 FROM information_schema.triggers 
            WHERE trigger_name LIKE '%bacteria%' 
            OR trigger_name LIKE '%aux_examenes%'
        ) THEN 'OK'::TEXT ELSE 'WARNING'::TEXT END,
        'Triggers de automatización verificados';
END;
$$ LANGUAGE plpgsql;
```

```sql
SELECT * FROM validar_integracion_real();
```

---

## Script Rollback

```sql
DROP TABLE IF EXISTS bacteria_mensajes_real CASCADE;
DROP TABLE IF EXISTS bacteria_resultado_organismos CASCADE;
DROP TABLE IF EXISTS aux_examenes_real CASCADE;

ALTER TABLE bacteria_resultado_encabezado 
DROP COLUMN IF EXISTS equipo_vitek_id,
DROP COLUMN IF EXISTS real_session_id,
DROP COLUMN IF EXISTS astm_message_id,
DROP COLUMN IF EXISTS hl7_message_id,
DROP COLUMN IF EXISTS codigo_barra_muestra,
DROP COLUMN IF EXISTS metodo_identificacion,
DROP COLUMN IF EXISTS multiple_organismos,
DROP COLUMN IF EXISTS organismo_principal_id,
DROP COLUMN IF EXISTS organismos_secundarios,
DROP COLUMN IF EXISTS real_raw_data,
DROP COLUMN IF EXISTS estado_sincronizacion,
DROP COLUMN IF EXISTS fecha_envio_real,
DROP COLUMN IF EXISTS fecha_respuesta_real,
DROP COLUMN IF EXISTS error_real,
DROP COLUMN IF EXISTS version_real,
DROP COLUMN IF EXISTS operador_equipo,
DROP COLUMN IF EXISTS numero_corrida,
DROP COLUMN IF EXISTS control_calidad,
DROP COLUMN IF EXISTS comentarios_tecnicos,
DROP COLUMN IF EXISTS requiere_confirmacion,
DROP COLUMN IF EXISTS confirmado_por,
DROP COLUMN IF EXISTS fecha_confirmacion,
DROP COLUMN IF EXISTS fecha_ultima_actualiza,
DROP COLUMN IF EXISTS telemedicina_enviado,
DROP COLUMN IF EXISTS telemedicina_fecha_envio,
DROP COLUMN IF EXISTS telemedicina_response,
DROP COLUMN IF EXISTS telemedicina_error,
DROP COLUMN IF EXISTS telemedicina_reintentos;

SELECT 'ROLLBACK COMPLETADO' as status;
```