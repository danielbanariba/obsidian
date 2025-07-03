---
tags:
  - Backend
  - Python
  - FastAPI
aliases:
  - Programación en Python
---
---
![[logo-teal.png]][Documentación](https://fastapi.tiangolo.com/tutorial/)

---
## Comandos

Instalacion 
```shell
pip install "fastapi[standard]"
```

Levantar el servidor
```shell
fastapi dev
```



---
## URL

Documentacion generada automaticamente
http://127.0.0.1:8000/docs



---
## Funciones asincrona




---
## Pydantic
 Es una libreria que viene incluido en el fastAPI y sirve para validar los datos, ya que el usuario es terco y puerde incresar mal los datos.

para poder definirlo o llamarlo, tenemos que importar el modulo de Pydantic
```python
from pydantic import BaseModel

class Formulario(BaseModel):
    name: str
    description: str | None
    email: str
    age: int
```

y despues podemos crear un endpoint especifico donde haga referencia a esos datos
```python
@app.post("/formulario")
async def create_customer(data: Formulario):
    return data
```


---
## Conectar la base de datos
SQLModel 

#### [Documentacion](https://sqlmodel.tiangolo.com/)

Instalar
```shell
pip install sqlmodel
```


