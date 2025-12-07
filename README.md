# WUDA - Mini Applications Setup

This repository contains two main applications: the back-end (server) and the front-end (user interface). This guide explains how to set up and run them locally.

---

## 🖥️ Back-End Setup

1. **Install dependencies:**

```bash
cd back-end
pip install -r requirements.txt
```

2. Create .env file in backend dir with given values:

```.env
# PostgreSQL database name
POSTGRES_DB=your_database_name

# PostgreSQL user
POSTGRES_USER=your_database_user

# PostgreSQL password
POSTGRES_PASSWORD=your_database_password

# Optional: PostgreSQL host (default is localhost)
POSTGRES_HOST=localhost

# Optional: PostgreSQL port (default is 5432)
PORT=5432

# Secret key for JWT
JWT_KEY=your_secret_jwt_key

```

3. In order to run app use command

```bash
python3 app.py
```

## 🌐 Front-End Setup

1. **Install dependencies:**

   ```bash
   cd front-end
   pip install -r requirements.txt

   ```

2. Create .env file

```.env

# Hostname of the back-end server (default: localhost)
BACKEND_HOSTNAME=localhost

# Port of the back-end server (default: 5000)
BACKEND_PORT=5000

# Secret key for front-end application
SECRET_KEY=your_secret_key_here

# Port where the front-end server will run (default: 8000)
SERVER_PORT=8000
```

3. Run app by following command:

```bash
python3 app.py
```

## Database setup

Required is postgresql. Either run given docker-compose.yaml file or provide setup for backend's `.env` file

```yaml
services:
  postgres:
    image: postgres
    environment:
      POSTGRES_PASSWORD: alamakota
      POSTGRES_USER: board_games
      POSTGRES_DB: games
    ports:
      - 5432:5432
```

## For students.

In `tasks` directory there are clear instructions for each task that you should do in order to pass the course.
Recommended order:

1. [CICD](./tasks/CICD.md)
2. [Kubernetes](./tasks/kubernetes.md)
3. [Monitoring](./tasks/monitoring.md)

## For MacOS

- There is a permission problem when connecting to Docker to test pipeline (using act), so AI recommended using:

```bash
DOCKER_HOST=unix:///Users/michaeleight/.docker/run/docker.sock act -j backend
```

```bash
DOCKER_HOST=unix:///Users/michaeleight/.docker/run/docker.sock act -j frontend
```

## Kubernetes Deployment

### Prerequisites

- Docker Desktop running
- Minikube installed
- Helm installed

### Quick Start

1. **Start Minikube:**

```bash
minikube start
```

2. **Load Docker images into Minikube** (for local testing):

```bash
minikube image load ghcr.io/michaeleight/wuda-backend:fix-readiness
minikube image load ghcr.io/michaeleight/wuda-frontend:latest
```

3. **Deploy with Helm:**

```bash
helm install wuda ./helm/wuda
```

4. **Wait for pods to be ready:**

```bash
kubectl wait --for=condition=Ready pod -l app.kubernetes.io/name=wuda --timeout=30s
```

5. **Access the frontend:**

```bash
minikube service wuda-frontend --url
```

Keep the terminal open and visit the displayed URL (e.g., `http://127.0.0.1:xxxxx`)

### Verify Deployment

```bash
# Check all pods are running
kubectl get pods

# Check services
kubectl get services

# View backend logs
kubectl logs -l app.kubernetes.io/component=backend

# View database logs
kubectl logs -l app.kubernetes.io/component=database
```

### Update Deployment

```bash
helm upgrade wuda ./helm/wuda
```

### Cleanup

```bash
helm uninstall wuda
minikube stop
```

### Known Issues & Fixes

**Issue: Backend readiness probe failing (503)**

- **Cause:** SQLAlchemy 2.x requires `text()` wrapper for raw SQL queries
- **Fix:** Updated `back-end/database.py:22` to use `session.execute(text("SELECT 1"))`
- **Image:** Use `ghcr.io/michaeleight/wuda-backend:fix-readiness`
