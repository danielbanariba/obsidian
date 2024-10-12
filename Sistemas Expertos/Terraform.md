---
tags:
  - Backend
  - Terraform
---
---
### Documentación
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


**DNS Zone**= Ya que no sabemos que dirección va a poder comunicarse, vamos a usar el DNS Zone, para traducir el 10.0.1.25 a **Private-link-db.Windows.databse.net** ![[Pasted image 20241005183037.png]]


**db.tf** = Vamos a configurar cada recurso de lo que tenemos planificado. 
Posdata: el codigo de abajo seria el cuadro azul
```java
resource "azurerm_mssql_server" "sql_server" {
    name                         = "sqlserver-${var.project}-${var.environment}"
    resource_group_name          = azurerm_resource_group.rg.name
    location                     = var.location
    version                      = "12.0"
    administrator_login          = "sqladmin"
    administrator_login_password = "ComplexP@ssw0rd123!"

    tags = var.tags
}
```


Configuramos la base de datos como tal
Posdata: Esto seria el cuadro naranja, otd.db, es el nombre de la base de datos
```java
resource "azurerm_mssql_database" "sql_db" {
    name                = "otd.db"
    server_id           = azurerm_mssql_server.sql_server.id
    sku_name            = "S0"
    
    tags = var.tags
}
```
![[Pasted image 20241006032545.png]]


Recordatorio, se conecta al servidor no a la base de datos
```java
resource "azurerm_private_endpoint" "sql_private_endpoint" {
    name                = "sql-private-endpoint-${var.project}-${var.environment}"
    resource_group_name = azurerm_resource_group.rg.name
    location            = var.location
    subnet_id           = azurerm_subnet.subnetdb.id
    private_service_connection {
        name                           = "sql-private-ec-${var.project}-${var.environment}"
        private_connection_resource_id = azurerm_mssql_server.sql_server.id
        subresource_names              = ["sqlServer"]
        is_manual_connection           = false
    }
    tags = var.tags
}
```
![[Pasted image 20241006040921.png]]


```java
//Define una zona DNS privada en Azure
resource "azurerm_private_dns_zone" "sql_private_dns_zone" {
    name                = "private.dbserver.database.windows.net"
    # Nombre del grupo de recursos donde se creará la zona DNS privada
    resource_group_name = azurerm_resource_group.rg.name
    tags = var.tags
}

//Define un registro A en la zona DNS privada
resource "azurerm_private_dns_a_record" "private_dns_a_record" {
    name                = "sqlserver-record-${var.project}-${var.environment}"
    zone_name           = azurerm_private_dns_zone.sql_private_dns_zone.name
    resource_group_name = azurerm_resource_group.rg.name
    ttl                 = 300
    records             = [azurerm_private_endpoint.sql_private_endpoint.private_service_connection[0].private_ip_address]
    tags = var.tags
}
```
![[Pasted image 20241006040217.png]]


```java
resource "azurerm_private_dns_zone_virtual_network_link" "vnet_link" {
    name                  = "vnet-link-${var.project}-${var.environment}"
    resource_group_name   = azurerm_resource_group.rg.name
    private_dns_zone_name = azurerm_private_dns_zone.sql_private_dns_zone.name
    virtual_network_id    = azurerm_virtual_network.vnet.id
    tags = var.tags
}
```
![[Pasted image 20241006040825.png]]



---
### Storage Account
![[Pasted image 20241006223510.png]]


El **Queue Storage** es para poder guardar las peticiones o mensajes cuando ocupemos hacer las calibraciones de nuestras bases de datos vectoriales
El **Blob Container** es para guardar archivos multimedias 

El archivo con el código completo lo puedes encontrar en **storage.tf** 
```java
resource "azurerm_storage_account" "storage_account" {

    name                     = "storage${var.project}${var.environment}3"
    resource_group_name      = azurerm_resource_group.rg.name
    location                 = var.location
    account_tier             = "Standard"
    account_replication_type = "LRS"

    tags = var.tags
}
```

```java
resource "azurerm_storage_container" "blog_container" {
    name                  = "blog"
    storage_account_name  = azurerm_storage_account.storage_account.name
    container_access_type = "private"
}
```

![[Pasted image 20241006232002.png]]


```java
resource "azurerm_private_endpoint" "blob_private_endpoint" {

    name                = "blob-private-endpoint-${var.project}-${var.environment}"
    resource_group_name = azurerm_resource_group.rg.name
    location            = var.location
    subnet_id           = azurerm_subnet.subnetapp.id

    private_service_connection {
        name                           = "storage-private-${var.project}-${var.environment}"
        private_connection_resource_id = azurerm_storage_account.storage_account.id
        subresource_names              = ["blob"]
        is_manual_connection           = false
    }

    tags = var.tags
}

resource "azurerm_private_endpoint" "queue_private_endpoint" {
    name                = "queue-private-endpoint-${var.project}-${var.environment}"
    resource_group_name = azurerm_resource_group.rg.name
    location            = var.location
    subnet_id           = azurerm_subnet.subnetapp.id

    private_service_connection {
        name                           = "storage-private-${var.project}-${var.environment}"
        private_connection_resource_id = azurerm_storage_account.storage_account.id
        subresource_names              = ["queue"]
        is_manual_connection           = false
    }

    tags = var.tags
}
```

![[Pasted image 20241007020033.png]]


```java
resource "azurerm_private_dns_zone" "sa_private_dns_zone" {
    name                = "privatelink.storage.core.windows.net"
    resource_group_name = azurerm_resource_group.rg.name

    tags = var.tags
}
```

![[Pasted image 20241007020348.png]]



---
### IO Bound & ACR

Outbound = entrada
inbound = Salida


![[Pasted image 20241007024651.png]]


![[Pasted image 20241007033735.png]]


![[Pasted image 20241007040653.png]]


![[Pasted image 20241007043208.png]]



---
### App Service plan WebApps FunctionApps

[[Docker]]