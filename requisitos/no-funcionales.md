# Requisitos no funcionales

Requisitos de calidad que no son una funcionalidad concreta pero que el sistema tiene que cumplir en todas sus partes. Cada uno indica cómo se verifica.

## Seguridad

### RNF-01. Contraseñas protegidas

- **Estado:** Propuesto
- **Origen:** ENUNCIADO §3.2, §9

Las contraseñas de los usuarios finales se guardan con un algoritmo de hash criptográfico adecuado para contraseñas, nunca en texto plano ni con un cifrado reversible. No aparecen en respuestas ni en logs.

**Verificación:** test que comprueba que el valor guardado no es la contraseña original y que se valida contra el hash; revisión de logs y respuestas del registro y del login.

### RNF-02. Secretos en configuración externa

- **Estado:** Propuesto
- **Origen:** ENUNCIADO §9; REF §3, §5.1

El JWT técnico, las claves de firma, las credenciales de base de datos y cualquier valor de `integration` se cargan desde configuración externa. No están en el código, en los repositorios, en los logs ni en la app.

**Verificación:** los repos incluyen solo ejemplos con valores ficticios (por ejemplo, `.env.example`); búsqueda de secretos en el historial de Git; revisión de logs durante la demo.

### RNF-03. La identidad sale del token

- **Estado:** Propuesto
- **Origen:** ENUNCIADO §9; REF §6

La identidad del usuario final se obtiene siempre del JWT de usuario validado, y la integración técnica, del JWT técnico. Ningún servicio acepta del cliente un id de usuario o un `groupId` para decidir sobre qué datos opera.

**Verificación:** tests de autorización en los que un usuario intenta operar sobre recursos de otro enviando identificadores ajenos, y se rechaza.

### RNF-04. Endpoints protegidos por defecto

- **Estado:** Propuesto
- **Origen:** ENUNCIADO §9, §11

Todo endpoint de los dos servicios exige un JWT válido, salvo los declarados explícitamente como públicos (registro e inicio de sesión).

**Verificación:** tests que llaman a endpoints protegidos sin token y con un token inválido o vencido, y esperan 401.

### RNF-05. Validación en cada límite

- **Estado:** Propuesto
- **Origen:** ENUNCIADO §9

Los datos que entran al sistema (desde la app, desde el otro servicio y desde la cátedra por REST, Redis o Kafka) se validan antes de usarse. Un dato inválido se rechaza con un error claro y no corrompe el estado local.

**Verificación:** tests con entradas inválidas en cada límite.
