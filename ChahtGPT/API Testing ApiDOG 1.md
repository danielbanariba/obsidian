Quiero que me ayudes a crear pruebas completas para APIs con Apidog. Para cada endpoint, necesito:

1. DESCRIPCIÓN GENERAL
- Una descripción detallada del endpoint y su propósito
- Los detalles técnicos como URL, método HTTP, parámetros, y permisos necesarios
- Cualquier restricción o regla de negocio relevante

1. SCHEMA
- Schema completo de request (si aplica)
- Schema completo de response
- Validaciones esperadas para el schema

1. GRUPOS DE PRUEBA
Necesito pruebas exhaustivas organizadas en estos grupos:

A. SUCCESS TESTS (200)
- Todas las combinaciones válidas de parámetros
- Casos especiales que devuelven respuestas exitosas
- Pruebas de límites aceptables
- Pruebas con diferentes tipos de datos válidos

B. BAD REQUEST TESTS (400)
- Parámetros faltantes
- Formatos inválidos
- Valores fuera de rango
- Tipos de datos incorrectos
- Validaciones de negocio fallidas

C. FORBIDDEN TESTS (403)
- Pruebas con permisos insuficientes
- Acceso a recursos de otro tenant
- Intentos de acciones no autorizadas

D. NOT FOUND TESTS (404)
- IDs inexistentes
- Recursos eliminados
- Tenants inexistentes

E. UNAUTHORIZED TESTS (401)
- Sin token
- Token expirado
- Token inválido
- Token malformado

1. DETALLES ESPECÍFICOS
Para cada prueba, necesito:
- Nombre descriptivo del test
- URL completa con todos los parámetros
- Body del request (si aplica)
- Headers necesarios
- Response esperada
- Explicación de lo que se prueba

1. SCRIPTS DE VALIDACIÓN
Necesito scripts para validar:
- Estructura de respuesta
- Códigos de estado
- Mensajes de error específicos
- Validación de datos en respuestas exitosas

Actualmente estoy trabajando con un backend en NestJS que usa DynamoDB, y las respuestas suelen seguir esta estructura para errores:
{
    "result": null,
    "error": null,
    "exception": {
        "code": "ErrorType",
        "message": "Mensaje específico",
        "stack": "Stack trace..."
    }
}

Para los casos de éxito, la estructura es:
{
    "items": [...],
    "lastKeyId": "string o null"
}

El código ya implementa guards para TenantGuard y PermissionsGuard que verifican:
- Que el usuario tenga el permiso específico (como Visit.View)
- Que el usuario tenga acceso al tenant especificado

Por favor, proporciona ejemplos concretos para cada caso de prueba con valores de ejemplo realistas.