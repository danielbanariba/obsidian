---
tags:
  - Testing
  - QA
---
## Casos de Prueba

Norma ISO 29.119
- Identificador unico
- Objetivo
- Prioridad
- Trazabilidad
- Precondiciones
- Entradas
- Resultados Esperados
- Resultados Actuales

A ver xd lo que voy entendiendo ahorita es hacer todas las posibles combinaciones para en este caso crear un usuario por ejemplo e ir registrando todo con una descripción como en el ejemplo

![[Pasted image 20241030103447.png]]

También hay que ingresar los pasos que hice, con los resultados obtenidos 
![[Pasted image 20241030103819.png]]

Nuevamente otro ejemplo donde refuerza lo que dije anteriormente 
![[Pasted image 20241030104657.png]]

> [!IMPORTANT ] 
> No es lo mas adecuado hacer la documentación en una plantilla como Excel



---
## Precondición



---
## Creación de casos de prueba

Necesitaremos una platatilla, en esta caso usaremos [[Jira]], 



---
## Pasos para ser un buen tester

1) Identificación de requerimientos y especificaciones:


2) Casos de Pruebas
	- Creacion de casos
	- Ejecucion de casos
	- Reportar el estado del sistema



---
## Ejemplos de una Estructura de caso de prueba
``` python 
# Caso de Prueba: Login de Usuario

## ID
TC-LOGIN-001
BO-1

## Summary
Verificar que un usuario pueda iniciar sesión exitosamente con credenciales válidas

## Estado
**Estado Actual**: Passed _(Estados posibles):_

- **Not Run**: No ejecutado
- **Passed**: Prueba exitosa
- **Failed**: Prueba fallida
- **Blocked**: Bloqueado por dependencia
- **In Progress**: En ejecución
- **Deferred**: Pospuesto
- **Retest**: Necesita revalidación

## Preconditions
1. La aplicación está accesible y funcionando
2. El usuario está registrado en el sistema
3. Base de datos está operativa
4. Se tiene acceso a Internet

## Steps
1. Navegar a la página de login
2. Ingresar email válido en el campo correspondiente
3. Ingresar contraseña válida en el campo correspondiente
4. Hacer clic en el botón "Iniciar Sesión"

## Postconditions
1. Usuario ha iniciado sesión exitosamente
2. Token de sesión generado
3. Log de acceso registrado en base de datos

## Expected Results
- Usuario es redirigido al dashboard
- Se muestra mensaje de "Bienvenido [nombre_usuario]"
- Menú de usuario muestra opciones personalizadas

## Actual Results
[Se llena durante la ejecución de la prueba]

- Usuario redirigido correctamente ✅
- Mensaje de bienvenida mostrado ✅
- Menú personalizado visible ✅

## Historial de Estados

- 2024-01-30 10:00 - Not Run - Test case creado
- 2024-01-30 11:00 - In Progress - Iniciando ejecución
- 2024-01-30 11:15 - Passed - Prueba completada exitosamente
```

## Un buen caso de uso es:
```python
# Caso de Prueba: Compra de Producto en E-commerce

## ID
TC-PURCHASE-001

## Summary
Verificar el flujo completo de compra de un producto usando tarjeta de crédito

## Comprehensive (Completo)

### Cobertura de escenarios críticos:
- Selección de producto
- Gestión del carrito
- Proceso de pago
- Confirmación de orden

## Accurate (Preciso)

### Datos específicos de prueba:
- Producto: Laptop HP Model X123
- Precio: $999.99
- Cantidad: 1
- Método de pago: Visa ending in 4242

## Easy to understand and execute (Fácil de entender y ejecutar)

### Pasos detallados:
1. Acceder a la tienda online (URL: shop.example.com)
2. Buscar "Laptop HP Model X123" en la barra de búsqueda
3. Hacer clic en "Agregar al carrito"
4. Hacer clic en "Proceder al pago"
5. Completar formulario de envío:
    - Nombre: John Doe
    - Dirección: 123 Test St.
    - Ciudad: Testville
6. Ingresar datos de tarjeta:
    - Número: 4242 4242 4242 4242
    - Fecha: 12/25
    - CVV: 123

## Easy to trace (Fácil de rastrear)

### Requisitos relacionados:

- REQ-001: Búsqueda de productos
- REQ-002: Gestión de carrito
- REQ-003: Proceso de pago
- REQ-004: Confirmación de orden

## Repeatable (Repetible)

### Condiciones iniciales claras:
- Usuario registrado y con sesión iniciada
- Producto en stock
- Sistema de pagos operativo
- Datos de prueba disponibles

## Reusable (Reutilizable)

### Componentes modulares:
1. Login de usuario (TC-LOGIN-001)
2. Búsqueda de producto (TC-SEARCH-001)
3. Validación de carrito (TC-CART-001)
4. Proceso de pago (TC-PAYMENT-001)

## Expected Results
1. Orden creada exitosamente
2. Correo de confirmación recibido
3. Inventario actualizado
4. Transacción registrada

## Actual Results
[Se completa durante la ejecución]
```


## Ejemplo de hacer un buen test sin tanta redundancia
```python
# Caso de Prueba: Registro de Usuario Nuevo

## ID
TC-REG-001

## Summary
✅ Específico y concreto (evita abstracción): "Verificar que un usuario nuevo pueda registrarse exitosamente utilizando email, completar su perfil básico y recibir email de confirmación"

## Preconditions
1. El ambiente de prueba está en `test.example.com`
2. Base de datos limpia de usuarios de prueba
3. Servidor de correo configurado y funcionando
4. Datos de prueba preparados:
    - Email: [testuser@example.com](mailto:testuser@example.com)
    - Contraseña: Test2024#
    - Nombre: Juan Test
    - Teléfono: 9999-0000

## Steps
✅ Balance correcto de detalles:

1. Acceder a la página de registro:
    - Hacer clic en el botón "Crear cuenta" en la barra superior
    - Verificar que se muestre el formulario de registro
2. Completar formulario principal:
    - Email: [testuser@example.com](mailto:testuser@example.com)
    - Contraseña: Test2024#
    - Confirmar contraseña: Test2024#
    - Hacer clic en "Siguiente"
3. Completar información de perfil:
    - Nombre completo: Juan Test
    - Teléfono: 9999-0000
    - País: [Seleccionar de dropdown] Honduras
    - Hacer clic en "Completar registro"
4. Verificar bandeja de entrada:
    - Abrir el correo recibido de [no-reply@example.com](mailto:no-reply@example.com)
    - Hacer clic en el botón "Verificar cuenta"

## Expected Results
✅ Resultados específicos y verificables:

1. Registro exitoso:
    - Mensaje de éxito visible: "¡Registro completado!"
    - Redirección al dashboard
    - Datos del perfil visibles en el panel
2. Email de confirmación:
    - Recibido dentro de 5 minutos
    - Contiene nombre correcto del usuario
    - Link de verificación funcional
3. Base de datos:
    - Usuario creado con status "verified"
    - Timestamp de registro guardado
    - Perfil completo almacenado

## Notes
✅ Referencias claras y accionables (evita links no clickeables):

- Para acceder al panel de administración, usar el menú desplegable "Admin" en la esquina superior derecha
- Los logs de registro se encuentran en la sección "User Management > Logs"

## Test Data
✅ Datos específicos y completos:

json
`{   
  "user": {
    "email": "testuser@example.com",    
	"password": "Test2024#",    
	"profile": {      
		"fullName": "Juan Test",      
		"phone": "9999-0000",      
		"country": "Honduras"    
		}  
	} 
}`

## Estado
- Status: Not Run
- Prioridad: Alta
- Ambiente: Test
- Sprint: Sprint 24-01

## Defect Tracking
- Bug ID: [Se llena si se encuentra un defecto]
- Severidad: [Se llena si aplica]
- Notas de ejecución: [Se llenan durante el test]
```



---
## Testing Aplicaciones Mobiles

##### Pruebas De Interrupción:
- Recibir una llamada mientras se usa la aplicación
- Recibir Mensajes
- Modo Avión
- Desconectar el wifi y los datos
- Notificaciones
- Ir al Home y volver a la aplicación (Segundo Plano)
- Rota el teléfono
- Subir y bajar el volumen
- Bloquear y desbloquear el teléfono
- Alarmas
- Apagar el teléfono y volverlo a encender


En mi caso voy a utilizar el emulador Android Studio, Genymotion o Firebase para poder hacer pruebas en diferentes dispositivos mobiles.

Video explicativo de como instalar y usar Genymotion - https://youtu.be/vc5xnEDGHys?si=VZC4oHcxUtIswdt3





Tesing con [[Python]], [[React]]
