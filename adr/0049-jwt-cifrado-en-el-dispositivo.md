# ADR-0049. El JWT de usuario se guarda cifrado en el dispositivo

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

La app conserva el JWT de usuario durante la sesión, hasta 30 días con `rememberMe` ([ADR-0033](0033-vigencia-jwt-usuario.md)). Es una credencial: quien la obtenga puede operar como el usuario hasta que venza. El enunciado pide proteger credenciales y tokens (ENUNCIADO §9).

## Decisión

El JWT se guarda en almacenamiento cifrado respaldado por el sistema de claves del dispositivo. Se borra al cerrar sesión o cuando un servicio responde 401.

## Alternativas consideradas

- **Guardarlo en las preferencias de la app en texto plano.** Descartada: cualquier acceso al almacenamiento de la app expone la credencial.
- **Guardarlo solo en memoria.** Descartada: habría que iniciar sesión cada vez que se abre la app, y `rememberMe` perdería sentido.

## Consecuencias

- La sesión sobrevive al cierre de la app sin dejar la credencial expuesta.
- Es código específico de Android dentro de KMP.
