## Análisis del Mapeo HL7 vs Tablas de Bacteriología

### 1. **Análisis de Tablas Existentes**
```sql
-- TABLAS EXISTENTES:
1. bacteria_catalogo_antibioticos (id, antibiotico_nombre, codigo_loinc, activo)
2. bacteria_catalogo_bacterias (id, bacteria_nombre, codigo_loinc, activo)
3. bacteria_catalogo_tipo_muestra (id, muestra_nombre, codigo_loinc, activo)
4. bacteria_resultado_encabezado (campos de orden, muestra, tiempos, validación)
5. bacteria_resultado_detalle (encabezado_id, antibiotico_id, cmi, interpretacion)
6. bacteria_tipo_resultado (id, codigo, nombre)
```

### 2. **Campos Típicos Requeridos por HL7 para Bacteriología**

Las tramas HL7 para resultados de bacteriología típicamente incluyen:

#### **Segmento MSH (Message Header)**

- Sending Application
- Sending Facility
- Message Date/Time
- Message Type
- Message Control ID

#### **Segmento PID (Patient Identification)**

- Patient ID
- Patient Name
- Date of Birth
- Sex
- Patient Address

#### **Segmento OBR (Observation Request)**

- Placer Order Number
- Filler Order Number
- Universal Service ID
- Observation Date/Time
- Specimen Source
- Ordering Provider

#### **Segmento OBX (Observation Result)**

- Observation Identifier
- Observation Value
- Units
- Reference Range
- Observation Result Status
- Date/Time of Observation

### 3. **Mapeo y Campos Faltantes**

#### **Agregar la tabla `bacteria_resultado_encabezado`**

```sql
ALTER TABLE bacteria_resultado_encabezado ADD COLUMN IF NOT EXISTS:
    -- Campos para trazabilidad HL7
    mensaje_hl7_id VARCHAR(50),                    -- Message Control ID
    fecha_mensaje_hl7 TIMESTAMP,                  -- Timestamp del mensaje HL7
    estado_mensaje_hl7 VARCHAR(20),               -- ACK, NACK, ERROR
    
    -- Campos del equipo/sistema origen
    equipo_origen VARCHAR(100),                    -- VITEK, otro equipo
    sistema_origen VARCHAR(50),                    -- REAL, otro middleware
    version_protocolo VARCHAR(20),                 -- HL7 v2.5, etc
    
    -- Campos adicionales del cultivo
    metodo_identificacion VARCHAR(100),           -- Método usado para identificar
    nivel_confianza_identificacion DECIMAL(5,2),  -- % confianza
    observaciones_cultivo TEXT,                    -- Observaciones generales
    
    -- Campos para múltiples organismos
    es_cultivo_polimicrobiano BOOLEAN,            -- Si tiene múltiples organismos
    cantidad_organismos INTEGER,                   -- Número de organismos
    
    -- Tracking de procesamiento
    fecha_recepcion_hl7 TIMESTAMP,                -- Cuando llegó el mensaje
    fecha_procesamiento_hl7 TIMESTAMP,            -- Cuando se procesó
    reintentos_procesamiento INTEGER DEFAULT 0;    -- Número de reintentos
```

#### **B. Nueva tabla para manejo de múltiples organismos:**

```sql
CREATE TABLE IF NOT EXISTS bacteria_organismos_identificados (
    id SERIAL PRIMARY KEY,
    encabezado_id INTEGER NOT NULL REFERENCES bacteria_resultado_encabezado(id),
    bacteria_id INTEGER REFERENCES bacteria_catalogo_bacterias(id),
    
    -- Identificación del organismo
    codigo_organismo_real VARCHAR(50),             -- Código en sistema REAL
    nombre_organismo_real VARCHAR(200),            -- Nombre como viene de REAL
    nivel_confianza DECIMAL(5,2),                  -- % confianza identificación
    es_organismo_principal BOOLEAN DEFAULT FALSE,  -- Si es el principal
    
    -- Cuantificación
    recuento_colonias VARCHAR(50),                 -- UFC/ml, etc
    cuantificacion_metodo VARCHAR(100),           -- Método de cuantificación
    
    -- Metadata
    fecha_identificacion TIMESTAMP,
    observaciones TEXT,
    activo BOOLEAN DEFAULT TRUE
);
```

#### **C. Agragar la tabla `bacteria_resultado_detalle`**

```sql
ALTER TABLE bacteria_resultado_detalle ADD COLUMN IF NOT EXISTS:
    -- Campos adicionales del antibiograma
    organismo_id INTEGER REFERENCES bacteria_organismos_identificados(id),
    metodo_prueba VARCHAR(50),                     -- MIC, Disco, E-test
    diametro_halo VARCHAR(20),                     -- Para método disco
    codigo_antibiotico_real VARCHAR(50),          -- Código en sistema REAL
    
    -- Interpretación extendida
    interpretacion_codigo VARCHAR(10),             -- S, I, R, NS
    mecanismo_resistencia TEXT,                    -- Mecanismos detectados
    
    -- Tracking
    fecha_resultado TIMESTAMP,
    equipo_prueba VARCHAR(100);                    -- Equipo que realizó la prueba
```

#### **D. Nueva tabla para mapeo de códigos:**

```sql
CREATE TABLE IF NOT EXISTS bacteria_mapeo_codigos (
    id SERIAL PRIMARY KEY,
    tipo_catalogo VARCHAR(50) NOT NULL,           -- 'bacteria', 'antibiotico', 'muestra'
    codigo_crm VARCHAR(50),                        -- Código en nuestro sistema
    codigo_real VARCHAR(50),                       -- Código en sistema REAL
    nombre_real VARCHAR(200),                      -- Nombre en sistema REAL
    
    -- Mapeo
    id_catalogo_local INTEGER,                     -- ID en nuestros catálogos
    mapeo_validado BOOLEAN DEFAULT FALSE,
    fecha_mapeo TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_mapeo VARCHAR(100),
    
    -- Control
    activo BOOLEAN DEFAULT TRUE,
    observaciones TEXT,
    
    UNIQUE(tipo_catalogo, codigo_real)
);
```

#### **E. Nueva tabla para logs de mensajes HL7:**

```sql
CREATE TABLE IF NOT EXISTS bacteria_hl7_mensajes_log (
    id SERIAL PRIMARY KEY,
    mensaje_id VARCHAR(50) NOT NULL,               -- Message Control ID
    tipo_mensaje VARCHAR(20),                      -- ORM, ORU, etc
    direccion VARCHAR(10),                         -- IN, OUT
    
    -- Contenido
    mensaje_completo TEXT,                         -- Mensaje HL7 completo
    mensaje_parseado JSONB,                        -- Mensaje parseado a JSON
    
    -- Relaciones
    orden_id VARCHAR(14),
    tubo_id BIGINT,
    encabezado_bacterias_id INTEGER,
    
    -- Estado
    estado_procesamiento VARCHAR(20),              -- RECEIVED, PROCESSING, COMPLETED, ERROR
    mensaje_error TEXT,
    
    -- Timestamps
    fecha_recepcion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_procesamiento TIMESTAMP,
    
    -- Índices para búsqueda rápida
    INDEX idx_mensaje_id (mensaje_id),
    INDEX idx_orden_id (orden_id),
    INDEX idx_fecha_recepcion (fecha_recepcion)
);
```

### 4. **Script completo para agregar campos:**

```sql
-- =====================================================
-- SCRIPT DE ADICIÓN DE CAMPOS PARA INTEGRACIÓN HL7
-- Proyecto: Integración REAL Bacteriología
-- NO MODIFICA campos existentes, solo AGREGA
-- =====================================================

-- 1. Agregar campos a bacteria_resultado_encabezado
ALTER TABLE bacteria_resultado_encabezado 
ADD COLUMN IF NOT EXISTS mensaje_hl7_id VARCHAR(50),
ADD COLUMN IF NOT EXISTS fecha_mensaje_hl7 TIMESTAMP,
ADD COLUMN IF NOT EXISTS estado_mensaje_hl7 VARCHAR(20),
ADD COLUMN IF NOT EXISTS equipo_origen VARCHAR(100),
ADD COLUMN IF NOT EXISTS sistema_origen VARCHAR(50),
ADD COLUMN IF NOT EXISTS version_protocolo VARCHAR(20),
ADD COLUMN IF NOT EXISTS metodo_identificacion VARCHAR(100),
ADD COLUMN IF NOT EXISTS nivel_confianza_identificacion DECIMAL(5,2),
ADD COLUMN IF NOT EXISTS observaciones_cultivo TEXT,
ADD COLUMN IF NOT EXISTS es_cultivo_polimicrobiano BOOLEAN DEFAULT FALSE,
ADD COLUMN IF NOT EXISTS cantidad_organismos INTEGER DEFAULT 1,
ADD COLUMN IF NOT EXISTS fecha_recepcion_hl7 TIMESTAMP,
ADD COLUMN IF NOT EXISTS fecha_procesamiento_hl7 TIMESTAMP,
ADD COLUMN IF NOT EXISTS reintentos_procesamiento INTEGER DEFAULT 0;

-- 2. Agregar campos a bacteria_resultado_detalle
ALTER TABLE bacteria_resultado_detalle
ADD COLUMN IF NOT EXISTS organismo_id INTEGER,
ADD COLUMN IF NOT EXISTS metodo_prueba VARCHAR(50),
ADD COLUMN IF NOT EXISTS diametro_halo VARCHAR(20),
ADD COLUMN IF NOT EXISTS codigo_antibiotico_real VARCHAR(50),
ADD COLUMN IF NOT EXISTS interpretacion_codigo VARCHAR(10),
ADD COLUMN IF NOT EXISTS mecanismo_resistencia TEXT,
ADD COLUMN IF NOT EXISTS fecha_resultado TIMESTAMP,
ADD COLUMN IF NOT EXISTS equipo_prueba VARCHAR(100);
```

## Análisis de Tablas Existentes

### Tablas Actuales de Bacteriología

| Tabla                            | Campos Principales                                 | Propósito                   |
| -------------------------------- | -------------------------------------------------- | --------------------------- |
| `bacteria_catalogo_antibioticos` | id, antibiotico_nombre, codigo_loinc, activo       | Catálogo de antibióticos    |
| `bacteria_catalogo_bacterias`    | id, bacteria_nombre, codigo_loinc, activo          | Catálogo de microorganismos |
| `bacteria_catalogo_tipo_muestra` | id, muestra_nombre, codigo_loinc, activo           | Tipos de muestra (1-4)      |
| `bacteria_resultado_encabezado`  | Orden, muestra, tiempos, validación                | Encabezado de resultados    |
| `bacteria_resultado_detalle`     | encabezado_id, antibiotico_id, cmi, interpretacion | Antibiogramas               |
| `bacteria_tipo_resultado`        | id, codigo, nombre                                 | Tipos de resultado          |

## Campos Requeridos por HL7

### Segmentos HL7 para Bacteriología

#### MSH - Message Header

- `MSH-10`: Message Control ID
- `MSH-7`: Message Date/Time
- `MSH-3`: Sending Application
- `MSH-4`: Sending Facility

#### PID - Patient Identification

- `PID-3`: Patient ID
- `PID-5`: Patient Name
- `PID-7`: Date of Birth
- `PID-8`: Sex

#### OBR - Observation Request

- `OBR-2`: Placer Order Number
- `OBR-3`: Filler Order Number
- `OBR-4`: Universal Service ID
- `OBR-7`: Observation Date/Time
- `OBR-15`: Specimen Source

#### OBX - Observation Result

- `OBX-3`: Observation Identifier
- `OBX-5`: Observation Value
- `OBX-6`: Units
- `OBX-7`: Reference Range
- `OBX-11`: Observation Result Status

## Campos y Tablas a Agregar

### 1. Campos Nuevos para `bacteria_resultado_encabezado`

```sql
-- Campos para trazabilidad HL7
ALTER TABLE bacteria_resultado_encabezado ADD COLUMN IF NOT EXISTS
    mensaje_hl7_id VARCHAR(50),                    -- MSH-10: Message Control ID
    fecha_mensaje_hl7 TIMESTAMP,                  -- MSH-7: Timestamp del mensaje
    estado_mensaje_hl7 VARCHAR(20),               -- Estado: ACK, NACK, ERROR
    
    -- Campos del equipo/sistema origen
    equipo_origen VARCHAR(100),                    -- VITEK, otro equipo
    sistema_origen VARCHAR(50),                    -- REAL, otro middleware
    version_protocolo VARCHAR(20),                 -- HL7 v2.5, etc
    
    -- Campos adicionales del cultivo
    metodo_identificacion VARCHAR(100),           -- Método de identificación
    nivel_confianza_identificacion DECIMAL(5,2),  -- % confianza
    observaciones_cultivo TEXT,                    -- Observaciones generales
    
    -- Campos para múltiples organismos
    es_cultivo_polimicrobiano BOOLEAN DEFAULT FALSE,
    cantidad_organismos INTEGER DEFAULT 1,
    
    -- Tracking de procesamiento
    fecha_recepcion_hl7 TIMESTAMP,
    fecha_procesamiento_hl7 TIMESTAMP,
    reintentos_procesamiento INTEGER DEFAULT 0;
```

### 2. Nueva Tabla: `bacteria_organismos_identificados`

```sql
CREATE TABLE IF NOT EXISTS bacteria_organismos_identificados (
    id SERIAL PRIMARY KEY,
    encabezado_id INTEGER NOT NULL REFERENCES bacteria_resultado_encabezado(id),
    bacteria_id INTEGER REFERENCES bacteria_catalogo_bacterias(id),
    
    -- Identificación del organismo
    codigo_organismo_real VARCHAR(50),             -- Código en sistema REAL
    nombre_organismo_real VARCHAR(200),            -- Nombre desde REAL
    nivel_confianza DECIMAL(5,2),                  -- % confianza
    es_organismo_principal BOOLEAN DEFAULT FALSE,
    
    -- Cuantificación
    recuento_colonias VARCHAR(50),                 -- UFC/ml, etc
    cuantificacion_metodo VARCHAR(100),
    
    -- Metadata
    fecha_identificacion TIMESTAMP,
    observaciones TEXT,
    activo BOOLEAN DEFAULT TRUE
);

-- Índices para rendimiento
CREATE INDEX idx_bacteria_org_encabezado ON bacteria_organismos_identificados(encabezado_id);
CREATE INDEX idx_bacteria_org_bacteria ON bacteria_organismos_identificados(bacteria_id);
```

### 3. Campos Nuevos para `bacteria_resultado_detalle`

```sql
ALTER TABLE bacteria_resultado_detalle ADD COLUMN IF NOT EXISTS
    -- Relación con organismo específico
    organismo_id INTEGER REFERENCES bacteria_organismos_identificados(id),
    
    -- Detalles del antibiograma
    metodo_prueba VARCHAR(50),                     -- MIC, Disco, E-test
    diametro_halo VARCHAR(20),                     -- Para método disco
    codigo_antibiotico_real VARCHAR(50),           -- Código en REAL
    
    -- Interpretación extendida
    interpretacion_codigo VARCHAR(10),             -- S, I, R, NS
    mecanismo_resistencia TEXT,                    -- BLEE, Carbapenemasa, etc
    
    -- Tracking
    fecha_resultado TIMESTAMP,
    equipo_prueba VARCHAR(100);
```

### 4. Nueva Tabla: `bacteria_mapeo_codigos`

```sql
CREATE TABLE IF NOT EXISTS bacteria_mapeo_codigos (
    id SERIAL PRIMARY KEY,
    tipo_catalogo VARCHAR(50) NOT NULL,           -- 'bacteria', 'antibiotico', 'muestra'
    codigo_crm VARCHAR(50),                        -- Código en CRM2
    codigo_real VARCHAR(50),                       -- Código en REAL
    nombre_real VARCHAR(200),                      -- Nombre en REAL
    
    -- Mapeo
    id_catalogo_local INTEGER,                     -- ID en catálogos locales
    mapeo_validado BOOLEAN DEFAULT FALSE,
    fecha_mapeo TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_mapeo VARCHAR(100),
    
    -- Control
    activo BOOLEAN DEFAULT TRUE,
    observaciones TEXT,
    
    UNIQUE(tipo_catalogo, codigo_real)
);

-- Índices para búsquedas rápidas
CREATE INDEX idx_mapeo_tipo_codigo ON bacteria_mapeo_codigos(tipo_catalogo, codigo_real);
CREATE INDEX idx_mapeo_codigo_crm ON bacteria_mapeo_codigos(codigo_crm);
```

### 5. Nueva Tabla: `bacteria_hl7_mensajes_log`

```sql
CREATE TABLE IF NOT EXISTS bacteria_hl7_mensajes_log (
    id SERIAL PRIMARY KEY,
    mensaje_id VARCHAR(50) NOT NULL,               -- Message Control ID
    tipo_mensaje VARCHAR(20),                      -- ORM, ORU, etc
    direccion VARCHAR(10),                         -- IN, OUT
    
    -- Contenido
    mensaje_completo TEXT,                         -- Mensaje HL7 raw
    mensaje_parseado JSONB,                        -- Mensaje en JSON
    
    -- Relaciones
    orden_id VARCHAR(14),
    tubo_id BIGINT,
    encabezado_bacterias_id INTEGER,
    
    -- Estado
    estado_procesamiento VARCHAR(20),              -- RECEIVED, PROCESSING, COMPLETED, ERROR
    mensaje_error TEXT,
    
    -- Timestamps
    fecha_recepcion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_procesamiento TIMESTAMP
);

-- Índices para búsqueda eficiente
CREATE INDEX idx_hl7_mensaje_id ON bacteria_hl7_mensajes_log(mensaje_id);
CREATE INDEX idx_hl7_orden_id ON bacteria_hl7_mensajes_log(orden_id);
CREATE INDEX idx_hl7_fecha ON bacteria_hl7_mensajes_log(fecha_recepcion);
```

## Script DDL Completo

```sql
-- =====================================================
-- SCRIPT DE ADICIÓN DE CAMPOS PARA INTEGRACIÓN HL7
-- Proyecto: Integración REAL Bacteriología
-- Fecha: 2025-01-02
-- Autor: Daniel Barrientos
-- 
-- IMPORTANTE: Este script NO MODIFICA campos existentes
--            Solo AGREGA nuevos campos y tablas
-- =====================================================

BEGIN;

-- 1. AGREGAR CAMPOS A bacteria_resultado_encabezado
ALTER TABLE bacteria_resultado_encabezado 
ADD COLUMN IF NOT EXISTS mensaje_hl7_id VARCHAR(50),
ADD COLUMN IF NOT EXISTS fecha_mensaje_hl7 TIMESTAMP,
ADD COLUMN IF NOT EXISTS estado_mensaje_hl7 VARCHAR(20),
ADD COLUMN IF NOT EXISTS equipo_origen VARCHAR(100),
ADD COLUMN IF NOT EXISTS sistema_origen VARCHAR(50),
ADD COLUMN IF NOT EXISTS version_protocolo VARCHAR(20),
ADD COLUMN IF NOT EXISTS metodo_identificacion VARCHAR(100),
ADD COLUMN IF NOT EXISTS nivel_confianza_identificacion DECIMAL(5,2),
ADD COLUMN IF NOT EXISTS observaciones_cultivo TEXT,
ADD COLUMN IF NOT EXISTS es_cultivo_polimicrobiano BOOLEAN DEFAULT FALSE,
ADD COLUMN IF NOT EXISTS cantidad_organismos INTEGER DEFAULT 1,
ADD COLUMN IF NOT EXISTS fecha_recepcion_hl7 TIMESTAMP,
ADD COLUMN IF NOT EXISTS fecha_procesamiento_hl7 TIMESTAMP,
ADD COLUMN IF NOT EXISTS reintentos_procesamiento INTEGER DEFAULT 0;

-- 2. AGREGAR CAMPOS A bacteria_resultado_detalle
ALTER TABLE bacteria_resultado_detalle
ADD COLUMN IF NOT EXISTS organismo_id INTEGER,
ADD COLUMN IF NOT EXISTS metodo_prueba VARCHAR(50),
ADD COLUMN IF NOT EXISTS diametro_halo VARCHAR(20),
ADD COLUMN IF NOT EXISTS codigo_antibiotico_real VARCHAR(50),
ADD COLUMN IF NOT EXISTS interpretacion_codigo VARCHAR(10),
ADD COLUMN IF NOT EXISTS mecanismo_resistencia TEXT,
ADD COLUMN IF NOT EXISTS fecha_resultado TIMESTAMP,
ADD COLUMN IF NOT EXISTS equipo_prueba VARCHAR(100);

-- 3. CREAR TABLA bacteria_organismos_identificados
CREATE TABLE IF NOT EXISTS bacteria_organismos_identificados (
    id SERIAL PRIMARY KEY,
    encabezado_id INTEGER NOT NULL REFERENCES bacteria_resultado_encabezado(id),
    bacteria_id INTEGER REFERENCES bacteria_catalogo_bacterias(id),
    codigo_organismo_real VARCHAR(50),
    nombre_organismo_real VARCHAR(200),
    nivel_confianza DECIMAL(5,2),
    es_organismo_principal BOOLEAN DEFAULT FALSE,
    recuento_colonias VARCHAR(50),
    cuantificacion_metodo VARCHAR(100),
    fecha_identificacion TIMESTAMP,
    observaciones TEXT,
    activo BOOLEAN DEFAULT TRUE
);

-- 4. CREAR TABLA bacteria_mapeo_codigos
CREATE TABLE IF NOT EXISTS bacteria_mapeo_codigos (
    id SERIAL PRIMARY KEY,
    tipo_catalogo VARCHAR(50) NOT NULL,
    codigo_crm VARCHAR(50),
    codigo_real VARCHAR(50),
    nombre_real VARCHAR(200),
    id_catalogo_local INTEGER,
    mapeo_validado BOOLEAN DEFAULT FALSE,
    fecha_mapeo TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_mapeo VARCHAR(100),
    activo BOOLEAN DEFAULT TRUE,
    observaciones TEXT,
    UNIQUE(tipo_catalogo, codigo_real)
);

-- 5. CREAR TABLA bacteria_hl7_mensajes_log
CREATE TABLE IF NOT EXISTS bacteria_hl7_mensajes_log (
    id SERIAL PRIMARY KEY,
    mensaje_id VARCHAR(50) NOT NULL,
    tipo_mensaje VARCHAR(20),
    direccion VARCHAR(10),
    mensaje_completo TEXT,
    mensaje_parseado JSONB,
    orden_id VARCHAR(14),
    tubo_id BIGINT,
    encabezado_bacterias_id INTEGER,
    estado_procesamiento VARCHAR(20),
    mensaje_error TEXT,
    fecha_recepcion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_procesamiento TIMESTAMP
);

-- 6. CREAR ÍNDICES PARA OPTIMIZACIÓN
CREATE INDEX IF NOT EXISTS idx_bacteria_org_encabezado ON bacteria_organismos_identificados(encabezado_id);
CREATE INDEX IF NOT EXISTS idx_bacteria_org_bacteria ON bacteria_organismos_identificados(bacteria_id);
CREATE INDEX IF NOT EXISTS idx_mapeo_tipo_codigo ON bacteria_mapeo_codigos(tipo_catalogo, codigo_real);
CREATE INDEX IF NOT EXISTS idx_mapeo_codigo_crm ON bacteria_mapeo_codigos(codigo_crm);
CREATE INDEX IF NOT EXISTS idx_hl7_mensaje_id ON bacteria_hl7_mensajes_log(mensaje_id);
CREATE INDEX IF NOT EXISTS idx_hl7_orden_id ON bacteria_hl7_mensajes_log(orden_id);
CREATE INDEX IF NOT EXISTS idx_hl7_fecha ON bacteria_hl7_mensajes_log(fecha_recepcion);

-- 7. AGREGAR FOREIGN KEY CONSTRAINT
ALTER TABLE bacteria_resultado_detalle
ADD CONSTRAINT fk_bacteria_detalle_organismo 
FOREIGN KEY (organismo_id) 
REFERENCES bacteria_organismos_identificados(id);

COMMIT;
```

## Mapeo de Campos HL7 a Tablas

### Mapeo Principal

|Campo HL7|Tabla/Campo Destino|Notas|
|---|---|---|
|MSH-10|bacteria_resultado_encabezado.mensaje_hl7_id|ID único del mensaje|
|MSH-7|bacteria_resultado_encabezado.fecha_mensaje_hl7|Timestamp del mensaje|
|OBR-2|core_ordenesexamenes.OrdenId|Número de orden|
|OBR-3|core_ordentubos.TuboCodigoBarra|Código de barras|
|OBX-3|bacteria_catalogo_bacterias.codigo_loinc|Código del organismo|
|OBX-5|bacteria_organismos_identificados.nombre_organismo_real|Nombre del organismo|
|OBX-11|bacteria_resultado_encabezado.resultado_validado|Estado del resultado|

## Consideraciones Importantes

### 1. Integridad con Sistema Gobierno

- Los campos agregados no afectan la integración existente
- Se mantiene compatibilidad con tablas core

### 2. Manejo de Múltiples Organismos

- Nueva tabla `bacteria_organismos_identificados` permite N organismos por cultivo
- Campo `es_organismo_principal` identifica el predominante

### 3. Trazabilidad Completa

- Tabla de logs HL7 permite auditoría completa
- Campos de timestamp en cada etapa del proceso

### 4. Mapeo de Catálogos

- Tabla dedicada para mapeo REAL ↔ CRM2
- Soporta 7,429 microorganismos de REAL
- Validación manual por expertos del laboratorio

### 5. Valores Especiales

- CMI: Se guarda concatenado (ej: "<=0.12")
- BLEE/Carbapenemasa: TRUE → "POSITIVO", FALSE → "NEGATIVO"
- Tipos de muestra válidos: 1-4

## Próximos Pasos

1. [ ]  Ejecutar script DDL en ambiente de desarrollo
2. [ ]  Validar con Natalia/Lidia campos adicionales requeridos
3. [ ]  Actualizar API de Daniel Barriba con nuevos campos
4. [ ]  Documentar mapeo específico REAL ↔ CRM2
5. [ ]  Crear procedimientos de carga inicial de catálogos
