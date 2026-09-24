# ADR-0012. PostgreSQL como motor de base de datos de los dos servicios

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

Cada servicio necesita una base de datos con servidor; no se aceptan H2, SQLite ni bases embebidas o en memoria (ENUNCIADO §3). El motor lo elige el alumno (ENUNCIADO §3, §10). Las decisiones de sincronización requieren que las lecturas no se bloqueen mientras se aplica una transacción de escritura ([ADR-0010](0010-snapshot-en-una-transaccion.md)).

## Decisión

Los dos servicios usan **PostgreSQL**. Cómo se separan las bases (instancias distintas o una instancia con bases separadas) se define en [`arq/despliegue.md`](../arq/despliegue.md), respetando la [Constitución P-03](../constitucion.md).

## Alternativas consideradas

- **MySQL.** Cumple los requisitos, pero sus cambios de esquema no son transaccionales: una migración que falla a mitad de camino puede quedar aplicada a medias. No aporta nada que PostgreSQL no tenga.
- **Motores distintos para cada servicio.** Permitido (ENUNCIADO §3), pero duplica la configuración, las imágenes y el conocimiento necesario sin ningún beneficio para este proyecto.

## Consecuencias

- Las lecturas ven una versión consistente sin bloquearse por una sincronización en curso.
- Las migraciones son transaccionales: si una falla, no queda aplicada a medias.
- Es el motor por defecto de JHipster, lo que simplifica la compatibilidad con su modelo de usuario (ENUNCIADO §3.2).
- Tiene imagen oficial de Docker para Docker Compose.
