# ADR-0001. La app KMP se comunica directamente con los dos servicios

- **Estado:** Aceptado
- **Fecha:** 2026-09-23

## Contexto

La app KMP necesita funcionalidades de los dos backends: búsquedas de profesionales (servicio de catálogo) y disponibilidad, reservas y cancelaciones (servicio de turnos). El enunciado exige exactamente dos servicios backend (ENUNCIADO §3) con responsabilidades separadas (ENUNCIADO §4) y no define cómo los consume la interfaz.

## Decisión

La app llama directamente a cada servicio por lo que es suyo: al catálogo para búsquedas y datos del catálogo, y a turnos para disponibilidad y reservas. Ambos servicios validan el JWT del usuario final.

## Alternativas consideradas

- **La app habla solo con turnos, y turnos reenvía las búsquedas al catálogo.** Descartada: turnos se convierte en intermediario de funciones que no le pertenecen, se desdibuja la separación de responsabilidades (ENUNCIADO §4) y una caída de turnos deja sin búsquedas a la app aunque el catálogo funcione.
- **Un API gateway delante de los dos servicios.** Descartada: agrega un tercer componente backend, cuando el enunciado pide exactamente dos (ENUNCIADO §3), y suma infraestructura que no aporta a los objetivos evaluados.

## Consecuencias

- Cada servicio expone y documenta su propia API hacia la app ([`arq/contratos.md`](../arq/contratos.md)).
- La app conoce dos URLs base, que van en su configuración.
- Los dos servicios tienen que validar el JWT del usuario final; cómo se emite y se comparte la validación se decide en [`arq/seguridad.md`](../arq/seguridad.md).
- CORS y la protección del canal app → backend se configuran en ambos servicios (ENUNCIADO §9).
- Si un servicio cae, la app sigue pudiendo usar el otro.
