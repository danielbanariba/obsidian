# Prompt Template for Test Cases

Para generar un caso de prueba estructurado, usa el siguiente formato:

```
Genera un test case detallado para [FUNCIONALIDAD] que incluya:
- Pasos numerados
- Datos de prueba necesarios
- Resultados esperados
- Escenarios negativos

El formato debe seguir esta estructura:
# Test Case: [NOMBRE]
## Title: [DESCRIPCIÓN BREVE]

### Test Steps:
1. **Action:** [ACCIÓN A REALIZAR]
   **Data:** [DATOS NECESARIOS]
   **Expected Result:** [RESULTADO ESPERADO]

[continuar con pasos numerados...]

### Negative Scenarios:
1. **Action:** [ACCIÓN A REALIZAR]
   **Data:** [DATOS NECESARIOS]
   **Expected Result:** [RESULTADO ESPERADO]

[continuar con escenarios negativos...]
```

Ejemplos de uso:

1. Para login:
```
Genera un test case detallado para inicio de sesión que incluya:
- Pasos numerados
- Datos de prueba necesarios
- Resultados esperados
- Escenarios negativos como credenciales inválidas y campos vacíos
```

2. Para registro:
```
Genera un test case detallado para registro de usuario que incluya:
- Pasos numerados
- Datos de prueba necesarios
- Resultados esperados
- Escenarios negativos como datos inválidos y validaciones de campos
```

3. Para cualquier otra funcionalidad:
```
Genera un test case detallado para [TU FUNCIONALIDAD] que incluya:
- Pasos numerados
- Datos de prueba necesarios
- Resultados esperados
- Escenarios negativos relevantes para la funcionalidad
```



---
