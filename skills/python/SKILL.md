---
name: python-clean-code
description: Write clean, maintainable Python code following PEP 8, type hints, and pragmatic clean architecture. Use when writing Python code, creating new Python projects, refactoring existing code, or reviewing Python implementations. Triggers on Python file creation, code generation requests, project scaffolding, or refactoring tasks.
---

# Python Clean Code

Write clean, maintainable Python code with proper structure and conventions.

## Core Principles

1. **Type hints always** - Every function signature and class attribute
2. **Self-explanatory code** - Comments explain *why*, not *what*
3. **Small focused functions** - Single responsibility, early returns
4. **Flat project structure** - Virtual layers, not rigid folders


### Data Structures

```python
# Small structures → dataclass
@dataclass
class Point:
    x: float
    y: float

# If Pydantic available → use it for validation
class UserCreate(BaseModel):
    email: str
    name: str = Field(min_length=1)

# Complex behavior → regular class
class OrderProcessor:
    def __init__(self, repo: Repository) -> None:
        self._repo = repo
```

### Function Design

```python
# ✅ Good: early return, clear naming
def get_user_discount(user: User, order: Order) -> Decimal:
    if not user or not order:
        return Decimal("0")

    if order.total <= MIN_ORDER_FOR_DISCOUNT:
        return Decimal("0")

    return VIP_DISCOUNT if user.is_vip else STANDARD_DISCOUNT


# ❌ Bad: nested, unclear
def calc(u, o):
    d = 0
    if u:
        if o:
            if o.total > 100:
                if u.is_vip:
                    d = 0.2
    return d
```

### Expressive Conditions

```python
# ✅ Extract complex conditions
is_business_hours = 9 <= current_hour <= 17
is_weekday = current_day < 6
can_process = is_business_hours and is_weekday and not is_holiday

if can_process:
    process_order()
```

### List Comprehensions

```python
# Good: List comprehension for simple transformations
names = [user.name for user in users if user.is_active]

# Bad: Manual loop
names = []
for user in users:
    if user.is_active:
        names.append(user.name)

# Complex comprehensions should be expanded
# Bad: Too complex
result = [x * 2 for x in items if x > 0 if x % 2 == 0]

# Good: Use a generator function
def filter_and_transform(items: Iterable[int]) -> list[int]:
    result = []
    for x in items:
        if x > 0 and x % 2 == 0:
            result.append(x * 2)
    return result
```

### Generator Expressions

```python
# Good: Generator for lazy evaluation
total = sum(x * x for x in range(1_000_000))

# Bad: Creates large intermediate list
total = sum([x * x for x in range(1_000_000)])
```

### Generator Functions

```python
def read_large_file(path: str) -> Iterator[str]:
    """Read a large file line by line."""
    with open(path) as f:
        for line in f:
            yield line.strip()

# Usage
for line in read_large_file("huge.txt"):
    process(line)
```

### Generator for Large Data

```python
# Bad: Returns full list in memory
def read_lines(path: str) -> list[str]:
    with open(path) as f:
        return [line.strip() for line in f]

# Good: Yields lines one at a time
def read_lines(path: str) -> Iterator[str]:
    with open(path) as f:
        for line in f:
            yield line.strip()
```

### Avoid String Concatenation in Loops

```python
# Bad: O(n²) due to string immutability
result = ""
for item in items:
    result += str(item)

# Good: O(n) using join
result = "".join(str(item) for item in items)

# Good: Using StringIO for building
from io import StringIO

buffer = StringIO()
for item in items:
    buffer.write(str(item))
result = buffer.getvalue()
```

## Anti-Patterns to Avoid

```python
# Bad: Mutable default arguments
def append_to(item, items=[]):
    items.append(item)
    return items

# Good: Use None and create new list
def append_to(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# Bad: Checking type with type()
if type(obj) == list:
    process(obj)

# Good: Use isinstance
if isinstance(obj, list):
    process(obj)

# Bad: Comparing to None with ==
if value == None:
    process()

# Good: Use is
if value is None:
    process()

# Bad: from module import *
from os.path import *

# Good: Explicit imports
from os.path import join, exists

# Bad: Bare except
try:
    risky_operation()
except:
    pass

# Good: Specific exception
try:
    risky_operation()
except SpecificError as e:
    logger.error(f"Operation failed: {e}")
```

__Remember__: Python code should be readable, explicit, and follow the principle of least surprise. When in doubt, prioritize clarity over cleverness.