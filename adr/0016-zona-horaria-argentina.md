# ADR-0016. La agenda usa la zona horaria fija de Argentina

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

Las horas de los horarios y de los turnos son "locales de atención" y los instantes se expresan en UTC (REF §4), pero el contrato no dice a qué zona horaria corresponde la hora de atención. El sistema necesita saber qué fecha es "hoy" y qué hora es "ahora" en la agenda para validar fechas y descartar slots pasados ([ADR-0015](0015-reglas-disponibilidad-turnos.md)).

## Decisión

La zona horaria de la agenda es **`America/Argentina/Buenos_Aires`**, **fija** y no configurable. Se usa para determinar "hoy" y "ahora". Las horas de la agenda se tratan siempre como hora local de esa zona.

## Alternativas consideradas

- **Zona horaria configurable.** Descartada: el sistema está pensado para profesionales que atienden en Argentina y no hay otra zona válida. Hacerla configurable solo agrega la posibilidad de configurarla mal, lo que haría ofrecer turnos pasados o esconder turnos válidos.
- **Usar la zona horaria del servidor.** Descartada: depende de dónde corra el contenedor (normalmente UTC) y daría resultados distintos según el entorno.

## Consecuencias

- El resultado no depende del entorno de ejecución.
- La cátedra no especificó la zona de la hora "local de atención". Si indicara otra, este ADR se reemplaza; el cambio queda acotado a un único valor.
