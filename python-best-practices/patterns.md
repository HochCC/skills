# Python Design Patterns

## Creational Patterns

### Factory Pattern

```python
from abc import ABC, abstractmethod
from typing import ClassVar

class Serializer(ABC):
    @abstractmethod
    def serialize(self, data: dict) -> str: ...

class JSONSerializer(Serializer):
    def serialize(self, data: dict) -> str:
        return json.dumps(data)

class XMLSerializer(Serializer):
    def serialize(self, data: dict) -> str:
        return dict_to_xml(data)

class SerializerFactory:
    _registry: ClassVar[dict[str, type[Serializer]]] = {
        "json": JSONSerializer,
        "xml": XMLSerializer,
    }

    @classmethod
    def register(cls, name: str, serializer_cls: type[Serializer]) -> None:
        cls._registry[name] = serializer_cls

    @classmethod
    def create(cls, format: str) -> Serializer:
        if format not in cls._registry:
            raise ValueError(f"Unknown format: {format}")
        return cls._registry[format]()
```

### Singleton (prefer module-level instance)

```python
# Recommended: module-level singleton
# config.py
_config: Config | None = None

def get_config() -> Config:
    global _config
    if _config is None:
        _config = Config.load()
    return _config

# Or use functools.cache
@cache
def get_config() -> Config:
    return Config.load()
```

### Builder Pattern

```python
from dataclasses import dataclass, field

@dataclass
class QueryBuilder:
    _select: list[str] = field(default_factory=list)
    _where: list[str] = field(default_factory=list)
    _order_by: str | None = None

    def select(self, *columns: str) -> "QueryBuilder":
        self._select.extend(columns)
        return self

    def where(self, condition: str) -> "QueryBuilder":
        self._where.append(condition)
        return self

    def order_by(self, column: str) -> "QueryBuilder":
        self._order_by = column
        return self

    def build(self) -> str:
        query = f"SELECT {', '.join(self._select)}"
        if self._where:
            query += f" WHERE {' AND '.join(self._where)}"
        if self._order_by:
            query += f" ORDER BY {self._order_by}"
        return query

# Usage
query = (
    QueryBuilder()
    .select("id", "name")
    .where("active = true")
    .order_by("created_at")
    .build()
)
```

## Structural Patterns

### Decorator Pattern

```python
import functools
import time
from collections.abc import Callable
from typing import TypeVar, ParamSpec

P = ParamSpec("P")
R = TypeVar("R")

def retry(
    max_attempts: int = 3,
    delay: float = 1.0,
    exceptions: tuple[type[Exception], ...] = (Exception,),
) -> Callable[[Callable[P, R]], Callable[P, R]]:
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        @functools.wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            last_exception: Exception | None = None
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e
                    if attempt < max_attempts - 1:
                        time.sleep(delay * (2 ** attempt))
            raise last_exception  # type: ignore
        return wrapper
    return decorator

def timed(func: Callable[P, R]) -> Callable[P, R]:
    @functools.wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        start = time.perf_counter()
        try:
            return func(*args, **kwargs)
        finally:
            elapsed = time.perf_counter() - start
            logger.debug(f"{func.__name__} took {elapsed:.3f}s")
    return wrapper
```

### Adapter Pattern

```python
from typing import Protocol

class ModernAPI(Protocol):
    def fetch_data(self, query: dict) -> list[dict]: ...

class LegacySystem:
    def get_records(self, sql: str) -> list[tuple]: ...

class LegacyAdapter:
    def __init__(self, legacy: LegacySystem) -> None:
        self._legacy = legacy

    def fetch_data(self, query: dict) -> list[dict]:
        sql = self._query_to_sql(query)
        rows = self._legacy.get_records(sql)
        return [self._row_to_dict(r) for r in rows]

    def _query_to_sql(self, query: dict) -> str: ...
    def _row_to_dict(self, row: tuple) -> dict: ...
```

## Behavioral Patterns

### Strategy Pattern

```python
from typing import Protocol

class PricingStrategy(Protocol):
    def calculate(self, base_price: float, quantity: int) -> float: ...

class RegularPricing:
    def calculate(self, base_price: float, quantity: int) -> float:
        return base_price * quantity

class BulkPricing:
    def __init__(self, threshold: int, discount: float) -> None:
        self.threshold = threshold
        self.discount = discount

    def calculate(self, base_price: float, quantity: int) -> float:
        if quantity >= self.threshold:
            return base_price * quantity * (1 - self.discount)
        return base_price * quantity

class Order:
    def __init__(self, pricing: PricingStrategy) -> None:
        self._pricing = pricing

    def total(self, base_price: float, quantity: int) -> float:
        return self._pricing.calculate(base_price, quantity)
```

### Observer Pattern

```python
from typing import Generic, TypeVar
from collections.abc import Callable

T = TypeVar("T")

class Observable(Generic[T]):
    def __init__(self) -> None:
        self._observers: list[Callable[[T], None]] = []

    def subscribe(self, callback: Callable[[T], None]) -> Callable[[], None]:
        self._observers.append(callback)
        return lambda: self._observers.remove(callback)

    def notify(self, data: T) -> None:
        for observer in self._observers:
            observer(data)

# Usage
events: Observable[dict] = Observable()
unsubscribe = events.subscribe(lambda e: print(e))
events.notify({"type": "user_login", "user_id": 123})
unsubscribe()
```

### State Machine

```python
from enum import Enum, auto
from typing import ClassVar

class OrderState(Enum):
    PENDING = auto()
    CONFIRMED = auto()
    SHIPPED = auto()
    DELIVERED = auto()
    CANCELLED = auto()

class Order:
    _transitions: ClassVar[dict[OrderState, set[OrderState]]] = {
        OrderState.PENDING: {OrderState.CONFIRMED, OrderState.CANCELLED},
        OrderState.CONFIRMED: {OrderState.SHIPPED, OrderState.CANCELLED},
        OrderState.SHIPPED: {OrderState.DELIVERED},
        OrderState.DELIVERED: set(),
        OrderState.CANCELLED: set(),
    }

    def __init__(self) -> None:
        self._state = OrderState.PENDING

    @property
    def state(self) -> OrderState:
        return self._state

    def transition(self, new_state: OrderState) -> None:
        if new_state not in self._transitions[self._state]:
            raise ValueError(
                f"Invalid transition: {self._state} -> {new_state}"
            )
        self._state = new_state
```

## Functional Patterns

### Pipe / Composition

```python
from collections.abc import Callable
from functools import reduce
from typing import TypeVar

T = TypeVar("T")

def pipe(*funcs: Callable[[T], T]) -> Callable[[T], T]:
    def piped(value: T) -> T:
        return reduce(lambda v, f: f(v), funcs, value)
    return piped

# Usage
process = pipe(
    str.strip,
    str.lower,
    lambda s: s.replace(" ", "_"),
)
result = process("  Hello World  ")  # "hello_world"
```

### Result Type

```python
from dataclasses import dataclass
from typing import TypeVar, Generic

T = TypeVar("T")
E = TypeVar("E", bound=Exception)

@dataclass(frozen=True)
class Ok(Generic[T]):
    value: T

@dataclass(frozen=True)
class Err(Generic[E]):
    error: E

Result = Ok[T] | Err[E]

def safe_divide(a: float, b: float) -> Result[float, ValueError]:
    if b == 0:
        return Err(ValueError("Division by zero"))
    return Ok(a / b)

# Usage
match safe_divide(10, 2):
    case Ok(value):
        print(f"Result: {value}")
    case Err(error):
        print(f"Error: {error}")
```
