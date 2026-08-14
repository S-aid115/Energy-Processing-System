# Energy Process

## Initial Setup

### 1. Clone the repository

```bash
git clone <REPOSITORY_URL>
cd Energy_process
```

### 2. Configure environment variables

Create a `.env` file in the project root and configure the required environment variables.

Make sure the Redis URL matches the way you are running the application:

```env
REDIS_URL=redis://localhost:6379/0
```

> **Note:** When running the application entirely with Docker, the Redis hostname may be `redis` instead of `localhost`, depending on the configuration in `docker-compose.yml`.

---

## 3. Run Everything with Docker (Recommended)

The recommended way to run the entire application is using Docker Compose.

With a single command, the following services are started:

* Backend (FastAPI/Uvicorn)
* Celery Worker
* Frontend
* PostgreSQL
* Redis

The backend waits for the database, initializes it with test data, and starts the API. The Celery worker processes files in the background.

```bash
docker-compose up -d --build
```

### Services

| Service    | Container         | Description                                     |
| ---------- | ----------------- | ----------------------------------------------- |
| Backend    | `peajes_backend`  | FastAPI API running with Uvicorn on port `8000` |
| Worker     | `peajes_worker`   | Celery background worker for file processing    |
| Frontend   | `peajes_frontend` | React application on port `3000`                |
| PostgreSQL | `peajes_postgres` | PostgreSQL database on port `5432`              |
| Redis      | `peajes_redis`    | Message broker for Celery on port `6379`        |

For file upload and background processing to work correctly, both the **Backend** and **Celery Worker** must be running. The command above starts both automatically.

---

# Local Development

## Run Frontend, Backend and Celery Separately

If you want to run the **frontend**, **backend (Uvicorn)** and **Celery worker** separately without running the application services inside Docker, follow the steps below.

### Step 0 — Required Services

PostgreSQL and Redis must be running.

If you previously executed:

```bash
docker-compose down
```

both PostgreSQL and Redis were stopped.

Start only these two services:

```bash
docker-compose up -d postgres redis
```

### Important

When running the backend and Celery locally, make sure your `.env` file contains:

```env
REDIS_URL=redis://localhost:6379/0
```

The hostname `redis` is only available inside the Docker network.

---

## Run the Application

Open **three terminals**.

### Terminal 1 — Backend

From the `backend` directory:

```powershell
cd backend
.\venv\Scripts\Activate.ps1
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Terminal 2 — Celery Worker

Open a new terminal:

```powershell
cd backend
.\venv\Scripts\Activate.ps1
celery -A app.celery_app worker --loglevel=info -P solo
```

### Terminal 3 — Frontend

Open another terminal:

```powershell
cd frontend
npm run dev
```

---

## Troubleshooting

### Error 10061 or Timeout Connecting to Redis

A common cause is that Docker Desktop is not running or the Redis container has not been started.

### Steps

1. Start **Docker Desktop** and wait until it is ready.
2. From the project root, start PostgreSQL and Redis:

```bash
docker-compose up -d postgres redis
```

3. Verify that the containers are running:

```bash
docker ps
```

You should see:

```text
peajes_redis
peajes_postgres
```

with an `Up` status.

4. From the `backend` directory, test the Redis connection:

```bash
python check_redis.py
```

If the command returns:

```text
OK
```

you can start the Celery worker.

### If you do not want to use Docker for Redis

You need to have Redis installed and running locally on Windows.

For example, you can use [Memurai](https://www.memurai.com/).

In this case, keep the following configuration in `.env`:

```env
REDIS_URL=redis://localhost:6379/0
```

---

# Service Configuration

## Backend — Python

From the `backend` directory:

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

## Celery Worker

From the `backend` directory, open another terminal:

```powershell
.\venv\Scripts\Activate.ps1
celery -A app.celery_app worker --loglevel=info -P solo
```

## Frontend — React

From the `frontend` directory:

```bash
npm install
npm run dev
```

---

# Database Initialization

To create the database tables and insert test data:

### Using Docker

```bash
docker-compose exec backend python init_db.py
```

### Using Local Development

From the `backend` directory:

```bash
python init_db.py
```

---

# Service Access

Once the application is running, the services can be accessed at:

| Service                   | URL / Address              |
| ------------------------- | -------------------------- |
| Frontend — Docker         | http://localhost:3000      |
| Frontend — Local          | http://localhost:5173      |
| Backend API               | http://localhost:8000      |
| Swagger API Documentation | http://localhost:8000/docs |
| PostgreSQL                | `localhost:5432`           |
| Redis                     | `localhost:6379`           |

### Frontend

```text
http://localhost:3000
```

or, when running locally:

```text
http://localhost:5173
```

### Backend API

```text
http://localhost:8000
```

### Swagger Documentation

```text
http://localhost:8000/docs
```

---

# Quick Start

If you only want to run the complete application using Docker:

```bash
git clone <REPOSITORY_URL>
cd Energy_process
docker-compose up -d --build
```

Then access:

```text
Frontend: http://localhost:3000
Backend:  http://localhost:8000
Swagger:  http://localhost:8000/docs
```
