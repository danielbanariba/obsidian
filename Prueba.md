# QA Testing Template for API Endpoints

## 1. Información General

### 1.1 Contexto del Testing
Estoy realizando pruebas de QA para un endpoint API. Necesito ayuda para crear pruebas exhaustivas en Postman que validen tanto casos exitosos como casos de error.

### 1.2 Endpoint a Probar
**URL**: {{apiURL}}/visit/list/
**Método**: POST
## 2. Especificaciones

### 2.1 Acceptance Criteria
Send POST request with valid data. Expect 201 status and list of visits. Test with invalid data. Expect 400 status.


### 2.2 Descripción Técnica
Create Postman test cases for the /visit/list endpoint


## 3. Estructura de Datos

### 3.1 Request Body Esperado
```json
{
    "LastKey": null,
    "PageSize": 10
}
```

### 3.2 Ejemplo de Respuesta Exitosa
```json
{

    "result": {

        "List": [

            {

                "UpdatedAt": 1733428460330,

                "ExpiresAt": 1733435660329,

                "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

                "History": [],

                "TenantId": "C4PER",

                "Frequencies": [],

                "Id": "c18647e4-6d36-4577-a7cc-f96c22f19d3f",

                "Type": "Delivery",

                "Participants": [],

                "Name": "Test Resident",

                "CreatedAt": 1733428460330,

                "HasExpired": true

            },

            {

                "UpdatedAt": 1733428344829,

                "ExpiresAt": 1733435544828,

                "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

                "History": [],

                "TenantId": "C4PER",

                "Frequencies": [],

                "Id": "49b96402-576e-43df-9624-b11b1057f77b",

                "Type": "Delivery",

                "Participants": [],

                "Name": "Test",

                "CreatedAt": 1733428344829,

                "HasExpired": true

            },

            {

                "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

                "TenantId": "C4PER",

                "Id": "visit-009",

                "Type": "Party",

                "Participants": [

                    {

                        "History": [],

                        "Name": "John Smith"

                    }

                ],

                "Name": "Fiesta Pendiente",

                "CreatedAt": 1733353090000,

                "HasExpired": false

            },

            {

                "ExpiresAt": 1733439490000,

                "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

                "TenantId": "C4PER",

                "HasExpired": true,

                "Id": "visit-006",

                "Type": "OneTime",

                "Name": "Visita Denegada",

                "CreatedAt": 1733353090000

            },

            {

                "ExpiresAt": 1733439490000,

                "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

                "History": [

                    {

                        "Type": "In",

                        "CreatedAt": 1733353090000

                    }

                ],

                "TenantId": "C4PER",

                "HasExpired": true,

                "Id": "visit-001",

                "Type": "OneTime",

                "Name": "Visita de Prueba",

                "CreatedAt": 1733353090000

            },

            {

                "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

                "TenantId": "C4PER",

                "Frequencies": [

                    1,

                    3,

                    5

                ],

                "Id": "visit-011",

                "Type": "Recurring",

                "Name": "Visita Frecuente",

                "CreatedAt": 1733353090000,

                "HasExpired": true

            },

            {

                "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

                "TenantId": "C4PER",

                "Id": "visit-007",

                "Type": "Party",

                "Participants": [

                    {

                        "History": [

                            {

                                "Type": "In",

                                "CreatedAt": 1733353090000

                            },

                            {

                                "Type": "Out",

                                "CreatedAt": 1733353190000

                            }

                        ],

                        "Name": "Invitado VIP"

                    },

                    {

                        "History": [],

                        "Name": "Invitado Regular"

                    }

                ],

                "Name": "Fiesta Grande",

                "CreatedAt": 1733353090000,

                "HasExpired": false

            },

            {

                "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

                "TenantId": "C4PER",

                "Frequencies": [

                    1,

                    3,

                    5

                ],

                "HasExpired": true,

                "Id": "visit-003",

                "Type": "Recurring",

                "Name": "Visita Semanal",

                "CreatedAt": 1733353090000

            },

            {

                "ExpiresAt": 1733439490000,

                "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

                "TenantId": "C4PER",

                "HasExpired": true,

                "Id": "visit-002",

                "Type": "Party",

                "Participants": [

                    {

                        "History": [],

                        "Name": "Invitado 1"

                    },

                    {

                        "History": [],

                        "Name": "Invitado 2"

                    }

                ],

                "Name": "Fiesta de Cumpleaños",

                "CreatedAt": 1733353090000

            },

            {

                "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

                "History": [

                    {

                        "Type": "In",

                        "CreatedAt": 1733353090000

                    },

                    {

                        "Type": "Out",

                        "CreatedAt": 1733353190000

                    }

                ],

                "TenantId": "C4PER",

                "Frequencies": [

                    1,

                    3,

                    5

                ],

                "Id": "visit-008",

                "Type": "Recurring",

                "Name": "Mantenimiento Semanal",

                "CreatedAt": 1733353090000,

                "HasExpired": true

            }

        ],

        "LastKey": {

            "TenantId": "C4PER",

            "Id": "visit-008",

            "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",

            "CreatedAt": 1733353090000

        }

    },

    "error": null,

    "exception": null

}
```

### 3.3 Ejemplos de Respuestas de Error
```json
{
    "result": null,
    "error": null,
    "exception": {
        "code": "BadRequestException",
        "message": "LastKey must be an object",
        "stack": "BadRequestException: Bad Request Exception\n    at ValidationPipe.exceptionFactory (/var/task/src/lambda.js:2:513412)\n    at ValidationPipe.transform (/var/task/src/lambda.js:2:513085)\n    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)\n    at async /var/task/src/lambda.js:2:671199\n    at async Promise.all (index 0)\n    at async /var/task/src/lambda.js:2:671064\n    at async /var/task/src/lambda.js:2:668777\n    at async /var/task/src/lambda.js:2:668696\n    at async /var/task/src/lambda.js:2:677981\n    at async /var/task/src/lambda.js:2:675827"
    }
}
```

## 4. Tipos de Pruebas Requeridas

### 4.1 Casos Exitosos
- Validación de estructura de respuesta
  - Presencia de campos requeridos
  - Tipos de datos correctos
  - Formato de valores
- Validación de códigos de estado
- Validación de datos específicos de negocio
- Verificación de relaciones entre datos

### 4.2 Validaciones de Campos
- Campos requeridos vs opcionales
- Tipos de datos permitidos
- Rangos y límites
- Formatos específicos
- Validaciones de negocio

### 4.3 Casos de Error
- Campos faltantes
- Datos inválidos
  - Tipos incorrectos
  - Formatos inválidos
  - Valores fuera de rango
- Errores de autenticación/autorización
- Violaciones de reglas de negocio

### 4.4 Casos Edge
- Valores límite
- Caracteres especiales
- Valores nulos/vacíos
- Casos extremos de datos
- Escenarios de concurrencia

### 4.5 Pruebas de Paginación
- Primera página
- Páginas intermedias
- Última página
- Diferentes tamaños de página
- Manejo de LastKey/tokens de paginación

## 5. Scripts de Prueba

### 5.1 Formato para Documentar Pruebas
Para cada caso de prueba, incluir:

1. **Descripción del Caso**
   - Objetivo de la prueba
   - Precondiciones necesarias
   - Resultado esperado

2. **Request Body**
```json
[Body específico para esta prueba]
```

3. **Scripts de Prueba**
```javascript
[Scripts de Postman completos]
```

4. **Explicación**
   - Qué se está probando específicamente
   - Por qué es importante esta prueba
   - Cómo interpretar los resultados

5. **Posibles Escenarios de Fallo**
   - Qué puede fallar
   - Cómo identificar la falla
   - Acciones recomendadas

### 5.2 Ejemplo de Documentación de Prueba

#### Prueba: Listado Exitoso de Visitas
**Descripción**: Verificar que el endpoint retorna correctamente la lista de visitas con paginación.

**Request Body**:
```json
{
    "LastKey": null,
    "PageSize": 10
}
```

**Scripts de Prueba**:
```javascript
pm.test("Verificar status 201 Created", function () {
    pm.response.to.have.status(201);
});

pm.test("Verificar estructura de respuesta", function () {
    const response = pm.response.json();
    pm.expect(response).to.have.property("result");
    pm.expect(response.result).to.have.property("List");
    pm.expect(response.result).to.have.property("LastKey");
});

// ... más scripts específicos
```

**Explicación**:  
Esta prueba verifica la funcionalidad básica del endpoint, asegurando que:
- Retorna el código de estado correcto
- La respuesta tiene la estructura esperada
- Los datos cumplen con el formato requerido

**Posibles Fallos**:
- Error de autenticación
- Timeout en la respuesta
- Estructura de datos incorrecta
- Problemas de paginación

## 6. Ejemplos de Scripts Existentes
[Incluir aquí ejemplos de scripts que ya están funcionando y pueden servir como referencia]

## 7. Notas Adicionales
- Consideraciones especiales para el ambiente de pruebas
- Dependencias con otros sistemas
- Datos de prueba requeridos
- Limitaciones conocidas
- Mejores prácticas específicas del proyecto