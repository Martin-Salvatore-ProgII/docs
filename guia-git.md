# Flujo de trabajo con Git

Cómo se trabaja en los tres repositorios de código (`app-kmp`, `catalogo-service` y `turnos-service`). El objetivo es que el historial muestre la evolución del trabajo y permita seguir cada cambio desde el requisito hasta el código (ENUNCIADO §13.1; [Constitución P-12](constitucion.md)).

## Reglas

1. **`main` está protegida en los repos de código.** Nadie commitea directo a `main`: todo entra por pull request.
2. **Toda unidad de trabajo tiene una issue.** Antes de empezar se crea la issue en el repo donde se hace el cambio.
3. **Una rama por issue**, creada desde `main` actualizada.
4. **Un PR por rama**, que cierra su issue al mergearse.
5. **El merge conserva los commits** (merge commit, sin squash), para que el historial muestre cómo se construyó cada cambio.
6. **Las pruebas pasan en CI** antes de mergear: el workflow de cada repo es un check obligatorio del ruleset ([ADR-0046](adr/0046-ci-con-github-actions.md)).

**Excepción: el repo `docs`.** No tiene `main` protegida y se commitea directo, con Conventional Commits. Es documentación de trabajo propia y no forma parte de la entrega evaluada, así que no necesita el flujo de issues y PR.

## Relación con SDD

| SDD | Git |
| --- | --- |
| Tareas de una feature (`specs/NNN-nombre.md`) | Se commitea directo en `docs` |
| Cada tarea de ese archivo | Una issue en el repo de código correspondiente, con su rama y su PR |
| Decisión nueva (ADR) | Se commitea directo en `docs`, antes de implementarla |

Si una tarea toca más de un repo (por ejemplo, un contrato entre servicios), se abre una issue en cada repo y se enlazan entre sí.

## Nombres

**Ramas:** `<tipo>/<número-de-issue>-<descripción-corta>`, en minúsculas y con guiones.

```
feat/12-registro-de-usuarios
fix/31-offset-kafka-tras-falla
docs/4-flujo-de-reserva
```

**Commits:** [Conventional Commits](https://www.conventionalcommits.org/). El tipo y los términos técnicos estándar van en inglés; el asunto, en español, breve y en imperativo. Cada commit tiene un solo cambio lógico.

```
feat(sync): agregar consumidor de CatalogUpdated
fix(auth): rechazar tokens vencidos en el catálogo
docs(adr): registrar decisión sobre zona horaria
```

Tipos: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `build`, `ci`. El ámbito es opcional.

## Contenido de issues y PR

En español, explicativos y breves.

**Issue:** qué se necesita y por qué, con referencia a la HU, CU, ADR o spec correspondiente, y los criterios para darla por terminada.

**PR:**

- **Contexto:** qué problema resuelve. Enlaza la issue (`Closes #N`) y cita el requisito (HU, spec, ENUNCIADO o REF).
- **Qué se hizo y por qué:** las decisiones tomadas y las alternativas descartadas, si no están ya en un ADR.
- **Cómo probarlo:** los tests que lo cubren y los pasos para verificarlo a mano, si hace falta.

## Secretos

Ningún commit, issue ni PR contiene el JWT técnico, credenciales, hosts de la cátedra ni valores de `integration` ([Constitución P-08](constitucion.md)). Si un secreto se commitea por error, borrarlo en un commit nuevo no alcanza, porque queda en el historial: hay que tratarlo como expuesto (REF §5.4).
