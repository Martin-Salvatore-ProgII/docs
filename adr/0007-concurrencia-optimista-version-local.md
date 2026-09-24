# ADR-0007. Sincronizaciones simultáneas: una versión solo se aplica sobre la versión local esperada

- **Estado:** Aceptado
- **Fecha:** 2026-09-24

## Contexto

Con varios disparadores ([ADR-0006](0006-disparadores-sincronizacion.md)) pueden correr dos ciclos de sincronización a la vez, por ejemplo un aviso de Kafka y el chequeo periódico. Los dos podrían intentar aplicar la misma versión o reemplazar la copia al mismo tiempo.

## Decisión

Guardar una versión (incremental o snapshot) solo tiene efecto si la versión local sigue siendo la que se leyó al empezar. La condición se verifica en la base, dentro de la misma transacción en la que se guardan los datos. Si otro ciclo ya la cambió, la transacción no aplica nada y ese ciclo termina sin error.

## Alternativas consideradas

- **Un lock en memoria para que solo corra una sincronización a la vez.** Descartada: solo protege dentro de una instancia del proceso y agrega coordinación explícita que la regla de versión ya resuelve.
- **Un lock explícito en la base durante todo el ciclo.** Descartada: bloquea durante las lecturas a Redis y a la API, que pueden tardar, sin aportar más garantías que la condición sobre la versión.

## Consecuencias

- No hay coordinación entre hilos ni instancias: la protección sale de la misma regla que ya asegura que la versión solo avanza con datos completos.
- Sigue siendo correcto aunque corran dos instancias del servicio.
- Dos ciclos simultáneos pueden hacer lecturas duplicadas a Redis; es un costo menor.
