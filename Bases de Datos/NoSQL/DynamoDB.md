---
tags:
  - NoSQL
  - DynamoDB
---
---
DynamoDB es una base de datos [[NoSQL]] y creo que es muy importante en el mundo de [[AWS]]



---
Correr de forma local y no desperdicia recursos
tenemos que utilizar [[Docker]] y ejecutar el siguiente comando
1) Esto va a descargar el contenedor de DynamoDB
```
docker run -p 8000:8000 amazon/dynamodb-local
```

2) para que los endpoints funcionen correctamente desde local tenemos que poner este codigo, en mi casi es en el **service** que lo tengo que colocar
```js
dynamoose.aws.ddb.local('http://localhost:8000');
```

3) Tenemos que comentar todos los que nos generen conflictos de seguridad, ya que como lo estamos corriendo de forma local, no hay ningun problema, en mi caso como estoy programando con [[Nestjs]], necesito comentar todos los que tienen estas palabras
```js
UseGuards,
SetMetadata,
import { PermissionsGuard } from '../../guards/permissions.guard';
@UseGuards(PermissionsGuard)
@SetMetadata('permissions', ['AnnouncementRead'])
{ provide: APP_GUARD, useClass: PermissionsGuard }
import { PermissionsGuard } from '../../guards/permissions.guard';
import { APP_GUARD } from '@nestjs/core';
@Inject(REQUEST) private readonly request: OurHouseRequest,

```

4) y para tener aun menos conflictos, hacemos una exclusion del router, esto se encuentran en el **app.module.ts** y tenemos que agregar nuestro enpoint aqui 
```js
const excludedRoutes = [
	// announcements
	{ path: 'tenant/:tenantId/announcement', method: RequestMethod.GET },
	{ path: 'tenant/:tenantId/announcement', method: RequestMethod.POST },
];
```

5) Crear tablas 
```shell
aws dynamodb create-table \
    --table-name cooper-Announcement \
    --attribute-definitions \
        AttributeName=TenantId,AttributeType=S \
        AttributeName=AnnouncementId,AttributeType=S \
    --key-schema \
        AttributeName=TenantId,KeyType=HASH \
        AttributeName=AnnouncementId,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
    --endpoint-url http://localhost:8000
```