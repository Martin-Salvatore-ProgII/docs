# ADR-0082. Administración de reservas en la app

- **Estado:** Aceptado
- **Fecha:** 2026-11-23

## Contexto

En la M7 de la app se construye la sección de administración ([HU-10](../requisitos/HU.md#hu-10-administrar-reservas)), con el alcance cerrado de [ADR-0037](0037-rol-administrador-alcance-cerrado.md). Los contratos ya están en turnos: `GET /api/admin/reservations` con los filtros `ownerLogin` (coincidencia exacta), `status` (un solo valor), `from` y `to`; `GET /api/admin/reservations/{id}` y `POST /api/admin/reservations/{id}/cancel` con motivo obligatorio ([`arq/contratos.md`](../arq/contratos.md#administración)). Quedaban sin decidir:

- cómo sabe la app que el usuario es administrador;
- qué filtros por estado y por dueño ofrece la lista, y cómo se aplican;
- cómo se muestran los datos de soporte, que incluyen el `failureReason`;
- a dónde lleva una reserva que todavía está en curso.

## Decisión

**El rol sale del claim `auth` del JWT, leído en el teléfono.** Igual que el vencimiento ([ADR-0077](0077-sesion-de-la-app.md)), la sesión lee los roles del JWT guardado, sin verificar la firma, y la pestaña Admin se muestra solo si incluye `ROLE_ADMIN`. Es solo interfaz: un JWT alterado mostraría la pestaña, pero turnos valida la firma y responde 401 o 403 en cada operación administrativa.

**Filtro por estado con todos los estados.** Un selector ofrece los diez estados de [`maquina-estados.md`](../arq/maquina-estados.md), cada uno con su nombre en español, o ninguno para todos. La lista y el detalle admin muestran el estado exacto, sin agrupar los pasos en curso como Mis reservas: el administrador busca justamente los procesos vencidos, fallidos, inválidos o detenidos en un paso (ADR-0037, Demostración).

**Filtro por dueño al confirmar, y fechas independientes.** El campo Usuario busca al tocar Buscar en el teclado, no mientras se escribe: turnos compara el login completo, así que buscar letra por letra daría listas vacías hasta terminar de escribirlo. Desde y Hasta son dos filtros independientes que abren el selector de fecha, sin límite de hoy ni de horizonte; el selector no deja elegir un Desde posterior al Hasta. Los filtros se suman como en Buscar ([ADR-0079](0079-busqueda-y-estado-del-catalogo-en-la-app.md)).

**Datos de soporte legibles, con el código al lado.** El detalle admin agrega un bloque con el dueño (nombre, apellido y login), el identificador del proceso en la cátedra (`reservationProcessId`), el motivo de la falla (el mensaje propio de la app y, entre paréntesis, el `failureReason`) y quién la canceló: el usuario, un administrador (con su login) o la cátedra (`CATEDRA`). Mostrar el código es una excepción a [ADR-0069](0069-toda-falla-del-hold-cierra-el-proceso.md), que vale para el usuario final: el administrador lo necesita para cruzarlo con los registros del servicio.

**Toda reserva abre el detalle admin, con un botón Actualizar.** Reserva en curso usa la ruta del dueño, que con una reserva ajena responde 404, y pide un teléfono que el administrador no puede enviar (ADR-0037). El detalle no consulta solo: se actualiza con el botón y al volver a mostrarse.

**Cancelar abre un diálogo con el motivo obligatorio.** Igual que el del dueño ([ADR-0081](0081-mis-reservas-en-la-app.md)), pero el botón Cancelar del diálogo se habilita recién con un motivo no vacío, de hasta 500 caracteres. El botón Cancelar turno aparece solo en una reserva `CONFIRMED` cuyo turno no empezó, y ante `409 RESERVATION_NOT_CANCELLABLE` la app avisa y recarga.

**La lista se recarga cada vez que se muestra**, con los filtros elegidos, igual que Mis reservas.

## Alternativas consideradas

- **Pedirle el rol a turnos** con un endpoint de cuenta. Descartada: suma un pedido y un cambio en un servicio aprobado para una decisión que es solo de interfaz.
- **Los chips de Mis reservas** (Todas, Confirmadas, Canceladas). Descartada: dejan afuera las vencidas, fallidas e inválidas, que son las que interesan para soporte.
- **Buscar por dueño mientras se escribe.** Descartada: con la coincidencia exacta de turnos, solo sirve con el login completo.
- **Mostrar el `failureReason` crudo** o solo el mensaje propio. Descartadas: el crudo obliga a conocer la tabla de códigos; sin el código no se puede cruzar con los registros.
- **Consultar el detalle cada pocos segundos** mientras la reserva está en curso. Descartada: es un mecanismo más sin un requisito que lo pida.

## Consecuencias

- Ningún contrato cambia.
- El detalle admin es la única pantalla que muestra un código técnico, y solo al administrador.
- Si el rol cambia mientras la sesión está abierta, la pestaña se actualiza recién con el próximo inicio de sesión; mientras tanto, turnos responde 403 y la app informa que no tiene permiso.
