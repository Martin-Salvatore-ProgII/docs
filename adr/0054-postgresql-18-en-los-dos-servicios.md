# ADR-0054. PostgreSQL 18 en los dos servicios, con la versión mayor fijada

- **Estado:** Aceptado
- **Fecha:** 2026-10-04

## Contexto

Los dos servicios usan PostgreSQL ([ADR-0012](0012-postgresql.md)), cada uno en su propio contenedor ([ADR-0041](0041-postgresql-por-servicio.md)), y las pruebas corren contra PostgreSQL real con Testcontainers ([ADR-0045](0045-postgresql-real-en-tests.md)). Ninguno de esos ADR fija la versión. La imagen se nombra en cuatro lugares: el Compose y la configuración de tests de cada servicio.

## Decisión

Los dos servicios usan **PostgreSQL 18**, la última versión mayor estable al crear los backends. En el Compose y en los tests la imagen se escribe con la versión mayor fija (`postgres:18`), nunca `latest`. Cambiar de versión mayor se hace en los cuatro lugares a la vez.

## Alternativas consideradas

- **`postgres:latest`.** Descartada: el motor cambiaría de versión mayor sin que nadie lo decida, y los tests y el Compose podrían correr versiones distintas según cuándo se descargó cada imagen.
- **Fijar también la versión menor (por ejemplo `18.6`).** Descartada: las versiones menores de PostgreSQL solo traen correcciones y son compatibles entre sí; fijarlas obliga a actualizar la etiqueta a mano sin ganar reproducibilidad relevante para este proyecto.
- **Una versión distinta en cada servicio.** Descartada por la misma razón que motores distintos ([ADR-0012](0012-postgresql.md)): duplica lo que hay que conocer sin beneficio.

## Consecuencias

- Las pruebas y el Compose corren el mismo motor, en los dos servicios: lo que se prueba es lo que se demuestra.
- Las correcciones de versiones menores llegan solas al volver a descargar la imagen.
- Subir de versión mayor es un cambio explícito, en un commit por repo.
