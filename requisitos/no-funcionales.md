# Requisitos no funcionales

Requisitos de calidad que no son una funcionalidad concreta pero que el sistema tiene que cumplir en todas sus partes. Cada uno indica cómo se verifica.

## Seguridad

### RNF-01. Contraseñas protegidas

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §3.2, §9

Las contraseñas de los usuarios finales se guardan con un algoritmo de hash criptográfico adecuado para contraseñas, nunca en texto plano ni con un cifrado reversible. No aparecen en respuestas ni en logs.

**Verificación:** test que comprueba que el valor guardado no es la contraseña original y que se valida contra el hash; revisión de logs y respuestas del registro y del login.

### RNF-02. Secretos en configuración externa

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §9; REF §3, §5.1

El JWT técnico, las claves de firma, las credenciales de base de datos y cualquier valor de `integration` se cargan desde configuración externa. No están en el código, en los repositorios, en los logs ni en la app.

**Verificación:** los repos incluyen solo ejemplos con valores ficticios (por ejemplo, `.env.example`); búsqueda de secretos en el historial de Git; revisión de logs durante la demo.

### RNF-03. La identidad sale del token

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §9; REF §6

La identidad del usuario final se obtiene siempre del JWT de usuario validado, y la integración técnica, del JWT técnico. Ningún servicio acepta del cliente un id de usuario o un `groupId` para decidir sobre qué datos opera.

**Verificación:** tests de autorización en los que un usuario intenta operar sobre recursos de otro enviando identificadores ajenos, y se rechaza.

### RNF-04. Endpoints protegidos por defecto

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §9, §11

Todo endpoint de los dos servicios exige un JWT válido, salvo los declarados explícitamente como públicos (registro e inicio de sesión).

**Verificación:** tests que llaman a endpoints protegidos sin token y con un token inválido o vencido, y esperan 401.

### RNF-05. Validación en cada límite

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §9

Los datos que entran al sistema (desde la app, desde el otro servicio y desde la cátedra por REST, Redis o Kafka) se validan antes de usarse. Un dato inválido se rechaza con un error claro y no corrompe el estado local.

**Verificación:** tests con entradas inválidas en cada límite.

## Robustez

### RNF-06. Procesamiento idempotente de mensajes

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §6.2, §8, §11; REF §4, §15.1, §18.1

Todo consumidor Kafka tolera recibir el mismo mensaje más de una vez: un `eventId` ya procesado no repite efectos y el estado final es el mismo ([ADR-0008](../adr/0008-deduplicacion-por-event-id.md)). Los consumidores ignoran los campos desconocidos compatibles con `schemaVersion` 1.

**Verificación:** tests que entregan dos veces el mismo evento y comprueban que el estado no cambia en la segunda entrega; demostración con un mensaje duplicado.

### RNF-07. Reintentos acotados y estado recuperable

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §8; REF §18.4

Ninguna falla de una dependencia externa genera reintentos ilimitados. Toda falla deja el estado local en un punto consistente desde el que el sistema puede continuar sin intervención manual.

**Verificación:** tests que simulan la caída de la dependencia a mitad de una operación y comprueban el estado resultante y la recuperación posterior.

### RNF-08. Lecturas consistentes durante las actualizaciones

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §6.1, §6.2

Ninguna lectura observa una actualización a medias: las búsquedas ven el catálogo completo de una versión o el de la siguiente, nunca una mezcla ([ADR-0010](../adr/0010-snapshot-en-una-transaccion.md)).

**Verificación:** test que busca mientras se aplica un snapshot y comprueba que el resultado corresponde completo a una sola versión.

## Mantenibilidad

### RNF-09. Sin supuestos fijos sobre los datos de la cátedra, y valores operativos configurables

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §1, §6; indicación del profesor sobre el versionado de datos

El catálogo de la cátedra cambia como en la vida real: aparecen y se dan de baja categorías y profesionales, y cambian días, horarios y duraciones de los slots. El sistema no asume nada fijo sobre esos datos: ninguna categoría, profesional, día, horario ni duración se escribe en el código; todo sale del catálogo sincronizado.

Los valores operativos del sistema se definen por configuración externa, con un valor inicial documentado. Hasta ahora:

| Valor | Inicial | Decisión |
| --- | --- | --- |
| Intervalo del chequeo periódico de sincronización | 5 minutos | [ADR-0006](../adr/0006-disparadores-sincronizacion.md) |
| Horizonte de fechas para la disponibilidad | 60 días | [ADR-0015](../adr/0015-reglas-disponibilidad-turnos.md) |
| Tamaño de página por defecto y máximo | 20 y 100 | [`arq/contratos.md`](../arq/contratos.md) |
| Intervalo de consulta del estado de una reserva (en la app) | 2 segundos | [ADR-0021](../adr/0021-app-consulta-estado-por-polling.md) |
| Timeouts de conexión y de respuesta hacia la cátedra y el catálogo | 2 y 5 segundos | [ADR-0029](../adr/0029-reintentos-acotados-por-operacion.md) |
| Reintentos de operaciones seguras de repetir | 2 (1 en lecturas) | [ADR-0029](../adr/0029-reintentos-acotados-por-operacion.md) |
| Intervalo y margen de la reconciliación de procesos | 60 y 30 segundos | [ADR-0030](../adr/0030-reconciliacion-periodica-de-procesos.md) |
| Reintentos al procesar un evento Kafka | 3 | [ADR-0031](../adr/0031-eventos-kafka-problematicos.md) |
| Vigencia del JWT de usuario (normal y con `rememberMe`) | 24 horas y 30 días | [ADR-0033](../adr/0033-vigencia-jwt-usuario.md) |
| Orígenes CORS permitidos | Ninguno | [ADR-0036](../adr/0036-cors-cerrado.md) |

La zona horaria de la agenda es la excepción: es fija por decisión ([ADR-0016](../adr/0016-zona-horaria-argentina.md)).

**Verificación:** los tests usan catálogos con datos variados (distintas duraciones, varios horarios por día, entidades deshabilitadas) y cambian los valores configurables; revisión de que el código no contiene datos del catálogo.
