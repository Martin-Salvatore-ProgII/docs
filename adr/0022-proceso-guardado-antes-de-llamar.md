# ADR-0022. El proceso se guarda antes de llamar a la cátedra y se actualiza antes de confirmar

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Apenas la cátedra acepta la confirmación inicial, publica `AdditionalInformationRequested` (REF §10, §17). Ese evento puede llegar al servicio de turnos antes de que termine de procesar la respuesta 202. Además, el servicio puede caerse en cualquier punto del flujo, y el enunciado pide recuperar procesos pendientes o interrumpidos (ENUNCIADO §4.2, §8).

## Decisión

- El proceso se guarda en estado `STARTED`, asociado al usuario, **antes** de pedir el hold.
- Cuando la cátedra crea el hold, se guardan `holdId`, `reservationProcessId` y `expiresAt`, y el proceso pasa a `HELD` **antes** de pedir la confirmación inicial.

## Alternativas consideradas

- **Guardar el proceso recién cuando termina la confirmación inicial.** Descartada: el pedido de teléfono podría llegar para un `reservationProcessId` desconocido, y una caída entre el hold y la confirmación dejaría un hold sin rastro local.

## Consecuencias

- Cuando llega cualquier evento del proceso, este ya existe localmente, sin importar el orden entre REST y Kafka ([`arq/maquina-estados.md`](../arq/maquina-estados.md), regla 4).
- Cada intento de reserva queda registrado aunque falle, lo que es la base de la recuperación después de un reinicio.
