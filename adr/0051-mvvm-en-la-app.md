# ADR-0051. La app KMP sigue la arquitectura MVVM de la skill `/mvvm-kmp`

- **Estado:** Aceptado
- **Fecha:** 2026-09-29

## Contexto

El enunciado no fija la arquitectura interna de la interfaz; su organización la define el alumno (ENUNCIADO §10). El profesor recomendó usar MVVM en la app. Los backends ya siguen un patrón único y documentado, la skill `/hexagonal` ([Constitución P-02](../constitucion.md)).

## Decisión

La app se organiza con MVVM tal como lo define la skill `/mvvm-kmp`: paquetes por feature con capas `ui`, `domain` y `data`; un ViewModel por pantalla que expone un único estado inmutable y recibe acciones; flujo de datos unidireccional; repositorios detrás de interfaces; DTO y mappers encerrados en la capa de datos; errores traducidos una sola vez por `status` y `code`.

## Alternativas consideradas

- **MVI.** Descartada: es muy parecida (estado único y acciones, que la skill ya adopta), pero agrega formalismo (reductores y estados intermedios explícitos) sin beneficio para este tamaño de app, y no es lo recomendado.
- **Sin una arquitectura explícita, con la lógica en las pantallas.** Descartada: mezcla presentación, red y estado, y deja la lógica sin forma de probarse.

## Consecuencias

- La app tiene un patrón único y documentado, igual que los backends.
- Las pantallas son funciones puras del estado; la lógica queda en ViewModels y casos de uso que se prueban con fakes ([ADR-0047](0047-tests-basicos-en-la-app.md)).
