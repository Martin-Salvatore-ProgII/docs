# ADR-0028. Un timeout al crear el hold cierra el proceso como fallido, sin reintentar

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Un timeout no prueba que la operación haya fallado; antes de repetir una acción con efectos hay que reconciliar con los identificadores persistidos (REF §18.4). Si el pedido de hold termina en timeout, el proceso no tiene `holdId` ni `reservationProcessId`, y la cátedra no ofrece una consulta de holds: las ocupaciones no indican de quién es cada hold (REF §8). Un segundo pedido sobre el mismo slot respondería `SLOT_ALREADY_HELD` si el primero se creó, sin poder distinguir si ese hold es propio o ajeno.

## Decisión

No se reintenta. El proceso pasa a `FAILED` con el motivo `INTEGRATION_TIMEOUT` y el usuario ve que no se pudo reservar y puede intentar de nuevo.

## Alternativas consideradas

- **Reintentar el hold.** Descartada: no es seguro de repetir y su resultado no se puede interpretar.

## Consecuencias

- Si el hold se había creado, queda huérfano en la cátedra y vence solo en su `expiresAt`.
- Si el usuario reintenta el mismo slot antes de ese vencimiento, puede recibir `SLOT_ALREADY_HELD` por su propio hold huérfano. Es un caso raro que se corrige solo.
