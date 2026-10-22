# Context: where the project stands

Last updated: Thu Oct 22, 2026, during card #1 (Repo + Docker Compose).

## What exists
| Piece | State |
| --- | --- |
| Django project `config`, apps `accounts` and `leads` | Created, no models besides the custom `User` |
| Custom User (`accounts.User`, subclass of `AbstractUser`) | Defined; no migrations generated or applied yet |
| Settings read from env vars | In place: `DJANGO_SECRET_KEY`, `DJANGO_DEBUG`, `DJANGO_ALLOWED_HOSTS`, `POSTGRES_*` |
| `docker-compose.yml` | Postgres 16 and Redis 7, both with healthchecks and named volumes |
| `requirements.txt` | Django, DRF, psycopg[binary], celery, redis (pinned) |
| `.env.example`, `.gitignore` | Added; `.env` is ignored |
| Git | Remote `origin` = github.com/viclespinozas/lead-router |

## What does not exist yet
- Agent and Lead models, migrations (card #4)
- Admin, API, tests, Celery app, webhooks (later cards)
- Celery itself is installed but not configured. It is a Python process, not a compose service: the worker will be started from the venv in card #13. `REDIS_URL` in `.env.example` is for that.
- No Dockerfile for Django. Django runs on the host from `.venv`; only the databases are containerised.

## Environment variables
See `.env.example`. Compose uses the same `POSTGRES_*` names with defaults (`postgres` / `password` / `leadrouter`), so it works without a `.env`, but Django requires `DJANGO_SECRET_KEY` and `POSTGRES_*` to be set.

## Verified
- `docker compose up -d --wait`: db and redis healthy
- `psql select 1` and `redis-cli ping` succeed
- `manage.py check --database default` passes against the container Postgres
- `manage.py migrate` has NOT been run

## Next
Card #2 (custom User before first migrate; the model is already there, so confirm the order and generate migrations), then #3 (env settings; mostly done), then #4.
