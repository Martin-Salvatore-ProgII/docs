# ADR-0013. El filtro de disponibilidad de la búsqueda es de agenda: fecha y franja horaria opcional

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

La búsqueda de profesionales debe ofrecer, como mínimo, un filtro por disponibilidad, y los cuatro filtros obligatorios se resuelven con la copia local del catálogo (ENUNCIADO §4.1). Consultar al servicio central en cada búsqueda no se acepta (ENUNCIADO §6.2). Qué turnos están realmente libres depende de las ocupaciones, que solo informa la cátedra (REF §8).

## Decisión

El filtro de disponibilidad es **de agenda**: recibe una **fecha** y, opcionalmente, una **franja horaria**. Devuelve los profesionales que tienen al menos un horario semanal habilitado para el día de la semana de esa fecha y, si se indicó la franja, que se superpone con ella. No garantiza que haya turnos libres; eso se ve al consultar la disponibilidad de turnos en el servicio de turnos.

La app puede reutilizar la fecha elegida en la búsqueda al pasar a la disponibilidad de turnos.

## Alternativas consideradas

- **Filtrar por día de la semana.** Descartada: el usuario piensa en fechas ("el jueves 15"), no en días de la semana, y la fecha se puede reutilizar en el paso siguiente.
- **Filtrar por turnos realmente libres.** Descartada: requiere consultar ocupaciones a la cátedra en cada búsqueda, lo que contradice ENUNCIADO §4.1 y §6.2.

## Consecuencias

- El filtro se resuelve con datos locales, así que funciona aunque la cátedra no esté disponible.
- Un profesional puede aparecer con la agenda disponible y tener todos los turnos ocupados. Queda documentado como comportamiento esperado.
