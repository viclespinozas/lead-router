# LeadRouter: guidance for Claude

Django 6.1 lead-routing service, built as a 72-hour learning project. Window: Thu Oct 22 12:45 PM to Sun Oct 25 12:45 PM (ET).
The owner is learning Django and prepping for interviews, so explain design choices instead of only writing code.

## Stack
- Python 3.13 (project requires >= 3.12), venv at `.venv/`
- Django 6.1, Django REST Framework, Celery, Redis, PostgreSQL (psycopg 3)
- Postgres 16 and Redis 7 run via `docker compose`; Django and Celery run from the venv on the host

## Commands
```bash
docker compose up -d --wait          # Postgres + Redis
set -a; . ./.env; set +a             # load env (copy .env.example to .env first)
.venv/bin/python manage.py check
.venv/bin/python manage.py makemigrations && .venv/bin/python manage.py sqlmigrate <app> <n>
.venv/bin/python manage.py migrate
```

## Layout
- `config/`: project settings, urls, wsgi/asgi
- `accounts/`: custom `User` (`AUTH_USER_MODEL = "accounts.User"`)
- `leads/`: Lead and Agent models, API, tasks (to be built)

## Rules
- Settings come from environment variables. Never hardcode secrets; `.env` is gitignored, `.env.example` is committed.
- Custom User is set before the first `migrate`. Do not change `AUTH_USER_MODEL` after migrating.
- Read the SQL (`sqlmigrate`) before running `migrate` on new models.
- Scope every API query by `agent__user=request.user`; use `select_related` / `prefetch_related`.
- Enqueue Celery tasks with `transaction.on_commit`, not inside the transaction.
- Outbound HTTP needs `timeout=10`; inbound webhooks verify HMAC with `compare_digest`.

## Task tracking
The board lives in Trello: https://trello.com/b/LEK75xOn/leadrouter-django-in-72h
Each card has a Definition of Done: code pushed, tests green (where they apply), and the owner can explain the decision in 60 seconds. Keep at most one card in Doing. Only tick a card when the owner says it is done.

## See also
`CONTEXT.md` for the current state of the project, `STORY.md` for why things were set up the way they were.
