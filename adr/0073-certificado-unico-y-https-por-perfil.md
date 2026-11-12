# ADR-0073. Un solo certificado autofirmado para los dos servicios, y HTTPS activado por un perfil

- **Estado:** Aceptado
- **Fecha:** 2026-11-12

## Contexto

[ADR-0035](0035-https-entre-app-y-backends.md) decide que los dos servicios exponen su API por HTTPS con un certificado autofirmado, cargado desde configuración externa, y que la llamada de turnos al catálogo también usa HTTPS. No dice cuántos certificados hay, cómo se generan ni qué pasa al levantar un servicio sin certificado, por ejemplo en las pruebas automáticas. La app va a tener que confiar en lo que se decida acá (app M6, Canal seguro).

## Decisión

- **Un solo certificado para los dos servicios.** Lo genera un script del repo de turnos, que es donde está el Compose del sistema completo, y lo deja en `secrets/` de los dos repos, fuera de Git. Vale para los nombres con los que se llega a los servicios: `localhost` y `127.0.0.1` (desde la máquina), `10.0.2.2` (la máquina vista desde el emulador de Android) y los nombres de los contenedores (`turnos-service`, `catalogo-service`).
- **HTTPS se activa con un perfil de configuración (`tls`)** que Docker Compose prende siempre. Con el perfil, cada servicio habla solo HTTPS, en los mismos puertos del [ADR-0056](0056-imagen-con-dockerfile-y-puertos.md), sin puerto HTTP ni redirección. Sin el perfil (pruebas automáticas, arranque con una base descartable) queda en HTTP.
- **Turnos confía solo en ese certificado al llamar al catálogo** y verifica que el certificado sea del host al que llama.
- **Las pruebas de HTTPS usan un certificado propio**, válido solo para `localhost`, guardado junto a las pruebas. No es un secreto: no protege nada fuera de ellas.

## Alternativas consideradas

- **Un certificado por servicio.** Descartada: la app tendría que confiar en dos certificados, y no hay nada que separar entre dos servicios que corren en la misma máquina.
- **HTTPS obligatorio también en las pruebas.** Descartada: todas las pruebas existentes necesitarían un certificado sin que eso pruebe nada nuevo. Lo que se quiere demostrar (que el servidor solo habla TLS, que un cliente que no confía es rechazado y que se verifica el host) lo prueban pruebas específicas con el perfil activo.
- **Generar el certificado de prueba al vuelo.** Descartada: requiere sumar una librería criptográfica solo para eso.

## Consecuencias

- La app confía en un único archivo, `tls.crt`, y llega a los servicios por `10.0.2.2` desde el emulador.
- Hay que correr el script una vez antes del primer arranque con Compose. Si se regenera el certificado, la app tiene que volver a confiar en el nuevo.
- Un servicio levantado fuera de Compose y sin el perfil habla HTTP: es solo para desarrollo y pruebas, nunca lo que se demuestra.
