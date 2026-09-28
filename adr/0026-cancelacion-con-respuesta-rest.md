# ADR-0026. La cancelación se registra con la respuesta REST de la cátedra

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Cancelar una reserva es `POST /api/appointments/{reservationProcessId}/cancel`. La cátedra responde 200 con el estado `CANCELLED`, trata como idempotente la cancelación de algo ya cancelado y además publica `AppointmentCancelled` (REF §12, §15.10). Solo se pueden cancelar reservas confirmadas (`RESERVATION_NOT_CANCELLABLE`, REF §13.1).

## Decisión

- Solo el dueño puede cancelar, y solo si el proceso está `CONFIRMED`. Si ya está `CANCELLED`, la operación responde éxito sin llamar a la cátedra.
- El usuario puede indicar un motivo opcional, de hasta 500 caracteres, que se envía a la cátedra.
- Con el 200 de la cátedra el proceso pasa a `CANCELLED`. El evento `AppointmentCancelled` posterior no produce cambios.

## Alternativas consideradas

- **Marcar la cancelación recién cuando llega `AppointmentCancelled`.** Descartada: el usuario vería la reserva como confirmada durante unos segundos después de cancelarla, cuando el 200 ya es la confirmación autoritativa.

## Consecuencias

- La respuesta al usuario es inmediata y coherente con lo que registró la cátedra.
- El evento posterior es un duplicado funcional que la máquina de estados ignora.
- Si la cancelación termina en timeout, el resultado se reconcilia según la parte de robustez.
