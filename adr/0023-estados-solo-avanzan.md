# ADR-0023. Máquina de estados de la reserva: los estados solo avanzan y los finales no se reabren

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

El estado de un proceso de reserva se entera por dos canales que llegan en momentos distintos: respuestas REST y eventos Kafka, que además pueden llegar duplicados. La solución tiene que converger sin duplicar reservas ni retroceder un proceso que ya alcanzó un estado final (ENUNCIADO §7; REF §18.3). La máquina de estados local la define el alumno (ENUNCIADO §10).

## Decisión

Se adopta la máquina de estados de [`arq/maquina-estados.md`](../arq/maquina-estados.md), con estas reglas:

- cada estado representa lo que el proceso está esperando;
- un disparador sin transición definida desde el estado actual se ignora y se registra;
- los estados finales no se abandonan, salvo `CONFIRMED` → `CANCELLED`;
- los eventos terminales de la cátedra se aplican desde cualquier estado no final;
- el rechazo del teléfono vuelve a `AWAITING_PHONE` en lugar de ser un estado propio.

## Alternativas consideradas

- **Aplicar siempre el último evento recibido.** Descartada: un evento tardío o duplicado podría retroceder o reabrir un proceso final.
- **Un estado propio para el rechazo del teléfono.** Descartada: después de un rechazo se espera lo mismo que antes (un teléfono), así que sería un estado distinto con el mismo comportamiento.

## Consecuencias

- El resultado no depende del orden de llegada ni de la cantidad de entregas de cada evento.
- La app muestra algo distinto para cada estado, y la recuperación sabe qué reconciliar según el estado.
