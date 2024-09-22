---
tags:
  - Typescript
  - Programacion
---
---
**![[Pasted image 20240921002134.png]]
Documentacion: https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html

---
Instalar el compilador de typescript
```bash
npm i -g typescript
```

Para poder compilar o traducir de typescript a javascript
```bash
tsc "nombre-del-archivo.ts"
```

---
### Configuracion

Cambiar la configuración de typescript
```bash
tsc --init
```
Se nos va a crear un archivo con el nombre de tsconfig.json y adentro del archivo 
cambiamos de "target": "es2016" al mas reciente
```typescript
"target": "es2023",
```

de igual forma descomentamos esta linea y creamos la carpeta **src**
```Typescript
"rootDir": "./src",
```

Esta linea nos indica donde vamos a guardar los archivos javascript, cuando se compile el codigo typescript y obvio le ponemos el nombre de **dist** para que sea mas ordenado
``` Typescript
 "outDir": "./dist"
```

Descomentamos
```Typescript
"removeComments": true,
```

Por defecto viene el **true**, pero tenemos que ponerlo el false, ya que si no hacemos este cambio, igual nos va a generar codigo javascritp aun con errores, obvio si esta en true
```Typescript
"noEmitOnError": false,
```

---

### Sintaxis

Tipar una variable sea de un dato especifico, tenemos que poner **: 'El tipo de de dato'** en este caso como queremos que la variabla **mensaje** sea de tipo String, entonces ponemos **: string**
```Typescript
let mensaje: string = 'Hola Mundo';
```


---
