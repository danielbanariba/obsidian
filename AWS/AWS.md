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
Desplegar (este comando solo sirve en gitbash)
```shell
sls deploy --stage cooper --region us-east-1
```

Saber mi ID Account
```shell
aws sts get-caller-identity
```
Y me va a devolver un arreglo asi 
```json
{
    "UserId": "AIDA3L446LVM755R4HJ5C",
    "Account": "781476322649",
    "Arn": "arn:aws:iam::781476322649:user/luqa-dev-cooper"
}
```


IGNORAR CUALQUIER COMANDO QUE DIGA EN SU COMANDO 'delete' ES PELIGRO! ⚠️ ⚠️ ⚠️ ⚠️ ⚠️ ⚠️ ⚠️ ⚠️ ⚠️ ⚠️ ⚠️ 

---
## API GATEWAY