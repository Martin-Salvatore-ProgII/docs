# ADR-0053. La regla de dependencias de la hexagonal se verifica con un test de ArchUnit

- **Estado:** Aceptado
- **Fecha:** 2026-10-04

## Contexto

La arquitectura hexagonal de la cátedra es obligatoria en los dos backends ([Constitución P-02](../constitucion.md), [ADR-0040](0040-paquetes-segun-skill-hexagonal.md)). Su regla central es de dependencias: `domain` no conoce a `application` ni a `infrastructure`, ni a Spring, JPA, Jackson o validación; `application` no conoce a `infrastructure`. La skill `/hexagonal` propone verificarla a mano, buscando imports. El código lo escriben el alumno y agentes de IA, en muchas issues, y un import fuera de lugar compila sin problemas.

## Decisión

Cada backend tiene un test con **ArchUnit** que falla si una clase de `domain` depende de `application`, de `infrastructure` o de esos frameworks, o si una clase de `application` depende de `infrastructure`. Corre con el resto de las pruebas, también en CI ([ADR-0046](0046-ci-con-github-actions.md)).

## Alternativas consideradas

- **Revisión manual con la checklist de la skill.** Se mantiene, pero sola no alcanza: depende de acordarse de hacerla en cada PR.
- **Un módulo de Gradle por capa.** Descartada: el compilador impediría el import, pero la estructura dejaría de parecerse al repo de referencia de la cátedra, que es un solo módulo con paquetes.

## Consecuencias

- Un PR que rompe la regla de dependencias no pasa el CI.
- La regla queda escrita como código ejecutable, que sirve de evidencia en la defensa.
- Una dependencia más, solo de test.
- El test cubre la dirección de las dependencias, no el resto del patrón (un puerto por caso de uso, la fachada, los mappers a mano): eso sigue en la revisión con la checklist.
