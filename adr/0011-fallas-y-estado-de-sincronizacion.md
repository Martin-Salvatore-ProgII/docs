# ADR-0011. Fallas de sincronización sin reintentos en loop y estado expuesto por el servicio

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

El servicio de catálogo tiene que informar el estado y los errores de sincronización (ENUNCIADO §4.1) y evitar ciclos de reintento ilimitados y estados imposibles de recuperar (ENUNCIADO §8). Durante un ciclo pueden fallar Redis, la API de la cátedra o la base. Si ante una falla no se confirmara el offset del aviso de Kafka, el mismo aviso se reprocesaría una y otra vez mientras dure la falla.

## Decisión

**Ante una falla:** la versión local queda en la última aplicada por completo, el error se registra en el estado de sincronización y en los logs, el offset del aviso se confirma igual y no se reintenta en el momento. El próximo disparador (el chequeo periódico, [ADR-0006](0006-disparadores-sincronizacion.md)) vuelve a intentar.

**Estado expuesto:** el servicio ofrece a usuarios autenticados su estado de sincronización: versión local, si hay una sincronización en curso, fecha y tipo de la última sincronización exitosa, y último error con su fecha.

## Alternativas consideradas

- **Reintentar inmediatamente con backoff hasta que funcione.** Descartada: es un loop de reintentos cuyo límite depende de cuánto dure la falla externa, y el chequeo periódico ya garantiza el reintento.
- **No confirmar el offset para que Kafka reentregue el aviso.** Descartada: produce reprocesamiento continuo mientras dure la falla, y el aviso no tiene información que se pierda (la decisión siempre se toma con Redis).
- **Guardar el estado en el namespace privado de Redis (`alumnos:{groupId}:*`).** Descartada: es opcional (REF §14.6) y duplicaría lo que ya está en la base del servicio.

## Consecuencias

- Una falla nunca deja la copia a medias ni genera tráfico continuo contra la cátedra.
- La recuperación demora, como máximo, un intervalo del chequeo periódico.
- El estado sirve para el aviso de "catálogo actualizándose" en la app ([ADR-0010](0010-snapshot-en-una-transaccion.md)) y para mostrar las transiciones de versión en la demo.
- El contrato de la operación de estado se define en [`arq/contratos.md`](../arq/contratos.md).
