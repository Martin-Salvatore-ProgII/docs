# ADR-0081. Mis reservas y su detalle en la app

- **Estado:** Aceptado
- **Fecha:** 2026-11-23

## Contexto

En la M5 de la app se construyen Mis reservas y Detalle de reserva ([HU-08](../requisitos/HU.md#hu-08-consultar-mis-reservas), [HU-09](../requisitos/HU.md#hu-09-cancelar-una-reserva)). Los contratos ya están: `GET /api/reservations` con un filtro `status` de un solo valor, `GET /api/reservations/{id}` y `POST /api/reservations/{id}/cancel` ([`arq/contratos.md`](../arq/contratos.md)). La cancelación sigue [ADR-0026](0026-cancelacion-con-respuesta-rest.md) y [ADR-0071](0071-no-se-cancela-un-turno-que-ya-empezo.md). Quedaban sin decidir:

- qué filtros por estado ofrece la lista, si el contrato acepta un solo estado por pedido;
- a dónde lleva una reserva que todavía está en curso;
- cómo se ofrece la cancelación y qué pasa con un turno que ya empezó;
- cómo se entera la lista de los cambios.

## Decisión

**Tres filtros: Todas, Confirmadas y Canceladas.** Cada chip es un valor de `status` del contrato (o ninguno). No hay un chip "En curso": hay como máximo un proceso activo ([ADR-0024](0024-un-proceso-activo-por-usuario.md)), aparece primero en Todas y Buscar ya lo ofrece ([ADR-0080](0080-disponibilidad-y-reserva-en-la-app.md)). Las reservas que no se completaron se ven en Todas.

**La tarjeta muestra lo que guarda turnos.** Fecha, profesional, horario de inicio a fin y estado. Sin la categoría: la reserva no la trae, y pedírsela al catálogo haría que la lista dependa de él ([ADR-0025](0025-reservas-propias-desde-base-local.md)).

**Una reserva en curso abre Reserva en curso; una final, el Detalle.** Reserva en curso ya sigue el proceso y pide el teléfono; un detalle sería una copia peor.

**Cancelar abre un diálogo de confirmación con el motivo opcional.** El botón "Cancelar turno" aparece solo en una reserva `CONFIRMED` cuyo turno todavía no empezó, en la zona horaria del negocio ([ADR-0016](0016-zona-horaria-argentina.md)). El diálogo trae el campo del motivo, de hasta 500 caracteres. La autoridad sigue siendo turnos: ante `409 RESERVATION_NOT_CANCELLABLE` la app avisa que el turno ya no se puede cancelar y recarga la reserva.

**La lista se recarga cada vez que se muestra.** Al volver del Detalle o de Reserva en curso, pide de nuevo la primera página con el filtro elegido. Así refleja la cancelación y también los cambios que llegan por Kafka.

## Alternativas consideradas

- **Aceptar varios estados en el filtro de turnos** para tener grupos como "En curso" o "Sin completar". Descartada: cambia un contrato, un servicio y pruebas ya aprobados, para algo que es solo de presentación.
- **Sumar un chip Vencidas.** Descartada: dejaría afuera los fallidos e inválidos, y la separación confundiría.
- **Llevar siempre al Detalle.** Descartada: suma un toque y repite lo que ya muestra Reserva en curso.
- **Mostrar el botón aunque el turno haya empezado** y dejar que avise el 409. Descartada: ofrece algo que siempre falla.
- **Devolverle a la lista la reserva cancelada** en lugar de recargar. Descartada: ahorra un pedido, pero no ve otros cambios y es un mecanismo más.

## Consecuencias

- Ningún contrato cambia.
- Mis reservas hace un pedido cada vez que se muestra.
- Si el reloj del teléfono está atrasado, el botón puede verse unos minutos de más; el 409 lo corrige.
