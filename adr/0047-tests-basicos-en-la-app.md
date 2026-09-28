# ADR-0047. La app KMP tiene pocas pruebas, básicas y sobre lógica pura

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Las pruebas de la interfaz KMP son opcionales y se consideran trabajo complementario (ENUNCIADO §10.1). Aun así, la app aplica algunas reglas por su cuenta cuyo error sería visible para el usuario.

## Decisión

La app tiene pocas pruebas unitarias, en el código común y sin emulador ni pruebas de interfaz, sobre: la normalización y validación del teléfono, qué se muestra para cada estado de una reserva, la interpretación de errores `problem+json` y las validaciones de formularios. Se ejecutan en CI como los backends.

## Alternativas consideradas

- **Sin pruebas en la app.** Descartada: las reglas de la app quedarían sin ninguna verificación.
- **Pruebas de interfaz con emulador.** Descartada: alto costo de armado y mantenimiento para algo opcional.

## Consecuencias

- La diferencia de profundidad con los backends es intencional y está documentada en [`arq/pruebas.md`](../arq/pruebas.md): los backends son lo obligatorio y evaluado.
