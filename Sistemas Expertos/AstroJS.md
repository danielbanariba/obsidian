---
tags:
  - Frontend
  - AstroJS
aliases:
---
---

![[Pasted image 20240917193417.png]]
Documentacion: [https://docs.astro.build/en/getting-started/](https://docs.astro.build/en/getting-started/) 

---
### Comandos

Crear un proyecto en astro: 
```bash
npm create astro@latest 
```

Subirlo a producción es: 
```bash
npm run build (la carpeta dist estan los archivos)
```

Obtener los comandos: 
```bash
npx astro add --help
```

Levantar el servidor: 
```bash
npm run dev
```



---
### Sintaxis

Es un componente especial de Astro, el titulo tiene que ir obligatoriamente  
```HTML
<Layout title="About">
	Hola Astro!
</Layout>
```

Esto sirve para renderizar cada una de las paginas
```HTML
<slot>
```



---
### Colecciones

El ejemplo que vamos a seguir es tener una colección de libros que queremos vender en una pagina, entonces tenemos que crearnos 2 carpetas que van hacer donde contenga los libros, archivos
``` 
content/books/"Varios archivos de libro"
```

dentro de la carpeta content, tenemos que crear un archivo con el nombre de **config.ts** y procedemos a importar las librerías: 
```Typescript
import { defineCollection, z} from "astro:content";
```















---
NOTA:
Aprender Typescript


Slot =  es como una especia de variable para poder remplazar un componente, es para evitar reescribir todo
```html
<Slot>
```


EL .map() escomo un forech
```typescript
.map()
```

mdx son archivos markodown (o como se escriba xd) 

---
Palabras que buscar
render 
