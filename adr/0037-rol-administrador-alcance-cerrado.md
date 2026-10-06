# ADR-0037. Rol administrador con alcance cerrado: ver todas las reservas y cancelar en nombre de un usuario

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

El enunciado permite, de forma opcional, un rol administrador capaz de consultar o gestionar reservas de otros usuarios, siempre que se documenten sus permisos, se demuestre que sus operaciones están correctamente autorizadas y no reemplace el aislamiento entre usuarios comunes (ENUNCIADO §9). El modelo de usuario de JHipster ya incluye autoridades, con `ROLE_USER` y `ROLE_ADMIN` ([ADR-0018](0018-usuario-compatible-sin-generador-jhipster.md)).

Tres necesidades lo justifican:

1. **Soporte:** ver por qué un proceso falló o quedó colgado y cancelar una reserva en nombre de un usuario, sin entrar a la base.
2. **Demostración:** mostrar en vivo todos los procesos con sus estados y motivos (confirmados por Kafka, cerrados por la reconciliación, vencidos), que es la parte más compleja del sistema.
3. **Autorización por rol:** complementa la autorización por dueño y permite demostrar que un usuario común es rechazado en operaciones administrativas.

## Decisión

Existe el rol `ROLE_ADMIN`, con un alcance **cerrado**. Un administrador también es un usuario (`ROLE_USER`) y puede reservar como cualquiera, con las mismas reglas.

### Lo que hace

| Capacidad | Detalle |
| --- | --- |
| Listar todas las reservas | De todos los usuarios, paginadas, con filtros por login del dueño, estado y rango de fechas del turno |
| Ver el detalle de cualquier reserva | Los mismos datos que ve el dueño, más: el dueño (login, nombre y apellido), `reservationProcessId`, `failureReason` y quién la canceló |
| Cancelar una reserva confirmada de cualquier usuario | Con motivo **obligatorio**. Queda registrado qué administrador la canceló. Mismas reglas de estado que la cancelación del dueño ([ADR-0026](0026-cancelacion-con-respuesta-rest.md)) |

### Lo que no hace

- No gestiona usuarios: no los crea, edita, bloquea ni borra, no cambia roles ni resetea contraseñas.
- No crea reservas ni envía teléfonos en nombre de otro usuario.
- No modifica estados a mano ni fuerza transiciones: la máquina de estados es la misma para todos ([ADR-0023](0023-estados-solo-avanzan.md)).
- No modifica el catálogo (es de la cátedra) ni fuerza sincronizaciones ([ADR-0006](0006-disparadores-sincronizacion.md)).
- No ve teléfonos (no se guardan) ni datos de la integración técnica.

Cualquier capacidad nueva requiere un ADR que reemplace a este.

### Cómo se protege

- Las operaciones administrativas viven en rutas propias (`/api/admin/**`), protegidas por rol a nivel de ruta. Las rutas de usuario no tienen lógica condicional por rol.
- Un usuario sin `ROLE_ADMIN` recibe 403 en cualquier ruta administrativa.

### Cómo nace un administrador

> Esta sección fue reemplazada por el [ADR-0061](0061-administrador-inicial-creado-al-arrancar.md): el administrador ya no es un usuario registrado al que se le asigna el rol, sino una cuenta que crea el servicio al arrancar. El resto de este ADR sigue vigente.

No hay credenciales de administrador en el código ni en los repositorios ([Constitución P-08](../constitucion.md)). La configuración externa del servicio de turnos puede indicar el login de un usuario ya registrado; al arrancar, ese usuario recibe `ROLE_ADMIN`. Si no se indica, no hay administrador.

## Alternativas consideradas

- **Sin rol administrador.** Descartada: se pierde la visibilidad para soporte y para la demo, y la autorización queda limitada al criterio de dueño.
- **Un administrador con gestión de usuarios y del sistema.** Descartada: suma superficie de ataque y trabajo sin aportar a los requisitos, y el acceso cruzado es justamente lo que se evalúa.
- **Crear el administrador con una contraseña por defecto.** Descartada: sería un secreto en el repositorio.

## Consecuencias

- Se implementa **después** del flujo obligatorio, salvo la protección de `/api/admin/**` por rol, que se incluye desde el esqueleto con su test.
- Hay que demostrar con tests que un usuario común recibe 403 en todas las rutas administrativas y que el aislamiento entre usuarios comunes no cambia.
- La app muestra la sección de administración solo a quien tiene `ROLE_ADMIN`.
