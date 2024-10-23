---
tags:
  - Git
  - Backend
  - Github
  - Shell
  - Terminal
---
---
## Los comandos mas usados en Git

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
## Ramas

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



---
## Supervisar o analizar el repositorio

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

Ver todas las ramas del repositorio
```Shell
git branch
```



---
## Configuración Global

Agrega un nombre
```bash
git config –global user.name <<”Nombre de la persona”>>
```

Agrega un correo
```bash
git config –global user.email <<”La dirrecion de correo electronico”>>
```



---
## Si cometemos errores

Revertir commit a uno anterior
```Shell
git revert <<commit_ID>>
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

  

//Ignorar archivos que no queremos en nuestro repositorio

.gitignore  //Dentro del archivo ponemos **/<<Dirección de archivo>>

  
  
  

Donde me quede para aprender git: 

[https://youtu.be/3GymExBkKjE?si=pvjOrxIdSKY_Asju&t=4144](https://youtu.be/3GymExBkKjE?si=pvjOrxIdSKY_Asju&t=4144) 

  
  

//Aqui esta todo los pasos a seguir si se me olvida como hacer un repositorio y como subirlo a GitHub

[https://bluuweb.dev/03-git/02-git.html](https://bluuweb.dev/03-git/02-git.html) 

  

**Investigar sobre el Merge (Esta potente), pero lo que entendi es que lo que hace es unir diferentes ramas**