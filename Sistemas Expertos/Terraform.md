---
tags:
  - Backend
  - Terraform
---
---
### Instalación
https://developer.hashicorp.com/terraform/install?product_intent=terraform#windows

Entrar sesión de [[Azure]]
```bash
az login
```

Seleccionamos la sección donde queremos comenzar
![[Pasted image 20241002215933.png]]

---
### Creación de grupo de recursos





---
### Configuración

main.tf
```java
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

**terraform.tfvars** es para guardar variables de entorno





En el control de versiones de [[Git]], en el archivo **.gitignore**, tenemos que ignorar estos archivos 
```git
*.tfstate
*.tfstate.*
*.tfvars
.terraform/
.terraform.*
```




---
### VNets & Subnets

**variables.tf**  Variables de entorno, donde se especifica las constantes y solo al poner . y ya tener el valor fijo de la variable, me recuerda bastante a los **enum** de [[Python]]
``` java
variable "project" {
    description = "the name project"
    default     = "otd"
}

variable "environment" {
    description = "The environment to release"
    default     = "dev"
}

variable "location" {
    description = "Azure region"
    default     = "East US 2"
}
```

![[Pasted image 20241005174208.png]]

---
### Configuración de la [[Bases de Datos]]

**Private Endpoint** = es como si fuera una interfaz de red, ya que necesitamos conectar la base de datos 
 ![[Pasted image 20241005175249.png]]
 DNS Zone= 
 ![[Pasted image 20241005183037.png]]








---
### Storage Account
**storage.tf** 