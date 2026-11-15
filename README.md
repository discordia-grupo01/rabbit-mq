# rabbit-mq

RabbitMQ de Discordia.

## Tabla de contenidos

- [Introducción](#introducción)
- [Pre-requisitos](#pre-requisitos)
- [Cómo correr](#cómo-correr)
- [Variables de entorno](#variables-de-entorno)
- [Comandos Docker](#comandos-docker)
- [Tests](#tests)
- [Más documentación](#más-documentación)

## Introducción

Este repositorio levanta el RabbitMQ que comparten los servicios de Discordia. Es el bus por el que se avisan los cambios entre sí: cuando `servers` crea un canal, `messaging` y `notifications` se enteran por acá, sin llamarse por HTTP.

No tiene código propio: es un `docker-compose.yml` con la imagen oficial de RabbitMQ (con su panel de administración), un volumen para que los mensajes sobrevivan a un reinicio y una comprobación de salud. Corre en su propia máquina en producción, por eso vive en un repositorio aparte.

Todos los eventos viajan por el exchange `discordia.events`. Cada servicio crea sus propias colas al arrancar, así que acá no hay nada que configurar a mano.

## Pre-requisitos

- **Docker** y **Docker Compose** v2.24 o más nuevo.

## Cómo correr

### En tu máquina

Es lo que necesitás antes de levantar cualquier servicio que use eventos (`messaging`, `notifications`, `billing`, `admin` y `voice-sfu`, entre otros):

```bash
docker compose up -d
```

Para comprobar que respondió, abrí el panel en http://localhost:15672 (usuario y contraseña `guest`). Los servicios que corren en un contenedor se conectan con `amqp://guest:guest@host.docker.internal:5672`, y los que corren en tu máquina, con `amqp://guest:guest@localhost:5672`.

### En producción

Un push a `main` lo despliega solo, como se explica abajo. La primera vez, en la VM de Oracle, hay que clonarlo y levantarlo a mano una sola vez:

```bash
git clone -b main https://github.com/discordia-grupo01/rabbit-mq.git
cd rabbit-mq
docker compose up -d
```

El despliegue automático hace por SSH `git pull && docker compose up -d` en la VM, solo si antes pasó la validación. Necesita estos secretos en **Settings → Secrets and variables → Actions** del repositorio:

| Secreto | Valor |
|---|---|
| `ORACLE_HOST` | IP pública de la VM. |
| `ORACLE_USER` | Usuario SSH, por ejemplo `ubuntu`. |
| `ORACLE_SSH_KEY` | Clave privada SSH de despliegue (no tu clave personal). |

## Variables de entorno

| Variable | Default | Descripción |
|---|---|---|
| `RABBITMQ_BIND_IP` | `0.0.0.0` | IP en la que se publica el puerto `5672`. En producción conviene poner la IP de Tailscale de la VM. |

Se leen del entorno o de un archivo `.env` junto al `docker-compose.yml`. El panel de administración (`15672`) siempre queda publicado solo en `127.0.0.1`.

## Comandos Docker

Todos desde la raíz del repositorio.

| Qué querés hacer | Comando |
|---|---|
| Levantar RabbitMQ | `docker compose up -d` |
| Esperar a que esté listo | `docker compose up -d --wait` |
| Ver los logs | `docker compose logs -f rabbitmq` |
| Ver qué está corriendo | `docker compose ps` |
| Listar las colas | `docker compose exec rabbitmq rabbitmqctl list_queues name messages` |
| Bajar, conservando los mensajes | `docker compose down` |
| Bajar y borrar todo | `docker compose down -v` |

## Tests

No tiene tests propios. En cada pull request y en cada push a `develop` o `main`, el CI valida el `docker-compose.yml` con `docker compose config --quiet`, levanta RabbitMQ con `docker compose up -d --wait` y espera a que responda su comprobación de salud. Si no levanta, el pipeline falla y no se despliega.

Lo mismo se puede hacer a mano:

```bash
docker compose config --quiet
docker compose up -d --wait
docker compose down -v
```

## Más documentación

- [`docs`](https://github.com/discordia-grupo01/docs/tree/main/asyncapi): todos los eventos que viajan por RabbitMQ, con quién los publica y quién los consume.
- [`reintentos-y-dlq.md`](https://github.com/discordia-grupo01/docs/blob/main/reintentos-y-dlq.md): qué pasa cuando un servicio no puede procesar un evento y cómo revisar las colas de eventos fallidos.
