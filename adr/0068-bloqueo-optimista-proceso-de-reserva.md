# ADR-0068. Cambios simultáneos sobre un proceso de reserva: bloqueo optimista con versión

- **Estado:** Aceptado
- **Fecha:** 2026-10-30

## Contexto

El estado de un proceso de reserva cambia por dos canales que pueden actuar al mismo tiempo: la respuesta REST de una operación (por ejemplo, el 202 de la confirmación inicial) y un evento Kafka del mismo proceso (por ejemplo, `AdditionalInformationRequested`, que la cátedra publica apenas acepta la confirmación, REF §10, §17). Cada uno lee el proceso, decide la transición con la [máquina de estados](../arq/maquina-estados.md) y guarda.

Si los dos parten de la misma lectura, el último en guardar pisa al otro: el 202 procesado con una copia en `HELD` dejaría en `AWAITING_REQUEST` un proceso que el evento ya había llevado a `AWAITING_PHONE`, y borraría el `requestEventId`. El proceso retrocedería, que es justo lo que prohíben [ADR-0023](0023-estados-solo-avanzan.md) y la [Constitución P-17](../constitucion.md). La regla de transiciones no alcanza sola: se aplica sobre la copia leída, que puede estar vieja.

## Decisión

Cada proceso de reserva tiene una **versión** que aumenta en cada cambio. Un cambio solo se guarda si la versión en la base sigue siendo la que se leyó; si otro cambio la aumentó en el medio, no se guarda nada y se detecta el conflicto.

Ante un conflicto, se vuelve a leer el proceso y se aplica de nuevo el mismo disparador sobre el estado actual. Si desde ese estado no hay transición, se ignora como cualquier disparador tardío (ADR-0023). La repetición es acotada.

## Alternativas consideradas

- **Bloquear la fila mientras se decide (bloqueo pesimista).** Descartada: la decisión de una operación REST depende de una llamada a la cátedra que puede tardar varios segundos, y no se puede mantener la fila bloqueada durante esa espera. Bloquear solo al guardar obliga igual a releer el estado, y agrega esperas al consumidor de Kafka sin dar más garantías.
- **Guardar con una condición sobre el estado anterior** (cambiar el estado solo si sigue siendo el esperado). Descartada: obliga a escribir una condición por transición y reparte la regla de qué estado avanza a cuál entre la máquina de estados y las consultas a la base.

## Consecuencias

- La regla de transiciones queda en un solo lugar y nunca se aplica sobre datos viejos, sin importar el orden de llegada entre REST y Kafka ([`maquina-estados.md`](../arq/maquina-estados.md), regla 4).
- Funciona aunque corran varias instancias del servicio: la garantía la da la base, no la memoria del proceso.
- No bloquea: si no hay choque, que es lo habitual, no cuesta nada más que una columna.
- Es la misma idea que protege la versión local del catálogo ([ADR-0007](0007-concurrencia-optimista-version-local.md)).
