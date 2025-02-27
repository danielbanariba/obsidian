---
tags:
  - QA
  - Testing
---
![[Pasted image 20250120093250.png]]



---
# 



---
## Comandos

Instalar la dependecia
```shell
npm init playwright@latest
```

Ejecutar los test
```
npx playwright test
```

Resumen de todo el reporte de los test que ejecutamos
``` shell
npx playwright show-report
```

Ejecutar muchos test en paralelo
```shell
npx playwright test --workers 3
```

Modo UI
```shell
npx playwright test --ui
```



---
### Configuración

En el archivo **playwright.config.ts** podemos agregar el siguientes configuraciones:
Buscamos la propiedad use:
```js
use:{
	headless: false, //se visualiza los diferentes navegadores
	video: 'on', // Se genera un video
}
```



---
### Métodos

Tomar capturas
```js
await page.screenshot({path: "./captures/" + Data.now() + "screenshot.jpg"}) 
```

