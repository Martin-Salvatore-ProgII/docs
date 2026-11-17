# ADR-0074. La app tiene dos módulos: el código común y la app Android

- **Estado:** Aceptado
- **Fecha:** 2026-11-16

## Contexto

La app es Kotlin Multiplatform con Android como único destino ([ADR-0072](0072-app-solo-android.md)), y el código va en el código común ([ADR-0048](0048-compose-multiplatform.md)). La estructura que se planeó era un solo módulo (`composeApp`) que contenía el código común y además era la app Android.

Desde la versión 9 del plugin de Android para Gradle (AGP), el plugin de Kotlin Multiplatform ya no es compatible con el plugin de aplicación Android en el mismo módulo. La estructura oficial de JetBrains para proyectos nuevos separa el código común en una biblioteca KMP y la app Android en su propio módulo.

## Decisión

El proyecto usa la última versión estable de AGP 9 y tiene dos módulos:

- **`composeApp`**: biblioteca KMP con todo el código común (pantallas, ViewModels, dominio, repositorios, red) y sus pruebas. Tiene solo `commonMain` y `commonTest`.
- **`androidApp`**: la app Android. Contiene `MainActivity`, el manifiesto, la configuración de red (confianza en el certificado local, [ADR-0073](0073-certificado-unico-y-https-por-perfil.md)) y las implementaciones Android de las interfaces de plataforma, por ejemplo el almacenamiento cifrado del JWT ([ADR-0049](0049-jwt-cifrado-en-el-dispositivo.md)). Depende de `composeApp`, nunca al revés.

## Alternativas consideradas

- **Un solo módulo, con AGP 8.** Descartada: se arrancaría un proyecto nuevo sobre una versión de AGP que ya no es la vigente, solo para conservar la estructura planeada.

## Consecuencias

- La regla "lo de Android solo donde la plataforma lo exige" queda garantizada por el build: el código común no puede depender de clases de Android, porque su módulo no las tiene.
- Las interfaces de plataforma se definen en `composeApp` y se implementan en `androidApp`. `MainActivity` crea esas implementaciones y se las pasa al contenedor de la app ([ADR-0052](0052-stack-de-la-app.md)).
- La app se compila con `./gradlew :androidApp:assembleDebug`. Las pruebas del código común se corren desde `composeApp`.
- La skill `/mvvm-kmp` describe esta estructura de dos módulos.
