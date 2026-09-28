# ADR-0036. CORS cerrado por defecto: ningún origen web permitido

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Hay que configurar CORS de acuerdo con los clientes permitidos (ENUNCIADO §9). CORS es un mecanismo de los navegadores: controla qué páginas web de otros orígenes pueden llamar a la API. El único cliente del sistema es la app Android nativa, que no está sujeta a CORS.

## Decisión

Ningún origen web está permitido por defecto. La lista de orígenes permitidos es configurable y está vacía.

## Alternativas consideradas

- **Permitir todos los orígenes.** Descartada: habilita que cualquier página web llame a la API desde el navegador de un usuario.

## Consecuencias

- No hay clientes web habilitados, porque no existen.
- Si en el futuro se agrega un cliente web (por ejemplo, otra plataforma KMP), se agrega su origen a la lista.
