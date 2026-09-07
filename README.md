# DSO202 - Assignment 1: Three-Tier Application Deployment on Kubernetes Cluster

## What It Is

A simple three-tier Task Tracker app:

- **frontend/** - HTML, CSS and JavaScript served by nginx. It displays tasks and allows users to create, update and delete them.
- **backend/** - Node.js/Express API that handles tasks and connects to PostgreSQL.
- **db/** - PostgreSQL database with an initialization script that creates the `tasks` table and adds some sample tasks.

## Rebuilding the Images

Run these commands from the project root:

```bash
docker build -t <dockerhub-username>/dso202-frontend:1.0 ./frontend
docker build -t <dockerhub-username>/dso202-backend:1.0 ./backend
docker build -t <dockerhub-username>/dso202-db:1.0 ./db
````

For Apple Silicon and Intel compatibility:

```bash
docker buildx build --platform linux/amd64,linux/arm64 \
  -t <dockerhub-username>/dso202-frontend:1.0 ./frontend --push
```

## Testing Locally

```bash
docker compose up --build
```

The services run on:

* Frontend: `http://localhost:18081`
* Backend: `http://localhost:18080`
* Database: `db`

Test the app before pushing the images to Docker Hub.

## Publishing

```bash
docker push <dockerhub-username>/dso202-frontend:1.0
docker push <dockerhub-username>/dso202-backend:1.0
docker push <dockerhub-username>/dso202-db:1.0
```

Use version tags such as `1.0` instead of `latest`. Do not overwrite a tag that has already been given to students.

## Environment Variables

| Variable            | Used By  | Purpose                       |
| ------------------- | -------- | ----------------------------- |
| `DB_HOST`           | Backend  | Database service name         |
| `DB_PORT`           | Backend  | Database port, default `5432` |
| `DB_NAME`           | Backend  | Database name                 |
| `DB_USER`           | Backend  | Database username             |
| `DB_PASSWORD`       | Backend  | Database password             |
| `APP_PORT`          | Backend  | Backend port, default `8080`  |
| `CORS_ORIGIN`       | Backend  | Allowed frontend origin       |
| `POSTGRES_DB`       | Database | Must match `DB_NAME`          |
| `POSTGRES_USER`     | Database | Must match `DB_USER`          |
| `POSTGRES_PASSWORD` | Database | Must match `DB_PASSWORD`      |
| `BACKEND_URL`       | Frontend | Backend service URL           |

The backend uses `DB_*`, while PostgreSQL uses `POSTGRES_*`. The values should match between the two.


