# Contratos

APIs que exponen nuestros servicios: hacia la app KMP ([ADR-0001](../adr/0001-app-habla-con-ambos-servicios.md)) y de turnos hacia el catálogo ([ADR-0002](../adr/0002-comunicacion-unidireccional-turnos-catalogo.md)). Los contratos con la cátedra no se repiten acá: están en la REF.

La versión validable de cada contrato está en OpenAPI:

| Servicio | OpenAPI |
| --- | --- |
| Catálogo | [`contratos/catalogo-api.yaml`](contratos/catalogo-api.yaml) |
| Turnos | Pendiente: se agrega cuando se definan las operaciones de reserva |

- **Estado:** Aceptado

## Convenciones comunes

Se siguen las mismas convenciones que la cátedra (REF §4), para que un mismo cliente entienda los tres contratos:

- JSON con propiedades en `camelCase`.
- Fechas `yyyy-MM-dd`; horas `HH:mm:ss` en las respuestas, y se aceptan `HH:mm` y `HH:mm:ss` en las entradas; instantes ISO-8601 en UTC. Las horas de agenda son hora de Argentina ([ADR-0016](../adr/0016-zona-horaria-argentina.md)).
- Autenticación con `Authorization: Bearer <jwt-de-usuario>` en todo lo que no sea explícitamente público ([RNF-04](../requisitos/no-funcionales.md#rnf-04-endpoints-protegidos-por-defecto)).
- Listas paginadas con `page` (desde 0), `size` (por defecto 20, máximo 100) y `sort`; el total va en la cabecera `X-Total-Count` y la navegación en `Link`, como en REF §11.
- Errores en `application/problem+json` con un `code` funcional estable, como en REF §13. Los clientes deciden por `status` y `code`, nunca por el texto de `detail`. Los errores de validación incluyen `fieldErrors`.

### Códigos de error propios

| HTTP | `code` | Cuándo |
| --- | --- | --- |
| 400 | `VALIDATION_ERROR` | Falta un campo o tiene formato, longitud o rango inválido |
| 400 | `USERNAME_ALREADY_EXISTS` | Registro con un `login` ya usado |
| 400 | `EMAIL_ALREADY_EXISTS` | Registro con un `email` ya usado |
| 400 | `DATE_OUT_OF_RANGE` | Fecha de disponibilidad pasada o más allá del horizonte |
| 400 | `PROFESSIONAL_DISABLED` | Disponibilidad o reserva de un profesional no habilitado |
| 400 | `INVALID_SLOT` | El slot no pertenece a la agenda vigente del profesional para esa fecha |
| 400 | `INVALID_PHONE_NUMBER` | El teléfono no cumple el formato de REF §15.5 |
| 401 | (puede no traer `code`) | Sin JWT, JWT inválido o vencido, o credenciales incorrectas |
| 403 | (puede no traer `code`) | JWT válido sin permiso para la operación |
| 404 | `PROFESSIONAL_NOT_FOUND` | Profesional inexistente |
| 404 | `RESERVATION_NOT_FOUND` | Reserva inexistente o de otro usuario |
| 409 | `ACTIVE_RESERVATION_EXISTS` | El usuario ya tiene un proceso de reserva activo |
| 409 | `PHONE_NOT_REQUESTED` | Se envía un teléfono y el proceso no está esperándolo |
| 409 | `RESERVATION_NOT_CANCELLABLE` | Se cancela una reserva que no está confirmada ni cancelada |
| 503 | `CATALOG_NOT_READY` | El catálogo todavía no tiene una versión local |
| 503 | `CATALOG_UNAVAILABLE` | Turnos no pudo obtener datos del catálogo |
| 503 | `INTEGRATION_UNAVAILABLE` | Turnos no pudo obtener datos de la cátedra |

Los códigos que coinciden con los de la cátedra (`VALIDATION_ERROR`, `USERNAME_ALREADY_EXISTS`, `EMAIL_ALREADY_EXISTS`, `PROFESSIONAL_NOT_FOUND`, `PROFESSIONAL_DISABLED`, `INVALID_SLOT`, `RESERVATION_NOT_FOUND`, `RESERVATION_NOT_CANCELLABLE`) significan lo mismo que en REF §13.1.

Una reserva de otro usuario responde igual que una inexistente (`404 RESERVATION_NOT_FOUND`), para no revelar que existe.

## Servicio de catálogo

| Operación | Método y ruta | Consumidor | Requisito |
| --- | --- | --- | --- |
| Listar categorías | `GET /api/professional-categories` | App | HU-05 |
| Buscar profesionales | `GET /api/professionals` | App | HU-05, ADR-0013, ADR-0014 |
| Agenda vigente de un profesional | `GET /api/professionals/{id}/agenda` | App y turnos | ADR-0017 |
| Estado de sincronización | `GET /api/catalog/sync-status` | App | HU-04, ADR-0011 |

### Listar categorías

Devuelve las categorías habilitadas, ordenadas por nombre, para ofrecerlas como filtro. Respuesta: lista de `{ id, name, description }`.

### Buscar profesionales

Parámetros, todos opcionales y combinables:

| Parámetro | Tipo | Regla |
| --- | --- | --- |
| `categoryId` | entero | Categoría del profesional |
| `name` | texto, 1 a 100 | Coincidencia parcial sobre nombre, apellido o nombre completo, sin distinguir mayúsculas ni tildes |
| `enabled` | booleano, por defecto `true` | Estado habilitado según [ADR-0014](../adr/0014-habilitado-efectivo.md) |
| `date` | fecha | Filtro de disponibilidad de agenda ([ADR-0013](../adr/0013-filtro-disponibilidad-de-agenda.md)) |
| `timeFrom`, `timeTo` | hora | Franja horaria opcional; requieren `date` y `timeFrom` < `timeTo` |
| `page`, `size`, `sort` | | Paginación; orden por defecto `lastName,firstName` |

Respuesta: lista paginada de `{ id, firstName, lastName, category: { id, name }, enabled }`.

Errores: `400 VALIDATION_ERROR`, `401`, `503 CATALOG_NOT_READY`.

### Agenda vigente de un profesional

Respuesta, leída en una sola lectura consistente:

```json
{
  "id": 15,
  "firstName": "Ana",
  "lastName": "Gomez",
  "category": { "id": 1, "name": "Clinica medica" },
  "enabled": true,
  "weeklySchedules": [
    { "id": 21, "dayOfWeek": "MONDAY", "startTime": "09:00:00", "endTime": "12:00:00", "slotDurationMinutes": 30 }
  ],
  "catalogVersion": 7
}
```

- `enabled` es el habilitado efectivo (profesional y categoría).
- `weeklySchedules` incluye solo los horarios habilitados.
- Un profesional deshabilitado se devuelve igual, con `enabled: false`: decidir qué hacer es responsabilidad del consumidor.

Errores: `401`, `404 PROFESSIONAL_NOT_FOUND`, `503 CATALOG_NOT_READY`.

La autenticación de turnos ante esta operación se decide en la parte de seguridad.

### Estado de sincronización

```json
{
  "localVersion": 7,
  "syncInProgress": false,
  "lastSuccessAt": "2026-09-24T13:45:00Z",
  "lastSuccessType": "INCREMENTAL",
  "lastErrorAt": null,
  "lastError": null
}
```

`localVersion` es `null` si el catálogo nunca se sincronizó. `lastSuccessType` es `SNAPSHOT` o `INCREMENTAL`. `lastError` es un mensaje sin secretos ni detalles internos.

## Servicio de turnos

| Operación | Método y ruta | Consumidor | Requisito |
| --- | --- | --- | --- |
| Registro | `POST /api/register` (público) | App | HU-01 |
| Inicio de sesión | `POST /api/authenticate` (público) | App | HU-02 |
| Disponibilidad | `GET /api/availability` | App | HU-06, CU-03 |
| Iniciar una reserva | `POST /api/reservations` | App | HU-07, CU-04 |
| Ver una reserva (y seguir el proceso) | `GET /api/reservations/{id}` | App | HU-07, HU-08, ADR-0021 |
| Enviar el teléfono | `POST /api/reservations/{id}/phone` | App | HU-07, CU-04 |
| Listar mis reservas | `GET /api/reservations` | App | HU-08, ADR-0025 |
| Cancelar una reserva | `POST /api/reservations/{id}/cancel` | App | HU-09, ADR-0026 |

### Registro e inicio de sesión

Compatibles con JHipster (ENUNCIADO §3.2):

- **Registro:** recibe `login`, `password`, `firstName`, `lastName`, `email`, `imageUrl` (opcional) y `langKey`, con las reglas de [HU-01](../requisitos/HU.md#hu-01-registro-de-usuario-final). Responde `201` sin cuerpo. Errores: `400 VALIDATION_ERROR`, `400 USERNAME_ALREADY_EXISTS`, `400 EMAIL_ALREADY_EXISTS`.
- **Inicio de sesión:** recibe `{ "username", "password", "rememberMe" }` y responde `200` con `{ "id_token": "<jwt-de-usuario>" }`. Credenciales incorrectas: `401`.

### Disponibilidad

Parámetros obligatorios: `professionalId` (entero) y `date` (fecha).

```json
{
  "professionalId": 15,
  "professionalFirstName": "Ana",
  "professionalLastName": "Gomez",
  "date": "2026-10-05",
  "catalogVersion": 7,
  "slots": [
    { "startTime": "10:00:00", "endTime": "10:30:00" },
    { "startTime": "10:30:00", "endTime": "11:00:00" }
  ]
}
```

Una lista `slots` vacía es una respuesta válida.

Errores: `400 VALIDATION_ERROR`, `400 DATE_OUT_OF_RANGE`, `400 PROFESSIONAL_DISABLED`, `401`, `404 PROFESSIONAL_NOT_FOUND`, `503 CATALOG_UNAVAILABLE`, `503 INTEGRATION_UNAVAILABLE`.

### Reservas

Una reserva es un proceso de reserva local, con los estados de [`arq/maquina-estados.md`](maquina-estados.md). El `id` es el identificador local del proceso; el `reservationProcessId` de la cátedra no se expone a la app. Todas las operaciones actúan solo sobre reservas del usuario autenticado.

**Representación de una reserva**

```json
{
  "id": "5b1f0c2e-8d7a-4a51-9e0b-1f2d3c4b5a69",
  "status": "AWAITING_PHONE",
  "professionalId": 15,
  "professionalFirstName": "Ana",
  "professionalLastName": "Gomez",
  "date": "2026-10-05",
  "startTime": "10:00:00",
  "endTime": "10:30:00",
  "expiresAt": "2026-10-01T14:44:50Z",
  "message": "Ingrese un numero de telefono para continuar.",
  "failureReason": null,
  "createdAt": "2026-10-01T14:35:00Z",
  "confirmedAt": null,
  "cancelledAt": null,
  "cancellationReason": null
}
```

- `status`: uno de los estados de la máquina (`STARTED`, `HELD`, `AWAITING_REQUEST`, `AWAITING_PHONE`, `PHONE_SUBMITTED`, `CONFIRMED`, `CANCELLED`, `EXPIRED`, `INVALID`, `FAILED`).
- `expiresAt`: vencimiento informado por la cátedra; solo tiene sentido mientras el proceso no es final.
- `message`: último mensaje de la cátedra para el usuario (el del pedido o el del rechazo del teléfono).
- `failureReason`: motivo de `FAILED`, `EXPIRED` o `INVALID`, como código estable (por ejemplo, `SLOT_ALREADY_HELD`, `HOLD_EXPIRED`, `REQUEST_EVENT_MISMATCH`).

**Iniciar una reserva:** recibe `{ "professionalId", "date", "startTime" }`. Responde `201` con la reserva, ya en `AWAITING_REQUEST` o en un estado final si la cátedra la rechazó. Errores: `400 VALIDATION_ERROR`, `400 DATE_OUT_OF_RANGE`, `400 PROFESSIONAL_DISABLED`, `400 INVALID_SLOT`, `401`, `404 PROFESSIONAL_NOT_FOUND`, `409 ACTIVE_RESERVATION_EXISTS`, `503 CATALOG_UNAVAILABLE`, `503 INTEGRATION_UNAVAILABLE`.

**Ver una reserva:** responde `200` con la reserva. Es la operación que la app consulta periódicamente mientras el proceso no es final. Errores: `401`, `404 RESERVATION_NOT_FOUND`.

**Enviar el teléfono:** recibe `{ "phoneNumber" }`. Responde `202` con la reserva en `PHONE_SUBMITTED`. Errores: `400 INVALID_PHONE_NUMBER`, `401`, `404 RESERVATION_NOT_FOUND`, `409 PHONE_NOT_REQUESTED`, `503 INTEGRATION_UNAVAILABLE`.

**Listar mis reservas:** filtro opcional `status` y paginación; orden por defecto `createdAt` descendente. Responde la lista paginada. Errores: `400 VALIDATION_ERROR`, `401`.

**Cancelar una reserva:** cuerpo opcional `{ "reason" }` de hasta 500 caracteres. Responde `200` con la reserva en `CANCELLED`, también si ya estaba cancelada. Errores: `400 VALIDATION_ERROR`, `401`, `404 RESERVATION_NOT_FOUND`, `409 RESERVATION_NOT_CANCELLABLE`, `503 INTEGRATION_UNAVAILABLE`.
