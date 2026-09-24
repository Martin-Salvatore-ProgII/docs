# ADR-0004. `externalPatientId` es un UUID propio de cada usuario

- **Estado:** Aceptado
- **Fecha:** 2026-09-23

## Contexto

Al crear y confirmar un hold se envía `externalPatientId`: un identificador estable del usuario dentro de la aplicación, no vacío y de hasta 100 caracteres (REF §2, §9, §10). La cátedra conserva ese valor en las reservas de la cuenta técnica, que persisten del lado de la cátedra aunque nuestras bases se reinicien.

## Decisión

Cada usuario final tiene un **UUID** generado por el servicio de turnos al registrarse. No cambia nunca y es el único valor que se envía como `externalPatientId`.

## Alternativas consideradas

- **El id numérico interno del usuario.** Descartada: si la base de turnos se recrea (por ejemplo, al borrar un volumen de Docker durante el desarrollo), los ids vuelven a empezar. Un usuario nuevo podría quedar con el mismo `externalPatientId` que reservas viejas de otra persona en la cátedra, y cualquier cruce por ese campo mezclaría dueños.
- **El `login`.** Descartada: es un dato identificable que no hace falta compartir con un tercero (ENUNCIADO §9 pide no exponer más de lo necesario).

## Consecuencias

- El usuario tiene una columna más con restricción de unicidad.
- El valor no se repite entre reinicios de la base ni revela datos personales ni la cantidad de usuarios.
- La propiedad de las reservas no se decide solo por `externalPatientId`: turnos guarda su propia asociación entre proceso de reserva y usuario (ver [`arq/seguridad.md`](../arq/seguridad.md)).
