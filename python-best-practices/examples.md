# Python Best Practices - Code Examples

## Project Structure

```
my_project/
├── src/
│   └── my_project/
│       ├── __init__.py
│       ├── core/
│       │   ├── __init__.py
│       │   ├── models.py
│       │   └── services.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── routes.py
│       │   └── schemas.py
│       └── utils/
│           ├── __init__.py
│           └── helpers.py
├── tests/
│   ├── conftest.py
│   ├── unit/
│   └── integration/
├── pyproject.toml
└── README.md
```

## pyproject.toml Configuration

```toml
[project]
name = "my-project"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "pydantic>=2.0",
    "httpx>=0.25",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-asyncio>=0.21",
    "pytest-cov>=4.0",
    "mypy>=1.0",
    "ruff>=0.1",
]

[tool.ruff]
line-length = 88
target-version = "py311"
select = ["E", "F", "I", "N", "UP", "B", "A", "C4", "SIM"]

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
addopts = "-v --cov=src --cov-report=term-missing"
```

## Complete Service Class Example

```python
"""User service module.

Provides user management functionality including creation, retrieval,
and updates. Uses repository pattern for data access abstraction.
"""
from __future__ import annotations

import logging
from dataclasses import dataclass, field
from datetime import datetime, UTC
from typing import Protocol
from uuid import UUID, uuid4

from pydantic import BaseModel, EmailStr, Field

logger = logging.getLogger(__name__)


# ============ Data Models ============

class UserCreate(BaseModel):
    """Request model for user creation."""

    email: EmailStr
    name: str = Field(..., min_length=1, max_length=100)


class User(BaseModel):
    """User entity with timestamps."""

    id: UUID
    email: EmailStr
    name: str
    created_at: datetime
    updated_at: datetime


# ============ Repository Interface ============

class UserRepository(Protocol):
    """Abstract repository for user persistence."""

    async def get_by_id(self, user_id: UUID) -> User | None: ...
    async def get_by_email(self, email: str) -> User | None: ...
    async def create(self, user: User) -> User: ...
    async def update(self, user: User) -> User: ...


# ============ Custom Exceptions ============

class UserError(Exception):
    """Base exception for user service errors."""


class UserNotFoundError(UserError):
    def __init__(self, user_id: UUID) -> None:
        self.user_id = user_id
        super().__init__(f"User not found: {user_id}")


class EmailExistsError(UserError):
    def __init__(self, email: str) -> None:
        self.email = email
        super().__init__(f"Email already registered: {email}")


# ============ Service Implementation ============

@dataclass
class UserService:
    """Service for user management operations.

    Handles user creation, retrieval, and updates with validation.
    Thread-safe for concurrent access.

    Attributes:
        repo: Repository implementation for user persistence.
    """

    repo: UserRepository

    async def create_user(self, data: UserCreate) -> User:
        """Create a new user.

        Args:
            data: User creation request with email and name.

        Returns:
            The created User entity with generated ID and timestamps.

        Raises:
            EmailExistsError: If email is already registered.
        """
        existing = await self.repo.get_by_email(data.email)
        if existing is not None:
            raise EmailExistsError(data.email)

        now = datetime.now(UTC)
        user = User(
            id=uuid4(),
            email=data.email,
            name=data.name,
            created_at=now,
            updated_at=now,
        )

        created = await self.repo.create(user)
        logger.info("User created", extra={"user_id": str(created.id)})
        return created

    async def get_user(self, user_id: UUID) -> User:
        """Retrieve a user by ID.

        Raises:
            UserNotFoundError: If user does not exist.
        """
        user = await self.repo.get_by_id(user_id)
        if user is None:
            raise UserNotFoundError(user_id)
        return user

    async def update_name(self, user_id: UUID, new_name: str) -> User:
        """Update user's display name."""
        user = await self.get_user(user_id)
        updated = user.model_copy(
            update={
                "name": new_name,
                "updated_at": datetime.now(UTC),
            }
        )
        return await self.repo.update(updated)
```

## Complete Test Example

```python
"""Unit tests for user service."""
import pytest
from datetime import datetime, UTC
from unittest.mock import AsyncMock, Mock
from uuid import uuid4

from my_project.core.models import User, UserCreate
from my_project.core.services import (
    UserService,
    UserNotFoundError,
    EmailExistsError,
)


@pytest.fixture
def mock_repo() -> AsyncMock:
    """Create mock repository."""
    return AsyncMock()


@pytest.fixture
def service(mock_repo: AsyncMock) -> UserService:
    """Create service instance with mock dependencies."""
    return UserService(repo=mock_repo)


@pytest.fixture
def sample_user() -> User:
    """Sample user for testing."""
    now = datetime.now(UTC)
    return User(
        id=uuid4(),
        email="test@example.com",
        name="Test User",
        created_at=now,
        updated_at=now,
    )


class TestCreateUser:
    """Tests for user creation."""

    async def test_creates_new_user(
        self,
        service: UserService,
        mock_repo: AsyncMock,
    ) -> None:
        mock_repo.get_by_email.return_value = None
        mock_repo.create.side_effect = lambda u: u

        result = await service.create_user(
            UserCreate(email="new@example.com", name="New User")
        )

        assert result.email == "new@example.com"
        assert result.name == "New User"
        mock_repo.create.assert_awaited_once()

    async def test_raises_when_email_exists(
        self,
        service: UserService,
        mock_repo: AsyncMock,
        sample_user: User,
    ) -> None:
        mock_repo.get_by_email.return_value = sample_user

        with pytest.raises(EmailExistsError) as exc_info:
            await service.create_user(
                UserCreate(email=sample_user.email, name="Another")
            )

        assert exc_info.value.email == sample_user.email


class TestGetUser:
    """Tests for user retrieval."""

    async def test_returns_existing_user(
        self,
        service: UserService,
        mock_repo: AsyncMock,
        sample_user: User,
    ) -> None:
        mock_repo.get_by_id.return_value = sample_user

        result = await service.get_user(sample_user.id)

        assert result == sample_user

    async def test_raises_when_not_found(
        self,
        service: UserService,
        mock_repo: AsyncMock,
    ) -> None:
        user_id = uuid4()
        mock_repo.get_by_id.return_value = None

        with pytest.raises(UserNotFoundError) as exc_info:
            await service.get_user(user_id)

        assert exc_info.value.user_id == user_id


@pytest.mark.parametrize(
    "name,expected",
    [
        ("Alice", "Alice"),
        ("  Bob  ", "  Bob  "),  # No auto-trim
        ("名前", "名前"),  # Unicode support
    ],
)
async def test_update_name_variations(
    service: UserService,
    mock_repo: AsyncMock,
    sample_user: User,
    name: str,
    expected: str,
) -> None:
    mock_repo.get_by_id.return_value = sample_user
    mock_repo.update.side_effect = lambda u: u

    result = await service.update_name(sample_user.id, name)

    assert result.name == expected
```

## API Routes Example (FastAPI)

```python
"""User API routes for FastAPI."""
from typing import Annotated
from uuid import UUID

from fastapi import APIRouter, Depends, HTTPException, status

from my_project.core.models import User, UserCreate
from my_project.core.services import (
    UserService,
    UserNotFoundError,
    EmailExistsError,
)
from .deps import get_user_service

router = APIRouter(prefix="/users", tags=["users"])


@router.post("", response_model=User, status_code=status.HTTP_201_CREATED)
async def create_user(
    data: UserCreate,
    service: Annotated[UserService, Depends(get_user_service)],
) -> User:
    """Create a new user account."""
    try:
        return await service.create_user(data)
    except EmailExistsError as e:
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail=str(e),
        )


@router.get("/{user_id}", response_model=User)
async def get_user(
    user_id: UUID,
    service: Annotated[UserService, Depends(get_user_service)],
) -> User:
    """Retrieve user details by ID."""
    try:
        return await service.get_user(user_id)
    except UserNotFoundError:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User {user_id} not found",
        )
```

## Configuration Management Example

```python
"""Application configuration management."""
from functools import cache
from pathlib import Path

from pydantic import Field, PostgresDsn, SecretStr
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    """Application settings loaded from environment variables.

    Supports loading from .env file with automatic type conversion
    and validation. All settings are case-insensitive.

    Attributes:
        app_name: Display name for the application.
        debug: Enable debug mode with verbose logging.
        database_url: PostgreSQL connection string.
        secret_key: Secret key for JWT signing.
    """

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
    )

    # Application
    app_name: str = "MyProject"
    debug: bool = False

    # Database
    database_url: PostgresDsn
    db_pool_size: int = Field(default=5, ge=1, le=20)

    # Redis
    redis_url: str = "redis://localhost:6379/0"

    # Security
    secret_key: SecretStr
    jwt_expire_minutes: int = 60 * 24  # 24 hours

    # External services
    api_key: SecretStr | None = None


@cache
def get_settings() -> Settings:
    """Get cached settings singleton."""
    return Settings()  # type: ignore
```

## Async Task Processing Example

```python
"""Background task processing with async queue."""
import asyncio
from collections.abc import Callable, Coroutine
from dataclasses import dataclass, field
from typing import Any
import logging

logger = logging.getLogger(__name__)


@dataclass
class TaskQueue:
    """Simple async task queue with worker pool.

    Provides a lightweight background task system using asyncio.
    Supports graceful shutdown and error isolation per task.

    Attributes:
        _queue: Internal async queue for pending tasks.
        _workers: List of active worker tasks.
        _running: Flag indicating if queue is accepting tasks.
    """

    _queue: asyncio.Queue[Callable[[], Coroutine[Any, Any, None]]] = field(
        default_factory=asyncio.Queue
    )
    _workers: list[asyncio.Task[None]] = field(default_factory=list)
    _running: bool = False

    async def start(self, num_workers: int = 3) -> None:
        """Start worker pool.

        Args:
            num_workers: Number of concurrent workers to spawn.
        """
        self._running = True
        self._workers = [
            asyncio.create_task(self._worker(i))
            for i in range(num_workers)
        ]
        logger.info(f"Started {num_workers} workers")

    async def stop(self) -> None:
        """Stop queue and cancel all workers gracefully."""
        self._running = False
        for worker in self._workers:
            worker.cancel()
        await asyncio.gather(*self._workers, return_exceptions=True)
        logger.info("Task queue stopped")

    async def enqueue(
        self,
        coro_func: Callable[[], Coroutine[Any, Any, None]],
    ) -> None:
        """Add a task to the queue."""
        await self._queue.put(coro_func)

    async def _worker(self, worker_id: int) -> None:
        """Worker loop that processes tasks from queue."""
        while self._running:
            try:
                coro_func = await asyncio.wait_for(
                    self._queue.get(),
                    timeout=1.0,
                )
                try:
                    await coro_func()
                except Exception:
                    logger.exception(f"Worker {worker_id}: task failed")
                finally:
                    self._queue.task_done()
            except asyncio.TimeoutError:
                continue


# Usage example
async def main() -> None:
    queue = TaskQueue()
    await queue.start()

    async def send_email(to: str) -> None:
        await asyncio.sleep(0.1)  # Simulate sending
        print(f"Email sent to {to}")

    await queue.enqueue(lambda: send_email("user@example.com"))
    await queue._queue.join()  # Wait for all tasks to complete
    await queue.stop()
```
