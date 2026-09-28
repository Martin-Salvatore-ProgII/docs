# Casos de uso

Flujos detallados, con sus alternativas y errores, solo para los procesos complejos o que no inicia una persona. Lo que una historia de usuario ya describe bien no se repite acá.

## Sincronización del catálogo

La estrategia completa, con la tabla de decisiones, está en [`arq/sincronizacion.md`](../arq/sincronizacion.md).

### CU-01. Sincronizar el catálogo

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §4.1, §6.2, §8; REF §14, §15.3, §16, §18.1
- **Actor:** servicio de catálogo
- **Disparadores:** aviso `CatalogUpdated` por Kafka, arranque del servicio o chequeo periódico ([ADR-0006](../adr/0006-disparadores-sincronizacion.md))

**Precondiciones:** el servicio obtuvo la configuración de la integración ([HU-03](HU.md#hu-03-alta-y-configuración-de-la-integración-técnica)).

**Flujo principal (incremental)**

1. Si el disparador es un aviso de Kafka, el servicio verifica que su `eventId` no haya sido procesado y lo registra ([ADR-0008](../adr/0008-deduplicacion-por-event-id.md)).
2. El servicio marca que hay una sincronización en curso.
3. Lee de Redis la versión actual (C) y la más vieja disponible (O), y las compara con su versión local (L).
4. Como O ≤ L < C, aplica en orden cada versión desde L+1 hasta C. Para cada una:
   1. lee los IDs afectados en `changes:{v}`;
   2. lee el estado actual de esas entidades en los hashes, y el de las entidades referenciadas que falten en la copia local ([ADR-0009](../adr/0009-referencias-faltantes-desde-redis.md));
   3. guarda las entidades (incluidas las deshabilitadas) y avanza la versión local a `v` en una sola unidad, siempre que la versión local siga siendo `v-1` ([ADR-0007](../adr/0007-concurrencia-optimista-version-local.md)).
5. Registra la sincronización exitosa (fecha, tipo y versión) y quita la marca de sincronización en curso.
6. Si el disparador fue un aviso de Kafka, confirma su offset.

**Postcondiciones:** la versión local es C y la copia local refleja el catálogo en C.

**Flujos alternativos**

- **1a. El `eventId` ya fue procesado:** se registra como duplicado, se confirma el offset y termina sin cambios (REF §18.1).
- **3a. L = C:** no hay nada que aplicar; va al paso 5.
- **3b. No hay versión local, L < O o L > C:** se ejecuta [CU-02](#cu-02-reconstruir-el-catálogo-desde-un-snapshot) y después continúa en el paso 5.
- **4a. Falta `changes:{v}` para alguna versión del rango, o un ID afectado o una entidad referenciada no está en su hash:** se descarta la versión en curso (L queda en `v-1`) y se ejecuta [CU-02](#cu-02-reconstruir-el-catálogo-desde-un-snapshot).
- **4b. Al guardar, la versión local ya no es `v-1` (otro ciclo la avanzó):** no se aplica nada y el ciclo termina sin error.
- **Cualquier paso. Redis, la API de la cátedra o la base no responden:** la versión local queda en la última aplicada por completo, se registra el error con su fecha, se quita la marca de sincronización en curso, se confirma el offset si hubo aviso y el ciclo termina. El próximo disparador vuelve a intentar ([ADR-0011](../adr/0011-fallas-y-estado-de-sincronizacion.md)).
- **1b. El aviso tiene un formato inválido:** se registra el error, se confirma el offset y se inicia igual un ciclo desde el paso 2, porque la decisión no depende del contenido del aviso.

### CU-02. Reconstruir el catálogo desde un snapshot

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §6.1, §6.2; REF §7, §18.2
- **Actor:** servicio de catálogo
- **Disparador:** [CU-01](#cu-01-sincronizar-el-catálogo) detecta base vacía, L < O, L > C o una discontinuidad

**Flujo principal**

1. El servicio pide el snapshot con `GET /api/synchronization/snapshot`.
2. Valida que la respuesta tenga `snapshotVersion` y las tres colecciones con datos válidos.
3. Reemplaza las tres colecciones y fija la versión local en `snapshotVersion`, todo en una sola unidad, siempre que la versión local siga siendo la leída al empezar ([ADR-0007](../adr/0007-concurrencia-optimista-version-local.md), [ADR-0010](../adr/0010-snapshot-en-una-transaccion.md)).
4. Vuelve a CU-01, paso 3: si mientras tanto se publicó una versión más nueva, la aplica por la vía incremental.

**Postcondiciones:** la copia local refleja exactamente `snapshotVersion`, con entidades habilitadas y deshabilitadas.

**Flujos alternativos**

- **1a. La API no responde o responde con error:** no se modifica la copia local; se registra el error y termina como la falla de CU-01.
- **2a. El snapshot es inválido (faltan datos o hay referencias inconsistentes):** no se aplica; se registra el error y termina como la falla de CU-01.
- **3a. La versión local cambió durante el ciclo:** no se aplica nada; otro ciclo ya la actualizó.

## Disponibilidad

### CU-03. Consultar la disponibilidad de turnos

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §4.2, §7, §8; REF §8
- **Actor:** usuario final, a través de la app
- **Historia:** [HU-06](HU.md#hu-06-ver-turnos-disponibles)

**Precondiciones:** el usuario está autenticado.

**Flujo principal**

1. El usuario elige un profesional y una fecha, y la app pide la disponibilidad al servicio de turnos.
2. Turnos valida que la fecha esté entre hoy y el horizonte permitido, según la hora de Argentina ([ADR-0015](../adr/0015-reglas-disponibilidad-turnos.md), [ADR-0016](../adr/0016-zona-horaria-argentina.md)).
3. Turnos pide al catálogo la agenda vigente del profesional ([ADR-0017](../adr/0017-operacion-agenda-vigente.md)).
4. Turnos pide a la cátedra las ocupaciones del profesional para esa fecha (REF §8).
5. Turnos arma los slots de los horarios habilitados de ese día de la semana, descarta los que no entran completos, los ya pasados y los ocupados.
6. Turnos devuelve los slots libres, junto con los datos del profesional y la versión del catálogo usada.

**Postcondiciones:** no se guarda nada; la disponibilidad es una foto del momento.

**Flujos alternativos**

- **2a. Fecha pasada o fuera del horizonte:** se rechaza como entrada inválida, sin consultar al catálogo ni a la cátedra.
- **3a. El profesional no existe:** se rechaza indicando que no existe.
- **3b. El profesional no está habilitado:** se rechaza indicando que está deshabilitado, sin consultar a la cátedra.
- **3c. El catálogo no responde, responde con error o todavía no tiene una versión local:** se rechaza indicando que el servicio no está disponible temporalmente. No se usa ningún dato guardado del catálogo ([Constitución P-05](../constitucion.md)).
- **4a. La cátedra no responde o responde con error:** se rechaza indicando que el servicio no está disponible temporalmente. No se muestran slots sin haber descontado las ocupaciones.
- **5a. El profesional no atiende ese día o no quedan slots libres:** se devuelve una lista vacía.

## Reserva

### CU-04. Reservar un turno

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §4.2, §7; REF §9, §10, §15.4–§15.9, §17
- **Actor:** usuario final, a través de la app
- **Historia:** [HU-07](HU.md#hu-07-reservar-un-turno)
- **Estados:** [`arq/maquina-estados.md`](../arq/maquina-estados.md)

**Precondiciones:** el usuario está autenticado y eligió un slot de la disponibilidad ([CU-03](#cu-03-consultar-la-disponibilidad-de-turnos)).

**Flujo principal**

1. La app pide iniciar la reserva con profesional, fecha y hora de inicio.
2. Turnos verifica que el usuario no tenga otro proceso activo ([ADR-0024](../adr/0024-un-proceso-activo-por-usuario.md)).
3. Turnos pide al catálogo la agenda vigente y verifica que el profesional esté habilitado y que el slot pertenezca a su agenda para esa fecha.
4. Turnos guarda el proceso en `STARTED`, asociado al usuario, con los datos históricos del profesional y del turno ([ADR-0022](../adr/0022-proceso-guardado-antes-de-llamar.md), [ADR-0025](../adr/0025-reservas-propias-desde-base-local.md)).
5. Turnos pide el hold a la cátedra con el `externalPatientId` del usuario. Guarda `holdId`, `reservationProcessId` y `expiresAt`, y pasa a `HELD`.
6. Turnos pide la confirmación inicial con el `externalPatientId`, el nombre y el apellido del usuario, y pasa a `AWAITING_REQUEST`. Responde a la app con el proceso.
7. La app consulta el estado del proceso cada pocos segundos ([ADR-0021](../adr/0021-app-consulta-estado-por-polling.md)).
8. Llega `AdditionalInformationRequested`. Turnos guarda su `eventId` como `requestEventId`, su mensaje y su `expiresAt`, y pasa a `AWAITING_PHONE`.
9. La app muestra el mensaje y el tiempo restante; el usuario ingresa el teléfono.
10. Turnos normaliza y valida el teléfono ([ADR-0027](../adr/0027-validacion-local-del-telefono.md)) y publica `AdditionalInformationSubmitted` con un `eventId` nuevo, el `requestEventId` guardado y la key `reservationProcessId` (REF §15.5). Pasa a `PHONE_SUBMITTED`.
11. Llega `AppointmentConfirmed`. Turnos guarda el `reservationId` y pasa a `CONFIRMED`.
12. La app muestra la reserva confirmada.

**Postcondiciones:** el proceso está `CONFIRMED`, asociado al usuario, con los datos del turno y el `reservationId` de la cátedra.

**Flujos alternativos**

- **2a. El usuario ya tiene un proceso activo:** se rechaza sin llamar al catálogo ni a la cátedra.
- **3a. El catálogo no está disponible:** no se inicia la reserva; se informa que el servicio no está disponible temporalmente.
- **3b. El profesional no existe, no está habilitado o el slot no pertenece a su agenda:** se rechaza indicando el motivo.
- **5a. La cátedra rechaza el hold** (`SLOT_ALREADY_HELD`, `SLOT_ALREADY_RESERVED`, `INVALID_SLOT`, `PROFESSIONAL_DISABLED`, `PROFESSIONAL_NOT_FOUND`): el proceso pasa a `FAILED` con el motivo, y la app lo muestra.
- **6a. La cátedra responde `HOLD_EXPIRED`:** el proceso pasa a `EXPIRED`.
- **6b. La cátedra rechaza la confirmación por otro motivo:** el proceso pasa a `FAILED` con el motivo.
- **8a. El pedido llega antes de que turnos registre el 202 del paso 6:** el proceso pasa de `HELD` a `AWAITING_PHONE`; cuando se registra el 202, no hay retroceso (el 202 no tiene transición desde `AWAITING_PHONE`).
- **10a. El teléfono tiene formato inválido:** se rechaza al usuario sin publicar; el proceso sigue en `AWAITING_PHONE`.
- **10b. El proceso no está en `AWAITING_PHONE`** (por ejemplo, ya venció): se rechaza el envío indicando el estado actual.
- **11a. Llega `AdditionalInformationRejected`:** el proceso vuelve a `AWAITING_PHONE` con el motivo y el mensaje; se repite desde el paso 9, con un `eventId` nuevo en cada envío.
- **11b. Llega `AppointmentProcessExpired`:** el proceso pasa a `EXPIRED`. Es el caso de la evidencia "reserva que expira sin completar la información" (ENUNCIADO §11).
- **11c. Llega `AppointmentProcessInvalid`:** el proceso pasa a `INVALID` con el motivo.
- **Cualquier paso. Llega un evento repetido, tardío o sin transición válida:** se ignora y se registra ([ADR-0023](../adr/0023-estados-solo-avanzan.md)).

Los timeouts, las respuestas perdidas, los eventos que no llegan y los reinicios con procesos a medias se definen en la parte de robustez.
