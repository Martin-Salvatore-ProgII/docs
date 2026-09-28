# ADR-0048. La interfaz se construye con Compose Multiplatform en el código común

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

La interfaz se desarrolla con Kotlin Multiplatform y tiene que tener una app Android ejecutable. Una segunda plataforma es opcional y se puede agregar como extensión (ENUNCIADO §3.2).

## Decisión

Las pantallas se construyen con Compose Multiplatform, en el código común de KMP, con componentes de Material 3. El código específico de Android queda limitado a lo que la plataforma exige (por ejemplo, el almacenamiento cifrado y la configuración de red).

## Alternativas consideradas

- **Interfaz nativa de Android con solo la lógica compartida.** Descartada: agregar otra plataforma obligaría a rehacer todas las pantallas.

## Consecuencias

- Una eventual segunda plataforma reutiliza las pantallas.
- Material 3 da componentes consistentes sin diseñarlos desde cero.
