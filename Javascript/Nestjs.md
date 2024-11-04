---
tags:
  - Nestjs
  - Javascript
  - Backend
---
![[Pasted image 20241022223702.png]]

[Documentacion](https://docs.nestjs.com/)



--- 
## Teoria
Nest esta basado en [[Express]], pero si no quieres trabajar con Express, con una sola linea te puedes cambiar a [[Fastify]].



---
## Comandos

Tenemos que instalarlo de manera global 
```shell
npm i -g @nestjs/cli
```

crear un nuevo proyecto
```shell
nest new [project-name]
```

Crea una proyecto pero si ningún archivo especifico 
```
nest g res [project-name] --no-spec 
```

A veeeeeeeer, este comando lo puedo usar bastante cuando haga [[Testing]], debido a que poner validaciones y tranforma string en number y asi, [link del video](https://youtu.be/Qet5I3Y5qsg?si=h-HU-ZJGQUN4ByXa)
```shell
npm i class-validator class-transformer
```

Compila el código de [[TypeScript]] a [[JavaScript]] (todo el codigo va estar en la carpeta dist)
```shell
npm rum build
```



---
## Arquitectura Monolítica
![[Pasted image 20241104102140.png]]
![[Pasted image 20241104102208.png]]



---
## Conectar a una BD



---
## Config



---
## [[Testing]]
