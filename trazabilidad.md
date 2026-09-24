# Trazabilidad

Relación entre lo que exige el enunciado, dónde lo especificamos, dónde se implementa y prueba, y cómo se demuestra. Es el checklist de aprobación: una fila sin completar es algo que falta.

**Estados:** ⬜ pendiente · 🟨 especificado · 🟦 implementado · ✅ probado y demostrable

## Alcance funcional (ENUNCIADO §5)

| § 5 | Requisito | Especificación | Spec | Tests | Estado |
| --- | --- | --- | --- | --- | --- |
| 1 | Registrar y autenticar usuarios finales | HU-01, HU-02, RNF-01, RNF-04 | | | 🟨 |
| 2 | Registrar la cuenta técnica una vez | HU-03 | | | 🟨 |
| 3 | Autenticar la integración y recuperar su configuración | HU-03, ADR-0005 | | | 🟨 |
| 4 | Obtener y guardar un snapshot completo | | | | ⬜ |
| 5 | Mantener el catálogo actualizado con Kafka y Redis | | | | ⬜ |
| 6 | Detectar discontinuidad y reconstruir | | | | ⬜ |
| 7 | Buscar y filtrar con la copia local | | | | ⬜ |
| 8 | Construir y mostrar turnos disponibles | | | | ⬜ |
| 9 | Solicitar un hold | | | | ⬜ |
| 10 | Iniciar la confirmación por REST | | | | ⬜ |
| 11 | Recibir el pedido de información y responder por Kafka | | | | ⬜ |
| 12 | Reflejar el resultado final localmente | | | | ⬜ |
| 13 | Consultar las reservas propias | | | | ⬜ |
| 14 | Cancelar una reserva propia confirmada | | | | ⬜ |
| 15 | Informar errores de integración y recuperarse | | | | ⬜ |

## Evidencias (ENUNCIADO §11)

| Evidencia | Especificación | Cómo se demuestra | Estado |
| --- | --- | --- | --- |
| Inicialización desde snapshot | | | ⬜ |
| Actualización incremental | | | ⬜ |
| Recuperación por snapshot tras una discontinuidad | | | ⬜ |
| Búsquedas sobre datos locales | | | ⬜ |
| Filtros por categoría, nombre, habilitado y disponibilidad | | | ⬜ |
| Reserva confirmada por REST y Kafka | | | ⬜ |
| Reserva que expira sin completar la información | | | ⬜ |
| Consulta y cancelación propia sin acceso cruzado | RNF-03 | | ⬜ |
| Procesamiento idempotente de un mensaje duplicado | | | ⬜ |
| Separación efectiva de datos entre servicios | Constitución P-03, ADR-0002 | | ⬜ |
| Rechazo de accesos no autenticados o no autorizados | HU-02, RNF-03, RNF-04 | | 🟨 |
| Tests automatizados de backend | | | ⬜ |

## Requisitos no funcionales

| RNF | Tema | Tests | Estado |
| --- | --- | --- | --- |
| RNF-01 | Contraseñas protegidas | | 🟨 |
| RNF-02 | Secretos en configuración externa | | 🟨 |
| RNF-03 | La identidad sale del token | | 🟨 |
| RNF-04 | Endpoints protegidos por defecto | | 🟨 |
| RNF-05 | Validación en cada límite | | 🟨 |
