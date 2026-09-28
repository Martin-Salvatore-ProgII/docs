# ADR-0020. El teléfono se pide cuando llega el pedido de la cátedra

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Después de la confirmación inicial, la cátedra pide el teléfono con `AdditionalInformationRequested`, que incluye un `message` para el usuario y el `expiresAt` del proceso (REF §15.4). El enunciado exige demostrar una reserva que expira sin completar la información solicitada (ENUNCIADO §11).

## Decisión

La app pide el teléfono cuando el proceso pasa a `AWAITING_PHONE`, es decir, cuando llega el pedido. Muestra el mensaje de la cátedra y el tiempo que queda. Ante un rechazo, lo vuelve a pedir con el mensaje del rechazo.

## Alternativas consideradas

- **Pedir el teléfono al principio, junto con "Reservar", y enviarlo automáticamente al llegar el pedido.** Descartada: el caso de la reserva que expira sin completar la información no se podría mostrar de forma natural, se ignoraría el mensaje pensado para el usuario y el rechazo obligaría igual a volver a pedirlo.

## Consecuencias

- El flujo en la app sigue el diseño de la cátedra y se puede defender paso a paso.
- La evidencia de vencimiento se demuestra no ingresando el teléfono.
- La app necesita enterarse del pedido ([ADR-0021](0021-app-consulta-estado-por-polling.md)).
