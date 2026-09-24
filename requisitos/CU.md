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
