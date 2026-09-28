# ADR-0050. Las pantallas principales se prototipan con las herramientas de diseño de Claude antes de implementarlas

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

La organización de la interfaz la define el alumno (ENUNCIADO §10). Las pantallas y su navegación están definidas en [`arq/interfaz.md`](../arq/interfaz.md), pero una tabla no muestra cómo se ven ni si el flujo se entiende.

## Decisión

Antes de implementar las pantallas principales (Buscar, Turnos disponibles, Reserva en curso y Mis reservas), se arma un prototipo visual con las herramientas de diseño de Claude, para acordar disposición y flujo. Se implementan con componentes de Material 3, y se pueden tomar como referencia catálogos de componentes prefabricados.

## Alternativas consideradas

- **Diseñar directamente en código.** Descartada: cada cambio de disposición cuesta más en código que en un prototipo.
- **Solo catálogos de componentes prefabricados.** Descartada como única fuente: resuelven piezas sueltas, no el flujo entre pantallas.

## Consecuencias

- La disposición y el flujo se acuerdan antes de escribir código.
- El prototipo es una guía visual, no un requisito: si difiere de [`arq/interfaz.md`](../arq/interfaz.md), manda el documento. El enlace al prototipo se agrega a ese documento cuando exista.
