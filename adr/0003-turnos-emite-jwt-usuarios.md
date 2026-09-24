# ADR-0003. El servicio de turnos registra a los usuarios finales y emite su JWT

- **Estado:** Aceptado
- **Fecha:** 2026-09-23

## Contexto

Los usuarios finales se registran e inician sesión desde la app KMP contra "el backend desarrollado por el alumno", que emite un JWT para la sesión (ENUNCIADO §3.2). El enunciado no dice cuál de los dos servicios lo hace. Por otro lado, la cátedra solo conoce la cuenta técnica; la propiedad de cada proceso y reserva por usuario final la tiene que llevar nuestro sistema (ENUNCIADO §4.2; REF §2).

## Decisión

El **servicio de turnos** es dueño de las cuentas de usuario final: las registra, las autentica y emite su JWT. El servicio de catálogo solo **valida** ese JWT.

## Alternativas consideradas

- **Que lo haga el servicio de catálogo.** Descartada: el catálogo trata datos públicos de profesionales y agendas; a nadie le importa *quién* busca. Lo que requiere identidad es la reserva: de quién es cada turno. Además, confirmar un hold exige `patientFirstName` y `patientLastName` (REF §10), que son datos del usuario. Con los usuarios en el catálogo, turnos tendría que pedírselos en cada reserva y el catálogo mezclaría responsabilidades ajenas a ENUNCIADO §4.1.

## Consecuencias

- La tabla de usuarios, sus migraciones y las contraseñas hasheadas viven en la base de turnos.
- Turnos tiene a mano la identidad y los datos del paciente para asociar procesos y reservas y para confirmar holds.
- El catálogo tiene que poder validar un token que no emite. El mecanismo (clave compartida o par de claves) se decide junto con la autenticación entre servicios (ver [`arq/seguridad.md`](../arq/seguridad.md), pendientes).
- Si turnos no está disponible, no se puede iniciar sesión ni registrarse. Un token ya emitido sigue siendo válido para buscar en el catálogo mientras no venza.
