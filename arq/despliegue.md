# Despliegue

Topología de ejecución local del sistema.

## Requisitos del enunciado

- Docker Compose levanta los dos servicios backend, sus bases de datos y las dependencias necesarias. *(ENUNCIADO §3)*
- La app Android se ejecuta fuera de los contenedores. *(ENUNCIADO §3)*
- El servicio de la cátedra (REST, Redis y Kafka) es una instancia central; no se levanta localmente. *(REF §3)*
- Los hosts, credenciales y topics de la cátedra se inyectan como configuración externa. *(REF §3; [Constitución P-08](../constitucion.md))*

## Topología

```mermaid
flowchart LR
    APP[App KMP<br/>Android, fuera de Docker]

    subgraph TR[turnos-service/compose.yaml: sistema completo]
        TS[Servicio de turnos]
        TDB[(PostgreSQL<br/>turnos)]
        subgraph CR[catalogo-service/compose.yaml, incluido]
            CS[Servicio de catálogo]
            CDB[(PostgreSQL<br/>catálogo)]
        end
    end

    CAT[Servicio de la cátedra<br/>REST, Redis, Kafka]

    APP -->|HTTPS| TS
    APP -->|HTTPS| CS
    TS -->|HTTPS| CS
    TS --- TDB
    CS --- CDB
    TS --> CAT
    CS --> CAT
```

| Contenedor | Definido en | Contenido |
| --- | --- | --- |
| Servicio de catálogo | `catalogo-service/compose.yaml` | El backend de catálogo |
| PostgreSQL del catálogo | `catalogo-service/compose.yaml` | Base, usuario y volumen propios ([ADR-0041](../adr/0041-postgresql-por-servicio.md)) |
| Servicio de turnos | `turnos-service/compose.yaml` | El backend de turnos |
| PostgreSQL de turnos | `turnos-service/compose.yaml` | Base, usuario y volumen propios ([ADR-0041](../adr/0041-postgresql-por-servicio.md)) |

## Formas de levantarlo

([ADR-0042](../adr/0042-ubicacion-docker-compose.md))

- **Un servicio solo:** con el `compose.yaml` de su repo, que levanta el servicio y su base.
- **El sistema completo:** con el `compose.yaml` de `turnos-service`, que incluye el del catálogo. Requiere los repos clonados uno al lado del otro:

```
carpeta-del-proyecto/
├── catalogo-service/
└── turnos-service/
```

El servicio de la cátedra se alcanza por la red que indique la cátedra (en el entorno actual, una red privada virtual a la que se une la máquina que ejecuta los contenedores).

## Configuración

([ADR-0043](../adr/0043-configuracion-externa-env-y-secrets.md))

| Qué | Dónde | En Git |
| --- | --- | --- |
| URL de la cátedra, JWT técnico, credenciales de base, credenciales del administrador inicial (solo en turnos) | `.env` de cada repo | No; se commitea `.env.example` |
| Par de claves del JWT de usuario, certificado TLS | Carpeta `secrets/`, montada en los contenedores | No |
| Redis, Kafka y topics de la cátedra | Se obtienen de la cátedra al arrancar ([ADR-0005](../adr/0005-configuracion-integracion-al-arrancar.md)) | No |
| Valores operativos ([RNF-09](../requisitos/no-funcionales.md#rnf-09-sin-supuestos-fijos-sobre-los-datos-de-la-cátedra-y-valores-operativos-configurables)) | Configuración de la aplicación, sobrescribible por variables de entorno | Sí, solo los valores iniciales |

Los puertos, nombres de contenedores y variables concretas se documentan en el README y el `.env.example` de cada repo. Los puertos publicados en la máquina están fijados en el [ADR-0056](../adr/0056-imagen-con-dockerfile-y-puertos.md).

### Variables al levantar el sistema completo

Cada repo tiene su propio `.env`, y al levantar el sistema completo hacen falta los dos: el Compose de turnos lee el suyo y el del catálogo, incluido, lee el del repo del catálogo.

**Los nombres de las variables de los dos `.env` no se pueden repetir.** Cuando un Compose incluye a otro, las variables del que incluye tienen prioridad sobre las del incluido: si el `.env` de turnos define una variable con el mismo nombre que una del catálogo, el catálogo arranca con el valor de turnos. Con nombres repetidos para la base, las dos bases se crearían con el mismo nombre, usuario y clave.

Por eso las variables del `.env` de turnos llevan el prefijo `TURNOS_`. El prefijo existe solo en el `.env` y en el `compose.yaml`: dentro de su contenedor, cada servicio recibe la conexión con los mismos nombres (`DB_URL`, `DB_USER`, `DB_PASSWORD`; [ADR-0055](../adr/0055-convenciones-comunes-del-esqueleto.md)).

El volumen de la base del catálogo es distinto según desde dónde se levante: levantado desde su repo usa un volumen, y levantado como parte del sistema completo usa otro. Los datos de una forma no aparecen en la otra.
