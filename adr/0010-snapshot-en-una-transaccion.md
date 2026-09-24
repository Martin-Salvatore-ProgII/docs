# ADR-0010. El snapshot se aplica en una sola transacción y la app avisa que el catálogo se está actualizando

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

El snapshot tiene que aplicarse de forma consistente y registrar como versión local exactamente la que corresponde a los datos guardados (ENUNCIADO §6.1; REF §7). Mientras tanto, los usuarios siguen buscando profesionales. Además, un usuario puede saber que un profesional existe y no verlo porque la copia local todavía no incorporó ese cambio.

## Decisión

El reemplazo de las tres colecciones y de la versión local se hace en una sola transacción. Las búsquedas ven la copia anterior completa hasta que la transacción termina y, a partir de ahí, la nueva completa. Lo mismo vale para cada versión incremental.

Mientras hay una sincronización en curso, la app muestra un aviso no bloqueante de que el catálogo se está actualizando, y el usuario sigue pudiendo buscar sobre la copia vigente.

## Alternativas consideradas

- **Rechazar o bloquear las búsquedas durante la sincronización.** Descartada: deja al usuario sin servicio por una operación interna, cuando la copia anterior es válida hasta que termine la nueva.
- **Reemplazar las colecciones por partes, en transacciones separadas.** Descartada: las búsquedas podrían ver una mezcla de datos nuevos y viejos (por ejemplo, horarios de profesionales que todavía no están), lo que contradice el "de forma consistente" de ENUNCIADO §6.1.

## Consecuencias

- Nadie ve nunca una copia a medias y el servicio no deja de responder.
- Requiere un motor en el que las lecturas no se bloqueen por una escritura en curso ([ADR-0012](0012-postgresql.md)).
- El servicio tiene que informar que hay una sincronización en curso ([ADR-0011](0011-fallas-y-estado-de-sincronizacion.md)) para que la app pueda mostrar el aviso.
- El catálogo es chico, así que la transacción del snapshot es corta.
