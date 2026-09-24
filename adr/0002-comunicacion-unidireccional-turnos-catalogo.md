# ADR-0002. Comunicación entre servicios en una sola dirección: turnos → catálogo

- **Estado:** Aceptado
- **Fecha:** 2026-09-23

## Contexto

El servicio de turnos necesita datos vigentes del catálogo (profesional, horarios semanales) antes de armar la disponibilidad o iniciar una reserva, y no puede mantener su propia réplica (ENUNCIADO §3, §4.2, §7). En el enunciado no hay ningún caso en que el catálogo necesite información de turnos.

## Decisión

La única comunicación entre nuestros servicios es **turnos → catálogo**, por un contrato REST explícito protegido con JWT. El catálogo no llama a turnos.

## Alternativas consideradas

- **Comunicación en ambas direcciones.** Descartada: no hay un requisito que la justifique, crea una dependencia circular (ninguno de los dos podría arrancar ni probarse sin el otro) y acopla la disponibilidad del catálogo a la de turnos.
- **Integración por eventos propios (Kafka entre nuestros servicios).** Descartada para esta necesidad: turnos requiere el dato vigente **en el momento** de operar, y consumir eventos lo llevaría a mantener una réplica local del catálogo, lo que prohíbe ENUNCIADO §3.

## Consecuencias

- El catálogo no depende de turnos: se despliega, se prueba y funciona solo.
- Turnos depende del catálogo para operaciones nuevas. Si el catálogo no está disponible, no las inicia y sigue atendiendo lo que depende solo de sus datos (ENUNCIADO §8).
- El contrato turnos → catálogo se documenta en [`arq/contratos.md`](../arq/contratos.md) y en OpenAPI. La autenticación entre servicios se decide en [`arq/seguridad.md`](../arq/seguridad.md).
