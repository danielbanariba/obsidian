---
tags:
  - Backend
  - Terraform
---
---
### Documentacion
https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs
----
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




---
### Configuración

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

Queremos hacer una subred que contenga una red para la base de datos y otra para la aplicación
![[Pasted image 20241005174208.png]]

Con este código ya tenemos configurado las dos subred

**network.tf** = 
``` java
# Define una red virtual en Azure
resource "azurerm_virtual_network" "vnet" {
    name                = "vnet-${var.project}-${var.environment}"# Nombre de la red virtual, utilizando variables para el proyecto y el entorno
    resource_group_name = azurerm_resource_group.rg.name
    location            = var.location
    address_space       = ["10.0.0.0/16"]
    
    tags = var.tags
}

# Define una subred dentro de la red virtual para la base de datos
resource "azurerm_subnet" "subnetdb" { # Subred para la base de datos
    name                 = "subnet-db-${var.project}-${var.environment}"
    resource_group_name  = azurerm_resource_group.rg.name
    virtual_network_name = azurerm_virtual_network.vnet.name
    address_prefixes     = ["10.0.1.0/24"]
}

# Define una subred dentro de la red virtual para la aplicación
resource "azurerm_subnet" "subnetapp" { # Subred para la aplicación
    name                 = "subnet-app-${var.project}-${var.environment}"
    resource_group_name  = azurerm_resource_group.rg.name
    virtual_network_name = azurerm_virtual_network.vnet.name
    address_prefixes     = ["10.0.2.0/24"]
}
```

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
---
### Configuración de la [[Bases de Datos]]

**Private Endpoint** = es como si fuera una interfaz de red, ya que necesitamos conectar la base de datos 

Otd.db = La base de datos del producto
 ![[Pasted image 20241005175249.png]]

**DNS Zone**= Ya que no sabemos que direccion va ser praa comunicarne, vamos a usar el DNS Zone, para traducir el 10.0.1.25 a **Private-link-db.Windows.databse.net** ![[Pasted image 20241005183037.png]]

**db.tf** = 


---
### Storage Account
**storage.tf** 