# ADR-0058. Los campos de auditoría los completa Spring Data, no los casos de uso

- **Estado:** Aceptado
- **Fecha:** 2026-10-06

## Contexto

El modelo de usuario compatible con JHipster incluye cuatro campos de auditoría: quién y cuándo creó el registro, y quién y cuándo lo modificó por última vez ([`arq/modelo-db.md`](../arq/modelo-db.md)). Otras entidades, como el proceso de reserva, también guardan fechas de creación y modificación. Hay que decidir qué parte del código los completa.

## Decisión

Los completa la **auditoría automática de Spring Data JPA**, en la capa de infraestructura: las entities marcan esos campos y se llenan solos al guardar. Es el mismo mecanismo que usa JHipster.

El autor del cambio es el login del usuario autenticado. Cuando no hay ninguno, como en el registro, es el valor fijo `system`.

## Alternativas consideradas

- **Que los complete el caso de uso**, con un reloj inyectado. Descartada: quién y cuándo se guardó un registro es un dato técnico de persistencia, no una regla de negocio. Además obliga a repetir el mismo código en cada caso de uso que guarda algo.

## Consecuencias

- Los casos de uso se ocupan solo de las reglas de negocio, y sus pruebas no necesitan controlar la hora para estos campos.
- Se configura una vez y lo usan todas las entities.
- Los campos se llenan sin que ninguna línea del caso de uso lo haga. Para que no sea opaco, la entity y la clase de configuración llevan un comentario que lo explica.
- En un registro, el autor queda como `system` y no como el propio usuario.
