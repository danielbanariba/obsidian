---
tags:
  - SQL
---
---
## Consultas SQl

Crear una sucursal
```sql
INSERT INTO public.roles (code, name, is_active, "createdAt", "updatedAt")
VALUES ('ADMIN', 'Administrador', true, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP);
```


Darle permisos a un usuario
```sql
UPDATE public.users 
SET role_id = 1, is_authorized = true 
WHERE email = 'banaribad@gmail.com';
```


