# Stockpile — Backend (FastAPI)

FastAPI + SQLAlchemy + PostgreSQL service for the Stockpile inventory app.
Designed to deploy to **Railway** with a Railway-managed Postgres.

## Stack

- Python 3.12 · FastAPI · SQLAlchemy 2 · psycopg2
- Gunicorn (Uvicorn workers) for production
- Multi-stage Docker build, non-root runtime, tini as PID 1, healthcheck

## Run locally

```bash
cp .env.example .env
python -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Or with Docker:

```bash
docker build -t inventory-backend .
docker run --rm -p 8000:8000 --env-file .env inventory-backend
```

Swagger docs: <http://localhost:8000/docs>

## Deploy to Railway

1. Push this folder to its own GitHub repo.
2. In Railway, **New Project → Deploy from GitHub repo** and pick it.
3. Add the **Postgres** plugin. Railway exposes `DATABASE_URL` as a service variable.
4. In the backend service → **Variables**, reference the Postgres URL and add the rest:

   | Variable              | Value                                                      |
   | --------------------- | ---------------------------------------------------------- |
   | `DATABASE_URL`        | `${{ Postgres.DATABASE_URL }}`                             |
   | `CORS_ORIGINS`        | `https://<your-app>.vercel.app`                            |
   | `LOW_STOCK_THRESHOLD` | `10` (optional)                                            |
   | `WEB_CONCURRENCY`     | `2` (optional)                                             |

5. Railway uses `railway.toml` → builds with the `Dockerfile`, runs the start
   command, and probes `/health`. `PORT` is injected automatically.
6. Generate a public domain under **Settings → Networking** and put that URL
   into the frontend's `VITE_API_URL`.

## Configuration

All config is environment-driven (see `.env.example`). `DATABASE_URL` takes
precedence over the discrete `POSTGRES_*` vars, and `postgres://` is auto-rewritten
to `postgresql://` for SQLAlchemy.
