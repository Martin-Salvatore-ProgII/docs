# ADR-0066. Turnos toma el JWT que reenvía al catálogo del pedido en curso, así que solo llama al catálogo dentro de un pedido del usuario

- **Estado:** Aceptado
- **Fecha:** 2026-10-29

## Contexto

[ADR-0034](0034-propagacion-jwt-usuario-entre-servicios.md) decide que turnos reenvía al catálogo el JWT del usuario que recibió de la app, pero no dice cómo llega ese token hasta el cliente que llama al catálogo. Hay dos formas: pasarlo como parámetro desde el controller, por la fachada, el caso de uso y el puerto, o tomarlo en el cliente HTTP del pedido que se está atendiendo, que Spring Security ya validó.

## Decisión

- El cliente HTTP del catálogo toma el JWT del **pedido en curso** y lo agrega a cada llamada. El puerto del catálogo y los casos de uso no reciben el token: sus firmas tienen solo datos del dominio.
- En consecuencia, turnos **llama al catálogo solo dentro de un pedido del usuario**. Un consumidor de Kafka, una tarea programada o código que corra en otro hilo no pueden llamar al catálogo. Si alguno lo intenta, la llamada falla sin llegar al catálogo.
- Cuando el catálogo rechaza el token (vencido entre las dos llamadas), turnos responde `401` como ante cualquier token vencido.

## Alternativas consideradas

- **El token como parámetro del puerto.** Más explícito, pero mete un detalle de autenticación en las firmas del dominio y de la aplicación, que no deberían conocer cómo se autentica turnos ante el catálogo.

## Consecuencias

- El dominio y los casos de uso no dependen de la seguridad, y cambiar la forma de autenticarse ante el catálogo no toca sus firmas.
- La reconciliación de procesos ([ADR-0030](0030-reconciliacion-periodica-de-procesos.md)) y los consumidores de Kafka no pueden usar el catálogo. Hoy ninguno lo necesita. Si alguno lo necesitara, hace falta el token de servicio que anticipa [ADR-0034](0034-propagacion-jwt-usuario-entre-servicios.md), con un nuevo ADR.
- La dependencia del pedido en curso no se ve en las firmas: se explica en el cliente HTTP del catálogo y en este ADR.
