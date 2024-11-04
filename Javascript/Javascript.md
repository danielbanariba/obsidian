---
tags:
  - Javascript
  - Frontend
  - Backend
  - FullStack
---
---
## Comandos

Para ejecutar codigo javascript en la consola seria con
```shell
node index.js
```
para eso necesitamos tener instalado [[NodeJS]]



---
## Función tipo flecha

ES6 proporciona una característica conocida como Funciones de flecha. Proporciona una sintaxis más concisa para escribir expresiones de funciones eliminando las palabras clave "función" y "devolver".  
  
Las funciones de flecha se definen usando la flecha de grasa (`=>`) notación.

```jsx
// Arrow function
let sumOfTwoNumbers = (a, b) => a + b;
console.log(sum(10, 20)); // Output 30
```

Es evidente que no hay una palabra clave de "retorno" o "función" en la declaración de función de flecha.  

También podemos omitir el paréntesis en el caso de que haya exactamente un parámetro, pero siempre tendremos que usarlo cuando tenga cero o más de un parámetro.  
  
Pero, si hay múltiples expresiones en el cuerpo de la función, entonces debemos envolverlo con aparatos ortopédicos rizados ("{}"). También necesitamos usar la declaración de "devolución" para devolver el valor requerido.



---
## Asignación de Destructuración




4. Literales de Objetos Mejorados
5. Promesas


---
## Palabras reservadas

### **.at()** 
sirve para extraer información de una posición específica.
```jsx
const mensaje = "¡Hola Mundo!"; console.log(mensaje.at(-1)); // !
console.log(mensaje.at(-5));                                 // M
```

Imagina que tienes una fila de personas esperando el autobús. Con .at() puedes:
```jsx
const filaDePersonas = ["Ana", "Juan", "María", "Pedro", "Luis"]; 
// Si usas números positivos, cuenta desde el principio de la fila 
console.log(filaDePersonas.at(0));  // "Ana" (primera persona)
console.log(filaDePersonas.at(1));  // "Juan" (segunda persona) 

// Si usas números negativos, cuenta desde el final de la fila 
console.log(filaDePersonas.at(-1)); // "Luis" (última persona) 
console.log(filaDePersonas.at(-2)); // "Pedro" (penúltima persona)
```
Es como si tuvieras dos formas de contar en la fila:
1. Desde el principio: 1º, 2º, 3º... (usando números positivos)
2. Desde el final: último, penúltimo, antepenúltimo... (usando números negativos)


### ForEach
Iterera un arreglo para poder imprimir los datos dentro de un array
```jsx
let nombres = ['Daniel', 'Alejandro', 'Barrientos'];

for (let nombre of nombres){
	console.log(nombre);
}
```


### For In
Itera un objeto 
```jsx
let user ={
	id:1,
	name: 'Daniel',
	age: 27,
};

for (let prop in user) {
	console.log(prop, user[prop]);
}
```

> [!IMPORTANT ] 
> Utilizar siempre el For Of, ya que es lo nuevo y lo mas recomendado, pero se explica en for in, ya que se puede encontrar codigo que esta escrito de esa manera


### Switch
Existe una libreria que se comporta igual que un Switch y este se llama [[Redux]]
```jsx
let accion = 'actualizar';

switch (accion){
	case 'listar':
		console.log('Accion de listar');
		break;
	case 'guardar':
		console.log('Accion de guardar');
		break;
	default:
		console.log('Accion no reconocida');
}
```




---
## Short Circuit



---
## Objetos
