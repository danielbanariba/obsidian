---
tags:
  - Typescript
  - Programacion
---
---
**![[Pasted image 20240921002134.png]]
### [Documentacion](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)



---
## Instalación

Instalar el compilador de typescript
```bash
npm i -g typescript
```

Para poder compilar o traducir de typescript a javascript
```bash
tsc
```



---
## Configuración

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
## Sintaxis

Tipar una variable sea de un dato especifico, tenemos que poner **: 'El tipo de de dato'** en este caso como queremos que la variable **mensaje** sea de tipo String, entonces ponemos **: string**
```Typescript
let mensaje: string = 'Hola Mundo';
```



---
## Palabras reservadas

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



---
## Decoradores
Los decoradores son una envoltura, que envuelva la función original para poder darles mas poder, mas métodos u obtener mas información, 

como por ejemplo el siguiente codigo, que tenemos un decorador que nos hace medir el tiempo que tarda en obetener los datos del arreglo
```typescript
// Creamos un decorador para medir el tiempo
function medirTiempo(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // Guardamos la función original
    const metodoOriginal = descriptor.value;

    // Creamos una nueva función que envuelve a la original
    descriptor.value = function(...args: any[]) {
        const inicio = performance.now();
        
        // Ejecutamos el método original
        const resultado = metodoOriginal.apply(this, args);
        
        const fin = performance.now();
        console.log(`El método ${propertyKey} tardó ${fin - inicio}ms`);
        
        return resultado;
    }
}

// Ahora usamos el decorador
class Usuarios {
    @medirTiempo
    obtenerUsuarios() {
        return ['Juan', 'María', 'Pedro'];
    }
}
```


Los decoradores son mas utlizados para:

- Validacion de datos
```Typescript
class Formulario {
    @validarEmail  // Verifica que sea un email válido
    email: string;

    @validarLongitud(8)  // Verifica que tenga al menos 8 caracteres
    password: string;
}
```

- Control de Acceso
```Typescript
class Admin {
    @soloAdmin  // Solo permite ejecutar si el usuario es administrador
    borrarUsuario(id: number) {
        // Código para borrar usuario
    }
}
```

- Logging Automatico
```Typescript
class Banco {
    @registrarOperacion  // Registra cada operación realizada
    transferir(monto: number, destino: string) {
        // Código para transferir dinero
    }
}
```

