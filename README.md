# Kittygram

Kittygram is an educational Django REST API and React application for sharing cats, their photos, colors, and achievements. The project is fully containerized and includes a GitHub Actions CI/CD workflow for linting, tests, Docker image builds, Docker Hub publishing, and Telegram notifications.

## Technologies

- Python 3.10, Django 3.2, Django REST Framework, Djoser
- PostgreSQL 13
- React 17
- Nginx
- Docker and Docker Compose
- GitHub Actions
- Ruff, pytest

## Environment

Create `.env` in the project root using `.env.example` as a template:

```env
SECRET_KEY=change-me
DEBUG=False
ALLOWED_HOSTS=localhost,127.0.0.1

POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=kittygram_password
DB_NAME=kittygram
DB_USER=kittygram_user
DB_PASSWORD=kittygram_password
DB_HOST=db
DB_PORT=5432
```

For production, add your real domain or server IP to `ALLOWED_HOSTS`.

## Local Docker Run

Build and start the project:

```bash
docker compose up --build
```

The app will be available at:

```text
http://localhost:9000
```

Useful commands:

```bash
docker compose exec backend python manage.py createsuperuser
docker compose exec backend python manage.py test
docker compose down
```

The `backend` container runs migrations and collects static files on startup. The `frontend` container copies the built React app into the shared `static` volume. The `gateway` container serves frontend static files and uploaded media, and proxies `/api/` and `/admin/` requests to Django.

## Volumes

- `pg_data` stores PostgreSQL data.
- `static` is shared by `backend`, `frontend`, and `gateway`.
- `media` is shared by `backend` and `gateway`.

## Production Compose

After images are published to Docker Hub, use `docker-compose.production.yml` on the server:

```bash
export DOCKERHUB_USERNAME=<your_dockerhub_username>
docker compose -f docker-compose.production.yml pull
docker compose -f docker-compose.production.yml up -d
```

Images:

- `<dockerhub_username>/kittygram_backend`
- `<dockerhub_username>/kittygram_frontend`
- `<dockerhub_username>/kittygram_gateway`

## CI/CD

The workflow is defined in `.github/workflows/main.yml` and copied to `kittygram_workflow.yml` for educational tests.

On every push it:

1. Installs backend dependencies and runs `ruff check backend`.
2. Runs backend tests.
3. Installs frontend dependencies and runs frontend tests.
4. Builds backend, frontend, and gateway Docker images.
5. Pushes images to Docker Hub only for `main` or `master`.
6. Sends a Telegram notification after a successful workflow when Telegram secrets are configured.

Required GitHub Secrets:

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`
- `TELEGRAM_TOKEN`
- `TELEGRAM_TO`

## Local Tests

Backend tests:

```bash
python -m pip install -r backend/requirements.txt
pytest backend
```

Frontend tests:

```bash
cd frontend
npm ci
CI=true npm test -- --watchAll=false
```

Educational checker tests from the repository root:

```bash
pytest
```
