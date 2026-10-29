# ADR-0067. Una ocupación bloquea solo el slot que empieza a su misma hora

- **Estado:** Aceptado
- **Fecha:** 2026-10-29

## Contexto

[ADR-0015](0015-reglas-disponibilidad-turnos.md) decide que no se ofrecen los slots que la cátedra informa como ocupados, sean `CONFIRMED` o `HELD`, pero no dice cuándo una ocupación cae sobre un slot. La cátedra informa cada ocupación con una fecha y una hora de inicio, sin duración (REF §8).

En el caso normal no hay dudas: la cátedra solo acepta holds en horarios de la agenda vigente (`INVALID_SLOT`, REF §13.1), así que toda ocupación coincide con el inicio de un slot. La duda aparece si la agenda cambia después de una reserva. Por ejemplo, una reserva de las 11:30 hecha con slots de 30 minutos queda desalineada si después los slots pasan a ser de 20 (11:00, 11:20, 11:40...).

## Decisión

Una ocupación bloquea el slot de esa fecha que empieza **exactamente a su hora de inicio**. Una ocupación que no coincide con el inicio de ningún slot no bloquea ninguno.

## Alternativas consideradas

- **Bloquear el slot dentro del cual cae la hora de inicio de la ocupación.** En el ejemplo, la de las 11:30 bloquearía el slot de 11:20 a 11:40. Descartada: resuelve el caso a medias, porque sin la duración de la ocupación el slot de 11:40 se seguiría ofreciendo, y supone algo que la REF no dice. Además puede esconder slots que están libres.

## Consecuencias

- Es la misma forma en que la cátedra identifica un turno: fecha y hora de inicio.
- Después de un cambio de agenda, puede ofrecerse un slot que se superpone con una reserva vieja. Como la disponibilidad es una foto del momento, la cátedra decide al crear el hold (ADR-0015).
