# ADR-0025. Las reservas propias se consultan desde la base local de turnos

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

`GET /api/appointments` de la cátedra devuelve las reservas de toda la cuenta técnica (REF §11). El servicio de turnos ya conoce la propiedad de cada proceso ([ADR-0022](0022-proceso-guardado-antes-de-llamar.md)) y lo mantiene al día con los eventos. El enunciado pide que turnos siga atendiendo lo que depende solo de sus datos cuando algo externo no está disponible (ENUNCIADO §8), y permite guardar datos descriptivos históricos del profesional en una reserva (ENUNCIADO §4.2).

## Decisión

La lista y el detalle de las reservas del usuario se responden desde la base local de turnos, filtrando por el usuario autenticado. Cada proceso guarda, como dato histórico, el profesional (id, nombre y apellido), la fecha y los horarios del turno. La lista de la cátedra se usa para reconciliar estados, no para responder cada consulta.

## Alternativas consideradas

- **Consultar la cátedra en cada pedido y filtrar por `externalPatientId`.** Descartada: deja de funcionar si la cátedra no está disponible y hace depender la propiedad de un dato enviado a un tercero, en lugar de la asociación local.

## Consecuencias

- El usuario ve sus reservas aunque la cátedra o el catálogo no estén disponibles.
- Los datos del profesional en una reserva son históricos: si después cambia en el catálogo, la reserva muestra los del momento de reservar, y nunca se usan como catálogo vigente ([Constitución P-05](../constitucion.md)).
- La precisión del estado local depende de los eventos y de la reconciliación, que se define en la parte de robustez.
