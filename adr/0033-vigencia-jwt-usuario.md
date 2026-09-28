# ADR-0033. Vigencia del JWT de usuario: 24 horas, o 30 días con `rememberMe`, sin refresh tokens

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

El JWT de usuario se usa durante la sesión en la app (ENUNCIADO §3.2). El contrato de inicio de sesión es compatible con JHipster e incluye `rememberMe` ([ADR-0018](0018-usuario-compatible-sin-generador-jhipster.md)).

## Decisión

El JWT vence a las 24 horas, o a los 30 días si se inicia sesión con `rememberMe`, como en JHipster. Ambos valores son configurables. Al vencer, el usuario vuelve a iniciar sesión.

## Alternativas consideradas

- **Tokens cortos con refresh tokens.** Descartada: agrega un flujo completo (emisión, almacenamiento y revocación de refresh tokens) que el enunciado no pide.

## Consecuencias

- Mantiene el comportamiento observable de JHipster.
- Un token robado vale hasta su vencimiento; no hay revocación anticipada. Es una limitación documentada.
