# ADR-0071. Una reserva cuyo turno ya empezó no se cancela

- **Estado:** Aceptado
- **Fecha:** 2026-11-03

## Contexto

[ADR-0026](0026-cancelacion-con-respuesta-rest.md) permite cancelar una reserva propia `CONFIRMED`, sin mirar la fecha del turno. Ni el enunciado ni la referencia de integración dicen qué pasa con un turno que ya pasó (REF §12 solo exige que esté confirmado). Cancelar un turno al que ya no se puede asistir no libera nada útil, y la app no debería ofrecerlo.

## Decisión

Si la fecha y la hora de inicio del turno ya pasaron, en la zona horaria del negocio ([ADR-0016](0016-zona-horaria-argentina.md)), la cancelación se rechaza con `409 RESERVATION_NOT_CANCELLABLE`, sin llamar a la cátedra. Es el mismo criterio de "turno que ya empezó" que se usa al reservar (`INVALID_SLOT`). Una reserva que ya está `CANCELLED` sigue respondiendo éxito sin cambios, aunque el turno haya pasado.

## Alternativas consideradas

- **Dejar que decida la cátedra.** Descartada: no está documentado que la cátedra lo rechace, y el resultado dependería de un comportamiento que no controlamos.

## Consecuencias

- Una regla local más en la cancelación del dueño, con su prueba. La cancelación administrativa sigue las mismas reglas de estado ([ADR-0037](0037-rol-administrador-alcance-cerrado.md)), así que también la aplica.
- La app puede no ofrecer cancelar un turno pasado; si igual lo pide, recibe el mismo código que para una reserva no confirmada.
