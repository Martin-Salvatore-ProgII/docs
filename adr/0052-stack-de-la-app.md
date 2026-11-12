# ADR-0052. Stack de la app: librerías oficiales, Ktor, kotlinx.serialization y dependencias armadas a mano

- **Estado:** Aceptado
- **Fecha:** 2026-11-12

## Contexto

La app usa Compose Multiplatform ([ADR-0048](0048-compose-multiplatform.md)) y MVVM ([ADR-0051](0051-mvvm-en-la-app.md)). Necesita ViewModels, navegación, un cliente HTTP, serialización JSON y una forma de conectar las piezas entre sí, todo en el código común de KMP. Cualquier tecnología es válida mientras sea defendible ([Constitución P-19](../constitucion.md)).

El alumno no conocía KMP al empezar la app y tiene que defender el código en la evaluación oral. Por eso el criterio, además de que funcione en el código común, es que se pueda seguir leyendo el código: dónde se crea cada objeto y por dónde pasa cada dato, sin mecanismos que actúen por detrás.

## Decisión

| Necesidad | Elección |
| --- | --- |
| ViewModel y ciclo de vida | `androidx.lifecycle` multiplataforma |
| Navegación | `navigation-compose` multiplataforma, con rutas tipadas |
| HTTP | Ktor client |
| JSON | `kotlinx.serialization` |
| Fechas y horas | `kotlinx-datetime` |
| Inyección de dependencias | A mano: un contenedor de la app crea cada objeto con su constructor, y cada pantalla crea su ViewModel con los repositorios del contenedor |
| Código de plataforma | Una interfaz en el código común y su implementación de Android, que se le pasa al contenedor al arrancar. Sin `expect`/`actual` |
| Paginación | A mano, con las cabeceras del contrato. Sin librería de paginado |

## Alternativas consideradas

- **Koin para la inyección de dependencias.** Descartada: funciona en KMP y es liviana, pero agrega su propio lenguaje de configuración y crea los objetos por detrás. Con cinco o seis repositorios, el contenedor a mano son unas pocas decenas de líneas y responde por sí solo "¿dónde se crea esto?". Es la forma que la guía de Android enseña primero.
- **Hilt o Dagger.** Descartados: solo funcionan en Android, no en el código común, y generan código.
- **`expect`/`actual` para el almacenamiento del token.** Descartado: es el mecanismo de KMP para elegir la implementación según la plataforma, y la app tiene una sola ([ADR-0072](0072-app-solo-android.md)). Una interfaz con su implementación es el mismo patrón de puerto y adaptador de los backends.
- **Retrofit y Gson o Moshi para HTTP y JSON.** Descartadas: son solo para JVM y Android, no funcionan en el código común de KMP.
- **Navegación propia con un estado de "pantalla actual".** Descartada: obliga a resolver a mano el botón de atrás y la pila de pantallas, que la librería oficial ya resuelve.

## Consecuencias

- Todo el stack funciona en el código común.
- Cada repositorio o servicio nuevo suma una línea al contenedor de la app.
- La skill `/mvvm-kmp` toma estas librerías y esta forma de inyectar como referencia.
