# STORY: what was added for card #1, and why

Card #1 asked for: a repo, Docker Compose with Postgres + Redis, a venv, and Django/psycopg/DRF/Celery/redis installed on Python >= 3.12. This is what the project looked like when reviewed, and what changed.

## What was already good
- Python 3.13 venv with Django 6.1.2, DRF, psycopg 3, Celery and redis installed.
- A GitHub repo with `origin` set.
- `config/settings.py` already read secrets and database settings from env vars.

## What was wrong
The `docker-compose.yml` looked copied from a different project:
- It had no Redis, so Celery would have had no broker.
- Database was `tracker` / `tracker_test`, but this project is LeadRouter.
- `POSTGRES_MULTIPLE_DATABASES` is not a feature of the official Postgres image. It needs a custom init script, and the `init-scripts/` folder did not exist.
- A `backend` service built from `leads/Dockerfile`, which ran `uvicorn app.main:app`. That is a FastAPI template; this is Django, so it could never start.
- The healthcheck used `${POSTGRES_USER}` with no default, so it failed when the variable was unset.
- There was no `requirements.txt`, nothing was committed, and `.venv` was not ignored.

## What changed, and why
- **`docker-compose.yml`**: two services only, `db` (postgres:16-alpine) and `redis` (redis:7-alpine). Each has a healthcheck so `docker compose up --wait` blocks until they are ready, and a named volume so data survives restarts. Database name is `leadrouter`. Defaults let it start without a `.env`.
- **Django is not in Compose.** Running Django on the host from the venv gives faster reloads and simpler debugging while learning. Only the stateful services are containerised.
- **Celery is not in Compose either.** Celery is a library you run as a process (`celery -A config worker`). It only needs a broker, which is the Redis service now added. Wiring the worker is card #13.
- **Deleted `leads/Dockerfile`**: it was the FastAPI leftover and nothing used it. A real Dockerfile can come back later if you want to containerise the app.
- **`requirements.txt`**: pinned to what is installed, so the environment can be rebuilt. `psycopg[binary]` ships the compiled driver so no system libpq is needed.
- **`.gitignore`**: keeps `.venv/`, `.env`, caches and editor files out of git.
- **`.env.example`**: documents every variable settings.py reads, plus `REDIS_URL`. Copy it to `.env`. This overlaps card #3, which stays open until you review the settings yourself.
- **`config/settings.py`**: removed a commented-out hardcoded `SECRET_KEY`. The repo is public, and a leftover key in a comment is a bad habit even for a throwaway dev key.
- **`CLAUDE.md`, `CONTEXT.md`, `STORY.md`**: so any session, or you in a week, can see the rules, the current state, and the reasoning.

## How it was checked
Containers started and reported healthy; `select 1` ran in Postgres; `redis-cli ping` returned PONG; `manage.py check --database default` passed. `migrate` was deliberately not run (card #2 and #4 cover it).

## Interview angle (60-second version)
"I run only stateful services in Compose and Django on the host. Redis is there as the Celery broker. Healthchecks make startup deterministic. Secrets come from the environment, with an example file committed and the real one ignored."
