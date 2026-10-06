# ADR-0060. El JWT de usuario lleva solo login y roles, se firma con la librería de Spring Security y sus claves nunca están en Git

- **Estado:** Aceptado
- **Fecha:** 2026-10-06

## Contexto

Turnos emite el JWT de usuario y el catálogo lo valida ([ADR-0003](0003-turnos-emite-jwt-usuarios.md)), con un par de claves RSA ([ADR-0032](0032-jwt-firmado-con-par-de-claves.md)) y una vigencia de 24 horas o 30 días ([ADR-0033](0033-vigencia-jwt-usuario.md)). Ni el enunciado ni la REF definen qué lleva el token adentro; solo exigen validar firma, vigencia y permisos, y autorizar según la identidad autenticada (ENUNCIADO §9). Tampoco fijan la librería, ni de dónde salen las claves en cada entorno.

## Decisión

- **Contenido:** el `login` en `sub`, la lista de roles en `auth`, y las fechas de emisión y vencimiento. Nada más. El detalle está en [`arq/seguridad.md`](../arq/seguridad.md#jwt-de-usuario).
- **Librería:** la de JWT de Spring Security, tanto para firmar en turnos como para validar en los dos servicios.
- **Claves:**
  - Turnos recibe solo la clave privada, como archivo en `secrets/` ([ADR-0043](0043-configuracion-externa-env-y-secrets.md)), y calcula la pública a partir de ella.
  - El catálogo recibe solo la clave pública, exportada de ese mismo archivo.
  - Las pruebas, también en CI, generan un par en memoria en cada ejecución.
- **Emisión detrás de un puerto de salida:** los casos de uso no conocen el formato del token. La vigencia se resuelve en el adaptador, a partir de la configuración y de si el usuario pidió `rememberMe`.

## Alternativas consideradas

- **Incluir en el token el UUID del usuario (`externalPatientId`).** Le ahorraría a turnos una consulta a su base en cada reserva. Descartada por ahora: el token lo recibe la app, que no necesita ese dato, y el catálogo tampoco lo usa. Se puede agregar más adelante sin romper nada, porque quien no conoce un claim lo ignora.
- **Los roles como texto separado por espacios**, como JHipster. Descartada: el interior del token lo leen solo nuestros dos servicios; el enunciado pide compatibilidad con el modelo de usuario, no con el formato del token. Una lista se lee sin procesar nada.
- **La librería JJWT.** Simple para firmar, pero la validación en cada pedido habría que escribirla a mano en un filtro propio, en los dos servicios. La de Spring Security ya trae ese filtro.
- **Un par de claves de prueba commiteado.** Descartada: sería una clave privada en Git, aunque no proteja nada real ([Constitución P-08](../constitucion.md)).
- **Configurar la clave pública de turnos por separado.** Descartada: son dos archivos que tienen que coincidir, con una forma más de equivocarse.

## Consecuencias

- El contrato entre turnos y el catálogo sobre el token es chico y está escrito en un solo lugar.
- Turnos consulta su base para obtener el UUID del usuario cuando lo necesita.
- Cambiar la clave privada invalida todos los tokens emitidos.
- No hay ninguna clave en los repositorios, ni de producción ni de prueba.
- Levantar turnos requiere generar antes la clave; el README de cada repo explica cómo.
