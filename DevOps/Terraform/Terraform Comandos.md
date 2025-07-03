---
tags:
  - Terraform
  - Comandos
  - Terminal
---
---
### Comandos 

Instalar [[Terraform]] 
```winget
winget install Hashicorp.Terraform
```


Configura el entorno de trabajo de [[Terraform]]
Posdata, tiene que estar ya programado el **main.tf** para que funcione el comando
```shell
terraform init 
```


Conectarnos en la plataforma de Azure 
```Shell
terraform plan
```

Se va a tardar un poco, pero nos tiene que devolver este resultado
![[Pasted image 20241003040806.png]]


Aplicar los cambios
``` Shell
terraform apply
```


Eliminar los recursos
```shell
terraform destroy
```

