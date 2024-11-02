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

Para poder empezar a usar REACT, necesitaremos tener instalado [[NodeJS]] y en la consola de [[Comandos]] escribimos lo siguiente para crear un proyecto en react.
```shell
npm create-react-app <<"Nombre de la aplicacion">>
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















La sintaxis de react se parece como si estuvieramos combinando [[Javascript]] con [[HTML]]


En el proyecto de [[React]] Siempre vamos a tener un archivo que se llama **App.test.js** este archivo nos va a permitir hacer diferentes [[Testing]]