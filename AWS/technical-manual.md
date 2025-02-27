# Manual Técnico - Implementación Campus Virtual
## Descripción General del Sistema

Este documento describe la implementación de un sistema básico de campus virtual para fines de investigación y análisis de rendimiento. El objetivo es comparar el rendimiento entre diferentes lenguajes de programación y frameworks.

## Estructura Base

### Componentes Principales
1. Sistema de Autenticación
2. Gestión de Archivos
3. Navegación de Cursos
4. Gestión de Sesiones

### Endpoints Principales

#### 1. Autenticación
```
POST /api/auth/login
POST /api/auth/logout
GET /api/auth/session-status
POST /api/auth/extend-session
```

#### 2. Gestión de Archivos
```
POST /api/files/upload
GET /api/files/{fileId}
DELETE /api/files/{fileId}
GET /api/files/list
```

#### 3. Navegación de Cursos
```
GET /api/courses
GET /api/courses/{courseId}
GET /api/courses/{courseId}/materials
```

## Implementaciones por Lenguaje

### 1. Python (FastAPI)
```python
from fastapi import FastAPI, File, UploadFile
from fastapi.middleware.cors import CORSMiddleware
import uvicorn
from datetime import datetime, timedelta

app = FastAPI()

# Configuración de sesión
SESSION_TIMEOUT = timedelta(minutes=30)

@app.post("/api/auth/login")
async def login():
    # Implementación básica de login
    return {"session_id": "test_session", "expires_at": datetime.now() + SESSION_TIMEOUT}

@app.post("/api/files/upload")
async def upload_file(file: UploadFile):
    # Validación de tamaño (4MB máximo)
    if await file.read(4_000_000 + 1):
        return {"error": "File too large"}
    # Implementación de subida
    return {"file_id": "test_file_id"}
```

### 2. Java (Spring Boot)
```java
@RestController
@RequestMapping("/api")
public class AuthController {
    
    private static final int SESSION_TIMEOUT_MINUTES = 30;
    
    @PostMapping("/auth/login")
    public ResponseEntity<LoginResponse> login() {
        LocalDateTime expiresAt = LocalDateTime.now().plusMinutes(SESSION_TIMEOUT_MINUTES);
        return ResponseEntity.ok(new LoginResponse("test_session", expiresAt));
    }
    
    @PostMapping("/files/upload")
    public ResponseEntity<FileResponse> uploadFile(@RequestParam("file") MultipartFile file) {
        if (file.getSize() > 4_000_000) {
            return ResponseEntity.badRequest().body(new FileResponse("File too large"));
        }
        return ResponseEntity.ok(new FileResponse("test_file_id"));
    }
}
```

### 3. Node.js (Nest.js)
```typescript
@Controller('api')
export class AuthController {
  private readonly SESSION_TIMEOUT = 30 * 60 * 1000; // 30 minutes in milliseconds

  @Post('auth/login')
  login() {
    const expiresAt = new Date(Date.now() + this.SESSION_TIMEOUT);
    return { session_id: 'test_session', expires_at: expiresAt };
  }

  @Post('files/upload')
  @UseInterceptors(FileInterceptor('file'))
  uploadFile(@UploadedFile() file) {
    if (file.size > 4_000_000) {
      throw new BadRequestException('File too large');
    }
    return { file_id: 'test_file_id' };
  }
}
```

### 4. PHP (Laravel)
```php
class AuthController extends Controller
{
    private const SESSION_TIMEOUT = 30; // minutes

    public function login(Request $request)
    {
        $expiresAt = now()->addMinutes(self::SESSION_TIMEOUT);
        return response()->json([
            'session_id' => 'test_session',
            'expires_at' => $expiresAt
        ]);
    }

    public function uploadFile(Request $request)
    {
        $request->validate([
            'file' => 'required|file|max:4000'
        ]);

        return response()->json([
            'file_id' => 'test_file_id'
        ]);
    }
}
```

### 5. Go
```go
package main

import (
    "net/http"
    "time"
)

const sessionTimeout = 30 * time.Minute

func login(w http.ResponseWriter, r *http.Request) {
    expiresAt := time.Now().Add(sessionTimeout)
    json.NewEncoder(w).Encode(map[string]interface{}{
        "session_id": "test_session",
        "expires_at": expiresAt,
    })
}

func uploadFile(w http.ResponseWriter, r *http.Request) {
    r.ParseMultipartForm(4 << 20) // 4MB
    file, handler, err := r.FormFile("file")
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    defer file.Close()

    if handler.Size > 4_000_000 {
        http.Error(w, "File too large", http.StatusBadRequest)
        return
    }

    json.NewEncoder(w).Encode(map[string]string{
        "file_id": "test_file_id",
    })
}
```

## Puntos de Medición para Análisis de Rendimiento

### Métricas Clave
1. Tiempo de Respuesta
   - Latencia promedio por endpoint
   - Percentil 95 de tiempo de respuesta
   - Tiempo máximo de respuesta

2. Throughput
   - Solicitudes por segundo
   - Conexiones concurrentes máximas

3. Uso de Recursos
   - Consumo de CPU
   - Uso de memoria
   - Uso de disco I/O

4. Gestión de Sesiones
   - Tiempo de procesamiento de login/logout
   - Overhead de mantenimiento de sesiones
   - Tiempo de verificación de sesión

### Herramientas Recomendadas para Pruebas
1. Apache JMeter
2. K6
3. Artillery
4. Prometheus (para métricas)
5. Grafana (para visualización)

## Consideraciones de Implementación

### Manejo de Sesiones
- Implementar temporizador de 30 minutos
- Almacenar timestamp de última actividad
- Verificar estado de sesión en cada request
- Implementar mecanismo de extensión de sesión

### Subida de Archivos
- Validación de tamaño máximo (4MB)
- Verificación de tipo de archivo
- Gestión de almacenamiento temporal
- Limpieza de archivos temporales

### Optimizaciones Recomendadas
1. Implementar caché para:
   - Sesiones activas
   - Datos de cursos frecuentemente accedidos
   - Metadatos de archivos

2. Configuración de conexiones a base de datos:
   - Pool de conexiones
   - Timeouts apropiados
   - Consultas optimizadas

3. Gestión de recursos:
   - Liberación apropiada de recursos
   - Manejo de conexiones persistentes
   - Gestión de memoria eficiente

## Notas para la Investigación

Para obtener resultados comparativos significativos, se recomienda:

1. Mantener condiciones de prueba consistentes:
   - Mismo hardware
   - Misma configuración de red
   - Mismo conjunto de datos de prueba

2. Realizar pruebas en diferentes escenarios:
   - Carga normal
   - Carga pico
   - Pruebas de estrés
   - Recuperación tras fallos

3. Documentar variables adicionales:
   - Versión del lenguaje/framework
   - Configuración del servidor
   - Configuración de la base de datos
   - Parámetros de tuning del sistema operativo
