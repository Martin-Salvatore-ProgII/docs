# ADR-0019. Hold y confirmación inicial son una sola acción del usuario

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Reservar requiere crear un hold (REF §9) y después confirmarlo inicialmente (REF §10). La confirmación pide `externalPatientId`, `patientFirstName` y `patientLastName`, que el servicio de turnos ya tiene del usuario autenticado ([ADR-0004](0004-external-patient-id-uuid.md), [HU-01](../requisitos/HU.md#hu-01-registro-de-usuario-final)). El hold tiene una vigencia limitada que corre desde que se crea (`expiresAt`).

## Decisión

El usuario toca "Reservar" una sola vez y el servicio de turnos crea el hold y lo confirma inmediatamente, sin intervención del usuario entre los dos pasos.

El hold se toma **antes** de pedir cualquier dato al usuario y se mantiene durante **todo** el proceso, incluido el tiempo en que el usuario ingresa el teléfono. Mientras está vigente, nadie más puede tomar ese slot.

## Alternativas consideradas

- **Dos acciones del usuario (retener y después confirmar).** Descartada: la confirmación no necesita ningún dato que el usuario tenga que ingresar, así que la pausa solo consumiría tiempo del hold sin aportar nada. Separarlas tendría sentido únicamente si hiciera falta un dato del usuario en el medio, y aun así el hold tendría que tomarse antes, para que nadie más saque el turno mientras tanto.

## Consecuencias

- Para el usuario, reservar es una sola intención y una sola acción.
- El tiempo del hold se reserva para lo único que requiere al usuario: el teléfono ([ADR-0020](0020-telefono-a-pedido.md)).
- Si el usuario no completa a tiempo, el proceso vence y el slot se libera en la cátedra.
