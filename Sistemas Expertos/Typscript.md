---
tags:
  - Typescript
  - Programacion
---
---
**![[Pasted image 20240921002134.png]]
### [Documentacion](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)



---
### Instalación

Instalar el compilador de typescript
```bash
npm i -g typescript
```

Para poder compilar o traducir de typescript a javascript
```bash
tsc
```



---
### Configuración

Cambiar la configuración de typescript
```bash
tsc --init
```

Se nos va a crear un archivo con el nombre de **tsconfig.json** y adentro del archivo 
cambiamos de "target": "es2016" al mas reciente
```typescript
"target": "es2023",
```

de igual forma descomentamos esta linea y creamos la carpeta **src**
```Typescript
"rootDir": "./src",
```

Esta linea nos indica donde vamos a guardar los archivos javascript, cuando se compile el código typescript y obvio le ponemos el nombre de **dist** para que sea mas ordenado
``` Typescript
 "outDir": "./dist"
```

Descomentamos
```Typescript
"removeComments": true,
```

Por defecto viene el **true**, pero tenemos que ponerlo el false, ya que si no hacemos este cambio, igual nos va a generar código JavaScript aun con errores y obvio queremos que se nos muestre los errores al momento de estar programando y no en producción
```Typescript
"noEmitOnError": false,
```



---
### Sintaxis

Tipar una variable sea de un dato especifico, tenemos que poner **: 'El tipo de de dato'** en este caso como queremos que la variable **mensaje** sea de tipo String, entonces ponemos **: string**
```Typescript
let mensaje: string = 'Hola Mundo';
```



---
### Palabras reservadas

Es un forech en typescript, recorre los arrays
```typescript
.map()
```

ejemplo:
```typescript
let animales: string = ["Perro", "Gato", "Caballo"]

animales.map()
```

ENUM:
```Typescript
enum Talla { Chica = 's', 
			Mediana = 'm', 
			Grande = 'l', 
			ExtraGrande = 'xl' }

const variable1 = Talla.ExtraGrande;

console.log(variable1); // xl
```

Otro ejemplo de Enum
```Typescript
const enum LoadingState { Idle, 
						 Loading, 
						 Success, 
						 Error }

const stado = LoadingState.Success
```

