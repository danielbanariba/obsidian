# Mapeo HL7 - Sistema Bacteriología CRM2

## 1. Scripts SQL para Nuevos Tipos de Muestra
```sql
-- Insertar nuevos tipos de muestra en el catálogo
INSERT INTO bacteria_catalogo_tipo_muestra (id, muestra_nombre, codigo_loinc, activo) VALUES
(5, 'Cultivo de esterilidad', '87969-8', true),
(6, 'Cultivo de líquidos', '600-7', true),
(7, 'Cultivo de punta de catéter', '89869-6', true),
(8, 'Cultivo Micológico', '580-1', true);

-- Nota: Los códigos LOINC son ejemplos, deben validarse con el catálogo oficial
```

## 2. Tabla de Mapeo de Tipos de Muestra Actualizada

| ID  | Tipo Muestra Interno        | Código LOINC | Código REAL | Descripción                    |
| --- | --------------------------- | ------------ | ----------- | ------------------------------ |
| 1   | Cultivo de orina            | 630-4        | UR          | Urocultivo                     |
| 2   | Cultivo de heces            | 625-4        | HE          | Coprocultivo                   |
| 3   | Cultivo de sangre           | 600-7        | SA          | Hemocultivo                    |
| 4   | Cultivo de esputo           | 624-7        | ES          | Cultivo de esputo              |
| 5   | Cultivo de esterilidad      | 87969-8      | CE          | Cultivo de esterilidad         |
| 6   | Cultivo de líquidos         | 600-7        | CL          | Cultivo de líquidos corporales |
| 7   | Cultivo de punta de catéter | 89869-6      | PC          | Cultivo punta de catéter       |
| 8   | Cultivo Micológico          | 580-1        | CM          | Cultivo micológico             |

## 3. Scripts ALTER TABLE para Campos HL7 Faltantes

```sql
-- Agregar campos HL7 faltantes a bacteria_resultado_encabezado
ALTER TABLE bacteria_resultado_encabezado 
ADD COLUMN IF NOT EXISTS message_control_id VARCHAR(20),
ADD COLUMN IF NOT EXISTS fecha_coleccion_muestra TIMESTAMP,
ADD COLUMN IF NOT EXISTS crecimiento_ufc_ml VARCHAR(50),
ADD COLUMN IF NOT EXISTS unidades_medida VARCHAR(20),
ADD COLUMN IF NOT EXISTS metodo_prueba VARCHAR(100),
ADD COLUMN IF NOT EXISTS json_respuesta_real TEXT,
ADD COLUMN IF NOT EXISTS fecha_envio_real TIMESTAMP,
ADD COLUMN IF NOT EXISTS fecha_respuesta_real TIMESTAMP;
```

## 4. Tablas de Configuración de Mapeo

### 4.1 Tabla de Mapeo de Exámenes
```sql
-- Crear tabla de mapeo examen Analiza -> código REAL
CREATE TABLE IF NOT EXISTS bacteria_mapeo_examenes (
    id SERIAL PRIMARY KEY,
    examen_id VARCHAR(20) NOT NULL REFERENCES core_examenes(ExamenId),
    codigo_real VARCHAR(20) NOT NULL,
    nombre_real VARCHAR(100),
    activo BOOLEAN DEFAULT true,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(examen_id, codigo_real)
);

-- Insertar mapeos iniciales
INSERT INTO bacteria_mapeo_examenes (examen_id, codigo_real, nombre_real) VALUES
('CUOR', 'UR', 'Urocultivo'),
('CUHEC', 'HE', 'Coprocultivo'),
('CUSAN', 'SA', 'Hemocultivo'),
('CUESP', 'ES', 'Cultivo de esputo'),
('CUEST', 'CE', 'Cultivo de esterilidad'),
('CULIQ', 'CL', 'Cultivo de líquidos'),
('CUCAT', 'PC', 'Cultivo punta catéter'),
('CUMIC', 'CM', 'Cultivo micológico');
```

### 4.2 Tabla de Mapeo de Tipos de Muestra
```sql
-- Crear tabla de mapeo tipo muestra interno -> tipo muestra REAL
CREATE TABLE IF NOT EXISTS bacteria_mapeo_tipo_muestra (
    id SERIAL PRIMARY KEY,
    tipo_muestra_interno_id INTEGER NOT NULL REFERENCES bacteria_catalogo_tipo_muestra(id),
    codigo_real VARCHAR(10) NOT NULL,
    descripcion_real VARCHAR(100),
    activo BOOLEAN DEFAULT true,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(tipo_muestra_interno_id, codigo_real)
);

-- Insertar mapeos
INSERT INTO bacteria_mapeo_tipo_muestra (tipo_muestra_interno_id, codigo_real, descripcion_real) VALUES
(1, 'UR', 'Urocultivo'),
(2, 'HE', 'Coprocultivo'),
(3, 'SA', 'Hemocultivo'),
(4, 'ES', 'Cultivo de esputo'),
(5, 'CE', 'Cultivo de esterilidad'),
(6, 'CL', 'Cultivo de líquidos'),
(7, 'PC', 'Cultivo punta catéter'),
(8, 'CM', 'Cultivo micológico');
```

### 4.3 Tabla de Mapeo de Antibióticos

sql

```sql
-- Crear tabla de mapeo antibióticos interno -> código REAL
CREATE TABLE IF NOT EXISTS bacteria_mapeo_antibioticos (
    id SERIAL PRIMARY KEY,
    antibiotico_interno_id INTEGER NOT NULL REFERENCES bacteria_catalogo_antibioticos(id),
    codigo_real VARCHAR(10) NOT NULL,
    nombre_real VARCHAR(100),
    activo BOOLEAN DEFAULT true,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(antibiotico_interno_id, codigo_real)
);
```

## 5. Documentación del Flujo JSON → Base de Datos

### 5.1 Flujo de Procesamiento

python

```python
# Pseudocódigo del flujo de procesamiento

def procesar_resultado_real(json_data):
    """
    Procesa resultado JSON del equipo REAL y actualiza base de datos
    """
    
    # 1. Validar y parsear JSON
    resultado = validar_json_real(json_data)
    
    # 2. Buscar tubo en el sistema
    tubo = buscar_tubo_por_codigo_barra(resultado['tubebarcode'])
    if not tubo:
        raise Exception("Tubo no encontrado")
    
    # 3. Obtener orden y examen asociados
    orden = obtener_orden_por_tubo(tubo.tubo_id)
    examen_prueba = obtener_examen_prueba_tubo(tubo.tubo_id)
    
    # 4. Crear/actualizar encabezado de resultado
    encabezado = {
        'orden_numero': orden.orden_id,
        'examen_id': examen_prueba.examen_id,
        'tubo_id': tubo.tubo_id,
        'prueba_id': examen_prueba.prueba_id,
        'fecha_recepcion': resultado['receptiondatetime'],
        'fecha_registro_resultado': resultado['resultdatetime'],
        'fecha_entrega': datetime.now(),
        'organismo_seleccionado_id': resultado['detail']['microorganismid'],
        'conclusion': resultado['conclusion'],
        'blee': traducir_booleano(resultado['blee']),
        'carbapenemasas': traducir_booleano(resultado['carbapenemasa']),
        'usuario_registro': 'SISTEMA_REAL',
        'json_respuesta_real': json.dumps(json_data),
        'fecha_respuesta_real': datetime.now()
    }
    
    encabezado_id = insertar_actualizar_encabezado(encabezado)
    
    # 5. Procesar antibiograma
    for antibiotico in resultado['detail']['antibiogram']:
        detalle = {
            'encabezado_id': encabezado_id,
            'antibiotico_id': mapear_antibiotico_real_interno(antibiotico['antibiotic']),
            'cmi': antibiotico['cmi'],
            'interpretacion': validar_interpretacion(antibiotico['interpretation'])
        }
        insertar_detalle_antibiograma(detalle)
    
    # 6. Actualizar estado de la prueba
    actualizar_estado_prueba(tubo.tubo_id, examen_prueba.prueba_id)
    
    # 7. Generar mensaje HL7 de respuesta si es necesario
    hl7_response = generar_hl7_oru(encabezado_id)
    
    return encabezado_id

def traducir_booleano(valor):
    """Traduce valores booleanos a POSITIVO/NEGATIVO"""
    if valor.upper() in ['TRUE', 'POSITIVO', 'POSITIVE', '+']:
        return 'POSITIVO'
    return 'NEGATIVO'

def validar_interpretacion(valor):
    """Valida y normaliza valores de interpretación"""
    mapeo = {
        'SENSIBLE': 'S',
        'INTERMEDIO': 'I',
        'RESISTENTE': 'R',
        'NO SENSIBLE': 'NS',
        'SENSIBLE DOSIS DEPENDIENTE': 'SDD'
    }
    return mapeo.get(valor.upper(), valor)

def actualizar_estado_prueba(tubo_id, prueba_id):
    """Actualiza el estado de la prueba como finalizada"""
    sql = """
        UPDATE core_tuboprueba 
        SET PruebaFinalizada = true,
            PruebaFechaHoraEnviaDato = CURRENT_TIMESTAMP
        WHERE TuboId = %s AND PruebaId = %s
    """
    ejecutar_sql(sql, [tubo_id, prueba_id])
```

### 5.2 Mapeo Detallado JSON → Tablas

|Campo JSON|Tabla Destino|Campo Destino|Transformación|
|---|---|---|---|
|tubebarcode|core_ordentubos|TuboCodigoBarra|Buscar ID del tubo|
|receptiondatetime|bacteria_resultado_encabezado|fecha_recepcion|String → Timestamp|
|resultdatetime|bacteria_resultado_encabezado|fecha_registro_resultado|String → Timestamp|
|conclusion|bacteria_resultado_encabezado|conclusion|Directo|
|detail.microorganismid|bacteria_resultado_encabezado|organismo_seleccionado_id|String → Integer|
|carbapenemasa|bacteria_resultado_encabezado|carbapenemasas|POSITIVO/NEGATIVO|
|blee|bacteria_resultado_encabezado|blee|POSITIVO/NEGATIVO|
|detail.antibiogram[].antibiotic|bacteria_resultado_detalle|antibiotico_id|Mapear código REAL → ID interno|
|detail.antibiogram[].cmi|bacteria_resultado_detalle|cmi|Directo|
|detail.antibiogram[].interpretation|bacteria_resultado_detalle|interpretacion|Normalizar a S/I/R/NS/SDD|

### 5.3 Validaciones Críticas

1. **Validación de Tubo**: El código de barras debe existir en `core_ordentubos`
2. **Validación de Microorganismo**: El ID debe existir en `bacteria_catalogo_bacterias`
3. **Validación de Antibióticos**: Los códigos deben estar mapeados en `bacteria_mapeo_antibioticos`
4. **Validación de Interpretación**: Solo valores permitidos: S, I, R, NS, SDD
5. **Validación de Estado**: La orden debe estar en estado válido para recibir resultados

### 5.4 Consideraciones de Integración

1. **Transaccionalidad**: Todo el proceso debe ser transaccional (BEGIN/COMMIT/ROLLBACK)
2. **Idempotencia**: Soportar reenvío de resultados sin duplicar datos
3. **Auditoría**: Guardar JSON original en campo `json_respuesta_real`
4. **Notificaciones**: Disparar eventos al completar resultado
5. **Compatibilidad**: Mantener estructura para módulo del gobierno

## 6. Queries de Verificación

sql

```sql
-- Verificar mapeos completos
SELECT 
    e.ExamenId,
    e.ExamenNombre,
    bme.codigo_real,
    bme.nombre_real
FROM core_examenes e
LEFT JOIN bacteria_mapeo_examenes bme ON e.ExamenId = bme.examen_id
WHERE e.ExamenId LIKE 'CU%'
AND e.ExamenActivo = true;

-- Verificar resultados procesados
SELECT 
    bre.orden_numero,
    bre.conclusion,
    bre.blee,
    bre.carbapenemasas,
    COUNT(brd.id) as antibiogramas
FROM bacteria_resultado_encabezado bre
LEFT JOIN bacteria_resultado_detalle brd ON bre.id = brd.encabezado_id
WHERE bre.fecha_registro_resultado >= CURRENT_DATE - INTERVAL '7 days'
GROUP BY bre.id, bre.orden_numero, bre.conclusion, bre.blee, bre.carbapenemasas;
```

Esta documentación proporciona una guía completa para implementar el mapeo HL7 manteniendo compatibilidad con el sistema existente y permitiendo la integración bidireccional con el equipo REAL.