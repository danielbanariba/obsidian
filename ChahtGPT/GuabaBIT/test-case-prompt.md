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
# Test Case: Quote_Submission_Modal
## Título: Verificar Funcionalidad del Componente de Cotización

### Pasos de Prueba:
1. **Acción:** Hacer clic en una solicitud de la tabla
   **Datos:** Solicitud REQ-1 con detalles existentes (Toyota Corolla 2019)
   **Resultado Esperado:** Modal de cotización se abre con detalles pre-poblados

2. **Acción:** Ingresar precio válido
   **Datos:** Precio = 150.00
   **Resultado Esperado:** Campo de precio acepta entrada numérica

3. **Acción:** Ingresar cantidad válida
   **Datos:** Cantidad = 77
   **Resultado Esperado:** Campo de cantidad acepta entrada numérica

4. **Acción:** Agregar comentarios
   **Datos:** "Se aplican términos de entrega estándar"
   **Resultado Esperado:** Comentarios se guardan en área de texto

5. **Acción:** Subir archivo válido
   **Datos:** Tipo de archivo: PNG, tamaño < 800x400px
   **Resultado Esperado:** Archivo se carga exitosamente con vista previa

6. **Acción:** Hacer clic en "Enviar cotización"
   **Datos:** Todos los campos del formulario completados
   **Resultado Esperado:** Mensaje de éxito se muestra, modo de vista muestra cotización creada

### Escenarios Negativos:
1. **Acción:** Enviar formulario con precio vacío
   **Datos:** Campo de precio en blanco
   **Resultado Esperado:** Error de validación mostrado, envío bloqueado

2. **Acción:** Enviar formulario con cantidad inválida
   **Datos:** Cantidad = 0 o número negativo
   **Resultado Esperado:** Error de validación mostrado, envío bloqueado

3. **Acción:** Subir formato de archivo inválido
   **Datos:** Tipo de archivo: EXE
   **Resultado Esperado:** Mensaje de error mostrado, carga rechazada

4. **Acción:** Subir archivo demasiado grande
   **Datos:** Tamaño de archivo > límite máximo
   **Resultado Esperado:** Mensaje de error mostrado, carga rechazada