# Python Coding Style Reference

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Variables | `snake_case` | `user_name`, `total_count` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT`, `API_URL` |
| Functions | `snake_case` | `get_user_by_id()`, `calculate_total()` |
| Classes | `PascalCase` | `UserService`, `DataProcessor` |
| Modules/Files | `snake_case` | `user_service.py`, `data_loader.py` |
| Private | `_single_underscore` | `_internal_cache`, `_helper()` |
| "Very private" | `__double_underscore` | `__private_value` |

## Type Hints (Required)

Always use type hints for function signatures and class attributes.

```python
# Function signatures
def process_items(items: list[str], limit: int = 10) -> dict[str, int]:
    ...

# Class attributes
class UserConfig:
    api_key: str
    timeout: int = 30
    enabled: bool = True

# Complex types
from typing import Optional, Callable, TypeVar

T = TypeVar("T")

def find_first(items: list[T], predicate: Callable[[T], bool]) -> Optional[T]:
    ...

# Union types (Python 3.10+)
def parse_input(value: str | int | None) -> str:
    ...
```

## Data Structures Selection

### Use `dataclass` for small, simple structures:

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float

@dataclass(frozen=True)
class Config:
    host: str
    port: int = 8080
```

### Use `Pydantic` when available in project dependencies:

```python
from pydantic import BaseModel, Field

class UserCreate(BaseModel):
    email: str
    name: str = Field(min_length=1, max_length=100)
    age: int = Field(ge=0, le=150)

class UserResponse(BaseModel):
    id: int
    email: str
    name: str

    model_config = {"from_attributes": True}
```

### Use regular classes for complex behavior:

```python
class OrderProcessor:
    def __init__(self, repository: OrderRepository, notifier: Notifier) -> None:
        self._repository = repository
        self._notifier = notifier

    def process(self, order: Order) -> ProcessingResult:
        ...
```

## String Formatting

Prefer f-strings:

```python
# ✅ Good
message = f"User {user.name} has {len(items)} items"

# ❌ Avoid
message = "User {} has {} items".format(user.name, len(items))
message = "User %s has %d items" % (user.name, len(items))
```

## Imports

Always place imports at the top of the file, immediately after module docstrings.

```python
"""Module docstring goes first."""

# Standard library
import os
from pathlib import Path
from typing import Optional

# Third-party
import httpx
from pydantic import BaseModel

# Local
from myproject.domain.entities import User
from myproject.infrastructure.database import Repository
```

Rules:
- Group imports: stdlib → third-party → local
- One import per line for `from x import y`
- Avoid `from x import *`

## Constants and Magic Numbers

```python
# ❌ Bad
if elapsed_seconds > 86400:
    expire_session()

# ✅ Good
SECONDS_IN_A_DAY = 86400

if elapsed_seconds > SECONDS_IN_A_DAY:
    expire_session()
```

## Comments

Only explain **why**, never **what**:

```python
# ❌ Bad - explains what
# Increment counter
counter += 1

# ✅ Good - explains why
# Skip CSV header row
counter += 1

# ✅ Good - non-obvious decision
# High timeout needed: external API has variable latency during peak hours (see INC-2847)
response = httpx.get(url, timeout=30)
```

## Docstrings

Use for public APIs, not for obvious methods:

```python
def calculate_compound_interest(
    principal: float,
    rate: float,
    periods: int,
    compounds_per_period: int = 12,
) -> float:
    """Calculate compound interest over time.

    Args:
        principal: Initial investment amount.
        rate: Annual interest rate as decimal (e.g., 0.05 for 5%).
        periods: Number of years.
        compounds_per_period: Compounding frequency per year.

    Returns:
        Final amount after compound interest.

    Raises:
        ValueError: If principal or rate is negative.
    """
    if principal < 0 or rate < 0:
        raise ValueError("Principal and rate must be non-negative")

    return principal * (1 + rate / compounds_per_period) ** (compounds_per_period * periods)
```

## Nested Functions

Never define functions inside other functions. Extract them as module-level functions or methods.

```python
# ❌ Bad - nested function
def process_orders(orders: list[Order]) -> list[Result]:
    def calculate_total(order: Order) -> Decimal:
        return sum(item.price * item.qty for item in order.items)

    def apply_discount(total: Decimal, customer: Customer) -> Decimal:
        if customer.is_vip:
            return total * Decimal("0.9")
        return total

    results = []
    for order in orders:
        total = calculate_total(order)
        final = apply_discount(total, order.customer)
        results.append(Result(order_id=order.id, total=final))
    return results


# ✅ Good - module-level functions
def _calculate_order_total(order: Order) -> Decimal:
    return sum(item.price * item.qty for item in order.items)


def _apply_customer_discount(total: Decimal, customer: Customer) -> Decimal:
    if customer.is_vip:
        return total * Decimal("0.9")
    return total


def process_orders(orders: list[Order]) -> list[Result]:
    results = []
    for order in orders:
        total = _calculate_order_total(order)
        final = _apply_customer_discount(total, order.customer)
        results.append(Result(order_id=order.id, total=final))
    return results
```
