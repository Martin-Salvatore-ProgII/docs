# Despliegue

Topología de ejecución local del sistema.

## Requisitos del enunciado

- Docker Compose levanta los dos servicios backend, sus bases de datos y las dependencias necesarias. *(ENUNCIADO §3)*
- La app Android se ejecuta fuera de los contenedores. *(ENUNCIADO §3)*
- El servicio de la cátedra (REST, Redis y Kafka) es una instancia central; no se levanta localmente. *(REF §3)*
- Los hosts, credenciales y topics de la cátedra se inyectan como configuración externa. *(REF §3; [Constitución P-08](../constitucion.md))*

## Pendiente

Se completa cuando se decidan:

- el motor de base de datos y si ambos servicios comparten instancia física (con bases o esquemas separados, [Constitución P-03](../constitucion.md));
- en qué repositorio vive el archivo de Docker Compose, dado que los servicios están en repos separados;
- cómo se entrega la configuración externa a los contenedores.
