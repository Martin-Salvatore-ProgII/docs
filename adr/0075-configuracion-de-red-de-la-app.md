# ADR-0075. Configuración de red de la app: URL en `BuildConfig`, Ktor sobre OkHttp y el certificado copiado al compilar

- **Estado:** Aceptado
- **Fecha:** 2026-11-16

## Contexto

La app habla con los dos backends ([ADR-0001](0001-app-habla-con-ambos-servicios.md)) con Ktor ([ADR-0052](0052-stack-de-la-app.md)). Sus URL tienen que salir de la configuración del build, nunca del código. Los backends levantados con Compose hablan solo HTTPS con un certificado autofirmado, `tls.crt`, que se genera en cada máquina y no se commitea ([ADR-0073](0073-certificado-unico-y-https-por-perfil.md)). La app tiene que confiar en ese certificado y solo en él para esos hosts. Lo de Android va en el módulo `androidApp` ([ADR-0074](0074-app-en-dos-modulos-con-agp-9.md)).

## Decisión

- **Las URL de los dos servicios son campos de `BuildConfig`** del módulo `androidApp`, que se leen de `local.properties` y tienen valores por defecto para el emulador: `https://10.0.2.2:8080` (turnos) y `https://10.0.2.2:8081` (catálogo), los puertos del [ADR-0056](0056-imagen-con-dockerfile-y-puertos.md). `MainActivity` arma con ellas un objeto `ServiceUrls` y se lo pasa al contenedor de la app.
- **Ktor usa el motor OkHttp.** Lo crea `androidApp` y se lo pasa al contenedor; el código común solo conoce la API de Ktor.
- **Los timeouts son de 5 segundos para conectar y 60 para la respuesta** (tabla de RNF-09). La respuesta tiene que cubrir el peor caso de iniciar una reserva, para que la app no dé por fallada una operación que el backend todavía está haciendo.
- **La confianza en el certificado es configuración de red de Android** (`network_security_config`): para `10.0.2.2` y `localhost`, la única autoridad aceptada es `tls.crt`. Para el resto de los hosts rigen las del sistema.
- **Gradle copia `tls.crt` a los recursos de la app al compilar**, desde `../turnos-service/secrets/tls.crt` o desde la ruta que indique `local.properties`. Si el archivo falta o no es un certificado PEM, el build falla con un mensaje que explica cómo generarlo. El CI genera un certificado descartable antes de compilar.

## Alternativas consideradas

- **Generar las URL en el código común con un plugin (BuildKonfig).** Descartada: suma una librería para algo que Android ya resuelve con `BuildConfig`, que es lo estándar en una app con una sola plataforma.
- **Motor CIO de Ktor.** Descartada: no usa la configuración de red de Android, y la confianza en el certificado habría que programarla a mano.
- **Copiar `tls.crt` a mano dentro de la app.** Descartada: es fácil olvidarlo al regenerar el certificado, y entonces la app deja de conectar sin un error claro.
- **Compilar sin el certificado si falta.** Descartada: la app compilaría bien pero no podría hablar con ningún backend.

## Consecuencias

- Cambiar de backend (otra máquina, un dispositivo físico) es editar `local.properties` y recompilar, sin tocar código.
- Si se regenera el certificado, alcanza con recompilar la app.
- Antes de compilar la app por primera vez hay que generar el certificado con el script de `turnos-service`.
