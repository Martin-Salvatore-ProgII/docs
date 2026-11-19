# Prototipo de la app

Copia de las pantallas aprobadas del prototipo de la app ([ADR-0050](../../adr/0050-prototipos-con-herramientas-de-diseno.md)). El original se edita en el lienzo de diseño de Claude del proyecto; cuando una pantalla se aprueba, se copia acá, para que quede versionada junto a la especificación y se pueda abrir sin una cuenta.

Cada archivo es una pantalla de 390 × 844 y se abre directo en el navegador. Los enlaces entre pantallas funcionan (por ejemplo, tocar un profesional en Buscar abre Profesional). El `support.js` que piden los archivos es el del lienzo: sin él, la pantalla se ve igual.

La fuente de verdad sigue siendo [`interfaz.md`](../interfaz.md) y los ADR: si el prototipo difiere, mandan ellos.

| Pantalla | Archivo | Milestone de la app |
| --- | --- | --- |
| Buscar | [`Main.dc.html`](Main.dc.html) | M3 |
| Buscar, con filtros | [`BuscarFiltros.dc.html`](BuscarFiltros.dc.html) | M3 |
| Buscar, elegir categoría | [`BuscarCategoria.dc.html`](BuscarCategoria.dc.html) | M3 |
| Buscar, elegir franja | [`BuscarFranja.dc.html`](BuscarFranja.dc.html) | M3 |
| Buscar, final de la lista | [`BuscarMas.dc.html`](BuscarMas.dc.html) | M3 |
| Buscar, catálogo no listo | [`BuscarNoListo.dc.html`](BuscarNoListo.dc.html) | M3 |
| Profesional | [`Profesional.dc.html`](Profesional.dc.html) | M3 |
| Profesional no habilitado | [`ProfesionalNoHabilitado.dc.html`](ProfesionalNoHabilitado.dc.html) | M3 |
| Login | [`Login.dc.html`](Login.dc.html) | M2, ajustada en la M3 (#43) |
| Login, sesión vencida y error | [`LoginAviso.dc.html`](LoginAviso.dc.html) | M2, ajustada en la M3 (#43) |
| Registro | [`Registro.dc.html`](Registro.dc.html) | M2, ajustada en la M3 (#43) |
| Registro, errores por campo | [`RegistroErrores.dc.html`](RegistroErrores.dc.html) | M2, ajustada en la M3 (#43) |
| Turnos disponibles | [`Turnos.dc.html`](Turnos.dc.html) | M4 |
| Turnos disponibles, reservando | [`TurnosReservando.dc.html`](TurnosReservando.dc.html) | M4 |
| Turnos disponibles, el turno ya no está libre | [`TurnosTomado.dc.html`](TurnosTomado.dc.html) | M4 |
| Turnos disponibles, ya hay una reserva en curso | [`TurnosEnCurso.dc.html`](TurnosEnCurso.dc.html) | M4 |
| Turnos disponibles, sin turnos libres | [`TurnosVacio.dc.html`](TurnosVacio.dc.html) | M4 |
| Turnos disponibles, fecha fuera de rango | [`TurnosFueraDeRango.dc.html`](TurnosFueraDeRango.dc.html) | M4 |
| Turnos disponibles, servicio no disponible | [`TurnosNoDisponible.dc.html`](TurnosNoDisponible.dc.html) | M4 |
| Buscar, retomar la reserva en curso | [`BuscarEnCurso.dc.html`](BuscarEnCurso.dc.html) | M4 |
| Reserva en curso, reservando | [`ReservaIniciando.dc.html`](ReservaIniciando.dc.html) | M4 |
| Reserva en curso, pide el teléfono | [`Reserva.dc.html`](Reserva.dc.html) | M4 |
| Reserva en curso, teléfono con formato inválido | [`ReservaTelefonoInvalido.dc.html`](ReservaTelefonoInvalido.dc.html) | M4 |
| Reserva en curso, teléfono rechazado | [`ReservaRechazada.dc.html`](ReservaRechazada.dc.html) | M4 |
| Reserva en curso, confirmando | [`ReservaConfirmando.dc.html`](ReservaConfirmando.dc.html) | M4 |
| Reserva en curso, se terminó el tiempo | [`ReservaTiempoTerminado.dc.html`](ReservaTiempoTerminado.dc.html) | M4 |
| Reserva confirmada | [`ReservaConfirmada.dc.html`](ReservaConfirmada.dc.html) | M4 |
| Reserva vencida | [`ReservaVencida.dc.html`](ReservaVencida.dc.html) | M4 |
| Reserva que no se pudo completar | [`ReservaFallida.dc.html`](ReservaFallida.dc.html) | M4 |
| Mis reservas | [`MisReservas.dc.html`](MisReservas.dc.html) | M5 (guía) |

Las pantallas marcadas como guía son una primera idea: se revisan y se aprueban en su milestone.
