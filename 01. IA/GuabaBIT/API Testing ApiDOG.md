Necesito crear tests completos para un endpoint de API en Apidog. NO quiero tests de Postman.

Para el endpoint, requiero:

1. DESCRIPCIÓN DEL ENDPOINT
- Descripción detallada de la funcionalidad
- URL completa con estructura de ruta
- Método HTTP
- Headers requeridos
- Parámetros (path, query o body) con descripción de cada uno

1. SCHEMA COMPLETO
- Schema exacto de request (si aplica)
- Schema exacto de response con todas las propiedades
- El schema debe ser en formato JSON y compatible con Apidog

1. CASOS DE PRUEBA ESPECÍFICOS
Para cada grupo de pruebas (Success, Bad Request, Forbidden, etc.), necesito:
- Nombre exacto del test (para copiar y pegar)
- URL completa con parámetros
- Body del request exacto (en JSON si aplica)
- Respuesta esperada con formato exacto

1. GRUPOS DE PRUEBA
Organiza los tests en estos grupos:
- SUCCESS (200/201)
- BAD REQUEST (400)
- FORBIDDEN (403)
- NOT FOUND (404)
- UNAUTHORIZED (401)

Para cada grupo, necesito todas las combinaciones posibles de parámetros y escenarios.

1. IMPORTANTE: NO INCLUYAS SCRIPTS DE POSTMAN
- No generes código JavaScript para tests de Postman
- Solo proporciona los casos de prueba, URLs y cuerpos de request para Apidog

El formato de respuesta esperado para errores es:
{
    "result": null,
    "error": null,
    "exception": {
        "code": "TipoDeError",
        "message": "Mensaje específico",
        "stack": "Stack trace..."
    }
}

Para casos de éxito es:
{
    "items": [...],
    "lastKeyId": "string o null"
}

Por favor, sé extremadamente específico con los parámetros y valores de ejemplo para que pueda copiarlos directamente.