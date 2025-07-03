---
tags:
  - Backend
  - Server
  - NodeJS
  - Agiles
---
![[Pasted image 20241107141507.png]]

## [Documentacion](https://www.serverless.com/framework/docs)



---
## Como funciona?
Hasta donde voy entendiendo esto sirva para no tener un servidor, pero estara conectado con [[DevOps/AWS/AWS]] y como gestor de bases de datos con [[DynamoDB]]
![[Pasted image 20241107141632.png]]



---
## Comandos

Instalar una versión en especifico
```shell
npm install -g serverless@3.30.1
```

Desinstalar
```Shell
npm uninstall -g serverless
```

Desplegar serverless
```Shell
serverless deploy --stage cooper --region us-east-1 --verbose
```

Desplegar de forma local 
```shell
serverless offline --stage <<Nombre del Ambiente>> --region us-east-1
```


