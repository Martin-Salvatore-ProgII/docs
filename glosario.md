# Glosario

Términos del dominio y de la integración, con el significado exacto que tienen en este proyecto. Los nombres técnicos se escriben como en el contrato de la cátedra.

## Piezas del sistema

**Servicio de la cátedra** (o servicio central). Instancia administrada por la cátedra que es la **fuente autoritativa** del catálogo y el registro oficial de holds y reservas. Se accede por REST, Redis y Kafka. No se ejecuta localmente. *(ENUNCIADO §1; REF §3)*

**Catálogo.** Conjunto de categorías de profesionales (`professionalCategories`), profesionales (`professionals`) y horarios semanales de atención (`weeklySchedules`). Lo publica la cátedra y puede cambiar en cualquier momento. *(ENUNCIADO §1; REF §7)*

**Copia local del catálogo.** Réplica del catálogo que guarda el servicio de catálogo en su propia base. Es la que se usa para las búsquedas y para armar la disponibilidad. Se mantiene al día por sincronización. *(ENUNCIADO §4.1)*

**Servicio de catálogo** (`catalogo-service`). Backend dueño de la copia local del catálogo y de su sincronización. Responde las búsquedas de profesionales y es la única fuente vigente de profesionales y horarios para el resto del sistema. *(ENUNCIADO §4.1)*

**Servicio de turnos** (`turnos-service`). Backend dueño de los procesos de reserva y de las reservas de los usuarios finales. Arma la disponibilidad, habla con la cátedra para reservar y cancelar, y sabe a qué usuario pertenece cada reserva. *(ENUNCIADO §4.2)*

**App KMP** (`app-kmp`). Aplicación Android desarrollada con Kotlin Multiplatform. Es la interfaz del usuario final para todo el flujo funcional. Se ejecuta fuera de Docker Compose. *(ENUNCIADO §3.2)*

## Identidades

**Cuenta técnica.** Única cuenta del proyecto ante la cátedra. La registra a mano el responsable del proyecto y la usan los dos backends para todo acceso a REST, Redis y Kafka de la cátedra. No es un usuario de la app. *(REF §2, §5)*

**JWT técnico.** Token que emite la cátedra para la cuenta técnica. Dura un año y tiene el rol `ROLE_STUDENT_CLIENT`. Es secreto: nunca sale de los backends. *(REF §4, §5.1)*

**`groupId`.** Identificador de la integración técnica ante la cátedra. A pesar del nombre, no representa un grupo de personas ni un usuario final. Aísla los topics, el namespace de Redis y las reservas del proyecto. Se normaliza a minúsculas y guiones (`Proyecto Juan Pérez` → `proyecto-juan-perez`). *(REF §2.1, §5.1)*

**Objeto `integration`.** Configuración de conexión que la cátedra asigna a la cuenta técnica: Redis, Kafka, topics, consumer group y `provisioningStatus`. *(REF §5.1, §5.3)*

**`provisioningStatus`.** Estado de aprovisionamiento de la integración: `PENDING`, `PROVISIONED`, `FAILED` o `REVOKED`. Solo con `PROVISIONED` se entrega la contraseña de Redis. *(REF §5.1)*

**Usuario final.** Persona que se registra e inicia sesión en la app. Vive en el servicio de turnos y la cátedra no lo conoce. *(ENUNCIADO §3.2; REF §2)*

**JWT de usuario.** Token que emite el servicio de turnos al usuario final cuando inicia sesión. La app lo usa en las llamadas a nuestros dos servicios. Es independiente del JWT técnico. *(ENUNCIADO §3.2)*

**`externalPatientId`.** Identificador del usuario final que se envía a la cátedra al crear y confirmar un hold. En este proyecto es un UUID propio de cada usuario. *(REF §2, §9; [ADR-0004](adr/0004-external-patient-id-uuid.md))*
