# Glosario

Términos del dominio y de la integración, con el significado exacto que tienen en este proyecto. Los nombres técnicos se escriben como en el contrato de la cátedra.

## Piezas del sistema

**Servicio de la cátedra** (o servicio central). Instancia administrada por la cátedra que es la **fuente autoritativa** del catálogo y el registro oficial de holds y reservas. Se accede por REST, Redis y Kafka. No se ejecuta localmente. *(ENUNCIADO §1; REF §3)*

**Catálogo.** Conjunto de categorías de profesionales (`professionalCategories`), profesionales (`professionals`) y horarios semanales de atención (`weeklySchedules`). Lo publica la cátedra y puede cambiar en cualquier momento. *(ENUNCIADO §1; REF §7)*

**Copia local del catálogo.** Réplica del catálogo que guarda el servicio de catálogo en su propia base. Es la que se usa para las búsquedas y para armar la disponibilidad. Se mantiene al día por sincronización. *(ENUNCIADO §4.1)*

**Servicio de catálogo** (`catalogo-service`). Backend dueño de la copia local del catálogo y de su sincronización. Responde las búsquedas de profesionales y es la única fuente vigente de profesionales y horarios para el resto del sistema. *(ENUNCIADO §4.1)*

**Servicio de turnos** (`turnos-service`). Backend dueño de los procesos de reserva y de las reservas de los usuarios finales. Arma la disponibilidad, habla con la cátedra para reservar y cancelar, y sabe a qué usuario pertenece cada reserva. *(ENUNCIADO §4.2)*

**App KMP** (`app-kmp`). Aplicación Android desarrollada con Kotlin Multiplatform. Es la interfaz del usuario final para todo el flujo funcional. Se ejecuta fuera de Docker Compose. *(ENUNCIADO §3.2)*
