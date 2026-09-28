# Historias de usuario

Cada historia describe qué necesita un actor y cómo se verifica que está cumplida. Los criterios de aceptación son observables desde afuera del sistema; no describen la implementación.

**Actores**

- **Usuario final:** persona que usa la app KMP.
- **Responsable del proyecto:** el alumno, cuando opera la integración con la cátedra.

## Identidad y acceso

### HU-01. Registro de usuario final

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §3.2, §5 (punto 1), §9
- **Actor:** usuario final

**Como** usuario final **quiero** crear una cuenta desde la app **para** poder buscar profesionales y reservar turnos.

**Criterios de aceptación**

1. **Dado** que completo `login`, `password`, `firstName`, `lastName`, `email` y `langKey`, y opcionalmente `imageUrl`, con datos válidos, **cuando** me registro, **entonces** la cuenta queda creada y activa, y puedo iniciar sesión sin verificar el correo.
2. **Dado** un `login` que ya está en uso, **cuando** me registro, **entonces** el registro se rechaza indicando que el login ya existe.
3. **Dado** un `email` que ya está en uso, **cuando** me registro, **entonces** el registro se rechaza indicando que el email ya existe.
4. **Dado** un dato fuera de las reglas de validación, **cuando** me registro, **entonces** el registro se rechaza indicando qué campo es inválido.
5. **Dado** que envío en el registro un id, autoridades, estado de activación o campos de auditoría, **cuando** me registro, **entonces** esos valores se ignoran y los asigna el backend.
6. **Dado** un registro exitoso, **entonces** ninguna respuesta ni log contiene la contraseña, y la contraseña no se guarda en texto plano.

**Reglas de validación** (compatibles con el usuario de JHipster, ENUNCIADO §3.2)

| Campo | Obligatorio | Regla |
| --- | --- | --- |
| `login` | Sí | 1 a 50 caracteres; patrón de login de JHipster (`^(?>[a-zA-Z0-9!$&*+=?^_`{\|}~.-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*)\|(?>[_.@A-Za-z0-9-]+)$`); se guarda en minúsculas |
| `password` | Sí | 4 a 100 caracteres |
| `firstName` | Sí | 2 a 50 caracteres |
| `lastName` | Sí | 2 a 50 caracteres |
| `email` | Sí | Email válido, 5 a 254 caracteres, único; se guarda en minúsculas |
| `imageUrl` | No | Hasta 256 caracteres |
| `langKey` | Sí | 2 a 10 caracteres |

> `firstName` y `lastName` son obligatorios aunque JHipster los permita vacíos, porque confirmar un hold exige `patientFirstName` y `patientLastName` de 2 a 100 caracteres (REF §10). Por la misma razón, su mínimo es de 2 caracteres.

### HU-02. Inicio de sesión de usuario final

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §3.2, §5 (punto 1), §9
- **Actor:** usuario final

**Como** usuario final registrado **quiero** iniciar sesión desde la app **para** usar las funciones protegidas con mi identidad.

**Criterios de aceptación**

1. **Dado** un login y una contraseña correctos, **cuando** inicio sesión, **entonces** recibo un JWT de usuario que la app usa durante la sesión.
2. **Dado** un login inexistente o una contraseña incorrecta, **cuando** inicio sesión, **entonces** se rechaza con 401, sin indicar cuál de los dos datos falló.
3. **Dado** que llamo a una función protegida de cualquiera de los dos servicios sin JWT, o con un JWT inválido o vencido, **entonces** se rechaza con 401.
4. **Dado** un JWT de usuario válido, **entonces** los dos servicios reconocen mi identidad a partir del token, sin necesitar que la app envíe un id de usuario.

### HU-03. Alta y configuración de la integración técnica

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §5 (puntos 2 y 3); REF §5
- **Actor:** responsable del proyecto

**Como** responsable del proyecto **quiero** registrar una única cuenta técnica ante la cátedra y que los backends obtengan su configuración **para** que el sistema pueda usar REST, Redis y Kafka de la cátedra.

**Criterios de aceptación**

1. **Dado** que registro la cuenta técnica una sola vez con Postman o una herramienta equivalente, **entonces** obtengo el JWT técnico y el `groupId` normalizado, y quedan guardados fuera de cualquier repositorio.
2. **Dado** que un backend arranca con la URL de la cátedra y el JWT técnico en su configuración externa, **cuando** la integración está `PROVISIONED`, **entonces** obtiene la configuración de Redis, Kafka y topics desde la cátedra y queda operativo ([ADR-0005](../adr/0005-configuracion-integracion-al-arrancar.md)).
3. **Dado** que la cátedra no responde o la integración no está `PROVISIONED`, **cuando** un backend arranca, **entonces** no arranca e informa el motivo sin mostrar secretos.
4. **Dado** cualquier estado del sistema, **entonces** el JWT técnico y los valores de `integration` no aparecen en los repositorios, los logs, las respuestas a la app ni la documentación.
5. **Dado** que el JWT técnico queda expuesto, **entonces** el procedimiento documentado es crear otra cuenta técnica, reconfigurar ambos backends y avisar a la cátedra (REF §5.4).

## Catálogo

### HU-04. Catálogo actualizado

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §4.1, §5 (puntos 4 a 6), §6
- **Actor:** usuario final

**Como** usuario final **quiero** que el catálogo que veo en la app refleje los cambios que publica la cátedra **para** buscar y reservar sobre profesionales y horarios vigentes.

**Criterios de aceptación**

1. **Dado** que la cátedra publica una versión nueva del catálogo, **cuando** el servicio de catálogo la sincroniza, **entonces** mis búsquedas muestran los cambios, sin que tenga que hacer nada.
2. **Dado** que hay una sincronización en curso, **cuando** busco profesionales, **entonces** veo los resultados de la copia anterior completa y un aviso no bloqueante de que el catálogo se está actualizando.
3. **Dado** cualquier momento, **cuando** busco, **entonces** nunca veo una mezcla de datos de dos versiones distintas del catálogo.
4. **Dado** que la sincronización falla, **cuando** busco, **entonces** sigo viendo la última copia completa y la búsqueda no falla por eso.
5. **Dado** que estoy autenticado, **cuando** consulto el estado del catálogo, **entonces** veo la versión local, si hay una sincronización en curso, la última sincronización exitosa y el último error, si lo hubo.

Los escenarios de sincronización del lado del servicio están en [CU-01 y CU-02](CU.md).

### HU-05. Buscar profesionales

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §4.1, §5 (punto 7), §6.2, §11
- **Actor:** usuario final

**Como** usuario final **quiero** buscar profesionales combinando filtros **para** encontrar con quién sacar un turno.

**Criterios de aceptación**

1. **Dado** que estoy autenticado, **cuando** busco, **entonces** puedo combinar los filtros por categoría, nombre, estado habilitado y disponibilidad, y los resultados cumplen todos los filtros indicados.
2. **Dado** que filtro por nombre, **entonces** encuentro coincidencias parciales sobre el nombre, el apellido o el nombre completo, sin importar mayúsculas ni tildes ("ana gom" encuentra a "Ana Gómez").
3. **Dado** que no indico el filtro de estado, **entonces** solo veo profesionales habilitados. **Dado** que pido los deshabilitados, **entonces** los veo marcados como no disponibles para reservar. Un profesional está habilitado solo si él y su categoría lo están ([ADR-0014](../adr/0014-habilitado-efectivo.md)).
4. **Dado** que filtro por disponibilidad con una fecha, y opcionalmente una franja horaria, **entonces** veo los profesionales con algún horario habilitado ese día de la semana que se superpone con la franja. Es disponibilidad de agenda: no garantiza turnos libres ([ADR-0013](../adr/0013-filtro-disponibilidad-de-agenda.md)).
5. **Dado** que puedo filtrar por categoría, **entonces** la app me ofrece la lista de categorías habilitadas.
6. **Dado** muchos resultados, **entonces** vienen paginados, ordenados por apellido y nombre, con el total disponible.
7. **Dado** que la cátedra no está disponible, **cuando** busco, **entonces** la búsqueda funciona igual, porque usa solo la copia local (ENUNCIADO §6.2).
8. **Dado** un filtro inválido (por ejemplo, una franja cuyo inicio es posterior al fin, o una franja sin fecha), **entonces** la búsqueda se rechaza indicando el problema.

## Turnos

### HU-06. Ver turnos disponibles

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §4.2, §5 (punto 8), §7, §8
- **Actor:** usuario final

**Como** usuario final **quiero** ver los horarios libres de un profesional en una fecha **para** elegir uno y reservarlo.

**Criterios de aceptación**

1. **Dado** un profesional habilitado y una fecha válida, **cuando** consulto la disponibilidad, **entonces** veo los slots que surgen de sus horarios semanales habilitados para ese día, menos los que la cátedra informa como ocupados (`CONFIRMED` o `HELD`).
2. **Dado** un horario de 09:00 a 12:00 con slots de 50 minutos, **entonces** se ofrecen 09:00, 09:50 y 10:40: nunca un slot que termine después del fin del horario ([ADR-0015](../adr/0015-reglas-disponibilidad-turnos.md)).
3. **Dado** que consulto la fecha de hoy, **entonces** no veo slots cuyo inicio ya pasó según la hora de Argentina ([ADR-0016](../adr/0016-zona-horaria-argentina.md)).
4. **Dado** una fecha pasada o más allá del horizonte permitido, **entonces** la consulta se rechaza como inválida.
5. **Dado** un profesional inexistente o deshabilitado, **entonces** la consulta se rechaza indicando el motivo.
6. **Dado** que el profesional no atiende ese día, o todos sus slots están ocupados, **entonces** veo una lista vacía, no un error.
7. **Dado** que el servicio de catálogo o la cátedra no están disponibles, **entonces** no veo una disponibilidad incompleta: la consulta falla indicando que el servicio no está disponible temporalmente (ENUNCIADO §8).
8. **Dado** que veo un slot como libre, **entonces** eso no garantiza la reserva: la cátedra lo confirma al crear el hold.

### HU-07. Reservar un turno

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §4.2, §5 (puntos 9 a 12), §7, §11
- **Actor:** usuario final
- **Flujo:** [CU-04](CU.md#cu-04-reservar-un-turno)

**Como** usuario final **quiero** reservar un turno libre **para** asegurarme la atención con ese profesional.

**Criterios de aceptación**

1. **Dado** un slot libre, **cuando** toco "Reservar", **entonces** el turno queda bloqueado para mí mientras completo la reserva, sin que tenga que hacer otra acción ([ADR-0019](../adr/0019-hold-y-confirmacion-una-accion.md)).
2. **Dado** que la cátedra pide información adicional, **entonces** la app me muestra su mensaje, el tiempo que me queda y me pide el teléfono ([ADR-0020](../adr/0020-telefono-a-pedido.md)).
3. **Dado** un teléfono con formato inválido, **cuando** lo envío, **entonces** se rechaza al instante y puedo corregirlo ([ADR-0027](../adr/0027-validacion-local-del-telefono.md)).
4. **Dado** que la cátedra rechaza el teléfono, **entonces** veo el motivo y puedo enviar otro mientras el proceso no venza.
5. **Dado** un teléfono aceptado, **entonces** veo la reserva confirmada.
6. **Dado** que no envío el teléfono antes del vencimiento, **entonces** veo que el tiempo venció y el turno queda liberado.
7. **Dado** que el turno ya fue tomado por otro, o dejó de ser válido, **cuando** intento reservarlo, **entonces** veo el motivo y puedo elegir otro.
8. **Dado** que ya tengo una reserva en curso, **cuando** intento iniciar otra, **entonces** se rechaza indicando que tengo una en curso ([ADR-0024](../adr/0024-un-proceso-activo-por-usuario.md)).
9. **Dado** que cierro la app durante el proceso, **cuando** la vuelvo a abrir, **entonces** veo el proceso en el estado en que quedó y puedo continuarlo ([ADR-0021](../adr/0021-app-consulta-estado-por-polling.md)).
10. **Dado** que el catálogo no está disponible, **cuando** intento reservar, **entonces** no se inicia la reserva y veo que el servicio no está disponible temporalmente (ENUNCIADO §8).
11. **Dado** cualquier orden de llegada o repetición de las respuestas y eventos de la cátedra, **entonces** nunca se generan reservas duplicadas y un proceso terminado no cambia de resultado ([`arq/maquina-estados.md`](../arq/maquina-estados.md)).

### HU-08. Consultar mis reservas

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §4.2, §5 (punto 13), §8, §9, §11
- **Actor:** usuario final

**Como** usuario final **quiero** ver mis reservas y su estado **para** saber qué turnos tengo.

**Criterios de aceptación**

1. **Dado** que estoy autenticado, **cuando** consulto mis reservas, **entonces** veo solo las que inicié yo, con profesional, fecha, horario y estado.
2. **Dado** una reserva de otro usuario, **cuando** intento verla por su identificador, **entonces** la respuesta es la misma que si no existiera.
3. **Dado** que la cátedra o el catálogo no están disponibles, **cuando** consulto mis reservas, **entonces** las veo igual ([ADR-0025](../adr/0025-reservas-propias-desde-base-local.md)).
4. **Dado** muchas reservas, **entonces** vienen paginadas, de la más reciente a la más antigua, y puedo filtrarlas por estado.

### HU-09. Cancelar una reserva

- **Estado:** Aceptado
- **Origen:** ENUNCIADO §4.2, §5 (punto 14), §9, §11; REF §12
- **Actor:** usuario final

**Como** usuario final **quiero** cancelar una reserva confirmada **para** liberar el turno si no voy a asistir.

**Criterios de aceptación**

1. **Dado** una reserva mía confirmada, **cuando** la cancelo, opcionalmente con un motivo, **entonces** queda cancelada y el turno se libera ([ADR-0026](../adr/0026-cancelacion-con-respuesta-rest.md)).
2. **Dado** una reserva mía ya cancelada, **cuando** la vuelvo a cancelar, **entonces** la operación responde éxito sin cambios.
3. **Dado** una reserva mía que no está confirmada, **cuando** intento cancelarla, **entonces** se rechaza indicando que no se puede cancelar.
4. **Dado** una reserva de otro usuario, **cuando** intento cancelarla, **entonces** la respuesta es la misma que si no existiera y la reserva no cambia.
5. **Dado** que la cátedra no está disponible, **cuando** intento cancelar, **entonces** la reserva no cambia y veo que el servicio no está disponible temporalmente.
