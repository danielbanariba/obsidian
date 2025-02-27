# LuQA Technical Document: Community Payments Feature

## Overview
The Community Payments feature allows residents within a home to upload proof of payment for community fees on a monthly basis. Each uploaded proof begins with a "Pending" status and requires admin validation to be marked as "Approved" or "Rejected". This feature ensures clear tracking and accountability for all users in a home.

## Evaluación de Soluciones

Para implementar la funcionalidad de pagos comunitarios en LuQA, se han considerado varias alternativas:

### Opción 1: Extensión de un módulo existente
**Descripción**: Extender un módulo existente (como reservas o anuncios) para incluir la funcionalidad de pagos.
**Ventajas**:
- Menor tiempo de implementación inicial
- Reutilización de código existente
- Menos cambios en la arquitectura global

**Desventajas**:
- Mezcla de responsabilidades en un solo módulo
- Posibles problemas de escalabilidad futura
- Mayor complejidad para el mantenimiento

### Opción 2: Nuevo módulo independiente
**Descripción**: Crear un nuevo módulo dedicado para gestionar los pagos comunitarios.
**Ventajas**:
- Separación clara de responsabilidades
- Mayor facilidad de mantenimiento
- Mejor escalabilidad para futuros requisitos
- Cumple con el principio de responsabilidad única

**Desventajas**:
- Mayor tiempo inicial de implementación
- Requiere integración con múltiples servicios existentes

### Opción 3: Servicio externo
**Descripción**: Desarrollar un microservicio separado para la gestión de pagos.
**Ventajas**:
- Máxima independencia y escalabilidad
- Posibilidad de utilizarse en otras aplicaciones

**Desventajas**:
- Complejidad significativamente mayor
- Tiempo de desarrollo más largo
- Sobrecarga para la integración y sincronización

### Solución recomendada
Después de evaluar las opciones, se recomienda implementar la **Opción 2: Nuevo módulo independiente** por las siguientes razones:
- Mantiene la arquitectura modular existente de LuQA
- Proporciona separación clara de responsabilidades
- Ofrece un buen equilibrio entre mantenibilidad y tiempo de desarrollo
- Permite escalabilidad para futuras mejoras
- Sigue el patrón arquitectónico existente (como se observa en los módulos de visitas y reservas)

## Architecture Updates
El Community Payments module será implementado como un nuevo módulo independiente dentro de la arquitectura NestJS existente de LuQA. Este módulo deberá seguir la estructura modular existente con controladores, servicios y esquemas, e integrarse con el sistema de notificaciones actual que utiliza AWS SNS. El módulo utilizará AWS S3 para el almacenamiento seguro de los documentos de pago.

## Data Model

### Payment Proof Table

| **Attribute**      | **Type**     | **Description**                                        |
|--------------------|--------------|--------------------------------------------------------|
| Id                 | `String`     | Unique payment proof identifier                        |
| HomeId             | `String`     | Identifier of the home                                 |
| Month              | `Number`     | Month for which payment was made (1-12)                |
| Year               | `Number`     | Year for which payment was made                        |
| Status             | `Enum`       | Status of the payment proof (Pending, Approved, Rejected) |
| DocumentUrl        | `String`     | URL to the stored document                             |
| IdUser             | `String`     | ID of user who uploaded the payment proof              |
| UploadedAt         | `Number`     | Timestamp when the document was uploaded               |
| ReviewedBy         | `String`     | ID of admin who reviewed the payment                   |
| ReviewedAt         | `Number`     | Timestamp when the payment was reviewed                |
| RejectionReason    | `String`     | Optional reason for rejection                          |

### User Schema (Referenced)

| **Attribute**      | **Type**     | **Description**                                        |
|--------------------|--------------|--------------------------------------------------------|
| Id                 | `String`     | Unique user identifier                                 |
| DisplayName        | `String`     | User's display name                                    |
| IsAdmin            | `Boolean`    | Flag indicating if user has admin privileges           |
| HomeId             | `String`     | Identifier of the home the user belongs to             |

## API Endpoints

### 1. Upload Payment Proof
- **Endpoint**: `POST /homes/{homeId}/payments`
- **Description**: Allows a resident to upload a payment proof for a specific month and year
- **Request Body**:
  ```json
  {
    "month": 2,
    "year": 2025,
    "document": "[Binary file data]"
  }
  ```
- **Response**: Returns the created payment proof object

### 2. Get Payment Proofs for Home
- **Endpoint**: `GET /homes/{homeId}/payments`
- **Description**: Retrieves all payment proofs for a specific home
- **Query Parameters**:
  - `year` (optional): Filter by year
  - `month` (optional): Filter by month
  - `status` (optional): Filter by status
- **Response**: Returns an array of payment proof objects

### 3. Admin Get All Payment Proofs
- **Endpoint**: `GET /admin/payments`
- **Description**: Allows admins to retrieve payment proofs across all homes
- **Query Parameters**:
  - `homeId` (optional): Filter by home
  - `year` (optional): Filter by year
  - `month` (optional): Filter by month
  - `status` (optional): Filter by status
- **Response**: Returns an array of payment proof objects

### 4. Review Payment Proof
- **Endpoint**: `PUT /admin/payments/{paymentId}/review`
- **Description**: Allows an admin to review a payment proof, setting it as approved or rejected
- **Request Body**:
  ```json
  {
    "status": "Approved",
    "rejectionReason": ""
  }
  ```
- **Response**: Returns the updated payment proof object

## Logic

### Payment Proof Upload
1. **Start**
   * Usuario navega a la sección "Comprobante de Pago"
   * Usuario selecciona un mes y año
   * Usuario carga un documento como comprobante de pago
2. **Validación**
   * Sistema valida que el usuario pertenece al hogar especificado
   * Sistema valida el tipo de archivo (PDF, JPEG, PNG)
   * Sistema verifica si ya existe un comprobante para ese mes y año
3. **Almacenamiento de Archivo**
   * Si es válido, el sistema carga el archivo a AWS S3
   * Sistema obtiene la URL del archivo almacenado
4. **Creación de Registro**
   * Sistema crea un nuevo registro de comprobante con:
     * Status establecido como "PENDING"
     * DocumentUrl desde S3
     * IdUser del residente que realizó la carga
     * UploadedAt con la marca de tiempo actual
5. **Notificación**
   * Sistema notifica a los administradores sobre el nuevo comprobante utilizando el NotificationService existente

### Payment Proof Review
1. **Start**
   * Admin navigates to the payment proofs list
   * Admin filters as needed by home, month, year, status
   * Admin selects a payment proof to review
2. **Review Process**
   * Admin views the proof of payment document
   * Admin decides to approve or reject the payment
   * If rejecting, admin may provide a reason
3. **Status Update**
   * System updates the payment proof record with:
     * New status (Approved or Rejected)
     * Admin information
     * Review timestamp
     * Rejection reason (if applicable)
4. **Notification**
   * System notifies all users in the home about the status change using AWS SNS

## Notification System Integration

La funcionalidad de Pagos Comunitarios debe integrarse con el sistema de notificaciones existente de LuQA que utiliza AWS SNS, siguiendo los patrones establecidos en los servicios de notificaciones existentes como NotificationVisitService y NotificationReservationService. Esta integración debería incluir:

1. **Tipos de Notificaciones**:
   * Confirmación de carga de pago al residente
   * Notificaciones de cambio de estado a todos los residentes en el hogar
   * Recordatorios de pagos pendientes a los administradores

2. **Canales de Notificación**:
   * Notificaciones push a través de la aplicación móvil utilizando la infraestructura AWS SNS existente
   * Notificaciones por correo electrónico (opcional)

3. **Contenido de las Notificaciones**:
   * Mensajes claros y concisos sobre el cambio de estado
   * Información contextual apropiada (mes, año, detalles del hogar)
   * Elementos de acción cuando corresponda (p. ej., "Por favor, revise el pago rechazado")

## Permisos

### Permisos de Residente
1. **PaymentProofUpload**: Permiso para cargar un nuevo comprobante de pago
2. **PaymentProofView**: Permiso para ver comprobantes de pago de su hogar
3. **PaymentProofHistory**: Permiso para ver el historial de pagos de su hogar

### Permisos de Administrador
1. **AdminPaymentProofView**: Permiso para ver todos los comprobantes de pago
2. **AdminPaymentProofReview**: Permiso para revisar y aprobar/rechazar comprobantes de pago
3. **AdminPaymentProofReport**: Permiso para generar informes sobre el estado de los pagos

## File Size Handling
Basado en el documento de investigación sobre problemas de tamaño de payload en API Gateway y Lambda, se deben implementar las siguientes limitaciones y soluciones:

1. **Limitaciones de Tamaño de Archivo**:
   * Tamaño máximo para cargas directas: 4 MB
   * Advertencia a los usuarios cuando los archivos se acerquen a este límite

2. **Para Archivos Más Grandes**:
   * Implementación de URLs prefirmadas de S3 para archivos mayores de 4 MB
   * Carga del lado del cliente directamente a buckets S3, evitando las limitaciones de tamaño de API Gateway y Lambda

3. **Manejo de Errores**:
   * Mensajes de error claros cuando se excedan los límites de tamaño de archivo
   * Orientación a los usuarios sobre cómo reducir el tamaño de los archivos (compresión, reducción de resolución)

## Consideraciones de Seguridad
1. **Almacenamiento de Documentos**: Todos los documentos cargados se almacenarán de forma segura en AWS S3 con acceso restringido
2. **Control de Acceso**: 
   * Los residentes solo pueden ver y cargar documentos para su propio hogar
   * Los administradores tienen acceso para revisar documentos de todos los hogares
3. **Validación de Archivos**: 
   * Validación estricta de tipos de archivo (PDF, JPEG, PNG)
   * Límites de tamaño de archivo para prevenir ataques de denegación de servicio
4. **Protección de Datos**:
   * Cifrado de información de pago sensible
   * Registro de auditoría de todos los accesos y cambios a los registros de pago

## Estrategia de Pruebas

### Pruebas Unitarias
1. **Carga de Archivos**:
   * Validar restricciones de tipo de archivo (solo PDF, JPEG, PNG)
   * Probar las limitaciones de tamaño de archivo y los mensajes de error apropiados
   * Verificar que los registros de comprobante de pago se creen correctamente

2. **Gestión de Estado de Pagos**:
   * Probar las transiciones de estado (Pendiente → Aprobado, Pendiente → Rechazado)
   * Validar que el rechazo requiera un motivo
   * Verificar que las marcas de tiempo y las referencias de usuario se registren correctamente

### Pruebas de Integración
1. **Flujo de Extremo a Extremo**:
   * Probar el proceso completo de carga, revisión y notificación
   * Verificar la integración con el sistema de almacenamiento de archivos (AWS S3)
   * Probar la entrega de notificaciones a todas las partes interesadas a través de AWS SNS

2. **Controles de Permisos**:
   * Verificar las restricciones de acceso de residentes a los registros de su propio hogar
   * Probar el acceso de administrador a todos los registros de pago
   * Validar que los intentos de acceso no autorizados sean rechazados correctamente

## Historias de Usuario y Tareas

### Historia de Usuario 1: Carga de Pago por Residente
**Como residente**, quiero cargar un comprobante de pago para un mes específico, para que el pago de mi hogar quede registrado.

**Tareas**:
1. Crear componente de UI para selección de mes/año
2. Implementar componente de carga de archivos con validación
3. Diseñar e implementar el diseño de la página de comprobante de pago
4. Implementar endpoint de API para cargas de comprobante de pago
5. Desarrollar sistema de notificación para cargas exitosas

### Historia de Usuario 2: Visualización de Estado de Pago
**Como residente**, quiero ver el estado de pago para cada mes.

**Tareas**:
1. Crear vista de calendario/cuadrícula mensual para estado de pago
2. Implementar indicadores de estado (Pendiente, Aprobado, Rechazado)
3. Mostrar motivos de rechazo cuando corresponda
4. Implementar filtrado por año para vista histórica

### Historia de Usuario 3: Interfaz de Revisión de Administrador
**Como administrador**, quiero revisar comprobantes de pago mensuales.

**Tareas**:
1. Crear dashboard de administrador para revisión de comprobante de pago
2. Implementar filtrado robusto (por hogar, mes, año, estado)
3. Desarrollar componente de visor de documentos
4. Crear interfaz de aprobación/rechazo con entrada de motivo
5. Implementar procesamiento por lotes para múltiples revisiones

## Cronograma de Implementación
1. **Semana 1**: Actualizaciones de esquema de base de datos y desarrollo de endpoint de API
2. **Semana 2**: Integración de carga y almacenamiento de archivos
3. **Semana 3**: Desarrollo de componentes de UI para residentes
4. **Semana 4**: Desarrollo de interfaz de administrador
5. **Semana 5**: Pruebas y refinamiento

## Monitoreo y Registro
1. **Métricas de Rendimiento**:
   * Tasa de éxito y duración de carga de archivos
   * Tiempos de respuesta de revisión
   * Rendimiento de endpoint de API

2. **Registro de Errores**:
   * Fallos de carga de archivos
   * Errores de procesamiento
   * Fallos de solicitud de API

3. **Registros de Auditoría**:
   * Cambios de estado de pago
   * Registros de acceso a documentos
   * Actividades de revisión de administrador
