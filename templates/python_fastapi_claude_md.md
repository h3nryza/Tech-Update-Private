# Python FastAPI — CLAUDE.md Template

> Copy this entire file into `.claude/CLAUDE.md` at your project root.
> Replace all `<!-- CUSTOMIZE -->` sections with your project-specific values.

---

```markdown
# CLAUDE.md

<!-- CUSTOMIZE: Replace with your project name and one-line description -->
## Project: My API
A FastAPI application with SQLAlchemy, Alembic migrations, and containerized deployment.

---

## Build & Run Commands

<!-- CUSTOMIZE: Pick either Poetry or uv — delete the section you don't use -->

### Package Manager: uv (Recommended)
```bash
uv sync                          # Install dependencies
uv run fastapi dev app/main.py   # Start dev server with auto-reload
uv run fastapi run app/main.py   # Start production server
uv add <package>                 # Add dependency
uv add --dev <package>           # Add dev dependency
uv lock                          # Update lock file
```

### Package Manager: Poetry (Alternative)
```bash
poetry install                   # Install dependencies
poetry run fastapi dev app/main.py   # Start dev server
poetry run fastapi run app/main.py   # Production server
poetry add <package>             # Add dependency
poetry add --group dev <package> # Add dev dependency
```

### Testing
```bash
uv run pytest                              # Run all tests
uv run pytest -x                           # Stop on first failure
uv run pytest -v tests/test_users.py       # Run specific test file
uv run pytest -k "test_create_user"        # Run tests matching pattern
uv run pytest --cov=app --cov-report=html  # Coverage report
uv run pytest -n auto                      # Parallel execution (pytest-xdist)
```

### Code Quality
```bash
uv run ruff check .              # Lint (ruff)
uv run ruff check . --fix        # Lint with auto-fix
uv run ruff format .             # Format (ruff formatter)
uv run mypy app/                 # Type checking
uv run bandit -r app/            # Security scanning
```

### Database
```bash
uv run alembic upgrade head          # Apply all migrations
uv run alembic downgrade -1          # Rollback last migration
uv run alembic revision --autogenerate -m "description"  # Create migration
uv run alembic history               # View migration history
uv run alembic current               # Show current revision
```

### Docker
```bash
docker compose up -d                 # Start all services
docker compose up -d --build         # Rebuild and start
docker compose down                  # Stop all services
docker compose logs -f app           # Follow app logs
docker compose exec app bash         # Shell into app container
```

---

## Code Conventions

### Python Style
- **Python 3.11+** — use modern syntax (type unions with `|`, `match` statements)
- **Strict typing** — all functions must have type annotations for parameters and return values
- Use `ruff` for linting and formatting (replaces black, isort, flake8)
- Line length: 88 characters (ruff default)
- Docstrings: Google style for public functions

```python
# GOOD: Fully typed with docstring
async def get_user_by_email(
    db: AsyncSession,
    email: str,
) -> User | None:
    """Fetch a user by their email address.

    Args:
        db: Async database session.
        email: The email to search for.

    Returns:
        The user if found, None otherwise.
    """
    stmt = select(User).where(User.email == email)
    result = await db.execute(stmt)
    return result.scalar_one_or_none()
```

### Import Ordering (handled by ruff)
```python
# 1. Standard library
from datetime import datetime
from typing import Annotated

# 2. Third-party
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

# 3. Local
from app.core.config import settings
from app.models.user import User
```

### Naming
- Files/modules: `snake_case.py`
- Classes: `PascalCase`
- Functions/variables: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Private methods: `_leading_underscore`
- Database tables: `snake_case` plural (`users`, `order_items`)
- Pydantic models: `PascalCase` with suffix — `UserCreate`, `UserResponse`, `UserUpdate`

---

## FastAPI Patterns

### Router Structure
```python
# app/api/v1/users.py
router = APIRouter(prefix="/users", tags=["users"])

@router.get("/", response_model=list[UserResponse])
async def list_users(
    db: Annotated[AsyncSession, Depends(get_db)],
    skip: int = 0,
    limit: int = Query(default=20, le=100),
) -> list[UserResponse]:
    """List users with pagination."""
    return await user_service.list_users(db, skip=skip, limit=limit)
```

### Dependency Injection
- Use `Depends()` for all shared logic (db sessions, auth, permissions)
- Define dependencies in `app/core/deps.py`
- Use `Annotated` type aliases for common dependency patterns

```python
# app/core/deps.py
from typing import Annotated

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session() as session:
        yield session

async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
    db: Annotated[AsyncSession, Depends(get_db)],
) -> User:
    # ... token validation logic
    return user

# Type aliases for clean signatures
DB = Annotated[AsyncSession, Depends(get_db)]
CurrentUser = Annotated[User, Depends(get_current_user)]
```

### Pydantic Models (Schemas)
- Separate models for Create, Update, Response, and DB
- Use `model_config = ConfigDict(from_attributes=True)` for ORM models
- Validate at the boundary — Pydantic handles request/response, SQLAlchemy handles DB

```python
# app/schemas/user.py
from pydantic import BaseModel, ConfigDict, EmailStr

class UserBase(BaseModel):
    email: EmailStr
    name: str

class UserCreate(UserBase):
    password: str

class UserUpdate(BaseModel):
    email: EmailStr | None = None
    name: str | None = None

class UserResponse(UserBase):
    id: int
    created_at: datetime
    model_config = ConfigDict(from_attributes=True)
```

### Error Handling
```python
# app/core/exceptions.py
from fastapi import HTTPException, status

class NotFoundError(HTTPException):
    def __init__(self, resource: str, id: str | int):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"{resource} with id {id} not found",
        )

class PermissionDeniedError(HTTPException):
    def __init__(self):
        super().__init__(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="You do not have permission to perform this action",
        )
```

---

## Project Structure

<!-- CUSTOMIZE: Adjust to match your actual layout -->

```
app/
  __init__.py
  main.py                    # FastAPI app factory, middleware, startup/shutdown
  core/
    config.py                # Settings (pydantic-settings, reads from env)
    deps.py                  # Dependency injection (DB, auth, etc.)
    security.py              # Password hashing, JWT creation/verification
    exceptions.py            # Custom exception classes
  api/
    v1/
      __init__.py            # v1 router aggregation
      users.py               # User endpoints
      auth.py                # Auth endpoints (login, register, refresh)
    health.py                # Health check endpoint
  models/
    base.py                  # SQLAlchemy declarative base, mixins
    user.py                  # User ORM model
  schemas/
    user.py                  # Pydantic request/response models
    common.py                # Shared schemas (pagination, errors)
  services/
    user_service.py          # Business logic layer
  repositories/
    user_repository.py       # Database query layer (optional, for complex queries)
alembic/
  versions/                  # Migration files
  env.py                     # Alembic environment config
alembic.ini
tests/
  conftest.py                # Fixtures (test DB, client, factory functions)
  test_users.py
  test_auth.py
  factories/                 # Factory Boy factories for test data
docker-compose.yml
Dockerfile
pyproject.toml               # Project config, dependencies, tool settings
```

---

## Testing

### Test Setup (conftest.py)
```python
# tests/conftest.py
import pytest
from httpx import ASGITransport, AsyncClient
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine

from app.main import create_app
from app.core.deps import get_db

@pytest.fixture
async def db_session():
    """Create a test database session with rollback."""
    engine = create_async_engine("sqlite+aiosqlite:///:memory:")
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    async with AsyncSession(engine) as session:
        yield session

@pytest.fixture
async def client(db_session):
    """Async test client with overridden DB dependency."""
    app = create_app()
    app.dependency_overrides[get_db] = lambda: db_session
    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test",
    ) as client:
        yield client
```

### Test Patterns
```python
# tests/test_users.py
import pytest
from httpx import AsyncClient

@pytest.mark.anyio
async def test_create_user(client: AsyncClient):
    response = await client.post("/api/v1/users/", json={
        "email": "alice@example.com",
        "name": "Alice",
        "password": "StrongP@ss1",
    })
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "alice@example.com"
    assert "password" not in data  # Never leak passwords

@pytest.mark.anyio
async def test_create_user_duplicate_email(client: AsyncClient):
    # Create first user
    await client.post("/api/v1/users/", json={...})
    # Try duplicate
    response = await client.post("/api/v1/users/", json={...})
    assert response.status_code == 409
```

### Coverage Requirements
- Minimum 85% overall coverage
- 100% coverage for `core/security.py` and auth flows
- Run `uv run pytest --cov=app --cov-fail-under=85`

---

## Database Conventions

### SQLAlchemy Models
```python
# app/models/base.py
from datetime import datetime
from sqlalchemy import DateTime, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

class Base(DeclarativeBase):
    pass

class TimestampMixin:
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now()
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now(), onupdate=func.now()
    )
```

### Migration Rules
- Never edit existing migration files — create new ones
- Always review autogenerated migrations before applying
- Migration names must be descriptive: `add_user_role_column`, not `update_1`
- Test migrations against a copy of production data before deploying

---

## Configuration & Secrets

```python
# app/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # <!-- CUSTOMIZE: Add your settings -->
    database_url: str
    redis_url: str = "redis://localhost:6379"
    secret_key: str
    access_token_expire_minutes: int = 30
    debug: bool = False

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
    )

settings = Settings()
```

- `.env` is gitignored — never commit secrets
- `.env.example` is committed with placeholder values
- Production secrets come from environment variables or secret managers

---

## Deployment

### Docker
```dockerfile
# Dockerfile
FROM python:3.12-slim AS base
WORKDIR /app
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

FROM base AS deps
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev

FROM base AS runtime
COPY --from=deps /app/.venv /app/.venv
COPY app/ app/
COPY alembic/ alembic/
COPY alembic.ini .
ENV PATH="/app/.venv/bin:$PATH"
EXPOSE 8000
CMD ["fastapi", "run", "app/main.py", "--host", "0.0.0.0", "--port", "8000"]
```

### Pre-deployment Checklist
1. `uv run ruff check .` passes
2. `uv run mypy app/` passes
3. `uv run pytest --cov-fail-under=85` passes
4. `uv run bandit -r app/` shows no high-severity issues
5. `uv run alembic upgrade head` succeeds against staging DB
6. Docker image builds and health check passes

---

## Common Tasks

### Add a new endpoint
1. Create/update router in `app/api/v1/your_resource.py`
2. Add Pydantic schemas in `app/schemas/your_resource.py`
3. Add service logic in `app/services/your_resource_service.py`
4. Register router in `app/api/v1/__init__.py`
5. Write tests in `tests/test_your_resource.py`

### Add a new database model
1. Create model in `app/models/your_model.py`
2. Import in `app/models/__init__.py`
3. Run `uv run alembic revision --autogenerate -m "add_your_model_table"`
4. Review the generated migration
5. Run `uv run alembic upgrade head`

### Add background tasks
```python
from fastapi import BackgroundTasks

@router.post("/send-report")
async def send_report(
    background_tasks: BackgroundTasks,
    current_user: CurrentUser,
):
    background_tasks.add_task(generate_and_send_report, current_user.id)
    return {"status": "Report generation started"}
```
```
