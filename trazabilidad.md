# Trazabilidad

Relación entre lo que exige el enunciado, dónde lo especificamos, en qué milestone se implementa, cómo se prueba y cómo se demuestra. Es el checklist de aprobación: una fila sin completar es algo que falta.

No hay fechas: el trabajo avanza por milestones chicas, en el orden de la tabla siguiente, que sale de las dependencias entre las piezas. Cada milestone es una milestone de GitHub en el repo indicado; sus issues detallan las tareas.

**Estados:** ⬜ pendiente · 🟨 especificado · 🟦 implementado y probado, en revisión · ✅ aprobado (en `main`, revisado por el profesor)

Una fila marcada como *parcial* tiene una parte hecha y otra planificada en una milestone posterior; la columna de milestone indica cuáles.

## Orden de implementación

| # | Repo | Milestone | Qué deja funcionando | Depende de | Estado |
| --- | --- | --- | --- | --- | --- |
| 1 | turnos | M1 Esqueleto hexagonal | Proyecto, paquetes de la hexagonal con su test de arquitectura, PostgreSQL con Flyway, Docker Compose del sistema completo, manejo de errores, CI | — | ✅ |
| 2 | catálogo | M1 Esqueleto hexagonal | Lo mismo que el anterior, para el catálogo | — | ✅ |
| 3 | turnos | M2 Usuario y Auth | Registro, inicio de sesión con JWT, endpoints protegidos, administrador inicial | 1 | 🟦 |
| 4 | catálogo | M2 Endpoint categorías | Validación del JWT de turnos, endpoints protegidos, tablas del catálogo, listado de categorías | 2, 3 | 🟦 |
| 5 | catálogo | M3 Sincronización por snapshot | Configuración de la integración, copia local completa desde el snapshot, estado de sincronización | 4 | 🟨 |
| 6 | catálogo | M4 Búsqueda de profesionales y agenda | Los cuatro filtros de búsqueda y la agenda vigente de un profesional | 5 | 🟨 |
| 7 | turnos | M3 Disponibilidad de turnos | Configuración de la integración, clientes del catálogo y de ocupaciones, cálculo de la disponibilidad | 3, 6 | 🟨 |
| 8 | turnos | M4 Inicio de reserva | Proceso de reserva, hold y confirmación inicial por REST | 7 | 🟨 |
| 9 | catálogo | M5 Sincronización incremental | Redis, aviso `CatalogUpdated` por Kafka, discontinuidad, duplicados, chequeo periódico | 5 | 🟨 |
| 10 | turnos | M5 Reserva por Kafka | Pedido y envío del teléfono, eventos terminales, reconciliación de procesos | 8 | 🟨 |
| 11 | turnos | M6 Mis reservas | Consulta y cancelación de las reservas propias | 10 | 🟨 |
| 12 | app-kmp | M1 a M4 | Registro e inicio de sesión, búsqueda, disponibilidad y reserva, mis reservas | 3, 6, 7, 10, 11 | 🟨 |
| 13 | turnos y catálogo | Canal seguro | HTTPS entre la app y los backends (ADR-0035) | 12 | 🟨 |
| 14 | turnos | Administración (opcional) | Listar, ver y cancelar reservas de cualquier usuario (HU-10) | 11 | 🟨 |

Las milestones 5 a 8 ya tienen sus issues creadas en GitHub. Las demás se detallan al llegar a ellas. La app puede empezar antes de que los backends terminen: cada una de sus milestones necesita solo los endpoints que usa.

## Alcance funcional (ENUNCIADO §5)

| § 5 | Requisito | Especificación | Milestone | Tests | Estado |
| --- | --- | --- | --- | --- | --- |
| 1 | Registrar y autenticar usuarios finales | HU-01, HU-02, RNF-01, RNF-04 | Turnos M2 | `UserAuthenticationFlowTest`, `UserControllerTest`, `RegisterUserUseCaseImplTest`, `AuthenticateUserUseCaseImplTest` | 🟦 |
| 2 | Registrar la cuenta técnica una vez | HU-03 | Hecho a mano (REF §5.1) | No aplica | ✅ |
| 3 | Autenticar la integración y recuperar su configuración | HU-03, ADR-0005 | Catálogo M3, Turnos M3 | | 🟨 |
| 4 | Obtener y guardar un snapshot completo | HU-04, CU-02, ADR-0010, ADR-0012 | Catálogo M3. Tablas: Catálogo M2 | `DatabaseMigrationTest` (catálogo) | 🟨 |
| 5 | Mantener el catálogo actualizado con Kafka y Redis | HU-04, CU-01, ADR-0006 a ADR-0009 | Catálogo M5 | | 🟨 |
| 6 | Detectar discontinuidad y reconstruir | CU-01 (3b, 4a), CU-02 | Catálogo M5 | | 🟨 |
| 7 | Buscar y filtrar con la copia local | HU-05, ADR-0013, ADR-0014 | Catálogo M4. Categorías: Catálogo M2 | `CategoryAccessTest`, `JpaCategoryRepositoryAdapterTest` | 🟨 parcial |
| 8 | Construir y mostrar turnos disponibles | HU-06, CU-03, ADR-0015 a ADR-0017 | Turnos M3. Agenda: Catálogo M4 | | 🟨 |
| 9 | Solicitar un hold | HU-07, CU-04, ADR-0019, ADR-0022 | Turnos M4 | | 🟨 |
| 10 | Iniciar la confirmación por REST | HU-07, CU-04, ADR-0019 | Turnos M4 | | 🟨 |
| 11 | Recibir el pedido de información y responder por Kafka | HU-07, CU-04, ADR-0020, ADR-0021, ADR-0027 | Turnos M5 | | 🟨 |
| 12 | Reflejar el resultado final localmente | CU-04, ADR-0023, [`maquina-estados.md`](arq/maquina-estados.md) | Turnos M5 | | 🟨 |
| 13 | Consultar las reservas propias | HU-08, ADR-0025 | Turnos M6 | | 🟨 |
| 14 | Cancelar una reserva propia confirmada | HU-09, ADR-0026 | Turnos M6 | | 🟨 |
| 15 | Informar errores de integración y recuperarse | Sincronización: CU-01, ADR-0011. Reservas: CU-04, CU-05, ADR-0028 a ADR-0031. RNF-07 | Catálogo M3 y M5, Turnos M3 a M5 | | 🟨 |

## Evidencias (ENUNCIADO §11)

| Evidencia | Especificación | Cómo se demuestra | Milestone | Estado |
| --- | --- | --- | --- | --- |
| Inicialización desde snapshot | CU-02 | Base vacía al arrancar ([`sincronizacion.md`](arq/sincronizacion.md#cómo-se-demuestra-enunciado-11)) | Catálogo M3 | 🟨 |
| Actualización incremental | CU-01, HU-04 | Publicación de una versión con el servicio al día | Catálogo M5 | 🟨 |
| Recuperación por snapshot tras una discontinuidad | CU-01 (3b), CU-02 | Servicio detenido hasta quedar por debajo de O | Catálogo M5 | 🟨 |
| Búsquedas sobre datos locales | HU-05 (7), Constitución P-06 | Búsqueda que funciona con la cátedra inaccesible | Catálogo M4 | 🟨 |
| Filtros por categoría, nombre, habilitado y disponibilidad | HU-05 (1 a 4) | Cada filtro, solo y combinado, desde la app | Catálogo M4, app | 🟨 |
| Reserva confirmada por REST y Kafka | HU-07, CU-04 | Reserva completa desde la app, con el teléfono | Turnos M4 y M5, app | 🟨 |
| Reserva que expira sin completar la información | HU-07 (6), CU-04 (11b), ADR-0020 | Reserva en la que no se ingresa el teléfono | Turnos M5, app | 🟨 |
| Consulta y cancelación propia sin acceso cruzado | HU-08, HU-09, RNF-03 | Dos usuarios; uno intenta ver y cancelar la reserva del otro | Turnos M6, app | 🟨 |
| Procesamiento idempotente de un mensaje duplicado | RNF-06, ADR-0008, ADR-0023, CU-01 (1a) | `CatalogUpdated` reprocesado; evento de reserva repetido | Catálogo M5, Turnos M5 | 🟨 |
| Separación efectiva de datos entre servicios | Constitución P-03, ADR-0002, ADR-0041 | Cada servicio con su propio contenedor de PostgreSQL, base, usuario y volumen; el usuario de turnos no existe en la base del catálogo | Turnos M1, Catálogo M1 | ✅ |
| Rechazo de accesos no autenticados o no autorizados | HU-02, HU-08, HU-09, RNF-03, RNF-04, ADR-0032 | Pedidos sin token, con token vencido o falsificado, de un usuario común a rutas administrativas, y sobre reservas ajenas | Turnos M2, Catálogo M2. Reservas ajenas: Turnos M6 | 🟦 parcial |
| Tests automatizados de backend | RNF-10, [`arq/pruebas.md`](arq/pruebas.md) | Ejecución de las pruebas y el historial de CI de los PR | Todas | 🟦 parcial |

## Opcionales

| Requisito | Especificación | Milestone | Tests | Estado |
| --- | --- | --- | --- | --- |
| Rol administrador (ENUNCIADO §9) | HU-10, ADR-0037, ADR-0061, Constitución P-18 | Rol, administrador inicial y protección de `/api/admin/**`: Turnos M2. Operaciones administrativas: Turnos, administración | `ProvisionAdminUserUseCaseImplTest`, `AdminUserInitializerTest`, `SecurityConfigTest` (turnos), `UserAuthenticationFlowTest` | 🟦 parcial |

## Requisitos no funcionales

| RNF | Tema | Milestone | Tests | Estado |
| --- | --- | --- | --- | --- |
| RNF-01 | Contraseñas protegidas | Turnos M2 | `BcryptPasswordHasherAdapterTest`, `UserAuthenticationFlowTest`, `UserControllerTest` | 🟦 |
| RNF-02 | Secretos en configuración externa | Turnos y Catálogo M1 y M2 | Las pruebas generan sus claves en memoria (`TestJwtKeysConfiguration`); `.env` y `secrets/` fuera de Git | 🟦 |
| RNF-03 | La identidad sale del token | Turnos M2, Catálogo M2. Reservas: Turnos M4 a M6 | `SecurityConfigTest` (los dos servicios), `UserAuthenticationFlowTest` | 🟦 parcial |
| RNF-04 | Endpoints protegidos por defecto | Turnos M2, Catálogo M2 | `SecurityConfigTest` (los dos servicios), `CategoryAccessTest` | 🟦 |
| RNF-05 | Validación en cada límite | Turnos M2. Integraciones: milestones siguientes | `UserControllerTest`, `MaxBytesValidatorTest`, `GlobalExceptionHandlerTest` | 🟦 parcial |
| RNF-06 | Procesamiento idempotente de mensajes | Catálogo M5, Turnos M5 | | 🟨 |
| RNF-07 | Reintentos acotados y estado recuperable | Catálogo M3 y M5, Turnos M3 a M5 | | 🟨 |
| RNF-08 | Lecturas consistentes durante las actualizaciones | Catálogo M3 | | 🟨 |
| RNF-09 | Sin supuestos fijos sobre los datos y valores configurables | Catálogo M2 en adelante | `DatabaseMigrationTest` (catálogo); vigencia del JWT y CORS configurables | 🟦 parcial |
| RNF-10 | Pruebas automatizadas de los backends | Todas | CI de GitHub Actions en cada PR, en los dos servicios | 🟦 parcial |
