# ADR-0040. Los paquetes siguen la convención de la skill `/hexagonal`

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

La arquitectura hexagonal se implementa tal cual la enseña la cátedra ([Constitución P-02](../constitucion.md)). La skill `/hexagonal` define la estructura de paquetes: `com.example.<servicio>.<feature>.{domain,application,infrastructure}` y un paquete `shared` para lo transversal.

## Decisión

Se usa exactamente esa convención. Los prefijos de cada servicio son `com.example.catalogo` y `com.example.turnos`.

## Alternativas consideradas

- **Un prefijo propio (por ejemplo, el dominio de la universidad).** Descartada: no aporta nada y aleja el código del patrón de referencia con el que se lo va a comparar.

## Consecuencias

- El código se puede comparar directamente con el patrón de la cátedra.
- Cambiar el prefijo en el futuro es un renombrado mecánico.
