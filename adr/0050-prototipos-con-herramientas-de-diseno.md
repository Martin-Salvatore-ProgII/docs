# ADR-0050. Las pantallas se prototipan en un solo lienzo de diseño de Claude antes de implementarlas

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

La organización de la interfaz la define el alumno (ENUNCIADO §10). Las pantallas y su navegación están definidas en [`arq/interfaz.md`](../arq/interfaz.md), pero una tabla no muestra cómo se ven ni si el flujo se entiende.

## Decisión

Antes de implementar cada pantalla, se arma un prototipo visual con las herramientas de diseño de Claude, para acordar disposición y flujo. Se implementan con componentes de Material 3, y se pueden tomar como referencia catálogos de componentes prefabricados.

*Actualización 2026-11-18, con la M3 de la app:* el prototipo de Buscar resultó una guía fiel, así que pasa a ser el estándar para todas las pantallas, no solo las principales:

- **Un solo lienzo para toda la app**, el lienzo de diseño de Claude del proyecto, con una fila por milestone. Cualquier agente que diseñe una pantalla la agrega ahí y no crea otro lienzo.
- **El prototipo usa lo que existe en la app:** los colores del tema, componentes de Material 3 y datos del snapshot, para que se pueda pasar a Compose casi tal cual.
- **El alumno lo aprueba antes de codear.** Al aprobarse, las pantallas se copian a [`arq/prototipo/`](../arq/prototipo/README.md), que se abre sin una cuenta.

## Alternativas consideradas

- **Diseñar directamente en código.** Descartada: cada cambio de disposición cuesta más en código que en un prototipo.
- **Solo catálogos de componentes prefabricados.** Descartada como única fuente: resuelven piezas sueltas, no el flujo entre pantallas.

## Consecuencias

- La disposición y el flujo se acuerdan antes de escribir código.
- El prototipo es una guía visual, no un requisito: si difiere de [`arq/interfaz.md`](../arq/interfaz.md), manda el documento. Ese documento enlaza la copia de [`arq/prototipo/`](../arq/prototipo/README.md).
- Cada pantalla nueva suma un paso antes del código: prototipo, revisión del alumno y copia en `docs`.
- El tema de la app (`core/ui/theme`) define todos los roles de color que usa el prototipo. En la M3, el emulador mostró que un rol sin definir toma el violeta por defecto de Material: por eso se completó la paleta, y una pantalla nueva que use otro rol lo suma al tema.
