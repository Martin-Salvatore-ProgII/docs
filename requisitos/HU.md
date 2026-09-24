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
