**Comandos en Git

//Hace el repositorio

```bash
git init 
```


  

 //Agrega el archivo al repositorio

git add <<El nombre del archivo>>

  

//Agrega todos los archivos al repositorio

git add .

  

//Es el que manda los últimos cambios a la base de datos del control de versiones

git commit -m “Version 1”

  

//Ver el estado de la base de datos

git status

  

//Miras todos los cambios históricos hechos en el repositorio

git show

  

//Mira la historia entera de un archivo

git log <<Nombre del archivo>>

  

//Manda el repositorio a un servidor remoto

git push

  

//Crear carpetas

mkdir <<Nombre de la carpeta>>

  

//Crear archivos

touch <<Nombre del archivo con su extencion>>

  

//Ver el contenido de un archivo

cat <<Nombre del archivo>>

  

//Eliminar el archivo

rm<<Nombre del archivo>>

  

//Agrega un nombre

git config –global user.name <<”Nombre de la persona”>>

  

//Agrega un correo

git config –global user.email <<”La dirrecion de correo electronico”>>

  

//Mira el historial de un archivo

git log <<Nombre del archivo>>

  
  

//Mira detalles y modificaciones de un archivo

git show <<Nombre del archivo>>

  
  
  

Curso de MoureDEV

  

//Crea un archivo en blanco

touch <<Nombre del archivo>>

  

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