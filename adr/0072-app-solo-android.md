# ADR-0072. La app se compila solo para Android

- **Estado:** Aceptado
- **Fecha:** 2026-11-12

## Contexto

La interfaz se desarrolla con Kotlin Multiplatform y tiene que tener una app Android ejecutable. Otra plataforma es opcional y se puede agregar como extensión (ENUNCIADO §3.2).

## Decisión

La app tiene un único destino de compilación, Android. El código se sigue escribiendo en el código común de KMP ([ADR-0048](0048-compose-multiplatform.md)); el código de Android queda limitado a lo que la plataforma exige.

## Alternativas consideradas

- **Sumar iOS.** Descartada: compilar y probar para iOS exige una Mac, y duplica lo que hay que probar en algo opcional.
- **Sumar escritorio o web.** Descartada: no aporta a lo evaluado y suma configuración y pruebas.

## Consecuencias

- Hay una sola implementación de lo que depende de la plataforma (almacenamiento cifrado del token, configuración de red), sin `expect`/`actual` ([ADR-0052](0052-stack-de-la-app.md)).
- Sumar otra plataforma más adelante es posible: las pantallas y la lógica ya están en el código común, y habría que agregar su destino y su implementación de cada interfaz de plataforma.
