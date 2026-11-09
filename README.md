# rabbit-mq

RabbitMQ de Discordia, en su propia VM.


## Deploy

Automático: un push a `main` ejecuta `.github/workflows/ci.yml`, que tiene dos
jobs encadenados:

1. **`validate`**: comprueba el `docker-compose.yml` y levanta RabbitMQ con
   `docker compose up --wait`, esperando a que responda su healthcheck. Los
   pull requests y los push a `develop` corren solo este job.
2. **`deploy`**: solo si `validate` pasó, hace por SSH `git pull && docker compose up -d`
   en la VM de Oracle. Si RabbitMQ no levanta, no se despliega.

Necesita estos secrets configurados en **Settings → Secrets and variables →
Actions** de este repo:

| Secret | Valor |
|---|---|
| `ORACLE_HOST` | IP pública de la VM |
| `ORACLE_USER` | usuario SSH (ej. `ubuntu`) |
| `ORACLE_SSH_KEY` | clave privada SSH (la de despliegue, no tu clave personal) |

## Primera vez en la VM (manual, una sola vez)

```bash
git clone -b main https://github.com/discordia-grupo01/rabbit-mq.git
cd rabbit-mq
docker compose up -d
```
