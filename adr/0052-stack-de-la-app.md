# ADR-0052. Stack de la app: lifecycle y navegación multiplataforma, Ktor, kotlinx.serialization y Koin

- **Estado:** Propuesto
- **Fecha:** 2026-09-29

## Contexto

La app usa Compose Multiplatform ([ADR-0048](0048-compose-multiplatform.md)) y MVVM ([ADR-0051](0051-mvvm-en-la-app.md)). Necesita ViewModels, navegación, un cliente HTTP, serialización JSON e inyección de dependencias que funcionen en el código común de KMP. Cualquier tecnología es válida mientras sea defendible ([Constitución P-19](../constitucion.md)).

## Decisión

| Necesidad | Elección |
| --- | --- |
| ViewModel y ciclo de vida | `androidx.lifecycle` multiplataforma |
| Navegación | `navigation-compose` multiplataforma, con rutas tipadas |
| HTTP | Ktor client |
| JSON | `kotlinx.serialization` |
| Inyección de dependencias | Koin |
| Fechas y horas | `kotlinx-datetime` |

## Alternativas consideradas

- **Retrofit y Gson o Moshi para HTTP y JSON.** Descartadas: son solo para JVM y Android, no funcionan en el código común de KMP.
- **Inyección manual (construir las dependencias a mano).** Viable para una app chica, pero descartada: con siete features y sus ViewModels, el cableado a mano crece y se vuelve propenso a errores. Koin es liviano y funciona en KMP.
- **Hilt o Dagger.** Descartados: solo funcionan en Android.

## Consecuencias

- Todo el stack funciona en el código común; una segunda plataforma no requiere reemplazar librerías.
- Son las librerías que la skill `/mvvm-kmp` toma como referencia.
