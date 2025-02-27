# Technical Design Document - Community Payments Feature

## 1. System Architecture Overview

### 1.1 DynamoDB Schema

```yaml
Resources:
  communityPaymentTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: ${opt:stage}-CommunityPayment
      AttributeDefinitions:
        - AttributeName: TenantId
          AttributeType: S
        - AttributeName: Id
          AttributeType: S
        - AttributeName: Month
          AttributeType: S
        - AttributeName: Status
          AttributeType: S
        - AttributeName: CreatedAt
          AttributeType: N
      KeySchema:
        - AttributeName: TenantId
          KeyType: HASH
        - AttributeName: Id
          KeyType: RANGE
      GlobalSecondaryIndexes:
        - IndexName: Month-index
          KeySchema:
            - AttributeName: TenantId
              KeyType: HASH
            - AttributeName: Month
              KeyType: RANGE
          Projection:
            ProjectionType: ALL
        - IndexName: Status-index
          KeySchema:
            - AttributeName: Status
              KeyType: HASH
            - AttributeName: CreatedAt
              KeyType: RANGE
          Projection:
            ProjectionType: ALL
      BillingMode: PAY_PER_REQUEST
```

### 1.2 API Endpoints

#### Resident Endpoints:
```typescript
// Upload payment proof
POST /payment/upload
{
  month: string, // Format: "YYYY-MM"
  document: File, // PDF, JPEG, or PNG
}

// List payments
POST /payment/list
{
  lastKey?: LastKey,
  pageSize: number,
  month?: string // Optional filter by month
}

// Get payment details
GET /payment/{paymentId}
```

#### Admin Endpoints:
```typescript
// List all payments
POST /admin/payment/list
{
  lastKey?: LastKey,
  pageSize: number,
  status?: "Pending" | "Approved" | "Rejected",
  month?: string
}

// Review payment
POST /admin/payment/{paymentId}/review
{
  status: "Approved" | "Rejected",
  rejectionReason?: string
}
```

## 2. Data Models

### 2.1 Payment Model
```typescript
interface CommunityPayment {
  TenantId: string;
  Id: string;
  Month: string;                 // YYYY-MM format
  Status: "Pending" | "Approved" | "Rejected";
  DocumentUrl: string;           // S3 URL
  UploadedBy: string;           // User ID
  CreatedAt: number;            // Unix timestamp
  UpdatedAt: number;            // Unix timestamp
  ReviewedBy?: string;          // Admin User ID
  ReviewedAt?: number;          // Unix timestamp
  RejectionReason?: string;     // Required if status is Rejected
}
```

### 2.2 Response Models
```typescript
interface PaymentResponse {
  result: {
    payment: CommunityPayment;
  } | null;
  error: string | null;
  exception: {
    code: string;
    message: string;
    stack?: string;
  } | null;
}

interface PaymentListResponse {
  result: {
    payments: CommunityPayment[];
    lastKey?: LastKey;
  } | null;
  error: string | null;
  exception: {
    code: string;
    message: string;
    stack?: string;
  } | null;
}
```

## 3. Implementation Details

### 3.1 File Storage
- Utilizar el bucket S3 existente: `luqa-storage-bucket-${opt:stage}`
- Estructura de carpetas en S3:
  ```
  /payments/{tenantId}/{year}/{month}/{paymentId}.{extension}
  ```
- Implementar URLs firmadas para acceso seguro a los documentos

### 3.2 Security Considerations
- Validar permisos de usuario para cada operación
- Garantizar que los residentes solo puedan ver pagos de su propia casa
- Implementar límite de tamaño de archivo (máx. 10MB)
- Validar tipos de archivo permitidos (PDF, JPEG, PNG)
- Sanitizar nombres de archivo antes de subir a S3

### 3.3 Performance Considerations
- Implementar paginación para listados
- Utilizar GSIs para consultas eficientes por mes y estado
- Comprimir imágenes grandes antes de almacenar
- Utilizar caché para respuestas frecuentes

## 4. Error Handling

### 4.1 HTTP Status Codes
- 201: Creación exitosa
- 200: Operación exitosa
- 400: Error de validación
- 401: No autorizado
- 403: Prohibido
- 404: Recurso no encontrado
- 500: Error interno del servidor

### 4.2 Error Codes
```typescript
enum PaymentErrorCodes {
  INVALID_FILE_TYPE = "INVALID_FILE_TYPE",
  FILE_TOO_LARGE = "FILE_TOO_LARGE",
  PAYMENT_NOT_FOUND = "PAYMENT_NOT_FOUND",
  UNAUTHORIZED_ACCESS = "UNAUTHORIZED_ACCESS",
  INVALID_STATUS = "INVALID_STATUS",
  MISSING_REJECTION_REASON = "MISSING_REJECTION_REASON"
}
```

## 5. Implementation Plan

### 5.1 Phase 1: Infrastructure
1. Crear tabla DynamoDB
2. Configurar rutas en S3
3. Implementar manejo de archivos
4. Configurar permisos IAM

### 5.2 Phase 2: Core API
1. Implementar endpoint de subida
2. Implementar endpoints de listado
3. Implementar endpoint de detalles
4. Añadir validaciones base

### 5.3 Phase 3: Admin Features
1. Implementar revisión de pagos
2. Implementar filtros administrativos
3. Implementar notificaciones

### 5.4 Phase 4: Testing & Polish
1. Pruebas unitarias
2. Pruebas de integración
3. Documentación de API
4. Optimizaciones de rendimiento

## 6. Testing Strategy

### 6.1 Unit Tests
- Validación de archivos
- Lógica de negocio
- Manejo de errores
- Formateo de respuestas

### 6.2 Integration Tests
- Flujo completo de subida
- Flujo de revisión
- Paginación
- Permisos y seguridad

### 6.3 Performance Tests
- Carga de listados grandes
- Concurrencia en subidas
- Tiempo de respuesta

## 7. Monitoring & Logging

### 7.1 CloudWatch Metrics
- Latencia de endpoints
- Tasa de errores
- Uso de almacenamiento
- Tasa de aprobación/rechazo

### 7.2 Logging
- Errores de validación
- Accesos no autorizados
- Operaciones de administrador
- Errores de S3
