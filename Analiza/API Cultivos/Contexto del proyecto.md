---
tags:
  - Analiza
---
# CONTEXTO DEL PROYECTO INTEGRACIÓN REAL BACTERIOLOGÍA

El proyecto consiste en la integración del sistema REAL (middleware que conecta equipos de bacteriología VITEK) con el CRM de Analiza, específicamente para automatizar el procesamiento de resultados de cultivos bacteriológicos. Actualmente, los técnicos de laboratorio ingresan manualmente los resultados de cultivos positivos; con esta integración, los resultados viajarán automáticamente desde los equipos VITEK través de REAL al CRM. El proyecto es parte de una migración mayor de CRM1 a CRM2 y tiene alta prioridad debido a compromisos con el gobierno de El Salvador.

## INFORMACIÓN TÉCNICA DETALLADA

### Stack Tecnológico

- **Frontend**: Angular (CRM2)
- **Backend**:
    - Python con FastAPI (nuevo API de resultados)
    - Python para servicios listener
    - RabbitMQ para mensajería asíncrona
- **Base de datos**: PostgreSQL
- **Protocolos**: ASTM, HL7 (comunicación con equipos médicos)
- **Infraestructura**:
    - AWS (RDS para base de datos)
    - Docker para contenedorización
    - Jenkins para CI/CD
- **Herramientas adicionales**:
    - Obsidian (documentación)
    - Postman (pruebas API)
    - OpenVPN (acceso remoto)
    - DbBeaver/DbSchema (diagramas BD)

### Arquitectura

El sistema sigue una arquitectura orientada a eventos con los siguientes componentes:

1. **CRM2** → Genera órdenes y las envía a RabbitMQ
2. **RabbitMQ** → Cola de mensajes para procesamiento asíncrono
3. **Listener Service** → Lee de RabbitMQ, convierte JSON a ASTM/HL7
4. **Sistema REAL** → Middleware que se comunica con equipos VITEK
5. **API de Resultados** → Recibe resultados y los almacena en BD
6. **Base de Datos** → PostgreSQL con tablas específicas para bacteriología

### Módulos y Funcionalidades

1. **Módulo de Generación de Peticiones (Oned Gómez)**:
    - Estado: En desarrollo
    - Función: Generar JSON desde BD, convertir a trama ASTM/HL7
    - Script SQL que extrae datos de órdenes basado en número de orden
2. **Servicio Listener RabbitMQ (Oned Gómez)**:
    - Estado: En desarrollo
    - Función: Consumir mensajes de cola, comunicarse con REAL
    - Manejo bidireccional de tramas ASTM/HL7
3. **API de Resultados Bacteriología (Daniel Barriba)**:
    - Estado: Estructura base completada, falta método de ingreso
    - Endpoints: CRUD bacterias, antibióticos, ingreso resultados
    - Framework: FastAPI
4. **Interfaz Angular CRM2**:
    - Estado: Existente para órdenes gobierno
    - Pantallas: Cultivos pendientes de atender/validar

## EQUIPO Y ORGANIZACIÓN

### Miembros del Equipo

- **Orlin Corea**: Arquitecto/Líder Técnico
    - Responsable de arquitectura, integraciones, decisiones técnicas
    - Gestión de permisos y accesos
- **Oned Gómez**: Desarrollador Backend
    - Responsable del servicio listener RabbitMQ
    - Integración con sistema REAL
    - Mapeo de catálogos y estructuras
- **Daniel Barriba**: Desarrollador Backend
    - Responsable del API de resultados
    - Inserción en base de datos
    - Documentación de esquemas
- **Carlos Lenin Majano**: Desarrollador Senior
    - Apoyo técnico general
    - Conocimiento del sistema legacy
- **María José Linqui**: Project Manager
    - Coordinación del equipo
    - Gestión de tareas y seguimiento
- **Natalia y Lidia**: Expertas Operativas
    - Validación de procesos de laboratorio
    - Definición de catálogos necesarios

## DECISIONES CLAVE TOMADAS

1. **No modificar tablas existentes**: Solo agregar campos a tablas de bacteriología para no afectar integración con gobierno
2. **Mapeo a nivel de prueba**: El mapeo entre sistemas se hace a nivel de prueba, no de examen
3. **Conversión de booleanos**: Valores TRUE/FALSE se convierten a "POSITIVO"/"NEGATIVO" en campos carbapenemasa y BLEE
4. **Actualización de estados**: Al recibir resultados, actualizar tabla tubo_prueba además de bacterias
5. **Dockerización obligatoria**: Todos los nuevos servicios deben incluir Dockerfile
6. **Uso de catálogos específicos**: Solo tipos de muestra 1-4 de bacteria_catalogo_tipo_muestra
7. **Manejo de CMI**: Valores se guardan concatenados (ej: "<=0.12") no separados

## ESTÁNDARES Y CONVENCIONES

- **Estilo de código**: PEP 8 para Python
- **Nomenclatura BD**:
    - Tablas: snake_case
    - Mantener convenciones existentes
- **Estructura de archivos**:
    - Scripts SQL en carpeta dedicada
    - README con instrucciones de setup
- **Documentación**:
    - DDL scripts para todos los cambios de BD
    - Documentación API con Swagger

## INTEGRACIONES Y DEPENDENCIAS

- **APIs externas**:
    - Sistema REAL (comunicación TCP/Socket)
    - Gobierno El Salvador (envío de resultados)
- **Servicios internos**:
    - RabbitMQ (colas de mensajes)
    - CRM1 (sistema legacy)
    - CRM2 (sistema nuevo)
- **Protocolos**:
    - ASTM (comunicación con equipos)
    - HL7 (estándar mensajería médica)
- **Dependencias críticas**:
    - Catálogos de microorganismos (7,429 de REAL)
    - Catálogos de antibióticos (278 de REAL)

- **Sistema REAL**: Middleware propietario para comunicación con equipos VITEK
- **Equipos VITEK**: Equipos de bacteriología que procesan muestras físicas
- **RabbitMQ**: Servicio de colas para comunicación asíncrona
- **CRM1/CRM2**: Sistemas de gestión de órdenes y resultados
- **Sistema Gobierno El Salvador**: Receptor final de resultados para telemedicina
- **AWS Services**: RDS (base de datos), API Gateway (seguridad), EC2 (hosting)


## PROBLEMAS CONOCIDOS Y CONSIDERACIONES

- **Diferencias estructurales**: REAL maneja exámenes generalizados (ej: "CULTIVO MICOLÓGICO") vs Analiza específicos (ej: "CULTIVO MICOLÓGICO DE UÑA")
    - Solución propuesta: Tabla aux_examenes_real para mapeo
- **Volumen de catálogos**: 7,429 microorganismos en REAL vs cantidad limitada actual
    - Pendiente: Confirmar con Natalia cuáles son necesarios
- **Múltiples organismos**: REAL puede enviar múltiples microorganismos por cultivo
    - Impacto: Posible modificación de estructura de tablas
- **Código de barras**: Campo string 15 caracteres vs BigInt en algunas tablas
    - Usar tabla orden_tubo.codigo_barra correctamente

## INSTRUCCIONES PARA EL ASISTENTE IA

Cuando recibas una tarea relacionada con este proyecto, debes:

1. **Validar el contexto**: Asegurarte de que la tarea se alinea con las decisiones y arquitectura establecidas
2. **Hacer las siguientes preguntas clave**:
    - ¿En qué módulo específico trabajarás (Listener, API, BD)?
    - ¿Afecta esto la integración existente con el gobierno?
    - ¿Requiere mapeo entre catálogos de REAL y Analiza?
    - ¿Qué tipo de orden es (gobierno vs privada)?
    - ¿Se necesita actualizar múltiples tablas (encabezado, detalle, tubo_prueba)?
    - ¿Hay consideraciones especiales para el manejo de errores?
3. **Considerar automáticamente**:
    - No modificar estructura de tablas existentes
    - Valores booleanos se convierten a "POSITIVO"/"NEGATIVO"
    - Códigos de barra son strings de 15 caracteres
    - Tipos de muestra válidos: 1-4 (coprocultivo, hemocultivo, urocultivo, secreciones)
    - Actualizar tubo_prueba.prueba_finalizada = TRUE al insertar resultados
    - El flujo: CRM2 → RabbitMQ → Listener → REAL → Listener → API → BD
4. **Formato de respuesta**:
    - Confirmar comprensión del requerimiento
    - Identificar tablas y servicios afectados
    - Validar contra decisiones existentes
    - Proponer solución técnica
    - Incluir consideraciones de la integración gobierno

## GLOSARIO DE TÉRMINOS DEL PROYECTO

- **ASTM**: Protocolo de comunicación para equipos de laboratorio
- **HL7**: Health Level 7, estándar para intercambio de información médica
- **VITEK**: Marca de equipos de bacteriología
- **CMI**: Concentración Mínima Inhibitoria (antibióticos)
- **BLEE**: Beta-Lactamasas de Espectro Extendido
- **Carbapenemasa**: Enzima que confiere resistencia a antibióticos
- **Trama**: Mensaje estructurado en protocolo ASTM/HL7
- **Tubo**: Contenedor físico de muestra con código de barras único
- **Orden gobierno**: Orden de telemedicina referida por el gobierno
- **CRM1/CRM2**: Sistema legacy vs sistema nuevo de gestión
