# ADR-0009. Las entidades referenciadas que faltan se traen del estado actual en Redis

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

Los profesionales referencian una categoría y los horarios semanales referencian un profesional (REF §7). En la sincronización incremental cada entidad se lee en su estado actual, no en el de la versión que se está aplicando (REF §14.4, §14.5). Por eso, al aplicar una versión, una entidad puede apuntar a otra que se creó en una versión posterior y que la copia local todavía no tiene.

## Decisión

La copia local mantiene la integridad referencial entre categorías, profesionales y horarios. Si al aplicar una versión una entidad referencia a otra que no está en la copia local, también se trae el estado actual de la referenciada desde su hash de Redis y se guarda en la misma unidad. Si la referenciada tampoco está en Redis, el estado no se puede verificar y se hace un snapshot.

## Alternativas consideradas

- **No exigir integridad referencial y tolerar referencias colgadas hasta que llegue la versión correspondiente.** Descartada: la copia puede quedar inconsistente entre versiones y las búsquedas podrían devolver profesionales con una categoría inexistente.
- **Aplicar todo el rango de versiones pendientes en una sola transacción.** Descartada: contradice el avance versión por versión (ENUNCIADO §6.2) y una falla en la última versión descartaría el progreso de todas las anteriores.

## Consecuencias

- La copia local siempre es referencialmente consistente.
- Es coherente con el modelo de la cátedra: los hashes contienen el estado actual de todas las entidades, así que la referenciada está disponible aunque su versión no se haya aplicado todavía.
- Cuando se aplica la versión en la que cambió la entidad referenciada, se vuelve a escribir su mismo estado actual, sin efecto.
