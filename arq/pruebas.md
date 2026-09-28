# Estrategia de pruebas

Qué se prueba, en qué nivel y con qué criterio (ENUNCIADO §10, §10.1, §11). Las herramientas por capa siguen la sección de pruebas de la skill `/hexagonal`. Cómo ejecutar las pruebas de cada repo está en su README.

## Qué exige el enunciado

Las pruebas automatizadas de los backends son **obligatorias** y tienen que cubrir, como mínimo, registro y autenticación, sincronización, reservas, autorización e idempotencia (ENUNCIADO §10.1). No se exige un porcentaje de cobertura. Las pruebas de la app son **opcionales** y se consideran trabajo complementario.

Por eso las pruebas tienen dos calidades distintas, a propósito:

| | Backends | App KMP |
| --- | --- | --- |
| Carácter | Obligatorias y evaluadas | Complementarias |
| Alcance | Cada comportamiento fundamental, en todas las capas | Pocas y básicas: lógica pura de la app |
| Criterio | Cada criterio de aceptación y cada situación de robustez tiene al menos un test | Las reglas que la app aplica por su cuenta |
| En CI | Sí, obligatorio para mergear | Sí, obligatorio para mergear |

## Backends: niveles por capa

([ADR-0044](../adr/0044-pruebas-por-capa-con-catedra-simulada.md))

| Capa | Qué se prueba | Cómo |
| --- | --- | --- |
| **Dominio y casos de uso** | Las reglas: máquina de estados, decisiones de sincronización, generación de slots, idempotencia, propiedad de las reservas | JUnit y Mockito, sin Spring. Los puertos de salida (y detrás de ellos la cátedra, el catálogo, la base y Kafka) se reemplazan por mocks |
| **Fachada de servicio** | La traducción de resultados vacíos a errores del dominio | Mockito sobre los puertos de entrada |
| **Adaptadores de persistencia** | El mapeo ida y vuelta, las restricciones y las transacciones | PostgreSQL real con Testcontainers ([ADR-0045](../adr/0045-postgresql-real-en-tests.md)) |
| **Adaptadores externos** | Clientes REST de la cátedra y del catálogo, lectura de Redis, consumidores y productores Kafka | La cátedra simulada (ver abajo); Kafka y Redis reales con Testcontainers |
| **Controllers y seguridad** | Validaciones, códigos HTTP, `problem+json`, 401, 403, 404 de reservas ajenas y conformidad con el OpenAPI | Tests web con la fachada mockeada y la configuración de seguridad real |

## La cátedra simulada

Ningún test habla con el servicio real de la cátedra ([ADR-0044](../adr/0044-pruebas-por-capa-con-catedra-simulada.md)). La cátedra cambia sus datos, puede no estar disponible y no permite provocar escenarios; un test tiene que dar siempre el mismo resultado. La cátedra real se usa en la demo.

La cátedra se simula en dos niveles:

1. **Como puerto mockeado**, en las pruebas de casos de uso. La lógica de negocio no sabe que existe una cátedra: habla con puertos de salida, y en el test esos puertos son mocks de Mockito que responden lo que el escenario necesita (un hold creado, un `SLOT_ALREADY_HELD`, un timeout, un evento duplicado).
2. **Como servidor falso**, en las pruebas de adaptadores. Un servidor HTTP de prueba (WireMock) responde como la API de la cátedra según la REF, incluidos sus errores `problem+json` y sus demoras. Para Kafka y Redis, el test publica los eventos y carga las claves con el formato de la REF §14 y §15, sobre contenedores reales.

Con la cátedra simulada se prueban a propósito los casos que en la realidad son difíciles de provocar: discontinuidades, eventos duplicados o fuera de orden, timeouts, vencimientos y respuestas perdidas.

El servicio de catálogo, visto desde turnos, se simula de la misma forma.

## Reglas

- **Nada de H2 ni bases en memoria:** PostgreSQL real en los tests ([ADR-0045](../adr/0045-postgresql-real-en-tests.md)).
- **Tests deterministas:** el tiempo se controla en los tests (vencimientos, "hoy" y "ahora" en hora de Argentina), sin esperas reales ni dependencia de la hora de la máquina.
- **Datos variados:** los catálogos de prueba incluyen distintas duraciones de slot, varios horarios por día y entidades deshabilitadas ([RNF-09](../requisitos/no-funcionales.md#rnf-09-sin-supuestos-fijos-sobre-los-datos-de-la-cátedra-y-valores-operativos-configurables)).
- **Sin secretos reales:** los tests usan claves, certificados y tokens generados para las pruebas.

## Criterio de cobertura

Sin porcentaje, pero verificable:

- **Cada criterio de aceptación** de las HU obligatorias tiene al menos un test.
- **Cada situación** de [`robustez.md`](robustez.md) tiene al menos un test.
- **Cada RNF** tiene los tests que indica su verificación.

La columna *Tests* de [`trazabilidad.md`](../trazabilidad.md) registra qué test cubre cada cosa.

| Área obligatoria (ENUNCIADO §10.1) | Qué cubre, como mínimo |
| --- | --- |
| Registro y autenticación | HU-01, HU-02, RNF-01 |
| Sincronización | HU-04, CU-01, CU-02 y las situaciones del catálogo en `robustez.md` |
| Reservas | HU-06 a HU-09, CU-03 a CU-05 y cada transición de [`maquina-estados.md`](maquina-estados.md) |
| Autorización | RNF-03, RNF-04, HU-08 y HU-09 (acceso cruzado) y HU-10 (403 en rutas administrativas) |
| Idempotencia | RNF-06 y los duplicados en catálogo y reservas |

## Integración continua

([ADR-0046](../adr/0046-ci-con-github-actions.md))

En cada PR de un repo de código se ejecutan todas sus pruebas con GitHub Actions. El resultado es un check obligatorio del ruleset de `main`: un PR con tests rotos no se puede mergear.

## App KMP

([ADR-0047](../adr/0047-tests-basicos-en-la-app.md))

Pocas pruebas, básicas y sin emulador ni pruebas de interfaz, sobre la lógica que la app aplica por su cuenta:

- normalización y validación del teléfono antes de enviarlo;
- qué muestra la app para cada estado de una reserva;
- interpretación de los errores `problem+json` por `status` y `code`;
- validaciones de los formularios de registro y búsqueda.
