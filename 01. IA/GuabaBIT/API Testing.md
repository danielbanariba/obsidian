# API Testing Prompt Template

## 1. Información del Endpoint

### Endpoint
```
URL: [URL del endpoint, ejemplo: {{apiURL}}/visit/create]
Método: [GET/POST/PUT/DELETE]
```

### Headers Requeridos
```json
{
    "Content-Type": "application/json",
    "Authorization": "Bearer [token]"
}
```

## 2. Criterios de Aceptación y Descripción

### Acceptance Criteria
- [Lista numerada de criterios de aceptación]
- Ejemplo:
  1. Send POST request with valid visit data. Expect 201 status.
  2. Test with missing fields. Expect 400 status.

### Descripción Funcional
[Descripción detallada de qué hace el endpoint y su propósito]

## 3. Estructura de Datos

### Request Body Esperado
```json
[Estructura del body con todos los campos posibles]
```

### Ejemplo de Respuesta Exitosa
```json
[Ejemplo de respuesta cuando todo sale bien]
```

### Ejemplos de Respuestas de Error
```json
[Ejemplos de respuestas de error comunes]
```

## 4. Pruebas Existentes (si hay)
[Lista de pruebas que ya existen para este endpoint]

## 5. Ejemplo de Implementación

### Ejemplo de un Test Exitoso
```javascript
[Código de ejemplo de una prueba exitosa]
```

### Ejemplo de un Test de Error
```javascript
[Código de ejemplo de una prueba de error]
```

---

## Ejemplo Completo (basado en /visit/create)

### Endpoint
```
URL: {{apiURL}}/visit/create
Método: POST
```

### Acceptance Criteria
1. Send POST request with valid visit data. Expect 201 status.
2. Test with missing fields. Expect 400 status.

### Descripción Funcional
Endpoint para crear nuevas visitas en el sistema. Permite registrar diferentes tipos de visitas (Delivery, Party, OneTime, Recurring) y maneja la inicialización automática de campos opcionales como Frequencies.

### Request Body
```json
{
    "Name": "Test Visit",
    "Type": "Delivery",
    "Frequencies": []
}
```

### Respuesta Exitosa
```json
{
    "result": {
        "Name": "Test Visit",
        "Type": "Delivery",
        "Frequencies": [],
        "Id": "12a36c33-d8e2-461a-8d8b-86410f7da87b",
        "TenantId": "C4PER",
        "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",
        "ExpiresAt": 1736968732591,
        "CreatedAt": 1736795932593,
        "UpdatedAt": 1736795932593,
        "History": [],
        "Participants": []
    },
    "error": null,
    "exception": null
}
```

### Test de Ejemplo
```javascript
pm.test("Successful POST request with valid visit data", function () {
    pm.response.to.have.status(201);
    
    const response = pm.response.json();
    pm.expect(response).to.have.property("result");
    pm.expect(response.error).to.be.null;
    pm.expect(response.exception).to.be.null;
    
    const visit = response.result;
    pm.expect(visit).to.include.all.keys(
        "Id",
        "Name",
        "Type",
        "CreatedAt",
        "UpdatedAt",
        "ExpiresAt",
        "Frequencies"
    );
});
```

### Notas Adicionales
- Los campos Name y Type son obligatorios
- Frequencies es opcional y se inicializa como array vacío si no se proporciona
- Los tipos válidos son: Delivery, Party, OneTime, Recurring