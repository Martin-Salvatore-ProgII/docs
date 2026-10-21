# ADR-0065. El filtro de disponibilidad de la búsqueda acepta fechas entre hoy y el horizonte, y franjas completas

- **Estado:** Aceptado
- **Fecha:** 2026-10-20

## Contexto

[ADR-0013](0013-filtro-disponibilidad-de-agenda.md) define el filtro de disponibilidad de la búsqueda como un filtro de agenda: una fecha y, opcionalmente, una franja horaria. No dice qué fechas se aceptan ni si la franja puede venir incompleta.

[ADR-0015](0015-reglas-disponibilidad-turnos.md) limita la fecha de la disponibilidad de turnos a un rango entre hoy y un horizonte configurable. La app puede reutilizar la fecha de la búsqueda al pasar a la disponibilidad (ADR-0013), así que las dos operaciones reciben la misma fecha.

## Decisión

- La fecha del filtro tiene que estar **entre hoy y el horizonte de ADR-0015**, ambos incluidos. "Hoy" es la fecha en la zona horaria de la agenda ([ADR-0016](0016-zona-horaria-argentina.md)). Fuera de ese rango, la búsqueda se rechaza con `400 DATE_OUT_OF_RANGE`, el mismo código que usa la disponibilidad de turnos.
- La franja horaria va **completa**: `timeFrom` y `timeTo` juntos, con una fecha y con el inicio antes del fin. Una franja incompleta es un filtro inválido (`400 VALIDATION_ERROR`).

## Alternativas consideradas

- **Aceptar cualquier fecha.** Descartada: buscar en una fecha pasada no tiene uso, y la búsqueda ofrecería profesionales para una fecha que el paso siguiente rechaza.
- **Franja abierta (`timeFrom` solo, "desde esa hora").** Descartada: amplía el contrato sin un caso de uso claro. Una franja como "a la tarde" se expresa con su inicio y su fin.

## Consecuencias

- La búsqueda y la disponibilidad de turnos aceptan el mismo rango de fechas, y un usuario no encuentra un profesional para una fecha en la que después no puede ver turnos.
- El horizonte se configura en los dos servicios y tiene que valer lo mismo en ambos ([RNF-09](../requisitos/no-funcionales.md#rnf-09-sin-supuestos-fijos-sobre-los-datos-de-la-cátedra-y-valores-operativos-configurables)).
- El catálogo necesita la hora actual para resolver el filtro. Se inyecta, para que las pruebas la puedan fijar.
