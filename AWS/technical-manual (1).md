# Manual Técnico - Sistema Campus Virtual
## 1. Especificaciones Técnicas Base

### 1.1 Requerimientos Funcionales
- Login/Logout
- Gestión de sesiones con timeout de 30 minutos
- Subida y descarga de archivos
- Navegación de cursos
- Roles de usuario (estudiante, profesor, administrador)

### 1.2 Restricciones Técnicas
- Tamaño máximo de archivos: 4MB
- Tiempo de sesión: 30 minutos
- Tipos de archivo: Sin restricción

## 2. Arquitectura del Sistema

### 2.1 Estructura de Endpoints

#### Sistema de Autenticación
- `POST /api/auth/login`
  - Input: username, password
  - Output: session_token, expiration_time
  - Status codes: 200 (éxito), 401 (no autorizado)

- `POST /api/auth/logout`
  - Input: session_token
  - Output: success_status
  - Status codes: 200 (éxito), 401 (sesión inválida)

- `GET /api/auth/session-status`
  - Input: session_token
  - Output: is_valid, remaining_time
  - Status codes: 200 (éxito), 401 (sesión inválida)

- `POST /api/auth/extend-session`
  - Input: session_token
  - Output: new_expiration_time
  - Status codes: 200 (éxito), 401 (sesión inválida)

#### Sistema de Archivos
- `POST /api/files/upload`
  - Input: file (multipart/form-data), metadata
  - Output: file_id, upload_status
  - Status codes: 200 (éxito), 413 (archivo muy grande)

- `GET /api/files/{fileId}`
  - Input: file_id
  - Output: file_content, metadata
  - Status codes: 200 (éxito), 404 (no encontrado)

#### Sistema de Cursos
- `GET /api/courses`
  - Input: user_id, role
  - Output: lista_cursos
  - Status codes: 200 (éxito), 403 (no autorizado)

- `GET /api/courses/{courseId}`
  - Input: course_id
  - Output: course_details
  - Status codes: 200 (éxito), 404 (no encontrado)

### 2.2 Estructura de Datos

#### Sesión
```json
{
  "session_id": "string",
  "user_id": "string",
  "role": "string",
  "created_at": "timestamp",
  "expires_at": "timestamp",
  "last_activity": "timestamp"
}
```

#### Archivo
```json
{
  "file_id": "string",
  "filename": "string",
  "size": "number",
  "upload_date": "timestamp",
  "uploader_id": "string",
  "course_id": "string"
}
```

#### Curso
```json
{
  "course_id": "string",
  "name": "string",
  "description": "string",
  "teacher_id": "string",
  "students": ["string"]
}
```

## 3. Flujos de Proceso

### 3.1 Gestión de Sesiones
1. Inicio de sesión:
   - Validar credenciales
   - Generar token de sesión
   - Establecer tiempo de expiración (30 minutos)
   - Almacenar datos de sesión

2. Verificación de sesión:
   - Verificar existencia del token
   - Comprobar tiempo de expiración
   - Actualizar último acceso
   - Devolver estado actual

3. Extensión de sesión:
   - Validar sesión actual
   - Actualizar tiempo de expiración
   - Devolver nueva fecha de expiración

4. Cierre de sesión:
   - Invalidar token
   - Limpiar datos de sesión

### 3.2 Manejo de Archivos
1. Subida:
   - Validar tamaño (máx 4MB)
   - Generar ID único
   - Almacenar archivo
   - Registrar metadata

2. Descarga:
   - Verificar permisos
   - Recuperar archivo
   - Enviar contenido

## 4. Aspectos de Implementación

### 4.1 Gestión de Sesiones
- Implementar mecanismo de cleanup para sesiones expiradas
- Mantener registro de sesiones activas
- Implementar verificación de sesión en cada request
- Manejar concurrencia en actualizaciones de sesión

### 4.2 Manejo de Archivos
- Implementar validación de tamaño antes de procesamiento
- Establecer timeout para subidas
- Implementar manejo de chunks para archivos grandes
- Manejar limpieza de archivos temporales

### 4.3 Seguridad
- Sanitización de inputs
- Validación de tipos de archivo
- Verificación de permisos por rol
- Manejo seguro de sesiones

## 5. Monitoreo y Métricas

### 5.1 Métricas Clave
1. Rendimiento:
   - Tiempo de respuesta por endpoint
   - Latencia de operaciones de archivo
   - Tiempo de procesamiento de sesiones

2. Recursos:
   - Uso de memoria
   - Uso de CPU
   - I/O de disco
   - Conexiones activas

3. Errores:
   - Tasa de errores por endpoint
   - Timeouts
   - Errores de validación

### 5.2 Logs
- Formato estándar: `timestamp | level | service | message | metadata`
- Niveles: ERROR, WARN, INFO, DEBUG
- Información crucial: user_id, session_id, request_id

## 6. Pruebas de Rendimiento

### 6.1 Escenarios de Prueba
1. Login concurrente:
   - 100 usuarios simultáneos
   - Medición de tiempo de respuesta
   - Verificación de manejo de sesiones

2. Subida de archivos:
   - Archivos de diferentes tamaños
   - Subidas simultáneas
   - Verificación de límites

3. Navegación de cursos:
   - Carga de listas extensas
   - Acceso concurrente
   - Caché y optimización

### 6.2 Métricas a Evaluar
- Tiempo de respuesta promedio
- Percentil 95 de tiempo de respuesta
- Uso de recursos por request
- Tasa de errores
- Throughput máximo

## 7. Consideraciones de Implementación

### 7.1 Base de Datos
- Índices recomendados
- Estructura de tablas
- Optimización de queries
- Manejo de conexiones

### 7.2 Caché
- Datos de sesión
- Metadata de archivos
- Información de cursos
- Estrategias de invalidación

### 7.3 Escalabilidad
- Manejo de estado
- Sesiones distribuidas
- Almacenamiento de archivos
- Balanceo de carga
