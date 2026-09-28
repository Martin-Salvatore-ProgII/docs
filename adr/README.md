# Registro de decisiones de arquitectura (ADR)

Cada decisión que el enunciado deja a nuestro criterio (ENUNCIADO §10), o que tiene alternativas razonables, se registra acá. Un ADR aceptado no se edita para cambiar la decisión: se escribe uno nuevo que lo reemplaza y el viejo pasa a **Reemplazado**.

## Índice

| ADR | Decisión | Estado |
| --- | --- | --- |
| [0001](0001-app-habla-con-ambos-servicios.md) | La app KMP se comunica directamente con los dos servicios | Aceptado |
| [0002](0002-comunicacion-unidireccional-turnos-catalogo.md) | Comunicación entre servicios en una sola dirección: turnos → catálogo | Aceptado |
| [0003](0003-turnos-emite-jwt-usuarios.md) | El servicio de turnos registra a los usuarios finales y emite su JWT | Aceptado |
| [0004](0004-external-patient-id-uuid.md) | `externalPatientId` es un UUID propio de cada usuario | Aceptado |
| [0005](0005-configuracion-integracion-al-arrancar.md) | Los backends obtienen la configuración de la integración al arrancar | Aceptado |
| [0006](0006-disparadores-sincronizacion.md) | La sincronización se dispara por Kafka, al arrancar y con un chequeo periódico | Aceptado |
| [0007](0007-concurrencia-optimista-version-local.md) | Sincronizaciones simultáneas: una versión solo se aplica sobre la versión local esperada | Aceptado |
| [0008](0008-deduplicacion-por-event-id.md) | Los eventos Kafka procesados se registran por `eventId` | Aceptado |
| [0009](0009-referencias-faltantes-desde-redis.md) | Las entidades referenciadas que faltan se traen del estado actual en Redis | Aceptado |
| [0010](0010-snapshot-en-una-transaccion.md) | El snapshot se aplica en una sola transacción y la app avisa que el catálogo se está actualizando | Aceptado |
| [0011](0011-fallas-y-estado-de-sincronizacion.md) | Fallas de sincronización sin reintentos en loop y estado expuesto por el servicio | Aceptado |
| [0012](0012-postgresql.md) | PostgreSQL como motor de base de datos de los dos servicios | Aceptado |
| [0013](0013-filtro-disponibilidad-de-agenda.md) | El filtro de disponibilidad de la búsqueda es de agenda: fecha y franja horaria opcional | Aceptado |
| [0014](0014-habilitado-efectivo.md) | Un profesional está habilitado solo si él y su categoría lo están; la búsqueda muestra habilitados por defecto | Aceptado |
| [0015](0015-reglas-disponibilidad-turnos.md) | Reglas de la disponibilidad de turnos: una fecha, horizonte acotado, solo slots completos y futuros | Aceptado |
| [0016](0016-zona-horaria-argentina.md) | La agenda usa la zona horaria fija de Argentina | Aceptado |
| [0017](0017-operacion-agenda-vigente.md) | El catálogo expone la agenda vigente de un profesional en una sola operación | Aceptado |
| [0018](0018-usuario-compatible-sin-generador-jhipster.md) | El usuario se implementa a mano dentro de la hexagonal, compatible con JHipster, sin usar su generador | Aceptado (pendiente de confirmación de la cátedra) |
| [0019](0019-hold-y-confirmacion-una-accion.md) | Hold y confirmación inicial son una sola acción del usuario | Aceptado |
| [0020](0020-telefono-a-pedido.md) | El teléfono se pide cuando llega el pedido de la cátedra | Aceptado |
| [0021](0021-app-consulta-estado-por-polling.md) | La app sigue el proceso de reserva consultando su estado | Aceptado |
| [0022](0022-proceso-guardado-antes-de-llamar.md) | El proceso se guarda antes de llamar a la cátedra y se actualiza antes de confirmar | Aceptado |
| [0023](0023-estados-solo-avanzan.md) | Máquina de estados de la reserva: los estados solo avanzan y los finales no se reabren | Aceptado |
| [0024](0024-un-proceso-activo-por-usuario.md) | Un usuario tiene como máximo un proceso de reserva activo | Aceptado |
| [0025](0025-reservas-propias-desde-base-local.md) | Las reservas propias se consultan desde la base local de turnos | Aceptado |
| [0026](0026-cancelacion-con-respuesta-rest.md) | La cancelación se registra con la respuesta REST de la cátedra | Aceptado |
| [0027](0027-validacion-local-del-telefono.md) | El teléfono se valida con la regla de la cátedra antes de publicarlo | Aceptado |
| [0028](0028-timeout-al-crear-hold.md) | Un timeout al crear el hold cierra el proceso como fallido, sin reintentar | Aceptado |
| [0029](0029-reintentos-acotados-por-operacion.md) | Reintentos acotados solo en las operaciones seguras de repetir, con timeouts configurables | Aceptado |
| [0030](0030-reconciliacion-periodica-de-procesos.md) | Reconciliación periódica de los procesos de reserva colgados | Aceptado |
| [0031](0031-eventos-kafka-problematicos.md) | Los eventos Kafka que no se pueden procesar no bloquean el consumo | Aceptado |
| [0032](0032-jwt-firmado-con-par-de-claves.md) | El JWT de usuario se firma con un par de claves: turnos firma y el catálogo solo valida | Aceptado |
| [0033](0033-vigencia-jwt-usuario.md) | Vigencia del JWT de usuario: 24 horas, o 30 días con `rememberMe`, sin refresh tokens | Aceptado |
| [0034](0034-propagacion-jwt-usuario-entre-servicios.md) | Turnos se autentica ante el catálogo propagando el JWT del usuario | Aceptado |
| [0035](0035-https-entre-app-y-backends.md) | HTTPS entre la app y los backends, con certificado autofirmado en el entorno local | Aceptado |
| [0036](0036-cors-cerrado.md) | CORS cerrado por defecto: ningún origen web permitido | Aceptado |

## Plantilla

```markdown
# ADR-NNNN. Título en forma de decisión

- **Estado:** Propuesto | Aceptado | Reemplazado por ADR-NNNN
- **Fecha:** yyyy-MM-dd

## Contexto

Qué problema o necesidad obliga a decidir. Citar ENUNCIADO o REF.

## Decisión

Qué se decidió, en una o dos frases.

## Alternativas consideradas

- **Alternativa.** Por qué se descartó.

## Consecuencias

Qué implica la decisión: lo que facilita, lo que cuesta y lo que obliga a hacer en otros lados.
```
