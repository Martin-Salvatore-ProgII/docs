# ADR-0063. Lo que se puede reutilizar se lleva a `shared` en cuanto aparece el segundo uso

- **Estado:** Aceptado
- **Fecha:** 2026-10-16

## Contexto

La lectura a la cátedra con un reintento ([ADR-0029](0029-reintentos-acotados-por-operacion.md)) y con mensajes de error que no muestran la dirección de la cátedra se escribió primero para pedir la integración al arrancar. El cliente del snapshot necesitaba exactamente lo mismo, y turnos va a necesitarlo para ocupaciones, holds y reservas. Copiar el bloque deja dos versiones de una regla que tiene que ser idéntica, y cada copia es un lugar más que revisar, probar y explicar.

## Decisión

- Cuando al escribir código nuevo aparece algo que ya existe en una feature, o en una configuración puntual, y que se podría reutilizar, **se reformula el código existente y se lleva a `shared`** en esa misma issue, en lugar de copiarlo.
- La pieza de `shared` se prueba una vez, con todos sus casos. Las pruebas de quien la usa cubren solo lo propio.
- El README de cada servicio mantiene la tabla de las piezas de `shared` y quién las usa.
- Sigue valiendo el criterio de la skill `/hexagonal` para decidir qué es de `shared` y qué es de una feature: se lleva a `shared` lo transversal (infraestructura que usan varias features), no las reglas de negocio de una feature.

## Alternativas consideradas

- **Copiar y unificar más adelante.** Descartada: las copias se separan con el tiempo, y unificar después cuesta más que hacerlo con el segundo uso.
- **Esperar al tercer uso para extraer.** Descartada: con dos usos de una regla del sistema (como un ADR) ya hay riesgo de que diverjan.

## Consecuencias

- Cada regla transversal vive en un solo lugar, con sus pruebas, y se puede explicar diciendo dónde está y quién la usa.
- Algunas issues incluyen una refactorización de código ya mergeado. Sus pruebas existentes deben seguir pasando sin cambios de comportamiento.
- La regla está en la skill `/hexagonal`, así que se aplica igual en los dos servicios.
