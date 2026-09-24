# ADR-0006. La sincronización se dispara por Kafka, al arrancar y con un chequeo periódico

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

Kafka avisa cuando hay una versión nueva del catálogo (REF §15.3), pero el sistema tiene que soportar la pérdida de notificaciones y los reinicios (ENUNCIADO §8). Si el único disparador fuera Kafka, un aviso perdido sin otro posterior dejaría la copia local atrasada indefinidamente.

## Decisión

Un ciclo de sincronización se inicia por cualquiera de tres motivos: un aviso `CatalogUpdated`, el arranque del servicio y un chequeo periódico con intervalo configurable (valor inicial: 5 minutos). En todos los casos la decisión se toma comparando la versión local con Redis, no con el número del aviso.

## Alternativas consideradas

- **Solo Kafka.** Descartada: no cubre los avisos perdidos ni lo publicado mientras el servicio estuvo apagado.
- **Agregar un endpoint para forzar la sincronización.** Descartada: suma una operación más para proteger y no cubre ningún caso que no cubran el arranque y el chequeo periódico. Para la demo de discontinuidad alcanza con reiniciar el servicio.

## Consecuencias

- Un aviso perdido se recupera, como máximo, en el siguiente chequeo periódico.
- El chequeo periódico es barato: si la versión local es igual a la actual, no hace nada más que leer dos valores de Redis.
- Pueden coincidir dos disparadores; eso se resuelve en [ADR-0007](0007-concurrencia-optimista-version-local.md).
