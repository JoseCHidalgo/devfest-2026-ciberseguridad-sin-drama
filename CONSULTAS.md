# Consultas del taller

Cópialas tal cual en el **visor** ( https://lab.hidalgotechnologies.com ) o en el **Log Explorer de GCP**. La sintaxis es la misma del lenguaje de consulta de Cloud Logging.

> Las consultas de varias líneas son válidas: cada salto de línea equivale a `AND`.

## 1. Todo lo de Cloud Storage

Punto de partida: todos los eventos de buckets.

```
resource.type="gcs_bucket"
```

Ver qué hay antes de filtrar.

## 2. Cambios de permisos en buckets

Quién cambió permisos de un bucket (IAM).

```
protoPayload.methodName="storage.setIamPermissions"
```

Buscar exposición: es el evento clave del escenario.

## 3. Algo se hizo público (allUsers)

Cambios de permisos donde aparece `allUsers` (cualquier persona en internet).

```
protoPayload.methodName="storage.setIamPermissions"
protoPayload.serviceData.policyDelta.bindingDeltas.member="allUsers"
```

Incluye altas **y bajas** de allUsers.

## 4. Solo la ADICIÓN de allUsers (recomendada)

Igual que la anterior pero solo cuando se **agregó** el acceso público (`action="ADD"`).

```
protoPayload.methodName="storage.setIamPermissions"
protoPayload.serviceData.policyDelta.bindingDeltas.member="allUsers"
protoPayload.serviceData.policyDelta.bindingDeltas.action="ADD"
```

`bindingDeltas` también contiene los REMOVE; esta es la consulta que debes usar para detectar el momento en que se hizo público.

## 5. Público para allUsers o allAuthenticatedUsers

Ambos tipos de acceso público, con `OR` y paréntesis.

```
protoPayload.methodName="storage.setIamPermissions" AND (protoPayload.serviceData.policyDelta.bindingDeltas.member="allUsers" OR protoPayload.serviceData.policyDelta.bindingDeltas.member="allAuthenticatedUsers")
```

`allAuthenticatedUsers` es cualquiera con una cuenta Google.

## 6. Llaves de cuenta de servicio creadas

Creación de llaves JSON (credenciales de larga duración).

```
protoPayload.methodName="google.iam.admin.v1.CreateServiceAccountKey"
```

Una llave filtrada es un riesgo clásico.

## 7. Solo errores

Eventos con severidad ERROR o superior.

```
severity>=ERROR
```

Intentos denegados y fallos: buena pista de reconocimiento.

## 8. Eventos de una cuenta de servicio

Todo lo que hizo `app-pagos@…` (reto 4 del visor).

```
protoPayload.authenticationInfo.principalEmail="app-pagos@tiendapuravida-dev.iam.gserviceaccount.com"
```

Revisa IP de origen y user-agent: ¿es de la empresa?

## 9. Sin lecturas de objetos (NOT)

Eventos de buckets excluyendo `storage.objects.get`.

```
resource.type="gcs_bucket" AND NOT protoPayload.methodName="storage.objects.get"
```

Ejemplo de `AND NOT` para quitar ruido.

## 10. Regex sobre el método

`=~` con expresión regular.

```
protoPayload.methodName=~"^storage\.(set|get)IamPermissions$"
```

En GCP la regex es RE2; en el visor es JavaScript. Con patrones simples dan lo mismo.

## Lo que NO existe (y por qué)

- **Accesos anónimos a un bucket público:** GCP **no genera audit logs** cuando se lee un objeto público (`allUsers`). Por eso no hay una consulta para "quién descargó". Lección: si un bucket queda público por error, no podrás saber quién se llevó los datos.
