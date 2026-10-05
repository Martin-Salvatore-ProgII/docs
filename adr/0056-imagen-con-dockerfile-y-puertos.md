# ADR-0056. Imagen de cada backend con un Dockerfile en dos etapas y puertos locales sin superposición

- **Estado:** Aceptado
- **Fecha:** 2026-10-04

## Contexto

Docker Compose levanta los dos backends y sus bases ([`arq/despliegue.md`](../arq/despliegue.md)), cada servicio con su propio `compose.yaml` y el del sistema completo incluyendo al del catálogo ([ADR-0042](0042-ubicacion-docker-compose.md)). Falta definir cómo se construye la imagen de cada backend y qué puertos publica cada contenedor en la máquina, que no pueden repetirse cuando se levanta el sistema completo.

## Decisión

- Cada backend tiene un **`Dockerfile` en dos etapas**: la primera compila el jar con el JDK 25 de Eclipse Temurin y la segunda lleva solo el JRE 25 y el jar. El Compose construye la imagen a partir de ese archivo.
- **Puertos publicados en la máquina**, todos configurables desde el `.env` de cada repo:

| Contenedor | Puerto en la máquina |
| --- | --- |
| Servicio de turnos | 8080 |
| Servicio de catálogo | 8081 |
| PostgreSQL del catálogo | 5433 |
| PostgreSQL de turnos | 5434 |

- Dentro de la red de Docker, cada servicio escucha en 8080 y cada PostgreSQL en 5432.
- Los nombres de servicios y volúmenes de cada Compose llevan el prefijo de su servicio (`catalogo-`, `turnos-`).

## Alternativas consideradas

- **Generar la imagen con `bootBuildImage` de Spring Boot.** Descartada: no requiere escribir un Dockerfile, pero la imagen la arma una herramienta cuyo contenido no se ve en el repo. El Dockerfile se lee y se defiende línea por línea.
- **Compilar el jar en la máquina y copiarlo a la imagen.** Descartada: levantar el sistema exigiría tener Java y un paso previo de build. Con dos etapas alcanza con Docker.
- **No publicar los puertos de PostgreSQL.** Descartada: es más cerrado, pero impide mirar las bases con un cliente desde la máquina, que hace falta para desarrollar y para demostrar la separación de datos (ENUNCIADO §11).
- **El puerto estándar 5432 para alguna de las bases.** Descartada: choca con un PostgreSQL instalado en la máquina.

## Consecuencias

- Levantar un servicio o el sistema completo requiere solo Docker.
- Los dos Compose se pueden levantar juntos sin choques de puertos, nombres ni volúmenes.
- La app y el servicio de turnos tienen que conocer el puerto del catálogo: desde la máquina es el publicado; entre contenedores, el nombre del servicio y el 8080.
- La primera construcción de cada imagen descarga las dependencias y tarda; las siguientes reutilizan esa capa mientras no cambie el build.
