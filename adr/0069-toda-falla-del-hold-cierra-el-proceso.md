# ADR-0069. Toda falla al crear el hold cierra el proceso como fallido, y la app lo recibe como un rechazo

- **Estado:** Aceptado
- **Fecha:** 2026-10-30

## Contexto

[ADR-0028](0028-timeout-al-crear-hold.md) decide qué pasa si el pedido del hold termina en timeout, pero el pedido puede fallar de otras formas: la cátedra no acepta la conexión, responde con un error de su lado (5xx) o con un 4xx que no habla del turno elegido (un `VALIDATION_ERROR` por un pedido mal armado, o el JWT técnico rechazado). Cuando eso pasa, el proceso ya está guardado en `STARTED` ([ADR-0022](0022-proceso-guardado-antes-de-llamar.md)) y el usuario tiene que saber que no se pudo reservar, sin quedar bloqueado por un proceso activo ([ADR-0024](0024-un-proceso-activo-por-usuario.md)).

## Decisión

- **Cualquier falla del pedido del hold cierra el proceso en `FAILED` y no se reintenta**, igual que el timeout. El motivo es `INTEGRATION_TIMEOUT` si el pedido salió y no hubo respuesta a tiempo (el único caso en que puede quedar un hold huérfano), e `INTEGRATION_UNAVAILABLE` en cualquier otro caso. Los rechazos que hablan del turno (`SLOT_ALREADY_HELD`, `SLOT_ALREADY_RESERVED`, `INVALID_SLOT`, `PROFESSIONAL_DISABLED`, `PROFESSIONAL_NOT_FOUND`) conservan el `code` de la cátedra como motivo.
- **Una vez guardado el proceso, iniciar una reserva responde siempre `201` con el proceso**, en el estado en que quedó. Un `FAILED` por la cátedra caída llega igual que un `FAILED` por un turno ya tomado: con su `failureReason`.
- **El motivo es un código estable, nunca un texto técnico.** La app muestra un mensaje propio para cada motivo ([`arq/interfaz.md`](../arq/interfaz.md)); los de integración se muestran como que no se pudo reservar en ese momento y que se puede volver a intentar.

## Alternativas consideradas

- **Reintentar cuando la conexión fue rechazada**, que es el único caso en que se sabe que el pedido no llegó. Descartada: agrega un camino más para un caso raro, y el usuario puede volver a tocar "Reservar" por el camino normal.
- **Responder `503 INTEGRATION_UNAVAILABLE` cuando la cátedra no responde.** Descartada: el proceso ya existe y va a aparecer en "Mis reservas", pero la app no recibiría su id. Además la app tendría dos caminos para el mismo resultado (no se pudo reservar).

## Consecuencias

- El usuario nunca queda bloqueado por un proceso que no avanzó: queda en `FAILED` en el momento y puede reservar de nuevo.
- La app sigue un solo camino: siempre recibe el proceso y decide qué mostrar por `status` y `failureReason`.
- Antes de guardar el proceso no se llama a la cátedra, así que iniciar una reserva ya no responde `503 INTEGRATION_UNAVAILABLE`. Sí puede responder `503 CATALOG_UNAVAILABLE`: sin el catálogo no se guarda nada (CU-04, 3a).
