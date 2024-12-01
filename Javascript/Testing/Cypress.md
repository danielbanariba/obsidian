---
tags:
  - Testing
  - QA
  - Javascript
  - Frameworks
---
![[Pasted image 20241107181512.png]]

## [Documentacion](https://docs.cypress.io/app/get-started/why-cypress)



---
## Teoría
Es un frenkword que nos permite hacer [[Testing]] y automaizacion de pruebas, dicen por hay que es un remplazo de [[Selenium]] ya que [[Selenium]] no ejecutaba bien las pruebas, o hacia un poco de trapa y no hacian bien los test.

---
## Instalación

```shell
npm install cypress --save-dev
```



---
## Recomendación
En la documentación dice que en el archivo **package.json** en la seccion de scripts, le pongamos el siguiente codigo:
```jsx
{  
  "scripts": {  
    "cy:open": "cypress open"  
  }
}
```

y para ejecutar la aplicación con el siguiente Comandos
```shell
npm run cy:open
```

