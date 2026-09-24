# Registro de decisiones de arquitectura (ADR)

Cada decisión que el enunciado deja a nuestro criterio (ENUNCIADO §10), o que tiene alternativas razonables, se registra acá. Un ADR aceptado no se edita para cambiar la decisión: se escribe uno nuevo que lo reemplaza y el viejo pasa a **Reemplazado**.

## Índice

| ADR | Decisión | Estado |
| --- | --- | --- |
| [0001](0001-app-habla-con-ambos-servicios.md) | La app KMP se comunica directamente con los dos servicios | Aceptado |
| [0002](0002-comunicacion-unidireccional-turnos-catalogo.md) | Comunicación entre servicios en una sola dirección: turnos → catálogo | Aceptado |

## Plantilla

```markdown
# ADR-NNNN. Título en forma de decisión

- **Estado:** Propuesto | Aceptado | Reemplazado por ADR-NNNN
- **Fecha:** yyyy-MM-dd

## Contexto

Qué problema o necesidad obliga a decidir. Citar ENUNCIADO o REF.

## Decisión

Qué se decidió, en una o dos frases.

## Alternativas consideradas

- **Alternativa.** Por qué se descartó.

## Consecuencias

Qué implica la decisión: lo que facilita, lo que cuesta y lo que obliga a hacer en otros lados.
```
