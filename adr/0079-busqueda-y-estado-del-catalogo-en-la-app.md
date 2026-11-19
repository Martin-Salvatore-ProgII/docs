# ADR-0079. Búsqueda, estado del catálogo y pantalla del profesional en la app

- **Estado:** Aceptado
- **Fecha:** 2026-11-18

## Contexto

La pantalla Buscar ofrece los cuatro filtros de HU-05 sobre `GET /api/professionals`, y la pantalla Profesional muestra la agenda vigente ([ADR-0017](0017-operacion-agenda-vigente.md)) y deja elegir una fecha para ver los turnos. [`arq/interfaz.md`](../arq/interfaz.md) dice qué permite cada una, pero no:

- cuándo se ejecuta la búsqueda;
- cómo se elige la franja horaria;
- qué fechas deja elegir el calendario;
- cuándo se consulta el estado de sincronización ni dónde se ve.

El rango de fechas del filtro va de hoy al horizonte ([ADR-0065](0065-reglas-entrada-filtro-disponibilidad.md)), un valor que configura el backend ([RNF-09](../requisitos/no-funcionales.md#rnf-09-sin-supuestos-fijos-sobre-los-datos-de-la-cátedra-y-valores-operativos-configurables)) y que ningún contrato informa.

## Decisión

**Los filtros se van sumando.** La app es solo para el celular ([ADR-0072](0072-app-solo-android.md)), así que Buscar arranca con poco: un buscador grande por nombre y una fila de chips (Categoría, Fecha, Habilitados). Cada chip abre una hoja inferior con opciones de tamaño cómodo para el dedo. Un filtro elegido queda como chip marcado, con su valor y una cruz para quitarlo. El chip de la franja aparece recién cuando hay una fecha, porque sin fecha la franja no es válida.

**La búsqueda se ejecuta sola.** Al entrar a Buscar se busca sin filtros. Después, cada cambio de un filtro vuelve a buscar desde la primera página, sin un botón "Buscar". El nombre espera 300 ms sin cambios antes de buscar, para no mandar un pedido por cada letra. Cada búsqueda nueva cancela la anterior que siga pendiente, así una respuesta vieja nunca pisa a una nueva.

**La franja horaria tiene opciones fijas y una personalizada.**

| Opción | Franja |
| --- | --- |
| Mañana | 00:00 a 12:00 |
| Tarde | 12:00 a 18:00 |
| Noche | 18:00 a 23:59 |
| Personalizada | Abre los selectores Desde y Hasta |

Estos límites son una preferencia de la interfaz, no un valor del backend. La app valida antes de buscar lo que exige ADR-0065: que la franja tenga fecha y que su inicio sea anterior al fin.

**El calendario del filtro bloquea solo los días pasados.** La app no conoce el horizonte ([Constitución P-21](../constitucion.md)). Si la fecha queda más allá, el catálogo responde `400 DATE_OUT_OF_RANGE` y la app muestra su mensaje propio.

**El filtro de estado tiene dos opciones**, "Habilitados" (por defecto) y "No habilitados", porque el contrato no tiene "todos" ([ADR-0014](0014-habilitado-efectivo.md)). Un profesional no habilitado se muestra marcado como no disponible para reservar.

**El estado del catálogo se consulta con cada búsqueda y la app solo avisa.** No hay consulta periódica. Si `syncInProgress` es verdadero, Buscar muestra un aviso corto y no bloqueante, "Actualizando el catálogo…" ([ADR-0010](0010-snapshot-en-una-transaccion.md)). La app no muestra la versión, la última sincronización ni el último error: son datos técnicos que no le sirven al usuario para buscar, y `lastError` es un texto interno del servicio. El resto del estado (HU-04 criterio 5) lo informa `GET /api/catalog/sync-status` a cualquier usuario autenticado, y con eso se muestra en la demo. Ante `503 CATALOG_NOT_READY`, Buscar muestra que el catálogo todavía se está cargando y permite reintentar.

**Pantalla Profesional:**
- Muestra los días y horarios en que atiende, sin la duración de los turnos: se ve al elegir uno.
- Ofrece los próximos días en que atiende como accesos directos a Turnos disponibles. Si se buscó con una fecha, ese día va primero y marcado. Los calcula la app con los días de la semana de la agenda y la fecha de hoy.
- "Elegir otra fecha" abre un calendario que deja elegir solo días que no pasaron y en los que el profesional atiende.
- Un profesional no habilitado se ve con un aviso y sin accesos a turnos. Uno habilitado no lleva ninguna marca.

## Alternativas consideradas

- **Botón "Buscar".** Descartada: buscar mientras se escribe encuentra a "Martín" con "mar". Cancelar la búsqueda anterior y esperar a que se deje de escribir cuesta pocas líneas en el ViewModel.
- **Solo selectores Desde y Hasta.** Descartada: para el caso común (a la mañana, a la tarde) son más pasos. Se mantienen como opción personalizada, porque el contrato admite cualquier franja.
- **Solo franjas fijas.** Descartada: no permite buscar "de 10 a 11", algo que el contrato admite.
- **Limitar el calendario con el horizonte escrito en la app.** Descartada por [P-21](../constitucion.md): el mismo valor quedaría en tres lugares.
- **Que el catálogo informe el horizonte, por ejemplo una fecha máxima en el estado de sincronización.** Pospuesta: es la forma de limitar el calendario cumpliendo P-21, pero cambia un contrato y un servicio ya aprobados cerca de la entrega. Queda como mejora posible.
- **Consultar el estado de sincronización cada pocos segundos.** Descartada: suma tráfico continuo para un aviso que alcanza con ver al buscar.
- **Un detalle del estado del catálogo en la app (versión, última sincronización, último error).** Descartada: al usuario le alcanza con saber que el catálogo se está actualizando, y el detalle ya está en `sync-status`.
- **Todos los filtros a la vista, como un formulario.** Descartada: en la pantalla de un celular deja poco espacio para la lista y para tocar con el dedo.

## Consecuencias

- Cada cambio de filtro es un pedido al catálogo. Como responde con su copia local (Constitución P-06), no genera tráfico hacia la cátedra.
- Un usuario puede elegir una fecha más allá del horizonte y recibir un error al buscar.
- Un profesional puede aparecer con una franja y no tener turnos libres en ella: es disponibilidad de agenda ([ADR-0013](0013-filtro-disponibilidad-de-agenda.md)).
- En la demo de la sincronización incremental (ENUNCIADO §11), la versión local se muestra con `sync-status`, no desde la app.
- Un acceso directo de Profesional puede caer más allá del horizonte. En ese caso Turnos disponibles muestra el error de fecha fuera de rango, como en la búsqueda.
