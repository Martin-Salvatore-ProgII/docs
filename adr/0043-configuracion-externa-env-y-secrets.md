# ADR-0043. Configuración externa con `.env` y archivos secretos montados, ambos fuera de Git

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Credenciales, tokens y secretos se externalizan y nunca se commitean ([Constitución P-08](../constitucion.md), [RNF-02](../requisitos/no-funcionales.md#rnf-02-secretos-en-configuración-externa)). Además de valores simples (URL de la cátedra, JWT técnico, credenciales de base), hay secretos en archivo: el par de claves del JWT de usuario ([ADR-0032](0032-jwt-firmado-con-par-de-claves.md)) y el certificado TLS ([ADR-0035](0035-https-entre-app-y-backends.md)).

## Decisión

- Los valores simples van en un archivo `.env` por repo, ignorado por Git. Se commitea un `.env.example` con todas las variables y valores ficticios.
- Los archivos secretos van en una carpeta `secrets/` ignorada por Git y se montan en los contenedores.
- Los valores operativos configurables ([RNF-09](../requisitos/no-funcionales.md#rnf-09-sin-supuestos-fijos-sobre-los-datos-de-la-cátedra-y-valores-operativos-configurables)) tienen su valor inicial en la configuración de la aplicación y se pueden sobrescribir con variables de entorno.

## Alternativas consideradas

- **Un gestor de secretos (por ejemplo, Vault).** Descartada: agrega un servicio de infraestructura para un entorno local de desarrollo y demo.

## Consecuencias

- El `.env.example` documenta qué hay que configurar sin exponer nada.
- El README de cada repo explica cómo generar los secretos locales (claves y certificado).
