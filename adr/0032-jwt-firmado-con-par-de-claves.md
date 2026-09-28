# ADR-0032. El JWT de usuario se firma con un par de claves: turnos firma y el catálogo solo valida

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

El servicio de turnos emite el JWT de usuario y el catálogo tiene que validarlo ([ADR-0003](0003-turnos-emite-jwt-usuarios.md)). En todos los casos hay que validar firma, vigencia y permisos (ENUNCIADO §9).

## Decisión

El JWT se firma con un algoritmo asimétrico (RSA). Turnos tiene la clave privada y firma; el catálogo tiene solo la clave pública y valida. Las claves se generan una vez y se cargan desde configuración externa.

## Alternativas consideradas

- **Clave compartida (HMAC), como JHipster por defecto.** Descartada: con la misma clave el catálogo también podría emitir tokens, y una filtración desde cualquiera de los dos servicios permitiría falsificarlos.

## Consecuencias

- Cada servicio tiene exactamente el poder que necesita: solo turnos puede emitir tokens.
- Una filtración de la clave pública no permite falsificar tokens.
- Hay que generar y distribuir el par de claves; la clave privada es un secreto más ([RNF-02](../requisitos/no-funcionales.md#rnf-02-secretos-en-configuración-externa)).
