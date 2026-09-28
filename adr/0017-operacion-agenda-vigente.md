# ADR-0017. El catálogo expone la agenda vigente de un profesional en una sola operación

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

Antes de armar la disponibilidad o iniciar una reserva, el servicio de turnos necesita del catálogo los datos vigentes de un profesional y sus horarios semanales (ENUNCIADO §4.2, §7), por un contrato explícito ([ADR-0002](0002-comunicacion-unidireccional-turnos-catalogo.md)). El catálogo puede estar aplicando una sincronización mientras tanto.

## Decisión

El catálogo ofrece la operación **agenda vigente de un profesional**, que en una sola lectura consistente devuelve: los datos del profesional y su categoría, si está habilitado según [ADR-0014](0014-habilitado-efectivo.md), sus horarios semanales habilitados y la versión local del catálogo con la que se respondió. Si el profesional no existe, lo informa con un error específico. La misma operación la usa la app para el detalle de un profesional.

El contrato está en [`arq/contratos.md`](../arq/contratos.md) y en [`arq/contratos/catalogo-api.yaml`](../arq/contratos/catalogo-api.yaml). Cómo se autentica turnos se decide en la parte de seguridad.

## Alternativas consideradas

- **Que turnos use la búsqueda de la app.** Descartada: es una lista paginada con filtros, pensada para otro uso, y no incluye los horarios.
- **Dos operaciones separadas (profesional y horarios).** Descartada: una sincronización entre las dos llamadas podría devolver datos de versiones distintas, y duplica la latencia.

## Consecuencias

- Turnos recibe todo lo que necesita, consistente y en una sola llamada.
- La versión informada permite saber con qué versión del catálogo se calculó cada disponibilidad.
- Una sola operación para dos consumidores (turnos y la app).
