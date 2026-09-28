# ADR-0029. Reintentos acotados solo en las operaciones seguras de repetir, con timeouts configurables

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

El enunciado pide definir timeouts y reintentos, evitando ciclos ilimitados (ENUNCIADO §8). Los reintentos deben ser limitados y observables, y una operación con efectos solo se repite si es seguro hacerlo (REF §18.4). La cátedra da garantías distintas para cada operación: la confirmación de un hold ya confirmado responde `RESERVATION_PROCESS_ALREADY_CONFIRMED`, la cancelación de algo ya cancelado es idempotente y los eventos se deduplican por `eventId` (REF §10, §12, §13.1, §15.1).

## Decisión

| Operación | Reintentos | Regla |
| --- | --- | --- |
| Crear hold | 0 | [ADR-0028](0028-timeout-al-crear-hold.md) |
| Confirmación inicial | Hasta 2, con espera creciente | `RESERVATION_PROCESS_ALREADY_CONFIRMED` se trata como éxito |
| Cancelación | Hasta 2, con espera creciente | La cátedra es idempotente con lo ya cancelado |
| Publicar el teléfono | Hasta 2, con espera creciente | Se republica con el **mismo** `eventId`; el proceso pasa a `PHONE_SUBMITTED` solo cuando el broker confirma la recepción |
| Lecturas (ocupaciones, agenda del catálogo) | 1 | Si falla, se responde 503 |

Los timeouts de conexión y de respuesta son configurables (valores iniciales: 2 y 5 segundos). Cada reintento queda en el log con el identificador del proceso. Si se agotan los reintentos, el proceso queda en su estado actual y la reconciliación lo resuelve ([ADR-0030](0030-reconciliacion-periodica-de-procesos.md)).

## Alternativas consideradas

- **Reintentar todo de la misma forma.** Descartada: repetir una operación no idempotente puede duplicar efectos o dar resultados que no se pueden interpretar.
- **Sin reintentos.** Descartada: un corte breve haría fallar operaciones que se pueden repetir sin riesgo.

## Consecuencias

- Los cortes breves se absorben sin que el usuario espere más de unos segundos.
- Ningún reintento es ilimitado ni silencioso.
