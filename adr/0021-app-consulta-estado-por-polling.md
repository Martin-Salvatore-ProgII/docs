# ADR-0021. La app sigue el proceso de reserva consultando su estado

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

El pedido de teléfono y el resultado de la reserva llegan al servicio de turnos por Kafka, en cualquier momento después de la confirmación inicial (REF §15.4–§15.10). La app tiene que enterarse para pedir el teléfono y mostrar el resultado. El proceso completo dura, como mucho, lo que dura el hold.

## Decisión

La app consulta periódicamente al servicio de turnos el estado del proceso mientras no sea final, con un intervalo corto configurable en la app (valor inicial: 2 segundos).

## Alternativas consideradas

- **Notificación push (WebSocket, SSE o notificaciones del sistema).** Descartada: agrega infraestructura y exige manejar reconexiones y mensajes perdidos, para un proceso que dura pocos minutos.

## Consecuencias

- Si la app se cierra y se vuelve a abrir, consulta el estado y retoma donde quedó el proceso.
- La consulta del estado del proceso es la misma operación que sirve para ver una reserva ([`arq/contratos.md`](../arq/contratos.md)).
- La carga es acotada: una consulta liviana cada pocos segundos, solo mientras hay un proceso activo.
