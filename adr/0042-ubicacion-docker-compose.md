# ADR-0042. Un Compose por servicio y el del sistema completo en turnos, incluyendo al del catálogo

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Docker Compose tiene que levantar los dos backends, sus bases y sus dependencias (ENUNCIADO §3), pero cada servicio vive en su propio repositorio (ENUNCIADO §13.1). El repo `docs` no forma parte de la entrega.

## Decisión

- Cada servicio tiene su propio `compose.yaml` con el servicio y su PostgreSQL, para desarrollarlo y probarlo por separado.
- El Compose del sistema completo vive en `turnos-service` e **incluye** el del catálogo con la directiva `include` de Docker Compose.
- Los repos se clonan uno al lado del otro, como se documenta en el README de `turnos-service`.

## Alternativas consideradas

- **Un cuarto repositorio de infraestructura.** Descartada: suma una pieza más para mantener y entregar.
- **El Compose del sistema en `docs`.** Descartada: `docs` no es parte de la entrega.
- **Imágenes publicadas en un registro y un Compose que las descarga.** Descartada: requiere publicar imágenes en cada cambio, sin beneficio para un entorno local.

## Consecuencias

- Turnos es el servicio que depende del catálogo ([ADR-0002](0002-comunicacion-unidireccional-turnos-catalogo.md)), así que es natural que levante el sistema completo.
- Cada servicio sigue pudiendo levantarse solo.
- Levantar el sistema completo requiere tener los dos repos clonados en la misma carpeta.
