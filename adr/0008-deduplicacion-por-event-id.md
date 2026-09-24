# ADR-0008. Los eventos Kafka procesados se registran por `eventId`

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

La entrega de Kafka es al menos una vez y `eventId` es la clave funcional de idempotencia (REF §4, §15.1). El diagrama de sincronización de la cátedra incluye el paso "deduplicar `eventId`" (REF §16), y el enunciado pide demostrar el procesamiento idempotente de un mensaje duplicado (ENUNCIADO §11). En el catálogo, reprocesar un `CatalogUpdated` ya es inofensivo por la comparación de versiones; en turnos, un evento duplicado sí podría repetir efectos.

## Decisión

Cada servicio registra en su base los `eventId` de los eventos Kafka que procesó. Un evento con un `eventId` ya registrado se reconoce como duplicado, se ignora sin repetir efectos y se confirma su offset. En el catálogo, este registro se suma a la comparación de versiones.

## Alternativas consideradas

- **En el catálogo, solo la comparación de versiones.** Descartada: es correcta, pero el duplicado no queda identificado como tal, no sigue el paso explícito de REF §16 y deja al catálogo con un criterio distinto del de turnos.

## Consecuencias

- El mismo criterio de idempotencia en los dos servicios, fácil de explicar y de demostrar.
- Un duplicado queda registrado de forma visible (en logs), lo que sirve como evidencia del §11.
- Una tabla más por servicio. Cuánto tiempo se conservan los `eventId` se define al implementar.
