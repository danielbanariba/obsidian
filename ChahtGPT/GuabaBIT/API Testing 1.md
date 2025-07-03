 API Testing Prompt Template

## 1. Información del Endpoint

### Endpoint
```
URL: {{apiURL}}/visit/batch-recheck/
Método: POST
```
## 2. Criterios de Aceptación y Descripción

### Acceptance Criteria
Send POST request with valid batch data. Expect 201 status. Test with invalid data. Expect 400 status.

### Descripción Funcional
Create Postman test cases for the /visit/batch-recheck endpoint
```json
{
  "result": [
    {
      "Id": "string",
      "TenantId": "string",
      "Type": "OneTime",
      "HasExpired": true,
      "ExpiresAt": 0,
      "History": [
        {
          "Type": "In",
          "CreatedAt": 0
        }
      ],
      "Frequencies": [
        0
      ]
    }
  ],
  "error": "string",
  "exception": "string"
}
```

## 3. Estructura de Datos

### Ejemplo de Respuesta Exitosa
```json
{
    "result": [
        {
            "TenantId": "C4PER",
            "History": [
                {
                    "Type": "In",
                    "CreatedAt": 1733353090000
                }
            ],
            "ExpiresAt": 1733439490000,
            "Id": "visit-001",
            "Type": "OneTime",
            "HasExpired": true
        },
        {
            "TenantId": "C4PER",
            "Frequencies": [
                1,
                3,
                5
            ],
            "Id": "visit-005",
            "Type": "Resident",
            "HasExpired": true
        }
    ],
    "error": null,
    "exception": null
}
```

### Ejemplos de Respuestas de Error
```json
{
    "result": null,
    "error": null,
    "exception": {
        "code": "ValidationException",
        "message": "1 validation error detected: Value '[]' at 'requestItems.cooper-Visit.member.keys' failed to satisfy constraint: Member must have length greater than or equal to 1",
        "stack": "ValidationException: 1 validation error detected: Value '[]' at 'requestItems.cooper-Visit.member.keys' failed to satisfy constraint: Member must have length greater than or equal to 1\n    at /var/task/src/lambda.js:2:401507\n    at /var/task/src/lambda.js:2:401602\n    at Dr (/var/task/src/lambda.js:2:300347)\n    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)\n    at async /var/task/src/lambda.js:2:376075\n    at async /var/task/src/lambda.js:2:812756\n    at async /var/task/src/lambda.js:2:375181\n    at async /var/task/src/lambda.js:2:359136\n    at async t.default (/var/task/src/lambda.js:2:1607844)\n    at async /var/task/src/lambda.js:2:1557120"
    }
}
```
---

## Ejemplo Completo (basado en /visit/create)

### Endpoint
```
URL: {{apiURL}}/visit/create
Método: POST
```

### Acceptance Criteria
1. Send POST request with valid visit data. Expect 201 status.
2. Test with missing fields. Expect 400 status.

### Descripción Funcional
Endpoint para crear nuevas visitas en el sistema. Permite registrar diferentes tipos de visitas (Delivery, Party, OneTime, Recurring) y maneja la inicialización automática de campos opcionales como Frequencies.

### Request Body
```json
{
    "Name": "Test Visit",
    "Type": "Delivery",
    "Frequencies": []
}
```

### Respuesta Exitosa
```json
{
    "result": {
        "Name": "Test Visit",
        "Type": "Delivery",
        "Frequencies": [],
        "Id": "12a36c33-d8e2-461a-8d8b-86410f7da87b",
        "TenantId": "C4PER",
        "IdUser": "34987428-a051-703a-c80b-5e44d09c1a9d",
        "ExpiresAt": 1736968732591,
        "CreatedAt": 1736795932593,
        "UpdatedAt": 1736795932593,
        "History": [],
        "Participants": []
    },
    "error": null,
    "exception": null
}
```

### Test de Ejemplo
```javascript
pm.test("Successful POST request with valid visit data", function () {
    pm.response.to.have.status(201);
    
    const response = pm.response.json();
    pm.expect(response).to.have.property("result");
    pm.expect(response.error).to.be.null;
    pm.expect(response.exception).to.be.null;
    
    const visit = response.result;
    pm.expect(visit).to.include.all.keys(
        "Id",
        "Name",
        "Type",
        "CreatedAt",
        "UpdatedAt",
        "ExpiresAt",
        "Frequencies"
    );
});
```


POSDATA! el 201 Successful ya lo tengo, asi es como se mira:
```
pm.test("201 - Batch Recheck Visit - Successful", function () {

    pm.response.to.have.status(201);

    const response = pm.response.json();

    pm.expect(response.result).to.be.an('array');

    pm.expect(response.error).to.be.null;

    pm.expect(response.exception).to.be.null;

  

    response.result.forEach(visit => {

        pm.expect(visit).to.include.all.keys(

            "Id",

            "TenantId",

            "Type",

            "HasExpired"

        );

  

        // Validate Type enum value

        pm.expect([

            "OneTime",

            "Delivery",

            "Frequent",

            "Party",

            "Resident"

        ]).to.include(visit.Type);

  

        // Validate History if present

        if (visit.History) {

            pm.expect(visit.History).to.be.an('array');

            visit.History.forEach(history => {

                pm.expect(history).to.have.all.keys(

                    "Type",

                    "CreatedAt"

                );

                pm.expect(["In", "Out"]).to.include(history.Type);

                pm.expect(history.CreatedAt).to.be.a('number');

            });

        }

  

        // Validate Frequencies if present

        if (visit.Frequencies) {

            pm.expect(visit.Frequencies).to.be.an('array');

            visit.Frequencies.forEach(frequency => {

                pm.expect(frequency).to.be.a('number');

            });

        }

    });

});
```

donde tengo dudas es en mis tres test 400: que son Missing ids, Empty ids Array y Invalid if formt, ya que todos tienen la misma estructura de test que es 
```
pm.test("400 - Batch Recheck Visit - Missing Ids", function () {

    pm.response.to.have.status(400);

    const response = pm.response.json();

    pm.expect(response.result).to.be.null;

    pm.expect(response.error).to.be.null;

    pm.expect(response.exception).to.exist;

});
```

Y no se si esta bueno que todos esos test se repitan y solo se cambie el mensaje que seria esto: 
```
pm.test("400 - Batch Recheck Visit - Missing Ids", function () {

    pm.response.to.have.status(400);

    const response = pm.response.json();

    pm.expect(response.result).to.be.null;

    pm.expect(response.error).to.be.null;

    pm.expect(response.exception).to.exist;

});
```

```
pm.test("400 - Batch Recheck Visit - Empty Ids Array", function () {

    pm.response.to.have.status(400);

    const response = pm.response.json();

    pm.expect(response.result).to.be.null;

    pm.expect(response.error).to.be.null;

    pm.expect(response.exception).to.exist;

});
```

```
pm.test("400 - Batch Recheck Visit - Invalid Id Format", function () {

    pm.response.to.have.status(400);

    const response = pm.response.json();

    pm.expect(response.result).to.be.null;

    pm.expect(response.error).to.be.null;

    pm.expect(response.exception).to.exist;

});
```