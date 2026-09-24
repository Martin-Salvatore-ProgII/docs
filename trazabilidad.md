# Trazabilidad

Relación entre lo que exige el enunciado, dónde lo especificamos, dónde se implementa y prueba, y cómo se demuestra. Es el checklist de aprobación: una fila sin completar es algo que falta.

**Estados:** ⬜ pendiente · 🟨 especificado · 🟦 implementado · ✅ probado y demostrable

## Alcance funcional (ENUNCIADO §5)

| § 5 | Requisito | Especificación | Spec | Tests | Estado |
| --- | --- | --- | --- | --- | --- |
| 1 | Registrar y autenticar usuarios finales | HU-01, HU-02, RNF-01, RNF-04 | | | 🟨 |
| 2 | Registrar la cuenta técnica una vez | HU-03 | | | 🟨 |
| 3 | Autenticar la integración y recuperar su configuración | HU-03, ADR-0005 | | | 🟨 |
| 4 | Obtener y guardar un snapshot completo | HU-04, CU-02, ADR-0010, ADR-0012 | | | 🟨 |
| 5 | Mantener el catálogo actualizado con Kafka y Redis | HU-04, CU-01, ADR-0006 a ADR-0009 | | | 🟨 |
| 6 | Detectar discontinuidad y reconstruir | CU-01 (3b, 4a), CU-02 | | | 🟨 |
| 7 | Buscar y filtrar con la copia local | | | | ⬜ |
| 8 | Construir y mostrar turnos disponibles | | | | ⬜ |
| 9 | Solicitar un hold | | | | ⬜ |
| 10 | Iniciar la confirmación por REST | | | | ⬜ |
| 11 | Recibir el pedido de información y responder por Kafka | | | | ⬜ |
| 12 | Reflejar el resultado final localmente | | | | ⬜ |
| 13 | Consultar las reservas propias | | | | ⬜ |
| 14 | Cancelar una reserva propia confirmada | | | | ⬜ |
| 15 | Informar errores de integración y recuperarse | Sincronización: CU-01, ADR-0011, RNF-07. Reservas: pendiente | | | 🟨 |

## Evidencias (ENUNCIADO §11)

| Evidencia | Especificación | Cómo se demuestra | Estado |
| --- | --- | --- | --- |
| Inicialización desde snapshot | CU-02 | Base vacía al arrancar ([`sincronizacion.md`](arq/sincronizacion.md#cómo-se-demuestra-enunciado-11)) | 🟨 |
| Actualización incremental | CU-01, HU-04 | Publicación de una versión con el servicio al día | 🟨 |
| Recuperación por snapshot tras una discontinuidad | CU-01 (3b), CU-02 | Servicio detenido hasta quedar por debajo de O | 🟨 |
| Búsquedas sobre datos locales | | | ⬜ |
| Filtros por categoría, nombre, habilitado y disponibilidad | | | ⬜ |
| Reserva confirmada por REST y Kafka | | | ⬜ |
| Reserva que expira sin completar la información | | | ⬜ |
| Consulta y cancelación propia sin acceso cruzado | RNF-03 | | ⬜ |
| Procesamiento idempotente de un mensaje duplicado | RNF-06, ADR-0008, CU-01 (1a) | `CatalogUpdated` reprocesado. Reservas: pendiente | 🟨 |
| Separación efectiva de datos entre servicios | Constitución P-03, ADR-0002 | | ⬜ |
| Rechazo de accesos no autenticados o no autorizados | HU-02, RNF-03, RNF-04 | | 🟨 |
| Tests automatizados de backend | | | ⬜ |

## Requisitos no funcionales

| RNF | Tema | Tests | Estado |
| --- | --- | --- | --- |
| RNF-01 | Contraseñas protegidas | | 🟨 |
| RNF-02 | Secretos en configuración externa | | 🟨 |
| RNF-03 | La identidad sale del token | | 🟨 |
| RNF-04 | Endpoints protegidos por defecto | | 🟨 |
| RNF-05 | Validación en cada límite | | 🟨 |
| RNF-06 | Procesamiento idempotente de mensajes | | 🟨 |
| RNF-07 | Reintentos acotados y estado recuperable | | 🟨 |
| RNF-08 | Lecturas consistentes durante las actualizaciones | | 🟨 |
