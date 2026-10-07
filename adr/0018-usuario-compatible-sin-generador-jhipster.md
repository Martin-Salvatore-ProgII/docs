# ADR-0018. El usuario se implementa a mano dentro de la hexagonal, compatible con JHipster, sin usar su generador

- **Estado:** Aceptado
- **Fecha:** 2026-09-27

## Contexto

El modelo y el contrato de usuarios tienen que ser compatibles con el usuario generado por JHipster, de modo que quien no use JHipster implemente los mismos datos y comportamientos observables (ENUNCIADO §3.2). JHipster es opcional (ENUNCIADO §3.1). Los backends deben seguir la arquitectura hexagonal de la cátedra ([Constitución P-02](../constitucion.md)), y la estructura que genera JHipster (capas `web/rest`, `service`, `repository`, `domain` con anotaciones JPA) no la respeta.

## Decisión

No se usa el generador de JHipster. El usuario se implementa a mano en el servicio de turnos, dentro de la arquitectura hexagonal, respetando del usuario de JHipster:

- los **datos**: los campos, sus validaciones y las autoridades ([HU-01](../requisitos/HU.md#hu-01-registro-de-usuario-final), [`arq/modelo-db.md`](../arq/modelo-db.md));
- los **comportamientos observables**: registro e inicio de sesión con las mismas rutas y formatos (`/api/register`, `/api/authenticate` con `id_token`), según [`arq/contratos.md`](../arq/contratos.md).

Ante un conflicto entre la estructura de JHipster y la hexagonal, gana la hexagonal; de JHipster se respeta solo la estructura de datos y el contrato.

## Alternativas consideradas

- **Generar el proyecto con JHipster y adaptarlo.** Descartada: habría que refactorizar casi todo el código generado para llevarlo a la hexagonal, y lo que quedara de la estructura original chocaría con P-02.
- **Generar con JHipster solo la gestión de usuarios.** Descartada por la misma razón: la parte generada quedaría fuera de la arquitectura exigida.

## Consecuencias

- Todo el código de los backends sigue un único patrón.
- La compatibilidad se verifica por contrato (campos, validaciones y endpoints), no por el origen del código.
- No requiere una confirmación aparte de la cátedra: el enunciado establece que "el uso de JHipster será opcional, aunque recomendado por la cátedra", y que usarlo o no "no modifica los requisitos funcionales, de integración, seguridad, separación de datos, pruebas y documentación" (ENUNCIADO §3.1). Si la cátedra indicara otra cosa, este ADR se reemplaza.
