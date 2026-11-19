# ADR-0080. Disponibilidad y reserva en la app

- **Estado:** Aceptado
- **Fecha:** 2026-11-19

## Contexto

En la M4 de la app se construyen Turnos disponibles y Reserva en curso. [`arq/interfaz.md`](../arq/interfaz.md) y [`arq/maquina-estados.md`](../arq/maquina-estados.md) dicen qué muestra cada pantalla, y ADR-0019, ADR-0021, ADR-0024, ADR-0027 y ADR-0069 fijan cómo se comporta el backend. Quedaban sin decidir:

- cómo se cambia de día en Turnos disponibles y cómo se elige el turno;
- qué pasa cuando el inicio de la reserva vuelve ya rechazado;
- cómo encuentra la app un proceso activo para ofrecer retomarlo ([HU-07](../requisitos/HU.md#hu-07-reservar-un-turno), criterio 9), sin un contrato que lo informe;
- qué papel tiene la cuenta regresiva hasta `expiresAt` y cuándo se consulta el estado.

## Decisión

**Cambio de día por días de atención.** Las flechas de Turnos disponibles saltan al día anterior o siguiente en que el profesional atiende. Se calculan con la agenda vigente del catálogo, que la pantalla pide junto con la disponibilidad. La flecha "anterior" se deshabilita en hoy; la "siguiente" no tiene límite ([Constitución P-21](../constitucion.md)): más allá del horizonte se muestra el mensaje de `DATE_OUT_OF_RANGE`. Tocar la fecha abre el mismo calendario de Profesional. Si no quedan turnos, se ofrece el próximo día en que atiende.

**El turno se elige en dos toques.** Tocar un horario lo marca, y el botón de abajo dice "Reservar HH:mm". Mientras se espera la respuesta, los horarios y el botón quedan deshabilitados.

**Un rechazo al iniciar se muestra en Turnos disponibles.** Si turnos responde `201` con el proceso ya en un estado final ([ADR-0069](0069-toda-falla-del-hold-cierra-el-proceso.md)), la app se queda en la pantalla, muestra el motivo y recarga los horarios. Si arrancó, va a Reserva en curso y Turnos disponibles sale de la pila: su lista ya no está al día. Ante `409 ACTIVE_RESERVATION_EXISTS`, un diálogo ofrece ver la reserva en curso.

**El proceso activo es la última reserva creada.** La app pide `GET /api/reservations?size=1`, que ya viene ordenado por `createdAt` descendente, y si esa reserva no está en un estado final, es la activa. Vale por [ADR-0024](0024-un-proceso-activo-por-usuario.md): mientras hay un proceso activo no se puede crear otro, así que el activo, si existe, es el más nuevo. Buscar lo consulta cada vez que se muestra y, si hay uno, ofrece una tarjeta "Tenés una reserva en curso · Continuar".

**La cuenta regresiva informa y bloquea.** La app calcula cada segundo el tiempo que falta hasta `expiresAt` con el reloj del teléfono. Al llegar a cero, oculta el campo del teléfono y no deja enviarlo, aunque el estado todavía sea `AWAITING_PHONE`. No decide el resultado: sigue consultando hasta que turnos informe `EXPIRED`, porque la vigencia la define la cátedra ([`maquina-estados.md`](../arq/maquina-estados.md), regla 7). Es un doble control: la app no manda un teléfono que ya no tiene tiempo, y si uno sale justo en el límite, el backend lo rechaza (`409 PHONE_NOT_REQUESTED`) o el proceso termina en `EXPIRED`.

**La consulta del estado corre solo con la pantalla visible.** Cada 2 segundos ([ADR-0021](0021-app-consulta-estado-por-polling.md), [RNF-09](../requisitos/no-funcionales.md#rnf-09-sin-supuestos-fijos-sobre-los-datos-de-la-cátedra-y-valores-operativos-configurables)) mientras el proceso no sea final; se pausa con la app en segundo plano y, al volver, consulta en el momento. Un error de red no corta el seguimiento: se muestra un aviso chico y se sigue intentando.

**Mensajes propios para cada motivo** ([`interfaz.md`](../arq/interfaz.md)). El código nunca se muestra.

| Estado | `failureReason` | La app muestra |
| --- | --- | --- |
| `FAILED` | `SLOT_ALREADY_HELD`, `SLOT_ALREADY_RESERVED` | Alguien reservó ese horario hace un momento. Elegí otro. |
| `FAILED` | `INVALID_SLOT` | Ese horario ya no está disponible. Elegí otro. |
| `FAILED` | `PROFESSIONAL_DISABLED` | El profesional ya no recibe reservas. |
| `FAILED` | `PROFESSIONAL_NOT_FOUND` | No se encontró el profesional. |
| `FAILED` | `INTEGRATION_TIMEOUT`, `INTEGRATION_UNAVAILABLE` | No pudimos reservar en este momento. Probá de nuevo en unos minutos. |
| `FAILED`, `INVALID` | cualquier otro | No se pudo completar la reserva. Probá de nuevo. |
| `EXPIRED` | cualquiera | El tiempo para completar la reserva venció. El turno quedó libre. |
| `AWAITING_PHONE` | cualquiera (rechazo del teléfono) | No se aceptó el teléfono. Revisalo y envialo de nuevo, junto con el mensaje de la cátedra. |

## Alternativas consideradas

- **Un endpoint nuevo en turnos para el proceso activo.** Descartada: cambia un contrato y un servicio ya aprobados, y la regla de ADR-0024 ya deja encontrarlo con el listado que existe.
- **Guardar en el teléfono el id del proceso.** Descartada: se pierde al reinstalar la app o al entrar desde otro dispositivo.
- **Un diálogo al abrir la app para retomar.** Descartada: interrumpe, y una vez cerrado no se vuelve a encontrar.
- **Flechas día por día.** Descartada: un profesional que atiende dos días por semana obliga a pasar por cinco días vacíos.
- **Un toque en el horario y un diálogo para confirmar.** Descartada: suma una ventana para algo que ya se ve en el botón.
- **Una cuenta regresiva solo informativa.** Descartada: dejaría enviar un teléfono que ya no puede llegar a tiempo, y el usuario se enteraría recién con el vencimiento.
- **Corregir el desfase del reloj del teléfono con la hora del servidor.** Descartada: más código para una diferencia de segundos, que el resultado de la cátedra corrige igual.

## Consecuencias

- Turnos disponibles hace dos pedidos al abrirse: la disponibilidad a turnos y la agenda al catálogo, que responde con su copia local (Constitución P-06). Sin la agenda, las flechas avanzan día por día.
- Retomar no cambia ningún contrato. Si algún día se permitieran varios procesos activos, habría que reemplazar esta regla.
- Con el reloj del teléfono atrasado o adelantado, la cuenta regresiva puede llegar a cero unos segundos antes o después que la cátedra. El resultado final siempre es el que informa turnos.
- La pantalla deja de consultar al pasar a segundo plano: un proceso que termina mientras tanto se ve al volver.
