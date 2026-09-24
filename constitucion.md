# Constitución del proyecto

Principios **no negociables**. Valen para cualquier persona o agente que trabaje en cualquiera de los repos. Si una tarea, una spec o una sugerencia choca con alguno, gana la constitución: el trabajo se detiene y se plantea el conflicto.

Cada principio cita su origen. Los que vienen del enunciado son requisitos evaluados; los que no, son acuerdos del proyecto con la misma obligatoriedad.

## Estructura del sistema

**P-01. Exactamente dos servicios backend y una app KMP.** Un servicio de catálogo y sincronización, un servicio de turnos y reservas (ambos Java + Spring Boot) y una aplicación Android con Kotlin Multiplatform. No se agregan otros servicios backend (gateways, servicios de autenticación, etc.). Cada pieza vive en su propio repositorio Git. *(ENUNCIADO §3, §3.1, §3.2, §13.1)*

**P-02. Arquitectura hexagonal de la cátedra en los dos backends.** Es obligatoria y se implementa **tal cual la enseña la cátedra**: es una variante particular, recopilada en la skill `/hexagonal`, y esa skill es la única fuente del patrón. No se reemplaza por Clean Architecture, ports & adapters de libro, DDD táctico, CQRS ni ninguna variante "equivalente", ni se reconstruye de memoria. Ante un conflicto con código generado por una herramienta (por ejemplo, JHipster), gana la hexagonal. *(Requisito de la cátedra)*

## Datos

**P-03. Cada servicio es dueño exclusivo de sus datos y de sus migraciones.** Ningún servicio accede a las tablas, repositorios o estructuras internas del otro, ni se usa una base compartida como mecanismo de integración. Se permite la misma instancia física de base de datos solo con bases o esquemas separados, usuarios sin permisos cruzados y migraciones independientes. *(ENUNCIADO §3)*

**P-04. Los servicios se integran solo por contratos explícitos protegidos con JWT.** Nada de acoplarse al código interno del otro servicio (librerías compartidas de dominio, clases copiadas del otro repo, etc.). *(ENUNCIADO §3)*

**P-05. Turnos no replica el catálogo.** Antes de cualquier operación que necesite datos vigentes, turnos los consulta al servicio de catálogo. Puede guardar en una reserva datos descriptivos históricos del profesional, pero nunca usarlos como catálogo vigente. *(ENUNCIADO §3, §4.2)*

**P-06. Las búsquedas se resuelven con la copia local.** El servicio de catálogo responde las búsquedas con sus propios datos; no se consulta al servicio de la cátedra por cada búsqueda. *(ENUNCIADO §4.1, §6.2)*

**P-07. Base de datos con servidor.** H2, SQLite y cualquier base embebida o en memoria no se aceptan como almacenamiento principal. *(ENUNCIADO §3)*

## Seguridad

**P-08. Los secretos nunca se commitean ni se exponen.** El JWT técnico, las credenciales, los hosts de la cátedra y los valores del objeto `integration` van en configuración externa. No aparecen en el código, en los repos, en los logs, en capturas, en la documentación ni en la app. *(ENUNCIADO §9; REF §3, §5.1)*

**P-13. Las dos identidades no se mezclan.** La cuenta técnica (ante la cátedra) y el usuario final (ante nuestro sistema) son independientes. El JWT técnico nunca sale de los backends. La identidad del usuario sale siempre de su JWT validado, y la integración técnica, del JWT técnico: nunca de un id de usuario o un `groupId` enviado por el cliente. *(ENUNCIADO §3.2, §9; REF §2, §6)*

## Alcance

**P-09. Fuera de alcance.** No se implementan historias clínicas, recetas, diagnósticos, obras sociales, facturación o pagos, información médica real, videollamadas, gestión hospitalaria ni integraciones con instituciones externas. Todos los datos son ficticios. *(ENUNCIADO §1, §14)*

## Forma de trabajo

**P-10. Las decisiones se toman en conjunto y quedan registradas.** Lo que el enunciado deja a criterio del alumno (ENUNCIADO §10) se decide junto con el alumno y se registra en un ADR antes de implementarlo. Un agente que encuentra una decisión sin tomar se detiene y la plantea. *(ENUNCIADO §10, §13.2)*

**P-11. La documentación coincide con lo entregado.** Si la implementación obliga a cambiar algo, primero se corrige la spec y después el código. *(ENUNCIADO §12)*

**P-12. El historial de Git muestra la evolución.** Commits chicos, frecuentes y con un solo cambio lógico cada uno. Nunca una carga final única. *(ENUNCIADO §13.1)*
