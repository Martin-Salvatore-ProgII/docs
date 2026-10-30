# ADR-0070. Las reglas que tienen que valer ante pedidos simultáneos las garantiza la base de datos

- **Estado:** Aceptado
- **Fecha:** 2026-10-30

## Contexto

Varias reglas del sistema dicen "como máximo uno": un usuario con un solo proceso de reserva activo ([ADR-0024](0024-un-proceso-activo-por-usuario.md)), un `login` y un `email` por usuario ([HU-01](../requisitos/HU.md#hu-01-registro-de-usuario-final)), un evento Kafka procesado una sola vez por su `eventId` ([ADR-0008](0008-deduplicacion-por-event-id.md)). La forma obvia de cumplirlas es consultar antes de escribir ("¿ya existe?") y escribir solo si la respuesta es no.

Eso falla con pedidos simultáneos. Con un doble toque en "Reservar", los dos pedidos consultan antes de que cualquiera guarde, los dos reciben "no hay proceso activo" y los dos guardan. Entre la consulta y la escritura siempre puede entrar otro pedido, y con varias instancias del servicio ni siquiera comparten memoria.

## Decisión

Toda regla que tiene que valer aunque lleguen pedidos al mismo tiempo se garantiza **con una restricción de la base de datos**, en la misma escritura: una restricción `UNIQUE`, un índice único (parcial, si la regla vale solo para algunas filas), un `CHECK` o una condición sobre la versión leída ([ADR-0007](0007-concurrencia-optimista-version-local.md), [ADR-0068](0068-bloqueo-optimista-proceso-de-reserva.md)).

- Si la base rechaza la escritura, el servicio lo traduce al error funcional de la regla (por ejemplo, `409 ACTIVE_RESERVATION_EXISTS`), igual que si lo hubiera detectado antes.
- Una consulta previa se puede sumar como atajo, para no hacer trabajo de más en el caso normal (por ejemplo, no consultar al catálogo si el usuario ya tiene una reserva en curso), pero nunca es la garantía.

Aplicación en el servicio de turnos: un usuario tiene como máximo un proceso de reserva activo porque la base no admite dos procesos del mismo usuario en un estado no final (un índice único parcial sobre el usuario, limitado a esos estados).

## Alternativas consideradas

- **Solo la consulta previa.** Descartada: es la que falla con dos pedidos simultáneos.
- **Un bloqueo en memoria** (por ejemplo, `synchronized`). Descartada: solo protege dentro de una instancia del servicio; con dos instancias, cada una tiene el suyo.
- **Bloquear filas en la base mientras se decide** (`SELECT ... FOR UPDATE`). Descartada como regla general: obliga a elegir qué fila bloquear (en la reserva sería la del usuario, una tabla de otra feature), y el bloqueo dura todo lo que tarde la decisión, incluidas llamadas a otros servicios.

## Consecuencias

- La regla vale sin importar cuántos pedidos lleguen juntos ni cuántas instancias del servicio corran, porque la garantía está en un solo lugar compartido: la base.
- Cada regla de este tipo deja rastro en una migración, con el nombre de su restricción, y se prueba escribiendo dos veces lo mismo contra PostgreSQL real.
- El código tiene que reconocer el rechazo de esa restricción en particular y traducirlo; cualquier otro error de la base sigue siendo un error interno.
- Vale para los dos servicios: el catálogo lo aplica en su registro de eventos procesados.
