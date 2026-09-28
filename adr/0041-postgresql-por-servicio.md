# ADR-0041. Un contenedor de PostgreSQL por servicio

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Cada servicio es dueño exclusivo de sus datos. Se permite compartir una instancia física solo con bases separadas, usuarios sin permisos cruzados y migraciones independientes (ENUNCIADO §3). El enunciado pide demostrar la separación efectiva de datos entre los dos servicios (ENUNCIADO §11).

## Decisión

Cada servicio tiene su propio contenedor de PostgreSQL, con su propia base, usuario y volumen.

## Alternativas consideradas

- **Una instancia compartida con dos bases y dos usuarios.** Permitida, pero descartada: la separación depende de configurar bien los permisos y hay que demostrarla. Con dos contenedores, la separación es física: un servicio no tiene forma de conectarse a la base del otro.

## Consecuencias

- La separación de datos se demuestra mostrando la configuración: cada servicio solo conoce su propia base.
- Un contenedor más, con un costo de recursos despreciable.
