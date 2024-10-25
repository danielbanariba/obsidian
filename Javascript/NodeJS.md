---
tags:
  - NodeJS
  - Backend
  - Terminal
  - npm
  - Comandos
  - Javascript
---
![[Node.js_logo.svg]]
### [Documentación](https://nodejs.org/docs/latest/api/)

---
### Teoría
Es un entorno de ejecución (Runtime environment) 

[[NodeJS]] esta basada en la arquitectura basada en eventos (everlook)



---
### CommonJS

#### Exportando funciones
Para exportar una función a otro archivo, así como se hace en [[Java]] o [[Python]], Lo que tenemos que hacer es poner **module.exports**, ejemplo
```Javascript
function sum(a,b){
	return a + b
}

module.exports = {
	sum
}
```
Ponemos los corchetes para que se respecte el nombre de la función, y para importar la función en el archivo que nosotros queremos es
```Javascript
const { sum } = require('./sum')

console.log(sum(1,2))
```

> [!IMPORTANT ] 
> El metodo que vimos arriba es la versión antigua, lo que se recomienda en la version moderna, osea la del ejemplo de abajo

```Javascript
export function sum(a,b){
	return a + b
}
```



---
### Módulos

##### Módulos nativos de NodeJS
siempre que vayamos a usar un modulo nativo en JavaScript, siempre usaremos el node:el nombre del modulo

```javascript
const fs = requiere('node:fs/promises')
```



---
### Asíncrono vs Síncrono 

Consejo de un Senior es siempre utilizar Asíncrono, ya que no nos podemos dar el lujo de esperar que una ejecución termine para seguir con el proceso, por eso hay que evitar la sincronía ya que tiene que esperar que una tarea se termina para ejecutar lo demás. 
![[Pasted image 20241024230205.png]]

![[Pasted image 20241024230559.png]]

![[Pasted image 20241024230854.png]]

---
### Promesas

