# ADR-0027. El teléfono se valida con la regla de la cátedra antes de publicarlo

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

La cátedra normaliza el teléfono (quita espacios, guiones, paréntesis y puntos) y exige `^\+?[0-9]{7,15}$`; si no cumple, responde con `AdditionalInformationRejected` por Kafka (REF §15.5, §15.7). Cada envío inválido consume tiempo del hold y un ida y vuelta por Kafka.

## Decisión

El servicio de turnos aplica la misma normalización y la misma regla antes de publicar. Un teléfono inválido se rechaza en el momento, sin publicar, y el usuario puede corregirlo. Se publica el teléfono ya normalizado.

## Alternativas consideradas

- **Publicar siempre y dejar que valide la cátedra.** Descartada: hace esperar al usuario un rechazo que se puede detectar al instante y consume tiempo del hold.

## Consecuencias

- Respuesta inmediata ante errores de formato.
- El rechazo de la cátedra se sigue manejando ([ADR-0023](0023-estados-solo-avanzan.md)), por si su validación incluye algo más.
