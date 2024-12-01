---
tags:
  - Git
  - Backend
  - Github
  - Shell
  - Terminal
---
---
### Los comandos mas usados en Git

Guardar los cambios de forma temporal (esto es para poder hacer un **git pull** sin problemas)
```shell
git stash
```

Forzar los cambios del repositorio remoto al local
```shell
git reset --hard origin/feature/LQ-427
```



---
### Ramas

Crear una nueva rama
```Shell
git branch <<Nombre de la rama>>
```

Cambiar de rama 
```shell
git checkout <<Nombre de la rama>>
```

Unir ramas
```Shell
git merge 
```

Ver todas las ramas del repositorio
```Shell
git branch
```



---
### Supervisar o analizar el repositorio

Ver el estado de la base de datos
```bash
git status
```

Miras todos los cambios históricos hechos en el repositorio
```bash
git show
```

Mira detalles y modificaciones de un archivo
```bash
git show <<Nombre del archivo>>
```

Mira la historia entera de un archivo o mostrar todos los commits realizados
```bash
git log <<Nombre del archivo>>
```


---
### Configuración Global

Agrega un nombre
```bash
git config –global user.name <<”Nombre de la persona”>>
```

Agrega un correo
```bash
git config –global user.email <<”La dirrecion de correo electronico”>>
```



---
### Configuración al Nivel de proyecto

Consultar que nombre se encuentra en el proyecto actual
``` shell
git config user.name
```

``` shell
git config user.email
```


```shell
git config user.name "Daniel Barrientos"
```

```shell
git config user.email "dbarrientos@guababit.com"
```



---
### Si cometemos errores

Revertir commit a uno anterior
```Shell
git revert <<commit_ID>>
```






---
### Comando que ya me se de memoria

Hace el repositorio
```bash
git init 
```

Agrega el archivo al repositorio
```bash
git add <<El nombre del archivo>>
```

Agrega todos los archivos al repositorio
```bash
git add .
```

Es el que manda los últimos cambios a la base de datos del control de versiones
```bash
git commit -m “Version 1”
```

Manda el repositorio a un servidor remoto
```bash
git push -u origin <<Nombre de la rama>>
```

Trae todos los archivos del servidor remoto a la maquina local
```bash
git pull origin <<Nombre de la rama>>
```



---

### Curso de MoureDEV

  

//Volver al pasado

git checkout <<Nombre del archivo>>

  

//Diabloooooos, esto si esta bueno xdddd crear alias, son como especies de variables que lo //podemos utilizar para evitar recordar comandos tan largos

git config –global alias.<<Nombre del alias>> “El comando que queremos guardar”

git config –global alias.tree “log –graph –decorate –all –oneline”

//Y ejecutamos el siguiente comando

git tree





Donde me quede para aprender git: 

[https://youtu.be/3GymExBkKjE?si=pvjOrxIdSKY_Asju&t=4144](https://youtu.be/3GymExBkKjE?si=pvjOrxIdSKY_Asju&t=4144) 

  
  

//Aqui esta todo los pasos a seguir si se me olvida como hacer un repositorio y como subirlo a GitHub

[https://bluuweb.dev/03-git/02-git.html](https://bluuweb.dev/03-git/02-git.html) 

  

**Investigar sobre el Merge (Esta potente), pero lo que entendi es que lo que hace es unir diferentes ramas**




