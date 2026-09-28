# ADR-0039. Flyway para las migraciones de base de datos

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Cada servicio administra sus propias migraciones (ENUNCIADO §3). Ni el enunciado ni la REF fijan la herramienta, y la skill `/hexagonal` admite Flyway o Liquibase. En el cursado se vio Liquibase, que es además la que usa JHipster. Como no usamos el generador de JHipster ([ADR-0018](0018-usuario-compatible-sin-generador-jhipster.md)), ninguna tecnología obligatoria exige Liquibase.

## Decisión

Cada backend usa **Flyway**, con migraciones escritas en SQL plano y versionadas dentro de su propio repo.

## Alternativas consideradas

- **Liquibase.** Descartada: describe los cambios en XML, YAML o JSON, una capa más entre el cambio y el SQL que realmente se ejecuta. Sus ventajas (independencia del motor, rollbacks generados) no aplican: el motor es uno solo, PostgreSQL ([ADR-0012](0012-postgresql.md)), y las migraciones no se revierten sino que se corrigen con una migración nueva.

## Consecuencias

- Cada migración es un archivo `.sql` que se lee, se revisa y se defiende línea por línea.
- Se aprovecha que los cambios de esquema en PostgreSQL son transaccionales: una migración que falla no queda aplicada a medias.
- La compatibilidad con el usuario de JHipster es de datos y de contrato, no de herramienta: las tablas se escriben a mano con los mismos campos.
