# ADR-0076. El JWT se cifra con una clave del Android Keystore, sin librería

- **Estado:** Aceptado
- **Fecha:** 2026-11-17

## Contexto

El JWT de usuario se guarda cifrado con el sistema de claves del dispositivo ([ADR-0049](0049-jwt-cifrado-en-el-dispositivo.md)). Es código de Android, detrás de una interfaz del código común ([ADR-0052](0052-stack-de-la-app.md), [ADR-0072](0072-app-solo-android.md)). Hay que elegir cómo se cifra.

## Decisión

La implementación de Android genera una clave AES de 256 bits dentro del Android Keystore, que nunca sale de él, y cifra el JWT con AES-GCM. En las preferencias privadas de la app queda solo el JWT cifrado, con su vector de inicialización. No se agrega ninguna librería.

Si el JWT guardado no se puede descifrar (por ejemplo, porque se restauró en otro dispositivo, donde la clave no existe), se borra y el usuario vuelve a iniciar sesión.

## Alternativas consideradas

- **`EncryptedSharedPreferences` (androidx.security-crypto).** Descartada: está deprecada, y sumaría una dependencia sin mantenimiento para guardar un solo valor.
- **DataStore con Tink.** Descartada: dos librerías y un mecanismo de serialización para un único texto. El Keystore resuelve lo mismo con las APIs del sistema.

## Consecuencias

- Son unas cincuenta líneas, en un solo archivo del módulo `androidApp`, que se pueden leer completas en la defensa.
- La clave queda atada al dispositivo: un respaldo restaurado en otro equipo no trae una sesión utilizable, y eso es lo esperado.
