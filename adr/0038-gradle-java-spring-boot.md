# ADR-0038. Gradle en los tres repos, Java 25 en los backends, Java 21 en la app y Spring Boot 4

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Los backends se implementan con Java y Spring Boot (ENUNCIADO §3.1) y la app con Kotlin Multiplatform (ENUNCIADO §3.2). El enunciado no fija herramienta de build ni versiones.

## Decisión

- **Gradle**, con Kotlin DSL y el wrapper (`./gradlew`) commiteado, en los tres repos.
- **Java 25** en los dos backends y **Java 21** en la app, fijados con el toolchain de Gradle.
- **La última versión estable de Spring Boot 4.x** al crear los backends.

## Alternativas consideradas

- **Maven en los backends.** Descartada: la app KMP ya usa Gradle, así que se trabajaría con dos herramientas de build distintas sin ningún beneficio.
- **Depender del JDK instalado en la máquina.** Descartada: el build daría resultados distintos según el entorno. El toolchain descarga y usa la versión fijada.

## Consecuencias

- Una sola herramienta de build y un solo comando (`./gradlew`) para compilar y probar cualquier repo.
- Java 25 es la versión LTS vigente. La app usa Java 21 porque es la versión que soporta el tooling de Android y KMP.
