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
| [0018](0018-usuario-compatible-sin-generador-jhipster.md) | El usuario se implementa a mano dentro de la hexagonal, compatible con JHipster, sin usar su generador | Aceptado |
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
| [0037](0037-rol-administrador-alcance-cerrado.md) | Rol administrador con alcance cerrado: ver todas las reservas y cancelar en nombre de un usuario | Aceptado |
| [0038](0038-gradle-java-spring-boot.md) | Gradle en los tres repos, Java 25 en los backends, Java 21 en la app y Spring Boot 4 | Aceptado |
| [0039](0039-flyway.md) | Flyway para las migraciones de base de datos | Aceptado |
| [0040](0040-paquetes-segun-skill-hexagonal.md) | Los paquetes siguen la convención de la skill `/hexagonal` | Aceptado |
| [0041](0041-postgresql-por-servicio.md) | Un contenedor de PostgreSQL por servicio | Aceptado |
| [0042](0042-ubicacion-docker-compose.md) | Un Compose por servicio y el del sistema completo en turnos, incluyendo al del catálogo | Aceptado |
| [0043](0043-configuracion-externa-env-y-secrets.md) | Configuración externa con `.env` y archivos secretos montados, ambos fuera de Git | Aceptado |
| [0044](0044-pruebas-por-capa-con-catedra-simulada.md) | Pruebas por capa según la skill `/hexagonal`, con la cátedra siempre simulada | Aceptado |
| [0045](0045-postgresql-real-en-tests.md) | Las pruebas usan PostgreSQL real con Testcontainers, nunca H2 | Aceptado |
| [0046](0046-ci-con-github-actions.md) | Integración continua con GitHub Actions como check obligatorio para mergear | Aceptado |
| [0047](0047-tests-basicos-en-la-app.md) | La app KMP tiene pocas pruebas, básicas y sobre lógica pura | Aceptado |
| [0048](0048-compose-multiplatform.md) | La interfaz se construye con Compose Multiplatform en el código común | Aceptado |
| [0049](0049-jwt-cifrado-en-el-dispositivo.md) | El JWT de usuario se guarda cifrado en el dispositivo | Aceptado |
| [0050](0050-prototipos-con-herramientas-de-diseno.md) | Las pantallas principales se prototipan con las herramientas de diseño de Claude antes de implementarlas | Aceptado |
| [0051](0051-mvvm-en-la-app.md) | La app KMP sigue la arquitectura MVVM de la skill `/mvvm-kmp` | Aceptado |
| [0052](0052-stack-de-la-app.md) | Stack de la app: lifecycle y navegación multiplataforma, Ktor, kotlinx.serialization y Koin | Propuesto |
| [0053](0053-archunit-regla-de-dependencias.md) | La regla de dependencias de la hexagonal se verifica con un test de ArchUnit | Aceptado |
| [0054](0054-postgresql-18-en-los-dos-servicios.md) | PostgreSQL 18 en los dos servicios, con la versión mayor fijada | Aceptado |
| [0055](0055-convenciones-comunes-del-esqueleto.md) | Convenciones comunes del esqueleto de los dos backends | Aceptado |
| [0056](0056-imagen-con-dockerfile-y-puertos.md) | Imagen de cada backend con un Dockerfile en dos etapas y puertos locales sin superposición | Aceptado |
| [0057](0057-contrasenas-con-bcrypt-hasta-72-bytes.md) | Las contraseñas se guardan con BCrypt y se limitan a 72 bytes | Aceptado |
| [0058](0058-auditoria-automatica-de-spring-data.md) | Los campos de auditoría los completa Spring Data, no los casos de uso | Aceptado |
| [0059](0059-excepciones-de-aplicacion-con-codigo.md) | Los casos de uso informan los rechazos de negocio con excepciones de aplicación que llevan el código funcional | Aceptado |
| [0060](0060-contenido-y-claves-del-jwt-de-usuario.md) | El JWT de usuario lleva solo login y roles, se firma con la librería de Spring Security y sus claves nunca están en Git | Aceptado |
| [0061](0061-administrador-inicial-creado-al-arrancar.md) | El administrador inicial lo crea el servicio al arrancar, con credenciales de la configuración externa | Aceptado |

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
