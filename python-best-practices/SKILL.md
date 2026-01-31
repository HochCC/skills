---
name: python-best-practices
description: Python 代码最佳实践指南，涵盖代码风格、设计模式、性能优化、错误处理、测试和类型注解。当用户编写 Python 代码、请求代码审查、询问 Python 编码规范、或需要优化 Python 代码时使用此技能。
---

# Python Best Practices

## 代码风格

### 命名规范

| 类型 | 风格 | 示例 |
|------|------|------|
| 模块/包 | snake_case | `data_utils.py` |
| 类 | PascalCase | `DataProcessor` |
| 函数/变量 | snake_case | `process_data` |
| 常量 | UPPER_SNAKE | `MAX_RETRIES` |
| 私有 | 前缀 `_` | `_internal_state` |
| 魔术方法 | 双下划线 | `__init__` |

### 导入顺序

```python
# 1. Standard library
import os
from typing import Optional

# 2. Third-party libraries
import pandas as pd
from pydantic import BaseModel

# 3. Local modules
from .utils import helper
```

### 简洁原则

```python
# ❌ Verbose
if condition == True:
if len(items) > 0:
result = value if value is not None else None

# ✅ Concise
if condition:
if items:
result = value
```

### 函数设计原则

**核心：函数应当简洁、优美、单一职责。**

**规范：**
- 单一职责：一个函数只做一件事
- 长度限制：理想 < 20 行，最多 < 30 行
- 参数数量：理想 ≤ 3 个，超过时考虑封装为对象
- 嵌套深度：最多 2-3 层，过深则提取子函数
- 提前返回：用 guard clause 减少嵌套

```python
# ❌ Bad: nested, multiple responsibilities
def process_order(order: Order) -> Result:
    if order is not None:
        if order.items:
            total = 0
            for item in order.items:
                if item.quantity > 0:
                    price = item.price * item.quantity
                    if item.discount:
                        price = price * (1 - item.discount)
                    total += price
            if total > 0:
                return Result(success=True, total=total)
            else:
                return Result(success=False, error="Empty order")
        else:
            return Result(success=False, error="No items")
    else:
        return Result(success=False, error="Invalid order")

# ✅ Good: flat, single responsibility, early returns
def process_order(order: Order) -> Result:
    if order is None:
        return Result.failure("Invalid order")
    if not order.items:
        return Result.failure("No items")
    
    total = _calculate_total(order.items)
    if total <= 0:
        return Result.failure("Empty order")
    
    return Result.success(total=total)

def _calculate_total(items: list[Item]) -> Decimal:
    return sum(_item_price(item) for item in items if item.quantity > 0)

def _item_price(item: Item) -> Decimal:
    price = item.price * item.quantity
    return price * (1 - item.discount) if item.discount else price
```

**函数命名：**
- 动词开头：`get_`, `create_`, `update_`, `delete_`, `is_`, `has_`, `can_`
- 布尔返回：`is_valid()`, `has_permission()`, `can_edit()`
- 转换函数：`to_dict()`, `from_json()`, `as_string()`

## 注释与文档

**核心原则：所有 docstring 和 comments 必须使用英文。**

### 注释原则

**Language: English only.** All comments must be written in English.

**When to comment (only when necessary):**
- Complex algorithms or business logic
- Non-obvious workarounds or edge cases
- Important decisions with context (why, not what)
- TODO/FIXME with ticket reference

**When NOT to comment:**
- Obvious code (let code speak for itself)
- Restating what code does
- Commented-out code (delete it)

```python
# ❌ Bad: states the obvious
# Increment counter by 1
counter += 1

# ❌ Bad: restates the code
# Check if user is admin
if user.role == "admin":

# ✅ Good: explains WHY
# Rate limit bypass for internal health checks (see JIRA-1234)
if request.source == "internal":
    skip_rate_limit = True

# ✅ Good: documents non-obvious behavior
# Use insertion order to maintain priority (Python 3.7+ dict guarantee)
task_queue = {}

# ✅ Good: warns about edge case
# Note: returns None for empty input, not empty list
def first_match(items: list[T], pred: Callable) -> T | None: ...
```

### Docstring (Google Style, English Only)

**Language: English only.** All docstrings must be written in English.

**Required for:**
- Public modules, classes, functions, methods
- Complex private functions

**Not required for:**
- Simple/obvious functions with clear type hints
- Private helpers under 5 lines
- `__init__` if class docstring covers it

#### Module Docstring

```python
"""User authentication and session management.

This module provides JWT-based authentication, session handling,
and role-based access control for the API layer.

Typical usage:
    from auth import authenticate, require_role

    @require_role("admin")
    def admin_endpoint(): ...
"""
```

#### Function Docstring

```python
def fetch_user_orders(
    user_id: str,
    *,
    status: OrderStatus | None = None,
    limit: int = 100,
) -> list[Order]:
    """Fetch orders for a specific user.

    Retrieves orders from the database with optional filtering.
    Results are sorted by creation date (newest first).

    Args:
        user_id: The unique identifier of the user.
        status: Filter by order status. If None, returns all statuses.
        limit: Maximum number of orders to return. Must be 1-1000.

    Returns:
        List of Order objects matching the criteria.
        Empty list if user has no orders.

    Raises:
        UserNotFoundError: If user_id doesn't exist.
        ValueError: If limit is out of valid range.

    Example:
        >>> orders = fetch_user_orders("u123", status=OrderStatus.PENDING)
        >>> len(orders)
        5
    """
```

#### Class Docstring

```python
class OrderProcessor:
    """Process and validate customer orders.

    Handles order validation, inventory checks, and payment processing.
    Thread-safe for concurrent order processing.

    Attributes:
        max_retries: Maximum payment retry attempts.
        timeout: Payment gateway timeout in seconds.

    Example:
        >>> processor = OrderProcessor(max_retries=3)
        >>> result = processor.process(order)
        >>> result.status
        'completed'
    """

    def __init__(self, max_retries: int = 3, timeout: float = 30.0) -> None:
        self.max_retries = max_retries
        self.timeout = timeout
```

#### Short Form (simple functions)

```python
def calculate_tax(amount: Decimal, rate: Decimal) -> Decimal:
    """Calculate tax amount from base price and rate."""
    return amount * rate
```

### Inline Comments Style

```python
# Single line comment with space after #
x = 1  # Inline comment: 2 spaces before #

# TODO(username): Implement retry logic (JIRA-456)
# FIXME: Race condition when cache expires
# NOTE: Assumes UTC timezone
# HACK: Workaround for upstream bug (remove after v2.1)
```

## 类型注解

**核心原则：所有函数的入参和返回值必须标明类型。**

```python
from typing import TypeVar, Generic, Protocol
from collections.abc import Callable, Iterable

T = TypeVar("T")
K = TypeVar("K")
V = TypeVar("V")

# ❌ Bad: missing type hints
def process(data, callback=None):
    return {"result": data}

# ✅ Good: fully typed
def process(
    data: list[dict[str, Any]],
    *,
    callback: Callable[[str], None] | None = None,
) -> dict[str, list[int]]:
    ...

# Generic class
class Cache(Generic[K, V]):
    def get(self, key: K) -> V | None: ...

# Protocol (structural subtyping)
class Comparable(Protocol):
    def __lt__(self, other: Self) -> bool: ...
```

**类型注解规范：**
- 所有公共函数/方法必须有完整类型注解
- 私有函数也应标注类型（提高可维护性）
- 使用 `-> None` 明确标注无返回值
- 优先使用 `list`, `dict` 而非 `List`, `Dict` (Python 3.9+)
- 复杂类型使用 `TypeAlias` 定义别名

## 设计模式

### 数据类优先

```python
from dataclasses import dataclass, field
from pydantic import BaseModel, Field

# Simple data container
@dataclass(frozen=True, slots=True)
class Point:
    x: float
    y: float

# Use Pydantic when validation needed
class UserConfig(BaseModel):
    name: str = Field(..., min_length=1)
    retries: int = Field(default=3, ge=1, le=10)
```

### 上下文管理器

```python
from collections.abc import Iterator, AsyncIterator
from contextlib import contextmanager, asynccontextmanager

@contextmanager
def managed_resource(name: str) -> Iterator[Resource]:
    resource = acquire(name)
    try:
        yield resource
    finally:
        resource.release()

# Async version
@asynccontextmanager
async def async_session() -> AsyncIterator[Session]:
    session = await create_session()
    try:
        yield session
    finally:
        await session.close()
```

### 依赖注入

```python
from typing import Protocol, Any

class Repository(Protocol):
    def get(self, id: str) -> dict[str, Any] | None: ...

class Service:
    def __init__(self, repo: Repository) -> None:
        self._repo = repo

# Inject mock for testing
service = Service(MockRepository())
```

## 性能优化

### 数据结构选择

| 场景 | 推荐 |
|------|------|
| 快速查找 | `dict`, `set` |
| 有序+快速查找 | `dict` (3.7+保序) |
| 队列操作 | `collections.deque` |
| 计数 | `collections.Counter` |
| 默认值字典 | `collections.defaultdict` |
| 缓存结果 | `functools.lru_cache` |

### 生成器优先

```python
from collections.abc import Iterable, Iterator
from typing import Any

# ❌ Load all at once
def load_all(paths: list[str]) -> list[dict[str, Any]]:
    return [json.load(open(p)) for p in paths]

# ✅ Lazy loading
def load_lazy(paths: Iterable[str]) -> Iterator[dict[str, Any]]:
    for path in paths:
        with open(path) as f:
            yield json.load(f)
```

### 并发处理

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
import asyncio

# IO-bound: threads/async
async def fetch_all(urls: list[str]) -> list[str]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        return await asyncio.gather(*tasks)

# CPU-bound: multiprocessing
with ProcessPoolExecutor() as executor:
    results = list(executor.map(compute, data_chunks))
```

## 错误处理

**核心原则：让错误尽早暴露，只在必要处捕获异常。**

### Fail Fast 原则

```python
# ❌ Bad: swallowing errors, hiding bugs
def get_user(user_id: str) -> User | None:
    try:
        return db.query(User).get(user_id)
    except Exception:
        return None  # Hides database errors!

# ✅ Good: let errors propagate, handle at boundary
def get_user(user_id: str) -> User:
    user = db.query(User).get(user_id)
    if user is None:
        raise UserNotFoundError(user_id)
    return user
```

### 最小化 try 范围

```python
# ❌ Bad: try block too large
def process_file(path: str) -> Data:
    try:
        with open(path) as f:
            content = f.read()
        parsed = json.loads(content)
        validated = validate(parsed)
        return transform(validated)
    except Exception as e:
        logger.error(f"Failed: {e}")
        raise

# ✅ Good: narrow try scope, specific exceptions
def process_file(path: str) -> Data:
    with open(path) as f:  # Let FileNotFoundError propagate
        content = f.read()
    
    try:
        parsed = json.loads(content)
    except json.JSONDecodeError as e:
        raise InvalidFormatError(f"Invalid JSON: {e}") from e
    
    validated = validate(parsed)  # Let ValidationError propagate
    return transform(validated)
```

### 何时使用 try

| 场景 | 是否使用 try |
|------|-------------|
| 外部 IO (网络/文件) 需要恢复 | ✅ 是 |
| 转换用户输入格式 | ✅ 是 |
| 业务逻辑错误 | ❌ 否，直接抛出 |
| 编程错误 (bug) | ❌ 否，让其崩溃 |
| 调用第三方库边界 | ✅ 是，转换为领域异常 |

### 异常设计

```python
class AppError(Exception):
    """Base exception for application errors."""

class ValidationError(AppError):
    def __init__(self, field: str, message: str) -> None:
        self.field = field
        super().__init__(f"{field}: {message}")

class NotFoundError(AppError):
    pass
```

### 边界处捕获

```python
# Only catch at system boundaries (API endpoints, CLI, etc.)
@app.post("/users")
async def create_user(data: UserCreate) -> User:
    try:
        return await user_service.create(data)
    except ValidationError as e:
        raise HTTPException(400, detail=str(e))
    except DuplicateError as e:
        raise HTTPException(409, detail=str(e))
    # Let unexpected errors propagate to global handler
```

### 穷尽检查

```python
from typing import Never

def assert_never(x: Never) -> Never:
    """Helper for exhaustive type checking."""
    raise AssertionError(f"Unexpected: {x}")

def handle_status(status: Status) -> str:
    match status:
        case Status.PENDING:
            return "waiting"
        case Status.DONE:
            return "completed"
        case _:
            assert_never(status)  # Type error if enum extended
```

### 结构化日志

```python
import structlog

logger = structlog.get_logger()

logger.info(
    "request_processed",
    user_id=user.id,
    duration_ms=elapsed,
    status="success",
)
```

## 测试

### 结构

```
tests/
├── conftest.py       # Shared fixtures
├── unit/
│   └── test_*.py
├── integration/
│   └── test_*.py
└── fixtures/
    └── *.json
```

### Pytest 模式

```python
import pytest
from unittest.mock import Mock

# Parameterized tests
@pytest.mark.parametrize("input_val,expected", [
    ("a", 1),
    ("b", 2),
])
def test_mapping(input_val: str, expected: int) -> None:
    assert func(input_val) == expected

# Fixtures with type hints
@pytest.fixture
def mock_client() -> Mock:
    client = Mock(spec=APIClient)
    client.get.return_value = {"status": "ok"}
    return client

# Async tests
@pytest.mark.asyncio
async def test_async_op() -> None:
    result = await async_func()
    assert result.success

# Exception assertions
def test_raises() -> None:
    with pytest.raises(ValidationError, match="invalid"):
        validate(bad_data)
```

## 代码审查清单

**类型与签名：**
- [ ] 所有函数入参和返回值都有类型注解
- [ ] 类型注解准确反映实际行为

**函数设计：**
- [ ] 函数单一职责，< 30 行
- [ ] 嵌套深度 ≤ 3 层
- [ ] 使用 guard clause 提前返回

**错误处理：**
- [ ] 无裸 `except:`
- [ ] try 范围最小化
- [ ] 业务异常直接抛出，不吞掉
- [ ] 仅在边界处（API/CLI）捕获并转换异常

**代码质量：**
- [ ] 资源使用 context manager
- [ ] 无硬编码配置值
- [ ] 公共 API 有 docstring (Google style, English)
- [ ] 注释解释 why 而非 what
- [ ] 无冗余/过时注释
- [ ] 边界条件有测试覆盖

## 详细参考

- 设计模式详解见 [patterns.md](patterns.md)
- 完整代码示例见 [examples.md](examples.md)
