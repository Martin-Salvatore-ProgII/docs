# Glosario

Términos del dominio y de la integración, con el significado exacto que tienen en este proyecto. Los nombres técnicos se escriben como en el contrato de la cátedra.

## Piezas del sistema

**Servicio de la cátedra** (o servicio central). Instancia administrada por la cátedra que es la **fuente autoritativa** del catálogo y el registro oficial de holds y reservas. Se accede por REST, Redis y Kafka. No se ejecuta localmente. *(ENUNCIADO §1; REF §3)*

**Catálogo.** Conjunto de categorías de profesionales (`professionalCategories`), profesionales (`professionals`) y horarios semanales de atención (`weeklySchedules`). Lo publica la cátedra y puede cambiar en cualquier momento. *(ENUNCIADO §1; REF §7)*

**Copia local del catálogo.** Réplica del catálogo que guarda el servicio de catálogo en su propia base. Es la que se usa para las búsquedas y para armar la disponibilidad. Se mantiene al día por sincronización. *(ENUNCIADO §4.1)*

**Servicio de catálogo** (`catalogo-service`). Backend dueño de la copia local del catálogo y de su sincronización. Responde las búsquedas de profesionales y es la única fuente vigente de profesionales y horarios para el resto del sistema. *(ENUNCIADO §4.1)*

**Servicio de turnos** (`turnos-service`). Backend dueño de los procesos de reserva y de las reservas de los usuarios finales. Arma la disponibilidad, habla con la cátedra para reservar y cancelar, y sabe a qué usuario pertenece cada reserva. *(ENUNCIADO §4.2)*

**App KMP** (`app-kmp`). Aplicación Android desarrollada con Kotlin Multiplatform. Es la interfaz del usuario final para todo el flujo funcional. Se ejecuta fuera de Docker Compose. *(ENUNCIADO §3.2)*

## Identidades

**Cuenta técnica.** Única cuenta del proyecto ante la cátedra. La registra a mano el responsable del proyecto y la usan los dos backends para todo acceso a REST, Redis y Kafka de la cátedra. No es un usuario de la app. *(REF §2, §5)*

**JWT técnico.** Token que emite la cátedra para la cuenta técnica. Dura un año y tiene el rol `ROLE_STUDENT_CLIENT`. Es secreto: nunca sale de los backends. *(REF §4, §5.1)*

**`groupId`.** Identificador de la integración técnica ante la cátedra. A pesar del nombre, no representa un grupo de personas ni un usuario final. Aísla los topics, el namespace de Redis y las reservas del proyecto. Se normaliza a minúsculas y guiones (`Proyecto Juan Pérez` → `proyecto-juan-perez`). *(REF §2.1, §5.1)*

**Objeto `integration`.** Configuración de conexión que la cátedra asigna a la cuenta técnica: Redis, Kafka, topics, consumer group y `provisioningStatus`. *(REF §5.1, §5.3)*

**`provisioningStatus`.** Estado de aprovisionamiento de la integración: `PENDING`, `PROVISIONED`, `FAILED` o `REVOKED`. Solo con `PROVISIONED` se entrega la contraseña de Redis. *(REF §5.1)*

**Usuario final.** Persona que se registra e inicia sesión en la app. Vive en el servicio de turnos y la cátedra no lo conoce. *(ENUNCIADO §3.2; REF §2)*

**JWT de usuario.** Token que emite el servicio de turnos al usuario final cuando inicia sesión. La app lo usa en las llamadas a nuestros dos servicios. Es independiente del JWT técnico. *(ENUNCIADO §3.2)*

**`externalPatientId`.** Identificador del usuario final que se envía a la cátedra al crear y confirmar un hold. En este proyecto es un UUID propio de cada usuario. *(REF §2, §9; [ADR-0004](adr/0004-external-patient-id-uuid.md))*

## Sincronización

**Versión del catálogo.** Número entero que identifica cada publicación del catálogo de la cátedra. Solo crece. *(REF §14.2)*

**Versión local.** Versión del catálogo que refleja exactamente la copia local. Solo avanza cuando los datos de esa versión quedaron guardados por completo. *(ENUNCIADO §6; [`arq/sincronizacion.md`](arq/sincronizacion.md))*

**Versión actual (C).** Última versión publicada por la cátedra, en `catedra:sync:current-version`. *(REF §14.2)*

**Versión más vieja disponible (O).** Versión más antigua desde la que se puede avanzar de forma incremental, en `catedra:sync:oldest-available-version`. Si la versión local es menor, hace falta un snapshot. *(REF §14.2, §18.2)*

**Snapshot.** Catálogo completo (habilitadas y deshabilitadas) obtenido por `GET /api/synchronization/snapshot`, junto con la versión a la que corresponde (`snapshotVersion`). *(REF §7)*

**Sincronización completa.** Reemplazo de toda la copia local por un snapshot. *(ENUNCIADO §6.1)*

**Sincronización incremental.** Aplicación, en orden y de a una, de las versiones posteriores a la local, leyendo de Redis los IDs afectados y el estado actual de esas entidades. *(ENUNCIADO §6.2; REF §14.5)*

**Discontinuidad.** Situación en la que no se puede avanzar de forma incremental con seguridad: falta historial, la versión local quedó fuera de la ventana disponible o hay datos que no se pueden verificar. Se resuelve con un snapshot. *(ENUNCIADO §6.2)*

**`CatalogUpdated`.** Evento Kafka que avisa que hay una versión nueva del catálogo. Es solo un aviso: no trae datos. *(REF §15.3)*

**Baja lógica.** Forma en que la cátedra da de baja una entidad del catálogo: sigue existiendo con `enabled: false`. *(REF §14.4)*

**`eventId`.** Identificador único de cada evento Kafka y clave funcional de idempotencia: un `eventId` ya procesado no se vuelve a aplicar. *(REF §15.1)*

## Búsqueda y disponibilidad

**Habilitado (efectivo).** Un profesional está habilitado solo si él y su categoría tienen `enabled: true`. Es el criterio que usan la búsqueda, la agenda vigente y la disponibilidad. *([ADR-0014](adr/0014-habilitado-efectivo.md))*

**Disponibilidad de agenda.** Filtro de la búsqueda: el profesional tiene algún horario semanal habilitado para el día de la semana de una fecha y, opcionalmente, dentro de una franja horaria. Se resuelve con la copia local y no garantiza turnos libres. *([ADR-0013](adr/0013-filtro-disponibilidad-de-agenda.md))*

**Agenda vigente.** Datos actuales de un profesional con sus horarios semanales habilitados, tal como los informa el servicio de catálogo en una sola lectura. *([ADR-0017](adr/0017-operacion-agenda-vigente.md))*

**Slot.** Turno posible de una fecha: un inicio y un fin que surgen de dividir un horario semanal en intervalos de `slotDurationMinutes`. Solo cuenta si termina dentro del horario. *(REF §7; [ADR-0015](adr/0015-reglas-disponibilidad-turnos.md))*

**Ocupación.** Slot que la cátedra informa como no disponible: `CONFIRMED` (reservado) o `HELD` (bloqueado temporalmente, con `expiresAt`). Si un slot está en los dos estados, prevalece `CONFIRMED`. *(REF §8)*

**Disponibilidad de turnos.** Slots libres de un profesional en una fecha: los de su agenda vigente menos las ocupaciones, sin los que ya pasaron. La calcula el servicio de turnos en cada consulta y no garantiza la reserva. *(ENUNCIADO §7; [CU-03](requisitos/CU.md#cu-03-consultar-la-disponibilidad-de-turnos))*

**Hora de Argentina.** Zona horaria fija `America/Argentina/Buenos_Aires`, con la que se interpretan las horas de la agenda y se determinan "hoy" y "ahora". *([ADR-0016](adr/0016-zona-horaria-argentina.md))*

## Reserva

**Hold.** Bloqueo temporal de un slot en la cátedra, a nombre del proyecto, para que nadie más lo tome mientras se completa la reserva. Vence en `expiresAt`; guardarlo localmente no lo extiende. *(REF §9)*

**`holdId`.** Identificador del hold en la cátedra. Se usa para confirmarlo. *(REF §9, §10)*

**Proceso de reserva.** Todo el recorrido de una reserva, desde que el usuario toca "Reservar" hasta su resultado final y una eventual cancelación. Localmente, cada proceso tiene su propio id, su dueño y un estado de la [máquina de estados](arq/maquina-estados.md). *(ENUNCIADO §4.2, §7)*

**`reservationProcessId`.** Identificador del proceso en la cátedra. Lo devuelve el hold y es la key de todos los mensajes Kafka de ese proceso. *(REF §9, §15.1)*

**`reservationId`.** Identificador de la reserva confirmada en la cátedra. Llega con `AppointmentConfirmed`. *(REF §11, §15.6)*

**Confirmación inicial.** Paso REST que acepta el hold y dispara el pedido de teléfono. No confirma la reserva: la deja `PHONE_PENDING` en la cátedra y `AWAITING_REQUEST` localmente. *(REF §10)*

**Pedido de teléfono** (`AdditionalInformationRequested`). Evento con el que la cátedra pide el teléfono. Su `eventId` se guarda y se devuelve como `requestEventId`. *(REF §15.4)*

**`requestEventId`.** El `eventId` del pedido de teléfono, devuelto en la respuesta. Si no coincide, la cátedra invalida el proceso (`REQUEST_EVENT_MISMATCH`). *(REF §15.5, §15.9)*

**Estado final.** Estado del que un proceso ya no sale: `CANCELLED`, `EXPIRED`, `INVALID` y `FAILED`. `CONFIRMED` también es final, salvo por la cancelación. *([`arq/maquina-estados.md`](arq/maquina-estados.md))*

**Proceso activo.** Proceso de reserva en un estado no final. Un usuario puede tener como máximo uno. *([ADR-0024](adr/0024-un-proceso-activo-por-usuario.md))*

## Robustez

**Reconciliación.** Consulta periódica a la cátedra para resolver procesos de reserva que quedaron sin resultado (por eventos perdidos, reinicios o reintentos agotados). Es la red de seguridad del flujo de reserva. *([CU-05](requisitos/CU.md#cu-05-reconciliar-procesos-de-reserva), [ADR-0030](adr/0030-reconciliacion-periodica-de-procesos.md))*

**Operación segura de repetir.** Operación con efectos que se puede reintentar sin duplicarlos, porque la cátedra la reconoce como ya hecha (confirmación inicial, cancelación, publicación con el mismo `eventId`). Crear un hold no lo es. *([ADR-0029](adr/0029-reintentos-acotados-por-operacion.md))*

**Hold huérfano.** Hold que la cátedra creó pero cuya respuesta nunca llegó. Vence solo en su `expiresAt`. *([ADR-0028](adr/0028-timeout-al-crear-hold.md))*
