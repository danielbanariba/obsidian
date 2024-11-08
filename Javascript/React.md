---
tags:
  - React
  - Javascript
  - Frontend
---
![[Pasted image 20241101161102.png]]

## Estados y vida util
Doc: https://legacy.reactjs.org/docs/state-and-lifecycle.html

---
## Comandos

Para poder empezar a usar REACT, necesitaremos tener instalado [[NodeJS]] y en la consola de Comandos escribimos lo siguiente para crear un proyecto en react.
```shell
npx create-react-app <<"Nombre de la aplicacion">>
```

Levantar la aplicacion
```shell
npm start
```



---
## Componentes

Existen dos tipos de componentes que son los **Funcionales** y **De Clase**

#### Componente funcionales
Es aquella función que retorna un elemento de React
```jsx
function Saludo(props) {
	return <h1> Hola, {props.nombre}!</h1>;
}
```

#### Componente De Clase
```jsx
class Saludo extends React.Component{
	render(){
		return <h1> Hola, {this.props.nombre}!</h1>;
	}
}
```



---
## Estados

Para empezar a usar estados necesitamos importar useState
```jsx
import React, {useState} from 'react'
```

Hook

Event Listener



---
## Atajos de teclado

Atajo de teclado para recrear un componente rápido: **rfc** 



---
## Stylesheets
Aqui es donde agregaremos los estilos [[CSS]]



---
## Props



---















La sintaxis de react se parece como si estuvieramos combinando [[JavaScript]] con [[HTML]]


---
## React Testing Library

Instalar
```shell
npm install --save-dev @testing-library/react
```

> [!IMPORTANT ] 
> Cuando vayamos hacer un test a un archivo o compoenente, se tiene que llamar exactamente igual pero con la extencion de **test.js**

importamos la librerias para hacer los test
```jsx
import { render, screen, fireEvent } from '@testing-library/react';
```



Ejecuta todos los test que tengamos
```shell
npm test
```








En el proyecto de [[React]] Siempre vamos a tener un archivo que se llama **App.test.js** este archivo nos va a permitir hacer diferentes [[Testing]]









#### Glosario

Vamos a describir el comportamiento que tiene en este caso **Login** 
```jsx
describe('Login', () => {})
```

Mock
```jsx
const mockedUser = {
	name:'Daniel',
	email" 'danielbanariba@protonmail.com'
}
```



---
## [[Testing]]
Para hacer las pruebas vamos a usar la libreria de [Testing Libray](https://testing-library.com/docs/)
