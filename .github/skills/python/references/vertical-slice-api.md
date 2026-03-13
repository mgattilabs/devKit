# Vertical Slice Architecture for APIs

## Overview

Vertical slice organizes code by **feature**, not by technical layer. Each slice contains everything needed for a single use case: route, schema, service, repository.

```
# ❌ Traditional layered (scattered)
routes/
    users.py
    orders.py
schemas/
    users.py
    orders.py
services/
    users.py
    orders.py

# ✅ Vertical slice (cohesive)
features/
    users/
        router.py
        schemas.py
        service.py
        repository.py
    orders/
        router.py
        schemas.py
        service.py
        repository.py
```

## Project Structure

```
my-api/
├── pyproject.toml
├── Makefile
├── configs/
│   ├── settings.yaml
│   └── settings.dev.yaml
├── tests/
│   ├── conftest.py
│   └── features/
│       ├── users/
│       │   └── test_users.py
│       └── orders/
│           └── test_orders.py
└── src/
    └── my_api/
        ├── __init__.py
        ├── main.py              # FastAPI app entry point
        ├── config.py            # Settings with Pydantic
        ├── database.py          # DB session management
        ├── dependencies.py      # Shared FastAPI dependencies
        │
        ├── shared/              # Cross-cutting concerns
        │   ├── __init__.py
        │   ├── exceptions.py    # Custom exceptions
        │   ├── middleware.py    # Auth, logging, etc.
        │   └── models.py        # Base SQLAlchemy model
        │
        └── features/
            ├── __init__.py
            │
            ├── users/
            │   ├── __init__.py
            │   ├── router.py      # FastAPI routes
            │   ├── schemas.py     # Pydantic models (request/response)
            │   ├── service.py     # Business logic
            │   ├── repository.py  # Data access
            │   └── models.py      # SQLAlchemy models
            │
            └── orders/
                ├── __init__.py
                ├── router.py
                ├── schemas.py
                ├── service.py
                ├── repository.py
                └── models.py
```

## Implementation Example

### main.py - App Entry Point

```python
"""FastAPI application entry point."""

from fastapi import FastAPI

from my_api.config import settings
from my_api.features.users.router import router as users_router
from my_api.features.orders.router import router as orders_router
from my_api.shared.middleware import setup_middleware


def create_app() -> FastAPI:
    app = FastAPI(
        title=settings.app_name,
        version=settings.version,
        debug=settings.debug,
    )

    setup_middleware(app)

    app.include_router(users_router, prefix="/users", tags=["users"])
    app.include_router(orders_router, prefix="/orders", tags=["orders"])

    return app


app = create_app()
```

### config.py - Settings

```python
"""Application configuration."""

from functools import lru_cache

from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
    )

    app_name: str = "My API"
    version: str = "1.0.0"
    debug: bool = False

    database_url: str = "sqlite:///./app.db"
    secret_key: str = "change-me-in-production"
    access_token_expire_minutes: int = 30


@lru_cache
def get_settings() -> Settings:
    return Settings()


settings = get_settings()
```

### database.py - Session Management

```python
"""Database session management."""

from collections.abc import Generator

from sqlalchemy import create_engine
from sqlalchemy.orm import Session, sessionmaker

from my_api.config import settings

engine = create_engine(
    settings.database_url,
    connect_args={"check_same_thread": False},  # SQLite only
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)


def get_db() -> Generator[Session, None, None]:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### dependencies.py - Shared Dependencies

```python
"""Shared FastAPI dependencies."""

from typing import Annotated

from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.orm import Session

from my_api.database import get_db
from my_api.features.users.repository import UserRepository
from my_api.features.users.schemas import UserRead

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="users/login")

DBSession = Annotated[Session, Depends(get_db)]
Token = Annotated[str, Depends(oauth2_scheme)]


def get_current_user(db: DBSession, token: Token) -> UserRead:
    user_repo = UserRepository(db)
    user = user_repo.get_by_token(token)

    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
        )

    return UserRead.model_validate(user)


CurrentUser = Annotated[UserRead, Depends(get_current_user)]
```

---

## Feature Slice Example: Users

### features/users/schemas.py

```python
"""User request/response schemas."""

from pydantic import BaseModel, EmailStr, Field


class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8)
    full_name: str = Field(min_length=1, max_length=100)


class UserRead(BaseModel):
    id: int
    email: EmailStr
    full_name: str
    is_active: bool

    model_config = {"from_attributes": True}


class UserUpdate(BaseModel):
    full_name: str | None = None
    password: str | None = Field(default=None, min_length=8)


class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"
```

### features/users/models.py

```python
"""User SQLAlchemy models."""

from sqlalchemy import Boolean, Column, Integer, String

from my_api.shared.models import Base


class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True, nullable=False)
    hashed_password = Column(String, nullable=False)
    full_name = Column(String, nullable=False)
    is_active = Column(Boolean, default=True)
```

### features/users/repository.py

```python
"""User data access layer."""

from sqlalchemy.orm import Session

from my_api.features.users.models import User
from my_api.features.users.schemas import UserCreate, UserUpdate


class UserRepository:
    def __init__(self, db: Session) -> None:
        self._db = db

    def get_by_id(self, user_id: int) -> User | None:
        return self._db.query(User).filter(User.id == user_id).first()

    def get_by_email(self, email: str) -> User | None:
        return self._db.query(User).filter(User.email == email).first()

    def create(self, data: UserCreate, hashed_password: str) -> User:
        user = User(
            email=data.email,
            hashed_password=hashed_password,
            full_name=data.full_name,
        )
        self._db.add(user)
        self._db.commit()
        self._db.refresh(user)
        return user

    def update(self, user: User, data: UserUpdate) -> User:
        update_data = data.model_dump(exclude_unset=True)

        for field, value in update_data.items():
            setattr(user, field, value)

        self._db.commit()
        self._db.refresh(user)
        return user

    def delete(self, user: User) -> None:
        self._db.delete(user)
        self._db.commit()
```

### features/users/service.py

```python
"""User business logic."""

from fastapi import HTTPException, status
from passlib.context import CryptContext
from sqlalchemy.orm import Session

from my_api.features.users.models import User
from my_api.features.users.repository import UserRepository
from my_api.features.users.schemas import UserCreate, UserUpdate

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


class UserService:
    def __init__(self, db: Session) -> None:
        self._repository = UserRepository(db)

    def create_user(self, data: UserCreate) -> User:
        existing = self._repository.get_by_email(data.email)
        if existing:
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Email already registered",
            )

        hashed_password = pwd_context.hash(data.password)
        return self._repository.create(data, hashed_password)

    def authenticate(self, email: str, password: str) -> User:
        user = self._repository.get_by_email(email)

        if not user or not pwd_context.verify(password, user.hashed_password):
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Incorrect email or password",
            )

        return user

    def get_user(self, user_id: int) -> User:
        user = self._repository.get_by_id(user_id)

        if not user:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="User not found",
            )

        return user

    def update_user(self, user_id: int, data: UserUpdate) -> User:
        user = self.get_user(user_id)

        if data.password:
            data.password = pwd_context.hash(data.password)

        return self._repository.update(user, data)
```

### features/users/router.py

```python
"""User API routes."""

from fastapi import APIRouter, Depends, status
from fastapi.security import OAuth2PasswordRequestForm

from my_api.database import get_db
from my_api.dependencies import CurrentUser, DBSession
from my_api.features.users.schemas import TokenResponse, UserCreate, UserRead, UserUpdate
from my_api.features.users.service import UserService

router = APIRouter()


def get_user_service(db: DBSession) -> UserService:
    return UserService(db)


@router.post("", response_model=UserRead, status_code=status.HTTP_201_CREATED)
def create_user(
    data: UserCreate,
    service: UserService = Depends(get_user_service),
) -> UserRead:
    user = service.create_user(data)
    return UserRead.model_validate(user)


@router.post("/login", response_model=TokenResponse)
def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    service: UserService = Depends(get_user_service),
) -> TokenResponse:
    user = service.authenticate(form_data.username, form_data.password)
    # In production: generate JWT token here
    return TokenResponse(access_token=f"token-for-{user.id}")


@router.get("/me", response_model=UserRead)
def get_current_user_info(current_user: CurrentUser) -> UserRead:
    return current_user


@router.patch("/me", response_model=UserRead)
def update_current_user(
    data: UserUpdate,
    current_user: CurrentUser,
    service: UserService = Depends(get_user_service),
) -> UserRead:
    user = service.update_user(current_user.id, data)
    return UserRead.model_validate(user)
```

---

## Shared Components

### shared/models.py

```python
"""Base SQLAlchemy model."""

from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass
```

### shared/exceptions.py

```python
"""Custom exceptions."""

from fastapi import HTTPException, status


class NotFoundError(HTTPException):
    def __init__(self, resource: str, resource_id: int | str) -> None:
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"{resource} with id {resource_id} not found",
        )


class ConflictError(HTTPException):
    def __init__(self, detail: str) -> None:
        super().__init__(
            status_code=status.HTTP_409_CONFLICT,
            detail=detail,
        )
```

### shared/middleware.py

```python
"""Application middleware."""

import time

from fastapi import FastAPI, Request
from starlette.middleware.cors import CORSMiddleware


def setup_middleware(app: FastAPI) -> None:
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],  # Configure for production
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )

    @app.middleware("http")
    async def add_process_time_header(request: Request, call_next):
        start_time = time.perf_counter()
        response = await call_next(request)
        process_time = time.perf_counter() - start_time
        response.headers["X-Process-Time"] = str(process_time)
        return response
```
