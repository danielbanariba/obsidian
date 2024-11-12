---
tags:
  - AWS
  - Amazon
---
![[Pasted image 20241107154724.png]]
## [Documentacion](https://docs.aws.amazon.com/?nc2=h_ql_doc_do)



---
## Comandos

Aqui tenemos que ingresar las credenciales para poder desplegar aws
```shell
aws configure
```

 En mi caso como estoy en la empresa GuabaBIT, estos serian las credenciales
```shell
AKIA3L446LVM3DPNBSXM
```
```shell
YCPzaGAxFuZPpy3LX0DMqRUqiIzF/PYg2Uz6C869
```
```shell
us-east-1
```
Como en el ejemplo de abajo
![[Pasted image 20241107192220.png]]
recordar siempre desplegarlo con [[Serverless]]

Consultar las tablas de mi base de datos
```shell
aws dynamodb list-tables --region us-east-1
```



---
En el archivo main.tf y usando [[Terraform]] ponemos
```Java
provider "aws" {
	region = "us-east-1"
}
```

Desplegar (este comando solo sirve en gitbash)
```shell
sls deploy --stage cooper --region us-east-1
```



---
## API GATEWAY