# ADR-0014. Un profesional está habilitado solo si él y su categoría lo están; la búsqueda muestra habilitados por defecto

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

El catálogo marca como habilitados o deshabilitados, por separado, a las categorías, los profesionales y los horarios (REF §7). La búsqueda debe ofrecer un filtro por estado habilitado utilizable desde la app (ENUNCIADO §4.1). La matriz de errores de la cátedra rechaza holds de profesionales deshabilitados (`PROFESSIONAL_DISABLED`), pero no dice qué pasa con una categoría deshabilitada (REF §13.1).

## Decisión

- Un profesional está **habilitado** solo si su propio `enabled` y el de su categoría son verdaderos. Este criterio vale para la búsqueda, para la agenda vigente y para la disponibilidad.
- La búsqueda devuelve solo profesionales habilitados salvo que el usuario pida explícitamente los deshabilitados, que se muestran como no disponibles para reservar.
- Un horario semanal deshabilitado nunca genera disponibilidad.

## Alternativas consideradas

- **Considerar solo el `enabled` del profesional.** Descartada: ofrecería como reservables profesionales de una especialidad que la cátedra dio de baja, sin saber si la cátedra aceptaría el hold.
- **No mostrar nunca los deshabilitados.** Descartada: el filtro por estado habilitado tiene que poder usarse desde la app (ENUNCIADO §4.1), y el caso real existe ("¿mi médico dejó de atender?").

## Consecuencias

- En el peor caso se oculta algo que la cátedra permitiría reservar, en lugar de ofrecer algo que después falla.
- El criterio de "habilitado" está en un solo servicio (el catálogo) y turnos lo recibe ya resuelto ([ADR-0017](0017-operacion-agenda-vigente.md)).
