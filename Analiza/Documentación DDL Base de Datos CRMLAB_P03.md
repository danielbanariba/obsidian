## 📋 Información General
- **Base de Datos:** CRMLAB_P03
- **Usuario:** dbarrientos
- **Total de Tablas:** 97
- **Fecha de Documentación:** 2025-01-15

---
## Estructura de Tablas

### Módulo de Bacteriología (6 tablas)

#### 1. bacteria_catalogo_antibioticos
**Propósito:** Catálogo maestro de antibióticos utilizados en las pruebas de sensibilidad bacteriana.

```sql
-- Script DDL: Crear tabla de catálogo de antibióticos
CREATE TABLE bacteria_catalogo_antibioticos (
    id INTEGER NOT NULL PRIMARY KEY,
    antibiotico_nombre VARCHAR(100) NOT NULL,
    codigo_loinc VARCHAR(10) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT true
);
```

```sql
-- Índices para optimización
CREATE INDEX idx_bacteria_antibioticos_activo ON bacteria_catalogo_antibioticos(activo);
CREATE INDEX idx_bacteria_antibioticos_loinc ON bacteria_catalogo_antibioticos(codigo_loinc);
```

```sql
-- Comentarios de tabla
COMMENT ON TABLE bacteria_catalogo_antibioticos IS 'Catálogo de antibióticos para pruebas de sensibilidad bacteriana';
COMMENT ON COLUMN bacteria_catalogo_antibioticos.codigo_loinc IS 'Código LOINC estándar internacional';
```

#### 2. bacteria_catalogo_bacterias
**Propósito:** Catálogo de bacterias identificables en el laboratorio.

```sql
-- Script DDL: Crear tabla de catálogo de bacterias
CREATE TABLE bacteria_catalogo_bacterias (
    id INTEGER NOT NULL PRIMARY KEY,
    bacteria_nombre VARCHAR(100) NOT NULL,
    codigo_loinc VARCHAR(10) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT true
);
```

```sql
-- Índices
CREATE INDEX idx_bacteria_bacterias_activo ON bacteria_catalogo_bacterias(activo);
CREATE UNIQUE INDEX idx_bacteria_bacterias_nombre ON bacteria_catalogo_bacterias(bacteria_nombre);
```

```sql
-- Restricciones adicionales
ALTER TABLE bacteria_catalogo_bacterias 
ADD CONSTRAINT chk_bacteria_nombre_no_vacio 
CHECK (LENGTH(TRIM(bacteria_nombre)) > 0);
```

#### 3. bacteria_resultado_encabezado
**Propósito:** Almacena los resultados principales de cultivos bacteriológicos.

```sql
-- Script DDL: Crear tabla de resultados bacteriológicos
CREATE TABLE bacteria_resultado_encabezado (
    id INTEGER NOT NULL DEFAULT nextval('bacteria_resultado_encabezado_id_seq'::regclass),
    orden_numero TEXT,
    examen_id TEXT,
    tubo_id BIGINT,
    prueba_id BIGINT,
    fecha_recepcion TIMESTAMP WITHOUT TIME ZONE,
    fecha_registro_resultado TIMESTAMP WITHOUT TIME ZONE,
    fecha_entrega TIMESTAMP WITHOUT TIME ZONE,
    organismo_seleccionado_id INTEGER,
    origin_muestra_id INTEGER,
    probabilidad_seleccion DOUBLE PRECISION,
    bionumero TEXT,
    horas_tiempo_analisis_identificacion DOUBLE PRECISION,
    horas_tiempo_analisis_sensibilidad DOUBLE PRECISION,
    conclusion TEXT,
    usuario_registro TEXT,
    usuario_validacion TEXT,
    resultado_validado BOOLEAN DEFAULT false,
    blee TEXT,
    carbapenemasas TEXT,
    enviado_telemedicina BOOLEAN DEFAULT false,
    CONSTRAINT pk_bacteria_resultado_encabezado PRIMARY KEY (id)
);
```

```sql
-- Secuencia para ID automático
CREATE SEQUENCE bacteria_resultado_encabezado_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```

```sql
-- Índices para búsquedas frecuentes
CREATE INDEX idx_bacteria_resultado_orden ON bacteria_resultado_encabezado(orden_numero);
CREATE INDEX idx_bacteria_resultado_fecha ON bacteria_resultado_encabezado(fecha_recepcion);
CREATE INDEX idx_bacteria_resultado_validado ON bacteria_resultado_encabezado(resultado_validado);
```

```sql
-- Foreign Keys
ALTER TABLE bacteria_resultado_encabezado
ADD CONSTRAINT fk_bacteria_resultado_organismo 
FOREIGN KEY (organismo_seleccionado_id) 
REFERENCES bacteria_catalogo_bacterias(id);

ALTER TABLE bacteria_resultado_encabezado
ADD CONSTRAINT fk_bacteria_resultado_muestra 
FOREIGN KEY (origin_muestra_id) 
REFERENCES bacteria_catalogo_tipo_muestra(id);
```

### Módulo Core del Sistema

#### 4. core_clientelaboratorio
**Propósito:** Tabla principal de clientes/pacientes del laboratorio.

```sql
-- Script DDL: Crear tabla de clientes
CREATE TABLE core_clientelaboratorio (
    NumeroCliente INTEGER NOT NULL DEFAULT nextval('core_clienteslaboratorio_Numero_de_Cliente_seq'::regclass),
    ClienteTipoId VARCHAR(1),
    ClienteId VARCHAR(40),
    ClientePrimerNombre VARCHAR(60) NOT NULL,
    ClienteSegundoNombre VARCHAR(60),
    ClientePrimerApellido VARCHAR(60) NOT NULL,
    ClienteSegundoApellido VARCHAR(60),
    ClienteFechaNace DATE,
    ClienteGenero VARCHAR(1) CHECK (ClienteGenero IN ('M', 'F', 'O')),
    ClienteTipoSangre VARCHAR(3),
    ClienteDireccion VARCHAR(200),
    ClienteNRC VARCHAR(30),
    ClienteNIT VARCHAR(100),
    ClienteExento BOOLEAN NOT NULL DEFAULT false,
    ClienteEmail VARCHAR(100),
    ClienteClave VARCHAR(120),
    ClienteCelular VARCHAR(30) NOT NULL,
    ClienteTelFijo VARCHAR(30),
    ClienteFechaUltimoAnalsis DATE,
    ClienteActivo BOOLEAN NOT NULL DEFAULT true,
    ClienteFechaVinculacion DATE NOT NULL DEFAULT CURRENT_DATE,
    ClienteVinculadoDoctor BOOLEAN NOT NULL DEFAULT false,
    ClienteDoctorVincula VARCHAR(20) NOT NULL DEFAULT '',
    UsuarioCrea VARCHAR(30) NOT NULL,
    ClienteEmailCreaEnviado BOOLEAN NOT NULL DEFAULT false,
    LabId INTEGER,
    ClienteDepto_id INTEGER,
    ClienteMuni_id INTEGER,
    PaisId INTEGER,
    ClienteTutor_id INTEGER,
    ClientePrimerIngreso BOOLEAN NOT NULL DEFAULT true,
    ClienteDUIAtras TEXT,
    ClienteDUIFrente TEXT,
    cliente_email_validado BOOLEAN DEFAULT false,
    cliente_exento_autorizacion VARCHAR(100),
    cliente_exento_vencimiento DATE,
    cliente_exento_observacion TEXT,
    empleado_empresa BOOLEAN DEFAULT false,
    empresa_id INTEGER,
    cliente_departamento_codigo TEXT,
    cliente_municipio_codigo TEXT,
    cliente_actividad_economica TEXT,
    fecha_ultima_actualiza DATE,
    cliente_retiente BOOLEAN,
    CONSTRAINT pk_clientelaboratorio PRIMARY KEY (NumeroCliente)
);

-- Secuencia
CREATE SEQUENCE core_clienteslaboratorio_Numero_de_Cliente_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;

-- Índices críticos para rendimiento
CREATE INDEX idx_cliente_email ON core_clientelaboratorio(ClienteEmail);
CREATE INDEX idx_cliente_dui ON core_clientelaboratorio(ClienteId);
CREATE INDEX idx_cliente_activo ON core_clientelaboratorio(ClienteActivo);
CREATE INDEX idx_cliente_fecha_nace ON core_clientelaboratorio(ClienteFechaNace);
CREATE INDEX idx_cliente_lab ON core_clientelaboratorio(LabId);

-- Trigger para actualizar fecha de última actualización
CREATE OR REPLACE FUNCTION update_fecha_ultima_actualiza()
RETURNS TRIGGER AS $$
BEGIN
    NEW.fecha_ultima_actualiza = CURRENT_DATE;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_update_fecha_actualiza
BEFORE UPDATE ON core_clientelaboratorio
FOR EACH ROW
EXECUTE FUNCTION update_fecha_ultima_actualiza();
```

#### 5. core_ordenesexamenes
**Propósito:** Tabla principal de órdenes de exámenes del laboratorio.

```sql
-- Script DDL: Crear tabla de órdenes de exámenes
CREATE TABLE core_ordenesexamenes (
    OrdenId VARCHAR(14) NOT NULL,
    OrdenClientePeso DOUBLE PRECISION,
    OrdenClienteAltura DOUBLE PRECISION,
    OrdenClientePresionSis DOUBLE PRECISION,
    OrdenClientePresionDis DOUBLE PRECISION,
    OrdenFecha DATE NOT NULL,
    OrdenHora TIME WITHOUT TIME ZONE NOT NULL,
    OrdenConvenio BOOLEAN NOT NULL DEFAULT false,
    OrdenUsuarioCustodio VARCHAR(100) NOT NULL,
    OrdenMoneda VARCHAR(4),
    OrdenEstatus VARCHAR(3) NOT NULL DEFAULT 'REG',
    OrdenImpresa BOOLEAN NOT NULL DEFAULT false,
    OrdenEpocketMonto DOUBLE PRECISION NOT NULL DEFAULT 0,
    OrdenEpocketPagada BOOLEAN NOT NULL DEFAULT false,
    CentroId INTEGER,
    ClienteNumero INTEGER,
    DoctorId INTEGER,
    OrdenDescuento DOUBLE PRECISION DEFAULT 0,
    OrdenImpuesto DOUBLE PRECISION DEFAULT 0,
    OrdenSubTotal DOUBLE PRECISION DEFAULT 0,
    OrdenEnvioMail BOOLEAN NOT NULL DEFAULT false,
    OrdenExternoTipoPago VARCHAR(1),
    RecolectorID VARCHAR(20),
    OrdenEnviaMail BOOLEAN NOT NULL DEFAULT true,
    OrdenEnviaWhatsapp BOOLEAN NOT NULL DEFAULT true,
    OrdenWhatsappEnviado BOOLEAN NOT NULL DEFAULT false,
    OrdenCultivoMarcada BOOLEAN NOT NULL DEFAULT false,
    OrdenPlataformaExterna BOOLEAN NOT NULL DEFAULT false,
    DescuentoId INTEGER,
    OrdenHoraRecibeAnalisis TIMESTAMP WITH TIME ZONE,
    OrdenGPSLat DOUBLE PRECISION,
    OrdenGPSLong DOUBLE PRECISION,
    OrdenEnviaMailDoctor BOOLEAN NOT NULL DEFAULT false,
    OrdenGenPlataformaExterna BOOLEAN NOT NULL DEFAULT false,
    OrdenEsCredito BOOLEAN NOT NULL DEFAULT false,
    OrdenGastoInsumo DOUBLE PRECISION DEFAULT 0,
    OrdenGastoReactivo DOUBLE PRECISION DEFAULT 0,
    OrdenRazonCancela TEXT,
    OrdenEstatusCobro VARCHAR(1) NOT NULL DEFAULT 'P',
    OrdenNumDocumento VARCHAR(20),
    OrdenLugar VARCHAR(1),
    OrdenAseguradora_id INTEGER,
    OrdenConvenioEmp_id INTEGER,
    OrdenPoliza_id INTEGER,
    OrdenSecretaria_id INTEGER,
    OrdenMuestraHistorial BOOLEAN NOT NULL DEFAULT false,
    OrdenFechaResultado TIMESTAMP WITH TIME ZONE,
    OrdenTipoDocumento VARCHAR(3),
    orden_total NUMERIC,
    orden_crm2 BOOLEAN,
    resultado_ingles BOOLEAN DEFAULT false,
    vlink TEXT,
    vlink_creado TIMESTAMP WITHOUT TIME ZONE,
    es_boleta_medica_analiza BOOLEAN DEFAULT false,
    es_orden_contable BOOLEAN DEFAULT false,
    es_orden_gobierno BOOLEAN DEFAULT false,
    num_orden_telemedicina TEXT,
    reenvios_telemedicina INTEGER DEFAULT 0,
    impuesto_retenido NUMERIC DEFAULT 0,
    CONSTRAINT pk_ordenesexamenes PRIMARY KEY (OrdenId)
);

-- Índices para consultas frecuentes
CREATE INDEX idx_orden_fecha ON core_ordenesexamenes(OrdenFecha);
CREATE INDEX idx_orden_cliente ON core_ordenesexamenes(ClienteNumero);
CREATE INDEX idx_orden_estatus ON core_ordenesexamenes(OrdenEstatus);
CREATE INDEX idx_orden_centro ON core_ordenesexamenes(CentroId);
CREATE INDEX idx_orden_fecha_resultado ON core_ordenesexamenes(OrdenFechaResultado);

-- Constraint para validar estatus
ALTER TABLE core_ordenesexamenes
ADD CONSTRAINT chk_orden_estatus 
CHECK (OrdenEstatus IN ('REG', 'PRO', 'ENT', 'CAN', 'VAL'));

-- Foreign Keys
ALTER TABLE core_ordenesexamenes
ADD CONSTRAINT fk_orden_cliente 
FOREIGN KEY (ClienteNumero) 
REFERENCES core_clientelaboratorio(NumeroCliente);

ALTER TABLE core_ordenesexamenes
ADD CONSTRAINT fk_orden_centro 
FOREIGN KEY (CentroId) 
REFERENCES core_centroservicio(CentroId);
```



---
## Scripts para Producción

### 1. Script de Migración Segura

```sql
-- Script: Migración segura con validaciones
BEGIN;

-- Verificar que no existan las tablas
DO $$
BEGIN
    IF EXISTS (SELECT 1 FROM information_schema.tables WHERE table_name = 'bacteria_catalogo_antibioticos') THEN
        RAISE EXCEPTION 'La tabla bacteria_catalogo_antibioticos ya existe';
    END IF;
END $$;

-- Crear tablas aquí...

-- Validar integridad
DO $$
DECLARE
    v_count INTEGER;
BEGIN
    SELECT COUNT(*) INTO v_count 
    FROM information_schema.tables 
    WHERE table_schema = 'public' 
    AND table_name LIKE 'bacteria_%';
    
    IF v_count != 6 THEN
        RAISE EXCEPTION 'No se crearon todas las tablas de bacteriología';
    END IF;
END $$;

COMMIT;
```

### 2. Script de Respaldo Antes de Cambios

```sql
-- Script: Crear respaldo de tablas antes de modificaciones
CREATE SCHEMA IF NOT EXISTS backup_20250115;

-- Respaldar tabla antes de modificación
CREATE TABLE backup_20250115.core_clientelaboratorio AS 
SELECT * FROM core_clientelaboratorio;

-- Agregar timestamp al respaldo
ALTER TABLE backup_20250115.core_clientelaboratorio 
ADD COLUMN backup_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
```

### 3. Script de Rollback

```sql
-- Script: Rollback en caso de error
BEGIN;

-- Restaurar desde backup
TRUNCATE TABLE core_clientelaboratorio;
INSERT INTO core_clientelaboratorio 
SELECT * FROM backup_20250115.core_clientelaboratorio;

-- Verificar integridad
DO $$
DECLARE
    v_count_original INTEGER;
    v_count_restored INTEGER;
BEGIN
    SELECT COUNT(*) INTO v_count_original 
    FROM backup_20250115.core_clientelaboratorio;
    
    SELECT COUNT(*) INTO v_count_restored 
    FROM core_clientelaboratorio;
    
    IF v_count_original != v_count_restored THEN
        RAISE EXCEPTION 'Error en restauración: cantidad de registros no coincide';
    END IF;
END $$;

COMMIT;
```



---
## Scripts de Mantenimiento

### Análisis de Tablas
```sql
-- Actualizar estadísticas
ANALYZE bacteria_catalogo_antibioticos;
ANALYZE bacteria_catalogo_bacterias;
ANALYZE bacteria_resultado_encabezado;

-- Reindexar tablas críticas
REINDEX TABLE core_clientelaboratorio;
REINDEX TABLE core_ordenesexamenes;
```


### Monitoreo de Espacio
```sql
-- Ver tamaño de tablas
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;
```