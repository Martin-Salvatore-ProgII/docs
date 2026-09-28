# ADR-0034. Turnos se autentica ante el catálogo propagando el JWT del usuario

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

La comunicación entre servicios debe usar y validar JWT, y hay que justificar si se propaga el JWT del usuario o se usa uno técnico, preservando identidad, autorización y trazabilidad (ENUNCIADO §9). Hoy todas las llamadas de turnos al catálogo (agenda vigente para disponibilidad y reserva) nacen de una acción del usuario; la reconciliación no usa el catálogo ([ADR-0030](0030-reconciliacion-periodica-de-procesos.md)).

## Decisión

Turnos reenvía al catálogo el mismo JWT de usuario que recibió de la app. El catálogo lo valida igual que cuando llama la app: firma, vigencia y rol.

## Alternativas consideradas

- **Un JWT técnico propio de turnos.** Descartada por ahora: pierde la identidad del usuario en el catálogo y agrega un segundo tipo de token, sin que haya ninguna llamada sin usuario que lo requiera.

## Consecuencias

- El catálogo sabe para qué usuario se hizo cada consulta: identidad y trazabilidad de punta a punta.
- La agenda vigente no requiere permisos especiales: la app también puede pedirla.
- Si en el futuro aparece una llamada sin usuario (por ejemplo, una tarea de fondo), se agrega un token de servicio con un nuevo ADR.
- Si el token vence justo entre la llamada de la app y la del catálogo, la operación falla con 401 y el usuario vuelve a iniciar sesión.
