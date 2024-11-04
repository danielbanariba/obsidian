---
tags:
  - QA
  - Testing
---
---
## Prueba de Software

1.- La prueba de software ¿que Son?
* Verificar si cumple con los requisitos esperados
* Manuales y Automaticas
* Que el software esta sin bugs (Errores)

2.- ¿Por qué es importante la prueba de software?
* Para identificar temprano los errores
* Confiable
* Satisfaccion del cliente 

3.- ¿Cuál es la necesidad de las pruebas?
* Perdidas economicas 
* perdidas humanas 
* Desprestigio Empresa

4.- ¿Cuáles son los beneficios de las pruebas de software?
* Rentable ahorramos dinero
* Seguridad 
* calidad del software 100% minimo
* Satisfaccion del cliente

5.- Tipos de pruebas de software
* Pruebas funcionales ¿El software cumple con lo esperado?
    - Pruebas unitarias
    - Pruebas de integracion 
    - Pruebas de sistema
    - Pruebas de aceptacion
* Pruebas no funcionales
    - Pruebas de rendimiento 
    - Pruebas de seguridad
    - Pruebas de usabilidad
* Mantenimiento
    - Regression 
    - Mantemiento



---
## Gestión de Pruebas



---
## Gestión de Errores



---
## Casos de prueba
Ejemplos:
https://platzi.com/tutoriales/1421-pruebas-software/9574-maneras-de-representar-los-casos-de-prueba/

#### Template 
Título del Caso de Prueba:
Verificación del Inicio de Sesión en la Aplicación

Objetivo:
Probar el proceso de inicio de sesión en la aplicación.

Descripción: 
El propósito de esta prueba es verificar que el inicio de sesión de la aplicación estén funcionando correctamente en los navegadores Chrome, Firefox, etc.

Precondiciones:
- La aplicación está instalada en el dispositivo.
- El usuario tiene credenciales válidas.

Pasos:
1. Abrir la aplicación de la pantalla de inicio.
2. Hacer clic en el botón "Iniciar Sesión".
3. Ingresar el nombre de usuario y la contraseña en los campos correspondientes.
4. Hacer clic en el botón "Iniciar Sesión".

Resultado Esperado:
El usuario debe ser redirigido a la página de inicio después de iniciar sesión correctamente.

Criterios de Éxito:
Después de hacer clic en "Iniciar Sesión", el usuario es redirigido a la página de inicio.
No hay mensajes de error mostrados durante el proceso de inicio de sesión.

#### Ejemplo
Escenario: Inicio de sesión en una aplicación de redes sociales
Descripción: Verificar el proceso de inicio de sesión en una aplicación de RS. 

Caso de prueba 1. Verificar las credenciales correctas
Caso de prueba 2. Verificar el login con credenciales incorrectas. 
Caso de prueba 3. Verificar multiples intentos de login con credenciales incorrectas.



---
## Plantilla de Reporte de Error
![[Pasted image 20241031101245.png]]





---
## Ciclo de vida del Software [[Testing]]
![[Pasted image 20241031155043.png]]



---
## Modelo V

#### ¿Que es el Modelo V?
	Analisis                                Pruebas 
	    Diseño                          Pruebas 
	        Implementacion          Pruebas 
	            Verificacion    Pruebas 
	                Mantenimiento 



---
## Fases STLC

1) Análisis de requisitos
2) Planificación de pruebas
3) Desarrollo de casos de prueba
4) Configuración del entorno de prueba
5) Ejecución de pruebas
6) Cierre del ciclo de prueba



---
## Automatización de pruebas

Se puede usar la librería Selelium de [[Python]] 

Selelium = Plataformas web
Appium = Plataforma Android e IOS
JUnit = Para desarrollo en [[Java]]
Cucumber = Basado en componentes, es compatible con varios lenguajes de programación
Pytest = Desarrollo en [[Python]]
Unit = [[.NET]]



---
## Pruebas Unitarias



---
## Pruebas de integración
Hacer pruebas reponsibe (lo que yo siempre hago xd) para mejor la experiencia usar la aplicación 
**ResponsivelyApp** 


---
## Pruebas de Sistema
Probar la app en diferentes navegadores, en diferentes sistemas operativos, si es móvil, probarlo en IOS y Android.

Posdata: Nota personal, supongo que al intentar probarlo en diferentes plataformas o en este caso sistemas, el trabajo va ser muy repetitivo, entonces hay si podre automatizarlo con [[Python]]



---
## SvS
Smoke vs Sanity Testing


La pruebas Smoke son las pruebas superficiales que solo verifican las funciones básicas, en cambio las Sanity son las pruebas mas profundas.



---
## Pruebas de Regresión

- Asegurar que los cambios recientes en el código no hayan roto nada. 

1) Identificar las Funcionalidades a Probar
2) Crear un Conjunto de Casos de Prueba
3) Automatizar Casos de Prueba
4) Ejecutar las Pruebas Automatizadas
5) Análisis de Resultados
6) Repetir el Proceso

Consejos Adicionales:
- Mantén una Suite de Pruebas Robusta
- Automatización Eficiente
- Versionamiento



---
## Pruebas No Funcionales




---
## Documentación



---
## Escenario de Prueba
Título del Escenario: [Describe brevemente qué se está probando]
Objetivo: [Propósito de la prueba, qué se quiere lograr]
Precondiciones: [Condiciones necesarias para ejecutar la prueba, por ejemplo, datos preexistentes, configuraciones específicas]
Pasos de Ejecución: [Pasos específicos para ejecutar la prueba, acciones que el probador debe realizar]
Criterios de Éxito: [Cómo se determinará si la prueba es exitosa]
Postcondiciones: [Estado esperado del sistema después de completar la prueba]



---
## Generacion de datos

##### ¿Qué es la Generación de Datos de Prueba?
La generación de datos de prueba se refiere al proceso de crear conjuntos de datos sintéticos que imitan las características y estructuras de los datos reales

##### Herramientas para Generación de Datos de Prueba:
1. Mockaroo: 
2. Faker (en [[Python]]): 
3. [[SQL]] Data Generator: 
4. DataFactory (en [[Azure]]): 
5. Postman:






