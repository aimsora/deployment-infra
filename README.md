# deployment-infra

![CI](https://img.shields.io/badge/CI-GitHub_Actions-2088FF?logo=githubactions&logoColor=white)
![CD](https://img.shields.io/badge/CD-GitHub_Deploy-2ea44f?logo=github&logoColor=white)
![IaC](https://img.shields.io/badge/IaC-Docker_Compose-1d63ed?logo=docker&logoColor=white)

Инфраструктурный репозиторий для локального запуска и деплоя через GitHub.

## Что делает этот репозиторий

- поднимает инфраструктуру платформы (PostgreSQL, Redis, RabbitMQ, MinIO);
- содержит compose-описание локального запуска и deploy-контура через GHCR-образы;
- фиксирует единые network/DNS-имена для межсервисного взаимодействия.

## Черновая реализация

- `docker-compose.yml` - инфраструктурные сервисы;
- `docker-compose.apps.yml` - локальный запуск `backend-api`, `scraper-service`, `processing-worker`, `frontend-app` из исходников;
- `docker-compose.deploy.yml` - запуск приложений из GHCR-образов для удаленного стенда;
- `contracts/events/*` - снапшот контрактов для `scraper-service` и `processing-worker`;
- `Makefile` с командами `up`, `down`, `up-all`, `down-all`;
- CI workflow с валидацией compose-конфигураций;
- CD workflow `.github/workflows/deploy.yml` для выкладки стенда по SSH.

## Локальный запуск

Только инфраструктура:

```bash
cp .env.example .env
make up
```

Инфраструктура + приложения:

```bash
cp .env.example .env
make up-all
```

Остановка полного стека:

```bash
make down-all
```

## Деплой стенда через GitHub Actions

`deployment-infra` использует workflow `Deploy Stand` и compose-файл `docker-compose.deploy.yml`.

### Что нужно настроить в GitHub (repository secrets)

- `SSH_HOST`, `SSH_USER`, `SSH_PRIVATE_KEY` - доступ к серверу;
- `GHCR_USERNAME`, `GHCR_TOKEN` - доступ к `ghcr.io` (token с `read:packages`);
- `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`;
- `RABBITMQ_DEFAULT_USER`, `RABBITMQ_DEFAULT_PASS`;
- `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`, `MINIO_BUCKET`;
- `JWT_ACCESS_SECRET`.

### Запуск деплоя

1. Открой `Actions -> Deploy Stand`.
2. Выбери `environment` (`staging` или `production`).
3. Укажи `image_tag` (обычно `edge`).
4. Нажми `Run workflow`.

Если меняются схемы в `shared-contracts`, обнови `contracts/events/*` в этом репозитории перед деплоем.

## Сеть и адреса

- сеть: `platform-net`
- `postgres:5432`
- `redis:6379`
- `rabbitmq:5672` (`15672` management)
- `minio:9000` (`9001` console)
- `backend-api:3000`
- `frontend-app:80` (внешний порт `8080`)
