---
tags:
  - Backend
  - Terraform
---
---
### Instalación
https://developer.hashicorp.com/terraform/install?product_intent=terraform#windows

---
Entrar session de [[Azure]]
```bash
az login
```

Seleccionamos la seccion donde queremos comenzar
![[Pasted image 20241002215933.png]]

---
### Configuración

main.tf
```javascript
provider "azurerm" { // Proveedor de Azure
  features {} // Configuraciones globales
  subscription_id = [ID] // ID de suscripción de Azure
}
```

"RG" es el nombre que le damos a la definición para usarlo después en la configuración de los otros recursos 
```java
// Recurso de grupo de recursos
resource "azurerm_resource_group" "rg" {
  name     = "rg-terraform" //rg-[nombre del proyecto que estamos trabajando]
  location = "East US"
  
  tags = { // Etiquetas del recurso
      environment = "dev" // Etiqueta de ambiente
      project     = "otd" // Etiqueta de proyecto
      created_by  = "terraform" // Etiqueta de creador
    }
}
```

---
## Comandos 

Configura el entorno de trabajo de Terraform
Posdata, tiene que estar ya programado el main.tf para que funcione el comando
```shell
terraform init 
```

En el control de versiones de [[Git]], en el archivo **.gitignore**, tenemos que ignorar estos archivos 
```git
*.tfstate
*.tfstate.*
*.tfvars
.terraform/
.terraform.*
```

Conectarnos en la plataforma de Azure 
```Shell
terraform plan
```

Se va a tardar un poco, pero nos tiene que devolver este resultado
![[Pasted image 20241003040806.png]]

Aplicamos los cambios 
``` Shell
terraform apply
```

---
