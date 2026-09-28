# ADR-0031. Los eventos Kafka que no se pueden procesar no bloquean el consumo

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Si un evento no se puede procesar y no se confirma su offset, Kafka lo reentrega indefinidamente y bloquea los siguientes. Los consumidores deben ignorar campos desconocidos compatibles con la versión 1 (REF §4, §15.1) y validar los datos que reciben (ENUNCIADO §9).

## Decisión

- **Formato inválido o `eventType` desconocido:** se registra, se confirma el offset y se continúa.
- **Evento de un proceso que no existe localmente:** se registra y se ignora.
- **Falla transitoria al procesar (por ejemplo, la base no responde):** hasta 3 reintentos. Si se agotan, se registra, se confirma el offset y se continúa.

## Alternativas consideradas

- **No confirmar el offset hasta poder procesar.** Descartada: un solo mensaje problemático detiene todo el topic.
- **Enviar los mensajes fallidos a un topic de errores.** Descartada: la ACL de la cátedra no contempla topics propios y la reconciliación ya recupera el estado.

## Consecuencias

- Un mensaje problemático nunca detiene el consumo.
- Saltear un evento es seguro porque la reconciliación ([ADR-0030](0030-reconciliacion-periodica-de-procesos.md)) recupera el estado desde la cátedra, y en el catálogo el chequeo periódico hace lo mismo ([ADR-0006](0006-disparadores-sincronizacion.md)).
