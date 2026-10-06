# ADR-0061. El administrador inicial lo crea el servicio al arrancar, con credenciales de la configuración externa

- **Estado:** Aceptado
- **Fecha:** 2026-10-06
- **Reemplaza:** la sección "Cómo nace un administrador" del [ADR-0037](0037-rol-administrador-alcance-cerrado.md). El alcance del rol, definido ahí, no cambia.

## Contexto

El ADR-0037 definía que la configuración externa indicaba el login de un usuario ya registrado y que, al arrancar, ese usuario recibía `ROLE_ADMIN`. Al implementarlo apareció un problema: el registro es público y el rol se asignaba solo por el login. Si el login configurado todavía no estaba registrado, cualquiera podía registrarlo primero y quedar como administrador en el siguiente arranque. La seguridad dependía de que quien opera el servicio hiciera los pasos en el orden correcto.

## Decisión

- El administrador **no se registra desde la app**. Lo crea el servicio de turnos al arrancar, con el login, la contraseña y el email que recibe por configuración externa. La contraseña se guarda con el mismo hash que cualquier otra ([ADR-0057](0057-contrasenas-con-bcrypt-hasta-72-bytes.md)).
- **La configuración manda.** Al arrancar, el único administrador es la cuenta que tiene el login **y** la contraseña configurados:
  - Si la cuenta no existe, se crea, con `ROLE_USER` y `ROLE_ADMIN`.
  - Si existe y tiene esa contraseña, conserva o recupera el rol.
  - Si existe con otra contraseña, no recibe el rol (y lo pierde si lo tenía), y el servicio lo informa en el log.
  - Cualquier otra cuenta que tuviera el rol lo pierde.
- Sin configuración no hay administrador: quien lo era queda como usuario común.
- Los tres valores van juntos. Una configuración incompleta impide el arranque.

## Alternativas consideradas

- **Asignar el rol a un usuario ya registrado, por su login** (la decisión anterior). Descartada por lo explicado en el contexto.
- **Reservar el login configurado para que el registro público lo rechace.** Descartada: cierra el hueco, pero el administrador igual tendría que crearse por otro camino, que es justamente esta decisión.
- **Un administrador con una contraseña por defecto en el código.** Descartada, como ya lo estaba en el ADR-0037: sería un secreto en el repositorio.
- **Crear el administrador con un script de SQL.** Descartada: habría que calcular el hash de la contraseña fuera del servicio y mantener un segundo lugar que conoce la estructura de la tabla.

## Consecuencias

- Ser administrador exige conocer un secreto de la configuración. Registrar un login no alcanza, sin importar el orden de los pasos.
- La contraseña del administrador es un secreto más de la configuración externa: va en el `.env`, fuera de Git ([ADR-0043](0043-configuracion-externa-env-y-secrets.md)), y no aparece en los logs.
- Cambiar la contraseña configurada no cambia la de la cuenta existente: esa cuenta deja de ser administrador. Para cambiarla hay que usar otro login.
- Los cambios de rol no se hacen en una sola transacción: una caída en el medio puede dejar al sistema sin administrador hasta el próximo arranque, que lo corrige.
- Un token emitido antes de un cambio conserva sus roles hasta que vence, porque no hay revocación.
- El administrador creado así tiene nombre y apellido fijos ("Admin"), que son los que se enviarían a la cátedra si reservara un turno.
