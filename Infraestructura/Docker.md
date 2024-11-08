---
tags:
  - Docker
  - Terminal
  - Comandos
---
![[Pasted image 20241020235257.png]]
### [Documentación](https://docs.docker.com/)


---
### Imagenes 
Se puede encontrar un monton de imagenes de muchas tecnologias
Docker Hush = https://hub.docker.com/ 


---
### Comandos

Nos permite ver cuantas imágenes tenemos instalado en nuestro Docker
``` bash
docker images
```

Eliminar una imagen
```Shell
docker image rm [nombre_imagen]
```

Crear un contenedor
```Shell
docker create [nombre_imagen]
```


```Shell
docker star [id_imagen]
```

Devuelve la información de los contenedores que tenemos (pero solo nos mostrara los contenedores que se encuentran ejecutandose)
```Shell
docker ps
```

Si queremos ver todos, incluso los que no se estan ejecutando
```Shell
docker ps -a
```

Detiene el contenedor
```Shell
docker stop [id_container]
```

---
### Port Mapping
