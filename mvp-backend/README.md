# mvp-backend

Backend service built with [Django](https://www.djangoproject.com/) and
[Django Ninja](https://django-ninja.dev/) / [Ninja Extra](https://eadwincode.github.io/django-ninja-extra/)
for the API layer.

## Tech stack

| Area            | Tool                             |
| --------------- | -------------------------------- |
| Language        | Python 3.12                      |
| Web framework   | Django 6.1                       |
| API             | django-ninja, django-ninja-extra |
| Configuration   | django-environ (`.env` files)    |
| Database        | PostgreSQL (via psycopg 3)       |
| Package manager | [uv](https://docs.astral.sh/uv/) |
| Testing         | pytest, pytest-django            |
| Lint / format   | ruff                             |

## Project structure

```
mvp-backend/
├── apps/
│   └── users/            # Users app
├── config/
│   ├── settings/
│   │   ├── base.py       # Shared settings
│   │   ├── dev.py        # Local development
│   │   ├── test.py       # Test runs
│   │   └── prod.py       # Production
│   ├── urls.py           # Root URL configuration
│   ├── asgi.py
│   └── wsgi.py
├── manage.py
├── pyproject.toml
├── uv.lock
└── .env.example          # Template for environment variables
```

New Django apps go under `apps/` and are registered in `INSTALLED_APPS` as `apps.<name>`.

## Getting started

### Prerequisites

- Python 3.12
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- PostgreSQL (running locally, with a database created for this project)

### Setup

```bash
# Install dependencies (including dev tools) into .venv
uv sync

# Create your local environment file
cp .env.example .env   # then fill in the values (see Configuration below)

# Create migration files for any model changes
uv run python manage.py makemigrations

# Apply database migrations
uv run python manage.py migrate

# Create an admin user
uv run python manage.py createsuperuser

# Start the development server
uv run python manage.py runserver
```

The app is then available at http://127.0.0.1:8000/, and the Django admin at
http://127.0.0.1:8000/admin/.

## Configuration

Settings are split per environment under `config/settings/`. `manage.py` uses
`config.settings.dev` by default; select another with the `DJANGO_SETTINGS_MODULE`
environment variable:

| Environment | Module                 |
| ----------- | ---------------------- |
| Development | `config.settings.dev`  |
| Testing     | `config.settings.test` |
| Production  | `config.settings.prod` |

`SECRET_KEY` is required and has no default. Generate one with:

```bash
uv run python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

Secrets and environment-specific values (secret key, debug flag, allowed hosts, database URL)
should be read from `.env` via django-environ. Keep `.env.example` up to date with every
variable the app expects, and never commit `.env`.

## Development

### Database migrations

Whenever you add or change a model, generate and apply migrations:

```bash
uv run python manage.py makemigrations          # all apps
uv run python manage.py makemigrations users    # a single app
uv run python manage.py migrate
```

Commit the generated files under `apps/<app>/migrations/` together with the model changes.

### Running tests

```bash
uv run pytest
```

pytest discovers `tests.py` and `test_*.py` files. The settings module used for tests is set in
`pyproject.toml` under `[tool.pytest.ini_options]`.

### Linting and formatting

```bash
uv run ruff check .          # lint
uv run ruff check . --fix    # lint and auto-fix
uv run ruff format .         # format
```

Ruff is configured in `pyproject.toml` (line length 100, rules `E`, `F`, `I`, `B`, `UP`, `DJ`).

### Adding dependencies

```bash
uv add <package>             # runtime dependency
uv add --dev <package>       # development-only dependency
```

Commit both `pyproject.toml` and `uv.lock` after changing dependencies.
