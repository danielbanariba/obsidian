---
tags:
  - Analiza
---
---
Claude Opus

# Proyecto Integración REAL Bacteriología

## RESUMEN DEL ANÁLISIS

Implementación de la integración entre el sistema REAL (middleware de equipos de bacteriología) y el CRM de Analiza.

---

# CONTEXTO DEL PROYECTO INTEGRACIÓN REAL BACTERIOLOGÍA

## RESUMEN EJECUTIVO

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
- **Git workflow**:
    - Desarrollo en branch "step"
    - Deploy a producción via Jenkins
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


---
Claude Sonnet
# CONTEXTO DEL PROYECTO: INTEGRACIÓN REAL BACTERIOLOGÍA

## RESUMEN EJECUTIVO

El proyecto consiste en la integración del sistema REAL (middleware de equipos de bacteriología) con el sistema CRM de Analiza Laboratorios. El objetivo principal es automatizar el procesamiento de resultados de cultivos microbiológicos (urocultivos, coprocultivos, hemocultivos y cultivos de secreciones) que actualmente se ingresan manualmente. La integración permitirá el intercambio bidireccional de información: envío de peticiones desde el CRM hacia los equipos VITEK a través de REAL, y recepción automática de resultados con detalles de bacterias encontradas y antibiogramas. 

## INFORMACIÓN TÉCNICA DETALLADA

### Stack Tecnológico

- **Frontend**: Angular (CRM1 y CRM2)
- **Backend**:
    - FastAPI (Python) para APIs nuevos
    - APIs existentes (framework no especificado)
    - Servicios listener en Python
- **Base de datos**: PostgreSQL
- **Infraestructura**:
    - AWS (RDS, API Gateway, EC2)
    - Docker containers para deployment
    - Jenkins para CI/CD
- **Comunicación**:
    - RabbitMQ para colas de mensajes
    - Protocolos HL7 y ASTM para comunicación con equipos
    - OpenVPN para acceso remoto
- **Herramientas adicionales**:
    - Swagger para documentación de APIs
    - Obsidian para documentación del proyecto
    - Postman para testing

### Arquitectura

El sistema utiliza una arquitectura basada en microservicios con comunicación asíncrona mediante colas de mensajes. El flujo principal es:

1. **CRM2** → Genera órdenes y las envía a **RabbitMQ**
2. **Servicio Listener (Oned)** → Consume mensajes, convierte JSON a tramas HL7/ASTM
3. **Sistema REAL** → Middleware que comunica con equipos VITEK
4. **Equipos VITEK** → Procesan muestras y devuelven resultados
5. **API CRM (Daniel)** → Recibe resultados y los almacena en base de datos

### Módulos y Funcionalidades

1. **Módulo de Generación de Peticiones (Oned)**:
    - Estado: En desarrollo activo
    - Consume colas RabbitMQ
    - Convierte JSON a tramas HL7/ASTM
    - Envía peticiones a sistema REAL
2. **Módulo API de Resultados (Daniel)**:
    - Estado: Estructura base completada
    - API REST con FastAPI
    - Endpoints para inserción de resultados de bacteriología
    - Documentación con Swagger
3. **Módulo de Base de Datos**:
    - Estado: Tablas existentes, requiere extensiones
    - Tablas principales: bacteria_resultado_encabezado, bacteria_resultado_detalle
    - Catálogos: bacterias, antibióticos, tipos de muestra
4. **Módulo Frontend CRM2**:
    - Estado: Implementado para gobierno
    - Pantallas de cultivos pendientes de atender/validar
    - Interfaz manual existente para bacteriología

## EQUIPO Y ORGANIZACIÓN

### Miembros del Equipo

- **Orlin Corea**: Arquitecto técnico y líder del proyecto
    - Responsable de arquitectura general, definición técnica, coordinación
- **Oned Gómez**: Desarrollador backend (servicios)
    - Responsable de servicios listener, integración con REAL, generación de tramas
- **Daniel Banariba**: Desarrollador backend (APIs)
    - Responsable de APIs de inserción, validación de estructura de datos
- **Lenin**: Desarrollador senior de soporte
    - Soporte técnico, revisión de código, configuraciones
- **María José**: Project Manager
    - Gestión de proyecto, seguimiento de tareas, coordinación con cliente

### Metodología de Trabajo

- Reuniones semanales de seguimiento (lunes 10:00 AM)
- Comunicación activa por Slack (#integracion-real-bacterologia)
- Documentación en tiempo real con Obsidian
- Control de versiones con Git (branch 'step' para desarrollo)
- Sistema de tareas en plataforma de gestión de proyectos

## DECISIONES CLAVE TOMADAS

1. **Protocolos de Comunicación**: Se adoptó HL7 como protocolo principal para comunicación con equipos, con ASTM como alternativa
2. **Arquitectura de Colas**: RabbitMQ para procesamiento asíncrono, evitando bloqueos en el proceso de facturación
3. **Estructura de Datos**: Mantener tablas existentes sin modificaciones para no afectar integración con gobierno, solo agregar campos
4. **Seguridad de APIs**: Usar AWS API Gateway para manejo de autenticación en lugar de implementar seguridad a nivel de aplicación
5. **Mapeo de Datos**: Crear tabla auxiliar `aux_examenes_real` para manejar diferencias entre estructuras de Analiza y REAL
6. **Catálogos**: Incluir todos los 7,429 microorganismos del catálogo REAL, no solo los más comunes

## ESTÁNDARES Y CONVENCIONES

- **Estilo de código**: Seguir convenciones estándar de Python (PEP 8) y Angular
- **Nomenclatura**:
    - Tablas: snake_case con prefijos (bacteria_, core_, aux_)
    - APIs: REST con verbos HTTP estándar
    - Variables: nombres descriptivos en español/inglés según contexto
- **Estructura de archivos**: Modular con separación clara de responsabilidades
- **Git workflow**: Feature branches mergeando a 'step', deploy a producción desde main
- **Documentación**: Scripts DDL documentados, README con instrucciones, comentarios en código

## INTEGRACIONES Y DEPENDENCIAS

- **Sistema REAL**: Middleware propietario para comunicación con equipos VITEK
- **Equipos VITEK**: Equipos de bacteriología que procesan muestras físicas
- **RabbitMQ**: Servicio de colas para comunicación asíncrona
- **CRM1/CRM2**: Sistemas de gestión de órdenes y resultados
- **Sistema Gobierno El Salvador**: Receptor final de resultados para telemedicina
- **AWS Services**: RDS (base de datos), API Gateway (seguridad), EC2 (hosting)

## PROBLEMAS CONOCIDOS Y CONSIDERACIONES

1. **Diferencias Estructurales**:
    - REAL maneja exámenes más generalizados que Analiza
    - Solución: Tabla de mapeo `aux_examenes_real`
2. **Volumen de Catálogos**:
    - 7,429 microorganismos requieren mapeo completo
    - 278 antibióticos más manejables
3. **Compatibilidad de Valores**:
    - Conversión de valores booleanos (true/false) a texto (POSITIVO/NEGATIVO)
    - Valores numéricos vs texto (ejemplo: "<=0.5" vs números)
4. **Manejo de Errores**:
    - Implementar logging detallado para trazabilidad
    - Control de ACK en protocolo HL7
    - Evitar duplicación de tramas
5. **Sincronización de Estados**:
    - Actualizar tabla tubo_prueba al recibir resultados
    - Mantener consistencia entre tablas de bacteriología y tabla general

## HISTÓRICO DE CAMBIOS IMPORTANTES

- **Inicio**: Definición de arquitectura y división de responsabilidades
- **Semana 1**: Implementación de estructura básica de tablas y APIs
- **Semana 2**: Desarrollo de servicios listener y pruebas de comunicación
- **Semana 3**: Integración de catálogos REAL y mapeo de datos
- **Actual**: Pruebas de integración y refinamiento de flujos

## INSTRUCCIONES PARA EL ASISTENTE IA

Cuando recibas una tarea relacionada con este proyecto, debes:

1. **Validar el contexto**: Asegurar alineación con arquitectura de colas RabbitMQ y protocolos HL7/ASTM
2. **Hacer las siguientes preguntas clave**:
    - ¿El cambio afecta tablas existentes del gobierno?
    - ¿Requiere mapeo de catálogos REAL?
    - ¿Es compatible con el flujo CRM→RabbitMQ→REAL→Equipos?
    - ¿Necesita documentación DDL?
    - ¿Afecta la comunicación asíncrona?
    - ¿Requiere manejo especial de errores/logging?
3. **Considerar automáticamente**:
    - No modificar tablas bacteria_resultado_* existentes
    - Incluir Dockerfile para nuevos servicios
    - Documentar scripts DDL para deployment
    - Mantener compatibilidad con integración gobierno
    - Verificar mapeo de microorganismos/antibióticos
4. **Formato de respuesta**:
    - Confirmar entendimiento técnico del flujo
    - Identificar dependencias con otros módulos (Oned/Daniel)
    - Proponer solución respetando arquitectura existente
    - Alertar sobre posibles impactos en integración gobierno

## GLOSARIO DE TÉRMINOS DEL PROYECTO

- **REAL**: Sistema middleware para comunicación con equipos de bacteriología
- **VITEK**: Marca de equipos automatizados para identificación bacteriana
- **Antibiograma**: Prueba de sensibilidad a antibióticos
- **CMI**: Concentración Mínima Inhibitoria
- **HL7/ASTM**: Protocolos estándar para comunicación con equipos médicos
- **Carbapenemasa/BLEE**: Enzimas de resistencia bacteriana específicas
- **Tubo**: Contenedor físico con código de barras para muestras
- **Trama**: Mensaje estructurado según protocolo HL7/ASTM
- **ACK**: Mensaje de confirmación de recepción

---

_Este documento se generó analizando 6 reuniones principales y conversaciones de Slack desde el inicio del proyecto hasta la fecha actual_

He analizado todas las transcripciones y la conversación de Slack para crear un contexto completo del proyecto. El análisis revela un proyecto complejo de integración entre sistemas médicos con múltiples actores, tecnologías y consideraciones técnicas específicas.

Los puntos clave que identifiqué:

1. **Proyecto bien estructurado** con roles claros y metodología definida
2. **Arquitectura moderna** usando microservicios, colas asíncronas y contenedores
3. **Integración crítica** con sistemas externos (REAL, equipos VITEK)
4. **Consideraciones de compatibilidad** con sistemas gubernamentales existentes
5. **Timeline ajustado** (3 semanas) que requiere coordinación eficiente

La información está organizada de manera que cualquier asistente IA pueda entender rápidamente:

- El contexto técnico y de negocio
- Las restricciones y dependencias
- Los estándares a seguir
- Los problemas conocidos
- Las preguntas correctas a hacer

Este contexto permitirá que futuras interacciones sean más efectivas y alineadas con las decisiones y arquitectura del proyecto.

He analizado completamente las transcripciones de las 6 reuniones principales y la conversación de Slack del proyecto de integración REAL Bacteriología. El análisis revela un proyecto técnicamente complejo y bien estructurado con los siguientes hallazgos principales:

## ANÁLISIS DETALLADO COMPLETADO

**Reuniones analizadas**: 6 reuniones técnicas + conversaciones Slack **Período**: Desde inicio del proyecto hasta estado actual **Participantes identificados**: 5 miembros del equipo con roles específicos

## INFORMACIÓN FALTANTE O AMBIGÜEDADES ENCONTRADAS

1. **Detalles de Configuración REAL**: Faltan especificaciones exactas del protocolo de comunicación con sistema REAL (TCP vs sockets)
2. **Catálogo Final**: Pendiente confirmación de cuáles de los 7,429 microorganismos son realmente necesarios
3. **Cronograma Específico**: Aunque se menciona "3 semanas", faltan hitos intermedios detallados
4. **Pruebas de Integración**: No se especifica completamente la estrategia de testing con equipos reales

## RECOMENDACIONES PARA MEJORAR LA DOCUMENTACIÓN

1. **Crear diagramas técnicos** del flujo de datos completo CRM→REAL→Equipos
2. **Documentar casos de error** y estrategias de recovery específicas
3. **Establecer métricas de éxito** cuantificables para la integración
4. **Definir procedimientos de rollback** en caso de fallos en producción
5. **Crear guías de troubleshooting** para operadores del laboratorio

## FORTALEZAS DEL PROYECTO IDENTIFICADAS

- **Arquitectura robusta** con separación clara de responsabilidades
- **Equipo experimentado** con conocimiento profundo del dominio
- **Comunicación efectiva** entre miembros del equipo
- **Consideración proactiva** de problemas de compatibilidad
- **Metodología clara** con seguimiento regular

El contexto generado proporciona toda la información necesaria para que cualquier asistente IA pueda contribuir efectivamente al proyecto, entendiendo tanto los aspectos técnicos como las restricciones de negocio y las decisiones arquitectónicas ya tomadas.



---
Gemini Pro

# CONTEXTO DEL PROYECTO: Integración REAL Bacteriología

## RESUMEN EJECUTIVO

El proyecto consiste en la integración del sistema de información de laboratorio (LIS) de Analiza con un middleware de un proveedor externo llamado REAL, con el objetivo de automatizar el procesamiento de exámenes de bacteriología. Actualmente, el flujo para estos exámenes requiere un ingreso manual de resultados. La integración busca automatizar tanto el envío de peticiones de análisis desde el CRM de Analiza hacia REAL, como la recepción e inserción de los resultados (bacterias encontradas y antibiogramas) provenientes de los equipos de bacteriología (Vitek) a través de REAL.

El flujo de trabajo es asíncrono, utilizando RabbitMQ como bus de mensajes. El proyecto se encuentra en la fase de desarrollo, donde se está construyendo un servicio intermediario (listener) que se comunica con REAL y una API para la inserción de resultados en la base de datos del CRM.

## INFORMACIÓN TÉCNICA DETALLADA

### Stack Tecnológico

- **Backend:** Python. Se utiliza FastAPI para la construcción de la API de resultados.
    
- **Base de datos:** PostgreSQL
    
- **Mensajería:** RabbitMQ para la comunicación asíncrona entre servicios.
    
- **Protocolos de Comunicación:** HL7 y ASTM para el intercambio de datos con el sistema REAL.
    
- **Infraestructura:** Se menciona AWS, específicamente para el despliegue a través de un API Gateway y el uso de RDS para la base de datos.
    
- **Herramientas adicionales:** Docker para la contenerización de las APIs, Postman para pruebas de API, Obsidian para documentación, y se planea usar Jenkins para CI/CD.
    

### Arquitectura

El sistema sigue una arquitectura de microservicios basada en eventos y colas de mensajes.

**Flujo de Petición (Analiza → REAL):**

1. **CRM2:** Se genera y paga una orden de laboratorio. En el proceso de pago, se generan los tubos y se dispara un evento.
    
2. **Publicador (en CRM2):** Se construye un JSON con la información del paciente y los exámenes, y se publica en una cola específica de bacteriología en RabbitMQ.
    
3. **Servicio Listener (desarrollado por Oned):** Un servicio en Python consume el mensaje JSON de la cola de RabbitMQ.
    
4. **Traducción a Trama:** El listener convierte el JSON a una trama en formato HL7/ASTM (trama de petición).
    
5. **Envío a REAL:** La trama de petición se envía al sistema REAL, que a su vez la comunica al equipo de procesamiento (ej. Vitek).
    

**Flujo de Resultado (REAL → Analiza):**

1. **Equipo de Bacteriología:** Procesa la muestra y envía el resultado al sistema REAL.
    
2. **Sistema REAL:** Envía el resultado como una trama HL7/ASTM al servicio listener.
    
3. **Servicio Listener (desarrollado por Oned):** Recibe y parsea la trama de resultado, la convierte en un formato JSON estandarizado y realiza traducciones de datos (ej. boolean a "POSITIVO"/"NEGATIVO").
    
4. **API de Resultados (desarrollado por Daniel):** El listener envía el JSON al endpoint de la API REST (`/ingreso_resultado_bacteria`).
    
5. **Inserción en Base de Datos:** La API procesa el JSON y guarda la información en las tablas correspondientes (`bacteria_resultado_encabezado`, `bacteria_resultado_detalle`) y actualiza el estado de la prueba en `tubo_prueba`.
    

### Módulos y Funcionalidades

1. **Servicio Listener de Bacteriología:**
    
    - **Descripción:** Servicio responsable de la comunicación bidireccional con el sistema REAL. Lee peticiones de una cola de RabbitMQ para enviarlas a REAL y escucha activamente los resultados de REAL para enviarlos a la API interna.
        
    - **Estado:** En desarrollo. El script para generar el JSON de petición y enviarlo a RabbitMQ está listo para pruebas.
        
2. **API de Resultados de Bacteriología (Parte de API-Análisis):**
    
    - **Descripción:** API REST que expone endpoints para recibir los resultados procesados del listener. Su función principal es validar los datos y persistirlos en la base de datos del CRM.
        
    - **Estado:** En desarrollo. Se han documentado los endpoints existentes y se está trabajando en el endpoint específico para la inserción de resultados de bacteriología.
        
3. **Mapeo de Catálogos:**
    
    - **Descripción:** Lógica y tablas para homologar los catálogos de exámenes, muestras, microorganismos y antibióticos entre Analiza y REAL.
        
    - **Estado:** En análisis. Se han identificado diferencias estructurales y se ha propuesto la creación de tablas auxiliares (`aux_examenes_real`) para manejar el mapeo.
        

## EQUIPO Y ORGANIZACIÓN

### Miembros del Equipo

- **Oned Gómez:** [Rol: Desarrollador Backend]
    
    - Responsable del servicio listener, la comunicación con el sistema REAL, la generación y parseo de tramas HL7/ASTM, y la creación del JSON de resultados para la API.
        
- **Daniel Banariba:** [Rol: Desarrollador Backend]
    
    - Responsable de desarrollar la API REST (API-Análisis) para la inserción de resultados, validar la estructura de la base de datos y crear los scripts DDL necesarios.
        
- **Orlin Corea / Carlos Lenin Majano:** [Rol: Líder Técnico / Arquitecto]
    
    - Responsables de la supervisión técnica, definición de la arquitectura, provisión de acceso y resolución de bloqueos.
        
- **María José:** [Rol: Project Manager]
    
    - Responsable del seguimiento del proyecto y la gestión de tareas.
        
- **Lic. Natalia / Lidia:** [Rol: Experto de Dominio / Usuario Experto]
    
    - Personal de laboratorio consultado para validar flujos, catálogos y requerimientos funcionales.
        

### Metodología de Trabajo

- El equipo tiene reuniones de seguimiento semanales.
    
- La comunicación se centraliza en un canal de Slack (`#integracion-real-bacterologia`).
    
- Se requiere la documentación de decisiones y scripts (DDL) en repositorios o herramientas como Obsidian.
    
- Se utiliza un flujo de trabajo basado en ramas en Git (desarrollo en `dev`, despliegue desde `produccion`).
    

## DECISIONES CLAVE TOMADAS

1. **División de Responsabilidades:** Oned se encarga del servicio de comunicación con REAL (el "listener"), y Daniel se encarga de la API de recepción de resultados y la base de datos.
    
2. **Manejo de Datos en JSON:** Se acordó que el servicio de Oned será responsable de traducir los datos a un formato simple y directo que la API de Daniel pueda consumir sin lógica de negocio compleja (e.g., `true` se convierte en `"POSITIVO"`).
    
3. **Persistencia de Datos:** Los resultados de bacteriología se guardarán en las tablas `bacteria_resultado_encabezado` y `bacteria_resultado_detalle`. Además, se debe actualizar el estado de la prueba a finalizado (`prueba_finalizada = TRUE`) en la tabla `tubo_prueba`.
    
4. **Manejo de Estructuras de BD:** No se modificarán las tablas existentes que dan soporte a procesos actuales (como los del gobierno). Si se requieren campos adicionales, se agregarán a las tablas existentes o se crearán tablas nuevas. Todos los cambios de DDL deben ser documentados en scripts.
    
5. **Manejo de Diferencias de Catálogos:** Para resolver la diferencia en cómo Analiza y REAL definen los exámenes (Analiza es más específico, REAL más general), se creará una tabla de mapeo (`aux_examenes_real`) que relacione los exámenes de Analiza con la combinación de examen + tipo de muestra de REAL.
    

## ESTÁNDARES Y CONVENCIONES

- **Estructura de archivos:** Se debe seguir la estructura de módulos definida en el proyecto principal de Angular, aunque este proyecto es mayormente de backend.
    
- **Nomenclatura:** El código de barras del tubo tiene una longitud de 15 caracteres y se compone del número de orden más un secuencial de dos dígitos (ej. `[NroOrden]01`).
    
- **Git workflow:** Todo el desarrollo se realiza sobre la rama `dev`. El paso a producción es gestionado por un proceso de CI/CD.
    
- **Documentación:** Es mandatorio documentar los scripts de base de datos (DDL) y mantener un `README` actualizado.
    

## INTEGRACIONES Y DEPENDENCIAS

- **Sistema REAL:** Es el middleware principal con el que se integra el sistema. La comunicación se realiza vía tramas HL7/ASTM sobre TCP/IP o sockets.
    
- **Equipos Vitek:** Son los equipos físicos de bacteriología que procesan las muestras. La integración con ellos es indirecta a través de REAL.
    
- **RabbitMQ:** Servicio de mensajería utilizado para el procesamiento asíncrono de peticiones.
    
- **CRM1 y CRM2:** Sistemas internos de Analiza. CRM2 se usa para la creación y pago de órdenes. CRM1 se usa para la gestión del flujo de laboratorio (toma de muestra, validación, etc.). La nueva funcionalidad busca automatizar pasos que hoy son manuales en estos sistemas.
    

## PROBLEMAS CONOCIDOS Y CONSIDERACIONES

- **Múltiples Microorganismos por Cultivo:** Se identificó que REAL puede reportar múltiples microorganismos en un solo resultado de cultivo. La estructura de la tabla `bacteria_resultado_encabezado` de Analiza parece estar diseñada para un solo microorganismo por resultado. Se necesita validar con el experto de dominio (Lic. Natalia) si este escenario ocurre en la práctica en Analiza y, de ser así, rediseñar la estructura de la base de datos para soportarlo (posiblemente agregando una tabla intermedia entre `encabezado` y `detalle`).
    
- **Catálogo Extenso de Microorganismos:** REAL utiliza un catálogo de más de 7,400 microorganismos. Se debe validar con el laboratorio cuáles de estos son relevantes para Analiza para evitar la carga de datos innecesarios.
    
- **Seguridad de la API:** La API desarrollada por Daniel inicialmente no incluirá mecanismos de seguridad. Se planea protegerla posteriormente a través de un API Gateway en AWS, que gestionará la autenticación.
    
- **Concatenación de valores:** El sistema actual de Analiza almacena el valor y la unidad del resultado de un antibiótico en un solo campo de texto concatenado (ej. `"< = 0,12 µg/mL"`). La propuesta de separarlos en la nueva API fue pospuesta para no afectar los módulos existentes que dependen de esta estructura.
    

## GLOSARIO DE TÉRMINOS DEL PROYECTO

- **REAL:** Sistema intermediario (middleware) que conecta el LIS de Analiza con los equipos de bacteriología.
    
- **CRM1/CRM2:** Sistemas de gestión de relaciones con clientes de Analiza. CRM2 es más nuevo y gestiona la facturación; CRM1 maneja flujos de laboratorio más antiguos.
    
- **Trama:** Mensaje de texto con un formato estandarizado (ASTM o HL7) utilizado para la comunicación entre sistemas de laboratorio.
    
- **Examen:** Agrupación de pruebas que el cliente solicita (ej. Hemograma, Urocultivo).
    
- **Prueba:** Análisis individual que forma parte de un examen (ej. Conteo de glóbulos blancos). En bacteriología, suele haber una relación 1 a 1 entre examen y prueba.
    
- **Listener:** Servicio que se ejecuta de forma continua para "escuchar" y procesar mensajes de una cola (RabbitMQ) o un socket.
    
- **DDL (Data Definition Language):** Comandos de SQL para definir o modificar la estructura de la base de datos, como `CREATE TABLE` o `ALTER TABLE`.