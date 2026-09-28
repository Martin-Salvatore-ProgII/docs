# ADR-0035. HTTPS entre la app y los backends, con certificado autofirmado en el entorno local

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

La solución debe proteger la comunicación entre la interfaz y los servicios backend (ENUNCIADO §9). Por ese canal viajan contraseñas y JWT de usuario.

## Decisión

Los dos servicios exponen su API por HTTPS. En el entorno local se usa un certificado autofirmado, cargado desde configuración externa, y la app se configura para confiar en ese certificado.

## Alternativas consideradas

- **HTTP en local, documentado como limitación.** Queda como plan B si la configuración en Android se complica. Descartada como opción principal: contraseñas y tokens viajarían en texto plano, y el enunciado pide proteger el canal.

## Consecuencias

- Contraseñas y tokens viajan cifrados.
- Hay que generar el certificado y configurar la app para confiar en él. El certificado y su clave no se commitean.
- La llamada de turnos al catálogo, dentro de la red de Docker Compose, también usa HTTPS.
