---
tags:
  - Comandos
---
---

Saber el ip publico de que estoy conectado
```powershell
Invoke-RestMethod -Uri "https://ipinfo.io/ip"
```

Saber el ip publico de que estoy conectado
```bash
curl ifconfig.me
```

Generar la documentacion completa de un proyecto por medio de Docker 
```shell
docker run --rm -v C:\Users\banar\Desktop\agroservicio\backend-agroservicio:/app -w /app -p 8080:8080 node:18-alpine sh -c "npm install -g @compodoc/compodoc && compodoc -p tsconfig.json --theme material --language es-ES --disablePrivate --disableSourceCode --hideGenerator --serve --port 8080 --watch"
```
