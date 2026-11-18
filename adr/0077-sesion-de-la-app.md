# ADR-0077. Sesión de la app: vencimiento leído del JWT y el 401 resuelto en el cliente HTTP

- **Estado:** Aceptado
- **Fecha:** 2026-11-17

## Contexto

Al abrir la app, con un JWT vigente se entra a Buscar y, si no, a Login. Un 401 de cualquiera de los dos servicios borra el JWT y vuelve a Login, en un solo lugar ([`arq/interfaz.md`](../arq/interfaz.md), [ADR-0049](0049-jwt-cifrado-en-el-dispositivo.md)). El JWT vence a las 24 horas o a los 30 días ([ADR-0033](0033-vigencia-jwt-usuario.md)) y trae su vencimiento en `exp` ([ADR-0060](0060-contenido-y-claves-del-jwt-de-usuario.md)).

## Decisión

- **Al arrancar, la app lee `exp` del JWT guardado**, sin verificar la firma. Si ya venció o no se puede leer, lo borra y abre Login. La firma la verifica siempre el backend: un token falso se rechaza con 401 en el primer pedido.
- **El cliente HTTP compartido agrega el JWT a cada pedido y atiende el 401.** Si un pedido que llevaba el JWT recibe 401, se borra la sesión. Un 401 de un pedido sin JWT (credenciales incorrectas en Login) no toca la sesión.
- **La navegación observa la sesión.** Cuando se cierra, vuelve a Login vaciando la pila de pantallas, y avisa si fue por vencimiento y no por el botón "Cerrar sesión".

## Alternativas consideradas

- **Decidir la pantalla inicial con un pedido de prueba al backend.** Descartada: sin conexión, la app no podría ni abrir, y suma una espera en cada arranque.
- **Entrar siempre a Buscar y esperar el primer 401.** Descartada: con un token que ya se sabe vencido, el usuario vería por un instante una pantalla que enseguida se cierra.
- **Manejar el 401 en cada ViewModel.** Descartada: se repetiría en cada pantalla, y olvidarlo en una dejaría al usuario con una sesión rota.
- **El plugin `Auth` de Ktor.** Descartada: está pensado para renovar tokens, que no hay ([ADR-0033](0033-vigencia-jwt-usuario.md)), y suma una dependencia para lo que resuelve un plugin propio de pocas líneas.

## Consecuencias

- Ningún ViewModel ni pantalla toca el JWT ni el 401 de una sesión vencida.
- Leer el JWT en la app queda como base para mostrar la sección de administración según el rol (`auth`), que es solo una decisión de interfaz ([ADR-0037](0037-rol-administrador-alcance-cerrado.md)).
