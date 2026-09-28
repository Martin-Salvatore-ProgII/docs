# Idempotencia y recuperación

Cómo se comporta el sistema cuando algo se repite, se pierde, llega fuera de orden, se corta o se reinicia (ENUNCIADO §8, §12; REF §18). Esta página junta el criterio general y enlaza cada mecanismo con el documento donde está decidido.

## Principios

1. **La cátedra es la fuente de verdad.** El estado local se puede perder o atrasar; siempre hay un camino para volver a consultarlo (Redis o snapshot para el catálogo, `GET /api/appointments` para las reservas).
2. **Todo lo que llega por Kafka puede llegar repetido.** `eventId` es la clave de idempotencia y cada servicio registra los que procesó ([Constitución P-14](../constitucion.md), [ADR-0008](../adr/0008-deduplicacion-por-event-id.md)).
3. **Los avisos no son la fuente de la decisión.** En el catálogo, la decisión se toma comparando versiones con Redis, no con el número del aviso ([Constitución P-15](../constitucion.md)).
4. **Los estados solo avanzan.** Un proceso de reserva nunca retrocede ni sale de un estado final ([Constitución P-17](../constitucion.md), [`maquina-estados.md`](maquina-estados.md)).
5. **Un timeout no prueba que la operación falló.** Solo se repite lo que es seguro repetir; lo demás se reconcilia (REF §18.4, [ADR-0029](../adr/0029-reintentos-acotados-por-operacion.md)).
6. **Ningún reintento es ilimitado.** Cuando se agotan, el estado queda consistente y un mecanismo periódico lo retoma ([Constitución P-16](../constitucion.md)).

## Situaciones y mecanismos

### Catálogo

| Situación | Qué pasa | Dónde |
| --- | --- | --- |
| `CatalogUpdated` duplicado | Se reconoce por `eventId` y se ignora; aunque no se reconociera, la versión local ya estaría al día | [ADR-0008](../adr/0008-deduplicacion-por-event-id.md), [CU-01](../requisitos/CU.md#cu-01-sincronizar-el-catálogo) |
| Aviso perdido | El próximo aviso o el chequeo periódico comparan con Redis y ponen al día la copia | [ADR-0006](../adr/0006-disparadores-sincronizacion.md) |
| Avisos fuera de orden | No importa: la decisión sale de Redis y las versiones se aplican en orden | [`sincronizacion.md`](sincronizacion.md) |
| Historial faltante o versión fuera de la ventana | Snapshot | [`sincronizacion.md`](sincronizacion.md), [CU-02](../requisitos/CU.md#cu-02-reconstruir-el-catálogo-desde-un-snapshot) |
| Caída a mitad de una versión | La versión local no avanzó; se vuelve a aplicar entera | [`sincronizacion.md`](sincronizacion.md) |
| Dos sincronizaciones a la vez | Solo aplica la que encuentra la versión local esperada | [ADR-0007](../adr/0007-concurrencia-optimista-version-local.md) |
| Redis o la API de la cátedra no responden | Se registra el error, sin reintentos en loop; se reintenta en el próximo chequeo | [ADR-0011](../adr/0011-fallas-y-estado-de-sincronizacion.md) |
| Reinicio | Al arrancar se ejecuta un ciclo de sincronización | [ADR-0006](../adr/0006-disparadores-sincronizacion.md) |

### Reservas

| Situación | Qué pasa | Dónde |
| --- | --- | --- |
| Evento del topic de acciones duplicado | Se reconoce por `eventId` y se ignora | [ADR-0008](../adr/0008-deduplicacion-por-event-id.md) |
| Evento tardío o fuera de orden | Si no es un avance válido desde el estado actual, se ignora y se registra | [ADR-0023](../adr/0023-estados-solo-avanzan.md) |
| El pedido de teléfono llega antes que la respuesta de la confirmación | El proceso ya estaba guardado con su `reservationProcessId`; avanza igual | [ADR-0022](../adr/0022-proceso-guardado-antes-de-llamar.md) |
| Timeout al crear el hold | `FAILED`, sin reintentar; el hold huérfano vence solo | [ADR-0028](../adr/0028-timeout-al-crear-hold.md) |
| Timeout al confirmar, cancelar o publicar el teléfono | Reintento acotado; la cátedra reconoce lo ya hecho | [ADR-0029](../adr/0029-reintentos-acotados-por-operacion.md) |
| Evento final perdido | La reconciliación consulta la cátedra y cierra el proceso | [ADR-0030](../adr/0030-reconciliacion-periodica-de-procesos.md), [CU-05](../requisitos/CU.md#cu-05-reconciliar-procesos-de-reserva) |
| Vencimiento sin teléfono enviado | La reconciliación lo cierra como vencido sin consultar | [ADR-0030](../adr/0030-reconciliacion-periodica-de-procesos.md) |
| Evento inválido, desconocido o de un proceso inexistente | Se registra, se confirma el offset y se sigue | [ADR-0031](../adr/0031-eventos-kafka-problematicos.md) |
| La base falla al procesar un evento | Reintentos acotados; después se sigue y la reconciliación corrige | [ADR-0031](../adr/0031-eventos-kafka-problematicos.md) |
| Doble toque en "Reservar" | Un solo proceso activo por usuario | [ADR-0024](../adr/0024-un-proceso-activo-por-usuario.md) |
| Cancelar dos veces | Idempotente: responde el estado cancelado | [ADR-0026](../adr/0026-cancelacion-con-respuesta-rest.md) |
| Reinicio con procesos a medias | La reconciliación corre al arrancar | [ADR-0030](../adr/0030-reconciliacion-periodica-de-procesos.md) |
| El catálogo no responde | No se inician operaciones que necesiten datos vigentes; las reservas propias se siguen consultando | [ADR-0025](../adr/0025-reservas-propias-desde-base-local.md), [`arquitectura.md`](arquitectura.md) |

## Cómo se demuestra

| Evidencia (ENUNCIADO §11) | Escenario |
| --- | --- |
| Mensaje duplicado | Reprocesar un `CatalogUpdated` o un evento de reserva ya procesado: se ignora y el estado no cambia |
| Recuperación tras una discontinuidad | Ver [`sincronizacion.md`](sincronizacion.md#cómo-se-demuestra-enunciado-11) |
| Reserva que expira | No ingresar el teléfono: el proceso termina en `EXPIRED`, por el evento de la cátedra o por la reconciliación |
