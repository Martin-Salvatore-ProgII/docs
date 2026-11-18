# ADR-0078. La app pide el permiso de red local de Android 17

- **Estado:** Aceptado
- **Fecha:** 2026-11-18

## Contexto

La app se compila para la API 37 (Android 17) y habla con los backends en la máquina de desarrollo: `10.0.2.2` desde el emulador, o una IP de la red Wi-Fi desde un teléfono ([ADR-0075](0075-configuracion-de-red-de-la-app.md)). Desde Android 17, una app que apunta a la API 37 necesita el permiso `ACCESS_LOCAL_NETWORK` para conectarse a direcciones de la red local. Sin él, Android descarta la conexión sin avisar, y la app solo ve un timeout. El permiso es de los que el usuario otorga en tiempo de ejecución, y el sistema lo muestra dentro del grupo de "dispositivos cercanos".

## Decisión

La app declara `ACCESS_LOCAL_NETWORK` y lo pide al abrirse, solo en Android 17 o posterior, desde `MainActivity` (módulo `androidApp`, [ADR-0074](0074-app-en-dos-modulos-con-agp-9.md)). Si el usuario lo rechaza, la app abre igual y cada pedido se informa como que no se pudo conectar.

## Alternativas consideradas

- **Bajar `targetSdk` a 36.** Descartada: esquiva una protección del sistema en lugar de cumplirla, y obligaría a explicar por qué la app apunta a una versión vieja.
- **`adb reverse` y `localhost`.** Descartada como solución: `localhost` no cuenta como red local y no necesita el permiso, pero hay que repetir `adb reverse` en cada arranque del emulador y no sirve con un teléfono físico. Sigue siendo posible, porque la configuración de red ya confía en el certificado para `localhost`.

## Consecuencias

- La primera vez que se abre, la app muestra el diálogo del sistema antes del Login.
- En la demo hay que aceptar el permiso; si se rechazó, se vuelve a habilitar desde los ajustes de la app.
