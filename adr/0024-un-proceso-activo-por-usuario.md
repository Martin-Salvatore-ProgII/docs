# ADR-0024. Un usuario tiene como máximo un proceso de reserva activo

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Un doble toque en "Reservar" o dos pedidos simultáneos podrían crear dos holds para el mismo usuario. Además, un usuario podría bloquear varios slots a la vez sin completar ninguno.

## Decisión

Mientras un usuario tenga un proceso en un estado no final, no puede iniciar otro. El intento se rechaza indicando que ya hay una reserva en curso.

## Alternativas consideradas

- **Permitir varios procesos simultáneos por usuario.** Descartada: no hay un caso de uso real para reservar en paralelo y habilita duplicados y el bloqueo de varios slots.
- **Deduplicar solo pedidos idénticos (mismo profesional, fecha y hora).** Descartada: no evita que un usuario bloquee varios slots distintos.

## Consecuencias

- El doble toque no genera holds duplicados.
- Un usuario puede reservar varios turnos, pero uno después del otro.
- La regla tiene que garantizarse también ante pedidos concurrentes del mismo usuario, no solo con una consulta previa.
