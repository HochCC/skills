---
description: Python coding standards - auto-apply when working with .py files
globs: ["**/*.py"]
alwaysApply: true
---

# Python Best Practices

## Core Principles

1. **Type Hints**: All function params and returns must have type annotations
2. **Fail Fast**: Raise errors immediately; do not catch unless recovery is required at a system boundary
3. **Clean Code**: Use guard clauses for early return
4. **English Only**: All docstrings and comments must be in English

---

## File Naming

- Use `snake_case`: `user_service.py`, `data_utils.py`
- Test files: `test_<module>.py`

---

## Type Annotations

```python
# ❌ Bad
def get_user(user_id):
    return db.get(user_id)

# ✅ Good
def get_user(user_id: str) -> User:
    user = db.get(user_id)
    if user is None:
        raise UserNotFoundError(user_id)
    return user
```

- Use `-> None` for functions without return
- Prefer `list`, `dict` over `List`, `Dict` (Python 3.9+)

---

## Error Handling

**Fail Fast - don't swallow errors:**

```python
# ❌ Bad: hides bugs
def get_user(user_id: str) -> User | None:
    try:
        return db.get(user_id)
    except Exception:
        return None

# ✅ Good: let errors propagate and expose the problem directly
def get_user(user_id: str) -> User:
    user = db.get(user_id)
    if user is None:
        raise UserNotFoundError(user_id)  # Direct exposure
    return user
```

**Avoid `try` unless absolutely necessary:**

- ✅ **Unavoidable external IO**: Network requests, file system access, database transactions.
- ✅ **Library boundaries**: When a third-party library might throw unexpected errors that you can recover from.
- ❌ **Business logic**: Never wrap business rules in `try`. Let exceptions bubble up and handle them at the API/UI boundary.
- ❌ **Development/Testing**: Never catch errors in test scripts or `__main__` blocks; tracebacks are your friends.

Keep the `try` scope as small as possible.

---

## Function Design

**Use guard clauses for early return:**

```python
# ❌ Bad: nested
def process(order: Order) -> Result:
    if order is not None:
        if order.items:
            total = sum(i.price for i in order.items)
            if total > 0:
                return Result(success=True, total=total)
    return Result(success=False)

# ✅ Good: guard clauses
def process(order: Order) -> Result:
    if order is None:
        return Result.failure("Invalid")
    if not order.items:
        return Result.failure("No items")
    
    return Result.success(total=_calc_total(order.items))
```

**Naming:**
- Actions: `get_`, `create_`, `update_`, `delete_`
- Boolean: `is_valid()`, `has_permission()`, `can_edit()`

---

## Documentation (English Only)

**Docstring - Google Style:**

```python
def fetch_orders(user_id: str, *, limit: int = 100) -> list[Order]:
    """Fetch orders for a user.

    Args:
        user_id: Unique identifier of the user.
        limit: Max orders to return.

    Returns:
        List of Order objects.

    Raises:
        UserNotFoundError: If user doesn't exist.
    """
```

**Simple functions - one line:**

```python
def calc_tax(amount: Decimal, rate: Decimal) -> Decimal:
    """Calculate tax from base price and rate."""
    return amount * rate
```

**Comments - explain WHY, not WHAT:**

```python
# ❌ Bad
# Check if admin
if user.role == "admin":

# ✅ Good
# Bypass rate limit for health checks (JIRA-1234)
if request.source == "internal":
```

---

## Design Patterns

**Dataclass / Pydantic:**

```python
@dataclass
class User:
    id: str
    name: str
    email: str

class UserConfig(BaseModel):  # Pydantic for validation
    name: str = Field(..., min_length=1)
    retries: int = Field(default=3, ge=1, le=10)
```

**Protocol (Interface):**

```python
class Repository(Protocol):
    def get(self, id: str) -> Entity | None: ...
    def save(self, entity: Entity) -> None: ...
```

**Dependency Injection:**

```python
class UserService:
    def __init__(self, repo: UserRepository, cache: Cache) -> None:
        self._repo = repo
        self._cache = cache
```

**Factory:**

```python
class SerializerFactory:
    _registry: ClassVar[dict[str, type[Serializer]]] = {}

    @classmethod
    def register(cls, name: str, serializer: type[Serializer]) -> None:
        cls._registry[name] = serializer

    @classmethod
    def create(cls, name: str) -> Serializer:
        return cls._registry[name]()
```

**Strategy:**

```python
class PricingStrategy(Protocol):
    def calculate(self, price: Decimal, qty: int) -> Decimal: ...

class Order:
    def __init__(self, pricing: PricingStrategy) -> None:
        self._pricing = pricing
```

**Custom Exceptions:**

```python
class AppError(Exception):
    """Base application error."""

class UserNotFoundError(AppError):
    def __init__(self, user_id: str) -> None:
        self.user_id = user_id
        super().__init__(f"User not found: {user_id}")
```

---

## After Writing Code

**Always run and verify:** Test the code after implementation to ensure correctness.

---

## Code Review Checklist

- [ ] All functions have type annotations (params + return)
- [ ] No bare `except:`, try scope minimized
- [ ] Business errors raised, only caught at boundaries
- [ ] Docstrings/comments in English, Google style
- [ ] Comments explain WHY, not WHAT
- [ ] Code tested and verified working
