---
tags:
  - Comandos
  - Javascript
  - NodeJS
  - Terminal
---
## Comandos

Inicializa un nuevo proyecto
```shell 
npm init 
```

Si tenemos hueva de iniciar la vaina, podemos -y para poner todos los valores por defecto
```shell
npm init -y
```

Desinstalar los paquetes
```shell
npm uninstall [nombre-del-paquete]
```

Dependencias de desarrollo (lo que entendí es que hacemos eso ya que tenemos que separar las dependencias que tengamos instaladas del producción y el de desarrollo, ya que si subimos todo al servidor, ocuparan dependencias que no necesitamos y por eso consumirá recursos innecesarios.)
```shell
npm install standard -D
```

OJO! utilizar en el **gitbash**
Eliminar toda la carpeta de Node Modules
```shell
rm -rf ./node_modules
```

