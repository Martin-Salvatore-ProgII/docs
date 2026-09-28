# ADR-0030. Reconciliación periódica de los procesos de reserva colgados

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Un evento Kafka puede perderse, el servicio puede reiniciarse con procesos a medias y un reintento puede agotarse. En esos casos, un proceso quedaría esperando indefinidamente (ENUNCIADO §8). La cátedra sugiere reconciliar mediante REST (REF §15.9, §18.3), y `GET /api/appointments` informa cada reserva con su `reservationProcessId` y su estado (REF §11). Sin teléfono enviado, la cátedra no puede confirmar un proceso.

## Decisión

Una tarea periódica (intervalo configurable, valor inicial: 60 segundos), que también corre al arrancar el servicio, revisa los procesos no finales cuyo `expiresAt` pasó hace más de un margen configurable (valor inicial: 30 segundos), o que quedaron en `STARTED`:

| Estado local | Acción |
| --- | --- |
| `STARTED` | Pasa a `FAILED`: el hold nunca quedó registrado y, si existió, ya venció |
| `HELD`, `AWAITING_REQUEST`, `AWAITING_PHONE` | Pasa a `EXPIRED` sin consultar: sin teléfono no pudo confirmarse |
| `PHONE_SUBMITTED` | Se consulta la cátedra por el `reservationProcessId`: `CONFIRMED` → `CONFIRMED`; `FAILED` → `EXPIRED`; `PHONE_PENDING` o ausente → se revisa en el próximo ciclo |

En cada ciclo también se comparan con la cátedra los procesos `CONFIRMED` con turno futuro, para detectar un `AppointmentCancelled` perdido.

Si la cátedra no responde, el proceso queda como está y se revisa en el próximo ciclo.

## Alternativas consideradas

- **Cerrar como vencido todo proceso pasado su `expiresAt`, sin consultar.** Descartada: un proceso con el teléfono enviado pudo confirmarse justo antes del vencimiento con el evento perdido, y cerrarlo mal sería irreversible ([Constitución P-17](../constitucion.md)).
- **Consultar la cátedra por todos los procesos no finales.** Descartada: genera tráfico innecesario para casos que se pueden decidir localmente.

## Consecuencias

- Cubre eventos perdidos, vencimientos y reinicios con operaciones pendientes con un solo mecanismo.
- Solo se consulta a la cátedra cuando hay una duda real.
- Cada ciclo es un intento y el siguiente es el reintento: no hay reintentos ilimitados.
- La consulta a la cátedra filtra por el `externalPatientId` del dueño del proceso.
