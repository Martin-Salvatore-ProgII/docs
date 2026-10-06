# ADR-0059. Los casos de uso informan los rechazos de negocio con excepciones de aplicación que llevan el código funcional

- **Estado:** Aceptado
- **Fecha:** 2026-10-06

## Contexto

Los errores se responden en `application/problem+json` con un `code` funcional estable, y los clientes deciden por ese código ([`arq/contratos.md`](../arq/contratos.md)). La skill `/hexagonal` muestra un solo caso: una consulta que no encuentra nada devuelve un resultado vacío, y la fachada lo traduce en la excepción de la feature. No dice qué hacer cuando una operación puede rechazarse por varios motivos, cada uno con su código, como un registro con el login o con el email repetido.

## Decisión

- Cuando hay **un solo motivo** de rechazo, se sigue la skill al pie de la letra: el caso de uso devuelve un resultado vacío y la fachada lanza la excepción. Es el caso del inicio de sesión.
- Cuando hay **varios motivos**, el caso de uso lanza la excepción de aplicación de la feature, que lleva el código funcional del rechazo.
- La excepción vive en la capa de aplicación y no conoce HTTP. El manejador global, en infraestructura, la traduce al estado HTTP y al `code` del contrato.

## Alternativas consideradas

- **Que el caso de uso devuelva un tipo de resultado con el motivo, y la fachada lance la excepción.** Más cercana a la letra de la skill. Descartada: obliga a crear un tipo de resultado que el repositorio de referencia no tiene, para llegar al mismo lugar.
- **Una clase de excepción por cada motivo.** Descartada: multiplica las clases y el manejador tendría que conocerlas una por una.

## Consecuencias

- La regla de dependencias se mantiene: el caso de uso depende de una clase de su misma capa.
- El manejador global conoce la excepción de cada feature. Cada feature nueva agrega ahí su traducción.
- Es una interpretación de la skill en un punto que no cubre. Queda para confirmar con la cátedra.
