# ADR-0057. Las contraseñas se guardan con BCrypt y se limitan a 72 bytes

- **Estado:** Aceptado
- **Fecha:** 2026-10-05

## Contexto

Las contraseñas se guardan con un hash criptográfico adecuado, nunca en texto plano ([RNF-01](../requisitos/no-funcionales.md#rnf-01-contraseñas-protegidas); ENUNCIADO §3.2, §9). El enunciado no fija el algoritmo. El registro debe respetar las validaciones de la cátedra, que para la contraseña son de 4 a 100 caracteres (ENUNCIADO §3.2; REF §5.1).

BCrypt solo procesa los primeros 72 bytes de una contraseña. La versión de Spring Security que usa el proyecto no recorta en silencio: rechaza con un error una contraseña más larga.

## Decisión

- Las contraseñas se guardan con **BCrypt**, el algoritmo que usa JHipster.
- El máximo de la contraseña es de **72 bytes**, no de 100 caracteres. Una contraseña más larga se rechaza en el registro con `VALIDATION_ERROR`. Se mide en bytes: una letra con tilde o una `ñ` ocupan dos.
- El hasheo queda detrás de un puerto de salida, así que los casos de uso no dependen del algoritmo.

## Alternativas consideradas

- **Aplicar SHA-256 a la contraseña antes de BCrypt.** Permite cualquier largo y mantiene los 100 caracteres de la cátedra. Descartada: agrega un paso que hay que saber justificar y tiene riesgos propios si se implementa mal.
- **PBKDF2.** No tiene límite de largo y no requiere dependencias nuevas. Descartada: deja de ser el algoritmo de JHipster, cuyo modelo de usuario es la referencia que pide el enunciado.
- **Argon2.** Es más moderno y resistente. Descartada: agrega una dependencia y parámetros de configuración que no aportan al alcance del proyecto.
- **Recortar en silencio a 72 bytes.** Descartada: dos contraseñas distintas con el mismo comienzo serían equivalentes sin que el usuario lo sepa.

## Consecuencias

- El límite de la contraseña se aparta del máximo de la cátedra: una contraseña de 73 a 100 caracteres, válida para la cátedra, se rechaza en nuestro registro. Es una diferencia conocida, con una razón técnica concreta.
- El límite está a la vista en la validación y en [HU-01](../requisitos/HU.md#hu-01-registro-de-usuario-final), no escondido en el algoritmo.
- Cambiar de algoritmo en el futuro toca solo al adaptador de hasheo; las contraseñas ya guardadas seguirían necesitando BCrypt para verificarse.
