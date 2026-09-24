# ADR-0005. Los backends obtienen la configuración de la integración al arrancar

- **Estado:** Aceptado
- **Fecha:** 2026-09-23

## Contexto

La cuenta técnica se registra a mano una sola vez (ENUNCIADO §5, punto 2). La cátedra devuelve un JWT técnico y un objeto `integration` con los datos de conexión: Redis (host, puerto, usuario, contraseña), Kafka (servidores, consumer group, topics) y el estado de aprovisionamiento (REF §5.1). El mismo objeto se puede volver a pedir con `GET /api/student/integration` (REF §5.3).

ENUNCIADO §5, punto 3, pide que la aplicación permita "autenticar la cuenta de integración y recuperar la configuración asignada". No aclara si lo hace el sistema o la persona que lo configura.

## Decisión

La configuración externa de cada backend contiene solo la **URL base de la API de la cátedra** y el **JWT técnico**. Al arrancar, cada backend pide `GET /api/student/integration` y con esa respuesta configura sus conexiones a Redis y Kafka. Si la respuesta no llega o `provisioningStatus` no es `PROVISIONED`, el servicio no arranca y lo informa.

## Alternativas consideradas

- **Copiar a mano todos los valores de `integration` a la configuración.** Es más simple de implementar. Descartada porque obliga a manejar más secretos a mano (entre ellos la contraseña de Redis), porque no queda en el sistema nada que muestre el punto 3 del §5 y porque, si la cátedra cambia un valor, hay que editar la configuración de los dos servicios.

## Consecuencias

- Menos secretos para administrar: la contraseña de Redis nunca se escribe en un archivo nuestro.
- El punto 3 del §5 queda cubierto por el propio sistema y se puede demostrar.
- El arranque depende de que la API de la cátedra responda. Si no responde, el servicio falla al iniciar, de forma visible, en lugar de quedar a medio configurar.
- Las conexiones a Redis y Kafka se configuran con valores obtenidos en tiempo de ejecución, no con propiedades fijas, lo que suma trabajo de configuración en los backends.
- **A confirmar con el profesor:** la interpretación del punto 3 del §5.
