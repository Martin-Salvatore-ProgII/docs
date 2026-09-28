# ADR-0046. Integración continua con GitHub Actions como check obligatorio para mergear

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

`main` está protegida y todo entra por PR ([`guia-git.md`](../guia-git.md)). Las pruebas son obligatorias (ENUNCIADO §10.1) y el historial es parte de la evaluación (ENUNCIADO §13.1).

## Decisión

Cada repo de código tiene un workflow de GitHub Actions que compila y ejecuta todas sus pruebas en cada PR. El workflow se agrega como check obligatorio en el ruleset de `main`.

## Alternativas consideradas

- **Correr las pruebas solo en la máquina local.** Descartada: nada impide mergear un cambio que rompe las pruebas, y no queda evidencia de que cada PR las pasó.

## Consecuencias

- Un PR con pruebas rotas no se puede mergear.
- Cada PR muestra que pasó las pruebas, lo que suma evidencia al historial.
- El check obligatorio se agrega al ruleset cuando el workflow exista en cada repo.
