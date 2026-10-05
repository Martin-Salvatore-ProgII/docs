# ADR-0055. Convenciones comunes del esqueleto de los dos backends

- **Estado:** Aceptado
- **Fecha:** 2026-10-04

## Contexto

Los dos backends se crean por separado, cada uno en su repo, con la misma arquitectura ([ADR-0040](0040-paquetes-segun-skill-hexagonal.md)) y las mismas herramientas ([ADR-0038](0038-gradle-java-spring-boot.md), [ADR-0039](0039-flyway.md), [ADR-0043](0043-configuracion-externa-env-y-secrets.md)). Al armar el esqueleto del catálogo se tomaron decisiones chicas que esos ADR no cubren. Si el servicio de turnos las resuelve distinto, los dos repos quedan con diferencias que no responden a ninguna razón y que hay que explicar.

## Decisión

Los dos backends siguen estas convenciones:

- **Configuración en `application.yaml`**, no en `.properties`.
- **Actuator con solo el endpoint `health` expuesto**, que es el valor por defecto de Spring Boot. Es el punto de salud del servicio.
- **La conexión a la base llega por tres variables: `DB_URL`, `DB_USER` y `DB_PASSWORD`, sin valor por defecto.** Si falta alguna, el servicio no arranca.
- **Un `package-info.java` en cada paquete de la arquitectura**, que dice qué va ahí y qué no puede importar.
- **Cada dependencia entra al build en la issue que la usa por primera vez**, no todas al crear el proyecto.

## Alternativas consideradas

- **`application.properties`.** Es lo que genera Spring Initializr por defecto. Descartada: la configuración crece con base de datos, Redis, Kafka y valores operativos, y en YAML se lee agrupada, sin repetir prefijos.
- **Sin Actuator.** Descartada: no habría una forma estándar de saber si el servicio está en pie, ni para Docker Compose ni para una prueba rápida.
- **Valores por defecto de desarrollo para la base.** Descartada: dejaría una clave en el repo, aunque sea de desarrollo, y habría que justificarla frente a la [Constitución P-08](../constitucion.md). Para desarrollo local alcanza con el `.env` o con la base descartable de las pruebas.
- **Archivos de clase vacíos para marcar la estructura.** Descartada: fijan nombres de clases antes de discutir la feature y no explican nada. Git no versiona carpetas vacías, así que algún archivo hace falta; el `package-info.java` además documenta la capa.
- **Todas las dependencias desde el primer commit.** Descartada: el servicio no arrancaría sin configurar piezas que todavía no existen, y el historial no mostraría para qué se sumó cada una.

## Consecuencias

- Quien conoce un backend encuentra lo mismo en el otro.
- Cada paquete se explica solo, para una persona o para un agente de IA que llega sin contexto.
- El historial de cada repo muestra cuándo y por qué entró cada dependencia (ENUNCIADO §13.1).
- Exponer más endpoints de Actuator requiere una decisión explícita, porque pueden mostrar configuración interna.
