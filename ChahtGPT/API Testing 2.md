# API Testing Prompt Template

## 1. Información del Endpoint

### Endpoint
```
URL: {{apiURL}}/tenant/inquire/{id}
Método: GET
```
## 2. Criterios de Aceptación y Descripción

### Acceptance Criteria
Send a GET request with valid tenant ID. Expect 200 status and correct data structure. Test with invalid ID. Expect 404 status.

## 3. Estructura de Datos

### Ejemplo de Respuesta Exitosa
```json
{
    "result": {
        "TimeOffset": -6,
        "UpdatedAt": 1713385370568,
        "CommonAreas": [
            {
                "Id": "area-1",
                "Description": "Área de piscina con jacuzzi",
                "Icon": "pool",
                "Name": "Piscina"
            }
        ],
        "RegularVisitHours": 48,
        "IdentificationDigits": 13,
        "PhoneNumber": "+50494549899",
        "PartyInvitesMinimum": 10,
        "TenantId": "C4PER",
        "AreaReservationSettings": {
            "CalendarDaysAhead": 30,
            "MonthlyLimit": 2,
            "HasMonthlyLimit": true,
            "PaymentRequired": false
        },
        "Name": "Residencial Las Palmeras"
    },
    "error": null,
    "exception": null
}
```

### Ejemplos de Respuestas de Error
```json
{
    "result": null,
    "error": null,
    "exception": null
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