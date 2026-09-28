# ADR-0015. Reglas de la disponibilidad de turnos: una fecha, horizonte acotado, solo slots completos y futuros

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

El servicio de turnos arma la disponibilidad con las reglas semanales vigentes, las ocupaciones de la cátedra y la fecha y el profesional elegidos (ENUNCIADO §7). Cada horario semanal define inicio, fin y duración de los slots (REF §7). El enunciado no define cuántas fechas se consultan a la vez, hasta cuándo, ni qué hacer con un slot que no entra completo en el horario o que ya pasó.

## Decisión

- La disponibilidad se consulta **para una fecha por vez**.
- La fecha debe estar entre **hoy y un horizonte configurable** (valor inicial: 60 días). Fuera de ese rango, la consulta se rechaza como entrada inválida.
- Un slot se ofrece solo si **termina dentro del horario** (inicio + duración ≤ fin).
- Para la fecha de hoy, no se ofrecen slots cuyo inicio **ya pasó** según la hora de Argentina ([ADR-0016](0016-zona-horaria-argentina.md)).
- No se ofrecen slots ocupados según la cátedra, ya sean `CONFIRMED` o `HELD`.

## Alternativas consideradas

- **Un rango de fechas por consulta (por ejemplo, una semana).** Descartada por ahora: multiplica los casos borde y el §7 habla de una fecha elegida. Se puede agregar después sin romper el contrato de una fecha.
- **Sin horizonte.** Descartada: habilita consultas sin sentido y va contra la validación de entradas de ENUNCIADO §9.
- **Ofrecer el último slot aunque exceda el fin del horario.** Descartada: sería ofrecer atención fuera de la agenda publicada, y la cátedra probablemente lo rechace con `INVALID_SLOT` (REF §13.1).

## Consecuencias

- Una sola consulta de ocupaciones por pedido (`from` = `to`).
- La disponibilidad es una foto del momento: un slot mostrado como libre puede ocuparse antes de reservarlo. La cátedra lo resuelve al crear el hold (`SLOT_ALREADY_HELD`, `SLOT_ALREADY_RESERVED`).
- La disponibilidad no se guarda; se calcula en cada consulta.
