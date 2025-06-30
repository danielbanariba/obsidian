## Estructura de Tablas

### Módulo de Bacteriología (6 tablas)

#### 1. bacteria_catalogo_antibioticos
Catálogo maestro de antibióticos utilizados en las pruebas de sensibilidad bacteriana.

```sql
-- Crear tabla de catálogo de antibióticos
CREATE TABLE bacteria_catalogo_antibioticos (
    id INTEGER NOT NULL PRIMARY KEY,
    antibiotico_nombre VARCHAR(100) NOT NULL,
    codigo_loinc VARCHAR(10) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT true
);
```

```sql
-- Comentarios de tabla
COMMENT ON TABLE bacteria_catalogo_antibioticos IS 'Catálogo de antibióticos para pruebas de sensibilidad bacteriana';
COMMENT ON COLUMN bacteria_catalogo_antibioticos.codigo_loinc IS 'Código LOINC estándar internacional';
```


#### 2. bacteria_catalogo_bacterias
Catálogo de bacterias identificables en el laboratorio

```sql
-- Crear tabla de catálogo de bacterias
CREATE TABLE bacteria_catalogo_bacterias (
    id INTEGER NOT NULL PRIMARY KEY,
    bacteria_nombre VARCHAR(100) NOT NULL,
    codigo_loinc VARCHAR(10) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT true
);
```

```sql
-- Índice ÚNICO: Previene duplicados y mejora búsquedas por nombre
CREATE UNIQUE INDEX idx_bacteria_bacterias_nombre 
ON bacteria_catalogo_bacterias(bacteria_nombre);
```

```sql
-- Índice para código LOINC si se busca frecuentemente
CREATE INDEX idx_bacteria_bacterias_loinc 
ON bacteria_catalogo_bacterias(codigo_loinc);
```

```sql
-- Validación: El nombre no puede estar vacío o ser solo espacios
ALTER TABLE bacteria_catalogo_bacterias 
ADD CONSTRAINT chk_bacteria_nombre_no_vacio 
CHECK (LENGTH(TRIM(bacteria_nombre)) > 0);
```

```sql
-- Documentación
COMMENT ON TABLE bacteria_catalogo_bacterias 
IS 'Catálogo maestro de bacterias identificables en cultivos';

COMMENT ON COLUMN bacteria_catalogo_bacterias.bacteria_nombre 
IS 'Nombre científico de la bacteria (debe ser único)';
```


#### 3. bacteria_resultado_encabezado
Almacena los resultados principales de cultivos bacteriológicos.

```sql
-- Tabla principal
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
-- Secuencia (CACHE 1 para datos médicos críticos)
CREATE SEQUENCE bacteria_resultado_encabezado_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```

```sql
-- Índices SOLO los necesarios
CREATE INDEX idx_bacteria_resultado_orden ON bacteria_resultado_encabezado(orden_numero);
CREATE INDEX idx_bacteria_resultado_fecha ON bacteria_resultado_encabezado(fecha_recepcion);
```

```sql
-- Índice parcial para resultados pendientes
CREATE INDEX idx_bacteria_pendientes ON bacteria_resultado_encabezado(id, fecha_recepcion) 
WHERE resultado_validado = false;
```

```sql
-- Foreign Keys con comportamiento definido
ALTER TABLE bacteria_resultado_encabezado
ADD CONSTRAINT fk_bacteria_resultado_organismo 
FOREIGN KEY (organismo_seleccionado_id) 
REFERENCES bacteria_catalogo_bacterias(id)
ON DELETE RESTRICT;

ALTER TABLE bacteria_resultado_encabezado
ADD CONSTRAINT fk_bacteria_resultado_muestra 
FOREIGN KEY (origin_muestra_id) 
REFERENCES bacteria_catalogo_tipo_muestra(id)
ON DELETE RESTRICT;
```

```sql
-- Documentación
COMMENT ON TABLE bacteria_resultado_encabezado 
IS 'Resultados de cultivos bacteriológicos y antibiogramas';

COMMENT ON COLUMN bacteria_resultado_encabezado.blee 
IS 'Beta-lactamasas de espectro extendido detectadas';

COMMENT ON COLUMN bacteria_resultado_encabezado.carbapenemasas 
IS 'Enzimas carbapenemasas detectadas (resistencia antibiótica)';
```


---
### Módulo Core del Sistema

#### 4. core_clientelaboratorio
**Propósito:** Tabla principal de clientes/pacientes del laboratorio.

```sql
-- Tabla principal
CREATE TABLE core_clientelaboratorio (
    NumeroCliente INTEGER NOT NULL DEFAULT nextval('core_clienteslaboratorio_Numero_de_Cliente_seq'::regclass),
    ClienteTipoId VARCHAR(1),
    ClienteId VARCHAR(40),
    ClientePrimerNombre VARCHAR(60) NOT NULL,
    ClienteSegundoNombre VARCHAR(60),
    ClientePrimerApellido VARCHAR(60) NOT NULL,
    ClienteSegundoApellido VARCHAR(60),
    ClienteFechaNace DATE,
    ClienteGenero VARCHAR(1),
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
    CONSTRAINT pk_clientelaboratorio PRIMARY KEY (NumeroCliente),
    CONSTRAINT chk_cliente_genero CHECK (ClienteGenero IN ('M', 'F', 'O'))
);
```

```sql
-- Secuencia
CREATE SEQUENCE core_clienteslaboratorio_Numero_de_Cliente_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```

```sql
-- SOLO índices realmente necesarios
CREATE INDEX idx_cliente_email ON core_clientelaboratorio(ClienteEmail);
CREATE INDEX idx_cliente_documento ON core_clientelaboratorio(ClienteId);
CREATE INDEX idx_cliente_lab ON core_clientelaboratorio(LabId) WHERE LabId IS NOT NULL;
```

```sql
-- Índice compuesto para búsquedas comunes
CREATE INDEX idx_cliente_nombres ON core_clientelaboratorio(
    ClientePrimerApellido, 
    ClienteSegundoApellido, 
    ClientePrimerNombre
);
```

```sql
-- Trigger de auditoría
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

```sql
-- Documentación
COMMENT ON TABLE core_clientelaboratorio IS 'Tabla maestra de pacientes del laboratorio';
COMMENT ON COLUMN core_clientelaboratorio.ClienteId IS 'DUI o documento de identidad del paciente';
COMMENT ON COLUMN core_clientelaboratorio.fecha_ultima_actualiza IS 'Actualizada automáticamente por trigger';
```



---
#### 5. core_ordenesexamenes
**Propósito:** Tabla principal de órdenes de exámenes del laboratorio.

```sql
--Crear tabla de órdenes de exámenes
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
```

```sql
-- Índices simples esenciales
CREATE INDEX idx_orden_fecha ON core_ordenesexamenes(OrdenFecha DESC);
CREATE INDEX idx_orden_cliente ON core_ordenesexamenes(ClienteNumero);
CREATE INDEX idx_orden_fecha_resultado ON core_ordenesexamenes(OrdenFechaResultado) 
WHERE OrdenFechaResultado IS NOT NULL;
```

```sql
-- Índice compuesto para búsquedas comunes
CREATE INDEX idx_orden_fecha_centro_estatus 
ON core_ordenesexamenes(OrdenFecha DESC, CentroId, OrdenEstatus);
```

```sql
-- Índices parciales para casos específicos
CREATE INDEX idx_ordenes_activas 
ON core_ordenesexamenes(OrdenFecha DESC, OrdenId) 
WHERE OrdenEstatus IN ('REG', 'PRO');

CREATE INDEX idx_ordenes_credito_pendiente 
ON core_ordenesexamenes(ClienteNumero, OrdenFecha) 
WHERE OrdenEsCredito = true AND OrdenEstatusCobro = 'P';
```

```sql
-- Validación de estados
ALTER TABLE core_ordenesexamenes
ADD CONSTRAINT chk_orden_estatus 
CHECK (OrdenEstatus IN ('REG', 'PRO', 'ENT', 'CAN', 'VAL'));

-- Validación de cobro
ALTER TABLE core_ordenesexamenes
ADD CONSTRAINT chk_orden_estatus_cobro 
CHECK (OrdenEstatusCobro IN ('P', 'C', 'A')); -- Pendiente, Cobrado, Anulado

-- Validación de montos
ALTER TABLE core_ordenesexamenes
ADD CONSTRAINT chk_orden_montos 
CHECK (
    OrdenDescuento >= 0 AND 
    OrdenImpuesto >= 0 AND 
    OrdenSubTotal >= 0
);
```

```sql
-- FOREIGN KEYS con comportamiento específico
ALTER TABLE core_ordenesexamenes
ADD CONSTRAINT fk_orden_cliente 
FOREIGN KEY (ClienteNumero) 
REFERENCES core_clientelaboratorio(NumeroCliente)
ON DELETE RESTRICT; -- No permitir borrar cliente con órdenes

ALTER TABLE core_ordenesexamenes
ADD CONSTRAINT fk_orden_centro 
FOREIGN KEY (CentroId) 
REFERENCES core_centroservicio(CentroId)
ON DELETE RESTRICT;

-- Trigger para calcular total automáticamente
CREATE OR REPLACE FUNCTION calculate_orden_total()
RETURNS TRIGGER AS $$
BEGIN
    NEW.orden_total = NEW.OrdenSubTotal - NEW.OrdenDescuento + NEW.OrdenImpuesto - NEW.impuesto_retenido;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_calculate_total
BEFORE INSERT OR UPDATE OF OrdenSubTotal, OrdenDescuento, OrdenImpuesto, impuesto_retenido
ON core_ordenesexamenes
FOR EACH ROW
EXECUTE FUNCTION calculate_orden_total();
```