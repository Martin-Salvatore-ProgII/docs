# ADR-0044. Pruebas por capa según la skill `/hexagonal`, con la cátedra siempre simulada

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto

Las pruebas de los backends son obligatorias para registro y autenticación, sincronización, reservas, autorización e idempotencia (ENUNCIADO §10.1), y la estrategia la define el alumno (ENUNCIADO §10). La skill `/hexagonal` define herramientas por capa: JUnit y Mockito para casos de uso, Testcontainers para persistencia, WireMock para REST externo y tests web para controllers. La cátedra no provee una suite de pruebas (ENUNCIADO §11). El servicio real de la cátedra cambia sus datos, puede no estar disponible y no permite provocar escenarios.

## Decisión

- Se prueba cada capa con las herramientas de la skill `/hexagonal` ([`arq/pruebas.md`](../arq/pruebas.md)).
- **Ningún test usa el servicio real de la cátedra.** Se simula en dos niveles: como puerto de salida mockeado con Mockito en las pruebas de casos de uso, y como servidor HTTP falso (WireMock) más Kafka y Redis en contenedores, con datos en el formato de la REF, en las pruebas de adaptadores.
- El servicio de catálogo, visto desde turnos, se simula igual.

## Alternativas consideradas

- **Probar contra la cátedra real.** Descartada: los resultados dependerían de datos que cambian y de su disponibilidad, y no se podrían provocar duplicados, discontinuidades, timeouts ni vencimientos.
- **Solo pruebas de extremo a extremo.** Descartada: son lentas, no aíslan la causa de un fallo y no llegan a los casos borde de la lógica.

## Consecuencias

- La lógica de negocio se prueba rápido y aislada, gracias a la separación en puertos de la arquitectura hexagonal.
- Los escenarios difíciles de la integración se prueban a propósito y de forma repetible.
- La cátedra real se reserva para la demo.
