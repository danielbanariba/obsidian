### **Campos Correctamente Identificados en sus Documentos**

#### 1. **Tabla aux_examenes_real** (CRÍTICA)

Las tramas HL7 requieren identificadores únicos para las observaciones (OBX-3) [Hl7](https://hl7.eu/refactored/segOBX.html)[HCUP-US](https://hcup-us.ahrq.gov/datainnovations/clinicalcontentenhancementtoolkit/hi6.jsp), y LA propuesta de mapeo entre códigos REAL y Analiza es esencial.

#### 2. **Campos Adicionales en bacteria_resultado_encabezado**

Los campos propuestos para HL7/ASTM:

- `equipo_vitek_id`: Necesario para identificar el instrumento origen
- `astm_message_id` y `hl7_message_id`: Los mensajes HL7 requieren identificadores únicos de mensaje [HL7 Implementation Guide.](https://hcup-us.ahrq.gov/datainnovations/clinicalcontentenhancementtoolkit/hi6.jsp)
- `codigo_barra_muestra`: Identificador crítico para trazabilidad
- `multiple_organismos`: HL7 especifica que cada organismo aislado debe reportarse en segmentos OBX separados [HL7 V2.4 Chapter 7 +2](https://www.hl7.eu/HL7v2x/v24/std24/ch07.htm)

#### 3. **Tabla bacteria_resultado_organismos**

Es esencial porque HL7 requiere reportar cada organismo como una observación separada con su propio OBX-4 (sub-ID) [HL7 V2.2 Chapter 7](https://www.hl7.eu/HL7v2x/v22/std22/HL7CHP7.html)

#### 4. **Tabla bacteria_mensajes_real**

Correcta para la trazabilidad de mensajes ASTM/HL7 bidireccionales.

### **Campos Adicionales

Basándome en los estándares HL7 para microbiología, sugiero agregar estos campos:

En `bacteria_resultado_encabezado`:
```sql
-- Campo para el parent result (cultivos relacionados)
parent_result_id INTEGER REFERENCES bacteria_resultado_encabezado(id),

-- Método de cultivo específico
metodo_cultivo VARCHAR(100),

-- Tiempo de incubación
tiempo_incubacion_horas INTEGER,

-- Condiciones especiales del cultivo
condiciones_cultivo JSONB,

-- OBX-11 Status del resultado
resultado_status VARCHAR(20) CHECK (resultado_status IN ('F', 'P', 'C', 'R', 'I', 'S')),
-- F=Final, P=Preliminary, C=Correction, R=Results entered, I=In lab, S=Partial
```

En `bacteria_resultado_detalle`:
```sql
-- OBX-17 Método de observación
metodo_observacion VARCHAR(50),

-- Breakpoints utilizados (CLSI, EUCAST, etc)
breakpoint_standard VARCHAR(20),
breakpoint_version VARCHAR(20),

-- Valor numérico de CMI separado para cálculos
cmi_valor_numerico NUMERIC,
cmi_operador VARCHAR(2) -- '<=', '>=', '=', etc
```

### **Validación de Decisiones Tomadas**

1. ** Conversión BLEE/Carbapenemasa**: Correcta la decisión de usar "POSITIVO"/"NEGATIVO"
2. ** CMI concatenado**: El campo OBX-5 de HL7 permite valores texto [HL7 OBX Segment | Rhapsody](https://rhapsody.health/resources/hl7-obx-segment/), pero sugiero agregar campos numéricos separados
3. ** Múltiples organismos**: Estructura propuesta cumple con el requerimiento HL7 de reportar organismos separadamente [HL7 V2.3.1 Observation Reporting](https://www.hl7.eu/HL7v2x/v231/std231/Ch7.html)
