# ADR-0064. Una feature lee los datos de otra por su puerto, no con un JOIN a su tabla

- **Estado:** Aceptado
- **Fecha:** 2026-10-20

## Contexto

[ADR-0062](0062-features-se-comunican-por-puertos-de-dominio.md) fija que las features se comunican por los puertos de su dominio y que cada tabla tiene un solo adaptador que la escribe, pero no dice nada sobre las lecturas.

El caso apareció en la búsqueda de profesionales del catálogo (HU-05). Cada resultado lleva el nombre de su categoría, y el filtro de habilitado depende de si la categoría está habilitada ([ADR-0014](0014-habilitado-efectivo.md)). Las categorías son de la feature `category`. En la base, la forma más directa es un JOIN entre las dos tablas, desde el adaptador de `professional`.

## Decisión

- Cuando una feature necesita datos de otra, los lee con **el puerto de la feature dueña**, igual que para escribir. El adaptador de una feature consulta solo las tablas de su feature.
- Si hacen falta varias lecturas para responder una operación, se hacen dentro de una misma transacción con un aislamiento que garantice que todas ven el mismo estado de la base. Así una sincronización que se aplica en el medio no mezcla versiones ([ADR-0010](0010-snapshot-en-una-transaccion.md)).
- La regla de negocio que combina los datos de las dos features (por ejemplo, el habilitado efectivo) queda en el dominio de la feature que la necesita, no escondida en una consulta.

## Alternativas consideradas

- **Un JOIN a la tabla de la otra feature desde el adaptador.** Descartada: es una sola consulta y es consistente por sí misma, pero el adaptador depende del esquema de otra feature, y un cambio en esa tabla rompe una feature que no es la dueña. Además, la regla que combina los datos queda dentro del SQL, donde no se ve desde el dominio.
- **Una vista de base de datos con los datos de las dos features.** Descartada: tiene el mismo acoplamiento de esquema que el JOIN, solo que en otro lugar.

## Consecuencias

- La regla de dependencias es la misma para leer y para escribir, y se explica de una sola manera.
- Una operación que combina datos de dos features hace más de una lectura y necesita una transacción que las mantenga consistentes.
- Sirve cuando los datos de la otra feature que hay que leer son pocos o se pueden acotar. Si no lo fueran, se revisa esta decisión con un ADR nuevo.
