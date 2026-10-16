# ADR-0062. Las features de un servicio se comunican solo por los puertos de su dominio

- **Estado:** Aceptado
- **Fecha:** 2026-10-16

## Contexto

Cada servicio se organiza en features con sus capas `domain`, `application` e `infrastructure` ([ADR-0040](0040-paquetes-segun-skill-hexagonal.md)). La skill `/hexagonal` define las dependencias entre capas, pero no entre features: el repositorio de referencia de la cátedra tiene una sola.

El caso apareció en el catálogo. La sincronización (feature `sync`) reemplaza la copia local de categorías (feature `category`) y de profesionales y horarios (feature `professional`) en una sola transacción ([ADR-0010](0010-snapshot-en-una-transaccion.md)). Turnos va a tener casos parecidos.

## Decisión

- Una feature puede usar **los puertos y los modelos del `domain`** de otra feature. Son su interfaz pública.
- Una feature **nunca** usa el `application` (casos de uso, fachada, excepciones) ni el `infrastructure` (entities, repositorios JPA, mappers, controllers) de otra.
- Cada tabla tiene un solo adaptador que la escribe: el de la feature dueña del dato.
- Cuando la feature dueña no tiene reglas propias sobre una escritura, como pasa con la copia de los datos de la cátedra, la otra feature usa su **puerto de salida** (su repositorio). Si las tuviera, usaría su **puerto de entrada** (un caso de uso), para no saltearlas.
- `shared` no es una feature: lo transversal se usa desde cualquier lado.
- La regla la verifica `ArchitectureTest` en cada build.

## Alternativas consideradas

- **Un puerto propio en la feature que necesita el dato, implementado con las entities y repositorios JPA de la otra.** Descartada: acopla una infraestructura con otra, y la misma tabla quedaría con dos adaptadores que la escriben.
- **Siempre por el puerto de entrada de la otra feature, aunque no tenga reglas.** Descartada: obliga a crear casos de uso y fachadas que solo pasan los datos al repositorio, sin ninguna lógica.
- **Juntar en una sola feature todo lo que se escribe junto.** Descartada: las features crecerían con cada relación nueva entre datos.

## Consecuencias

- Las dependencias entre features son las mismas que entre capas: siempre hacia interfaces del dominio, nunca hacia implementaciones.
- Una feature puede cambiar su persistencia sin afectar a las que la usan.
- La regla está en la skill `/hexagonal`, así que se aplica igual en los dos servicios.
