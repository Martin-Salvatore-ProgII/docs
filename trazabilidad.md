# Trazabilidad

Relación entre lo que exige el enunciado, dónde lo especificamos, en qué feature (`specs/`) se implementa, cómo se prueba, y cómo se demuestra. Es el checklist de aprobación: una fila sin completar es algo que falta.

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
| 7 | Buscar y filtrar con la copia local | HU-05, ADR-0013, ADR-0014 | | | 🟨 |
| 8 | Construir y mostrar turnos disponibles | HU-06, CU-03, ADR-0015 a ADR-0017 | | | 🟨 |
| 9 | Solicitar un hold | HU-07, CU-04, ADR-0019, ADR-0022 | | | 🟨 |
| 10 | Iniciar la confirmación por REST | HU-07, CU-04, ADR-0019 | | | 🟨 |
| 11 | Recibir el pedido de información y responder por Kafka | HU-07, CU-04, ADR-0020, ADR-0021, ADR-0027 | | | 🟨 |
| 12 | Reflejar el resultado final localmente | CU-04, ADR-0023, [`maquina-estados.md`](arq/maquina-estados.md) | | | 🟨 |
| 13 | Consultar las reservas propias | HU-08, ADR-0025 | | | 🟨 |
| 14 | Cancelar una reserva propia confirmada | HU-09, ADR-0026 | | | 🟨 |
| 15 | Informar errores de integración y recuperarse | Sincronización: CU-01, ADR-0011. Reservas: CU-04, CU-05, ADR-0028 a ADR-0031. RNF-07 | | | 🟨 |

## Evidencias (ENUNCIADO §11)

| Evidencia | Especificación | Cómo se demuestra | Estado |
| --- | --- | --- | --- |
| Inicialización desde snapshot | CU-02 | Base vacía al arrancar ([`sincronizacion.md`](arq/sincronizacion.md#cómo-se-demuestra-enunciado-11)) | 🟨 |
| Actualización incremental | CU-01, HU-04 | Publicación de una versión con el servicio al día | 🟨 |
| Recuperación por snapshot tras una discontinuidad | CU-01 (3b), CU-02 | Servicio detenido hasta quedar por debajo de O | 🟨 |
| Búsquedas sobre datos locales | HU-05 (7), Constitución P-06 | Búsqueda que funciona con la cátedra inaccesible | 🟨 |
| Filtros por categoría, nombre, habilitado y disponibilidad | HU-05 (1 a 4) | Cada filtro, solo y combinado, desde la app | 🟨 |
| Reserva confirmada por REST y Kafka | HU-07, CU-04 | Reserva completa desde la app, con el teléfono | 🟨 |
| Reserva que expira sin completar la información | HU-07 (6), CU-04 (11b), ADR-0020 | Reserva en la que no se ingresa el teléfono | 🟨 |
| Consulta y cancelación propia sin acceso cruzado | HU-08, HU-09, RNF-03 | Dos usuarios; uno intenta ver y cancelar la reserva del otro | 🟨 |
| Procesamiento idempotente de un mensaje duplicado | RNF-06, ADR-0008, ADR-0023, CU-01 (1a) | `CatalogUpdated` reprocesado; evento de reserva repetido | 🟨 |
| Separación efectiva de datos entre servicios | Constitución P-03, ADR-0002 | | ⬜ |
| Rechazo de accesos no autenticados o no autorizados | HU-02, HU-08, HU-09, RNF-03, RNF-04, ADR-0032 | Pedidos sin token, con token vencido o falsificado, y sobre reservas ajenas | 🟨 |
| Tests automatizados de backend | RNF-10, [`arq/pruebas.md`](arq/pruebas.md) | Ejecución de las pruebas y el historial de CI de los PR | 🟨 |

## Opcionales

| Requisito | Especificación | Spec | Tests | Estado |
| --- | --- | --- | --- | --- |
| Rol administrador (ENUNCIADO §9) | HU-10, ADR-0037, Constitución P-18 | | | 🟨 |

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
| RNF-09 | Sin supuestos fijos sobre los datos y valores configurables | | 🟨 |
| RNF-10 | Pruebas automatizadas de los backends | | 🟨 |
