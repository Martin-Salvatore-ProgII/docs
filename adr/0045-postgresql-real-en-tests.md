# ADR-0045. Las pruebas usan PostgreSQL real con Testcontainers, nunca H2

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Varias decisiones dependen del comportamiento del motor: la aplicación consistente del snapshot ([ADR-0010](0010-snapshot-en-una-transaccion.md)), la concurrencia sobre la versión local ([ADR-0007](0007-concurrencia-optimista-version-local.md)), la búsqueda sin distinguir tildes ni mayúsculas y un solo proceso activo por usuario ante pedidos concurrentes ([ADR-0024](0024-un-proceso-activo-por-usuario.md)). Las herramientas de test de persistencia de Spring usan por defecto una base embebida.

## Decisión

Las pruebas de persistencia y de integración usan PostgreSQL real en un contenedor, con las mismas migraciones de Flyway que producción ([ADR-0039](0039-flyway.md)).

## Alternativas consideradas

- **H2 u otra base en memoria.** Descartada: se comporta distinto en transacciones, concurrencia y funciones de texto, así que un test podría pasar y el sistema fallar. Además, el enunciado rechaza las bases en memoria como almacenamiento principal (ENUNCIADO §3).

## Consecuencias

- Lo que se prueba es lo que corre.
- Correr las pruebas requiere Docker, en la máquina de desarrollo y en CI.
