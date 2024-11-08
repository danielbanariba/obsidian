---
tags:
  - Backend
  - Testing
---
![[Pasted image 20241105155858.png]]
Jest es una libreria para poder hacer [[Testing]] aplicaciones [[React]], [[Nestjs]], [[NEXT.js]], y todo lo relacionado de [[JavaScript]]



---
## Comandos

Instalación
```shell
npm i -D jest supertest
```

Ejecutar los test
```shell
npx jest
```

```shell
set NODE_OPTIONS=--experimental-vm-modules && npx jest
```

> [!IMPORTANT ] 
> Podemos personalizar comandos
---

Nos vamos al archivo de package.json y dentro del apartado **scripts** ponemos, posdata, ese comando es de windows!
```node
"scripts":{
	"test": "set NODE_OPTIONS=--experimental-vm-modules && jest"
}
```



---


